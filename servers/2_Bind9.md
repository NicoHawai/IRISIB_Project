
# Bind9 and failover on Debian 11

- Master
	- [Installation & Configuration](#installationonthemaster)
- Slave
	- [Installation & Configuration](#installationontheslave)
- Both
	- [Configuring the logs](#configuring-the-logs)
  - [Tips](#tips)  
- [On the DHCP servers](#on-the-dhcp-servers)
  

*Note, in our example, Alpha DNS server = 172.30.4.221 (master), Beta DNS server = 172.30.4.222 (slave)*


---

## Installation & Configuration - on the Master (172.30.4.221) <a id="installationonthemaster"></a>

```
sudo apt update
sudo apt install bind9
```
Now, 3 main files are in */etc/bind*   

**1) /etc/bind/named.conf**  

Add these two lines, and then we will create the files.

```
include "/etc/bind/named.conf.acl";
include "/etc/bind/rndc.key";
```
We're going to use ACL's, like templates.  
And also a secure key, this way only the slave DNS server will be able to download the db. 

Now we're going to create the **ACL's**.  
```
sudo vim /etc/bind/named.conf.acl 
```
And inside, we will put the addresses of who can use our servers as a client:
```
acl allow_queries {
	172.30.4.221;   // Alpha
	172.30.4.222;   // Beta
  172.30.4.220;   // Relay - This is the client's NAT
};
```
Now we're going to create our **secure key**:  
*Something random*
```
echo "kj/n65!dfgeçUIor6\t5jil<32n1S!rert?qs6-e54rSF/HG6t=ui5lo:l46" > pass
sudo sha256sum pass | sudo tee /etc/bind/rndc.key
rm pass
```
Then modify the file we just made *(/etc/bind/rndc.key)*:
```
key "rndc-key" {
    algorithm hmac-sha256;
    secret "0dfcqq1f61fdgd654654fgdf00b41904b351fd90bfe965fghfg489690d051";
};
```
Of course *secret* is the one you just made.  
Then change the access mode in:
```
sudo chmod 640 /etc/bind/rndc.key
```

============
chown to include **bind** in group and owner
```
sudo chown bind:bind /etc/bind/rndc.key
```
===================
So now we've just managed the 2 files that we previously added to named.conf
```
include "/etc/bind/rndc.key"
include "/etc/bind/named.conf.acl"
```
**2) /etc/named.conf.option**  

This is where we will put the  global server configuration.
```
options {
    directory "/var/cache/bind";        // Working directory
    dnssec-validation auto; 
    listen-on { any; };                 // IPV4 
    listen-on-v6 { none; };             // IPV6
    recursion yes;                      // Recursive queries
    allow-query { allow_queries; }; 		  // -> content is defined in ACL file
    allow-transfer {                    // Secondary DNS (failover)
      172.30.4.222; 
      key rndc-key; 
    };
    notify yes;                         // Notify the slave server when changes occur
    forwarders {                        // Public DNS
        1.1.1.1;
        8.8.8.8;
    };
    version none;                        // Security feature
};
```
We can see or rdnc-key and our *allow_queries* acl


**3) /etc/named.conf.local**  
This is our main configuration, we're going to add our server's zones and their reverse zones. 
*type master* because this is the master.  
*file* is where the config file for the zone is stored.
```
// ==============================================
zone "tux.lan" {
	type master;
	file "/etc/bind/db.tux.lan";
};
zone "4.30.172.in-addr.arpa" {
	type master;
	file "/etc/bind/db.4.30.172.in-addr.arpa";
	};

```
Create the **DNS zone**
```
sudo vim /etc/bind/db.tux.lan

$ORIGIN .
$TTL 10800      ; 3 hours
tux.lan               IN SOA  alpha.tux.lan. waldek.tux.lan. (
                                20211206   ; serial
                                10800      ; refresh (3 hours)
                                3600       ; retry (1 hour)
                                604800     ; expire (1 week)
                                3600       ; minimum (1 hour)
                                )
                        NS      alpha.tux.lan.
$ORIGIN tux.lan.
alpha                   A       172.30.4.221
beta                    A       172.30.4.222
relay                   A       172.30.4.220
router                  A       172.30.4.1
```
*A*     host address  
*SOA*   marks the start of a zone of authority  


Then create the **reverse DNS zone**:
```
sudo vim /etc/bind/db.4.30.172.in-addr.arpa

$TTL    10800
$ORIGIN 4.30.172.in-addr.arpa.
@       IN      SOA     alpha.tux.lan.        waldek.tux.lan. (
        20160505;
        3h;
        1h;
        1w;
        1h);
@       IN      NS      alpha.tux.lan.
222     IN      NS      beta.tux.lan.
222     IN      PTR     beta.tux.lan.
221     IN      PTR     alpha.tux.lan.
220     IN      PTR     relay.tux.lan.
```

*PTR*   domain name pointer  
*NS*    authoritative name server  
*SOA*   marks the start of a zone of authority  

Change permission :
```
sudo chown bind:bind /etc/bind/ -R
```
Let's modify apparmor to let Bind able to modify the entries. (it is because we choose to do that in /etc/bind/, if your not confortable to let bind modify its own directory, you can use /var/lib/bind instead)  
In **/etc/apparmor.d/usr.sbin.named** replace */etc/bind/** r,* by :
```
/etc/bind/** rw,
/etc/bind/ rw,
```
Then restart apparmor service
```
sudo systemctl restart apparmor.service
```

```
sudo systemctl restart named
```

If any errors, go to [Tips](#tips)

---

## Installation & Configuration - on the Slave (172.30.4.222) <a id="installationontheslave"></a>

Do all the same as we did for the master, <u>**except**</u> :

In the **named.conf.options**, these lines cannot appear:
```
    allow-transfer {                    // Secondary DNS (failover)
      172.30.4.222; 
      key rndc-key; 
    }; 

    notify yes;
```
In the **named.conf.local**, change *master* by *slave*:
```
type slave;
```
In the **named.conf.local**, add to **<u>each</u>** zone:
```
masters { 172.30.4.221; };
```
Then restart
```
sudo systemctl restart named
```
If any errors, go to [Tips](#tips)

---

## Configuring the logs

It is quiet long, but I like to control and monitor what's happening on my servers. You can obviously create less logs

[Here](https://www.isc.org/docs/BIND_Logging.pdf) is the logging syntax from ISC, and [here](https://bind9.readthedocs.io/en/latest/reference.html#logging-statement-grammar) from RedHat

```
logging {

  channel bind_default {
    file "/var/log/bind/default.log" versions 50 size 20m;
    severity notice;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category default { 
	bind_default; 
  };
  channel bind_update {
    file "/var/log/bind/update.log" versions 50 size 20m;
    severity info;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category update { 
    bind_update;
  };
  channel bind_update_security {
    file "/var/log/bind/update-security.log" versions 50 size 20m;
    severity debug;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category update-security { 
    bind_update_security;
  };
  channel bind_security {
    file "/var/log/bind/security.log" versions 50 size 20m;
    severity debug;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category security { 
    bind_security;
  };
  channel bind_queries {
    file "/var/log/bind/queries.log" versions 50 size 20m;
    severity notice;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category queries { 
    bind_queries;
  };
  channel bind_transfert_out {
    file "/var/log/bind/transfert-out.log" versions 50 size 20m;
    severity info;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category xfer-out {
    bind_transfert_out;
  };
  channel bind_general {
    file "/var/log/bind/general.log" versions 50 size 20m;
    severity notice;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category general {
    bind_general;
  };
  channel bind_client {
    file "/var/log/bind/client.log" versions 50 size 20m;
    severity notice;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category client {
    bind_client;
  };
  channel bind_config {
    file "/var/log/bind/config.log" versions 50 size 20m;
    severity debug;
    print-category yes;
    print-severity yes;
    print-time yes;
  };
  category config {
    bind_config;
  };
  category lame-servers { 
	null; 
  };
};
```
Then :

```
sudo mkdir /var/log/bind
sudo touch /var/log/bind/default.log
sudo touch /var/log/bind/update.log
sudo touch /var/log/bind/update-security.log
sudo touch /var/log/bind/security.log
sudo touch /var/log/bind/queries.log
sudo touch /var/log/bind/transfert-out.log
sudo touch /var/log/bind/general.log
sudo touch /var/log/bind/client.log
sudo touch /var/log/bind/config.log
```
```
sudo chown -R bind:root /var/log/bind
```
Add this line to **/etc/bind/named.conf**  
```
include "/etc/bind/named.conf.log";
```
Eventually restart the service:
```
sudo systemctl restart named.service
```

---
## Tips

If you have some errors, but you don't know what, you can check the configuration quickly with :
```
sudo named-checkconf 
```
---

## On the DHCP servers

Edit, in **/etc/dhcp.dhcpd.conf**:  

```
option domain-name-servers 172.30.4.221, 172.30.4.222;
```
Then restart service
