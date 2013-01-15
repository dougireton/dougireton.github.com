---
layout: post
title: "Creating an LWRP - part 3"
date: 2013-01-13 11:38
comments: true
categories: chef, LWRP
---
Ohai Chefs!

Last week, we started looking at how to create the Provider part of the LWRP, the code which creates, deletes, or changes the Resource on the managed node. We looked at implementing Action methods (`:create`, `:delete`), using the `load_current_resource` method to read in attributes of existing resources, and how to support Chef's `why-run` mode.

This week, we'll complete the Provider by looking at the `port_exists?`, `create_printer_port`, and `delete_printer_port` methods. The `load_current_resource` method uses the `port_exists?` method to determine if the printer port already exists. The latter two methods leverage Windows PowerShell to create and delete printer ports respectively. Collectively, these are the private methods in this class, meant to be called only by the `:create` and `:delete` Action methods and `load_current_resource`.

<!--more-->

## [OOP] [1] ([there it is] [2])...

We've moved some of the code which might otherwise go into our `:create` and `:delete` Action methods into separate methods which create and delete printer ports using PowerShell. Breaking up our methods into smaller, logical chunks follows the Composed Method technique which I first learned about in [Eloquent Ruby] [3] by Russ Olsen.

[1]: http://en.wikipedia.org/wiki/Object-oriented_programming
[2]: http://en.wikipedia.org/wiki/Whoomp!_(There_It_Is)
[3]: http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional/dp/0321584104

{% blockquote %}
"The composed method technique advocates dividing your class up into methods that have three characteristics. First, each method should do a single thing—focus on solving a single aspect of the problem. By concentrating on one thing, your methods are not only easier to write, they are also easier to understand. 

Second, each method needs to operate at a single conceptual level: Simply put, don’t mix high-level logic with the nitty-gritty details. A method that implements the business logic around, say, currency conversions, should not suddenly veer off into the details of how the various accounts are stored in a database. 

Finally, each method needs to have a name that reflects its purpose."

{% endblockquote %}

Olsen, Russ (2011-02-07). _Eloquent Ruby_ (Addison-Wesley Professional Ruby Series) (Kindle Locations 2102-2107). Pearson Education (USA). Kindle Edition. 

## The `:create` action

Let's look at the `:create` Action to see how we applied the Composed Method technique.

``` ruby
action :create do
  if @current_resource.exists
    Chef::Log.info "#{ @new_resource } already exists - nothing to do."
  else
    converge_by("Create #{ @new_resource }") do
      create_printer_port
    end
  end
end

```

Even without comments it should be pretty clear what this method does; it almost reads like plain English. If the printer port already exists, it logs a message and does nothing, otherwise it creates a printer port. We have made it so readable by extracting out the code which checks for a pre-existing printer port into the `load_current_resource` method, and the code which creates the printer port into the `create_printer_port` method.

By composing our `:create` method this way, it's become a template. You should be able to use the code above for almost any `:create` Action in any LWRP you will write, just by changing the name of the `create_printer_port` method to something more suitable.

Now, let's look at the private `create_printer_port` method, which actually creates the printer port.

## `create_printer_port` method

``` ruby
def create_printer_port

  port_name = new_resource.port_name || "IP_#{ new_resource.ipv4_address }"

  # create the printer port using PowerShell
  powershell "Creating printer port #{ new_resource.port_name }" do
    code <<-EOH

      Set-WmiInstance -class Win32_TCPIPPrinterPort `
        -EnableAllPrivileges `
        -Argument @{ HostAddress = "#{ new_resource.ipv4_address }";
                     Name        = "#{ port_name }";
                     Description = "#{ new_resource.port_description }";
                     PortNumber  = "#{ new_resource.port_number }";
                     Protocol    = "#{ new_resource.port_protocol }";
                     SNMPEnabled = "$#{ new_resource.snmp_enabled }";
                  }
    EOH
  end
end
```

The `create_printer_port` method starts off by setting a local variable, `port_name` which is set to the `new_resource.port_name` if the user set the `port_name` attribute, or `IP_<ipv4_address>` if the user didn't set `port_name`.

After that, it's just a straightforward PowerShell resource block which creates the printer port using the attributes passed in from the `windows_printer_port` resource in the Recipe.

One thing to note is how we are using [Ruby String Interpolation] [4] to pass our Resource Attributes in to the PowerShell script.The `"#{ new_resource.foo }"` sections are how we can pass Ruby variables into a PowerShell or batch script. Pretty handy.

[4]: http://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Literals#Interpolation

## `port_exists?` method

`port_exists?` is a simple method which queries the Windows Registry to see if the printer port has already been created. The `load_current_resource` method calls `port_exists?`, and if the printer port exists, it sets `@current_resource.exists` to `true`.

One thing you should notice is that the question mark is part of the `port_exists?` method name. In Ruby, it is perfectly acceptable, and expected that methods which return _true_ or _false_ have a `?` appended to the method name.

``` ruby
PORTS_REG_KEY = 'HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Print\Monitors\Standard TCP/IP Port\Ports\\'

def port_exists?(name)
  port_reg_key = PORTS_REG_KEY + name

  Chef::Log.debug "Checking to see if this reg key exists: '#{ port_reg_key }'"
  Registry.key_exists?(port_reg_key)
end
```

This code block creates a Ruby [Constant] [5], the `PORTS_REG_KEY`, which is the key we'll query to determine if the printer port already exists.

In our `port_exists?` method, we call the Windows Cookbook [`Registry.key_exists?`] [6] method which returns `true` or `false`, telling us whether the printer port is already in the Windows Registry or not. Notice that `?` in the method name again?

[5]: http://rubylearning.com/satishtalim/ruby_constants.html
[6]: https://github.com/opscode-cookbooks/windows#library-methods 

## `delete_printer_port` method

The `delete_printer_port` is much the same as the `create_printer_port` method so I won't cover it here.

## Wrap-up

So now we've completed our look at how to create LWRPs. We've covered both the Reource and Provider and looked at structuring your Ruby methods using the Composed Methods pattern. Certainly, my printer port LWRP isn't perfect. In writing these blog posts, I've already come up with some changes to make it better, but the biggest glaring omission is the complete lack of test coverage!

## Next Steps

We'll probably take a break from LWRPs next week but look for a blog post in the near future on testing LWRPs using my absolute favorite Ruby testing libarary, [RSpec] [7]. Huge shout out to [Joshua Timberman] [8] to whom I'm indebted for great example [RSpec tests in the Runit] [9] cookbook.

[7]: http://rspec.info/
[8]: https://twitter.com/jtimberman
[9]: https://github.com/opscode-cookbooks/runit/commits/CHEF-154
