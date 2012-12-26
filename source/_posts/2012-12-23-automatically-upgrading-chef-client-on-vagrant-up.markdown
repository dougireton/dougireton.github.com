---
layout: post
title: "Automatically upgrading Chef client on Vagrant Up"
date: 2012-12-23 19:51
comments: true
categories: chef
---
Ohai Chefs!

If you do enough Vagrant testing, you'll soon run into a Vagrant box with an outdated Chef client. Even the Opscode Test Kitchen boxes come with Chef 10.14.4, not the latest 10.16.2 version. In this post I'll show you how to automatically upgrade the Chef client on `vagrant up`. The trick is to use two `config.vm.provision` blocks in your Vagrantfile.

<!--more-->

## Download the Opscode Chef-Client cookbook and add an Upgrade recipe

You'll need a Chef recipe to upgrade your Chef client. Below is a recipe we added to our fork of the [Opscode Chef-Client] [1] cookbook.

[1]: http://community.opscode.com/cookbooks/chef-client

``` ruby Chef Client Upgrade Recipe
# put this in `chef-client/recipes/upgrade.rb`

windows_package "Opscode Chef Client Installer for Windows v10.16.2" do
  source "https://www.opscode.com/chef/install.msi"
  action :install
end
```
We're using Windows in this case but obviously, use the Chef client for your platform. We've successfully used this pattern with Linux boxes as well.

## In your Vagrantfile

``` ruby Vagrantfile
config.vm.provision :chef_solo do |chef|
  # this provision block upgrades the Chef Client before the real 
  # Chef run starts
  chef.run_list = [
    "recipe[chef-client::upgrade]"
  ]
end

# This is the real Chef run
config.vm.provision :chef_solo do |chef|
  chef.run_list = [
    "recipe[my_recipe]",
    "recipe[my_other_recipe::beer]"
  ]
end
```

## OK, but two Chef runs? Really?

{% blockquote %}
"But, why can't you just use a single provision block and add the Chef Client upgrade recipe to the first position in the run list?" 
{% endblockquote %}
That was my question the first time Kevin showed me this pattern. He explained,
{% blockquote %}
"Because, Chef will complete the Chef run with the same version it started with."
{% endblockquote %}

In other words, the first Chef run starts with 10.x and upgrades itself. The second Chef run starts with the new, shiny, upgraded client.

## Credit Where Credit is Due
Once again, mad props to my Talented and Gifted&trade; teammate [Kevin] [2] for figuring this out. This really helped us out when we were stuck with a RHEL 5.8 Vagrant box with Chef 10.8 and needed to test Chef recipes written for Chef 10.16.2.

[2]: https://twitter.com/moserke

## Ladies and Gentlemen, `'vagrant up'`
Now everytime, you `vagrant destroy` and `vagrant up`, you'll have the latest Chef Client without having to crack open and repackage your Vagrant box. Happy testing and let me know how it works for you.
