---
title: 'Using bind for local domain'
date: '30-10-2020 07:24'
publish_date: '30-10-2020 07:24'
taxonomy:
    category:
        - docs
---

## Network configuration
Difficulty: ★★★★★

## Serving a local network
---
Resolving local network devices can be a pain because the name query is often forwarded to root nameservers or ISP cache servers  which returns NXDOMAIN. 

To create a robust LAN whether it is for home  or small business use you need an internet facing domain which access to DNS control panel. These notes uses bind on a Raspberry Pi running the default Raspberry Pi OS.

!! **DO NOT**  
!! .local is used with the AVAHI zeroconf specification. Do not invent you own .lan or something similar - the result will not be as expected.

!!! **REQUIRED**  
!!! Experience with terminal is mandatory as all setup is done using a SSH connection and any error will break your connection. Knowledge of DNS, zone files and dhcp especially [ISC bind][1] and [ISC dhcp][2] is also mandatory.

## Install bind and dhcp
---
Ensure your system is up-to-date then install **bind** and **dhcp** packages
```

apt update && sudo apt upgrade -y
apt install bind9 isc-dhcp-server -y
```
## Example config
---
| Object | Value |
|---|---|
|domain | mydomain.org
| local domain | corp.mydomain.org
| local net/subnet | 172.16.10.0/24 
| local gateway IP |172.16.10.1
| local nameserver IP | 172.16.10.2
| dhcp range | 172.16.10.100 to 200
| temp nameserver IP | 91.239.100.100

The temporary nameserver is provided by [uncensoreddns.org][3]

### Static IP
Assign a static IP to your Pi by editing your network configuration.

Create a file `/etc/network/interfaces` and add content 

```text

auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
        address 172.16.10.2
        netmask 255.255.255.0
        gateway 172.16.10.1
        dns-nameservers 91.239.100.100 
        # dns-nameservers 127.0.0.1
```
Double check your entries and reload
```

sudo systemctl daemon-reload
sudo systemctl reload network
```
From another terminal on your workstation ping your Pi
```text

ping 172.16.10.2
```

On your Pi check the lookup
```text

dig domain.org
```

Ensure you have a working Pi before continuing

### Setup your public DNS
Login into your domain dns controlpanel and create the following records and save the changes

1. A-record `ns.corp.domain.org` with IP ```172.16.10.2```
2. Nameserver `ns.corp.domain.org` for `corp.domain.org` 

Dump zone file from your control panel and verify the zone is correct
```text

ns.corp.domain.org.     43200   IN  A   172.16.10.2
corp.domain.org.        43200   IN  NS  ns.corp.domain.org.
                        43200       A   172.16.10.0
```

### corp.domain.org zone
Enable and start the local nameserver
```text

systemctl enable bind9.service
systemctl start bind9.service
```

Create a local zone file `/etc/bind/db.corp.domain.org` with content
```text

$TTL 1h
@       IN  SOA ns.corp.domain.org. admin.domain.org. (
                1       ; Serial
                3600    ; Refresh
                3600    ; Retry
                1W      ; Exire
                3H      ; Negative Cache TTL
                )
@       IN      NS      ns.corp.domain.org.
ns      IN      A       172.16.10.2
mail    IN      A       172.16.10.3
webmail IN      CNAME   mail.corp.domain.org.
```

Check the zone 
```text

named-checkconf /etc/bind/db.corp.domain.org
```
### 172.16.10.0/24 reverse zone
Create a reverse zone file `/etc/bind/db.172.16.10` with content
```text

@       IN      SOA     ns.corp.domain.org. admin.domain.org. (
                        1               ; Serial
                        604800          ; Refresh
                        86400           ; Retry
                        2419200         ; Expire
                        604800          ; Negative Cache TTL
                        )
@       IN      NS      ns.corp.domain.org.
2       IN      PTR     ns.corp.domain.org
3       IN      PTR     mail.corp.domain.org
```
Check the zone 
```text

named-checkconf /etc/bind/db.172.160.10
```

Edit `/etc/bind/zones.rfc1918` and comment the line for your reverse zone file and save
```text

# zone "16.172.in-addr.arpa"  { type master; file "/etc/bind/db.empty"; };
```

Check the zone 
```text

named-checkconf /etc/bind/zones.rfc1918
```

### Adding zones
Edit `/etc/bind/named.conf.local`  and add the new new zones and save the file
```text

zone "corp.domain.org" {
    type master;
    file "/etc/bind/db.corp.domain.org;
};

zone "10.16.172.in-addr.arpa" {
    type master;
    file "/etc/bind/db.172.16.10";
};
```

Check the zone 
```text

named-checkconf /etc/bind/named.conf.local
```
### Check root servers
Check the root hints in  `db.root` and update the file if necessary (remember to check the zone if you edit)
```text

dig +bufsize=1200 +norec NS . @a.root-servers.net | egrep -v ';|^$' | sort
```
### Check the options file
Edit the file `/etc/bind/named.conf.options` if necessary - remember to check the zone after edit.

