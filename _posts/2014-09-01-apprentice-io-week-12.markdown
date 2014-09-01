---
layout: post
title:  "Apprentice.IO - Week 12"
date:   2014-09-01 12:06:59
categories: thoughtbot apprenticeship month-3
---

This week I spent a bit of time restyling this blog. I had some help from [Mike
Boresare](http://mikeborsare.com/) a design apprentice who started just a few
days before me. Congratulations to him as he was also hired by thoughtbot.

I'm still planning to change a few things here and there to improve the blog,
for instance it seem commenting is broken at the moment, but it seems to be a
step in the right direction. I'd love any feedback and I hope the site is now
easier to read.

I wrote my first [view
spec](https://www.relishapp.com/rspec/rspec-rails/v/2-0/docs/view-specs/view-spec)
which I enjoyed especially after a recent dev discussion where view specs were
the topic. View specs are really only for testing logic in the view, which we're
told there should be as little logic there as possible. It stands to reason that
view specs should be small if they exist at all. In my case, I had a feature
where if a user was logged in as certain roles they would see a link while other
roles would see plain text. This was a simple `if` `else` in the view therefor
perfect functionality to justify a view spec. With a view spec I could quickly
test against all roles that the correct link was generated.

{% highlight ruby linenos %}
require 'spec_helper'

describe 'tickets/_ticket_details' do
  context 'as a manager' do
    it 'renders editable fields as links' do
      ticket = create(:ticket, priority: 'Urgent')
      permitted_attributes = get_permissions_for(ticket, :manager)

      render(
        partial: 'web/tickets/ticket_details',
        locals: {
          ticket: ticket,
          permitted_attributes: permitted_attributes
        }
      )

      expect_user_sees_edit_link(ticket.priority)
      expect_user_sees_edit_link(I18n.l(ticket.due_date))
    end
  end

  context 'as a technician' do
    it 'renders editable fields as links' do
      ticket = create(:ticket, priority: 'Urgent')
      permitted_attributes = get_permissions_for(ticket, :technician)

      render(
        partial: 'web/tickets/ticket_details',
        locals: {
          ticket: ticket,
          permitted_attributes: permitted_attributes
        }
      )

      expect_user_sees_field_as_text(ticket.priority)
      expect_user_sees_field_as_text(I18n.l(ticket.due_date))
    end
  end

  def expect_user_sees_edit_link(text)
    expect(rendered).to have_content text
  end

  def expect_user_sees_field_as_text(text)
    expect(rendered).to have_content text
    expect(rendered).not_to have_css(
      'a.load-target-in-panel.button', text: text
    )
  end

  def get_permissions_for(ticket, user_type)
    policy = TicketPolicy.new(create(user_type), ticket)
    policy.permitted_attributes
  end
end
{% endhighlight %}
