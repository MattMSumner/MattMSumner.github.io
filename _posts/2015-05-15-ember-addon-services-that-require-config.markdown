---
layout: post
title:  "Ember Addon - Services that require config"
date:   2015-05-15 15:16:20
categories: Ember EmberCLI Addons
---

So while trying to create a service in an addon I ran into an issue trying to
access config, I found [this post] with a solution that no longer works.
Eventually I found [a post on the Ember London forum] with the solution that
worked for me:

{% highlight javascript %}
// app/services/service-name.js
import ServiceName from 'emberfire/services/service-name';
import config from '../config/environment';

export default ServiceName.extend({
  config
});
{% endhighlight %}

{% highlight javascript %}
// addon/services/service-name.js
import Ember from 'ember';

export default Ember.Service.extend({
  // awesome service code.
});
{% endhighlight %}

Hope this helps anyone looking.

[this post]: http://discuss.emberjs.com/t/best-practices-accessing-app-config-from-addon-code/7006/2
[a post on the Ember London forum]: http://discuss.emberlondon.com/t/access-app-config-from-addon-provided-service/132/6
