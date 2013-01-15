---
layout: post
title: "Creating an LWRP, part 2: The Provider"
date: 2013-01-07 14:35
comments: true
categories: chef, LWRP
---
Ohai Chefs!

Last week, we looked at part one of creating an LWRP - the Resource. This week, we'll look at part two - the Provider. We'll look at a real Provider which uses Ruby and PowerShell to create and delete printer ports. Since the Provider code is so long, I'll cover the first half this week, and the second half next week. The first half will cover the `:create` and `:delete` Action methods, how to support `why_run` (dry-run or what-if mode) and how to use the `load_current_resource` method.  

<!--more-->

As a reminder, LWRPs eanble you to easily install, create, delete, start, stop or otherwise manipulate resources; things like packages, printers, services, etc. The Resource is a simple interface, an API if you will, which makes it very easy for sysadmins to create Recipes which do a lot of work in a few lines of code.

## The Provider

The Provider part of an LWRP is the OS-specific code which actually installs, creates, deletes, starts, or stops the resource on the managed node. As we'll see in the example below, Providers are written in Ruby but often use Bash, PowerShell, or command-line utilities to do their work.

## Or maybe _Providers_...

In an LWRP, a given Resource may have more than one Provider. For example the [`windows_feature`] [1] LWRP in the Windows cookbook has two Providers, one for installing features via [`dism.exe`] [2], and one for installing features using the older [`servermanagercmd.exe`] [3].

[1]: https://github.com/opscode-cookbooks/windows#windows_feature
[2]: http://msdn.microsoft.com/en-us/library/dd371719(v=vs.85).aspx
[3]: http://technet.microsoft.com/en-us/library/ee344834(v=ws.10).aspx

## Show me the code!

Continuing our example from last week we'll be looking at the [Windows Printer Port LWRP] [4] .

[4]: https://github.com/opscode-cookbooks/windows#windows_printer_port

```ruby Windows Printer Port Provider
# Support whyrun
def whyrun_supported?
  true
end

action :create do
  if @current_resource.exists
    Chef::Log.info "#{ @new_resource } already exists - nothing to do."
  else
    converge_by("Create #{ @new_resource }") do
      create_printer_port
    end
  end
end

action :delete do
  if @current_resource.exists
    converge_by("Delete #{ @new_resource }") do
      delete_printer_port
    end
  else
    Chef::Log.info "#{ @current_resource } doesn't exist - can't delete."
  end
end

def load_current_resource
  @current_resource = Chef::Resource::WindowsPrinterPort.new(@new_resource.name)
  @current_resource.name(@new_resource.name)
  @current_resource.ipv4_address(@new_resource.ipv4_address)
  @current_resource.port_name(@new_resource.port_name || "IP_#{ @new_resource.ipv4_address }")

  if port_exists?(@current_resource.port_name)
    # TODO: Set @current_resource port properties from registry
    @current_resource.exists = true
  end
end

# -- SNIP --
```

## The `:create` action

Let's look at the `:create` action first. We first check if the `@current_resource` already exists, and if so, we log a message and do nothing. `@current_resouce` is set to the resource on the managed node if it already exists. So if the printer port we are trying to create already exists, we don't create it again. This is how we acheive idempotency in our LWRP and it's a core tenet of Chef - don't change a node's state unless it's necessary.

So if our printer port hasn't yet been created, we call the `create_printer_port` method which actually creates the printer port using Windows a PowerShell cmdlet. We'll look at the `create_printer_port` method next week. The `create_printer_port` method call is wrapped in a `converge_by` block, which is the secret to implementing [`why-run`] [5] mode.

[5]: http://lists.opscode.com/sympa/arc/chef/2012-07/msg00025.html

## Why-Run

Why-Run is fairly simple to implement in a Provider. You just need to define a `whyrun_supported?` method which returns `true`, and wrap any code which actually makes changes on the managed node in a `converge_by` block with an appropriate message about what the code would do if you actually converged the node. For example, in our `:create` action, we wrap the `create_printer_port` method call in a `converge_by` block with a log message which says we would have created a printer port.

If you've looked at Provider code in the past, or have written LWRPs, you have probably seen the `new_resource.updated_by_last_action(true)` method call in the Provider Actions. This method call supports Notifications. So if the Resource changed, it would [notify other resources] [6].

