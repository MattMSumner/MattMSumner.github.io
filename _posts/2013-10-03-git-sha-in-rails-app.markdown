---
layout: post
title:  "Git SHA in Rails Application"
date:   2013-10-03 09:12:32
categories: rails
---

Sometimes you need to know what version you have up on your production app. I theory we should all be tagging our commits and only pushing ones to production we've tagged. I practise I'm bad! So I thought "no problem, I'll show the git SHA on the apps settings page!". So I tried:

{% highlight ruby %}
   GIT_SHA = `git rev-parse HEAD`.chomp
{% endhighlight %}

In config/initializers/git_sha.rb. Unfortunately, this doesn't work with a standard cap deploy as the current directory isn't set up as a git repo. Thankfully, they've thought ahead of me and capistrano automatically makes a REVISION file containing the sha and pops it in the applications root directory. This means I can change my initializer to:

{% highlight ruby %}
   GIT_SHA = File.read('REVISION').chomp
{% endhighlight %}

And everything works!
