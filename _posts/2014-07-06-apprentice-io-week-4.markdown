---
layout: post
title:  "Apprentice.IO - Week 4"
date:   2014-07-06 18:35:11
categories: thoughtbot apprenticeship month-1
---

I've completed the final week of my first month as an apprentice at thoughtbot.
It's been incredible so far and I'm excited to get started with my next mentor
[Blake](http://blakewilliams.me/).

This week Dan and I were working on optimizing the JavaScript front end of a
client application written in backbone. It had this horrifying behaviour where
in the android version of mobile chrome, the app would grind to a halt. From
page load to the app becoming intractable was a whopping 24 seconds! We ended up
using chrome's remote debugging capabilities to diagnose the issue and
discovered after some exploration that the issue was coming from excessive
repaints of the screen, not, as we had initially thought, from long running
JavaScript processes. A repaint is when the browser marks elements as having a
visual change and needs to tear down what has been rendered and replace it with
the new visuals. One of the first things we tried to narrow down the issue was
remove all css. The app immediately sped up so we started looking for what could
be causing the excessive repaints. Turns out, box-shadow is notorious for
causing these constant, expensive repaints and we happened to have the main
element of the app have an inset box-shadow. Now this wouldn't usually be an
issue, however, whenever anything inside this main element was changed, the
browser would require a repaint. As the whole application was essentially inside
this element, the browser seemed to be queuing up repaints before the previous
repaint had finished leading to our 24 second lockout time. Removing the box
shadow was like night and day, bringing the time before the app was interactive
down to 3 seconds. Far more reasonable.

I need to probably spend some time reading up on css optimizations as removing
one line of code here lend to a dramatic effect on this application. The
surprising thing was that the behaviour was only exhibiting on mobile chrome for
android. I presume other versions of chrome (and indeed other browsers) handle
multiple repaints more intelligently. More research is required!

Aside from working on client work, I've also been pairing with another
apprentice on switchboard, the previously mentioned internal app for managing
projects. This has been awesome as I'm managing to solidify a lot of my
understanding of ember through teaching. It's also been great for experiencing
working on a project with members all over the country. It's also taught me to
get work into code review sooner rather then later as the support from other
developers is phenomenal. The smaller the commit the better. This really isn't
practical when working on larger features though, so making sure I've discussed
the feature throughly with the whole team before writing a single line of code
leads to far fewer suggestions when I submit a pull request.