## Time of reckoning
---
Reload the bind9 service
```text

systemctl reload bind9.service
```
On the Pi test your lookup
```text

dig corp.domain.org
```

Change your Raspberry Pi network to use local bind service by editing the file `/etc/network/interfaces` file Change the temporay nameserver to `127.0.0.1` save the file and reload networking.
```text

#dns-nameservers 91.239.100.100 
dns-nameservers 127.0.0.1
```

```text

systemctl reload networking
```
On the Pi test your lookup one more time - you should get the same answer - this time from your Pi
```text

dig corp.domain.org
```

## Setting up isc-dhcp
---
Edit the file `/etc/dhcp/dhcpd.conf` with content
```text

ddns-update-style interim;
ddns-updates on;
do-forward-updates on;
option domain-name "corp.domain.org";
option domain-name-servers 172.16.10.2;
option broadcast-address 172.16.10.255;
option routers 172.16.10.1;
default-lease-time 86400;
max-lease-time 604800;

subnet 172.16.10.0 netmask 255.255.255.0 {
    range 172.16.10.101 172.16.10.200;
};
```
! Only one DHCP service is allowed for a network so disable all other DHCP services.

When you have disabled router DHCP - enable and start the `isc-dhcp-server.service`
```text

systemctl enable isc-dhcp-server.service
systemctl start isc-dhcp-server.service
```

# BIND ad blocker
When you have gone through lengths to set up your dns - you might as well utilize bind to block ads and known malware domains.

Below is the README from [https://github.com/Trellmor/bind-adblock][4] which I have utilized to sanitize my network.

The BIND ad blocker fetches various blocklists and generate a BIND zone from them.

Configure BIND to return `NXDOMAIN` for ad and tracking domains to stop clients from contacting them.

Requires BIND 9.8 or newer for [RPZ](https://en.wikipedia.org/wiki/Response_policy_zone) support.

Uses the following sources:

* [Peter Lowe’s Ad and tracking server list](https://pgl.yoyo.org/adservers/)
* [Malware domains](http://www.malwaredomains.com/)
* [MVPS HOSTS](http://winhelp2002.mvps.org/)
* [Adaway default blocklist](https://adaway.org/hosts.txt)
* [Dan Pollock’s hosts file](http://someonewhocares.org/hosts/zero/)
* [MalwareDomainList.com Hosts List](http://www.malwaredomainlist.com/hostslist/hosts.txt)
* [StevenBlack Unified hosts file](https://github.com/StevenBlack/hosts)
* [CAMELEON](http://sysctl.org/cameleon/)
* [Disconnect.me Basic tracking list](https://disconnect.me/trackerprotection)
* [Disconnect.me Ad Filter list](https://disconnect.me/trackerprotection)
* [Polish CERT Phishing list](https://www.cert.pl/ostrzezenia_phishing/)

## Setup

### Python packages

See requirements.txt

To install
```

pip install -r requirements.txt
```


### Configure BIND

Add the `response-policy` statement to the BIND options

```

// For AdBlock
response-policy {
	zone "rpz.example.com";
};
```

Add your rpz zone. Replace example.com with a domain of your choice.

```

// AdBlock
zone "rpz.example.com" {
	type master;
	file "/etc/bind/db.rpz.example.com";
	masterfile-format text;
	allow-query { none; };
};
```

Create a zone file for your zone. Replace example.com with the domain you used before.
```

@ 3600 IN SOA @ admin.example.com. 0 86400 7200 2592000 86400
@ 3600 IN NS ns.example.com.
```

## Usage
---
```

    usage: update-zonefile.py [-h] [--no-bind] [--raw] [--empty] zonefile origin

    Update zone file from public DNS ad blocking lists

    positional arguments:
      zonefile    path to zone file
      origin      zone origin

    optional arguments:
      -h, --help  show this help message and exit
      --no-bind   Don't try to check/reload bind zone
      --raw       Save the zone file in raw format. Requires named-compilezone
      --empty     Create header-only (empty) rpz zone file
```

Example: `update-zonefile.py /etc/bind/db.rpz.example.com rpz.example.com`

`update-zonefile.py` will update the zone file with the fetched adserver lists and issue a `rndc reload origin` afterwards.

## Whitelist
---
You can either use an additional zone to whitelist domains (Or add them to `config.yml`) 
See [Whitelist](https://github.com/Trellmor/bind-adblock/wiki/whitelist) for adding a whitelist zone.




[1]: https://www.isc.org/bind/
[2]: https://www.isc.org/dhcp/
[3]: https://blog.uncensoreddns.org/
[4]: https://github.com/Trellmor/bind-adblock