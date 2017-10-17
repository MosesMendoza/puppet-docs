---
layout: default
built_from_commit: ab595327c42b4fbafdd669d8a0208ce081c03133
title: Metaparameter Reference
toc: columns
canonical: "/puppet/latest/metaparameter.html"
---





**This page is autogenerated; any changes will get overwritten** *(last generated on 2017-10-04 17:16:41 -0700)*



Metaparameters are attributes that work with any resource type, including custom
types and defined types.

In general, they affect _Puppet's_ behavior rather than the desired state of the
resource. Metaparameters do things like add metadata to a resource (`alias`,
`tag`), set limits on when the resource should be synced (`require`, `schedule`,
etc.), prevent Puppet from making changes (`noop`), and change logging verbosity
(`loglevel`).

## Available Metaparameters

### alias

Creates an alias for the resource.  Puppet uses this internally when you
provide a symbolic title and an explicit namevar value:

    file { 'sshdconfig':
      path => $operatingsystem ? {
        solaris => '/usr/local/etc/ssh/sshd_config',
        default => '/etc/ssh/sshd_config',
      },
      source => '...'
    }

    service { 'sshd':
      subscribe => File['sshdconfig'],
    }

When you use this feature, the parser sets `sshdconfig` as the title,
and the library sets that as an alias for the file so the dependency
lookup in `Service['sshd']` works.  You can use this metaparameter yourself,
but note that aliases generally only work for creating relationships; anything
else that refers to an existing resource (such as amending or overriding
resource attributes in an inherited class) must use the resource's exact
title. For example, the following code will not work:

    file { '/etc/ssh/sshd_config':
      owner => root,
      group => root,
      alias => 'sshdconfig',
    }

    File['sshdconfig'] {
      mode => '0644',
    }

There's no way here for the Puppet parser to know that these two stanzas
should be affecting the same file.

### audit

(This metaparameter is deprecated and will be ignored in a future release.)

Marks a subset of this resource's unmanaged attributes for auditing. Accepts an
attribute name, an array of attribute names, or `all`.

Auditing a resource attribute has two effects: First, whenever a catalog
is applied with puppet apply or puppet agent, Puppet will check whether
that attribute of the resource has been modified, comparing its current
value to the previous run; any change will be logged alongside any actions
performed by Puppet while applying the catalog.

Secondly, marking a resource attribute for auditing will include that
attribute in inspection reports generated by puppet inspect; see the
puppet inspect documentation for more details.

Managed attributes for a resource can also be audited, but note that
changes made by Puppet will be logged as additional modifications. (I.e.
if a user manually edits a file whose contents are audited and managed,
puppet agent's next two runs will both log an audit notice: the first run
will log the user's edit and then revert the file to the desired state,
and the second run will log the edit made by Puppet.)

### before

