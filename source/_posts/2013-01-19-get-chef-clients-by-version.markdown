---
layout: post
title: "Get Chef Clients by version"
date: 2013-01-19 20:19
comments: true
categories: chef, metrics
---
Ohai Chefs!

At work, we're being converted to the gospel of Etsy's [Church of Graphs] [1]. We're sending Chef run times and other metrics to a combination of [StatsD] [2], [Graphite] [3], and [Team Dashboard] [4]. Last week, I wanted to add a graph of chef clients by version. In other words, I wanted to see how many Chef 0.10.8, and 10.12 clients we have left to upgrade. 

[1]: http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/
[2]: https://github.com/etsy/statsd/
[3]: http://graphite.wikidot.com/
[4]: http://fdietz.github.com/team_dashboard/

This week, we'll see how to get Chef client versions from the Chef Server, including one way which turned out to be more than <strong>30 times faster</strong> in my tests.

<!--more-->

## The First Attempt

I needed to get a count of Chef clients grouped by version. I envisioned ending up with a hash like this:

``` ruby
  {
    '10.12.0' => 112,
    '10.16.2' => 534,
    '10.18.2' => 1
  }
```

My first thought was to do this:

``` bash 
$ knife node list | xargs -I {} knife node show {} -a chef_packages.chef.version -Fj
```

This pipes a list of all nodes in an org to `knife node show` and returns `chef_packages.chef.version` in JSON format. 

This works, but it takes a __loooong__ time, nearly 40 __minutes__ on my quad-core Macbook Pro against our Private Chef server to get the Chef client version for 908 nodes, or ~2.4 seconds per node.

``` bash
knife node list
	1.82s user 0.36s system 75% cpu 2.878 total

xargs -I {} knife node show {} -a chef_packages.chef.version
	1771.53s user 278.24s system 85% cpu ** 39:46.55 total **
```

This takes so long because `knife node show` makes a round-trip to the Chef server for each node. We need to speed this up, preferably by an order of magnitude.

## The Second Attempt

What if we asked the Chef server to get Chef Client version info for every node in the org and send it to us in one batch?

```bash
knife search node 'name:*' -a chef_packages.chef.version –Fj
```

This approach is much more efficient; just over a minute instead of 40 minutes:
```bash
knife search node 'name:*' -a chef_packages.chef.version –Fj
	40.72s user 1.54s system 57% cpu ** 1:13.81 total **
```

The results from the knife search command look like this; easily parsable JSON. 

```javascript
{
  "results": 3,
  "rows": [
    {
      "chef_packages.chef.version": "10.16.2",
      "id": "webserver01.example.com"
    },
    {
      "chef_packages.chef.version": "10.16.2",
      "id": "webserver02.example.com"
    },
    {
      "chef_packages.chef.version": "10.12.0",
      "id": "db01.example.com"
    }
  ]
}
```

## Ohai Spelunking

But, hold on a second, how did I know the Chef client version attribute is named `chef_packages.chef.version`? I didn't, but here's how I found it:

``` bash
knife node show myserver01.example.com -l | grep -C 5 10.16.2
```

I knew that `myserver01.example.com` was running Chef Client 10.16.2. I did a `knife node show` with the `-l` option to show **all** Ohai attributes and grep'd for `10.16.2` with five lines of context above and below (`-C 5`).

Here's the result of that whole command:

``` bash
Automatic Attributes (Ohai Data):
chef_packages:    
  chef: 
    chef_root:  C:/opscode/chef/embedded/lib/ruby/gems/1.9.1/gems/chef-10.16.2/lib
    version:    10.16.2
  ohai: 
    ohai_root:  C:/opscode/chef/embedded/lib/ruby/gems/1.9.1/gems/ohai-6.14.0/lib/ohai
    version:    6.14.0
command:           {}
counters:
```

From the output above, I can walk down the `chef_packages` attribute to determine the attribute I'm looking for is `chef_packages.chef.version`.

