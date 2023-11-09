# Improving the Development Experience with Loft & DevSpace

![DevOps Header](./assets/images/devops-header.png)

## Introduction

The success of any software project relies on providing developers with the tools and resources they need to work efficiently and effectively. This is particularly true when it comes to [Kubernetes](https://kubernetes.io/), which is a powerful platform for building and deploying applications. However, Kubernetes can be complex and challenging to work with, especially for developers who are new to the platform. 

As a result, providing a good developer experience is essential to ensure successful Kubernetes projects.  In order to provide a good developer experience with Kubernetes, it's important to offer tools and resources that help developers work efficiently and effectively.  Let's discuss an excellent tool to help improve the developer experience with Kubernetes: [Loft](https://loft.sh).

## What is Loft?

**Loft** is a platform that can help improve the development experience for Kubernetes, but it does so by simplifying cluster management. By providing a self-service portal for developers and teams to create and manage their own namespaces and resources, Loft streamlines the deployment process and reduces the complexity of managing Kubernetes clusters. This means that developers can focus more on coding and less on cluster management.  Loft usese tools such a DevSpace and vcluster to create a comprehensive development environment that allows developers to build, test, and deploy applications directly in Kubernetes, without having to worry about the underlying infrastructure.

With Loft, developers can create and manage their own isolated environments, work collaboratively with team members, and deploy applications with confidence, knowing that they are using a secure, reliable, and scalable platform.

### DevSpace and vclusters?  What's are those?

**DevSpace** is a developer tool that helps streamline the Kubernetes development workflow by providing a local development environment that's synced with a remote Kubernetes cluster. With features like hot reloading, debugging, and CI/CD integration, DevSpace allows developers to work on their code locally and see changes reflected in the cluster in real-time.  

Another concept to not overlook is the fact Loft provides a virtual Kubernetes cluster environment called **vCluster** that runs inside a namespace of the underlying Kubernetes cluster. This allows users to test and deploy applications without the need for physical hardware or cloud resources, providing a lightweight and flexible development environment. Using virtual clusters can help reduce costs and improve productivity by providing a cost-effective alternative to creating separate full-blown clusters. Additionally, virtual clusters offer better multi-tenancy and isolation than regular namespaces, making them an ideal solution for teams that need to test and deploy applications in a shared environment.  We'll create one as part of this tutorial.

## Pre-requisites

Before we get started, we'll need to install the following tools:

- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) - configure it with your AWS credentials
- [Docker](https://docs.docker.com/get-docker/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- [Go](https://go.dev/) - Optional; we'll be using the Docker containers for this tutorial

## Enhancing the developer experience

Loft is can be our all-in-one solution for managing your Kubernetes clusters and workloads. With its user-friendly interface, we can keep everything in check without breaking a sweat.  It can also be our one-stop-shop for handling namespaces, users, and [RBAC policies](https://www.strongdm.com/rbac). Loft also provides:

- Collaboration features: Loft includes several collaboration features, such as team-based access controls, shared namespaces, and RBAC policies, which can help streamline collaboration among developers and teams.

- Plugin ecosystem: Loft has a growing plugin ecosystem, which provides additional functionality for managing your Kubernetes clusters and workloads, such as backup and restore tools, monitoring and logging integrations, and more.

Let's take a look at how we can use DevSpace and Loft together to improve the developer experience but first, let's create the AWS infrastructure we'll need for this tutorial.

### Creating the AWS infrastructure

With an AWS account with console access, we'll need to create a few resources.  These resources won't qualify for the Amazon free tier so it's important we delete all resources at the end of the tutorial.  If we don't, leaving the [Amazon Elastic Kubernetes Service](https://docs.aws.amazon.com/whitepapers/latest/overview-deployment-options/amazon-elastic-kubernetes-service.html) or EKS cluster running continously will at least incur a $1 per day.  We'll be using the AWS CLI to create these resources, but we can use the AWS console if we prefer.  To install the AWS CLI, follow the instructions for your OS on the [AWS CLI website](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html). Once the AWS CLI is installed, you can verify that it is working by running the following `aws --version` command

Next, we'll need to configure the AWS CLI with our AWS credentials. To do this, run the following command:

```bash
aws configure
```

This will prompt us for our AWS access key ID and secret access key, which we can find in the [AWS console](https://aws.amazon.com).  Once we've entered your credentials, we can verify that the AWS CLI is configured correctly by running the `aws sts get-caller-identity` command.  This should return our AWS account ID and user name.

Now that we have the AWS CLI configured, we can create the resources we need for this tutorial.  First, create a VPC and one or more subnets for our EKS cluster. When creating our VPC and subnets, it's recommended to use separate subnets for our control plane and worker nodes, and to configure our VPC to use private IP addresses only.  To create a VPC and subnets, run the following commands:

```bash
VPC_ID=$(aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16 \
  --tag-specification "ResourceType=vpc,Tags=[{Key=Name,Value=MyVpc}]" \
  --query 'Vpc.VpcId' \
  --output text)

SUBNET_1=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-2a \
  --query 'Subnet.SubnetId' \
  --output text)
SUBNET_2=$(aws ec2 create-subnet \
  --vpc-id $VPC_ID \
  --cidr-block 10.0.2.0/24 \
  --availability-zone us-east-2b \
  --query 'Subnet.SubnetId' \
  --output text)
```

Next, let's create an internet gateway and attach it to the VPC that you created previously. This will allow resources in the VPC to access the internet and vice versa:

```bash
IGW_ID=$(aws ec2 create-internet-gateway \
  --query 'InternetGateway.InternetGatewayId' \
  --output text)

aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID
```

Create a route table for the VPC and associate it with the subnets we created earlier. This will allow resources in the subnets to communicate with each other and with the internet. You can create a route table and associate it with the subnets by running the following commands:

```bash
ROUTE_TABLE_ID=$(aws ec2 create-route-table \
  --vpc-id $VPC_ID \
  --query 'RouteTable.RouteTableId' \
  --output text)

aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_1
aws ec2 associate-route-table --route-table-id $ROUTE_TABLE_ID --subnet-id $SUBNET_2
aws ec2 create-route --route-table-id $ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID
```

After creating our VPC and subnets, create a security group for our EKS cluster. When creating our security group, ensure that it allows inbound and outbound traffic on the necessary ports and protocols for our EKS cluster and any associated applications.  To create a security group, run the following command:

```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name eks-cluster-sg \
  --description "EKS cluster security group" \
  --vpc-id $VPC_ID \
  --query 'GroupId' \
  --output text)

aws ec2 authorize-security-group-ingress \
  --group-id $SG_ID \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

Next, let's create a cluster on EKS us the `eksctl` tool we installed as a pre-requisite. This will be the penulminate step we'll use for your AWS root or admin account for.  This step may take up to 20 minutes to complete:

```bash
eksctl create cluster \
  --name development \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --managed
```

While we wait for that complete, let's open another terminal window. We will use this eks-cluster-role to create the EKS cluster to authenticate with the cluster.  Let's add a custom policy to this role:

```bash
aws iam create-user --user-name eks-admin

AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
echo '{
  "Version": "2012-10-17",
  "Statement": {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::'$AWS_ACCOUNT_ID':user/eks-admin"
      },
      "Action": "sts:AssumeRole"
    }
}' >| eks-cluster-role-trust-policy.json
aws iam create-role --role-name eks-cluster-role --assume-role-policy-document file://eks-cluster-role-trust-policy.json

echo '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "eks:DescribeCluster",
                "ecr:BatchGetImage",
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:DescribeImages",
                "ecr:DescribeRepositories",
                "ecr:GetDownloadUrlForLayer",
                "ecr:InitiateLayerUpload",
                "ecr:ListImages",
                "ecr:PutImage",
                "ecr:UploadLayerPart",
                "ecr:GetAuthorizationToken"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "eks:*"
                "iam:ListRoles",
                "sts:AssumeRole"
            ],
            "Resource": "*"
        }
    ]
}' >| eks-role-policy.json
aws iam put-role-policy --role-name eks-cluster-role --policy-name eks-role-policy --policy-document file://eks-role-policy.json

