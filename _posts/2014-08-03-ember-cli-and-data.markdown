---
layout: post
title:  "Ember Cli and Data"
date:   2014-08-03 14:51:41
categories: ember ember-cli
---

I've been working on a breakable toy with a couple other developers and we
decided to build it using ember with a rails backend. This was a perfect
opportunity to try developing with [ember-cli](http://www.ember-cli.com/). I
wanted to talk about some of the tools it provides for dealing with data and
when I think you should use them.

Before we start a quick disclaimer; ember-cli is currently in heavy development,
we started on version 0.39, are now on 0.40 and breaking changes are coming
thick and fast. With that out of the way lets start with something native to
ember.

Fixture
-------

[Fixtures](http://emberjs.com/guides/models/the-fixture-adapter/) are ember's
solution for quick development of your application before the back end is ready.
It allows us to setup fake data for each model so we can see what our
application will look like with data before wiring things up.

The guides suggest attaching your fixtures to to your models using the global
`App` variable, however, ember-cli uses E6 modules, therefore we need to reopen
the class as described in [this blog
post](http://edgycircle.com/blog/2014-using-fixtures-in-combination-with-ember-cli/)
.

These end up appearing in your development environment and your tests. This is
not ideal, as it leads to [mystery
guests](http://robots.thoughtbot.com/mystery-guest) in your tests. It's also not
ideal for your development as there will be difficulties when it comes to wiring
up to the backend. For example, the `FixtureAdapter` [doesn't have querying data
implimented](https://github.com/emberjs/data/blob/v1.0.0-beta.8/packages/ember-data/lib/adapters/fixture_adapter.js#L89)
. If your app needs to query your backend (chances are it will) then you'll be
better off waiting on implementing that until the backend catches up to ensure
you don't waste time and resource.

Using fixtures is great as you can develop without having to concern yourself
about how the backend will look. Fixtures are also very dangerous for this same
reason. If you already have a backend then you should just wire everything up
from the beginning. If you don't, then you want your frontend to drive backend
development. In this case you should use the second tool I'd like to talk about.

API-stubs
---------

API-stubs are provided by ember-cli and can be very confusing as there is little
documentation concerning them. You can start using api-stubs by running the
generator provided by ember-cli:

`
ember g api-stub posts
`

This will create three files within a server folder, an `index.js`, a
`.jshintrc` and `routes/posts.js`. The index can be ignored but it defines a
minimal express server that ember-cli will start when you run `ember server`.
This server will use the files you've defined under the routes folder to return
requests. Here you can specify fake data that your backend will eventually
provide. Let's take a look at our `posts` file:

```javascript
module.exports = function(app) {
  var express = require('express');
  var postsRouter = express.Router();
  postsRouter.get('/', function(req, res) {
    res.send({posts:[]});
  });
  app.use('/api/posts', postsRouter);
};
```

We can update this to return our fake data:

```javascript
module.exports = function(app) {
  var express = require('express');
  var postsRouter = express.Router();
  postsRouter.get('/', function(req, res) {
    res.send({posts:[
        {id: '1', title: 'Awesome', body: 'first post'},
        {id: '2', title: 'Wah', body: 'second post'}
      ]});
  });
  app.use('/api/posts', postsRouter);
};
```

Run `ember serve` and see if it works!

```bash
[~] curl http://localhost:4200/api/posts
{"posts":[{"id":"1","title":"Awesome","body":"first post"},{"id":"2","title":"Wah","body":"second post"}]}
```

Now we can develop our ember application while also creating robust examples of
how we expect the backend to behave.

API-stubs are fantastic for driving development forward so we can understand
precisely what we require from the backend before writing any code. Once
completing a feature we can produce a fully developed example of how our api
should act.

The small express server that delivers the api-stubs when we run ember serve
does not get run when we run ember test. This is an indication from ember-cli
that we are not expected to use api-stubs in our tests. This confused me at
first, as I had assumed api-stubs were designed for testing. I started looking
around for other solutions to testing and found one in an [example
app](https://github.com/simplabs/ember-cli-simple-auth-example) from the
creators of [ember-simple-auth](https://github.com/simplabs/ember-simple-auth).

Pretender
--------
[Pretender](https://github.com/trek/pretender) is a mock server library for
capturing requests and returning stubbed responses. To get pretender in you
tests first install via `npm install ember-cli-pretender --save`. Here is a test
from our diplomacy app using pretender:


```coffee-script
`import startApp from '../helpers/start-app'`

App = null
server = null

module 'Authentication',
  setup: ->
    App = startApp()
    server = new Pretender ->
      this.post '/token', (request) ->
        [
          200
          { "Content-Type": "application/json" }
          '{ "access_token": "access_token" }'
        ]
      this.get '/api/current_users/me', (request) ->
        [
          200
          { "Content-Type": "application/json" }
          '{ "current_users": {"id":"me", "email":"registeredUser@example.com", "server_id":"1"} }'
        ]

  teardown: ->
    Ember.run App, 'destroy'
    server.shutdown()

test 'visitor sucessfully signs in', ->
  expect(3)

  visit '/login'
  fillIn 'input#identification', 'letme'
  fillIn 'input#password', 'in'
  click 'button#submit-login'

  andThen ->
    equal currentPath(), 'index'
    user_sees_logout_link()
    user_sees_own_email()

user_sees_logout_link = ->
  equal find('#logout').length, 1

user_sees_own_email = ->
  equal find('#current-user').text().trim(), 'registeredUser@example.com'
```

To be honest I don't feel great that a lot of this test file concerns stubbing
our requests but it does allow me to be clear about what the server is
responding with for each test.

Closing
-------
Obviously I'll be working more with api-stubs and pretender and I'll post more
on my experience with them here. Ember-cli is still very much a work in progress
but I've found working with it to be a joy. Although fixtures are easy to
employ, I've found using pretender makes it far easier to test drive the
application and api-stubs allows me to charge forward with code that expects a
backend without having built one out.
