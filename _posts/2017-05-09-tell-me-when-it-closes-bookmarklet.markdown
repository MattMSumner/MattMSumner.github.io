---
layout: post
title: "Tell Me When It Closes: Bookmarklet"
date: 2017-05-09 10:24:13
categories: news git tools productivity
excerpt: >
  Tired of copying and pasting GitHub URLs for Tell Me When It Closes? We've got
  you covered.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/tell-me-when-it-closes-bookmarklet)*

About a month ago we [launched Tell Me When It Closes], a service for getting
less noise from GitHub issues/pull requests and more signal. At thoughtbot we
believe we can always be improving, from the big issues down to the smallest
things like having to copy and paste a URL.

[launched Tell Me When It Closes]: https://thoughtbot.com/blog/announcing-tell-me-when-it-closes

## Introducing Our Bookmarklet

If a picture is worth ten thousand words then a gif may be invaluable:

![using-tell-me-when-it-closes-bookmarklet](https://images.thoughtbot.com/blog-vellum-image-uploads/3QE0sUDQo2B3nfQmi7YV_tmwic-bookmarklet.gif)

Checkout [these instructions] for installation.

[these instructions]: https://tellmewhenitcloses.com/pages/subscribing

## The Impetus

The following tweet:

<blockquote class="twitter-tweet" lang="en">
  <p>
    Like this a lot, but would you please allow pre-populating the URL field
    with a query param? This way I can fill it in with a bookmarklet.
  </p>&mdash; Zhiming Wang (@zmwangx)
  <a href="https://twitter.com/zmwangx/status/846956632881987586">
    March 29, 2017
  </a>
</blockquote>

## Iteration One

<blockquote class="twitter-tweet" lang="en">
  <p>
    Tell Me When It Closes now supports `url` as a query param to pre-fill the
    form. Wire it up with a bookmarklet! https://gist.github.com/MattMSumner/6591fb6b008fd7c0d9be3077097692bc
  </p>&mdash; thoughtbot (@thoughtbot)
  <a href="https://twitter.com/thoughtbot/status/850417562185854976">
    April 7, 2017
  </a>
</blockquote>

Neat, but I think we can make setting up this bookmarklet even easier.

## Iteration Two

We've [added a page] to the site describing how you can setup with a bookmarklet
on desktop and mobile. Let talk briefly about the two solutions.

[added a page]: https://tellmewhenitcloses.com/pages/subscribing

### Desktop

On desktop, we provide a link which you can drag straight to your bookmarks, and
that's it!

### Mobile

This ended up being a little tricker. We've added a [button to copy] the
bookmark link into you clipboard and then you'll need to create a bookmark and
edit the URL to have the copied text.

[button to copy]: https://clipboardjs.com/