aws iam attach-role-policy --role-name eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy
aws iam attach-role-policy --role-name eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSServicePolicy
aws iam attach-role-policy --role-name eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSVPCResourceController
aws iam attach-role-policy --role-name eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy
aws iam attach-role-policy --role-name eks-cluster-role --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly

echo '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::'$AWS_ACCOUNT_ID':role/eks-cluster-role"
        }
    ]
}' >| eks-admin-assume-cluster-role-policy.json
aws iam put-user-policy --user-name eks-admin --policy-name eks-admin --policy-document file://eks-admin-assume-cluster-role-policy.json
```

Now that we have the IAM role and policy created, let's create a user that will assume the eks-cluster-role.  To do this, run the following commands:

```bash
# Create the AWS CLI profile for the EKS admin by exporting the credentials.
# We'll then later use the '--profile eks-admin' flag to use these credentials in further steps
CREDS=$(aws iam create-access-key --user-name eks-admin --query 'AccessKey.{AccessKeyId:AccessKeyId,SecretAccessKey:SecretAccessKey}' --output text)
KEY_ID=$(echo $CREDS| awk '{print $1}')
ACCESS_KEY=$(echo $CREDS| awk '{print $2}')

echo "
[eks-admin]
aws_access_key_id=$KEY_ID
aws_secret_access_key=$ACCESS_KEY
region=us-east-2" >> ~/.aws/credentials
```

We want to do work under the context of the IAM user and role we created earlier:

```bash
# To use the temporary credentials to authenticate with the cluster, we'll assume the eks-cluster-role
# using the eks-admin credentials. For the purposes of this tutorial, we'll use a 60 minute session
TEMP_CREDS=$(aws sts assume-role \
  --role-arn arn:aws:iam::$AWS_ACCOUNT_ID:role/eks-cluster-role \
  --duration-seconds 3600 \
  --role-session-name eks-admin \
  --profile eks-admin)

