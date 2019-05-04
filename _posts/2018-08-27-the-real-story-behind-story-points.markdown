---
layout: post
title: "The Real Story Behind Story Points"
date: 2018-08-27 16:24:13
categories: process business
excerpt: I estimate a 13.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/the-real-story-behind-story-points)*

We've previously written about estimates when it comes to [Why Fixed Bids Are
Bad]. Today, we'd like to talk about a tool used for estimating that we've had
little success with.

[why fixed bids are bad]: https://thoughtbot.com/blog/why-fixed-bids-are-bad-for-clients-too#estimates

Story points have become [bog standard] in our industry to the extent that some
teams forget to consider alternatives. The purpose of this post is to discuss the
problems story pointing fails to solve and discuss an alternative approach.

[bog standard]: http://www.bbc.co.uk/worldservice/learningenglish/radio/specials/1728_uptodate/page25.shtml

## Predicting When a Large Body of Work Will Ship

The idea is compelling. Break up the larger body of work into the smallest
achievable parts, estimate how long each part will take and the total of those
estimates reveals when the work will ship.

Science!

This only works with the following assumptions:

- You know how to break up the work.
- You know how long the pieces of work will take.
- The product priorities will not change while delivering this work.
- It's more important to know when a feature will ship vs concentrating on
  shipping the most valuable version of the feature.

Given all the above assumptions, it's unlikely you can predict a larger body of
work with story pointing alone.

An alternative is to start predicting the larger body of work in its entirety
and checking in every week. We've personally found success with this approach.
It forces a re-evaluation of the work each week and allows the team to have
meaningful conversations around [deadlines], scope and specific technical
challenges. These predictions are subject to change and teams are not pressured
to give a prediction for work that isn't predictable e.g. using a new
technology.

[deadlines]: https://thoughtbot.com/blog/deadlines

_NOTE:_ We know in theory story points are an iterative process. In practice,
every project we have seen with story points has pointed once and not adjusted
later. There have even been times when we've acknowledged that work was far more
complex than initially anticipated, but the story kept the original estimate
anyway. These skills are difficult to master, and the main problem is that
simply adding story points gives the impression that estimating is easy!

## Tracking Team Velocity and Accountability

Again, on paper, using story points appears to make sense. We would like to have
a reliable indicator of the team's productivity. Measuring productivity allows
us to react to impediments before everything grinds to a halt or celebrate
increases in productivity and iterate on keeping the team productive.

Unfortunately, this approach suffers from the same fundamental failings as above,
in addition to:

- You accurately predicted the "point" cost of the work done.
- You update point cost when the prediction turns out to be inaccurate.
- You never spread work between sprints. (e.g. a ticket's points are allocated
  to one sprint but the ticket spanned two or three sprints.)

Story points are often an unreliable data source as the points were collected at
a time when the team knows the least about the work. A more accurate picture of
the team's velocity can be derived from daily stand-ups and sprint retros. Each
day is an opportunity to know more about the task at hand and provide a more
informed estimate regarding how much time the task requires.

In regards to accountability, we should strive to increase communication to
ensure no one is feeling unproductive. Reducing team members to the number of
points they can achieve in a sprint can be demoralizing. It's also not a full
representation of the individual's contributions to the team.

It's worth mentioning that we have seen story points work well when velocity is
being used solely by the team of developers and designers to predict how much
work they can complete in an iteration. If anyone outside of the team is
tracking these numbers it tends to lead to gamification of the system.

## Remaining Transparent

Referencing story points when communicating about a feature adds another
dimension to the conversation. This introduces what others may see as a barrier
to understanding the development team's process.

An alternative is to use a metric everyone understands. If an interested party
wants to discuss a feature, discuss the work in terms everyone understands, like
time. If the interested party would like additional visibility into the process,
invite them to stand-ups, retros, and sprint demos.

## Prioritizing Features

Discussing the effort required to complete a particular story and assigning
story points does have the benefit of highlighting which tickets are believed to
be "low effort". Combining this information with the knowledge of which work is
most valuable helps the team identify the work that will provide the most "bang
for their buck".

Alternatively, assigning story points is not a requirement to effectively
prioritize features. Try prioritizing stories without story points. This may
lead to shorter meetings while continuing to surface the work that delivers the
most value to users.

## Evaluating the Process

All process has a cost. Are you getting enough benefit for what you're paying?
If you're getting your whole team together into a room for 30 minutes to an hour
each week to put arbitrary numbers on tickets that almost never match the actual
time it takes to complete those tasks, maybe consider killing that meeting. Too
many times we have seen companies take on the process of story pointing without
gaining anything except lost time and mismatched expectations.

In our opinion, story pointing also pushes against recognizing work that no
longer adds value. Once a story has been assigned points, the sunk cost fallacy
makes it difficult for a team to ask "could we avoid building this"? The story
is also rarely revisited, regardless of how the system has changed.

If you find yourself in the situation where story pointing isn't working for
your team, that's okay. Have the courage to try something else. The goal is to
ship working software, not to collect the most points.
