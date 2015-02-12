---
layout: post
title:  "What I'm using!"
date:   2013-06-16 10:04:09
categories: blog design
---

I wanted to list the tools I'm using to make this blog and the reasons I chose
them. First and foremost I do most of my work in ruby on rails and so it would
seem like an obvious first choice to use for a blog. The problem with rails is
its a bit of a behemoth. Using rails to write a blog feels like building ikea
furniture. Each piece is well thought out and there are clear instructions for
putting it together but it is still up to you to build the bloody thing!

So I thought maybe wordpress is the blogging engine for me. I've set up a few
wordpress sites but they always feel... clunky. The issue might be I work with
the code everyday so if I want to add something to a site I want to be able to
just do it rather then go searching through the settings or installing another
plugin. I have nothing against wordpress and consider it a blessing to the web
because it allows people who don't write code to come up with something decent
with very little effort. A great example is my wife who knows wordpress settings
better then I do. She volenteers at [any rat rescure][anyratrescue] here in
Arizona and you can find her blog on all things ratty [here][jessblog].

It just so happens I was looking for a solution to a coding problem when I
stumbled across [this blog post][pagesmove], I think (it was a while ago so not
sure if I'm giving credit to the right post). Anyway I started looking into
github pages. Bascially you need to make a repo called YourUserName.github.io
and it'll load up a static site for you. Having nothing to loose I made the repo
and hit the generate page button.

So in maybe 5 minutes messing around on github I had a site up. So I pulled the
repo to see the source code and going throgh the [help guides][pageshelp] on
github. These constantly talk about jekyll and how great it is so I gem install
jekyll, destroyed everything in the repo and generated a new jekyll site.

[Jekyll][jekyll] is awesome. It's a static site generator that allows me to
setup the site once and then write new blogposts in sublime. Once I'm done
writing getting it up is as simple as git commit and git push. No fucking around
in settings or with databases, just markdown files.

Anyway I grabbed twitter bootstrap to get started with some styles and threw
disqus in there so you can tell me how wrong I am in all my choices. The whole
affair was over in about 45 minutes which was mostly spent learning rather then
frustratingly searching threw config settings. A much more pleasent experience
then whenever I've dived into wordpress.

[anyratrescue]: http://www.anyratrescue.org/
[jessblog]: http://rattributes.wordpress.com/
[pagesmove]: http://ocramius.github.io/blog/moving-my-blog-to-jekyll/
[pageshelp]: https://help.github.com/categories/20/articles
[jekyll]: http://jekyllrb.com/
