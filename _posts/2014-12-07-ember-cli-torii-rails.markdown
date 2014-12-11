---
layout: post
title:  "EmberCLI, Torii and Rails"
date:   2014-12-07 12:38:11
categories: Ember EmberCLI Github Divshot Torii Rails
excerpt: When we left off we had a ember app setup on divshot to talk to a rails application for authentication via github. We'll start by generating a new rails app.
redirect_from:
  - /ember/embercli/github/divshot/torii/rails/2014/12/07/ember-cli-torri-rails.html
---

This is the second part to my [previous post](
{% post_url 2014-11-02-ember-cli-github-authentication-and-divshot %}
).

## Rails

When we left off we had a ember app setup on divshot to talk to a rails
application for authentication via github. We'll start by generating a new rails
app:

{% highlight bash %}
rails new jibber
cd jibber
bundle
{% endhighlight %}

The first thing we'll want to do is create the route that our ember application
is posting the github temporary token to:

{% highlight ruby %}
# config/routes.rb
Rails.application.routes.draw do
  namespace :v1 do
    resources :sessions, only: [:create]
  end
end
{% endhighlight %}

Then we'll make our sessions controller:

{% highlight ruby %}
# app/controller/v1/sessions_controller.rb
module V1
  class SessionsController < ApplicationController
    skip_before_action :verify_authenticity_token

    def create
      github_authenticator = GithubAuthenticator.new(github_auth_code)
      user_factory = UserFactory.new(github_authenticator)
      user = user_factory.find_or_create_user

      render json: user, status: :created
    end

    private

    def github_auth_code
      params.require(:'github-auth-code')
    end
  end
end
{% endhighlight %}

We have to skip [the verify_authenticity_token
before_action](http://api.rubyonrails.org/classes/ActionController/RequestForgeryProtection.html#method-i-verify_authenticity_token)
which is used to ensure requests are coming from the same origin. As our ember
application will be on a different domain than our rails application, we need
this check to be skipped.

The create action is a great example of writing code I wish I had. The first
line creates a new instance of the `GithubAuthenticator` class. This class
should be passed the github auth token. The second line passes the instance of
`GithubAuthenticator` to instantiate another class that doesn't yet exist, the
`UserFactory`. We then call `find_or_create_user` on the `user_factory`, which
should return a `user`, and pass the result to be rendered as json with a status
of `:created`. We always pass `:created` as the status as we're letting the
browser know that a new `session`, not a `user`, has been created.

Let's go ahead and make the `UserFactory` class so we can spec out what the
`GithubAuthenticator` needs to provide:

{% highlight ruby %}
# app/services/user_factory.rb
class UserFactory
  def initialize(authenticator)
    @authenticator = authenticator
  end

  def find_or_create_user
    User.find_or_create_by(name: authenticator.name)
  end

  private

  attr_reader :authenticator
end
{% endhighlight %}

So we pass an authenticator to the `UserFactory`'s initialize. The
`find_or_create_user` then finds or creates the user using a name provided by
the `authenticator`. There's nothing here specific to github, so adding extra
sign in services is just a case of creating other authenticator classes that
implement a `name` method.

Now let's look at the `GithubAuthenticator` class:

{% highlight ruby %}
# app/services/github_authenticator.rb
require "net/http"
require "net/https"

class GithubAuthenticator
  GITHUB_OAUTH_PATH = "https://github.com/login/oauth/access_token"

  def initialize(auth_code)
    @auth_code = auth_code
  end

  def name
    github_user[:login]
  end

  private

  def github_user
    @github_user ||= github_client.user
  end

  def github_client
    Octokit::Client.new(access_token: access_token)
  end

  def access_token
    github_response["access_token"]
  end

  def token_type
    github_response["token_type"]
  end

  def scope
    github_response["scope"]
  end

  def github_response
    @github_response ||= JSON.parse(res.body)
  end

  def res
    http.request(req)
  end

  def req
    req = Net::HTTP::Post.new(uri.path)
    req.set_form_data(post_data)
    req["Accept"] = "application/json"
    req
  end

  def http
    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    http
  end

  def uri
    URI.parse(GITHUB_OAUTH_PATH)
  end

  def post_data
    {
      "client_id" => ENV["GITHUB_KEY"],
      "client_secret" => ENV["GITHUB_SECRET"],
      "code" => @auth_code
    }
  end
end
{% endhighlight %}

This class is pretty long, and I'd consider extracting a class to handle the
oauth post to github. I won't go over every line, as I hope the code is clear,
but here's an overview. We initialize with the temporary token provided by
github and make a `Net::HTTP` request following the [guidelines provided by
github](https://developer.github.com/v3/oauth/). This returns an oauth token. We
use [Octokit](https://github.com/octokit/octokit.rb) to request the user from
github and extract the `:login` for our `name` method.

We'll need to add `gem "octokit"` to our gemfile and run `bundle`. I also use
[dotenv]() to manage environment variables in development so go ahead and add
that to `dev` and `test` and put your github authentication details into `.env`:

{% highlight bash %}
# .env
GITHUB_KEY=github_develoment_key
GITHUB_SECRET=github_development_secret
{% endhighlight %}

Finally we'll want to create a `User` model and a migration:

{% highlight ruby %}
# app/models/user.rb
class User < ActiveRecord::Base
  validates :name, presence: true, uniqueness: true

  before_create :generate_token

  protected

  def generate_token
    self.token = loop do
      random_token = SecureRandom.urlsafe_base64(nil, false)
      break random_token unless User.exists?(token: random_token)
    end
  end
end
{% endhighlight %}

{% highlight ruby %}
# db/migrate/20141204213638_create_user.rb
class CreateUser < ActiveRecord::Migration
  def change
    create_table :users do |t|
      t.string :name
      t.string :token

      t.timestamps
    end
  end
end
{% endhighlight %}

The user class is simple enough, we generate a unique token on create that the
ember app can use for retrieving a session.

Now if we have everything wired up correctly, with our rails app running:

{% highlight bash %}
rails s -p9000
{% endhighlight %}

and our ember-cli app running:

{% highlight bash %}
ember s --proxy=http://localhost:9000
{% endhighlight %}

we should be able to visit localhost:4200 and click on our login button. We
then see our success alert and can see in the console the rails app returning
our user:

![jibber-auth-success][jibber-auth-success]

[jibber-auth-success]:/img/jibber-auth-success.gif

Now lets get this up on heroku.

## Heroku

Get yourself an account on heroku and install the toolbelt with `brew install
heroku-toolbelt`. Once you're setup, go ahead and create your heroku app. You'll
then want to commit all our changes to git (if you haven't already) and push
your code up to heroku:

{% highlight bash %}
heroku create jibber-rails
git add -A .
git commit
git push heroku master
heroku run rake db:migrate
{% endhighlight %}

We'll also want to set our two environment variables for github credentials:

{% highlight bash %}
heroku config:set GITHUB_KEY=github_production_key GITHUB_SECRET=github_production_secret
{% endhighlight %}

That gets us the same functionality in production!

## Up Next

Next time we'll setup creating a session on the ember side and allowing the
ember app to retrieve the session using the user's token.
