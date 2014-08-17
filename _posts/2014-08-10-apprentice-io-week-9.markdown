---
layout: post
title:  "Apprentice.IO - Week 9"
date:   2014-08-10 21:14:55
categories: thoughtbot apprenticeship month-3
---

This final month is an interesting one as Joël is working with the same client
as Blake but on a different project. The project I worked on with Blake is true
Legacy code, a Rails 2 application with plenty of plenty of time invested, while
this project is 6 months old and developing fast. It's almost a green fields
application but there are still some bad practises sneaking in that we're trying
to nip in the bud.

The first thing I tackled was the test suit. It fails randomly I believe due to
data leaking between tests but it's tough to tell as the suit is full of
[mystery guests](http://robots.thoughtbot.com/mystery-guest). As the project is
still young, it only took me 3 days to rewrite all the specs to use factory girl
in the specs themselves (not hidden away in another file) and setup [database
cleaner](https://github.com/DatabaseCleaner/database_cleaner). This brought the
test suit from around 1 minute 35 seconds down to 1 minute 15 seconds, which is
a nice bonus to the refactor.

I also had a brief encounter with adobe illustrator. I built up a svg of the
board for the diplomacy application. It should be fairly straight forward to
have ember create the svg nodes as components giving us an interactive board. I
had initially thought we'd use the javascript library
[ammaps](http://www.amcharts.com/javascript-maps/) but I found myself basically
disabling all the features. It'll be interesting to see how easy it is
interacting with the svg directly as this is something I have no experience
with.

I have a meeting scheduled next week with Chad and Josh concerning the
apprenticeship. This is also when they would put forward an offer of employment
if they feel I'm a good fit for the company. I'm nervous yet confident so
hopefully I'll have good news in next weeks post.
