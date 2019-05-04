---
layout: post
title: "Why Does Style Matter?"
date:   2015-02-12 10:24:13
categories: web productivity hound
excerpt: Working in a well-organized code base is like cooking in a clean kitchen.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/why-does-style-matter)*

A few days ago, our [Hound](https://houndci.com/) team asked a question that
gave me pause: "Why does style matter? If we were to stop
using style guides on all of our projects, what would your argument be for
bringing them back?"

This reminded me of a recent client engagement. I found myself in a delicate
situation while integrating with an existing developer team: They were not
accustomed to following a style guide while working on a project. When we first
pitched the benefits of introducing a style guide to the code base it appeared
the whole team was on board.

> Having a style guide makes a code base feel neat. It gives the impression of
> a team working well together towards a common goal, rather than individual
> developers working frantically and without purpose.
>
> Working in a well-organized code base is like cooking in a clean kitchen. If
> things feel messy, it's easy not to treat it with respect. If the formatting
> feels sloppy, it will tempt you to also be sloppy when it comes to
> readability, dependency management, and testing.
>
> Inconsistent formatting is the first sign of a code base that nobody really
> cares about.
>
> <cite>Joe Ferris</cite>

However, in practice, the team was not on board with a style guide. During code
review developers defended each style violation as a special case. This tension
was magnified by the fact that pull requests and code reviews were only
introduced shortly before the style guide.

We went through the exhausting process of searching through pull requests for
style violations and then having to explain the reasoning behind each guideline.
This exercise made me feel awful and wasted time we could have spent developing
features.

> I hate doing work that can be automated. There are better things I can be
> doing than looking for double/single quotes or hex capitalization. As to why
> these things are important, it's because I also hate to open a file with 3
> different CSS color styles and spend 5 minutes deciding which one I should
> use and why.
>
> <cite>Reda Lemeden</cite>

After a few pull requests I'd had enough and switched on
Hound. When the team learned that a robot dog was going
to start reviewing their code, their attitude completely changed. Instead of
arguing about the style choice everyone simply made the changes. After all, it
would be silly to be offended by a bot!

<blockquote>
  <p>You know, having Hound on OSS is really nice. I can point to it for style
  problems without feeling like a jerk.</p>
  <p><cite>Jon Yurek</cite></p>
</blockquote>

> Manually reviewing code for style sucks. It's time-intensive and socially
> awkward.
>
> Let machines do low-level things for us without emotion. "Hound said it, not
> me."
>
> <cite>Dan Croak</cite>

In the end, the hard problem wasn't convincing a team to implement a style guide
but navigating the social complexities of enforcing it. So: Does style matter?
Should we be using Hound?

> Style guides and Hound greatly reduce the time I have to spend searching for
> style violations in PRs. That way I can focus on actual code smells,
> opportunities to ask question, or chances to share important information.
>
> <cite>Britt Ballard</cite>

Try [Hound](https://houndci.com/) today!
