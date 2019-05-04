---
layout: post
title: Disabling Caching in Tests
date:   2016-05-06 10:24:13
categories: testing rails web
excerpt: You may not get the results you expected.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/fragment-caching-in-tests)*

I ran into a problem on a Rails app recently where tests were leaking state via
caching. Typically you may see the following configuration in
`config/environments/test.rb`:

```rb
Rails.application.configure do
  config.action_controller.perform_caching = false
end
```

However, this only disables controller caching. If you application has fragment
caching, custom caching behavior, or you are using a gem with caching, then the
above may not stop caching behavior between test runs. For me, this came up as a
test that would fail, but only if a previous test that hit the same page was run
immediately before it. And even then it only failed sometimes.

It turns out that the page being tested was using fragment caching. The cache
key depended on the `updated_at` and `id` of the record being shown. If the
previous test ran in less then a second, the following test would hit the same
page and see the cached result.

Instead, you should have the following line in `config/environments/test.rb`:

```rb
Rails.application.configure do
  config.cache_store = :null_store
end
```

This will have caching on but any writes to the cache are immediately
discarded.

There is a trade off here of time vs. coverage. Using `:null_store` saves us
time at the expense of not testing reads from the cache. This means that we need
to be careful of making sure that cache keys are truly unique.
