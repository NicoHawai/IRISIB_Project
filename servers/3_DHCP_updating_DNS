ToDo Link with Bind

# Make ISC DHCP & Bind9 work together, on Debian 11

- [The plan](#the-plan)
- [On the primary DHCP server](#on-the-primary-dhcp-server)
- [On the primary DNS server](#on-the-primary-dns-server)
- [On the secondary DHCP server](#on-the-secondary-dhcp-server)
- [On the secondary DNS server](#on-the-secondary-dns-server)

## The plan 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![](../globaldia.png)

## On the primary DHCP server

In **/etc/dhcp/dhcpd.conf**   
Add:

```
# ========= key for BIND9 ========
include "/etc/dhcp/rndc.key";
# ================================
```
Add:

```
ddns-updates on;
ddns-update-style standard;
```
Add :
```
#===========================
zone 12.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone brussels.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 13.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone paris.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 14.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone berlin.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 15.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone madrid.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 16.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone athens.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 17.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone rome.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 18.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone oslo.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 19.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone dublin.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 20.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone amsterdam.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 21.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone london.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 22.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone kiev.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 23.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone lisbon.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
#===========================
zone 24.1.10.in-addr.arpa. {
        primary 172.30.4.221;
        key rndc-key;
}
zone stockholm.lan. {
        primary 172.30.4.221;
        key rndc-key;
}
```
Then for each subnet, define the **option domain-name** of each client. the one you will see in /etc/resolv.conf

```
subnet 10.1.12.0 netmask 255.255.255.0 {
        option routers 10.1.12.1;
        option subnet-mask 255.255.255.0;
        option domain-name "brussels.lan";             // <- Here
        pool{
                failover peer "dhcp-failover";
                range 10.1.12.50 10.1.12.250;
        }
}
subnet 10.1.13.0 netmask 255.255.255.0 {
        option routers 10.1.13.1;
        option subnet-mask 255.255.255.0;
        option domain-name "paris.lan";             // <- Here
        pool{
                failover peer "dhcp-failover";
                range 10.1.13.50 10.1.13.250;
        }
}

...
...

subnet 10.1.24.0 netmask 255.255.255.0 {
        option routers 10.1.24.1;
        option subnet-mask 255.255.255.0;
        option domain-name "stockholm.lan";             // <- Here
        pool{
                failover peer "dhcp-failover";
                range 10.1.13.50 10.1.13.250;
        }
}

```
Reminder :

```
12  =   brussels
13  =   paris
14  =   berlin
15  =   madrid
16  =   athens
17  =   rome
18  =   oslo
19  =   dublin
20  =   amsterdam
21  =   london
22  =   kiev
23  =   lisbon
24  =   stockholm

```
Finally create the key (obviously it should be the same as the DNS servers):
```
echo "kj/n65!dfgeçUIor6\t5jil<32n1S!rert?qs6-e54rSF/HG6t=ui5lo:l46" > pass
sudo sha256sum pass | sudo tee /etc/dhcp/rndc.key
rm pass
```
Then modify the file we just made in :
```
key "rndc-key" {
    algorithm hmac-sha256;
    secret "0dfcqq1f61fdgd654654fgdf00b41904b351fd90bfe965fghfg489690d051";
};
```
Then change the access mode and restart isc-dhcp-server.
```
sudo chmod 640 /etc/bind/rndc.key
sudo systemctl restart isc-dhcp-server
```

---

## On the primary DNS server

Go to **/etc/bind/named.conf.acl** to defines DHCP servers addresses. Add:  

```
acl allow_update{
        127.0.0.1;      // our own DNS server
	key rndc-key;   // And the servers with the right key (dhcp servers)
};
```
Now go to **named.conf.options**, and add :
```
allow-update { allow_update; }; 	// -> content is defined in ACL file
```
Edit : **/etc/named.conf.local**  
And add all the zones definitions :
```
// ==============================================
zone "brussels.lan" {
	type master;
	file "/etc/bind/students_zones/db.brussels.lan";
};
zone "12.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.12.1.10.in-addr.arpa";
};
// ==============================================
zone "paris.lan" {
	type master;
	file "/etc/bind/students_zones/db.paris.lan";
};
zone "13.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.13.1.10.in-addr.arpa";
};
// ==============================================
zone "berlin.lan" {
	type master;
	file "/etc/bind/students_zones/db.berlin.lan";
};
zone "14.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.14.1.10.in-addr.arpa";
};
// ==============================================
zone "madrid.lan" {
	type master;
	file "/etc/bind/students_zones/db.madrid.lan";
};
zone "15.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.15.1.10.in-addr.arpa";
};
// ==============================================
zone "athens.lan" {
	type master;
	file "/etc/bind/students_zones/db.athens.lan";
};
zone "16.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.16.1.10.in-addr.arpa";
};
// ==============================================
zone "rome.lan" {
	type master;
	file "/etc/bind/students_zones/db.rome.lan";
};
zone "17.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.17.1.10.in-addr.arpa";
};
// ==============================================
zone "oslo.lan" {
	type master;
	file "/etc/bind/students_zones/db.oslo.lan";
};
zone "18.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.18.1.10.in-addr.arpa";
};
// ==============================================
zone "dublin.lan" {
	type master;
	file "/etc/bind/students_zones/db.dublin.lan";
};
zone "19.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.19.1.10.in-addr.arpa";
};
// ==============================================
zone "amsterdam.lan" {
	type master;
	file "/etc/bind/students_zones/db.amsterdam.lan";
};
zone "20.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.20.1.10.in-addr.arpa";
};
// ==============================================
zone "london.lan" {
	type master;
	file "/etc/bind/students_zones/db.london.lan";
};
zone "21.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.21.1.10.in-addr.arpa";
};
// ==============================================
zone "kiev.lan" {
	type master;
	file "/etc/bind/students_zones/db.kiev.lan";
};
zone "22.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.22.1.10.in-addr.arpa";
};
// ==============================================
zone "lisbon.lan" {
	type master;
	file "/etc/bind/students_zones/db.lisbon.lan";
};
zone "23.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.23.1.10.in-addr.arpa";
};
// ==============================================
zone "stockholm.lan" {
	type master;
	file "/etc/bind/students_zones/db.stockholm.lan";
};
zone "24.1.10.in-addr.arpa" {
	type master;
	file "/etc/bind/students_subnets/db.24.1.10.in-addr.arpa";
};
// ==============================================
```
We can now create the zones and reverse zones:
```
sudo mkdir /etc/bind/students_subnets
sudo mkdir /etc/bind/students_zones
```
So the folowing zones are in the students network (so in the classroom : 10.1.{12-24}.0/24 & country.lan)

```
sudo vim db.paris.lan
```
Inside:
```
$ORIGIN .
$TTL 10800      ; 3 hours
paris.lan         IN SOA  alpha.tux.lan. waldek.tux.lan. (
                                20211208   ; serial
                                10800      ; refresh (3 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                3600       ; minimum (1 hour)
                                )
                        NS      alpha.tux.lan.
$ORIGIN paris.lan.
```
Do the same for each zone, and only change the "paris.lan" by the right country:
```
db.brussels.lan
db.madrid.lan
db.berlin.lan
db.london.lan
db.kiev.lan
db.oslo.lan
db.dublin.lan
db.amsterdam.lan
db.lisbon.lan
db.stockholm.lan
db.rome.lan
db.athens.lan
```
Now the reverse zones, quiet similar:
```
cd /etc/bind/students_subnets
sudo vim db.12.1.10.in-addr.arpa
```
And inside:
```
$ORIGIN .
$TTL 10800      ; 3 hours
12.1.10.in-addr.arpa    IN SOA  alpha.tux.lan. waldek.tux.lan. (
                                20211208   ; serial
                                10800      ; refresh (3 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                3600       ; minimum (1 hour)
                                )
                        NS      alpha.tux.lan.
$ORIGIN 12.1.10.in-addr.arpa.
```
Do the same for each subnet, changing the revert ip address:
```
db.13.1.10.in-addr.arpa
db.14.1.10.in-addr.arpa
db.15.1.10.in-addr.arpa
...
db.24.1.10.in-addr.arpa
```
==================
Chown all the directory
```
sudo chown bind:bind /etc/bind/ -R
```
=====================
```
sudo systemctl restart named
```

---

## On the secondary DHCP server

Do all the same as the primary DHCP server

---

## On the secondary DNS server

Do all the same as we did for the master, <u>**except**</u> :

In the **named.conf.options**, this line cannot appear:
```
allow-update { allow-update;	};
```
And in **named.conf.local**, <u>for all the zones</u> replace *type master;* by
```
type slave;
masters {172.30.4.221};
```


---

