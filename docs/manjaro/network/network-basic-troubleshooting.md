---
title: 'Network know-how'
published: true
date: '02-09-2020 11:54'
publish_date: '02-09-2020 11:54'
taxonomy:
    category:
        - docs
---

**Difficulty: ☆☆☆☆☆**

---
## Network connections
---
This is not a guide on troubleshooting hardware and drivers but the network connection itself. The guide is assuming your hardware and driver(s) are in a working state.

The guide is intended as a reference for working your self through situations like

- I can't connect to the internet 
- I ping an IP but not a domain name
- How can I find my printer's IP

Every user will be in a situation when network troubleshooting is necessary - it is almost inevitable.

NOTE: This is a work in progress ... please add a comment if there is a network topic you would like covered - maybe in a separate guide.

IPv6 is a topic on it's own - completely different and I know nothing - so please don't ask.

---
## Troubleshooting
---
Network troubleshooting is like an onion. You peel one layer at a time until you reach the core. 

Always start with the system having issues. 

Having additional packages installed before problems arise is useful but not necessary.

```text
$ sudo pacman -Syu bind whois arp-scan net-tools traceroute
```

---
Network info
===
Getting some basic info can be done numerous ways using a variety of utilites. In the context of providing basic info NetworkManager's CLI tool provides a good overview and is Manjaro default network manager.

Example output

```
$ nmcli device show | grep IP4
IP4.ADDRESS[1]:                         10.10.10.20/24
IP4.GATEWAY:                            10.10.10.1
IP4.ROUTE[1]:                           dst = 0.0.0.0/0, nh = 10.10.10.1, mt = 100
IP4.ROUTE[2]:                           dst = 10.10.10.0/24, nh = 0.0.0.0, mt = 100
IP4.DNS[1]:                             10.10.10.4
IP4.DOMAIN[1]:                          lan.nix.dk
IP4.ADDRESS[1]:                         127.0.0.1/8
IP4.GATEWAY:                            --
```

Using the output from your system - look for answers to the following questions

* Do I have an **address**: &rarr; `IP4.ADDRESS`?
* Is the IP in the expected **sub net**: &rarr; `IP4.ROUTE dst =`?
* Is the **gateway** correct: &rarr; `IP4.GATEWAY`?
* Is the **DNS** correct: &rarr; `IP4.DNS`?
* Is the **route** correct:&rarr; `IP4.ROUTE`?

If everything checks - continue to testing your connection - use the data from your system.


### connectivity check

From **`man ping`**

> When using ping for fault isolation, it should first be run on the local host, to verify that the local network interface is up and running. Then, hosts and gateways further and further away should be “pinged”. Round-trip times and packet loss statistics are computed. If duplicate packets are received, they are not included in the packet loss calculation, although the round trip time of these packets is used in calculating the minimum/average/maximum/mdev round-trip time numbers.

The ping app will continue sending packages until you terminate with <kbd>Ctrl</kbd><kbd>c</kbd>. This can be useful if you want to know when your router becomes available after restart. But you can also limit the number of packages when you invoke the command

Gateway connection

    ping -c 3 IP4.GATEWAY

Name server connection

    ping -c 3 IP4.DNS

### routing check 

:information_source: Requires **`traceroute`**

To check how the package travels in the network we use **traceroute**. This app sends back all the hops it makes until it reaches it destination. These hops are useful to determine if the issue at hand is internal or external.

You can verify the internal hops by comparing the output of the traceroute command with the output of the above nmcli command.

To check external routing use a known external IP address e.g. one of Google's nameservers

    traceroute 8.8.8.8

### dns check

Verify if the name lookup works as expected using  **`drill`**.

Using a couple of different domain names will ensure it is not a cached result. Try a domain you have never looked up before e.g. nix.dk (one of mine)

```text
➜  ~ drill nix.dk
;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: 44615
;; flags: qr rd ra ; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; QUESTION SECTION:
;; nix.dk.	IN	A

;; ANSWER SECTION:
nix.dk.	3600	IN	A	5.103.137.72

;; AUTHORITY SECTION:

;; ADDITIONAL SECTION:

;; Query time: 18 msec
;; SERVER: 127.0.0.53
;; WHEN: Tue Dec 21 13:29:08 2021
;; MSG SIZE  rcvd: 40
```

### domain traceroute check

:information_source: Requires **`traceroute`**

Being able to resolve the domain name does not imply you can reach the domain - so knowing where the connection breaks is often informative. ISP support if often more responsive the more you know.

    traceroute nix.dk

## Locating devices on your network

:information_source: Requires  **`arp-scan`**

Every network'd device has several unique identifiers.

* MAC address
* IP address
* hostname

Locating the devices on your network can be done using arp-scan.

> You will need to be root, or arp-scan must be SUID root, in order to run arp-scan, because the functions that it uses to read and write packets require root privilege.

```
# arp-scan --localnet
Interface: eno1, type: EN10MB, MAC: 00:d8:61:xx:yy:zz, IPv4: 10.10.10.20
Starting arp-scan 1.9.7 with 256 hosts (https://github.com/royhills/arp-scan)
10.10.10.1      fc:ec:da:xx:yy:zz	Ubiquiti Networks Inc.
10.10.10.2      14:91:82:xx:yy:zz	Belkin International Inc.
10.10.10.3	    8e:3b:ad:xx:yy:zz	(Unknown: locally administered)
10.10.10.4	    b8:27:eb:xx:yy:zz	Raspberry Pi Foundation
10.10.10.5	    fc:ec:da:xx:yy:zz	Ubiquiti Networks Inc.
10.10.10.10	    00:11:32:xx:yy:zz	Synology Incorporated
10.10.10.19	    94:57:a5:xx:yy:zz	Hewlett Packard
10.10.10.30	    18:e8:29:xx:yy:zz	Ubiquiti Networks Inc.
10.10.10.31	    fc:ec:da:xx:yy:zz	Ubiquiti Networks Inc.
10.10.10.32	    b4:fb:e4:xx:yy:zz	Ubiquiti Networks Inc.
10.10.10.201	3c:d9:2b:xx:yy:zz	Hewlett Packard
10.10.10.204	b4:fb:e4:xx:yy:zz	Ubiquiti Networks Inc.

12 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 256 hosts scanned in 2.039 seconds (125.55 hosts/sec). 12 responded
```
