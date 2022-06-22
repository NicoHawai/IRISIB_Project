# Configure DHCP server, failover & relay 

- [On the primary DHCP server](#on-the-primary-dhcp-server)
- [On the secondary DHCP server](#on-the-secondary-dhcp-server)
- [On the Relay server](#on-the-relay-server)
- [On the clients](#on-the-clients)
- [Remarks/Tips](#remarks-tips)

*Note, in our example, Alpha DHCP server = 172.30.4.221 (master), Beta DHCP server = 172.30.4.222 (slave), Relay server = 172.30.4.220*

## On the primary DHCP server

First install the daemon:
```
sudo apt update
sudo apt install isc-dhcp-server
```

Then in **/etc/default/isc-dhcp-server**  
Aim the network card:  

```
INTERFACESv4="eno1"
INTERFACESv6=""
```

Then in **/etc/dhcp/dhcpd.conf**  
Configure the server:

```
default-lease-time 86400;                       # Time in seconds 86400 = 24h
max-lease-time 172800;                          # Time in seconds 172800 = 48h
option domain-name-servers 1.1.1.1;             # Later, it will be our own DNS server     

failover peer "dhcp-failover" {
        primary;
        address 172.30.4.221;
        port 847;
        peer address 172.30.4.222;
        port 647;
        max-response-delay 60;
        max-unacked-updates 10;
        mclt 1800;
        split 128;
        load balance max seconds 5;
}

# DHCP Server own subnet, it is mandatory to have it, 
# even if we don't use it.
# ====================================================

subnet 172.30.4.0 netmask 255.255.255.0 {
}

# All the subnets
# ====================================================
# Waldek subnet
# -------------
# Reminder : ports 9,10,11 & 12 on the physical switch
#            = vlan 12 = subnet 12 = 10.1.12.0/24
# ====================================================
subnet 10.1.12.0 netmask 255.255.255.0 {
        option routers 10.1.12.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.12.50 10.1.12.250;
        }
}
# ====================================================
# Students subnets
# ----------------
# Reminder : 
# switch port 13 = vlan 13 = subnet 13 = 10.1.13.0/24
# switch port 14 = vlan 14 = subnet 14 = 10.1.14.0/24
# ... 
# switch port 24 = vlan 24 = subnet 24 = 10.1.24.0/24
# ====================================================

subnet 10.1.13.0 netmask 255.255.255.0 {
        option routers 10.1.13.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.13.50 10.1.13.250;
        }
}
subnet 10.1.14.0 netmask 255.255.255.0 {
        option routers 10.1.14.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.14.50 10.1.14.250;
        }
}
subnet 10.1.15.0 netmask 255.255.255.0 {
        option routers 10.1.15.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.15.50 10.1.15.250;
        }
}
subnet 10.1.16.0 netmask 255.255.255.0 {
        option routers 10.1.16.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.16.50 10.1.16.250;
        }
}
subnet 10.1.17.0 netmask 255.255.255.0 {
        option routers 10.1.17.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.17.50 10.1.17.250;
        }
}
subnet 10.1.18.0 netmask 255.255.255.0 {
        option routers 10.1.18.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.18.50 10.1.18.250;
        }
}
subnet 10.1.19.0 netmask 255.255.255.0 {
        option routers 10.1.19.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.19.50 10.1.19.250;
        }
}
subnet 10.1.20.0 netmask 255.255.255.0 {
        option routers 10.1.20.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.20.50 10.1.20.250;
        }
}
subnet 10.1.21.0 netmask 255.255.255.0 {
        option routers 10.1.21.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.21.50 10.1.21.250;
        }
}
subnet 10.1.22.0 netmask 255.255.255.0 {
        option routers 10.1.22.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.22.50 10.1.22.250;
        }
}
subnet 10.1.23.0 netmask 255.255.255.0 {
        option routers 10.1.23.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.23.50 10.1.23.250;
        }
}
subnet 10.1.24.0 netmask 255.255.255.0 {
        option routers 10.1.24.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.24.50 10.1.24.250;
        }
}
```
Adding **routes** to the relay server  
**/etc/network/interfaces**

```
up ip route add 10.1.12.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.13.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.14.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.15.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.16.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.17.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.18.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.19.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.20.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.21.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.22.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.23.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.24.0/24 via 172.30.4.220 dev eno1
```
Restart **isc-dhcp-server** & **networking-service**

```
sudo systemctl restart networking-service
sudo systemctl restart isc-dhcp-server
```

<u>That's it!</u>

---

## On the Secondary DHCP server

First install the daemon:
```
sudo apt update
sudo apt install isc-dhcp-server
```

Then in **/etc/default/isc-dhcp-server**  
Aim the network card:  

```
INTERFACESv4="eno1"
INTERFACESv6=""
```

Then in **/etc/dhcp/dhcpd.conf**  
Configure the server:

```
option domain-name-servers 1.1.1.1;
default-lease-time 86400;
max-lease-time 172800;

# ================================
# DHCP FAILOVER CONFIGURATION
# ================================

failover peer "dhcp-failover" {
        secondary;
        address 172.30.4.222;
        port 647;
        peer address 172.30.4.221;
        port 847;
        max-response-delay 60;
        max-unacked-updates 10;
        load balance max seconds 5;
}
# DHCP Server own subnet, it is mandatory to have it, 
# even if we don't use it.
# ====================================================

subnet 172.30.4.0 netmask 255.255.255.0 {
}

# All the subnets
# ====================================================
# Waldek subnet
# -------------
# Reminder : ports 9,10,11 & 12 on the physical switch
#            = vlan 12 = subnet 12 = 10.1.12.0/24
# ====================================================
subnet 10.1.12.0 netmask 255.255.255.0 {
        option routers 10.1.12.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.12.50 10.1.12.250;
        }
}
# ====================================================
# Students subnets
# ----------------
# Reminder : 
# switch port 13 = vlan 13 = subnet 13 = 10.1.13.0/24
# switch port 14 = vlan 14 = subnet 14 = 10.1.14.0/24
# ... 
# switch port 24 = vlan 24 = subnet 24 = 10.1.24.0/24
# ====================================================

subnet 10.1.13.0 netmask 255.255.255.0 {
        option routers 10.1.13.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.13.50 10.1.13.250;
        }
}
subnet 10.1.14.0 netmask 255.255.255.0 {
        option routers 10.1.14.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.14.50 10.1.14.250;
        }
}
subnet 10.1.15.0 netmask 255.255.255.0 {
        option routers 10.1.15.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.15.50 10.1.15.250;
        }
}
subnet 10.1.16.0 netmask 255.255.255.0 {
        option routers 10.1.16.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.16.50 10.1.16.250;
        }
}
subnet 10.1.17.0 netmask 255.255.255.0 {
        option routers 10.1.17.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.17.50 10.1.17.250;
        }
}
subnet 10.1.18.0 netmask 255.255.255.0 {
        option routers 10.1.18.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.18.50 10.1.18.250;
        }
}
subnet 10.1.19.0 netmask 255.255.255.0 {
        option routers 10.1.19.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.19.50 10.1.19.250;
        }
}
subnet 10.1.20.0 netmask 255.255.255.0 {
        option routers 10.1.20.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.20.50 10.1.20.250;
        }
}
subnet 10.1.21.0 netmask 255.255.255.0 {
        option routers 10.1.21.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.21.50 10.1.21.250;
        }
}
subnet 10.1.22.0 netmask 255.255.255.0 {
        option routers 10.1.22.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.22.50 10.1.22.250;
        }
}
subnet 10.1.23.0 netmask 255.255.255.0 {
        option routers 10.1.23.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.23.50 10.1.23.250;
        }
}
subnet 10.1.24.0 netmask 255.255.255.0 {
        option routers 10.1.24.1;
        option subnet-mask 255.255.255.0;
        pool{
                failover peer "dhcp-failover";
                range 10.1.24.50 10.1.24.250;
        }
}
```
Adding **routes** to the relay server  
**/etc/network/interfaces**

```
up ip route add 10.1.12.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.13.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.14.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.15.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.16.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.17.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.18.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.19.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.20.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.21.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.22.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.23.0/24 via 172.30.4.220 dev eno1
up ip route add 10.1.24.0/24 via 172.30.4.220 dev eno1
```
Restart **isc-dhcp-server** & **networking-service**

```
sudo systemctl restart networking-service
sudo systemctl restart isc-dhcp-server
```

<u>That's it!</u>

---

## On the Relay server

First make sure the [trunk](./Trunk_on_Linux.md) is done, and everything pings fine

```
sudo apt install isc-dhcp-relay
```
Then in **/etc/default/isc-dhcp-relay**

```
#forward request to the DHCP server
SERVERS="172.30.4.221 172.30.4.222"

# What interfaces need to be listened, do not forget the interface to the DHCP server
INTERFACES="enx000ec6bc9ef6 eno1.12 eno1.13 eno1.14 eno1.15 eno1.16 eno1.17 eno1.18 eno1.19 eno1.20 eno1.21 eno1.22 eno1.23 eno1.24"

OPTIONS=""
```
-> enx000ec6bc9ef6 is the interface to the DHCP server (here in my lab: a usb-eth adapter)  
-> eno1 is the interface to the switch which connects the Students

```
sudo systemctl restart isc-dhcp-relay
```
## On the clients

Nothing to do on the clients. Let the default dhcp client configuration.


## Remarks/Tips :

### On the DHCP server:

- Check the lease in CLI :

> sudo dhcp-lease-list

- Check leases in FILE :

> sudo vim /var/lib/dhcpd/dhcpd.leases

- Try/test the config file : 

> sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf

How-To flush the DHCP server lease cache
How-To flush the DHCP server lease cache

Check your /var/lib/dhcpd/dhcpd.leases file and check for the entry. It contains the list of all dhcp leases.

Step 1:- Stop dhcp server

> service dhcpd stop

Step 2:- delete the temporary file dhcpd.leases~:

> rm dhcpd.leases~

Step 3:- flush the lease cache dhcpd.leases:

> echo "" > dhcpd.leases

Step4:- Start DHCP Server

> service dhcpd start
