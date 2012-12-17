---
layout: post
title: "How to include the Windows Cookbook Helper methods in your Chef recipe"
date: 2012-12-16 20:06
comments: true
categories: 
---
Ohai Chefs!

We've been writing a lot of Windows Cookbooks and Recipes lately and it's been very helpful to be able to use the helper methods in the Opscode Windows cookbook. In this post I'll show you how to include and use those helper methods. You can generalize this to library methods from any cookbook, even your own cookbooks.

## What are the helper methods?
The Windows cookbook includes several nice [helper methods] [1] for dealing with windows paths and the registry. Unfortunately, it's not obvious how to use these helper methods in a recipe. At first I tried the standard Ruby `require` statement in a recipe, but this doesn't work. My Talented and Gifted &trade; teammate Kevin was able to explain why to me, but now I can't remember the reason.

## How do I use them in a recipe?
Just use `::Chef::Recipe.send(:include, Windows::Helper)` like this:

``` ruby ::Chef::Recipe.send(:include, Windows::Helper)
# include Windows::Helper from Opscode Windows Cookbook
::Chef::Recipe.send(:include, Windows::Helper)

# now you can call helper methods like win_friendly_path directly
my_batch_file = win_friendly_path(
                     ::File.join( node['cookbook']['batch_dir'],'foo.bat'))

execute "My batch file" do
  command my_batch_file
  creates "e:/logs/my_batch_file.log"
end

```

## What Else Can I Do?

You might also want to use [Chef::Mixin::Shellout] [2] helper methods, e.g. `shell_out!`.

``` ruby Chef::Mixin::Shellout
# include Chef::Mixin::Shellout
::Chef::Recipe.send(:include, Chef::Mixin::ShellOut)

# later in your code ...
output = shell_out! my_cmd

Chef::Log.debug "Output: #{ output.stdout }"
Chef::Log.debug "Errors: #{ output.stderr }" unless output.stderr.empty?
```

## Creating your own helper methods
You can also use this pattern in your own cookbooks. If you have common methods you find yourself using over and over, you should put them into a file in the `libraries` directory in your cookbook.

Create a helper.rb inside the cookbook/libraries folder:

``` ruby Creating your own helpers
module CookbookName
  module Helper

    def my_helper_method(param1, param2)
      # your code here
    end

  end
end
```

Then use your helper like this:

``` ruby How to use your helper

::Chef::Recipe.send(:include, CookbookName::Helper)

output = my_helper_method('foo', 'bar')
```

So, now you can use helper methods in your own cookbooks for fun and profit. Let me know in the comments if you are using this pattern other patterns for including cookbook helpers.

[1]: https://github.com/opscode-cookbooks/windows/tree/master/libraries
[2]: https://github.com/opscode/chef/blob/master/lib/chef/mixin/shell_out.rb
