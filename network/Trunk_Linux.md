Need to finish this file

---

# VLAN TRUNK

Method

> apt-get install vlan

This provides the command vconfig, which you will not need to invoke directly, but which is needed by ifup and ifdown when using VLANs.

Next, add an interface definition for each VLAN to /etc/network/interfaces. The VLAN interface names must follow one of the naming conventions supported by vconfig. The one used and recommended here is of the form ethx.y, where ethx is the physical interface name and y is the VLAN number.
Apart from the special form of the interface name, the definitions are identical to those used for physical Ethernet interfaces:
```
auto eth0.2
iface eth0.2 inet static
  address 192.168.2.1
  netmask 255.255.255.0

auto eth0.3
iface eth0.3 inet static
  address 192.168.3.1
  netmask 255.255.255.0
```
Finally, bring the interfaces up in the normal way using ifup:
```
ifup eth0.2
ifup eth0.3
```
So on the relay :
```
allow-hotplug enx000ec6bc9ef6
auto enx000ec6bc9ef6
iface enx000ec6bc9ef6 inet static
        address 172.30.4.220
        netmask 255.255.255.0
        gateway 172.30.4.1

# The primary network interface
allow-hotplug eno1
auto eno1
iface eno1 inet static
        address 10.1.1.1
        netmask 255.255.255.0

# Trunk interface 12 -> 24
auto eno1.12
iface eno1.12 inet static
        address 10.1.12.1
        netmask 255.255.255.0

auto eno1.13
iface eno1.13 inet static
        address 10.1.13.1
        netmask 255.255.255.0


auto eno1.14
iface eno1.14 inet static
        address 10.1.14.1
        netmask 255.255.255.0

auto eno1.15
iface eno1.15 inet static
        address 10.1.15.1
        netmask 255.255.255.0

auto eno1.16
iface eno1.16 inet static
        address 10.1.16.1
        netmask 255.255.255.0
        
auto eno1.17
iface eno1.17 inet static
        address 10.1.17.1
        netmask 255.255.255.0

auto eno1.18
iface eno1.18 inet static
        address 10.1.18.1
        netmask 255.255.255.0

auto eno1.19
iface eno1.19 inet static
        address 10.1.19.1
        netmask 255.255.255.0

auto eno1.20
iface eno1.20 inet static
        address 10.1.20.1
        netmask 255.255.255.0

auto eno1.21
iface eno1.21 inet static
        address 10.1.21.1
        netmask 255.255.255.0

auto eno1.22
iface eno1.22 inet static
        address 10.1.22.1
        netmask 255.255.255.0

auto eno1.23
iface eno1.23 inet static
        address 10.1.23.1
        netmask 255.255.255.0

auto eno1.24
iface eno1.24 inet static
        address 10.1.24.1
        netmask 255.255.255.0

```
