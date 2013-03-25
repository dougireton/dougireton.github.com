---
layout: post
title: "A knife.rb for our time"
date: 2013-03-17 20:44
comments: true
categories: chef, knife
---

Ohai Chefs!

The basic knife.rb you get from the Chef server works, but it's not suitable to check into version control or share with your team. It has the name of your .pem file hardcoded into it and isn't flexible enough for team use. This week we'll look at a generic, flexible `knife.rb` you can keep in your `chef-repo` and share with your team.

<!--more-->

## What you get out of the box

Here's the knife.rb you'll get if you ask the Chef server to generate one for you:

```ruby
current_dir = File.dirname(__FILE__)
log_level                :info
log_location             STDOUT
node_name                "my_username"
client_key               "#{current_dir}/my_username.pem"
validation_client_name   "my_org-validator"
validation_key           "#{current_dir}/my_org-validator.pem"
chef_server_url          "https://chefserver.example.com/organizations/my_org"
cache_type               'BasicFile'
cache_options( :path => "#{ENV['HOME']}/.chef/checksums" )
cookbook_path            ["#{current_dir}/../cookbooks"]
```

## A Better Implementation

Here's what we've come up with after lots of tinkering. It works on Windows and Mac/Linux. Put this in your `chef-repo/.chef/` directory and check it into source control.

```ruby
current_dir = File.dirname(__FILE__)
user_email  = `git config --get user.email`
home_dir    = ENV['HOME'] || ENV['HOMEDRIVE']
org         = ENV['chef_org'] || 'my_org'

knife_override = "#{home_dir}/.chef/knife_override.rb"

chef_server_url          "https://chefserver.example.com/organizations/#{org}"
log_level                :info
log_location             STDOUT

# USERNAME is UPPERCASE in Windows, but lowercase in the Chef server,
# so `downcase` it.
node_name                ( ENV['USER'] || ENV['USERNAME'] ).downcase
client_key               "#{home_dir}/.chef/#{node_name}.pem"
cache_type               'BasicFile'
cache_options( :path => "#{home_dir}/.chef/checksums" )

# We keep our cookbooks in separate repos under a ~/chef/cookbooks dir
cookbook_path            ["#{current_dir}/../../../cookbooks"]
cookbook_copyright       "Your Company, Inc."
cookbook_license         "none"
cookbook_email           "#{user_email}"

http_proxy               "http://webproxy.example.com:80"
https_proxy              "http://webproxy.example.com:80"
no_proxy                 "localhost, 10.*, *.example.com, *.dev.example.com"

# Allow overriding values in this knife.rb
Chef::Config.from_file(knife_override) if File.exist?(knife_override)

```

## The Goods
Most of this should be self-explanitory, but there are a couple of interesting things to note. We are getting the user's email from git config on the fly. We use this to set the `cookbook_email` attribute so it's automatically populated when you create a new cookbook. On line three, we are getting the Home directory which is `HOME` on Mac and `HOMEDRIVE` on Windows. On line four, we are getting the `org` variable. It will default to `my_org`, but allows you to override it by setting the `chef_org` environment variable.

## Knife Override
On line six, we are setting the path to a `knife_override.rb` file. We source the file at the end of this `knife.rb` so you can override any values specified in this `knife.rb`. So far we've never used it, but it seemed like a good idea at the time.

## Client keys
On line 14 we are setting the `node_name` which is your Chef server username. On Mac/Linux, your username is stored in the `USER` env variable. On Windows, it's `USERNAME`. Windows usually stores your username in UPPERCASE. We downcase it here since Chef server usernames are lowercase.

On line 15, we are specifying the location of the user's Chef client key (.pem file). We store it in our home directories since we use the same client key for mulitple orgs.

## Proxies, etc
The rest of the file is pretty straightforward. We use a proxy server at work, so we specify proxy settings. Finally, we source our `knife_override.rb` if it exists.

## Wrap Up

So there you have it. We use this same `knife.rb` in all our `chef-repos` and check it into version control. When someone wants to use our repo, they just check it out of Git and they can start using knife immediately. The only other step they have to do is to move their client key (username.pem file) to `~/.chef/`.

I'd be remiss if I didn't point out Joshua Timberman's excellent ["Commented knife.rb for all the things"][1]. Lot's of good ideas in there too.

[1]: https://gist.github.com/jtimberman/1718805

Let me know any cool tricks you're using in your `knife.rb`. Thanks and see you next week!
