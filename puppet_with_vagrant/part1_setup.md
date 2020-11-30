# Puppet with Vagrant

## Introduction

The following are notes based on the Udemy course `Master Puppet for Devops Success`. 

## Setup

Create the following Vagrantfile.

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
#

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "centos/7"

  config.vm.define 'puppet' do |puppet|
    puppet.vm.hostname = 'puppet.localdomain'
    puppet.vm.network "private_network", type: "dhcp"

    puppet.vm.provider :virtualbox do |v|
      v.memory = 4096
    end

    puppet.vm.provision "shell", inline: %Q{
      cp -r /vagrant/.vim* /root
      rpm -Uvh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
      yum -y install git vim tree
      echo 'export PATH=$PATH:/opt/puppetlabs/puppet/bin' >> /root/.bashrc
    }
  end

  config.vm.define 'agent' do |agent|
    agent.vm.hostname = 'agent.localdomain'
    agent.vm.network "private_network", type: "dhcp"
    agent.vm.provision "shell", inline: %Q{
      cp -r /vagrant/.vim* /root
      rpm -Uvh https://yum.puppetlabs.com/puppetlabs-release-pc1-el-7.noarch.rpm
      yum -y install git vim
      echo 'export PATH=$PATH:/opt/puppetlabs/puppet/bin' >> /root/.bashrc
    }
  end

end
```

To bring the VMs up, type `vagrant up`. The script will create two VMs.

```
vagrant status
Current machine states:

puppet                    running (virtualbox)
agent                     running (virtualbox)
```

## Installing Puppet

### Agent

Login to the agent machine and install puppet agent.

```
yum install puppet-agent
```

### Server

On `puppet`, install the server.

```
yum install puppetserver
```

### Paths

The Vagrantfile automatically sets the correct path to the puppet executable, which is located at /opt/puppetlabs/puppet/bin. 

## First steps

On the agent VM, we can qeury the system for a user. In puppet, we deal with `resources` and a user is a type of resource. The result of this command shows that bob does not exist.

```
puppet resource user bob
user { 'bob':
  ensure => 'absent',
}
```

We can add the user bob with the following:

```
puppet resource user bob ensure=present
Notice: /User[bob]/ensure: created
user { 'bob':
  ensure => 'present',
}
```

We can confirm this.

```
id bob
uid=1001(bob) gid=1001(bob) groups=1001(bob)
```

You can find more information about the `user` resource online, or use the `describe` command.

```
puppet describe resource user |more
```

Instead of using imperative commands, we can also put the commands in a file. Puppet files end with a pp extension. On `puppet`, create a file called `bob.pp` with the following content:

```
group { 'sysadmins':
    ensure => 'present',
}

user { 'bob':
  ensure => 'present',
  uid => '9999',
  groups => 'sysadmins',
}
```

This file declares a resource type of `user` , the title of the resource as 'bob', and attribues. To run it type the following:

```
puppet apply bob.pp
```

In this example, we see that creating the user `bob` requires that we also create a group `sysadmins`. In puppet, similar tasks are grouped into classes. When we create a class, we create a singleton object that models a desired state. 

Below we define webserver class:

```
class webserver {
  group { 'sysadmins':
      ensure => 'present',
  }

  user { 'bob':
    ensure => 'present',
    uid => '9999',
    groups => 'sysadmins',
  }
}
```

In order to create this class, we need to declare the class. One way is to use the `include` keyword.

```
...
include webserver
```

Another is to use the `class` keyword.

```
...
class { 'webserver': }
```

Note: If at any time you want to know the names of the VMs, type `vagrant status`. 

Once we add `include webserver` in this file, we can run the puppet file with `puppet apply bob.pp`. 

Another way to run this class is declaratively on the commandline by typing `puppet apply -e webserver`. When we do this however puppet will not be able to find the class. We need to make the class as part of a module. Puppet finds modules in the location pointed by running `puppet config print modulepath`. Within the modules directory, we create a directory with the module name, a directory called manifests, and finally the base class with the file name `init.pp`. 

When we run `puppet config print modulepath` the returned value is `/etc/puppetlabs/code/environments/production/modules:/etc/puppetlabs/code/modules:/opt/puppetlabs/puppet/modules`. What we're interested in is `/etc/puppetlabs/code/environments/production/modules`. 

To create a module in this directory, run `mkdir -p /etc/puppetlabs/code/environments/production/modules/webserver/manifests`. 

Next, we copy bob.pp to `/etc/puppetlabs/code/environments/production/modules/webserver/init.pp`. 

Once we do this, we can run `puppet apply -e webserver`. To summarize:

```
cat /etc/puppetlabs/code/environments/production/modules/webserver/manifests/init.pp