# NOTE: these credentials will persist in your shell environment for the duration of the session
# You can unset them by running the following commands: unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN.
# Otherwise, your default and other profiles will not work until you restart your shell.
export AWS_ACCESS_KEY_ID=$(echo $TEMP_CREDS | jq -r '.Credentials.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo $TEMP_CREDS | jq -r '.Credentials.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo $TEMP_CREDS | jq -r '.Credentials.SessionToken')

# All of your AWS CLI, kubectl, and Helm install/upgrade commands will now use the temporary credentials.
# To use the eks-admin to authenticate with the cluster, we'll need to update the kubeconfig file while using the temporary credentials:
aws eks update-kubeconfig --name development

# Using our root accout's profile we'll need to set the eks-admin credentials to create an IAM identity mapping for the eks-cluster-role
eksctl create iamidentitymapping \
  --cluster development \
  --arn arn:aws:iam::$AWS_ACCOUNT_ID:role/eks-cluster-role \
  --username eks-admin \
  --group system:masters \
  --region us-east-2 \
  --profile default
```

Next let's install Loft via Helm.

### Installing Loft via Helm

**Helm** is a package manager for Kubernetes that allows you to package and deploy Kubernetes applications. Helm charts are a collection of files that describe a related set of Kubernetes resources. Helm charts can be used to deploy applications, but they can also be used to deploy other types of Kubernetes resources which are out of scope for this article.

To install Helm, follow the instructons for your OS on the [Helm website](https://helm.sh/docs/intro/install/). Once Helm is installed, you can verify that it is working by running the following command:

```bash
helm version
```

Helm charts are stored in a Helm chart repository, which is a collection of Helm charts. Helm chart repositories can be hosted on a public or private server, or they can be hosted on a cloud storage service, such as Amazon S3 or Google Cloud Storage. Helm chart repositories can be accessed using the Helm CLI, which can be installed using the following command:

```bash
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
```

Once the Helm CLI is installed, we can add a Helm chart repository.  To get started, we'll need to install Loft to our cluster for demonstration on how it would work on a remote instance.  To do this, we'll first need to add the Loft Helm repository to our local Helm installation.  To do this, run the following command in your terminal:

```bash
helm repo add loft https://charts.loft.sh
helm repo update
```

Using namespaces is useful when we need to logically separate workloads within a single cluster, but still want to share resources such as nodes and network infrastructure. Namespace-level resource quotas can also be applied to prevent any one workload from using up too much of the shared resources.

While Kubernetes namespaces are used to isolate workloads and resources within a single cluster, creating separate clusters with creates isolated environments with their own Kubernetes API server, etcd storage, and worker nodes. Each cluster can have its own configuration, resource allocation, and security settings. Separate clusters are also useful when you need to create isolated environments for different purposes, such as development, testing, or production. Each cluster can have its own resources and configurations, making it easier to manage and isolate workloads.

With that said, let's create a namespace for our loft resources with the `kubectl` command.  The `kubectl` command is a necessary command-line tool for interacting with Kubernetes clusters.  To create a these resources, run the following commands:

```bash
kubectl create namespace loft
helm install loft loft/loft --namespace loft
```

It may take the container a bit to start up, but once it's running, we can verify that it's running by running the following command:

```bash
kubectl get pods --namespace loft
```

## Getting Started with Loft

Once Loft is installed, let's expose it to our local machine so that we can access it.  We can then access the Loft UI by running the following command in our terminal:

```bash
kubectl port-forward service/loft 8080:80 --namespace loft &
```

Now, we can access the Loft UI at [https://localhost:8080](https://localhost:8080). The default username and password is `admin` and the default password is `my-password`.

After we've added user details, we can create a new cluster by clicking on the `Virtual Clusters` link in the sidebar.  vClusters in Loft provide a way to create separate virtual Kubernetes clusters within a single Kubernetes cluster.  Once we see the `Virtual Clusters Management` page, we can create a new virtual cluster by clicking on the `Create Virutal Cluster` button.  Next, select `Isolated Virtual Cluster Template` option.  In the subsequent dialog, if the YAML isn't shown by default, we can click on the `Show YAML` button at the bottom of the page, center-right.  Once the configuration appears, modify the `name` in the *metadata* block to `loft-default-v-tutorial-vcluster`.  

Within the right-hand dialog, in the **Add Permission to:** field, select the user you created when you configured Loft on initial login.  Click the *+* button to add the user to the list.  We can now click the **Create** button to create the virtual cluster.

## Getting started locally with DevSpace

**Loft** is particularly useful when working with **DevSpace** as it provides a streamlined development workflow that enables developers to work on their code locally and see changes reflected in the virtual cluster in real-time.

By using DevSpace with Loft, developers can benefit from features such as hot reloading, debugging, and integration with popular IDEs like Visual Studio Code. These features can significantly enhance the developer experience and improve productivity, as developers can focus on writing code and testing their applications, rather than dealing with the complexities of Kubernetes infrastructure.

However, it's worth noting that DevSpace can be used with any Kubernetes cluster, whether it's running locally (e.g., [minikube](https://minikube.sigs.k8s.io), [Kind](https://kind.sigs.k8s.io/), etc.) or in the cloud via EKS. DevSpace is a versatile tool that can be used to streamline the Kubernetes development workflow and provide a more efficient and productive development experience.

Here are the general steps to get started:

### 1.1

Install DevSpace by following the installation instructions on the DevSpace website: https://devspace.sh/docs/getting-started/installation.  For example, on Linux, you can install DevSpace by running the following command in your terminal: `curl -L -o devspace "https://github.com/loft-sh/devspace/releases/latest/download/devspace-linux-amd64" && sudo install -c -m 0755 devspace /usr/local/bin`.

Once DevSpace is installed, you can verify that it's working by running the following command in your terminal:

```bash
devspace version
```

If verified, let's switch the namespace to the virtual cluster we created earlier.  To do this, run the following command in your terminal:

```bash
aws eks update-kubeconfig --name development  # update kubeconfig to point to your cluster
devspace use namespace loft-default-v-tutorial-vcluster 
```

Yes, virtual clusters are namespeces.  They are a powerful feature of the Loft platform that allows you to create a dedicated namespace with its own set of resources and policies. This can help you isolate your application and manage your resources more efficiently. If you want to learn more about virtual clusters, you can check out the [Loft documentation](https://loft.sh/docs/).

### 1.2

Once you have both DevSpace installed, let's create a new project.  For this example, let's use Go.  Follow these steps to create a Hello World project to deploy to our cluster:

```bash
mkdir hello-world && cd $_
go mod init hello-world
echo 'package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello World!")
}

