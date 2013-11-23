---
layout: post
title:  "Ember JS"
date:   2013-08-17 14:08:13
categories: emberjs
---

I recently went on a three day course for ember put on by [Tilde]({% post_url 2013-07-12-training-at-tilde %}) and I thought I'd make a post talking about the whole thing. Just a forewarning; I have only been coding for a sort time and come from a rails background, this post will be from the point of view of a rails developer.

## Ember JS is a Framework

Just a quick note that taking up Ember is not for the faint of heart. There are a lot of concepts you need to get straight before becoming truly productive. This upfront investment will pay off, but it can put off new and old developers alike. One thing I can guarantee is that I know far more about javascript as a language then before I started. Another thing is that if you find yourself trying to do something ember doesn't really like then your making work for yourself. Ember will make you a better developer. Clearly defined boundaries are a good thing. Companies like apple don't produce such carefully designed products by tearing down boundaries, they do it by putting them up!

Also you shouldn't be looking into ember unless your planning something big. Like it says on the [front page][ember] ember is "A framework for creating **ambitious** web applications" and your only going to reap the true benefits of ember by being ambitious.

If none of the above has scared you away then read on!

## MVC

One of the most confusing things about learning ember is all my preconceptions. Ember is an MVC framework. Going into it I assumed life would be easy as Rails also is a MVC framework.

These are NOT the same D=

MVC stands for model-view-controller and is a design pattern based on the principle of [separation of concerns][soc]. This basically means that different objects should handle well defined responsibilities. MVC confused me as neither rails nor ember is exclusively made up of these three objects.

Also, ember and rails has different opinions on what each object should be responsible for. If you think about it, this makes sense as the two frameworks were built to handle very different types of application. Rails is a server-side framework while Ember is a client-side framework. Where the framework lives, at first glance, seems irrelevant but it fundamentally changes how and where objects are created and destroyed.

In a server-side framework such as rails your objects only exist long enough to create a response. When a request comes in from a client (i.e. a browser visits a page) rails checks the url to see which controller and action it should use, it then follows a stack to create the end result. Rails starts with the router which passes on to the controller. The controller will use the params hash (or not) to collect up all the information that the view is going to need. It then passes all this information to the view which generates the final html/json/etc to return to the client. All of these objects are then thrown away and new one's generated for the next request.

This stack structure makes data flow very obvious and you know that any objects created in the view can't be passed back to the controller. You do all your operations in the controller and then pass it to the view. This one way flow of data was something I was very familiar with but for some reason bothered me when I started learning ember.

Ember has it's own architecture. At the top you start with objects representing your data, models, which pass everything they know to a controller. The controller here acts a little more like a proxy object for the model, wrapping it to add any additional functionality specific to the current session (such as display status). This will then pass to the view layer to create views to be inserted into the dom. These will use templates to show the content. Most of the time you'll find yourself only working with templates and the views will be implicit.

This initial data flow may seem very similar to the rails approach, however, client side frameworks have an added complexity that is user interaction. In rails user interaction is handled by making a brand new request, in ember you can reuse all the stuff that's already there!

This user interaction acts upon the views. These should then turn the user interaction into a event so the framework can translate the interaction into some action. This event will then "bubble" back up the stack until it finds something that will handle it. For example you may click to show/hide some information. This would usually be snatched up by the controller which would then toggle a property and immediately update the view. Anything higher doesn't even need to know what has happened. Alternatively you may submit a form to change a model attribute. In this case the controller would also grab the event but would instead update the model data. The model would then update the controller and finally the view.

There is a final part of this stack I have yet to mention and arguably the most important, the route.

## Routes and the Router.

The ember router can almost be thought of as a state map for your application. You define in your router which routes url's match up to. Whenever your app visits a particular url the router will work out which route to load up and call any necessary hooks to get the application into the correct state. This includes fetching model data from the server, setting controller attributes and loading views into outlets. There is also this concept of nested routes which allows you to have access to data and functions that are common to the child routes.

Now there are some events that are not associated with a particular controller/model. If they are not handled by the controller then they'll bubble up to the route. This allows for some very powerful user interactions. For example, on the course we built a music player application, now you'd expect the 'play' event on a song should do the same thing no matter where you are in the application. Ergo, you can simply handle the event in the application route and be certain that no matter how you trigger the event, it will always do the same thing!

To be continued...

[ember]: http://emberjs.com/
[soc]: http://en.wikipedia.org/wiki/Separation_of_concerns
