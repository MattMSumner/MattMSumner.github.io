---
layout: post
title:  "EmberCli, Torii Sessions"
date:   2015-02-09 17:28:03
categories: Ember EmberCLI Github Divshot Torii Rails
excerpt: So far, we have a ember-cli application being served by divshot and a rails app providing authentication up on heroku. This is pretty sweet but we need to actually do something with the user returned by our backend.  Torii comes with a built in session that we're going to take advantage of.
---

This is the third part of a series of posts. Checkout [part 1](
{% post_url 2014-11-02-ember-cli-github-authentication-and-divshot %}
) and [part 2](
{% post_url 2014-12-07-ember-cli-torii-rails %}
).

## currentUser

So far, we have a ember-cli application being served by
[divshot](https://divshot.com/) and a rails app providing authentication up on
[heroku](https://www.heroku.com/). This is pretty sweet but we need to actually
do something with the user returned by our backend.
[Torii](http://vestorly.github.io/torii/) comes with a built in session that
we're going to take advantage of. It's pretty simple to extend our torii adapter
to give our `currentUser`:

{% highlight javascript linenos %}
// app/torii-adapters/application.js
import Ember from "ember";
import config from "../config/environment";

export default Ember.Object.extend({
  open: function(authorization){
    return this._fetchSession({
      "github-auth-code": authorization.authorizationCode
    });
  },

  _fetchSession: function(tokenData) {
    return new Ember.RSVP.Promise(function(resolve, reject){
      Ember.$.ajax({
        url: config.jibber.sessionUrl,
        type: "POST",
        data: tokenData,
        dataType: "json",
        success: function(data) {
          localStorage.setItem("token", data.token);
          var currentUser = Ember.Object.create(data);
          Ember.run.bind(null, resolve({ "currentUser": currentUser }));
        },
        error: function(jqXHR, textStatus, errorThrown){
          Ember.run.bind(null, reject({ "message": errorThrown }));
        }
      });
    });
  }
});
{% endhighlight %}
The open property on our adapter needs to return a promise. Once that promise
resolves, torii will merge any objects into the `session` object. On line 22, if
the promise resolves we return a pojo with the currentUser. On line 20 we're
saving the user token to `localStorage` so we can use that later when we
impliment fetching a session.

Let's go ahead and update our application template to show the currentUser:

{% highlight handlebars %}
{% raw %}
<!-- app/templates/application.hbs -->
<h2 id='title'>Welcome to Ember.js</h2>

{{#if session.currentUser}}
  Welcome {{session.currentUser.name}}
{{else}}
  {{#if session.isWorking}}
    One sec while we get you signed in...
  {{else}}
    {{error}}
    <button {{action 'signInViaGithub'}}>
      Jibber
    </button>
  {{/if}}
{{/if}}

{{outlet}}
{% endraw %}
{% endhighlight %}

And now when we sign in we can see the following:

![torii-github-auth][torii-github-auth]

Excellent! But we have a problem. If you refresh the page your sessions goes
away ;_;

## Fetching Sessions
Fixing this is pretty simple but requires some changes on the rails side. Let's
start by making the changes we need in ember before jumping into rails:

{% highlight javascript %}
app/routes/application.js
import Ember from "ember";

export default Ember.Route.extend({
  activate: function() {
    this.get("session").fetch();
  },

  actions: {
    signInViaGithub: function() {
      var route = this;

      this.get("session").open("github-oauth2").then(function(authorization){
        // do the things after login, like redirect to dashboard
      }, function(error) {
        route.controller.set("error", "Could not sign you in: " + error.message);
      });
    }
  }
});
{% endhighlight %}

The changes to application route make sure we fetch the session when the route
is activated. This should only happen once after the app boots. Now let's add
fetch to our adapter:

{% highlight javascript %}
// app/torii-adapters/application.js
import Ember from "ember";
import config from "../config/environment";

export default Ember.Object.extend({
  fetch: function() {
    return this._fetchSession({
      "token": localStorage.getItem("token")
    });
  },

  ...
});
{% endhighlight %}

And that should be all we need here so let's dive back over to the rails app to
make sure we can pass a github token or our custom token to the session create
action:

{% highlight ruby %}
module V1
  class SessionsController < ApplicationController
    skip_before_action :verify_authenticity_token

    def create
      if fetch_or_create_user
        render json: user, status: :created
      else
        render nothing: true, status: :bad_request
      end
    end

    private

    def fetch_or_create_user
      if github_auth_code
        find_or_create_user_from_github
      elsif token
        User.find_by(token: token)
      end
    end

    def find_or_create_user_from_github
      github_authenticator = GithubAuthenticator.new(github_auth_code)
      user_factory = UserFactory.new(github_authenticator)
      user_factory.find_or_create_user
    end

    def token
      @_token ||= session_params[:token]
    end

    def github_auth_code
      @_github_auth_code ||= session_params[:'github-auth-code']
    end

    def session_params
      params.permit(:'github-auth-code', :token)
    end
  end
end
{% endhighlight %}

Now our sessions controller handles fetching old sessions! All we need to do is
push up to Heroku and Divshot and see if everything works.

{% highlight bash %}
cd ~/development/jabber
ember build --environment=production
divshot push
cd ~/development/jibber
git push heroku master
{% endhighlight %}

You can checkout my version [here](http://jabber.divshot.io/) and the source on
[my github](https://github.com/MattMSumner):
[jibber](https://github.com/MattMSumner/jibber)
[jabber](https://github.com/MattMSumner/jabber)

[torii-github-auth]:/img/torii-github-auth.gif
