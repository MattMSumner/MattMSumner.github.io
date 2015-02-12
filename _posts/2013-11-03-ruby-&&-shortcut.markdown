---
layout: post
title:  "Ruby's && operator for params"
date:   2013-11-03 09:47:21
categories: rails ruby &&
---

I was reading through some ruby code and I found a cute little shortcut for
avoiding lookups on those frustrating nil objects. The code is question was:

{% highlight ruby %}
def search_value
  params[:search] && params[:search][:value]
end
{% endhighlight %}

This comes from the excellent [geocoding on rails][gor]. What's so cool about
this is it will return the first value if it is falsey otherwise it will return
whatever the second value is. It can do this through the magic of short circuit
evaluation.

What this means is that ruby is smart enough to know that if the first value is
falsey then it doesn't even need to check what the second value is. If the first
value is false then && (and) will always evaluate to falsey. If the first value
is truthy then whatever the second value is (truthy or falsey) that will also be
what the && should evaluate to. So it simply returns that second value.

I love this elegance. I just wished I found it sooner -_-


[gor]: https://learn.thoughtbot.com/products/22-geocoding-on-rails
