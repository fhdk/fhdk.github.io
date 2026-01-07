---
title: 'Basic firewall (iptables) rules'
taxonomy:
    category:
        - docs
---

## Basic iptables rules

|||
| --- | --- |
| Prefix | `#` - run command as `root` or by use of `sudo` command |
| | `$` - run command in regular user-context |

!!!! The order of iptables rules matters. Traffic is filtered by first applicable rule.
!!!! If a rule accept SSH traffic, followed by a rule to deny SSH traffic, iptables will always allow the traffic because allow comes before deny rule in the chain. 
!!!! You can change the rule order by specifying a rule number in your command.

### Reject all outgoing network traffic
Ruleset only allowing current outgoing and established traffic. Needed when you are logged in to a server via ssh.
```bash
# iptables -F OUTPUT
# iptables -A OUTPUT -m state --state ESTABLISHED -j ACCEPT
# iptables -A OUTPUT -j REJECT
```

### Reject all incoming network traffic
Ruleset rejecting all incoming traffic
```bash
# iptables -F INPUT
# iptables -A INPUT -m state --state ESTABLISHED -j ACCEPT
# iptables -A INPUT -j REJECT
```

### Reject all network traffic
Ruleset to drop and block all network traffic whether incoming or outgoing.
! This will also include current ongoing established traffic and it will break your traffic if remote.
```bash
# iptables -F
# iptables -A INPUT -j REJECT
# iptables -A OUTPUT -j REJECT
# iptables -A FORWARD -j REJECT
```

### Drop incoming ping requests
Drop all incoming ping requests. It is possible to use REJECT instead of DROP. The difference between DROP vs REJECT is that DROP silently discards the incoming package, whereas REJECT will result in ICMP error being returned.
```bash
# iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

### Reject outgoing on port
Reject all outgoing traffic coming from a local port 22 (ssh).
```bash
# iptables -A OUTPUT -p tcp --dport ssh -j REJECT
```

### Reject incoming on port
Reject all incoming traffic to a local port 22 (ssh).
```bash
# iptables -A INPUT -p tcp --dport ssh -j REJECT
```

### Reject all incoming traffic except
Ruleset rejecting all incoming traffic to system except port 22 (SSH). It will also accept traffic on the loopback interface.
```bash
# iptables -A INPUT -i lo -j ACCEPT
# iptables -A INPUT -p tcp --dport ssh -j ACCEPT
# iptables -A INPUT -j REJECT
```

### Accept incoming port from IP address
Ruleset blocking all incoming traffic to port 22 (ssh) except host address `1.3.5.7`. Only allow host IP `1.3.5.7` to access ssh.
```bash
# iptables -A INPUT -p tcp -s 1.3.5.7 --dport ssh -j ACCEPT
# iptables -A INPUT -p tcp --dport ssh -j REJECT
```

### Block incoming port except from MAC address
Ruleset blocking all incoming port (ssh) except host with MAC address `11:22:33:aa:bb:cc`. Only allow host with MAC address `11:22:33:aa:bb:cc` to access ssh.
```bash
# iptables -A INPUT -m mac --mac-source 11:22:33:aa:bb:cc -p tcp --dport ssh -j ACCEPT
# iptables -A INPUT -p tcp --dport ssh -j REJECT
```

### Reject incoming traffic on a specific TCP port
Drop all incoming traffic on TCP port `6000`.
```bash
# iptables -A INPUT -p tcp --dport 6000 -j REJECT
```

### Drop all incoming traffic on interface
Drop incoming traffic on a interface coming from `subnet 172.16.1.0/16`. Useful in attempt to drop illegal IP addresses on wan interface. If `enp3s1` is an external network interface, no incoming traffic originating from internal network should hit `enp3s1` network interface.
```bash
# iptables -A INPUT -i enp3s1 -s 172.16.1.0/16 -j DROP
```

### Create a simple IP Masquerading
Create a simple IP Masquerading gateway to allow all host on the same subnet to access the Internet. The `enp3s1` is the external interface connected to the Internet.
```bash
# echo "1" > /proc/sys/net/ipv4/ip\_forward
# iptables -t nat -A POSTROUTING -o enp3s1 -j MASQUERADE
```

### Reject all incoming port except specified IP address
Reject all incoming ssh (`port 22`) traffic except traffic request from IP `7.5.3.1`
```bash
# iptables -A INPUT -t filter ! -s 7.5.3.1 -p tcp --dport 22 -j REJECT
```

### Reject all incoming port except IP address range
Reject all incoming ssh (`port 22`) traffic except traffic request from IP address range 172.16.1.65 - 172.16.1.128.
Removing negator "!" from the below rule reject all ssh traffic originating from IP address range 172.16.1.65 - 172.16.1.128.
```bash
# iptables -A INPUT -t filter -m iprange ! --src-range 172.16.1.65 - 172.16.1.128  -p tcp --dport 22 -j REJECT
```

### reject all outgoing traffic to a specific remote host
Reject all outgoing traffic to a remote host with an IP address 7.5.3.1
```bash
# iptables -A OUTPUT -d 7.5.3.1 -j REJECT
```

### block access to a specific website
Block all incoming traffic from `facebook.com` where source port is www (`port 80`).
```bash
# iptables -A INPUT -s facebook.com -p tcp --sport www -j DROP
```

## More reading
[Simple statefull firewall on wiki.archlinux.org](https://wiki.archlinux.org/title/Simple_stateful_firewall)