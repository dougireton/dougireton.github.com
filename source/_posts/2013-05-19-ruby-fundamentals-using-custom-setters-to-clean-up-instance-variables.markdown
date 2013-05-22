---
layout: post
title: "Ruby Fundamentals: Using Custom Setters to clean up Instance Variables"
date: 2013-05-19 21:56
comments: true
categories: ruby
---
Ohai Fellow Rubyists!

This week we're going to look at using custom setter methods in your Ruby class
initializers to do any custom logic before setting instance variables. I just
had to do this last week for a gem I'm writing. Here's a quick snippet. Read on
for the full explanation.

```ruby
class StashNotifier
  attr_reader :job_status

  VALID_JOB_STATUSES = %w{ INPROGRESS SUCCESSFUL FAILED }

  def initialize(args)
    # other instance vars omitted
    self.job_status  = args[:job_status]
    @job_key         = args[:job_key]
    @job_url         = args[:job_url]
  end

  def job_status=(new_job_status)
    new_job_status = new_job_status.strip.upcase

    if VALID_JOB_STATUSES.include?(new_job_status)
      @job_status = new_job_status
    else
      raise ArgumentError, "'#{new_job_status}' is not a valid Stash Build
Status! Valid job statuses are #{VALID_JOB_STATUSES}."
    end
  end
# rest of class omitted
end

```
<!-- more -->

## Background
Recently, I've been working on a gem to send Jenkins build status to our [Stash git server][1] as part of our CD pipeline. 

This gem, 'stash\_notifier', takes `job_status` as an argument in its class initializer. At first I just set the `@job_status` instance variable directly like this:

```ruby
  def initialize(args)
    # other instance vars omitted
    @job_status = args[:job_status]
    @job_key    = args[:job_key]
    @job_url    = args[:job_url]
  end
```

But then I realized I really should `upcase` the `job_status` per the [Atlassian SDK docs][2]. Plus I wanted to check the `job_status` parameter the user passed into the `StashNotifier.new` method to make sure it was valid. To solve this I wrote a custom setter.

[1]: http://www.atlassian.com/software/stash/overview
[2]: https://developer.atlassian.com/stash/docs/latest/how-tos/updating-build-status-for-commits.html#Updating_build_results

## Custom Setters
Ruby includes some nice helper methods, `attr_reader`, `attr_writer`, and `attr_accessor` to create setters and getters for you. These just set or return the instance variables and don't allow for any customization. If you want to do some validation or modify the passed-in parameters, define a custom setter like this:

```ruby
Class StashNotifier
  attr_reader :job_status

  def initialize(args)
    # other instance vars omitted
    self.job_status  = args[:job_status]
    @job_key         = args[:job_key]
    @job_url         = args[:job_url]
  end

  def job_status=(new_job_status)
    # define custom logic here
    # see code sample above for full example
  end
end
```

We first define an `attr_reader` to have Ruby create the getter for us, while still letting us define a custom setter.

Instead of setting the `job_status` instance variable in our class initializer, we call the custom setter, `job_status=`. By defining it as a method with a trailing `=`, we make it a setter. We prepend it with `self.` to specify that it's a method and not a local variable. This setter method will be called during initialization, *and* anytime we do this:

```ruby
  notifier = StashNotifer.new(args)
  notifier.job_status = 'failed'
```

## Summary
I hope you've found this useful. It took quite a bit of Googling for me to find out this little tidbit. Here are a couple of links I found helpful during my research:

1. http://ruby.about.com/od/oo/ss/Using-Attributes.htm
2. http://stackoverflow.com/questions/12097726/ruby-classes-initialize-self-vs-variable