func main() {
	fmt.Println("Started server on :3000")

	http.HandleFunc("/", handler)
	http.ListenAndServe(":3000", nil)
}' >| main.go

# Now let's create a Makefile to help typing commands less painful
echo '.PHONY: build clean run deploy

build:
	go mod verify && go mod tidy
	go build -ldflags="-s -w" -o bin/hello_world ./main.go

clean:
	rm -rf ./bin
	mkdir -p ./bin
    
run: clean build
	./bin/hello_world' >| Makefile

make run & # run the server in the background
curl localhost:3000 # should return "Hello World!"

# NOTE: Whenever you want stop the server running in the background, 
# execute `kill %1` in the terminal

## Finally, let's build our Dockerfile
echo 'FROM golang:1.20-alpine as build

WORKDIR /opt/app

# cache dependencies
COPY go.*mod* ./
RUN go mod download

# build
COPY *.go ./
RUN CGO_ENABLED=0 go build -ldflags="-s -w" -o ./bin/hello_world ./main.go

## Deploy
FROM gcr.io/distroless/base-debian10

WORKDIR /
COPY --from=build /opt/app/bin/hello_world /bin/hello_world
EXPOSE 3000
USER nonroot:nonroot
CMD [ "./bin/hello_world" ]' >| Dockerfile

