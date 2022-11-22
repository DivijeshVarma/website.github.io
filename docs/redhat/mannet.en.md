---
title: "Managing Networking"
---

### DESCRIBING NETWORKING CONCEPTS ###

#### TCP/IP NETWORK MODEL ####

The *TCP/IP* network model is a simplified, four-layered set of abstractions that describes how
different protocols interoperate in order for computers to send traffic from one machine to another
over the Internet. It is specified by RFC 1122, Requirements for Internet Hosts -- Communication
Layers. The four layers are:

- **Application**

Each application has specifications for communication so that clients and servers may
communicate across platforms. Common protocols include SSH (remote login), HTTPS (secure
web), NFS or CIFS (file sharing), and SMTP (electronic mail delivery).

- **Transport**

Transport protocols are TCP and UDP. TCP is a reliable connection-oriented communication,
while UDP is a connectionless datagram protocol. Application protocols use TCP or UDP ports. A
list of well-known and registered ports can be found in the */etc/services* file.

When a packet is sent on the network, the combination of the service port and IP address forms
a socket. Each packet has a source socket and a destination socket. This information can be used
when monitoring and filtering.

- **Internet**

The Internet, or network layer, carries data from the source host to the destination host. The
IPv4 and IPv6 protocols are Internet layer protocols. Each host has an IP address and a prefix
used to determine network addresses. Routers are used to connect networks.

- **Link**

The link, or media access, layer provides the connection to physical media. The most common
types of networks are wired Ethernet (802.3) and wireless WLAN (802.11). Each physical device
has a hardware address (MAC) which is used to identify the destination of packets on the local
network segment.

#### DESCRIBING NETWORK INTERFACE NAMES ####

Each network port on a system has a name, which you use to configure and identify it.

Older versions of Red Hat Enterprise Linux used names like eth0, eth1, and eth2 for each
network interface. The name eth0 was the first network port detected by the operating system,
eth1 the second, and so on. However, as devices are added and removed, the mechanism
detecting devices and naming them could change which interface gets which name. Furthermore,
the PCIe standard does not guarantee the order in which PCIe devices will be detected on boot,
which could change device naming unexpectedly due to variations during device or system startup.

Newer versions of Red Hat Enterprise Linux use a different naming system. Instead of being based
on detection order, the names of network interfaces are assigned based on information from the
firmware, the PCI bus topology, and type of network device.

Network interface names start with the type of interface:

- Ethernet interfaces begin with en
- WLAN interfaces begin with wl
- WWAN interfaces begin with ww

The rest of the interface name after the type will be based on information provided by the server's
firmware or determined by the location of the device in the PCI topology.

- **oN** indicates that this is an on-board device and the server's firmware provided index number
N for the device. So eno1 is on-board Ethernet device 1. Many servers will not provide this
information.

- **sN** indicates that this device is in PCI hotplug slot N. So ens3 is an Ethernet card in PCI hotplug
slot 3.

- **pMsN** indicates that this is a PCI device on bus *M* in slot *N*. So wlp4s0 is a WLAN card on PCI bus
4 in slot 0. If the card is a multi-function device (possible with an Ethernet card with multiple
ports, or devices that have Ethernet plus some other functionality), you may see fN added to the
device name. So enp0s1f0 is function 0 of the Ethernet card on bus 0 in slot 1. There might also
be a second interface named enp0s1f1 that is function 1 of that same device.

Persistent naming means that once you know what the name is for a network interface on the
system, you also know that it will not change later. The trade off is that you cannot assume that a
system with one interface will name that interface eth0.

### IPV4 NETWORKING ###

IPv4 is the primary network protocol used on the Internet today. You should have at least a basic
understanding of IPv4 networking in order to manage network communication for your servers.

**IPv4 Addresses**

An IPv4 address is a 32-bit number, normally expressed in decimal as four 8-bit octets ranging in
value from 0 to 255, separated by dots. The address is divided into two parts: the network part and
the host part. All hosts on the same subnet, which can talk to each other directly without a router,
have the same network part; the network part identifies the subnet. No two hosts on the same
subnet can have the same host part; the host part identifies a particular host on a subnet.

In the modern Internet, the size of an IPv4 subnet is variable. To know which part of an IPv4
address is the network part and which is the host part, an administrator must know the netmask,
which is assigned to the subnet. The netmask indicates how many bits of the IPv4 address belong
to the subnet. The more bits available for the host part, the more hosts can be on the subnet.

The lowest possible address on a subnet (host part is all zeros in binary) is sometimes called the
network address. The highest possible address on a subnet (host part is all ones in binary) is used
for broadcast messages in IPv4, and is called the broadcast address.

Network masks are expressed in two forms. The older syntax for a netmask uses 24 bits for the
network part and reads 255.255.255.0. A newer syntax, called CIDR notation, specifies a
network prefix of /24. Both forms convey the same information; namely, how many leading bits in
the IP address contribute to its network address.

The following examples illustrate how the IP address, prefix (netmask), network part, and host part
are related.

The following examples illustrate how the IP address, prefix (netmask), network part, and host part
are related.

