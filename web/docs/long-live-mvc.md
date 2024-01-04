# Long Live MVC

![MVC Header](./assets/images/long-live-mvc-header.png)

And long live [MVVM, MVP,](https://dev.to/ayushsoni1010/mvc-mvp-mvvm-mvvm-c-and-viper-architecture-patterns-1l3g), [MTV](https://vegibit.com/what-is-djangos-mtv-model-template-view-architecture/), and whatever other M* pattern for your frontends, mother @#$%ers!

- **Editor's Note 1:** I'm going to need you to conviently ignore the cost of scaling servers and the need to not worry about mobile UIs as you read this mind-numbing blog post.  Ok?  Thanks.
- **Editor's Note 2:** Also, the schizophrenic nature of what a "full-stack" engineer is nowadays could also be also part of the discussion, but that's a rant (i.e., aneurysm) for a different day.
- **Editor's Note 3:** Agile is dead.  Or what it should be is dead.  The frameworks I'm about to mention are born out of Silicon Valley's abuse of "agile".  These frameworks focus providing the speed and flexibility (allegedly), but the ever-growing complexity and the need to specialize in frameworks poses a serious threat to software quality.  Software development is not a car assembly line and that's what this busted state of agile is probably better for.
- **Editor's Note 4:** Although Angular is MVVM, but's complex, verbose, and heavy.  Three things I'm against in the context of this article. See, *Editor's Note #3*.
- **Editor's Note 5:** Maybe I'm starting to hate JavaScript frameworks.  I need therapy.

I've been wanting to speak about this for a long time, but the arrogance I saw in a YouTube video and downright wrongness (go with it) I read in another blog post got me a tad triggered. Seemingly, every six months there is a new server-side web framework pushed. Seemingly, every six months, social media influencers try to convince you everything you did before was wrong.

When I'm writing code as a hobby, and I want to write something for a frontend, [Angular](https://angular.io/) and [React](https://react.dev/) make me ill with the thought of using them.  For work (i.e., you give me money), no problem, boss! But for hobby (yes, I'm one of those weirdos that write code as a hobby, it gives me comfort) then no, then I'd rather eat shit than use React/Angular in my free time.

I'll spare [Vue](https://vuejs.org/) for now.  Haven't used it, but looks like something I'll regret outside of work so with all of that out of the way...

## Architecture Patterns Will Never Die

*/start rant*

I find these frameworks trying to reinvent the wheel when all I need is a wheel. "Separation of concerns" isn't "bullshit" as I've heard one YouTuber phrase it. Yikes. Most of these frameworks make sense for public-facing consumer or enterprise applications, but for tools that aren't necessary for customer consumption, why are we using them? Why are we dumping so much JavaScript on the client-side or transpiling everything? There is a trade-off between simplicity and complexity; frameworks like Angular just hide much of that complexity and call it "simplicity," all the while adding a learning curve and barrier to entry.  Alternatively, if I need a primarily Python/Java/C# developer to pick up something that resembles my [Hotel Reservation example's frontend](http://github.com/WillSams/mvc-expressjs-fastapi-hotel-reservation) and ramp up very quickly than if it was written using a JavaScript framework they've never used.

When not building an enterprise application, I will always appreciate a more simple, server-centric, and straightforward approach with maintainable code and easier debugging.
If you need a feature-rich, "full-stack", SEO-friendly framework like [Next.js](https://nextjs.org/) or [Nuxt](https://nuxt.com/) (or to an extent, Angular)) for enterprise applications, you'll find yourself in the continual courtship of find highly skilled developers who know those frameworks.  That is perfectly fine, and these developers deserved to be paid handsomely for their services. I would still argue, for private enterprise applications, using established MVC-like frameworks like Django or rolling your own Model-View-ViewModel application is the way to go in 2024.  We can take lessons we've learned from those other framework and begin weaning ourselves off of them.  I think great new tools like [Alpine.JS](https://alpinejs.dev/) and [HTMX](https://htmx.org/) are statements that "we've gone too far, reset, and build based on lessons learned".

## In-Defense of MVC

Arguments against the MVC pattern can be debunked if you put a little thought into it:

- ```The MVC Pattern is hard to test.```  Really?  What?!
- ```Code/implementation is generally difficult to read?```  Again, whoever thinks this....you were doing it wrong.
- ```Deployment is a pain```  I have a bridge in "Nowhere" to sell you.  Containerization helped to solve this years ago.  Using AWS as an example, I could deploy the frontend independent of my backend [ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/create-container-image.html), [EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html), or even as a [serverless function](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html).
- ```Performance!``` If performance is a concern, you are doing it wrong.  Business logic in your controllers you say?

The proliferation of various frontend frameworks and libraries, often backed by large corporations, has led to a diverse but fragmented landscape.  Going back to the continous search for those highly skilled engineers who know those frameworks:  I'm not sure if this paradigm is sustainable because these same developers will move on to the next framework that solves the next problem they feel they are dying to solve (that probably didn't really need solving).  The final nail in the coffin?  Even when using these frameworks for enterprise applications, the ones built 10 years ago on [Rails](https://rubyonrails.org/), [Django](https://www.djangoproject.com/), [Laravel](https://laravel.com/), [CakePHP](https://cakephp.org/), [ASP.NET MVC (although, Blazor..lol)](https://dotnet.microsoft.com/en-us/apps/aspnet/mvc), and [Spring MVC](https://docs.spring.io/spring-framework/reference/web.html) will probably outlive your Angular, React/Next.js, or Vue/Nuxt application built yesterday.  Mic drop; hell of an assertion but to back that up:

- Employee turnover/knowledge transfer on those older frameworks have been going on for more that 15+ to 20+ years.  They are mature and stable.
- The breaking changes in these JavaScript frameworks seems to happen a lot.  And it's sooooooooooooooooooooo annoying.  God, this makes me want to commit hari kari every damn time.
- MVC still proves effective and maintainable today, despite the random 30 year old on social media says. Note:  I'm near 50 so I'm nearing "old man screams at cloud" disposition.  I'm fine with this. 

So, for a minimalist approach to my Hotel Reservation example...haha..yeah. MVC will never die. Suckas!!!!  Please, feel free to hit me up on LinkedIn to tell me what you think.

## Summary

I needed to write a defense of the MVC pattern and critique of the perpetual influx of frontend frameworks, some of them are now touting themselves as "full-stack" solutions.  it's a plea for thoughtful consideration when adopting frameworks, particularly in scenarios where their complexity may outweigh the benefits.  

I'm not questioning anyone's depth of understanding and mastery of your favorite JavaScript framework.  I'm also not calling for a return to basics; never will I call for us returning to building websites with plain HTML, CSS, and JavaScript.  At the end of the day, I can't dismiss the value of newer frameworks but I think, as engineers, we really need to emphasize the enduring legacy of foundational principles.

Let's engage in a constructive dialogue about the evolution of web development, recognizing the merits of both established patterns and innovative frameworks. We need a balance.  Let's strive for balance in our development choices, not alternating between adopting the hottest "agile" framework straight out of Silicon Valley states every six months.