echo '*.sh
.devspace/
Dockerfile' >| .dockerignore

git init .
echo '
*.swp' >> .gitignore

git add .
git commit -m "Initial commit"

# build our Docker image
docker build -t go-hello-world:latest .

AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REPOSITORY_PATH=$AWS_ACCOUNT_ID.dkr.ecr.us-east-2.amazonaws.com
aws ecr create-repository --repository-name go-hello-world
aws ecr get-login-password | docker login --username AWS --password-stdin $REPOSITORY_PATH

docker tag go-hello-world:latest $REPOSITORY_PATH/go-hello-world:latest
docker push $REPOSITORY_PATH/go-hello-world:latest
```

To test our Docker image works as expected, execute `docker run -p 3000:3000 go-hello-world` in the terminal.  In another terminal window, `curl localhost:3000` to see the output.

### 1.3

Now that we have a Docker image that we can deploy to our cluster, let's create a Helm chart.

```bash
helm create go-hello-world

# Replace the values.yml file with the following:
sed -i "s/repository: nginx/repository: $REPOSITORY_PATH\/go-hello-world:latest/g" go-hello-world/values.yaml
sed -i 's/port: 80/port: 3000/g' go-hello-world/values.yaml

# Replace the deployment.yml file with the following:
echo "apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-hello-world
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-hello-world
  template:
    metadata:
      labels:
        app: go-hello-world
    spec:
      containers:
      - name: go-hello-world
        image: $REPOSITORY_PATH/go-hello-world:latest
        ports:
        - containerPort: 3000" >| go-hello-world/templates/deployment.yaml
