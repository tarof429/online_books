# The Puppet Language (part 1)

## More Resources

The package resource is used to install packages on a system. For example, the following ensures that ntp is installed. Be sure to read the docs on the various options.

```
package { 'ntp':
  ensure => installed
}
```

There's also the following resources:

- service
- notify
- exec
- file


## File Serving

The following example creates a module to set motd, or message of the day.

In /etc/puppetlabs/code/environments/production/modules, create a directory called `motd`. Beneath this directory, create two subdirectories: `manifests` and `files`. In files, create a file called motd.txt with the content `Welcome to Puppet`.

Next we go to the manifests directory and create the base class for our module in the file `init.pp`. 

```
[root@puppet manifests]# cat init.pp
class motd {
  file { '/etc/motd':
    ensure => file,
    source => 'puppet:///modules/motd/motd.txt',
    owner  => 'root',
    group  => 'root',
    mode   => '0644',
  }
}
```

Then we need to edit `/etc/puppetlabs/code/environments/production/manifests/site.pp` so that it has the following content:

```
[root@puppet manifests]# cat site.pp
node "agent.localdomain" {
  include motd
}
```

The following output shows the initial content of /etc/motd (empty) and the final output.

```
[root@agent vagrant]# cat /etc/motd
[root@agent vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606441610'
Notice: /Stage[main]/Motd/File[/etc/motd]/content:
--- /etc/motd	2013-06-07 14:31:32.000000000 +0000
+++ /tmp/puppet-file20201127-4883-xcdprl	2020-11-27 01:46:50.878760926 +0000
@@ -0,0 +1,3 @@
+
+Welcome to Puppet
+

Info: Computing checksum on file /etc/motd
Info: /Stage[main]/Motd/File[/etc/motd]: Filebucketed /etc/motd to puppet with sum d41d8cd98f00b204e9800998ecf8427e
Notice: /Stage[main]/Motd/File[/etc/motd]/content: content changed '{md5}d41d8cd98f00b204e9800998ecf8427e' to '{md5}2e3fa077de611f6f023f306ab44129a7'
Notice: Applied catalog in 0.10 seconds
[root@agent vagrant]# cat /etc/motd

Welcome to Puppet

[root@agent vagrant]#
```

## Relationships

### etcd
Very often, resources have dependencies. For example, a running `etcd` service depends on the fac that `etcd` package is installed. We do this by using the `require` keyword in the resource. In order to do this, we need to use resource references, which in this case is `Package['etcd']`.

```
// etcd
package { 'etcd':
  ensure => installed,
}

service { 'etcd':
  ensure  => running,
  require => Package['etcd'],
}

[root@agent vagrant]# puppet apply etcd.pp
Notice: Compiled catalog for agent.localdomain in environment production in 0.78 seconds
Notice: /Stage[main]/Main/Package[etcd]/ensure: created
Notice: /Stage[main]/Main/Service[etcd]/ensure: ensure changed 'stopped' to 'running'
Notice: Applied catalog in 24.51 seconds
```

Another keyword that expresses relationships is `before`. It behaves the exact opposite as `require`. For example, you could say that the `etcd` package needs to be installed `before` the service can be set to running.

Other keywords used to express relationships are: `subscribe`, `notify`, `refreshonly`.

An alternative syntax to express relationships is `relationship chaining` and is done using the `->` operator. The example below says: `etcd` package BEFORE `etc` service.

```
Package['etcd'] -> Service['etcd']
```

An alternative syntax is shown below. 

```
package { 'etcd':
  ensure => installed,
} ->
service { 'etcd':
  ensure  => running,
}
```

In the following exercise, we are going to install the etcd package, ensure that etcd is running, and also change the instance name from `default` to `myetcd`. The instance name is set in the file /etc/etcd/etcd.conf in a variable called ETCD_NAME, which is set to default by default.

In order to perform this task, lets first install `puppetlabs-stdlib` module. This module provides a resource to replace a string in a file.

```
[root@agent vagrant]# puppet module install puppetlabs-stdlib
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environme
```

We then create `etcd.pp` with the following content:

```
package { 'etcd':
  ensure => installed,
}
service { 'etcd':
  ensure => running,
}

file_line {'etcd':
  ensure => present,
  path   => '/etc/etcd/etcd.conf',
  line   => 'ETCD_NAME="myetcd"',
  match  => 'ETCD_NAME="default"',
}

File_line['etcd'] -> Package['etcd'] ~> Service['etcd']
```

When we run our script, we get the following output:

```
[root@agent vagrant]# puppet apply etcd.pp
Notice: Compiled catalog for agent.localdomain in environment production in 0.07 seconds
Notice: /Stage[main]/Main/File_line[etcd]/ensure: created
Notice: Applied catalog in 0.12 seconds
```

Afterwards we can verify that our change has been made.

```
[root@agent vagrant]# grep ETCD_NAME /etc/etcd/etcd.conf
ETCD_NAME="myetcd"
```

This is great, so let's go back and create a module for this on the `puppet` VM. Let's first uninstall `etcd` on the agent and remove `/etc/etcd/etcd.conf`. Then on `puppet`, create the directory `/etc/puppetlabs/code/environments/production/modules/etcd` and withiin it a subdirectory called `etcd/manifests` and in this directory create init.pp. The content is as follows. Note that our dependency graph says "Install etcd before replacing the instance name and before starting the service".