class webserver {
  group { 'sysadmins':
      ensure => 'present',
  }

  user { 'bob':
    ensure => 'present',
    uid => '9999',
    groups => 'sysadmins',
  }

  user { 'mary':
    ensure => 'present',
    uid => '9998',
    groups => 'sysadmins',
  }

  user { 'andy':
    ensure => 'present',
    uid => '9997',
    groups => 'sysadmins',
  }
}

puppet apply -e 'include webserver'
Notice: Compiled catalog for puppet.localdomain in environment production in 0.05 seconds
Notice: Applied catalog in 0.02 seconds
```

Again, the file structure is:

```
[root@puppet modules]# tree
.
└── webserver
    └── manifests
        └── init.pp

2 directories, 1 file
```

To summarize, in order to run puppet modules, we need to copy them to a location beneath modulepath. 

## Puppet Server and Agent

The puppet agent has a variety of command-line options: -o (onetime), -n (nodaemonize) and -v (verbose). The -t (test) option is short of -onv. There is another option --noop which lets agents perform actions without making the actual changes. 

Communication between the agent and server is over SSL. We need to generate certificates on both sides to get this working.

First we need to ensure that the agent can ping the server. We do this by editing the agent's /etc/hosts file and adding an entry so that we can resolve the host named `puppet`.

Next, we need to look at the server's /etc/sysconfig/puppetserver. This contains JVM settings that we may need to tweak. 

To start the server, run `systemct start puppetserver`. This can take a while. 

Once started, on the agent side, we can run `puppet agent -t` to generate agent-side certs.

On the server-side, we can use the `puppet cert` command to manage certificates. 

If we run this command on the server, we see the agent's certificate.

```
[root@puppet vagrant]# puppet cert list
  "agent.localdomain" (SHA256) 3E:91:01:1A:39:5D:BE:62:26:C9:49:98:78:94:37:21:15:4A:6E:93:CB:D1:63:A6:DC:9C:4E:B8:6D:CC:69:85
  ```

  We then need to sign this.

  ```
root@puppet vagrant]# puppet cert --sign agent.localdomain
Signing Certificate Request for:
  "agent.localdomain" (SHA256) 3E:91:01:1A:39:5D:BE:62:26:C9:49:98:78:94:37:21:15:4A:6E:93:CB:D1:63:A6:DC:9C:4E:B8:6D:CC:69:85
