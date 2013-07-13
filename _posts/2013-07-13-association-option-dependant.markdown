---
layout: post
title:  "Association Option: dependant"
date:   2013-07-13 11:51:47
categories: rails
---

So I'm working on some billing systems for a healthcare application and I just stumbled on some rails magic I didn't even know was there. Both these options are methods for handling cases where you don't want to delete associated records but need to handle the reference to a record that no longer exists.

I'm sure everyone is familiar with adding the following assoication:

{% highlight ruby %}
class BillingReport < ActiveRecord::Base
   has_many :billing_items, dependent: :destroy
end
{% endhighlight %}

But I just discovered :restrict and :nullify which are awesome for a few rare associations you may need to accomidate for.

`:restrict` means that trying to destroy a record that has any assoications attached will throw a [ActiveRecord::DeleteRestrictionError][DeleteRestrictionError]. This basically takes away the ability to delete records with child assoications. Great if you want the user to really think about what they are trying to do and if they're sure this needs to be deleted should first handle the child records.

`:nullify`, which is what I need, will take all the billing_items and change their billing_report_id to null. This means that if a billing report is wrong (due to user error or something) it can safely be destroyed and all the billing_items will return to the unassigned state allowing future reports to handle them appropriately.

[DeleteRestrictionError]: http://apidock.com/rails/ActiveRecord/DeleteRestrictionError