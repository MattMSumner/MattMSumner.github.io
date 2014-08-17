---
layout: post
title:  "Apprentice.IO - Week 10"
date:   2014-08-17 11:51:14
categories: thoughtbot apprenticeship month-3
---

First things first.

This week I was made an offer to work at thoughtbot which I accepted. The
company has always done more then I expect and everyone here is a pleasure to
work with. I can't imagine going anywhere else after the apprenticeship. I'm
taking a week off before starting so my official start date is the 15th of
September.

I spent some more time working on the test suite have rewritten all the tests.
The next big hurdle was moving from selenium to a headless browser for the
integration tests. As it's maintained by thoughtbotters,
[capybara-webkit](https://github.com/thoughtbot/capybara-webkit) is a popular
choice. However, all our specs broke with the following error:

    Failure/Error: click_on 'All'
     Capybara::Webkit::ClickFailed:
       Failed to click element /html/body/div/div[1]/div/ul/li[1]/a because of overlapping element /html/body/div/div[1]/div/ul/li[4]/a at position 305, 848;
       A screenshot of the page at the time of the failure has been written to /var/folders/2j/ky7q20tj15g_01ny2ftfcrwr0000gn/T/click_failed_rc1818.png

Looking at the screen shot showed that all of our links on the dashboard were
being rendered on top of one another. Now another apprentice,
[Justin](http://reallybusywizards.com/), had Joël as a mentor last month and had
looked into this problem himself. He told me it was a particular css rule that
gave the links a position of absolute. This rule is required so that the links
fill the parent `<li>` elements. However, in capybara-webkit, these `<li>`
elements were not rendering with any padding and so all appeared on top of one
another. Without the `position: absolute;`, the links would somehow now act as
blocks and not be on top of one another so the tests would pass.

This seems a issue with webkit as in chrome and firefox everything renders
correctly. I think it has something to do with the elements having 0 height but
I'm not entirely sure what is happening here. The css rule only applies to
screens large then an iPhone's, so my eventual solution was to run the tests at
that resolution. While I'm not particularly happy with the solution, it does get
the tests to pass without changing any code and I can justify it as the
application has been developed mobile first. Here is what I needed to add to
have rspec run all integration tests on a small screen:

{% highlight ruby %}
# spec/support/screen_size.rb
RSpec.configure do |config|
  config.before(:each, :js => true) do
    page.driver.resize_window(320, 568)
  end
end
{% endhighlight %}

It does raise a question of testing responsive design. Should we be running our
integration specs as many times as screen sizes we account for? The answer seems
obvious when the interaction is different at different resolutions (e.g.
hamburger menus on small screens vs sidebars for large) but what about the rest
of the time? I don't have a good answer for this and most developers seem to
only test at whatever is the default.

There was also the Boston ruby meetup and ember meetup this week. Both were
great and I managed to talk to one of the core team members for ember, [Robert
Jackson](https://github.com/rwjblue). This was awesome and I'm still getting
used to meeting these contributors to open source. The community we have in this
industry is magical and I'm looking forward to getting more involved.