When you implement Why-Run, you don't need to call `new_resource.updated_by_last_action(true)` because the `converge_by` block does that for you automatically.

[6]: http://wiki.opscode.com/pages/viewpage.action?pageId=7274964#LightweightResourcesandProviders(LWRP)-Keyword:action

## The `load_current_resource` method

The `load_current_resource` method is proably the hardest to understand how to actually write. Conceptually, it's fairly straighforward. Using the Resource (`windows_printer_port`) attributes which the user specified in the Recipe, `load_current_resource` tries to find, on the server, an existing printer port which matches the one we are trying to create. If it finds a match, it sets `@current_resource.exists` to `true`. Remember that [last week] [7] we created the `exists` attribute by setting an `attr_accessor :exists` on our Resource. Now, we get to use it.

You should know that the `load_current_resource` method is already defined on the `Chef::Provider` class. You just need to define, or [_override_] [8] the method in your own Provider. Chef will call the `load_current_resouce` method automatically when it [iterates over the ResourceCollection during the chef client execution phase] [9].

[7]: http://dougireton.com/blog/2012/12/31/creating-an-lwrp/
[8]: http://www.rubydoc.info/github/opscode/chef/master/Chef/Provider#load_current_resource-instance_method
[9]: http://wiki.opscode.com/pages/viewpage.action?pageId=7274964#LightweightResourcesandProviders(LWRP)-Background

## Just Gettin' By...

We are just doing the bare minimum in our `load_current_resource` method. For creating and deleting printer ports, this is probably enough. If we wanted to be able to _modify_ a printer port, we would need to load in all the attributes from the current printer port on the managed node so we would have them available for comparison.

For example, if we wanted to modify an existing printer port to change the `snmp_enabled` attribute from `false` to `true`, we would need to query the existing printer port on the server to see if SNMP was enabled or not, and save that value to `@current_resource.snmp_enabled` for use later in our `:modify` action.

``` ruby load_current_resource
def load_current_resource
  @current_resource = Chef::Resource::WindowsPrinterPort.new(@new_resource.name)
  @current_resource.name(@new_resource.name)
  @current_resource.ipv4_address(@new_resource.ipv4_address)
  @current_resource.port_name(@new_resource.port_name || "IP_#{ @new_resource.ipv4_address }")

  if port_exists?(@current_resource.port_name)
    # TODO: Set @current_resource port properties from registry
    @current_resource.exists = true
  end
end
```

## `load_current_resource` nuts and bolts

So for our bare minimum `load_current_resource` method, we need to set `@current_resource` to an instance of `Chef::Resource::WindowsPrinterPort` and copy one or more attributes from the `@new_resource`, which is passed in from the `windows_printer_port` Resource in the Recipe. Chef  creates the `@new_resource` class instance from the attributes in the Recipe and makes it available to the Provider automatically.

In this case, to determine if the printer port already exists, we need to query the Windows Registry using the `port_name` attribute. The `port_name` is usually `IP_<ipv4_address>`, but could could be anthing if the user specified a custom `port_name` in the Recipe.

So in line 5 above, we set `@current_resouce.port_name` from `@new_resource.port_name` if the user specified a custom port name, or we use `IP_<ipv4_address>` if the user didn't specify a custom port name. 

## `port_exists?`

We then call our `port_exists?` method which queries the Windows Registry and returns `true` if the port already exists or `false` if it doesn't. We have a `# TODO` note in our code where we would load in additional printer port attributes from the registry in the future. 

Finally, we set our `@current_resource.exists` attribute to `true` since we now know that the printer port already exists.

## Summary

This week we learned how to create the basic skeleton for Action methods (`:create`, `:delete`, etc.), how to support `why-run` mode, and how to use the `load_current_resource` method to determine if the Resource we are trying to create already exsists on the managed node.

Next week, we'll cover the `create_printer_port`, and `port_exists?` private methods which do the real work on the server.

## Feedback
Do you have any good examples of Providers which do something especially cool? Maybe from an LWRP you wrote? Or have you found Providers challenging to write? I'd love to hear your feedback in the comments. Thanks!
