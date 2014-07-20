---
layout: post
title:  "Apprentice.IO - Week 6"
date:   2014-07-20 11:27:00
categories: thoughtbot apprenticeship month-2
---

This week I did some setup for the apprentice breakable toy involving setting up a ember front end using [ember-cli](http://iamstef.net/ember-cli/) and a rails back end. Once I had both repositories generated I decided to tackle authentication as I've never really had to authenticate on the front end. This was a lot of fun and I wanted to go over the solution I produced.

Lets start on the front end. I found [ember-simple-auth](https://github.com/simplabs/ember-simple-auth) while looking for authentication solutions and it looked like exactly what we needed. It basically sets up a session object to handle all your front-end authentication concerns. There is a fantastic [blog post on simplelabs](http://log.simplabs.com/post/90339547725/using-ember-simple-auth-with-ember-cli) concerning getting ember-simple-auth setup with ember-cli so I won't bother repeating that here. There are a few things I added though which I think is worth talking about.

The guide suggests using a simple-auth initializer to give simple-auth access to your applications ENV. I also found it a good place to override settings to allow for easier testing. Here is the initializer in our application:

{% highlight coffee-script linenos %}
# app/initializers/simple-auth-config.coffee
`import Ember from 'ember'`

SimpleAuthInitializer =
  name: 'simple-auth-config'
  before: 'simple-auth'
  initialize: () ->
    window.ENV = DiplomacyFrontendENV

    if Ember.testing is true
      window.ENV['simple-auth'] =
        store: 'simple-auth-session-store:ephemeral'
      window.ENV['simple-auth-oauth2'] =
        serverTokenEndpoint: 'api/oauth/token'

`export default SimpleAuthInitializer`
{% endhighlight %}

Line 10 is setting the [store](http://ember-simple-auth.simplabs.com/ember-simple-auth-api-docs.html#SimpleAuth-Stores-Base) for testing to use `ephemeral`. By default, ember-simple-auth will use `local-storage` which means that you don't get logged out on page refreshes. However, in testing we want to throw away our sessions with each test so we set the store to use `ephemeral`. Line 26 sets our token endpoint to an internal express server that ember-cli will setup for us if we run the api-stub generator. Speaking of which, we can run `ember g api-stub oauth/token` and edit `server/routes/oauth/token.js` as follows:

{% highlight js linenos %}
// server/routes/oauth/token.js
module.exports = function(app) {
  var express = require('express');
  var tokenRouter = express.Router();
  tokenRouter.post('/', function(req, res) {
    if(req.body.username === 'registeredUser@example.com') {
      res.status(200).send({
        "access_token":"fake_token",
        "token_type":"bearer",
        "expires_in":7200
      });
    } else {
      res.status(401).send({
        "error":"invalid_resource_owner",
        "error_description":"Wrong Username or Password"
      })
    }
  });
  app.use('/api/oauth/token', tokenRouter);
};
{% endhighlight %}

This will allow us to write some integration tests to ensure our login and logout capabilities are working:

{% highlight coffee-script linenos %}
# tests/acceptance/visitor-signs-in-test.coffee
`import Ember from 'ember'`
`import startApp from '../helpers/start-app'`

App = null

module 'Acceptance: VisitorSignsIn',
  setup: ->
    App = startApp()

  teardown: ->
    Ember.run App, 'destroy'

test 'visitor sucessfully signs in', ->
  expect(2)

  visit '/login'
  fillIn 'input#identification', 'registeredUser@example.com'
  fillIn 'input#password', 'password'
  click 'button#submit-login'

  andThen ->
    equal currentPath(), 'index'
    user_sees_logout_link()

test 'visitor attempts to sign in with incorrect credentials', ->
  visit '/login'
  fillIn 'input#identification', 'unregisteredUser@example.com'
  fillIn 'input#password', 'password'
  click 'button#submit-login'

  andThen ->
    equal currentPath(), 'login'
    user_sees_login_link()
    user_sees_error_message()

user_sees_logout_link = ->
  equal find('#logout').length, 1

user_sees_login_link = ->
  equal find('#login').length, 1

user_sees_error_message = ->
  equal find('#login-errors').text().trim(), "Wrong Username or Password"
{% endhighlight %}

With this we have a few tests checking we can login and logout. If your following through, one of these tests should fail as we haven't yet implemented showing error messages on the login form. We can get this passing with the following change:

{% highlight html linenos %}
<!-- app/templates/login.hbs -->
{{ "{{#if loginErrorMessage" }}}}
  <div id=login-errors>
    {{ "{{loginErrorMessage"}}}}
  </div>
{{ "{{/if"}}}}

<form {{action 'authenticate' on='submit'}}>
  <label for="identification">Login</label>
  {{ "{{input id='identification' placeholder='Enter Login' value=identification"}}}}
  <label for="password">Password</label>
  {{ "{{input id='password' placeholder='Enter Password' type='password' value=password"}}}}
  <button id=submit-login type="submit">Login</button>
</form>
{% endhighlight %}

And with that we should have some passing tests and an ember-cli application ready to authenticate with a rails backend. So lets take a quick look there. We'll start with setting up some authentication. As I'm apprenticing at thoughtbot I thought I'd use their authentication solution, [clearance](https://github.com/thoughtbot/clearance). As it turns out it fits our needs perfectly as we want users to be able to sign up on their own and clearance manages password resets and all that good stuff for us. The installation guide is straightforward enough so I won't go aver setting that up here.

Next we need to make our rails backend an oauth provider. As with most things, there's a gem for that! [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper) is a fantastic gem that handles oauth tokens for you. Doorkeepers install process setups almost everything for you so I only really needed to change the generated initializer to get things wired up. Here it is:

{% highlight ruby linenos %}
Doorkeeper.configure do
  orm :active_record

  resource_owner_authenticator do
    session[:return_to] = request.fullpath
    request.env[:clearance].current_user or redirect_to(sign_in_url)
  end

  admin_authenticator do
    user = request.env[:clearance].current_user
    user && user.admin? or redirect_to(sign_in_url)
  end

  resource_owner_from_credentials do |routes|
    User.authenticate(params[:username], params[:password])
  end

  # As we control the front end we skip the authorization
  skip_authorization do |resource_owner, client|
    true
  end
end
{% endhighlight %}

And that's it. Now we have an ember application that will login using our rails application as an oauth provider:

![ember-simple-auth login][ember-simple-auth-rails-backend]

[ember-simple-auth-rails-backend]:/img/ember-simple-auth-rails-backend.gif
