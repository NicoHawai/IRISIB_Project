

```
sudo iptables -t nat -A POSTROUTING -o enx000ec6bc9ef6 -j MASQUERADE
sudo sysctl net.ipv4.ip_forward=1
```

etc/network/interfaces

```
# USB-ETHERNET interface

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

DHCP relay

```
sudo apt install isc-dhcp-relay
```
/etc/default/isc-dhcp-relay

```
# What servers should the DHCP relay forward requests to?
SERVERS="172.30.4.221 172.30.4.222"

# On what interfaces should the DHCP relay (dhrelay) serve DHCP requests?
INTERFACES="enx000ec6bc9ef6 eno1.12 eno1.13 eno1.14 eno1.15 eno1.16 eno1.17 eno1.18 eno1.19 eno1.20 eno1.21 eno1.22 eno1.23 eno1.24"


# Additional options that are passed to the DHCP relay daemon?
OPTIONS=""
```
