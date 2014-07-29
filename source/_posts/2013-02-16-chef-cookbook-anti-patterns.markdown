---
layout: post
title: "Chef: Patterns and Anti-Patterns for Cookbooks, Environments, Roles"
date: 2013-02-16 20:25
comments: true
categories: chef
---
Ohai Chefs!

In this guide we'll look at some common Chef anti-patterns and I'll suggest alternative patterns which avoid the pitfalls and lead you to Chef nirvana.

This post was inspired by BryanWB's post [How to write reusable cookbooks, Gangnam Style][1]

[1]: http://devopsanywhere.blogspot.com/2012/11/how-to-write-reusable-chef-cookbooks.html

<!--more-->

## Anti-Pattern: Modifying (forking) community cookbooks

I've certainly done this and you probably have too. It starts off fairly innocently. You add an attribute here, maybe update the cookbook's metadata.rb. Pretty soon, you've added recipes, multiple attributes and maybe an LWRP or two. Now you have a mess, especially if the cookbook is under active development on Github. You _will_ have merge conflicts. You may get hard to debug logic errors if the upstream cookbook changes and your changes are similar but slightly different. And how do you version your "Frankenstein" cookbook? Do you take the upstream Github version number? Your own version number based on the upstream version number? In short this is an anti-pattern which can be hard to undo.

The only exeception to this is when you want to share your changes back with the community. You should create a feature branch in Git with your changes and submit a pull request from that branch. This should be a short-lived branch which lives only until the pull request is merged upstream.

## Pattern: Create a wrapper cookbook with your custom recipes and attributes

Instead of forking and modifying an upstream community or Opscode cookbook, you should use it unmodified with a company-specific wrapper cookbook on top. In your wrapper cookbook you will add a metadata dependency on the community cookbook and use `include_recipe` to run recipes from the community cookbook. Set company-specific attributes in your wrapper cookbook to override the community cookbook defaults. Company-specific recipes should go into this wrapper cookbook as well.

See [Chef-Rewind][2] for a gem which can help you with this pattern.

[2]: https://github.com/bryanwb/chef-rewind

## Anti-Pattern: Using Role Attributes

Role attributes are a particularly dangerous anti-pattern which can can break your production environment. Imagine this scenario. You have a `web_server` role with attributes for names and properties of two web sites you need to create on that server. Now imagine you need to split those web sites into two separate server roles, `app_server`, and `blog_server`. How do you test it in your dev environment without breaking prod? You can either roll the dice and make the change, hoping your don't break your prod environment or you have to try to override those attributes in environment files (first dev, then test, then staging, etc) and remember to clean them up after you've reached prod. Neither option is really workable.

You should use attributes in a wrapper cookbook instead.

## Pattern: Set custom attributes in a wrapper cookbook

Roles are not versioned. Cookkbooks are. By setting your custom attributes in a wrapper cookbook and pinning environments to specific cookbook versions, you can roll out attribute changes to dev, then test, then staging, then prod.

Ideally, you would have your environments automatically pinned by your CI server (e.g. Jenkins) as you test and promote cookbooks from dev to test to staging to prod, but that setup is beyond the scope of this blog post.

## Anti-Pattern: Setting a Server's Run List in a Role

Chef's Roles seem ideally suited to specifying the run list of all the recipes needed to build the server. However, this pattern suffers from the same problems as using Role Attributes above. If you add or remove recipes from the Role's `run_list`, that change affects all servers in the role, including Prod.

```ruby
name "web_server"
description "Role for web servers"

run_list(
  
  "recipe[base_server::disk_configuration]",
  "recipe[base_server::dhcp_reservation]",
  "recipe[base_server::pagefile]",
  "recipe[utility::install_tools]",
  "recipe[web_server::web_sites]",
  "recipe[base_server::ssl_certs]"

)

```

## Pattern: Setting a Server's Run List in a "role" or "application" cookbook's default recipe

Keep your Roles lightweight. The list of recipes needed to build your application or web server should be kept in an "application" cookbook instead of a Role. For example if you are building a prototypical web server, you Role should look like this:

```ruby
name "web_server"
description "Role for web servers"

run_list(
  "role[base]",
  "recipe[web_server]"
)
```

Your web_server "application" cookbook's default recipe should look like this.

```ruby
# web_server cookbook recipes/default.rb

  include_recipe "base_server::disk_configuration",
  include_recipe "base_server::dhcp_reservation",
  include_recipe "base_server::pagefile",
  include_recipe "utility::install_tools",
  include_recipe "web_server::web_sites",
  include_recipe "base_server::ssl_certs"

```

Again, this is because you can easily version cookbooks and test recipe and attribute changes to version 2.1.0 of your cookbook in your "Dev" environment while you keep your "Prod" environment pinned to v1.5.0.

## Wrap Up

By following the patterns above you will be save yourself major merging headaches by keeping your custom cookbook changes in wrapper cookbooks and be able to easily create targeted pull requests to share back with the community. By setting custom attributes in your wrapper cookbooks, you can roll changes from dev, to test, to prod by promoting an updated cookbook version through each environment. Finally, by keeping your server's run list in an "application" cookbook, you'll keep almost everything needed to build that server together in a versioned, testable cookbook.

I hope you'll be able to take away some good practices here. We've learned the hard way about each of the anti-patterns above and are still in the process of fully adopting the practices listed above. Feel free to comment about additional anti-patterns/patterns in the comments below.