![IPv4 addresses](https://upload.wikimedia.org/wikipedia/commons/thumb/7/74/Ipv4_address.svg/750px-Ipv4_address.svg.png "IPv4 addresses")

![IPv4 addresses and netmasks](https://static.packt-cdn.com/products/9781789535600/graphics/assets/c90133e2-ebcd-46fb-90cb-0764d20a5f3e.png "IPv4 addresses and netmasks")

**Calculating the network address for 192.168.1.107/24**

`Host addr: 192.168.1.107       11000000.10101000.00000001.01101011`

`Network prefix:  /24 (255.255.255.0) 11111111.11111111.11111111.00000000`

`Network addr:    192.168.1.0         11000000.10101000.00000001.00000000`

`Broadcast addr:  192.168.1.255       11000000.10101000.00000001.11111111`

**Calculating the network address for 10.1.1.18/8**

`Host addr:       10.1.1.18           00001010.00000001.00000001.00010010`

`Network prefix:  /8 (255.0.0.0)      11111111.00000000.00000000.00000000`

`Network addr:    10.0.0.0            00001010.00000000.00000000.00000000`

`Broadcast addr:  10.255.255.255      00001010.11111111.11111111.11111111`

**Calculating the network address for 172.16.181.23/19**

`Host addr:       172.168.181.23      10101100.10101000.10110101.00010111`

`Network prefix:  /19 (255.255.224.0) 11111111.11111111.11100000.00000000`

`Network addr:    172.168.160.0       10101100.10101000.10100000.00000000`

`Broadcast addr:  172.168.191.255     10101100.10101000.10111111.11111111`

The special address 127.0.0.1 always points to the local system (“localhost”), and the network
127.0.0.0/8 belongs to the local system, so that it can talk to itself using network protocols.

**IPv4 Routing**

Whether using IPv4 or IPv6, network traffic needs to move from host to host and network to
network. Each host has a routing table, which tells it how to route traffic for particular networks.
A routing table entry lists a destination network, which interface to use when sending traffic, and
the IP address of any intermediate router required to relay a message to its final destination. The
routing table entry matching the destination of the network traffic is used to route it. If two entries
match, the one with the longest prefix is used.

If the network traffic does not match a more specific route, the routing table usually has an entry
for a default route to the entire IPv4 Internet: 0.0.0.0/0. This default route points to a router on a
reachable subnet (that is, on a subnet that has a more specific route in the host's routing table).

If a router receives traffic that is not addressed to it, instead of ignoring it like a normal host,
it forwards the traffic based on its own routing table. This may send the traffic directly to the
destination host (if the router happens to be on the destination's subnet), or it may be forwarded
on to another router. This process of forwarding continues until the traffic reaches its final
destination.

![Example network topology](https://www.linuxprobe.com/links/RH124/images/net/ip4network.svg "Example network topology")

**Example routing table**

|         DESTINATION | INTERFACE | ROUTER (IF NEEDED) |
|:--------------------|:----------|:--------------------|
|        192.0.2.0/24 | wlo1      |                    |
|      192.168.5.0/24 | enp3s0    |                    |
| 0.0.0.0/0 (default) | enp3s0    | 192.168.5.254      |

In this example, traffic headed for the IP address 192.0.2.102 from this host is transmitted
directly to that destination via the wlo1 wireless interface, because it matches the
192.0.2.0/24 route most closely. Traffic for the IP address 192.168.5.3 is transmitted
directly to that destination via the enp3s0 Ethernet interface, because it matches the
192.168.5.0/24 route most closely.

Traffic to the IP address 10.2.24.1 is transmitted out the enp3s0 Ethernet interface to a router
at 192.168.5.254, which forwards that traffic on to its final destination. That traffic matches the
0.0.0.0/0 route most closely, as there is not a more specific route in the routing table of this
host. The router uses its own routing table to determine where to forward that traffic to next.

**IPv4 Address and Route Configuration**

A server can automatically configure its IPv4 network settings at boot time from a DHCP server. A
local client daemon queries the link for a server and network settings, and obtains a lease to use
those settings for a specific length of time. If the client does not request a renewal of the lease
periodically, it might lose its network configuration settings.

As an alternative, you can configure a server to use a static network configuration. In this case,
network settings are read from local configuration files. You must get the correct settings from
your network administrator and update them manually as needed to avoid conflicts with other
servers.

### IPV6 NETWORKING ###

IPv6 is intended as an eventual replacement for the IPv4 network protocol. You will need to
understand how it works since increasing numbers of production systems use IPv6 addressing.
For example, many ISPs already use IPv6 for internal communication and device management
networks in order to preserve scarce IPv4 addresses for customer purposes.

IPv6 can also be used in parallel with IPv4 in a dual-stack model. In this configuration, a
network interface can have an IPv6 address or addresses as well as IPv4 addresses. Red Hat
Enterprise Linux operates in a dual-stack mode by default.

**IPv6 Addresses**

An IPv6 address is a 128-bit number, normally expressed as eight colon-separated groups of four
hexadecimal nibbles (half-bytes). Each nibble represents four bits of the IPv6 address, so each
group represents 16 bits of the IPv6 address.

`2001:0db8:0000:0010:0000:0000:0000:0001`

To make IPv6 addresses easier to write, leading zeros in a colon-separated group do not need to be
written. However, at least one hexadecimal digit must be written in each colon-separated group.

`2001:db8:0:10:0:0:0:1`

Since addresses with long strings of zeros are common, one or more consecutive groups of zeros
only may be combined with exactly one :: block.

`2001:db8:0:10::1`

Notice that under these rules, 2001:db8::0010:0:0:0:1 would be another less convenient
way to write the example address. But it is a valid representation of the same address, and this can
confuse administrators new to IPv6. Some tips for writing consistently readable addresses:

- Suppress leading zeros in a group.

- Use :: to shorten as much as possible.

- If an address contains two consecutive groups of zeros, equal in length, it is preferred to shorten
the leftmost groups of zeros to :: and the rightmost groups to :0: for each group.

- Although it is allowed, do not use :: to shorten one group of zeros. Use :0: instead, and save
:: for consecutive groups of zeros.

- Always use lowercase letters for hexadecimal numbers a through f.

!!! danger "IMPORTANT"

	When including a TCP or UDP network port after an IPv6 address, always enclose the
	IPv6 address in square brackets so that the port does not look like it is part of the
	address.
	
	`[2001:db8:0:10::1]:80`

**IPv6 Subnetting**

A normal IPv6 unicast address is divided into two parts: the network prefix and interface ID. The
network prefix identifies the subnet. No two network interfaces on the same subnet can have the
same interface ID; the interface ID identifies a particular interface on the subnet.

Unlike IPv4, IPv6 has a standard subnet mask, which is used for almost all normal addresses, /64.
In this case, half of the address is the network prefix and half of it is the interface ID. This means
that a single subnet can hold as many hosts as necessary.

Typically, the network provider will allocate a shorter prefix to an organization, such as a /48. This
leaves the rest of the network part for assigning subnets (always of length /64) from that allocated
prefix. For a /48 allocation, that leaves 16 bits for subnets (up to 65536 subnets).

![ IPv6 address parts and subnetting](https://www.linuxprobe.com/links/RH124/images/net/ipv6-subnets-plain-fit.png " IPv6 address parts and subnetting")

**Common IPv6 Addresses and Networks**

| IPV6 ADDRESS OR NETWORK | PURPOSE                              | DESCRIPTION                                                                                                                                                                                                                                                                                                                                                             |
|:------------------------|:-------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
| "::1/128"               | localhost                            | The IPv6 equivalent to 127.0.0.1/8, set on the loopback interface.                                                                                                                                                                                                                                                                                                      |
| "::"                    | The unspecified address              | The IPv6 equivalent to 0.0.0.0. For a network service, this could indicate that it is listening on all configured IP addresses.                                                                                                                                                                                                                                         |
| "::/0"                  | The default route(the IPv6 Internet) | The IPv6 equivalent to 0.0.0.0/0. The default route in the routing table matches this network; the router for this network is where all traffic, for which there is no better route, is sent.                                                                                                                                                                           |
| "2000::/3"              | Global unicast addresses             | “Normal” IPv6 addresses are currently being allocated from this space by IANA. This is equivalent to all the networks ranging from 2000::/16 through 3fff::/16.                                                                                                                                                                                                         |
| "fd00::/8"              | Unique local addresses (RFC 4193)    | IPv6 has no direct equivalent of RFC 1918 private address space, although this is close. A site can use these to self-allocate a private routable IP address space inside the organization, but these networks cannot be used on the global Internet. The site must randomly select a /48 from this space, but it can subnet the allocation into /64 networks normally. |
| "fe80::/10"             | Link-local addresses                 | Every IPv6 interface automatically configures a link-local unicast address that only works on the local link on the fe80::/64 network. However, the entire fe80::/10 range is reserved for future use by the local link. This will be discussed in more detail later.                                                                                                   |
| "ff00::/8"                | Multicast                            | The IPv6 equivalent to 224.0.0.0/4. Multicast is used to transmit to multiple hosts at the same time, and is particularly important in IPv6 because it has no broadcast addresses.                                                                                                                                                                                      |

!!! danger "IMPORTANT"

	The table above lists network address allocations that are reserved for specific
	purposes. These allocations may consist of many different networks. Remember that
	IPv6 networks allocated from the global unicast and link-local unicast spaces have a
	standard /64 subnet mask.

A *link-local* address in IPv6 is an unroutable address used only to talk to hosts on a specific network
link. Every network interface on the system is automatically configured with a link-local address
on the fe80::/64 network. To ensure that it is unique, the interface ID of the link-local address
is constructed from the network interface's Ethernet hardware address. The usual procedure to
convert the 48-bit MAC address to a 64-bit interface ID is to invert bit 7 of the MAC address and
insert ff:fe between its two middle bytes.

- Network prefix: fe80::/64

- MAC address: 00:11:22:aa:bb:cc

- Link-local address: fe80::211:22ff:feaa:bbcc/64

The link-local addresses of other machines can be used like normal addresses by other hosts on
the same link. Since every link has a fe80::/64 network on it, the routing table cannot be used to
select the outbound interface correctly. The link to use when talking to a link-local address must
be specified with a scope identifier at the end of the address. The scope identifier consists of %
followed by the name of the network interface.

For example, to use ping6 to ping the link-local address fe80::211:22ff:feaa:bbcc using
the link connected to the ens3 network interface, the correct command syntax is the following:

``` bash
[user@host ~]$ ping6 fe80::211:22ff:feaa:bbcc%ens3
```

!!! info "NOTE"

	Scope identifiers are only needed when contacting addresses that have “link”
	scope. Normal global addresses are used just like they are in IPv4, and select their
	outbound interfaces from the routing table.

*Multicast* allows one system to send traffic to a special IP address that is received by multiple
systems. It differs from broadcast since only specific systems on the network receive the traffic. It
also differs from broadcast in IPv4 since some multicast traffic might be routed to other subnets,
depending on the configuration of your network routers and systems.

Multicast plays a larger role in IPv6 than in IPv4 because there is no broadcast address in IPv6.
One key multicast address in IPv6 is ff02::1, the all-nodes link-local address. Pinging this
address sends traffic to all nodes on the link. Link-scope multicast addresses (starting ff02::/8)
need to be specified with a scope identifier, just like a link-local address.

``` bash
[user@host ~]$ ping6 ff02::1%ens3
PING ff02::1%ens3(ff02::1) 56 data bytes
64 bytes from fe80::211:22ff:feaa:bbcc: icmp_seq=1 ttl=64 time=0.072 ms
64 bytes from fe80::200:aaff:fe33:2211: icmp_seq=1 ttl=64 time=102 ms (DUP!)
64 bytes from fe80::bcd:efff:fea1:b2c3: icmp_seq=1 ttl=64 time=103 ms (DUP!)
64 bytes from fe80::211:22ff:feaa:bbcc: icmp_seq=2 ttl=64 time=0.079 ms
...output omitted...
```

**IPv6 Address Configuration**

IPv4 has two ways in which addresses get configured on network interfaces. Network addresses
may be configured on interfaces manually by the administrator, or dynamically from the network
using DHCP. IPv6 also supports manual configuration, and two methods of dynamic configuration,
one of which is DHCPv6.

Interface IDs for static IPv6 addresses can be selected at will, just like IPv4. In IPv4, there are two
addresses on a network that could not be used: the lowest address in the subnet and the highest
address in the subnet. In IPv6, the following interface IDs are reserved and cannot be used for a
normal network address on a host.

- The all-zeros identifier 0000:0000:0000:0000 (“subnet router anycast”) used by all routers
on the link. (For the 2001:db8::/64 network, this would be the address 2001:db8::)

- The identifiers fdff:ffff:ffff:ff80 through fdff:ffff:ffff:ffff.

DHCPv6 works differently than DHCP for IPv4, because there is no broadcast address. Essentially,
a host sends a DHCPv6 request from its link-local address to port 547/UDP on ff02::1:2, the
all-dhcp-servers link-local multicast group. The DHCPv6 server then usually sends a reply
with appropriate information to port 546/UDP on the client's link-local address.

The dhcp package in Red Hat Enterprise Linux 8 provides support for a DHCPv6 server.

In addition to DHCPv6, IPv6 also supports a second dynamic configuration method, called *Stateless
Address Autoconfiguration (SLAAC)*. Using SLAAC, the host brings up its interface with a link-
local fe80::/64 address normally. It then sends a “router solicitation” to ff02::2, the all-
routers link-local multicast group. An IPv6 router on the local link responds to the host's link-local
address with a network prefix and possibly other information. The host then uses that network
prefix with an interface ID that it normally constructs in the same way that link-local addresses are
constructed. The router periodically sends multicast updates (“router advertisements”) to confirm
or update the information it provided.

The *radvd* package in Red Hat Enterprise Linux 8 allows a Red Hat Enterprise Linux  based IPv6
router to provide SLAAC through router advertisements.

!!! danger "IMPORTANT"

	A typical Red Hat Enterprise Linux 8 machine configured to get IPv4 addresses
	through DHCP is usually also configured to use SLAAC to get IPv6 addresses. This
	can result in machines unexpectedly obtaining IPv6 addresses when an IPv6 router
	is added to the network.
	
	Some IPv6 deployments combine SLAAC and DHCPv6, using SLAAC to only provide
	network address information and DHCPv6 to provide other information, such as
	which DNS servers and search domains to configure.

### HOST NAMES AND IP ADDRESSES ###

It would be inconvenient if you always had to use IP addresses to contact your servers. Humans
generally would prefer to work with names than long and hard-to-remember strings of numbers.
And so Linux has a number of mechanisms to map a host name to an IP address, collectively called
*name resolution*.

One way is to set a static entry for each name in the **/etc/hosts** file on each system. This
requires you to manually update each server's copy of the file.

For most hosts, you can look up the address for a host name (or a host name from an address) from
a network service called the *Domain Name System (DNS)*. DNS is a distributed network of servers
providing mappings of host names to IP addresses. In order for name service to work, a host needs
to be pointed at a nameserver. This nameserver does not need to be on the same subnet; it just
needs to be reachable by the host. This is typically configured through DHCP or a static setting in a
file called **/etc/resolv.conf**. Later sections of this chapter will discuss how to configure name
resolution.

### VALIDATING NETWORK CONFIGURATION ###

#### GATHERING NETWORK INTERFACE INFORMATION ####

**Identifying Network Interfaces**

The **ip link** command will list all network interfaces available on your system:

``` bash
[user@host ~]$ ip link show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT
 group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT
 group default qlen 1000
    link/ether 52:54:00:00:00:0a brd ff:ff:ff:ff:ff:ff
3: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT
 group default qlen 1000
    link/ether 52:54:00:00:00:1e brd ff:ff:ff:ff:ff:ff
```

In the preceding example, the server has three network interfaces: lo, which is the loopback device
that is connected to the server itself, and two Ethernet interfaces, ens3 and ens4.

To configure each network interface correctly, you need to know which one is connected to which
network. In many cases, you will know the MAC address of the interface connected to each network,
either because it is physically printed on the card or server, or because it is a virtual machine and
you know how it is configured. The MAC address of the device is listed after **link/ether** for each
interface. So you know that the network card with the MAC address 52:54:00:00:00:0a is the
network interface ens3.

**Displaying IP Addresses**

Use the **ip** command to view device and address information. A single network interface can have
multiple IPv4 or IPv6 addresses.

``` bash
[user@host ~]$ ip addr show ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP
 qlen 1000
    
	link/ether 52:54:00:00:00:0b brd ff:ff:ff:ff:ff:ff
    
	inet 192.0.2.2/24 brd 192.0.2.255 scope global ens3
       valid_lft forever preferred_lft forever
    
	inet6 2001:db8:0:1:5054:ff:fe00:b/64 scope global
       valid_lft forever preferred_lft forever
    
	inet6 fe80::5054:ff:fe00:b/64 scope link
       valid_lft forever preferred_lft forever
```

Up: An active interface is UP.

link/ether:	The link/ether line specifies the hardware (MAC) address of the device.

inet: The inet line shows an IPv4 address, its network prefix length, and scope.

inet6: The inet6 line shows an IPv6 address, its network prefix length, and scope. This address is of
global scope and is normally used.

inet6: This inet6 line shows that the interface has an IPv6 address of link scope that can only be
used for communication on the local Ethernet link.

**Displaying Performance Statistics**

The **ip** command may also be used to show statistics about network performance. Counters for
each network interface can be used to identify the presence of network issues. The counters record
statistics for things like the number of received (RX) and transmitted (TX) packets, packet errors,
and packets that were dropped.

``` bash
[user@host ~]$ ip -s link show ens3
2: ens3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen
 1000
link/ether 52:54:00:00:00:0a brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast
    269850     2931     0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    300556     3250     0       0       0       0
```

### CHECKING CONNECTIVITY BETWEEN HOSTS ###

The **ping** command is used to test connectivity. The command continues to run until **Ctrl+C** is
pressed unless options are given to limit the number of packets sent.

``` bash
[user@host ~]$ ping -c3 192.0.2.254
PING 192.0.2.1 (192.0.2.254) 56(84) bytes of data.
64 bytes from 192.0.2.254: icmp_seq=1 ttl=64 time=4.33 ms
64 bytes from 192.0.2.254: icmp_seq=2 ttl=64 time=3.48 ms
64 bytes from 192.0.2.254: icmp_seq=3 ttl=64 time=6.83 ms

--- 192.0.2.254 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 3.485/4.885/6.837/1.424 ms
```

The **ping6** command is the IPv6 version of **ping** in Red Hat Enterprise Linux. It communicates
over IPv6 and takes IPv6 addresses, but otherwise works like **ping**.

``` bash
[user@host ~]$ ping6 2001:db8:0:1::1
PING 2001:db8:0:1::1(2001:db8:0:1::1) 56 data bytes
64 bytes from 2001:db8:0:1::1: icmp_seq=1 ttl=64 time=18.4 ms
64 bytes from 2001:db8:0:1::1: icmp_seq=2 ttl=64 time=0.178 ms
64 bytes from 2001:db8:0:1::1: icmp_seq=3 ttl=64 time=0.180 ms
^C

--- 2001:db8:0:1::1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.178/6.272/18.458/8.616 ms
[user@host ~]$ 
```

When you ping link-local addresses and the link-local all-nodes multicast group (ff02::1),
the network interface to use must be specified explicitly with a scope zone identifier (such as
ff02::1%ens3). If this is left out, the error connect: Invalid argument is displayed.

Pinging ff02::1 can be useful for finding other IPv6 nodes on the local network.

``` bash
[user@host ~]$ ping6 ff02::1%ens4
PING ff02::1%ens4(ff02::1) 56 data bytes
64 bytes from fe80::78cf:7fff:fed2:f97b: icmp_seq=1 ttl=64 time=22.7 ms
64 bytes from fe80::f482:dbff:fe25:6a9f: icmp_seq=1 ttl=64 time=30.1 ms (DUP!)
64 bytes from fe80::78cf:7fff:fed2:f97b: icmp_seq=2 ttl=64 time=0.183 ms
64 bytes from fe80::f482:dbff:fe25:6a9f: icmp_seq=2 ttl=64 time=0.231 ms (DUP!)
^C
--- ff02::1%ens4 ping statistics ---
2 packets transmitted, 2 received, +2 duplicates, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.183/13.320/30.158/13.374 ms
[user@host ~]$ ping6 -c 1 fe80::f482:dbff:fe25:6a9f%ens4
PING fe80::f482:dbff:fe25:6a9f%ens4(fe80::f482:dbff:fe25:6a9f) 56 data bytes
64 bytes from fe80::f482:dbff:fe25:6a9f: icmp_seq=1 ttl=64 time=22.9 ms
--- fe80::f482:dbff:fe25:6a9f%ens4 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 22.903/22.903/22.903/0.000 ms
```

Remember that IPv6 link-local addresses can be used by other hosts on the same link, just like
normal addresses.

``` bash
[user@host ~]$ ssh fe80::f482:dbff:fe25:6a9f%ens4
user@fe80::f482:dbff:fe25:6a9f%ens4's password:
Last login: Thu Jun  5 15:20:10 2014 from host.example.com
[user@server ~]$ 
```

### TROUBLESHOOTING ROUTING ###

Network routing is complex, and sometimes traffic does not behave as you might have expected.
Here are some useful diagnosis tools.

**Displaying the Routing Table**

Use the ip command with the route option to show routing information.

``` bash
[user@host ~]$ ip route
default via 192.0.2.254 dev ens3 proto static metric 1024
192.0.2.0/24 dev ens3 proto kernel scope link src 192.0.2.2
10.0.0.0/8 dev ens4 proto kernel scope link src 10.0.0.11
```

This shows the IPv4 routing table. All packets destined for the 10.0.0.0/8 network are sent
directly to the destination through the device ens4. All packets destined for the 192.0.2.0/24
network are sent directly to the destination through the device ens3. All other packets are sent to
the default router located at 192.0.2.254, and also through device ens3.

Add the **-6** option to show the IPv6 routing table:

``` bash
[user@host ~]$ ip -6 route
unreachable ::/96 dev lo  metric 1024  error -101
unreachable ::ffff:0.0.0.0/96 dev lo  metric 1024  error -101
2001:db8:0:1::/64 dev ens3  proto kernel  metric 256
unreachable 2002:a00::/24 dev lo  metric 1024  error -101
unreachable 2002:7f00::/24 dev lo  metric 1024  error -101
unreachable 2002:a9fe::/32 dev lo  metric 1024  error -101
unreachable 2002:ac10::/28 dev lo  metric 1024  error -101
unreachable 2002:c0a8::/32 dev lo  metric 1024  error -101
unreachable 2002:e000::/19 dev lo  metric 1024  error -101
unreachable 3ffe:ffff::/32 dev lo  metric 1024  error -101
fe80::/64 dev ens3  proto kernel  metric 256
default via 2001:db8:0:1::ffff dev ens3  proto static  metric 1024
```

In this example, ignore the unreachable routes, which point at unused networks. That leaves three
routes:

1. The 2001:db8:0:1::/64 network, using the ens3 interface (which presumably has an
address on that network).

2. The fe80::/64 network, using the ens3 interface, for the link-local address. On a system
with multiple interfaces, there is a route to fe80::/64 out each interface for each link-local
address.

3. A default route to all networks on the IPv6 Internet (the ::/0 network) that do not have a
more specific route on the system, through the router at 2001:db8:0:1::ffff, reachable
with the ens3 device.

**Tracing Routes Taken by Traffic**

To trace the path that network traffic takes to reach a remote host through multiple routers, use
either **traceroute** or **tracepath**. This can identify whether an issue is with one of your routers
or an intermediate one. Both commands use UDP packets to trace a path by default; however,
many networks block UDP and ICMP traffic. The **traceroute** command has options to trace the
path with UDP (default), ICMP (**-I**), or TCP (**-T**) packets. Typically, however, the **traceroute**
command is not installed by default.

``` bash
[user@host ~]$ tracepath access.redhat.com
...output omitted...
 4:  71-32-28-145.rcmt.qwest.net                          48.853ms asymm  5
 5:  dcp-brdr-04.inet.qwest.net                          100.732ms asymm  7
 6:  206.111.0.153.ptr.us.xo.net                          96.245ms asymm  7
 7:  207.88.14.162.ptr.us.xo.net                          85.270ms asymm  8
 8:  ae1d0.cir1.atlanta6-ga.us.xo.net                     64.160ms asymm  7
 9:  216.156.108.98.ptr.us.xo.net                        108.652ms
10:  bu-ether13.atlngamq46w-bcr00.tbone.rr.com           107.286ms asymm 12
...output omitted...
```

Each line in the output of **tracepath** represents a router or hop that the packet passes through
between the source and the final destination. Additional information is provided as available,
including the round trip timing (RTT) and any changes in the maximum transmission unit (MTU)
size. The asymm indication means traffic reached that router and returned from that router using
different (asymmetric) routes. The routers shown are the ones used for outbound traffic, not the
return traffic.

The **tracepath6** and **traceroute -6** commands are the equivalent to **tracepath** and
**traceroute** for IPv6.

``` bash
[user@host ~]$ tracepath6 2001:db8:0:2::451
 1?: [LOCALHOST]                        0.091ms pmtu 1500
 1:  2001:db8:0:1::ba                   0.214ms
 2:  2001:db8:0:1::1                    0.512ms
 3:  2001:db8:0:2::451      0.559ms reached
     Resume: pmtu 1500 hops 3 back 3
```

### TROUBLESHOOTING PORTS AND SERVICES ###

TCP services use sockets as end points for communication and are made up of an IP address,
protocol, and port number. Services typically listen on standard ports while clients use a random
available port. Well-known names for standard ports are listed in the **/etc/services** file.

The **ss** command is used to display socket statistics. The **ss** command is meant to replace the
older tool **netstat**, part of the net-tools package, which may be more familiar to some system
administrators but which is not always installed.

``` bash
[user@host ~]$ ss -ta
State      Recv-Q Send-Q      Local Address:Port          Peer Address:Port
LISTEN     0      128                     *:sunrpc                   *:*
LISTEN     0      128                     *:ssh                      *:*
LISTEN     0      100             127.0.0.1:smtp                     *:*
LISTEN     0      128                     *:36889                    *:*
ESTAB      0      0           172.25.250.10:ssh         172.25.254.254:59392
LISTEN     0      128                    :::sunrpc                  :::*
LISTEN     0      128                    :::ssh                     :::*
LISTEN     0      100                   ::1:smtp                    :::*
LISTEN     0      128                    :::34946                   :::*
```

`*:ssh` -> The port used for SSH is listening on all IPv4 addresses. The “*” is used to represent “all”
when referencing IPv4 addresses or ports.

`127.0.0.1:smtp` -> The port used for SMTP is listening on the 127.0.0.1 IPv4 loopback interface.

`172.25.250.10:ssh` -> The established SSH connection is on the 172.25.250.10 interface and originates from a
system with an address of 172.25.254.254.

`:::ssh` -> The port used for SSH is listening on all IPv6 addresses. The “::” syntax is used to represent all
IPv6 interfaces.

`::1:smtp` -> The port used for SMTP is listening on the ::1 IPv6 loopback interface.

**Options for ss and netstat**

| OPTION    | DESCRIPTION                                             |
|:----------|:--------------------------------------------------------|
| '-n'      | Show numbers instead of names for interfaces and ports. |
| '-t'      | Show TCP sockets.                                       |
| '-u'      | Show UDP sockets.                                       |
| '-l'      | Show only listening sockets.                            |
| '-a'      | Show all (listening and established) sockets.           |
| '-p'      | Show the process using the sockets.                     |
| '-A inet' | Display active connections (but not listening sockets) for the inet address family. That is, ignore local UNIX domain sockets. For ss, both IPv4 and IPv6 connections are displayed. For netstat, only IPv4 connections are displayed. (netstat -A inet6 displays IPv6 connections, and netstat -46 displays IPv4 and IPv6 at the same time.)|

### CONFIGURING NETWORKING FROM THE COMMAND LINE ###

#### DESCRIBING NETWORKMANAGER CONCEPTS ####

NetworkManager is a daemon that monitors and manages network settings. In addition to the
daemon, there is a GNOME Notification Area applet providing network status information.

Command-line and graphical tools talk to NetworkManager and save configuration files in the **/
etc/sysconfig/network-scripts** directory.

- A device is a network interface.

- A connection is a collection of settings that can be configured for a device.

- Only one connection can be active for any one device at a time. Multiple connections may exist
for use by different devices or to allow a configuration to be altered for the same device. If you
need to temporarily change networking settings, instead of changing the configuration of a
connection, you can change which connection is active for a device. For example, a device for a
wireless network interface on a laptop might use different connections for the wireless network
at a work site and for the wireless network at home.

- Each connection has a name or ID that identifies it.

- The **nmcli** utility is used to create and edit connection files from the command line.

#### VIEWING NETWORKING INFORMATION ####

The **nmcli dev status** command displays the status of all network devices:

``` bash
[user@host ~]$ nmcli dev status
DEVICE  TYPE      STATE         CONNECTION
eno1    ethernet  connected     eno1
ens3    ethernet  connected     static-ens3
eno2    ethernet  disconnected  --
lo      loopback  unmanaged     --
```

The **nmcli con show** command displays a list of all connections. To list only the active
connections, add the **--active** option.

``` bash
[user@host ~]$ nmcli con show
NAME         UUID                                  TYPE            DEVICE
eno2         ff9f7d69-db83-4fed-9f32-939f8b5f81cd  802-3-ethernet  --
static-ens3  72ca57a2-f780-40da-b146-99f71c431e2b  802-3-ethernet  ens3
eno1         87b53c56-1f5d-4a29-a869-8a7bdaf56dfa  802-3-ethernet  eno1
[user@host ~]$ nmcli con show --active
NAME         UUID                                  TYPE            DEVICE
static-ens3  72ca57a2-f780-40da-b146-99f71c431e2b  802-3-ethernet  ens3
eno1         87b53c56-1f5d-4a29-a869-8a7bdaf56dfa  802-3-ethernet  eno1
```

#### ADDING A NETWORK CONNECTION ####

The **nmcli con add** command is used to add new network connections. The following example
**nmcli con add** commands assume that the name of the network connection being added is not
already in use.

The following command adds a new connection named eno2 for the interface eno2, which
gets IPv4 networking information using DHCP and autoconnects on startup. It also gets IPv6
networking settings by listening for router advertisements on the local link. The name of the
configuration file is based on the value of the **con-name** option, eno2, and is saved to the **/etc/
sysconfig/network-scripts/ifcfg-eno2** file.

``` bash
[root@host ~]# nmcli con add con-name eno2 type ethernet ifname eno2
```

The next example creates an eno2 connection for the eno2 device with a static IPv4 address, using
the IPv4 address and network prefix 192.168.0.5/24 and default gateway 192.168.0.254,
but still autoconnects at startup and saves its configuration into the same file. Due to screen size
limitations, terminate the first line with a shell \ escape and complete the command on the next
line.

``` bash
[root@host ~]# nmcli con add con-name eno2 type ethernet ifname eno2 \
> ip4 192.168.0.5/24 gw4 192.168.0.254
```

This final example creates an eno2 connection for the eno2 device with static IPv6 and IPv4
addresses, using the IPv6 address and network prefix 2001:db8:0:1::c000:207/64
and default IPv6 gateway 2001:db8:0:1::1, and the IPv4 address and network prefix
192.0.2.7/24 and default IPv4 gateway 192.0.2.1, but still autoconnects at startup and saves
its configuration into **/etc/sysconfig/network-scripts/ifcfg-eno2**. Due to screen size
limitations, terminate the first line with a shell \ escape and complete the command on the next
line.

``` bash
[root@host ~]# nmcli con add con-name eno2 type ethernet ifname eno2 \
> ip6 2001:db8:0:1::c000:207/64 gw6 2001:db8:0:1::1 ip4 192.0.2.7/24 gw4 192.0.2.1
```

#### CONTROLLING NETWORK CONNECTIONS ####

The **nmcli con up name** command activates the connection name on the network interface it
is bound to. Note that the command takes the name of a connection, not the name of the network
interface. Remember that the **nmcli con show** command displays the names of all available
connections.

``` bash
[root@host ~]# nmcli con up static-ens3
```

The **nmcli dev disconnect device** command disconnects the network interface device and
brings it down. This command can be abbreviated **nmcli dev dis device**:

``` bash
[root@host ~]# nmcli dev dis ens3
```

!!! danger "IMPORTANT"

	Use **nmcli dev dis device** to deactivate a network interface.
	
	The **nmcli con down name** command is normally not the best way to deactivate
	a network interface because it brings down the connection. However, by default,
	most wired system connections are configured with **autoconnect** enabled. This
	activates the connection as soon as its network interface is available. Since the
	connection's network interface is still available, **nmcli con down name** brings the
	interface down, but then NetworkManager immediately brings it up again unless the
	connection is entirely disconnected from the interface.

### MODIFYING NETWORK CONNECTION SETTINGS ###

NetworkManager connections have two kinds of settings. There are static connection properties,
configured by the administrator and stored in the configuration files in **/etc/sysconfig/
network-scripts/ifcfg-***. There may also be active connection data, which the connection
gets from a DHCP server and which are not stored persistently.

To list the current settings for a connection, run the **nmcli con show name** command,
where name is the name of the connection. Settings in lowercase are static properties that the
administrator can change. Settings in all caps are active settings in temporary use for this instance
of the connection.

``` bash
[root@host ~]# nmcli con show static-ens3
connection.id:                          static-ens3
connection.uuid:                        87b53c56-1f5d-4a29-a869-8a7bdaf56dfa
connection.interface-name:              --
connection.type:                        802-3-ethernet
connection.autoconnect:                 yes
connection.timestamp:                   1401803453
connection.read-only:                   no
connection.permissions:
connection.zone:                        --
connection.master:                      --
connection.slave-type:                  --
connection.secondaries:
connection.gateway-ping-timeout:        0
802-3-ethernet.port:                    --
802-3-ethernet.speed:                   0
802-3-ethernet.duplex:                  --
802-3-ethernet.auto-negotiate:          yes
802-3-ethernet.mac-address:             CA:9D:E9:2A:CE:F0
802-3-ethernet.cloned-mac-address:      --
802-3-ethernet.mac-address-blacklist:
802-3-ethernet.mtu:                     auto
802-3-ethernet.s390-subchannels:
802-3-ethernet.s390-nettype:            --
802-3-ethernet.s390-options:
ipv4.method:                            manual
ipv4.dns:                               192.168.0.254
ipv4.dns-search:                        example.com
ipv4.addresses:                         { ip = 192.168.0.2/24, gw =
 192.168.0.254 }
ipv4.routes:
ipv4.ignore-auto-routes:                no
ipv4.ignore-auto-dns:                   no
ipv4.dhcp-client-id:                    --
ipv4.dhcp-send-hostname:                yes
ipv4.dhcp-hostname:                     --
ipv4.never-default:                     no
ipv4.may-fail:                          yes
ipv6.method:                            manual
ipv6.dns:                               2001:4860:4860::8888
ipv6.dns-search:                        example.com
ipv6.addresses:                         { ip = 2001:db8:0:1::7/64, gw =
 2001:db8:0:1::1 }
ipv6.routes:
ipv6.ignore-auto-routes:                no
ipv6.ignore-auto-dns:                   no
ipv6.never-default:                     no
ipv6.may-fail:                          yes
ipv6.ip6-privacy:                       -1 (unknown)
ipv6.dhcp-hostname:                     --
...output omitted...
```

The **nmcli con mod name** command is used to change the settings for a connection. These
changes are also saved in the **/etc/sysconfig/network-scripts/ifcfg-name** file for the
connection. Available settings are documented in the nm-settings(5) man page.

To set the IPv4 address to 192.0.2.2/24 and default gateway to 192.0.2.254 for the
connection static-ens3:

``` bash
[root@host ~]# nmcli con mod static-ens3 ipv4.addresses "192.0.2.2/24 192.0.2.254"
```

To set the IPv6 address to 2001:db8:0:1::a00:1/64 and default gateway to
2001:db8:0:1::1 for the connection static-ens3:

``` bash
[root@host ~]# nmcli con mod static-ens3 ipv6.address "2001:db8:0:1::a00:1/64
 2001:db8:0:1::1"
```

!!! danger "IMPORTANT"

	If a connection that gets its IPv4 information from a DHCPv4 server is being
	changed to get it from static configuration files only, the setting **ipv4.method**
	should also be changed from auto to manual.
	
	Likewise, if a connection that gets its IPv6 information by SLAAC or a DHCPv6
	server is being changed to get it from static configuration files only, the setting
	**ipv6.method** should also be changed from auto or dhcp to manual.
	
	Otherwise, the connection may hang or not complete successfully when it is
	activated, or it may get an IPv4 address from DHCP or an IPv6 address from
	DHCPv6 or SLAAC in addition to the static address.

A number of settings may have multiple values. A specific value can be added to the list or deleted
from the list for a setting by adding a + or - symbol to the start of the setting name.

#### DELETING A NETWORK CONNECTION ####

The **nmcli con del name** command deletes the connection named name from the system,
disconnecting it from the device and removing the file **/etc/sysconfig/network-scripts/
ifcfg-name**.

``` bash
[root@host ~]# nmcli con del static-ens3
```

#### WHO CAN MODIFY NETWORK SETTINGS? ####

The root user can make any necessary network configuration changes with **nmcli**.

However, regular users that are logged in on the local console can also make many network
configuration changes to the system. They have to log in at the system's keyboard to either a text-
based virtual console or the graphical desktop environment to get this control. The logic behind
this is that if someone is physically present at the computer's console, it's likely being used as a
workstation or laptop and they may need to configure, activate, and deactivate wireless or wired
network interfaces at will. By contrast, if the system is a server in the datacenter, generally the only
users logging in locally to the machine itself should be administrators.

Regular users that log in using **ssh** do not have access to change network permissions without
becoming root.

You can use the **nmcli gen permissions** command to see what your current permissions are.

#### SUMMARY OF COMMANDS ####

The following table is a list of key **nmcli** commands discussed in this section.

| COMMAND                     | PURPOSE                                                                        |
|:----------------------------|:-------------------------------------------------------------------------------|
| nmcli dev status            | Show the NetworkManager status of all network interfaces.                      |
| nmcli con show              | List all connections.                                                          |
| nmcli con show name         | List the current settings for the connection name.                             |
| nmcli con add con-name name | Add a new connection named name.                                               |
| nmcli con mod name          | Modify the connection name.                                                    |
| nmcli con reload            | Reload the configuration files (useful after they have been edited by hand).   |
| nmcli con up name           | Activate the connection name.                                                  |
| nmcli dev dis dev           | Deactivate and disconnect the current connection on the network interface dev. |
| nmcli con del name          | Delete the connection name and its configuration file.                         |

### EDITING NETWORK CONFIGURATION FILES ###

#### DESCRIBING CONNECTION CONFIGURATION FILES ####

By default, changes made with **nmcli con mod name** are automatically saved to **/etc/
sysconfig/network-scripts/ifcfg-name**. That file can also be manually edited with a text
editor. After doing so, run nmcli con reload so that NetworkManager reads the configuration
changes.

For backward-compatibility reasons, the directives saved in that file have different names and
syntax than the nm-settings(5) names. The following table maps some of the key setting names
to **ifcfg-*** directives.

**Comparison of nm-settings and ifcfg-* Directives**

| NMCLI CON MOD                               | IFCFG-* FILE                                      | EFFECT                                                                                                                                                                                                        |
|:--------------------------------------------|:--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ipv4.method manual                          | BOOTPROTO=none                                    | IPv4 addresses configured statically.                                                                                                                                                                         |
| ipv4.method auto                            | BOOTPROTO=dhcp                                    | Looks for configuration settings from a DHCPv4 server. If static addresses are also set, will not bring those up until we have information from DHCPv4.                                                       |
| ipv4.addresses "192.0.2.1/24 192.0.2.254"   | IPADDR0=192.0.2.1 PREFIX0=24 GATEWAY0=192.0.2.254 | Sets static IPv4 address, network prefix, and default gateway. If more than one is set for the connection, then instead of 0, the ifcfg-* directives end with 1, 2, 3 and so on.                              |
| ipv4.dns 8.8.8.8                            | DNS0=8.8.8.8                                      | Modify /etc/resolv.conf to use this nameserver.                                                                                                                                                               |
| ipv4.dns-search example.com                 | DOMAIN=example.com                                | Modify /etc/resolv.conf to use this domain in the search directive.                                                                                                                                           |
| ipv4.ignore-auto-dns true                   | PEERDNS=no                                        | Ignore DNS server information from the DHCP server.                                                                                                                                                           |
| ipv6.method manual                          | IPV6_AUTOCONF=no                                  | IPv6 addresses configured statically.                                                                                                                                                                         |
| ipv6.method auto                            | IPV6_AUTOCONF=yes                                 | Configures network settings using SLAAC from router advertisements.                                                                                                                                           |
| ipv6.method dhcp                            | IPV6_AUTOCONF=no DHCPV6C=yes                      | Configures network settings by using DHCPv6, but not SLAAC.                                                                                                                                                   |
| ipv6.addresses "2001:db8::a/64 2001:db8::1" | IPV6ADDR=2001:db8::a/64 IPV6_DEFAULTGW=2001:db8   | Sets static IPv6 address, network prefix, and default gateway. If more than one address is set for the connection, IPV6_SECONDARIES takes a double-quoted list of space-delimited address/prefix definitions. |
| ipv6.dns ...                                | DNS0= ...                                         | Modify /etc/resolv.conf to use this nameserver. Exactly the same as IPv4.                                                                                                                                     |
| ipv6.dns-search example.com                 | DOMAIN=example.com                                | Modify /etc/resolv.conf to use this domain in the search directive. Exactly the same as IPv4.                                                                                                                 |
| ipv6.ignore-auto-dns true                   | IPV6_PEERDNS=no                                   | Ignore DNS server information from the DHCP server.                                                                                                                                                           |
| connection.autoconnect yes                  | ONBOOT=yes                                        | Automatically activate this connection at boot.                                                                                                                                                               |
| connection.id ens3                          | NAME=ens3                                         | The name of this connection.                                                                                                                                                                                  |
| connection.interface-name ens3              | DEVICE=ens3                                       | The connection is bound to the network interface with this name.                                                                                                                                              |
| 802-3-ethernet.mac-address . . .            | HWADDR= ...                                       | The connection is bound to the network interface with this MAC address.                                                                                                                                       |

### MODIFYING NETWORK CONFIGURATION ###

It is also possible to configure the network by directly editing the connection configuration files.
Connection configuration files control the software interfaces for individual network devices. These
files are usually named /etc/sysconfig/network-scripts/ifcfg-name, where name refers to the name of the device or connection that the configuration file controls. The following are

**IPv4 Configuration Options for ifcfg File**

| STATIC                  | DYNAMIC        | EITHER             |
|:------------------------|:---------------|--------------------|
| BOOTPROTO=none          | BOOTPROTO=dhcp | DEVICE=ens3        |
| IPADDR0=172.25.250.10   |                | NAME="static-ens3" |
| PREFIX0=24              |                | ONBOOT=yes         |
| GATEWAY0=172.25.250.254 |                | UUID=f3e8(...)ad3e |
| DEFROUTE=yes            |                | USERCTL=yes        |
| DNS1=172.25.254.254     |                |                    |

In the static settings, variables for IP address, prefix, and gateway have a number at the end. This
allows multiple sets of values to be assigned to the interface. The DNS variable also has a number
used to specify the order of lookup when multiple servers are specified.

After modifying the configuration files, run **nmcli con reload** to make NetworkManager read
the configuration changes. The interface still needs to be restarted for changes to take effect.

``` bash
[root@host ~]# nmcli con reload
[root@host ~]# nmcli con down "static-ens3"
[root@host ~]# nmcli con up "static-ens3"
```

### CONFIGURING HOST NAMES AND NAME RESOLUTION ###

#### CHANGING THE SYSTEM HOST NAME ####

The **hostname** command displays or temporarily modifies the system's fully qualified host name.

``` bash
[root@host ~]# hostname
host@example.com
```

A static host name may be specified in the **/etc/hostname** file. The **hostnamectl** command
is used to modify this file and may be used to view the status of the system's fully qualified host
name. If this file does not exist, the host name is set by a reverse DNS query once the interface has
an IP address assigned.

``` bash
[root@host ~]# hostnamectl set-hostname host@example.com
[root@host ~]# hostnamectl status
   Static hostname: host.example.com
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 73ab164e278e48be9bf80e80714a8cd5
           Boot ID: 6b1cbc4177164ef58c0e9ed4adb2904f
    Virtualization: kvm
  Operating System: Red Hat Enterprise Linux 8.0 beta (Ootpa)
       CPE OS Name: cpe:/o:redhat:enterprise_linux:8.0:beta
            Kernel: Linux 4.18.0-60.el8.x86_64
      Architecture: x86-64
[root@host ~]# cat /etc/hostname
host@example.com
```

!!! danger "IMPORTANT"

	In Red Hat Enterprise Linux 7 and later, the static host name is stored in /etc/
	hostname. Red Hat Enterprise Linux 6 and earlier stores the host name as a
	variable in the /etc/sysconfig/network file.

#### CONFIGURING NAME RESOLUTION ####

The stub resolver is used to convert host names to IP addresses or the reverse. It determines where
to look based on the configuration of the **/etc/nsswitch.conf** file. By default, the contents of
the **/etc/hosts** file are checked first.

``` bash
[root@host ~]# cat /etc/hosts
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
172.25.254.254 classroom.example.com
172.25.254.254 content.example.com
```

The **getent hosts hostname** command can be used to test host name resolution using the **/
etc/hosts** file.

If an entry is not found in the **/etc/hosts** file, by default the stub resolver tries to look up the
hostname by using a DNS nameserver. The **/etc/resolv.conf** file controls how this query is
performed:

- search: a list of domain names to try with a short host name. Both this and domain should not
be set in the same file; if they are, the last instance wins. See resolv.conf(5) for details.

- nameserver: the IP address of a nameserver to query. Up to three nameserver directives may
be given to provide backups if one is down.

``` bash
[root@host ~]# cat /etc/resolv.conf
# Generated by NetworkManager
domain example.com
search example.com
nameserver 172.25.254.254
```

NetworkManager updates the **/etc/resolv.conf** file using DNS settings in the connection
configuration files. Use the **nmcli** to modify the connections.

``` bash
[root@host ~]# nmcli con mod ID ipv4.dns IP
[root@host ~]# nmcli con down ID
[root@host ~]# nmcli con up ID
[root@host ~]# cat /etc/sysconfig/network-scripts/ifcfg-ID
...output omitted...
DNS1=8.8.8.8
...output omitted...
```

The default behavior of **nmcli con mod ID ipv4.dns IP** is to replace any previous DNS
settings with the new IP list provided. A + or - symbol in front of the **ipv4.dns** argument adds or
removes an individual entry.

``` bash
[root@host ~]# nmcli con mod ID +ipv4.dns IP
```

To add the DNS server with IPv6 IP address 2001:4860:4860::8888 to the list of nameservers
to use with the connection static-ens3:

``` bash
[root@host ~]# nmcli con mod static-ens3 +ipv6.dns 2001:4860:4860::8888
```

!!! info "NOTE"

	Static IPv4 and IPv6 DNS settings all end up as nameserver directives in /etc/
	resolv.conf. You should ensure that there is, at minimum, an IPv4-reachable
	name server listed (assuming a dual-stack system). It is better to have at least one
	name server using IPv4 and a second using IPv6 in case you have network issues
	with either your IPv4 or IPv6 networking.

**Testing DNS Name Resolution**

The **host HOSTNAME** command can be used to test DNS server connectivity.

``` bash
[root@host ~]# host classroom.example.com
classroom.example.com has address 172.25.254.254
[root@host ~]# host 172.25.254.254
254.254.25.172.in-addr.arpa domain name pointer classroom.example.com.
```

!!! danger "IMPORTANT"

	DHCP automatically rewrites the /etc/resolv.conf file as interfaces are started
	unless you specify PEERDNS=no in the relevant interface configuration files. Set this
	using the nmcli command.
	
	``` bash
	[root@host ~]# nmcli con mod "static-ens3" ipv4.ignore-auto-dns yes
	```

### SUMMARY ###

In this chapter, you learned:

- The TCP/IP network model is a simplified, four-layered set of abstractions that describes how
different protocols interoperate in order for computers to send traffic from one machine to
another over the Internet.

- IPv4 is the primary network protocol used on the Internet today. IPv6 is intended as an eventual
replacement for the IPv4 network protocol. By default, Red Hat Enterprise Linux operates in
dual-stack mode, using both protocols in parallel.

- NetworkManager is a daemon that monitors and manages network configuration.

- The **nmcli** command is a command-line tool for configuring network settings with
NetworkManager.

- The system's static host name is stored in the **/etc/hostname** file. The **hostnamectl**
command is used to modify or view the status of the system's host name and related settings.
The **hostname** command displays or temporarily modifies the system's host name.
