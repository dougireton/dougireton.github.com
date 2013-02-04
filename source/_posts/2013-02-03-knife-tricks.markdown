---
layout: post
title: "knife tricks"
date: 2013-02-03 20:40
comments: true
categories: chef, knife, howto
---
Ohai Chefs!

At work, we have yet to use search extensively in our Chef Recipes, but we do a fair number of ad hoc knife searches. The following are some knife tricks and tips we've picked up over the last few months. Most of the credit for these goes to my esteemed co-worker, the Impossibly Hip&trade; [Jon Decamp][1].

[1]: https://twitter.com/jondecamp

### Find all nodes in an environment

```bash
$ knife search node chef_environment:<environment name>
```

### Find all nodes which contain a role

```bash
$ knife search node role(s):<role name>

$ knife search node "role:web_server" -a hostname
```
Use 'roles' plural when looking in the expanded run list.

<!--more-->

### Find all nodes which contain a recipe

```bash
# looks for statsd_handler(::default)
$ knife search node "recipes:statsd_handler"

# note the use of \:\: to escape the double-colon
$ knife search node "recipes:windows\:\:reboot_handler"
```
Note the use of `recipes` plural to search the expanded run list.

### Find all non-64 bit nodes
```bash
$ knife search node "(NOT kernel_machine:x86_64)"
```

### Return selected attributes from knife search

```bash
$ knife search node "name:*" -a chef_packages.chef.version
```
This returns the `chef_packages.chef.version` attribute from all nodes in the Chef Org.

### Add a role to all nodes in an Environment

```bash
# First, run it like this to run without saving the results back to the Chef Server
$ knife exec -E 'nodes.transform("chef_environment:dev") {|n| puts n.run_list << "role[hosts_file]" }'

$ knife exec -E 'nodes.transform("chef_environment:dev") {|n| puts n.run_list << "role[hosts_file]"; n.save }'
```
To be on the safe side, run the command above without the `n.save` so results aren't saved back to the Chef server. When you are sure about the command run it with `n.save` to save the results back to the Chef server.

### Add a role to all nodes in an Environment which don't contain the given Role

```bash
$ knife exec -E 'nodes.find("chef_environment:dev") {|n| puts n.run_list << "role[base]" unless n.run_list.include?("role[base]"); n.save }'
```

### Remove a recipe from all nodes in an Environment

```bash
$ knife exec -E 'nodes.transform("chef_environment:dev") {|n| puts n.run_list.remove("recipe[chef-client::upgrade]"); n.save }'
```

### Remove all nodes from a given role
```bash
$ knife exec -E 'nodes.find("role:web_server") {|n| n.run_list.remove("role[web_server]"); n.save}'
```

### Set a node's run_list back to a single item

```bash
$ knife exec -E 'nodes.transform("name:webserver01.example.com") {|n| n.run_list(["role[base]"])}'
```

## Conclusion
So that's it for this week. I hope you picked up some valuable knife tricks. The knife search command is versatile and combining `knife exec` with search allows you to do some amazing things with your infrstructure. Share some of your own knife tricks in the comments or over on Twitter.
