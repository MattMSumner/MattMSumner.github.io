---
layout: post
title:  "EmberCLI, Github authentication and DivShot"
date:   2014-11-02 16:00:30
categories: Ember EmberCLI Github Divshot Torii
excerpt: This will be a multi part post, building a simple chat application. I may never actually build the chat functionality, as I'm more interested in setting up the authentication side of things. Non the less, context helps.
---

I don't want to do authentication.

It's just troublesome. I say let someone else deal with it and I'll just manage
my application. I don't want to store your passwords anymore.

This will be a multi part post, building a simple chat application. I may never
actually build the chat functionality, as I'm more interested in setting up the
authentication side of things. Non the less, context helps.

## Torii

To make authenticating with github super simple we're going to use
[torii](https://github.com/Vestorly/torii) to handle our user flow. Torii has
some really nice abstractions and isn't opinionated on how your app should
behave. It has an optional session manager (which I'll end up using), providers
for authenticating with external services (such as github), and adapters for
talking to your backend.

Let's go ahead and generate a new [ember-cli](http://www.ember-cli.com/) app and
add torii:

{% highlight bash %}
ember new jabber
cd jabber
npm install torii --save-dev
{% endhighlight %}

We can then setup our application template to show a login button:

{% highlight html %}
{% raw %}
<!-- app/templates/application.hbs -->

<h2 id='title'>Welcome to Ember.js</h2>

{{#if session.isWorking}}
  One sec while we get you signed in...
{{else}}
  {{error}}
  <button {{action 'signInViaGithub'}}>
    Jibber
  </button>
{{/if}}

{{outlet}}
{% endraw %}
{% endhighlight %}

`session` is provided by Torii via a config setting we'll supply to
`config/environments` later. Lets get that `signInViaGithub` action setup:

{% highlight javascript %}
// app/routes/application.js
import Ember from 'ember';

export default Ember.Route.extend({
  actions: {
    signInViaGithub: function() {
      var route = this;

      this.get('session').open('github-oauth2').then(function(){
        alert('success!');
      }, function(error) {
        route.controller.set('error', 'Could not sign you in: ' + error.message);
      });
    }
  }
});
{% endhighlight %}

and our Torii adapter so we can control the logic for the temporary token
returned from github:

{% highlight javascript %}
// app/torii-adapters/application.js
import Ember from 'ember';
import config from '../config/environment';

export default Ember.Object.extend({
  open: function(authorization){
    var temporaryCode = authorization.authorizationCode;

    return new Ember.RSVP.Promise(function(resolve, reject){
      Ember.$.ajax({
        url: config.jibber.sessionUrl,
        type: "POST",
        data: { 'github-auth-code': temporaryCode },
        dataType: 'json',
        success: Ember.run.bind(null, resolve),
        error: function(jqXHR, textStatus, errorThrown){
          Ember.run.bind(null, reject({ 'message': errorThrown }));
        }
      });
    });
  }
});
{% endhighlight %}

Here I've placed the url into `config/environment.js` as it will change between
development and production.

{% highlight javascript %}
// app/config/environment.js
...

  if (environment === 'development') {
    ENV.APP.LOG_ACTIVE_GENERATION = true;
    ENV.APP.LOG_VIEW_LOOKUPS = true;
    ENV.jibber = {
      sessionUrl: 'http://localhost:4200/v1/sessions'
    }

    ENV.torii = {
      sessionServiceName: 'session',
      providers: {
        'github-oauth2': {
          apiKey: GITHUB_API_KEY,
          redirectUri: 'http://localhost:4200',
        }
      }
    };

    ENV.contentSecurityPolicy = {
      'connect-src': "'self' http://localhost:9000"
    }
  }

...
{% endhighlight %}

I've modified it slightly, but all of the above can be found on the Torii github
page.

I can then run my server with `ember s --proxy=http://localhost:9000`. The
`--proxy` option means that any ajax requests my app makes will be forwarded to
the provided address. When I end up making my rails app, I'll have it host on
port 9000 and hopefully everything should be wired up.

## Github

Next we'll need to go get an api key from github. Login to github, hit settings
and head to 'Applications' (It should be a menu item on the right). You should
then be able to 'Register new application' and set it's callback URL to
localhost:4200.

![github-application][github-application]

[github-application]:/img/github-application.png

With this setup, when we go to `localhost:4200` we should see our button to
login to github. Pressing it will take you to github and ask you to allow the
app access to you github account. Accepting, will redirect back to the app and
try to post the temporary token from github to `localhost:9000/v1/sessions`. Of
course this won't work yet as there isn't anything listening at port 9000.

Usually at this point I'd start looking at the rails side of things but I'm
leaving that for the next blog post. Instead, I'm going to talk about getting
this off of localhost.

## Divshot
[Divshot](https://divshot.com/) is a static web hosting service that I like to
think of as Heroku for ember-cli. There's even an addon to help get setup so
we're going to be using it. Go get an account setup at divshot and run through
the getting started, then run the following:

{% highlight bash %}
npm install --save-dev ember-cli-divshot
ember generate divshot
{% endhighlight %}

This should give us a `divshot.json` file to setup settings for our app. Let's
add some settings for [proxying](http://docs.divshot.com/services/proxy) to
where our rails app will eventually live on heroku:

{% highlight javascript %}
// divshot.json
{
  "name": "jabber",
  "root": "./dist",
  "routes": {
    "/tests": "tests/index.html",
    "/tests/**": "tests/index.html",
    "/**": "index.html"
  },
  "proxy": {
    "v1": {
      "origin":"https://jibber-rails.herokuapp.com/v1/",
      "headers": {
        "Accept": "application/json"
      },
      "cookies": false,
      "timeout": 30
    }
  }
}
{% endhighlight %}

Now we just need some settings for our new production environment:


{% highlight javascript %}
// app/config/environment.js
...

  if (environment === 'production') {
    ENV.torii = {
      sessionServiceName: 'session',
      providers: {
        'github-oauth2': {
          apiKey: GITHUB_API_KEY,
          redirectUri: 'http://development.jabber.divshot.io',
        }
      }
    }

    ENV.jibber = {
      sessionUrl: '/__/proxy/v1/sessions'
    }
  }

...
{% endhighlight %}

Don't forget to setup a production github application to fill in the
`GITHUB_API_KEY` with.

Now let push our application up divshot. At the time of writing,
`ember-divshot-cli` doesn't build in the production environment before pushing
to divshot so we'll use the plain old divshot-cli (which should have been
installed during signup) and build the site ourselves:

{% highlight bash %}
ember build --environment=production
divshot push
{% endhighlight %}

## Next Time
We'll look into setting up a rails app to authenticate with github and push it
up to Heroku.
