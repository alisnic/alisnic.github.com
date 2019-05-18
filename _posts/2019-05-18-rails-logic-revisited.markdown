---
layout: post
title: On logic in a Rails app, revisited 6 years later
category: posts
---

6 years ago, I wrote one of my most popular blog posts - [On logic in a Rails app][1], in which I debate one of [@dhh][2]'s
critiques of [extracting code from controller][3]:

> So all you've done here is extract the work of the controller action into a separate object. This adds needless indirection, makes the code base harder to follow, and is completely unnecessary.

*NOTE: the original article changed its permalink, which wiped out
the original disqus comments including [@dhh][2]'s comment
in that post*

To summarize my article, my argument was that Rails is just an UI layer, and business
logic should be put in domain objects instead of keeping somewhere in Model, View or Controller.

To give you some context, it was a time when I was starting to grow as a Rails/Ruby
developer. I was reading a lot of blog posts on the topics and (as any young
developer with a lot of self esteem) I started to have
very strong feelings as to how Rails code should be written.

Fast forward 7 years, and a lot of things happened in my carreer as software developer:
- My experience got more rich. I launched projects, working with a lot of different
constraints and people
- My day to day work changed. I climbed the company ladder, eventually ending up in the
CTO position. The types of decisions that I have to make changed.
- I got more emotionally mature, which thankfully prevented me from writing more
controversial blog posts

So what do I think about that post, and the comment from [@dhh][2] now?
Where should logic of your application be put? model? controller? service objects?

It's about tradeoffs and personal preferences. If you found a way to keep all the logic in controllers, while:
- your team iterates fast and does not make silly mistakes
- code is easily digestible
- new features can be added without pain

Then good for you! It doesn't matter what me or [@dhh][2] or any other person says. If not,
well maybe putting logic in domain objects will work for your team, maybe not. Just explore and
find what suits you best.

Each team operates in an unique set of technical and non-technical circumastances, which are almost impossible to communicate
via internet during technical arguments. That's due language limitations, lack of proper
writing skills for all parties, silly code samples (most code is under NDA, so people use animals that inherit each other
to teach OO)

All the factors mentioned above mean only one thing - it's very hard to maintain a productive
discussion on the internet about the "right" approach of building software, because that may
be very similar to discussing religion. There's so much hidden context, that you might as
well just respect the other's decisions and focus on their personal insight instead.

[1]: /posts/rails-logic/
[2]: https://mobile.twitter.com/dhh
[3]: https://grantammons.me/2012/12/22/where-the-logic-hides/
