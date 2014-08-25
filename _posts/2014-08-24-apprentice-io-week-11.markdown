---
layout: post
title:  "Apprentice.IO - Week 11"
date:   2014-08-24 10:44:01
categories: thoughtbot apprenticeship month-3
---

This week was the thoughtbot summer summit, a yearly get together of all the
offices to get to know each other a little better. This year the summit was at
the San Francisco office. Those of us in Boston flew out Thursday and most,
including me, are flying back Sunday.

The emphasis of the summit was really getting to know each other better.
Mornings usually had some sort of outing/activity, afternoons were filled with
lightning talks and evenings going out for dinner then heading back to the
office for drinks. Chad gave an illuminating speech about the direction of the
company, emphasizing that the main goal has been, and always will be, for
thoughtbot to be a place we want to work. We want to do the things we love with
people we enjoy spending our time with.

Before the summit I did some more work on our breakable toy and ran into an
issue with Ember and SVG elements. We're building the game board using an SVG
and I'm generating elements using ember components and views. I'd put a fair bit
of time into the game board, and when I noticed that bindings were simply not
working I started googling. Turns out, this is a known issue, and the latest
versions of ember have [exposed some of the metamorph
internals](https://github.com/emberjs/ember.js/commit/641f2804e81ea5b9a32d2fd450d2e9e58cea5954)
so that custom elements can be added when special support is required. I found
[this patch](https://github.com/blesh/ember-handlebars-svg/blob/master/ember-handlebars-svg.js)
for handling SVG tags in ember and converted it into an initializer. I also
translated to coffeescript:

{% highlight coffee-script linenos %}
# app/initializers/svg.coffee
`import Ember from "ember"`

SvgInitializer =
  name: "svg"
  initialize: ->
    Ember._metamorphSvgTags = [
      "altGlyph"
      "altGlyphDef"
      "altGlyphItem"
      "animate"
      etc...
    ]

    Ember._metamorphSvgTags.forEach (tag) ->
      Ember._metamorphWrapMap[tag] = [1, "<svg>", "</svg>"]

    applyClasses = (elem, value, fn) ->
      classes = value.split(" ")
      Ember.$(elem).each ->
        item = this
        fn(item, cls) for cls in classes when cls

    Ember.$.fn.addClass = (value) ->
      applyClasses this, value, (elem, cls) ->
        elem.classList.add cls
      this

    Ember.$.fn.removeClass = (value) ->
      applyClasses this, value, (elem, cls) ->
        elem.classList.remove cls
      this

`export default SvgInitializer`
{% endhighlight %}

I'm not sure if an initializer is the correct place for this code, so let me
know if you think it isn't.