```

### 1.4

Let's create a new DevSpace project by running the following command in our terminal `devspace init`.  This will create a new DevSpace configuration file (e.g., `devspace.yaml`) in the current directory. When prompted, select the following options:

```cli
? Select a template:  *Go*
? How do you want to deploy this project?  *Helm Chart*
? Do you already have a Helm chart for this project?  *Yes*
? Which Helm chart do you want to use?  *local*
? Please enter the relative path to the Helm chart:  *./go-hello-world*
? Do you want to develop this probject with DevSpace or just deploy it? *I want to develop this project and my current working dir contains the source code*
? Which image do you want to develop with DevSpace?  Manually enter the image I want to work on
Manually enter the image I want to work on  *<REPOSITORY-PATH>go-hello-world:latest*
? How should DevSpace build the container image for this project?  Use this existing Dockerfile: *./Dockerfile*
```

Note: replace `<REPOSITORY-PATH>` with the repository path from the previous step.

### 1.5

Finally, let's run `devspace dev` to start our local development environment.  This will deploy our application to the local kubernetes cluster in the virutal cluster namespace.  It will also sync our local files with the cluster, so any changes we make to our code will be reflected in the cluster in real-time.  If you get a '[screen is terminated]' message, run `devspace sync &` to resolve the issue and then run `devspace dev` again.  You will then be logged into the container and can start developing your application with changes syncing to your local files in real-time.  

In another terminal window, you can view the deployed app in the cluster by running the following:

```bash
kubectl get pods --namespace loft-default-v-tutorial-vcluster
```

For security reasons, the port of our application isn't forwarded.  To forward the port of our application, run the following:

```bash
kubectl port-forward deployment/go-hello-world 3000:3000 --namespace loft-default-v-tutorial-vcluster &
curl localhost:3000  #  you should see "Hello, world!"
```

The purpose of running DevSpace is to create a local development environment that closely mirrors your other environments, allowing you to develop and test your application locally before deploying it to a higher shared environment (staging/qa/production). This is particularly useful in the context of Kubernetes, as it allows you to test your application in a containerized environment and ensure that it will work correctly when deployed to a cluster.  Changes you make to your local code should sync to your locally deployed container and vice-versa, allowing you to test your application in real-time.

Other than using the terminal connection to your container, you can also use a web interface to make changes, view the logs of your application, and debug any issues.  To access the web interface, run `devspace ui` in the terminal.  This will open a new browser window with the DevSpace dashboard.  You can view the logs of your application by clicking on the 'Logs' tab and selecting the 'go-hello-world' container.  You can also view the logs of your application by running `devspace logs go-hello-world` in the terminal.  However, it is very limited in functionality, so it is recommended that you use Loft's web interface instead.

In addition, Devspace provides a number of other features that can be useful in a local development environment, such as automatically restarting your application when you make changes to your code (see, ['Configure Auto-Reloading'](https://www.devspace.sh/docs/5.x/configuration/development/auto-reloading)), [providing access to logs and debugging information](#logging), and [allowing you to easily switch between different versions of your application](#profiles).

Overall, DevSpace provides a powerful tool for local development and testing of Kubernetes applications, allowing you to catch and fix issues early in the development process and reduce the risk of issues when deploying to a production environment.

## Integrating with CI/CD tools

DevSpace can be integrated with a number of popular CI/CD tools, such as GitHub Actions, GitLab CI/CD, and Jenkins.  This allows you to automate the deployment of your application to a Kubernetes cluster, which can be useful in a number of scenarios, such as deploying your application to a staging environment after a pull request is merged, or deploying your application to a production environment after a release is created.

Tools like [Flux](https://fluxcd.io/) and [Argo CD](https://argoproj.github.io/) are also commonly used for automating the deployment of Kubernetes applications.  However, these tools are typically used for deploying applications to a production environment, as they are designed to be used in a continuous delivery (CD) pipeline, where the application is deployed to a production environment after a release is created.  Outside of the scope of this tutorial but worth mentioning.

## Logging

Devspace provides you with access to the logs generated by your application and its underlying infrastructure, such as the container runtime, the Kubernetes cluster, and any other services that your application depends on. These logs are typically used for debugging and troubleshooting purposes, as they contain valuable information about the behavior of your application and the environment it's running in.

Additionally, Devspace allows you to configure log forwarding to external services such as Elasticsearch, Fluentd, or Datadog, where you can analyze and visualize your logs in more detail. This can be useful for monitoring and alerting purposes, as well as for gaining deeper insights into the behavior of your application and the environment it's running in.  This can be done by:

```bash
devspace logs -f
```

Subsequently, you should see output like the following:

```cli
> devspace logs -f
info Using namespace 'loft-default-v-tutorial-vcluster'
info Using kube context 'arn:aws:eks:us-east-2:xxxxxx:cluster/development'
info Printing logs of pod:container go-hello-world-devspace-xxxxx-xxxxx:go-hello-world
```

Devspace provides a powerful set of tools for logging and debugging your application, making it easier to diagnose issues and improve the quality of your code. By providing access to logs and debugging information, Devspace helps you get the most out of your local development environment, and streamlines the development process.

## Profiles

We often need to test different versions of the code to see how they behave and to compare their performance when developing applications. For example, we may need to test a new feature or fix a bug in an older version of our code. Switching between different versions of our application can be time-consuming and error-prone, especially if you need to manually manage multiple code branches, containers, and environments.

Devspace makes it easy to switch between different versions of our application by providing a feature called "profiles". A profile is a set of configuration files that define the environment variables, container images, and other parameters that our application needs to run. By defining different profiles for each version of our code, we can easily switch between them without having to manually manage multiple containers or environments.  For an example, for a monorepo we can have a profile for each service in the repo.  For example, in our `.devspace/config.yaml` file we can have the following:

```yaml
images:
  backend:
    image: my-backend-image
    tag: latest
  frontend:
    image: my-frontend-image
    tag: latest
