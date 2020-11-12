# Part 2: Networking

## The iproute2 suite of tools

iproute2 is a suite of command-line tools used to configure the network. It obsoletes a number of older utilities. Its development is closely tied to the development of networking components of the Linux kernel.

## Getting the IP address

The old way:

```
$ ifconfig
```

The new way:

```
$ ip address
```

Neither is very clear about which IP address is assigned to you by your router. For this task, you can use `ip route`. The exerpt below shows that my IP addrresss is 192.168.10.102, assigned by the router IP at 192.168.10.1, on network interface enp37s0.

```
$ ip route
...
default via 192.168.10.1 dev enp37s0 proto dhcp src 192.168.10.102 metric 202
```

## Testing network performance


The `mtr` tool can be used to test packet loss over multiple routers. Below is an example of using mtr to check the packet loss to www.google.com over 100 samples. The default interval is 1 per second.

```bash
$ mtr -c 100  www.google.com -rw
```

The example below sends a packet every 10 second to prevent ICMP rate limiting.

```bash
$ mtr -c 10 -i 10 www.comcast.com -rw
```

## Enabling and disabling network

Let's see what happens if we disable our device.

```bash
# ip link set enp37s0 down
# ip route
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.18.0.0/16 dev br-f953c5d49caa proto kernel scope link src 172.18.0.1 linkdown
172.19.0.0/16 dev br-4c80cbe0e19d proto kernel scope link src 172.19.0.1 linkdown
```

Later when we bring up the network again, we can see our device is up.

```bash
 # ip link set enp37s0 up
# ip route
default via 192.168.10.1 dev enp37s0 proto dhcp src 192.168.10.102 metric 202
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown
172.18.0.0/16 dev br-f953c5d49caa proto kernel scope link src 172.18.0.1 linkdown
172.19.0.0/16 dev br-4c80cbe0e19d proto kernel scope link src 172.19.0.1 linkdown
192.168.10.0/24 dev enp37s0 proto dhcp scope link src 192.168.10.102 metric 202
```

## Scanning IP addresses

One tool that most IT admins should know is *arp*. This command is used to manipulate the kernel's IPv4 network neighbor cache.

If you just *arp* you will see all known IP address found on your LAN.

```bash
Address                  HWtype  HWaddress           Flags Mask            Iface
_gateway                 ether   xx:xx:xx:xx:xx:xx   C                     enp37s0
```

Alternatively, you can print the IP address using BSD-style output by passing in the *-a* option.

```bash
$ arp -a
_gateway (192.168.10.1) at xx:xx:xx:xx:xx:xx [ether] on enp37s0
```

The *arp* command is deprecated and its replacement is *ip neighbor*, or imply *ip n*.

```bash
$ ip n
192.168.10.1 dev enp37s0 lladdr xx:xx:xx:xx:xx:xx  STALE
```

However, if you want to find the gateway using the *ip* command, run *ip route*. The *default* entry points to the gateway.

```bash
$ ip route
default via 192.168.10.1 dev enp37s0 proto dhcp src 192.168.10.102 metric 202
...
```

Once you find the gateway, You could run `nmap`.

```bash
$ nmap -sP 192.168.10.1/24
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-11 09:44 PST
Nmap scan report for aterm.me (192.168.10.1)
Host is up (0.00023s latency).
MAC Address: xx:xx:xx:xx:xx:xx (NEC Platforms)
Nmap scan report for 192.168.10.104
Host is up (0.00053s latency).
MAC Address: xx:xx:xx:xx:xx:xx (Action Star Enterprise)
Nmap scan report for 192.168.10.102
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 27.99 seconds
```

This command found two IP addresses of interest: 192.168.10.104 and 192.168.10.102.

Another interesting use of `nmap` is its ability to discover the operating system associated with IP addresses. I have a Mac running on my network; let's see if nmap can discover this fact. This command needs root privileges.

```bash
$ nmap -sT 192.168.10.1/24
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-11 09:47 PST
Nmap scan report for aterm.me (192.168.10.1)
Host is up (0.0012s latency).
Not shown: 997 closed ports
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
52869/tcp open  unknown
MAC Address: xx:xx:xx:xx:xx:xx (NEC Platforms)

Nmap scan report for 192.168.10.104
Host is up (0.0013s latency).
All 1000 scanned ports on 192.168.10.104 are closed (500) or filtered (500)
MAC Address: xx:xx:xx:xx:xx:xx (Action Star Enterprise)

Nmap scan report for 192.168.10.102
Host is up (0.00024s latency).
All 1000 scanned ports on 192.168.10.102 are closed

Nmap done: 256 IP addresses (3 hosts up) scanned in 31.18 seconds
```

The conclusion is that, no, `nmap` did not discover the OSs.

### Case Study

The objective is to find all IPs on the LAN. 

First we find the gateway.

```bash
$ ip route
default via 192.168.10.1 dev enp37s0 proto dhcp src 192.168.10.102 metric 202
...
```

Next we scan the network.

```bash
$ nmap -sP 192.168.10.1/24
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-11 09:44 PST
Nmap scan report for aterm.me (192.168.10.1)
Host is up (0.00023s latency).
MAC Address: xx:xx:xx:xx:xx:xx (NEC Platforms)
Nmap scan report for 192.168.10.104
Host is up (0.00053s latency).
MAC Address: xx:xx:xx:xx:xx:xx (Action Star Enterprise)
Nmap scan report for 192.168.10.102
Host is up.
Nmap done: 256 IP addresses (3 hosts up) scanned in 27.99 seconds
```


## References

https://en.wikipedia.org/wiki/Iproute2

https://en.wikipedia.org/wiki/MTR_(software)
