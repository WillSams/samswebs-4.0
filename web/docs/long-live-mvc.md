# Long Live MVC

![MVC Header](./assets/images/long-live-mvc-header.png)

And long live [MVVM, MVP,](https://dev.to/ayushsoni1010/mvc-mvp-mvvm-mvvm-c-and-viper-architecture-patterns-1l3g), [MTV](https://vegibit.com/what-is-djangos-mtv-model-template-view-architecture/), and whatever other M* pattern for your frontends, mother @#$%ers!

I wanted to speak about this for a long time but arrogance I saw in a YouTube video and downright wrongness (go with it) I read in another blog post got me a tad triggered.  Seemingly, every 6 months there is a new server-side web framework pushed.  Seemingly, every 6 months, social media influencers try to convince you everything you did before was wrong.

When I'm writing code as a hobby, and I want to write something for a frontend, [Angular](https://angular.io/) and [React](https://react.dev/) make me ill with the thought of using them.  For work (i.e., you give me money), no problem, boss! But for hobby (yes, I'm one of those weirdos that write code as a hobby, it gives me comfort) then no, then I'd rather eat shit than use React/Angular in my free time.

I'll spare [Vue](https://vuejs.org/) for now.  Haven't used it, but looks like something I'll regret outside of work so with all of that out of the way...

## Architecture Patterns Will Never Die

*/start rant*

I find these frameworks trying to reinvent the wheel when all I need is a wheel. "Separation of concerns" isn't "bullshit" as I've heard one YouTuber phrase it. Yikes.  Most of these frameworks make sense for public-facing consumer or enterprise applications but for tools that aren't necessary for customer consumption, why are we using them?  Why are we dumping so much JavaScript on the client-side or transpiling everything? There is a trade-off between simplicity and complexity; frameworks like React and Angular just hide much of that complexity and call it "simplicity", all the while adding a learning curve and barrier to entry.  

For my doe-eyed engineers who probably got their start after the year of 2015: let's consider that for your next job, you'll need to learn a new framework (if being allowed to learn a new framework on the job is a thing in 2024). If you're one of those engineers who scoffs at jQuery, I'm going out on a limb and say that you've probably never built a ton of websites with plain HTML, CSS, and JavaScript.  So, you might haven't the strongest grasp of what's going on under the hood of your favorite JavaScript frontend framework. Congratulations, you are now a master of none.  

Still, not a barrier to entry for me to hire you.  Furthermore, if I need a Python/Java/C# developer to pick up something that resembles my [Hotel Reservation example's frontend](http://github.com/WillSams/mvc-expressjs-fastapi-hotel-reservation) and ramp up very quickly than would if it was written using a JavaScript framework they've never used.

For non-public facing tools and examples, I appreciate a more simple, server-centric, and straightforward approach with maintainable code and easier debugging. If performance is a concern (something overblown in the 2020s...jeez, I really need to write a separate blog post on http://samswebs.com), then maybe you have business logic in your controller, i.e., that isn't what the controller is for.  I can find many develoers

If you feature-rich, SEO-friendly frameworks like [Next.js](https://nextjs.org/) or [Nuxt](https://nuxt.com/), or [Angular](https://angular.io/) for enterprise applications (and need to find highly skilled developers who know those frameworks), that is perfectly fine. I would still argue for enterprise in-house applications, rolling your own Model-View-ViewModel application in 2024, is the way to go.  We can take lessons we've learned from those other framework and move on.  I think great new tools like [Alpine.JS](https://alpinejs.dev/) and [HTMX](https://htmx.org/) are statements that we've gone too far, time to build based on lessons learned.

## In-Defense of MVC

Arguments against the MVC pattern can be debunked if you put a little thought into it:

- ```The MVC Pattern is hard to test.```  Really?  What?!
- ```Code/implementation is generally difficult to read?```  Again, whoever thinks this....you were doing it wrong.
- ```Deployment is a pain```  I have a bridge in "Nowhere" to sell you.  Containerization helped to solve this years ago.  Using AWS as an example, I could deploy the frontend independent of my backend [ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html), [EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html), or even as a [serverless function](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html).

The proliferation of various frontend frameworks and libraries, often backed by large corporations, has led to a diverse but fragmented landscape. I'm not sure if this is sustainable if employee turnover ratios at most employers don't improve.  The final nail in the coffin?  Enterprise applications built 10 years ago on Rails, Django, Laravel, CakePHP, ASP.NET MVC, and Spring MVC will probably outlive your Angular, Next.js, or Nuxt application built yesterday.  Mic drop, let's re-visit this in 10 years.  

So, for a minimalist approach to my Hotel Reservation example...haha..yeah. MVC will never die. Suckas!!!!  Please, feel free to hit me up on LinkedIn to tell me what you think.

Note: The cost of scaling servers, lack of needing to worry about mobile UIs (but CSS frameworks help), and the schizophrenic nature of what a "full-stack" engineer is nowadays can be also part of the discussion, but that's a rant for a different day.
Note2: And Vue...2 versus 3.  Ha ha! 

