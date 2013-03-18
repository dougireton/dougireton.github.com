---
layout: post
title: "Send application deploy times to StatsD in a Chef recipe"
date: 2013-03-02 14:48
comments: true
categories: metrics, chef
---

Ohai Chefs!

This week, I'll show you how to time application deploys (or anything else) inside a Chef recipe and send metrics to [StatsD][1].

[1]: http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/

At work, we're working to integrate metrics into more and more aspects of our development pipeline. We are already sending Chef run data to Graphite and [Chef client version metrics][1] to StatsD/Graphite. This past week, I worked on timing our application deploys via the [Statsd-Ruby][3] library inside a Chef recipe. Read on to see how easy it is.

[2]: http://dougireton.com/blog/2013/01/19/get-chef-clients-by-version
[3]: https://github.com/reinh/statsd

<!--more-->

## Deploy Recipe Part One - Require StatsD

```ruby
GEM_SERVER = node['gem_server']

chef_gem 'statsd-ruby' do
version '1.2.0'
options("--clear-sources --source #{GEM_SERVER}")
end

require 'statsd'
```

We use an internal Gem server, since our servers are behind firewalls without open access to the Internet. The first part of our app deployment recipe just gets the `statsd-ruby` gem installed and ready.

## Deploy Recipe Part Two - Start the Timer

```ruby

statsd = Statsd.new('mystatsd-server.example.com', 8125)

ruby_block 'Start of app deployment' do
  block do
    # start the timer
    node.run_state['app_deploy_start'] = Time.now
    Chef::Log.info "Starting app deploy at #{node.run_state['app_deploy_start']}"
  end
end
```

In this section of our deply recipe, we initialize a new StatsD server variable and save our app deploy start time in a `node.run_state` variable. I had trouble getting local variables to work for saving the start time for use later so I ended up using Chef's `node.run_state` to save the start time. Big thanks to [Chris McClimans][4] for this solution.

[4]: https://twitter.com/hippiehacker

## Deploy Recipe Part Three - Send Elapsed Time to StatsD

```ruby
# --------------------------------------------------------------------
# Insert Chef resources here needed to deploy the app to our web sever
# --------------------------------------------------------------------
execute "Deploy the app" do
  # ...
end

ruby_block "End of app deployment" do
  block do
    # calculate elapsed time for deployment and send to StatsD
    deploy_end = Time.now
    elapsed_time = deploy_end - node.run_state['app_deploy_start']

    # Replace "." with underscores in node name so Graphite doesn't create
    # a bucket for each part of the FQDN
    node_name_underscores = node.name.gsub('.', '_')
    
    # Use `statsd-ruby` timing method to send data to StatsD
    statsd.timing "my_app.#{node.chef_environment}.#{node_name_underscores}.deploy_time", elapsed_time
    Chef::Log.info "App deployment took #{elapsed_time} seconds."
  end
end

```

In the final part of our recipe, we do the actual deploy (not shown), then calculate the elapsed time and send to StatsD in a final Ruby block resource. We have to use a `ruby_block` resource because that will ensure the timing code is run at [Convege time instead of earlier in Compile time][5].

[5]: http://wiki.opscode.com/display/chef/Anatomy+of+a+Chef+Run

## Results
<div>
{% img left /images/app_deploy_times.png 800 600 'App Deploy Times' %}
</div>

As you can see, our app is deploying in roughly 100 seconds. Although we are just starting out with tracking "all the things", it's pretty addicting once you get started. 

Hopefully you've seen this week how easy it is to send metrics to StatsD/Graphite from inside a Chef recipe. I'd love to hear in the comments what kind of stats you are tracking. Thanks for reading.

Finally, I'd be remiss if I didn't point to the blog post which got us started down the metrics road, Etsy's ["Tracking Every Release"][6]

[6]: http://codeascraft.etsy.com/2010/12/08/track-every-release/
