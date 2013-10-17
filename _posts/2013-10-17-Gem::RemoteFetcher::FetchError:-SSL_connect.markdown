---
layout: post
title:  "Gem::RemoteFetcher::FetchError: SSL_connect"
date:   2013-10-17 13:45:49
categories: general
---

I tried making a new rails app today and got a wierd error after bundle:

{% highlight bash %}
Gem::RemoteFetcher::FetchError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://s3.amazonaws.com/production.s3.rubygems.org/gems/multi_json-1.8.2.gem)
An error occurred while installing multi_json (1.8.2), and Bundler cannot continue.
Make sure that `gem install multi_json -v '1.8.2'` succeeds before bundling.
{% endhighlight %}

The solution was to run:

{% highlight bash %}
rvm get stable
rvm osx-ssl-certs update all
{% endhighlight %}

And then bundle runs fine.

I'm duplicating the answer here. [I got it from here originally][answer].

[answer]: http://railsapps.github.io/openssl-certificate-verify-failed.html