## Parsing the Results

So, now that we have the raw JSON data, how can we turn it into this?

``` ruby
  {
    '10.12.0' => 112,
    '10.16.2' => 534,
    '10.18.2' => 1
  }
```

Let's take a look at a script to parse the JSON list of nodes into a nice "grouped-by" version hash:

```ruby
require 'json'

KNIFE_RB = '.chef/knife.rb'
NODE_LIST = `knife search node -c #{ KNIFE_RB } 'name:*' -a chef_packages.chef.version --format json 2>&1` 

def get_chef_clients_by_version(nodes)

  # turn JSON into Ruby objects
  nodes_json = JSON.parse nodes

  # create an array of all chef client versions
  client_versions = nodes_json['rows'].map { |item| item['chef_packages.chef.version'] }
  
  # initialize an empty hash to store our final counts grouped by version
  number_of_clients_by_version = Hash.new(0)

  # For each item in the client_versions array, create a unique key in our
  # number_of_clients_by_version hash and increment our counter
  client_versions.each { |version| number_of_clients_by_version[version] += 1 }

  number_of_clients_by_version
end

puts get_chef_clients_by_version(NODE_LIST)
```

Let's take it line by line. On line one we're requiring [JSON][5]. On line four we're executing the `knife search` command. Lines 6 - 21 are a function to parse the node data into our final count of versions.

[5]: http://www.ruby-doc.org/stdlib-1.9.3/libdoc/json/rdoc/JSON.html

## The `get_chef_clients_by_version` method

This method is where all the exciting stuff happens. On line 9, we're parsing the JSON data and creating a Ruby data structure which looks like this:

```ruby
{"results"=>3,
 "rows"=>
  [{"chef_packages.chef.version"=>"10.16.2", "id"=>"webserver01.example.com"},
   {"chef_packages.chef.version"=>"10.16.2", "id"=>"webserver02.example.com"},
   {"chef_packages.chef.version"=>"10.16.2", "id"=>"db01.example.com"},
  ]
}
```

Line 12 is my favorite line of the method. 

```ruby 
  # create an array of all chef client versions
  client_versions = nodes_json['rows'].map { |item| item['chef_packages.chef.version'] }
```

It uses Ruby's super useful [`map`][6] method to create an simple array of versions from the `rows` array of two-element hashes. The result of the `map` looks like this:

```ruby
["10.16.2", "10.16.2", "10.12.0"]
```

From there we create and return the `number_of_clients_by_version` hash to hold our results and iterate over each item in the `client_versions` array, counting the nodes by version.

```ruby
# initialize an empty hash to store our final counts grouped by version
  number_of_clients_by_version = Hash.new(0)

  # For each item in the client_versions array, create a unique key in our
  # number_of_clients_by_version hash and increment our counter
  client_versions.each { |version| number_of_clients_by_version[version] += 1 }
```

Thanks to my Talented and Gifted&trade; co-worker [Kevin][7] for [this StackOverflow link][8] which explained how to do the group-by version.


[6]: http://www.ruby-doc.org/core-1.9.3/Enumerable.html#method-i-map
[7]: https://twitter.com/moserke
[8]: http://stackoverflow.com/questions/569694/count-duplicate-elements-in-ruby-array

So here's the result of the script, which is exactly what we set out to accomplish:

```ruby
  {
    '10.12.0' => 112,
    '10.16.2' => 534,
    '10.18.2' => 1
  }
```

## [Fade to Black][9]
So there you have it. We compared two approaches to returning data from the Chef server, with one being an order of magnitude faster. We figured out how to find specific Ohai attribute names, and we created a script to transform the raw data to something truly useful.

These posts seem to keep getting longer, so maybe next week, we'll have something short and sweet. Thanks for reading and I'd love to hear your comments.

[9]: http://www.youtube.com/watch?v=WEQnzs8wl6E