Notice: Signed certificate request for agent.localdomain
Notice: Removing file Puppet::SSL::CertificateRequest agent.localdomain at '/etc/puppetlabs/puppet/ssl/ca/requests/agent.localdomain.pem'
```

Now that the server has signed the certificate, the agent can use it. So let's run `puppet agent -t` again on the agent box.

```
root@agent etc]# puppet agent -t
Info: Caching certificate for agent.localdomain
Info: Caching certificate_revocation_list for ca
Info: Caching certificate for agent.localdomain
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606346350'
Notice: Applied catalog in 0.02 seconds
```

If we run `puppet cert --list` again on the server, we see no certificates. 

If we want to see all certificates again, we can run `puppet cert --all`.

```
[root@puppet vagrant]# puppet cert --list --all
+ "agent.localdomain"  (SHA256) 62:5E:E2:A1:4F:6C:AF:15:7D:24:87:85:27:BF:75:76:26:ED:2B:23:57:2A:37:C4:D0:61:4B:FE:A0:FD:1A:88
+ "puppet.localdomain" (SHA256) 31:98:C9:A8:3D:C0:77:C1:58:EB:A4:BA:3B:ED:34:1E:38:66:A4:D2:17:C5:5C:C4:AA:4B:65:08:5F:4B:00:95 (alt names: "DNS:puppet", "DNS:puppet.localdomain")
[root@puppet vagrant]#
```

How did the client find the server? It turns out that the agent will try to contact a host named puppet. As long as we have an entry in the clients /etc/hosts file named `puppet`, the client will try to contact the server at that IP. If instead we want to use a different host name, we can specify that on the command line.

```
puppet agent --server puppetmaster -t
```

Also note that the server name must match the DNS name in the certificate. 

### Common SSL issues

- Run NTP on the puppet and agents to keep them in sync
- Certname mismatch between what the agent expects and what the puppet provides. In this case, the server needs to remove the old agent certificate. 

To resolve certificate issues:

- On the puppet side, remove the old agent's certificate

  ```
  puppet cert clean agent.localdomain
  Notice: Revoked certificate with serial 3
  Notice: Removing file Puppet::SSL::Certificate agent.localdomain at '/etc/puppetlabs/puppet/ssl/ca/signed/agent.localdomain.pem'
  Notice: Removing file Puppet::SSL::Certificate agent.localdomain at '/etc/puppetlabs/puppet/ssl/certs/agent.localdomain.pem'
  ```

- On the agent, run:

  ```
  puppet config print ssldir
  ```

  When we go to that directory, we see a lot of files. Remove all of these.

  ```
  [root@agent etc]# ls /etc/puppetlabs/puppet/ssl/
  certificate_requests  certs  crl.pem  private  private_keys  public_keys
  root@agent etc]# rm -r /etc/puppetlabs/puppet/ssl/*
  ```
 
  The next step is to regenerate client-side certificates.

  ```
  [root@agent etc]# puppet agent -t
  Info: Creating a new SSL key for agent.localdomain
  Info: Caching certificate for ca
  Info: csr_attributes file loading from /etc/puppetlabs/puppet/csr_attributes.yaml
  Info: Creating a new SSL certificate request for agent.localdomain
  Info: Certificate Request fingerprint (SHA256): 6E:09:6D:8C:DF:0C:4B:E6:AA:C7:F1:41:5F:64:91:3C:E3:B7:CC:95:EF:77:63:8A:F3:FD:6F:C0:51:0A:F1:35
  Info: Caching certificate for ca
  Exiting; no certificate found and waitforcert is disabled
  ```

  Back on the puppet side, we can see this new certificate. We can then import it.

  ```
  [root@puppet vagrant]# puppet cert --list
  "agent.localdomain" (SHA256) 6E:09:6D:8C:DF:0C:4B:E6:AA:C7:F1:41:5F:64:91:3C:E3:B7:CC:95:EF:77:63:8A:F3:FD:6F:C0:51:0A:F1:35

  [root@puppet vagrant]# puppet cert --sign agent.localdomain
  Signing Certificate Request for:
  "agent.localdomain" (SHA256) 6E:09:6D:8C:DF:0C:4B:E6:AA:C7:F1:41:5F:64:91:3C:E3:B7:CC:95:EF:77:63:8A:F3:FD:6F:C0:51:0A:F1:35
  Notice: Signed certificate request for agent.localdomain
  Notice: Removing file Puppet::SSL::CertificateRequest agent.localdomain at '/etc/puppetlabs/puppet/ssl/ca/requests/agent.localdomain.pem'
  [root@puppet vagrant]
  ```

### Facter

Facter gathers facts about a system. We can run it on the command-line and even provide the key that we want to interrogate.

```
[root@agent etc]# facter os
{
  architecture => "x86_64",
  family => "RedHat",
  hardware => "x86_64",
  name => "CentOS",
  release => {
    full => "7.8.2003",
    major => "7",
    minor => "8"
  },
  selinux => {
    config_mode => "enforcing",
    config_policy => "targeted",
    current_mode => "enforcing",
    enabled => true,
    enforced => true,
    policy_version => "31"
  }
}
```

Facter is important to know as it used as part of the lifecycle of agent-pupet communication. When the agent comes up, it authenticates to the server via SSL. Once this is done, the agent compiles facts about itself and sends it to the server. The server responds with a catalog, a list of states that the agent should be in based on its facts. The agent then compares its state with the catalog (RAL) and makes changes as necessary (drift is detected). Afterwwards, the agent reports its changes back to the server. 

### Classification

Above, we said that when an agent comes up, it authnticates to the server and sends information about itself to the puppet master. The master then classifes the agent. What this means is that the server identifies what classes should be applied to it and generates a catalog.

In puppet, we can define this classification in 2 different ways: 1) Using an external node classifier or ENC (typically done in production), and 2) Creating a manifest file or site.pp. To use ENC, either use open-source foreman or the Enterprise Console. 

The manifiest file, or site.pp, looks like the following. You can see that it includes classes. You could have other arbirtrary things in it, but it's not recommended.

```
node "agent.localhost.com" {
  include webserver
  include database
}
```

The argument to the node keyword can also be a regular expression.

```
node /*.foo.com/ {
  ...
}
```

There's also a default node keyword that applies to any nodes that don't match known criteria.

```
node default {
  ...
}
```

Let's run the agent again with the -t option.

```
Notice: Applied catalog in 0.02 seconds
[root@agent etc]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606355931'
Notice: Applied catalog in 0.02 seconds
```

We seee that the agent talked to the server, but nothing changed. This is because the agent was not classified. 

To find out where we create site.pp run the following command:

```
[root@puppet vagrant]# puppet config print manifest
/etc/puppetlabs/code/environments/production/manifests
```

So let's go to this directory and create site.pp with the following conntent:

```
[root@puppet vagrant]# cd /etc/puppetlabs/code/environments/production/manifests
vi site.pp
node "agent.localdomain" {
  include webserver
}
```

Then if we run `puppet agent -t` again, we get new output.

```
[root@agent etc]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606356409'
Notice: /Stage[main]/Webserver/Group[sysadmins]/ensure: created
Notice: /Stage[main]/Webserver/User[bob]/uid: uid changed '1001' to '9999'
Notice: /Stage[main]/Webserver/User[bob]/groups: groups changed '' to ['sysadmins']
Notice: /Stage[main]/Webserver/User[mary]/ensure: created
Notice: /Stage[main]/Webserver/User[andy]/ensure: created
Notice: Applied catalog in 0.14 seconds
```

As a reminder, this is the content of init.pp for the webserver module.

```
[root@puppet manifests]# cat /etc/puppetlabs/code/environments/production/modules/webserver/manifests/init.pp

class webserver {
  group { 'sysadmins':
      ensure => 'present',
  }

  user { 'bob':
    ensure => 'present',
    uid => '9999',
    groups => 'sysadmins',
  }

  user { 'mary':
    ensure => 'present',
    uid => '9998',
    groups => 'sysadmins',
  }

  user { 'andy':
    ensure => 'present',
    uid => '9997',
    groups => 'sysadmins',
  }
}
```

So if we change the UID of mary.

```
usermod -u 9995 mary
```

And run agent again.

```
[root@agent etc]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606356660'
Notice: /Stage[main]/Webserver/User[mary]/uid: uid changed '9995' to '9998'
Notice: Applied catalog in 0.05 seconds
```

We see that puppet has change the UID of mary.

So to summarize, manifests and modules are both stored at /etc/puppetlabs/code/environments/production.

```
[root@puppet production]# tree
.
├── environment.conf
├── hieradata
├── manifests
│   └── site.pp
└── modules
    └── webserver
        └── manifests
            └── init.pp

5 directories, 3 files
```


### Troubleshooting

The agent may display the following error message

```
[root@agent vagrant]# puppet agent -t
Warning: Unable to fetch my node definition, but the agent run will continue:
Warning: Connection refused - connect(2) for "puppet" port 8140
Info: Retrieving pluginfacts
Error: /File[/opt/puppetlabs/puppet/cache/facts.d]: Failed to generate additional resources using 'eval_generate': Connection refused - connect(2) for "puppet" port 8140
Error: /File[/opt/puppetlabs/puppet/cache/facts.d]: Could not evaluate: Could not retrieve file metadata for puppet:///pluginfacts: Connection refused - connect(2) for "puppet" port 8140
Info: Retrieving plugin
Error: /File[/opt/puppetlabs/puppet/cache/lib]: Failed to generate additional resources using 'eval_generate': Connection refused - connect(2) for "puppet" port 8140
Error: /File[/opt/puppetlabs/puppet/cache/lib]: Could not evaluate: Could not retrieve file metadata for puppet:///plugins: Connection refused - connect(2) for "puppet" port 8140
Error: Could not retrieve catalog from remote server: Connection refused - connect(2) for "puppet" port 8140
Warning: Not using cache on failed catalog
Error: Could not retrieve catalog; skipping run
Error: Could not send report: Connection refused - connect(2) for "puppet" port 8140
```

If you see this error, make sure `puppet` is running. If not, then start it.

```
[root@puppet etc]# systemctl status puppetserver
● puppetserver.service - puppetserver Service
   Loaded: loaded (/usr/lib/systemd/system/puppetserver.service; disabled; vendor preset: disabled)
   Active: inactive (dead)

Nov 26 23:58:33 puppet.localdomain systemd[1]: Starting puppetserver Service...
Nov 26 23:58:33 puppet.localdomain puppetserver[3695]: OpenJDK 64-Bit Server VM warning: ignoring option Max...8.0
Nov 26 23:59:18 puppet.localdomain systemd[1]: Started puppetserver Service.
Nov 27 00:00:04 puppet.localdomain systemd[1]: Stopping puppetserver Service...
Nov 27 00:00:05 puppet.localdomain systemd[1]: Stopped puppetserver Service.
Hint: Some lines were ellipsized, use -l to show in full.
[root@puppet etc]# systemctl start puppetserver
```

Afterwards, the agent should work.

```
[root@agent vagrant]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606435368'
Notice: Applied catalog in 0.03 seconds
```