```
class etcd {
  package { 'etcd':
      ensure => installed,
  }

  service { 'etcd':
      ensure => running,
  }

  file_line {'etcd':
        ensure  => present,
        path    => '/etc/etcd/etcd.conf',
        line    => 'ETCD_NAME="myetcd"',
        match   => 'ETCD_NAME="default"',
  }
  
  Package['etcd'] -> File_line['etcd'] ~> Service['etcd']
}
```

Next we need to edit `/etc/puppetlabs/code/environments/production/manifests/site.pp` so that it is classified with our agent node:

```
node "agent.localdomain" {
  include etcd
}
```

Next, ensure that the `puppetserver` service is running on the puppet VM:

```
[root@puppet manifests]# systemctl start puppetserver
```

Install the `puppetlabs-stdib module on the master:

```
[root@puppet manifests]# puppet module install puppetlabs-stdlib
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└── puppetlabs-stdlib (v6.5.0)
```

Aferwards when we run the agent, we get the following output:

```
[root@agent vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606692702'
Notice: /Stage[main]/Etcd/Package[etcd]/ensure: created
Notice: /Stage[main]/Etcd/File_line[etcd]/ensure: created
Info: /Stage[main]/Etcd/File_line[etcd]: Scheduling refresh of Service[etcd]
Notice: /Stage[main]/Etcd/Service[etcd]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Etcd/Service[etcd]: Unscheduling refresh on Service[etcd]
Notice: Applied catalog in 5.57 seconds
```

And we can also verify the instance name in /etc/etcd/etcd.conf:

```
[root@agent vagrant]# grep ETCD_NAME /etc/etcd/etcd.conf
ETCD_NAME="myetcd"
```

### httpd

Let's try another exercise. The goal is to install Apache as a service and change the listener port from 80 to 8080. 

First lets download the httpd RPM.

```
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/httpd-2.4.6-95.el7.centos.x86_64.rpm
```

Then extract it.

```
rpm2cpio httpd-2.4.6-95.el7.centos.x86_64.rpm | cpio -id
```

Then create the following directory.

```
mkdir -p  /etc/puppetlabs/code/environments/production/modules/webserver/files
```

And copy httpd.conf to this directory.

```
cp /root/etc/httpd/conf/httpd.conf /etc/puppetlabs/code/environments/production/modules/webserver/files/httpd.conf
```

Edit this file so that it says `Listen 8080` instead of `Listen 80`.

Next, we'll go to /etc/puppetlabs/code/environments/production/modules/webserver/manifests and create init.pp with the following content:

```
class webserver {
  package { 'httpd':
      ensure => installed,
  }

  file { '/etc/httpd/conf/httpd.conf':
    ensure => file,
    source => 'puppet:///modules/webserver/httpd.conf',
    require => Package['httpd'],
  }

  service { 'httpd':
      ensure    => running,
      enable    => true,
      subscribe => File['/etc/httpd/conf/httpd.conf'],
  }
}
```

Next we need to edit `/etc/puppetlabs/code/environments/production/manifests/site.pp` so that it is classified with our agent node:

```
node "agent.localdomain" {
  include webserver
}
```

Then run the agent:

```
root@agent conf]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606711803'
Notice: /Stage[main]/Webserver/Package[httpd]/ensure: created
Notice: /Stage[main]/Webserver/File[/etc/httpd/conf/httpd.conf]/content:
--- /etc/httpd/conf/httpd.conf	2020-11-16 14:44:03.000000000 +0000
+++ /tmp/puppet-file20201130-10536-akm5dd	2020-11-30 04:50:06.095901701 +0000
@@ -39,7 +39,7 @@
 # prevent Apache from glomming onto all bound IP addresses.
 #
 #Listen 12.34.56.78:80
-Listen 80
+Listen 8080

 #
 # Dynamic Shared Object (DSO) Support

Info: Computing checksum on file /etc/httpd/conf/httpd.conf
Info: FileBucket got a duplicate file {md5}f5e7449c0f17bc856e86011cb5d152ba
Info: /Stage[main]/Webserver/File[/etc/httpd/conf/httpd.conf]: Filebucketed /etc/httpd/conf/httpd.conf to puppet with sum f5e7449c0f17bc856e86011cb5d152ba
Notice: /Stage[main]/Webserver/File[/etc/httpd/conf/httpd.conf]/content:

Notice: /Stage[main]/Webserver/File[/etc/httpd/conf/httpd.conf]/content: content changed '{md5}f5e7449c0f17bc856e86011cb5d152ba' to '{md5}04e9239e7bd5d5b9b85864226d60eee5'
Info: /Stage[main]/Webserver/File[/etc/httpd/conf/httpd.conf]: Scheduling refresh of Service[httpd]
Notice: /Stage[main]/Webserver/Service[httpd]/ensure: ensure changed 'stopped' to 'running'
Info: /Stage[main]/Webserver/Service[httpd]: Unscheduling refresh on Service[httpd]
Notice: Applied catalog in 3.08 seconds
```

Verify:

```
[root@agent /]# curl http://localhost:8080
```