```

Subsequently, we can switch between different versions of our application by simply executing `devspace use profile <profile-name>`, where `<profile-name>` is the name of the profile we want to use.

As an another example, we might define a "production" profile that uses the latest stable version of our code, and a "development" profile that uses the latest version of our code from the development branch. When we switch between these profiles, Devspace will automatically create or update the required containers, volumes, and other resources for us.

Devspace's profile feature provides a powerful tool for managing and switching between different versions of your application, making it easier to test and iterate on your code. By allowing you to easily switch between different profiles, Devspace streamlines the development process and helps you get the most out of our local development environment.

## Cleanup

To clean up the resources created by this tutorial, run the following commands:

```bash
killall -s 9 kubectl  # kill the port forwarding process
killall -s 9 devspace  # kill the sync process

# Delete the DevSpace deployment and vCluster VM
devspace purge && devspace reset vars
devspace cleanup images
devspace use namespace default  # or whatever namespace you want to switch to
kubectl delete namespace loft-default-v-tutorial-vcluster

eksctl delete cluster --name development & # this will take a bit to do, so run it in the background or another terminal

# Deleting the AWS resources we created
aws iam delete-role-policy --role-name eks-cluster-role --policy-name eks-role-policy
aws iam delete-role --role-name eks-cluster-role
KEY_FOR_USER=$(aws iam list-access-keys --user-name eks-admin | jq -r '.AccessKeyMetadata[0].AccessKeyId')
aws iam delete-access-key --access-key-id $KEY_FOR_USER --user-name eks-admin
aws iam delete-user --user-name eks-admin

aws ec2 delete-security-group --group-id $SG_ID
aws ec2 delete-subnet --subnet-id $SUBNET_1
aws ec2 delete-subnet --subnet-id $SUBNET_2
aws ec2 delete-route  \
  --route-table-id $ROUTE_TABLE_ID \
  --destination-cidr-block 0.0.0.0/0
aws ec2 delete-route-table --route-table-id $ROUTE_TABLE_ID 
aws ec2 detach-internet-gateway \
  --internet-gateway-id $IGW_ID \
  --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID
aws ec2 delete-vpc --vpc-id $VPC_ID
```

## Conclusion

Long tutorial, but we've covered a lot of ground.  The only bits of this reflecting a developer's daily work experience is collaborating in Loft and the `devspace dev`/`devspace deploy` commands.  The rest of the commands are for setting up the environment and cleaning up after ourselves.

In this tutorial, we learned how to use Loft with DevSpace to develop and test Kubernetes applications locally. We also learned how to use DevSpace to automate the deployment of our application to a Kubernetes cluster, and how to use DevSpace's profile feature to switch between different versions of our application.  However, the latter isn't really how we would use DevSpace for staging and production environments.  The next tutorial will cover how to bring everything we've discussed in this tutorial together to create a full CI/CD pipeline for our application using GitHub Actions and Argo CD.