---
layout: post
title: "Creating an LWRP, part 1: The Resource"
date: 2012-12-31 11:00
comments: true
categories: chef, LWRP
---
Ohai Chefs!

If you've written any Chef recipes at all, you've almost certainly used Lightweight Resources and Providers (LWRPs). LWRPs enable you start/stop services, install packages, manage firewalls, deploy apps and many other common configuration tasks. [LWRPs] [1] combine a simple interface (Resource) with one or more usually OS-specific implementations (Providers). For example this resource installs Windows packages:

[1]: http://docs.opscode.com/essentials_cookbook_lwrp.html

``` ruby Windows Package Resource
windows_package "7-Zip 9.20 (x64 edition)" do
  source "http://downloads.sourceforge.net/sevenzip/7z920-x64.msi"
  action :install
end
```

The `windows_package` _provider_, which comes with the Windows cookbook, is 250 lines of code. It handles five different kinds of package types, e.g. msi, inno, nullsoft, etc. LWRPs make it very easy for sysadmins to write Chef recipes with a minimal amount of code because someone has already done the hard work of writing the Resource and Provider.

Even though Opscode has provided many LWRPs "out of the box", you will still need to write your own at some point. This week we'll look at how to write an LWRP starting with the Resource part. Next week, we'll complete the two-part series by learning how to write the corresponding Provider.

<!--more-->

## Example, please...

Your first step should be to determine if the resource you need already exists. Read and bookmark [this page](http://docs.opscode.com/essentials_cookbook_lwrp.html). It provides a good introduction to LWRPs and lists the Opscode provided LWRPs. Don't reinvent the wheel.

Here's an example resource we'll be looking at. It allows you to create Windows TCP/IP printer ports.

```ruby Windows Printer Port Resource
require 'resolv'

actions :create, :delete
default_action :create

attribute :ipv4_address, :name_attribute => true, :kind_of => String,
            :required => true, :regex => Resolv::IPv4::Regex

attribute :port_name       , :kind_of => String
attribute :port_number     , :kind_of => Fixnum, :default => 9100
attribute :port_description, :kind_of => String
attribute :snmp_enabled    , :kind_of => [ TrueClass, FalseClass ],
            :default => false

attribute :port_protocol, :kind_of => Fixnum, :default => 1, :equal_to => [1, 2]

attr_accessor :exists
```

Let's take it line by line. The first line requires the `resolv` library, "a thread-aware DNS resolver library written in Ruby". It provides a very good IPv4 regex we will use to verify the user has passed in a valid IPv4 address for the `:ipv4_address` attribute. It's easy to forget, but Resources are just Ruby and you can `require` libraries and use any other Ruby to help you out.

The next line specifies the allowed actions. Actions are what your resource can do, e.g. start, stop, create, delete, etc. In this case, you can `:create`, or `:delete` printer ports.

Line four defines the `default_action` for our resource, in this case `:create`. If you don't specify an action when you use the resource in a recipe, it will default to creating a printer port, which is what you probably want. A general philosophy of Chef is to define intelligent or "sane" defaults.

Lines 6 - 13 define attributes, or properties of the printer port resource we are creating. Let's look at each of these attributes in turn.

Line 6 defines an `:ipv4_address` attribute. Its `:name_attribute` is true, which means that this attribute will be set to the string between `windows_printer_port` and `do`:

```ruby `name_attribute`
windows_printer_port "This is the name attribute part" do
end

windows_printer_port '10.2.32.47' do
  # The :ipv4_address attribute will be set to '10.2.32.47'
end
```

In the second example above, the printer_port `:ipv4_address` attribute wll be set to '10.2.32.47'.

Also, on line 6, we are definining the `kind_of` validation parameter to tell the resource which kind of data we should expect (in this case, a string), whether this attribute is required (yes), and setting a validation regex (Resolv::IPv4::Regex). Instead of attempting to write a regex to validate IPv4 addresses, I am using a pre-defined regex supplied by the [`Resolv`] [2] Ruby library.

[2]: http://www.ruby-doc.org/stdlib-1.9.3/libdoc/resolv/rdoc/Resolv/IPv4.html

Line 8 defines a `port_name` attribute, which is an optional string with no default. Line 9 defines a `port_number` attribute, a Ruby Fixnum (i.e. an integer) with a default of 9100, which is the default when you create a printer port in Windows.

Line 10 defines a `port_description` attribute, an optional string.

Line 11 defines an `snmp_enabled` attribute, a boolean which defaults to false.

Line 13 defines a `port_protocol` attribute, a Ruby Fixnum, which defaults to 1. The `equal_to` constraint limits the possible values to 1 or 2.

It's important to note that the constraints and defaults in the `windows_printer_port` Resource are very carefully chosen based on knowlege of how the [Win32_TCPIPPrinterPort] [3] class in Windows works. You can't write a Resource and Provider unless you really understand the underlying resource you are modeling.

[3]: http://msdn.microsoft.com/en-us/library/windows/desktop/aa394492(v=vs.11).aspx

I'll explain the `attr_accessor :exists` in more detail next week, but in short, it defines an `exists` property on the Resource so we can test whether a given printer port already exists, so we don't create it again.


So that's it for this week. Tune in [next week] [4] for an overview of writing Providers.

[4]: http://dougireton.com/blog/2013/01/07/creating-an-lwrp-part-2/
