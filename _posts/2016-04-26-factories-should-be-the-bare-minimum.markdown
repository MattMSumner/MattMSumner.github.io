---
layout: post
title: Factories Should be the Bare Minimum
date:   2016-04-26 10:24:13
categories: testing rails web factory-bot
excerpt: Keep it simple.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/factories-should-be-the-bare-minimum)*

I left the following feedback on a recent pull request:

```rb
factory :deliverable do
  title "A cool title"
  tags { { assigned: true, status: "draft" } }
```

> Is this required for the factory to be valid? How would you feel about having
> the factory only add the bare minimum required to create this model and then
> add the other attributes in the test or as traits?

> This is beneficial as tests that don't require this additional setup can avoid
> that work. Additionally, our test suite could then catch if there is a problem
> with nil value here. By having our factories only give us the minimum to
> create a valid record, we are testing creation of models with the minimal set
> of data.

If a property or relationship is really required for a factory to make sense
then we should add a validation before adding it to the base factory of that
model. Let's take the checklist example: does a checklist make sense
without any list items? This is the first state your users may see. By adding
list items to the default factory you're opening your test suite up to
surprising behavior. Not only that but you are introducing a [mystery guest]!

[mystery guest]: https://thoughtbot.com/blog/mystery-guest

Imagine that you are new on a project and for your first feature you write a
presenter for some data. You write the following test:

```rb
require "rails_helper"

describe CodeListPresenter do
  context "#data" do
    context "with code_list_items" do
      it "returns those code_list_items for a standard" do
        standard = create(:standard)
        code_list = create(:code_list, standard: standard)
        code_list_item = create(:code_list_item, code_list: code_list)

        code_list = CodeListPresenter.new(standard: standard)

        expect(code_list.data[:code_list_items].count).to eq(1)
      end
    end
  end
end
```

And you get the following failure:

```bash
F

Failures:

  1) CodeListPresenter#data with code_list_items returns those code_list_items for a standard
     Failure/Error: expect(parameter_design.data[:values].count).to eq(2)

       expected: 1
            got: 3

       (compared using ==)
     # ./spec/models/parameter_design_spec.rb:44:in &#96;block (5 levels) in <top (required)>'

Finished in 0.35508 seconds (files took 3.86 seconds to load)
1 example, 1 failure

Failed examples:

rspec ./spec/models/parameter_design_spec.rb:33 # CodeListPresenter#data with code_list_items return those code_list_items for a standard

Randomized with seed 22400
```

What? Does this mean that the factory creates the extra two items or does
`code_list` create two `code_list_item`s in a callback? It's not clear from
this test what is happening. After some investigation you find out that it's in
the factory and you remove it. Removing it causes 15 or so specs to start
failing. You fix the ones that obviously depend on a `code_list_item` being
present but you find a couple that fail because the case of no `code_list_items`
had not been considered. You fix the bugs and submit a PR. Not bad for your
first contribution!

Of course, always consider [if you really need a factory] in the first place.

We talk about this more in the upcoming [testing-rails] book. You can see the
announcement [post here].

If you'd like to find out more about factory girl, checkout this free episode of
[the Weekly Iteration].

[if you really need a factory]: https://thoughtbot.com/blog/speed-up-tests-by-selectively-avoiding-factory-girl
[testing-rails]: http://testingrailsbook.com?utm_source=giant-robots&utm_medium=blog
[post here]: https://thoughtbot.com/blog/announcing-testing-rails
[the Weekly Iteration]: https://thoughtbot.com/upcase/videos/factory-girl?utm_medium=blog