One or more resources that depend on this resource, expressed as
[resource references](https://docs.puppetlabs.com/puppet/latest/lang_data_resource_reference.html).
Multiple resources can be specified as an array of references. When this
attribute is present:

* This resource will be applied _before_ the dependent resource(s).

This is one of the four relationship metaparameters, along with
`require`, `notify`, and `subscribe`. For more context, including the
alternate chaining arrow (`->` and `~>`) syntax, see
[the language page on relationships](https://docs.puppetlabs.com/puppet/latest/lang_relationships.html).

### consume

Consume a capability resource.

The value of this parameter must be a reference to a capability resource,
or an array of such references. Each capability resource referenced here
must have been exported by another resource in the same environment.

The referenced capability resource(s) will be looked up, added to the
current node catalog, and processed following the underlying consumes
clause.

It is an error if this metaparameter references resources whose type is not
a capability type, or of there is no consumes clause for the type of the
current resource and the capability resource mentioned in this parameter.

For example:

define web(..) { .. }
Web consumes Sql { .. }
web { server:
  consume => Sql[my_db]
}

### export

Export a capability resource.

The value of this parameter must be a reference to a capability resource,
or an array of such references. Each capability resource referenced here
will be instantiated in the node catalog and exported to consumers of this
resource. The title of the capability resource will be the title given in
the reference, and all other attributes of the resource will be filled
according to the corresponding produces statement.

It is an error if this metaparameter references resources whose type is not
a capability type, or of there is no produces clause for the type of the
current resource and the capability resource mentioned in this parameter.

For example:

define web(..) { .. }
Web produces Http { .. }
web { server:
  export => Http[main_server]
}

### loglevel

Sets the level that information will be logged.
The log levels have the biggest impact when logs are sent to
syslog (which is currently the default).

The order of the log levels, in decreasing priority, is:

* `emerg`
* `alert`
* `crit`
* `err`
* `warning`
* `notice`
* `info` / `verbose`
* `debug`

Valid values are `debug`, `info`, `notice`, `warning`, `err`, `alert`, `emerg`, `crit`, `verbose`.

### noop

Whether to apply this resource in noop mode.

When applying a resource in noop mode, Puppet will check whether it is in sync,
like it does when running normally. However, if a resource attribute is not in
the desired state (as declared in the catalog), Puppet will take no
action, and will instead report the changes it _would_ have made. These
simulated changes will appear in the report sent to the puppet master, or
be shown on the console if running puppet agent or puppet apply in the
foreground. The simulated changes will not send refresh events to any
subscribing or notified resources, although Puppet will log that a refresh
event _would_ have been sent.

**Important note:**
[The `noop` setting](https://docs.puppetlabs.com/puppet/latest/configuration.html#noop)
allows you to globally enable or disable noop mode, but it will _not_ override
the `noop` metaparameter on individual resources. That is, the value of the
global `noop` setting will _only_ affect resources that do not have an explicit
value set for their `noop` attribute.

Valid values are `true`, `false`.

### notify

One or more resources that depend on this resource, expressed as
[resource references](https://docs.puppetlabs.com/puppet/latest/lang_data_resource_reference.html).
Multiple resources can be specified as an array of references. When this
attribute is present:

* This resource will be applied _before_ the notified resource(s).
* If Puppet makes changes to this resource, it will cause all of the
  notified resources to _refresh._ (Refresh behavior varies by resource
  type: services will restart, mounts will unmount and re-mount, etc. Not
  all types can refresh.)

This is one of the four relationship metaparameters, along with
`before`, `require`, and `subscribe`. For more context, including the
alternate chaining arrow (`->` and `~>`) syntax, see
[the language page on relationships](https://docs.puppetlabs.com/puppet/latest/lang_relationships.html).

### require

One or more resources that this resource depends on, expressed as
[resource references](https://docs.puppetlabs.com/puppet/latest/lang_data_resource_reference.html).
Multiple resources can be specified as an array of references. When this
attribute is present:

* The required resource(s) will be applied **before** this resource.

This is one of the four relationship metaparameters, along with
`before`, `notify`, and `subscribe`. For more context, including the
alternate chaining arrow (`->` and `~>`) syntax, see
[the language page on relationships](https://docs.puppetlabs.com/puppet/latest/lang_relationships.html).

### schedule

A schedule to govern when Puppet is allowed to manage this resource.
The value of this metaparameter must be the `name` of a `schedule`
resource. This means you must declare a schedule resource, then
refer to it by name; see
[the docs for the `schedule` type](https://docs.puppetlabs.com/puppet/latest/type.html#schedule)
for more info.

    schedule { 'everyday':
      period => daily,
      range  => "2-4"
    }

    exec { "/usr/bin/apt-get update":
      schedule => 'everyday'
    }

Note that you can declare the schedule resource anywhere in your
manifests, as long as it ends up in the final compiled catalog.

### stage

Which run stage this class should reside in.

**Note: This metaparameter can only be used on classes,** and only when
declaring them with the resource-like syntax. It cannot be used on normal
resources or on classes declared with `include`.

By default, all classes are declared in the `main` stage. To assign a class
to a different stage, you must:

* Declare the new stage as a [`stage` resource](https://docs.puppetlabs.com/puppet/latest/type.html#stage).
* Declare an order relationship between the new stage and the `main` stage.
* Use the resource-like syntax to declare the class, and set the `stage`
  metaparameter to the name of the desired stage.

For example:

    stage { 'pre':
      before => Stage['main'],
    }

    class { 'apt-updates':
      stage => 'pre',
    }

### subscribe

One or more resources that this resource depends on, expressed as
[resource references](https://docs.puppetlabs.com/puppet/latest/lang_data_resource_reference.html).
Multiple resources can be specified as an array of references. When this
attribute is present:

* The subscribed resource(s) will be applied _before_ this resource.
* If Puppet makes changes to any of the subscribed resources, it will cause
  this resource to _refresh._ (Refresh behavior varies by resource
  type: services will restart, mounts will unmount and re-mount, etc. Not
  all types can refresh.)

This is one of the four relationship metaparameters, along with
`before`, `require`, and `notify`. For more context, including the
alternate chaining arrow (`->` and `~>`) syntax, see
[the language page on relationships](https://docs.puppetlabs.com/puppet/latest/lang_relationships.html).

### tag

Add the specified tags to the associated resource.  While all resources
are automatically tagged with as much information as possible
(e.g., each class and definition containing the resource), it can
be useful to add your own tags to a given resource.

Multiple tags can be specified as an array:

    file {'/etc/hosts':
      ensure => file,
      source => 'puppet:///modules/site/hosts',
      mode   => '0644',
      tag    => ['bootstrap', 'minimumrun', 'mediumrun'],
    }

Tags are useful for things like applying a subset of a host's configuration
with [the `tags` setting](/puppet/latest/configuration.html#tags)
(e.g. `puppet agent --test --tags bootstrap`).



----------------

*This page autogenerated on 2017-10-04 17:16:41 -0700*