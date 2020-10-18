# Part 1: Setting Firewall

When working with `ufw`, the firewall program for Linux, get the status.

```bash
$ ufw status
Status: inactive
```

If it says `inactive`, ufw is not enabled and willl not start when the system boots. 

Let's start by resetting UFW. This will disable the firewall and backup active rules.

```bash
# ufw reset
Resetting all rules to installed defaults. Proceed with operation (y|n)? y
Backing up 'user.rules' to '/etc/ufw/user.rules.20201017_165958'
Backing up 'before.rules' to '/etc/ufw/before.rules.20201017_165958'
Backing up 'after.rules' to '/etc/ufw/after.rules.20201017_165958'
Backing up 'user6.rules' to '/etc/ufw/user6.rules.20201017_165958'
Backing up 'before6.rules' to '/etc/ufw/before6.rules.20201017_165958'
Backing up 'after6.rules' to '/etc/ufw/after6.rules.20201017_165958'
WARN: '/etc/ufw/user.rules' is world readableWARN: '/etc/ufw/before.rules' is world readableWARN: '/etc/ufw/after.rules' is world readableWARN: '/etc/ufw/user6.rules' is world readableWARN: '/etc/ufw/before6.rules' is world readableWARN: '/etc/ufw/after6.rules' is world readable
```

Let's reject all incoming traffic.

```bash
# ufw default reject incoming
Default incoming policy changed to 'reject'
(be sure to update your rules accordingly)
```

Now if we enable the firewall we can verify our rule.


```bash
# ufw enable
Firewall is active and enabled on system startup
# ufw status verbose
Status: active
Logging: on (low)
Default: reject (incoming), allow (outgoing), deny (routed)
New profiles: skip
```

We can also see that logging is enabled.

To verify that the firewall works, all we have to do is log into a host on our network and try to SSH back.

```bash
$ ssh <my.host.ip>
ssh: connect to host <my.host.ip> port 22: Connection refused
```

We can confirm that access is blocked by our firewall by checking dmeg.

```bash

#  dmesg | grep '\[UFW'
[ 2446.471349] [UFW BLOCK] IN=enp37s0 OUT= MAC=XXXXX SRC=XXXX DST=XXXX LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=52999 DF PROTO=TCP SPT=43396 DPT=22 WINDOW=64240 RES=0x00 SYN URGP=0
```

We could allow SSH to our host as long as it's from our subnet.

```bash
# ufw allow from 192.168.10.0/24 to any proto tcp port 22
Rule added
```

Let's confirm this.

```bash
# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    192.168.10.0/24
```

Maybe that was too broad. Let's remove it.

```bash
# ufw delete 1
Deleting:
 allow from 192.168.10.0/24 to any port 22 proto tcp
Proceed with operation (y|n)? y
Rule deleted
```

And lets only limit SSH from one host on our subnet.


```bash
# ufw allow from 192.168.10.100 to any proto tcp port 22
Rule added
# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    192.168.10.100
```

Check the file /etc/ufw/user.rules for the new rule.

```bash
### tuple ### allow tcp 22 0.0.0.0/0 any 192.168.10.100 in
-A ufw-user-input -p tcp --dport 22 -s 192.168.10.100 -j ACCEPT
```

Let's try increasing our log level.

```bash
# ufw logging medium
```

```And follow our logs.

```bash
# dmesg -W
```

Now when we try to SSH from our remote host we can see the origin.

```bash
[ 3950.816179] [UFW ALLOW] IN= OUT=enp37s0 SRC=192.168.10.102 DST=66.85.78.80 LEN=76 TOS=0x10 PREC=0x00 TTL=64 ID=50185 DF PROTO=UDP SPT=35981 DPT=123 LEN=56
```

Now that we're satisfied with `ufw`, let's set logging level to `low` again.


```bash
# ufw logging low
```
