# The Puppet Language (part 2)

## Variables

Variables are created using the `$` sign and cannot be changed once declared.

```
$hostname = 'etcd-host'
```

For example, lets' try to use the `wget` module to download the `nginx` tarball on an agent. First lets find the module.

```
root@puppet nginx]# puppet module search puppet-wget
Notice: Searching https://forgeapi.puppet.com ...
NAME          DESCRIPTION                                               AUTHOR        KEYWORDS
puppet-wget   Download files with wget                                  @puppet
pest-nexus    Puppet module for Sonatype Nexus                          @pest
```

Then install it.

```
[root@puppet nginx]# puppet module install puppet-wget
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└── puppet-wget (v2.0.1)
```

Next, create the following class in `/etc/puppetlabs/code/environments/production/modules/nginx/manifests/init.pp`.

```
class nginx {
    wget::fetch { "Download nginx":
      source      => 'https://nginx.org/download/nginx-1.19.5.tar.gz',
      destination => '/tmp/',
      timeout     => 0,
      verbose     => false,
    }
}
```

Finally modify `/etc/puppetlabs/code/environments/production/manifests/site.pp` to include our module.

```
node "agent.localdomain" {
  include nginx
}
```

When we run the agent, we see that the file was downloaded.

```
[root@agent conf]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Loading facts
Info: Caching catalog for agent.localdomain
Info: Applying configuration version '1606761287'
Notice: /Stage[main]/Wget/Package[wget]/ensure: created
Notice: /Stage[main]/Nginx/Wget::Fetch[Download nginx]/Exec[wget-Download nginx]/returns: executed successfully
Notice: Applied catalog in 4.82 seconds
```

We could rewrite our `nginx` class to use variables.

```
class nginx {
    $nginx_version = '1.19.5'

    wget::fetch { "Download nginx":
      source      => "https://nginx.org/download/nginx-${nginx_version}.tar.gz",
      destination => '/tmp/',
      timeout     => 0,
      verbose     => false,
    }
}
```

We could use variables to improve our last example where we installed `etcd`.

```
class etcd {
  $package_name = "etcd"
  $service_name = "etcd"
  $config_file = "/etc/etcd/etcd.conf"
  $etcd_default_instance = "default"
  $etcd_prod_instance = "myetcd"

  package { $package_name:
      ensure => installed,
  }


  file_line { $config_file:
      ensure  => present,
      path    => $config_file,
      line    => "ETCD_NAME=\"${etcd_prod_instance}\"",
      match   => "ETCD_NAME=\"${etcd_default_insdtance}\"",
      require => Package[$package_name],
  }

  service { $service_name:
      ensure    => running,
      enable    => true,
      subscribe => File_line[$config_file],
  }

}
```

Be very careful of the following:

- variable names MUST be prefixed with `$`
- Resource references MUST refer to existing resources

After apply these changes and running the class again, we can confirm that the instance name has been changed.

```
[root@agent conf]# grep myetcd /etc/etcd/etcd.conf
ETCD_NAME="myetcd"
```

Other things to consider:

### Arrays

```
$users = ['harry', 'mary', 'bob']

user { $users 
    ensure => present,
}
```

### Hashes

```
$uids = {
    'bob'   => '9999',
    'susan' => '9998',
    'greg'  => '9997',
}

notify { "Creating user bob with UID ${uids_bob}": }
```

### Example

We can use facts in puppet. We prefix the `facts` variable with two colons to indicate that it comes from the `top` scope.

```
$whoami = $::facts['os']['family']

notify { "I am running on $whoami": }
```
