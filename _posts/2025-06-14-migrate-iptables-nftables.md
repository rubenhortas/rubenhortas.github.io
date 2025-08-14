---
title: Migrate from iptables to nftables. Boost your Linux firewall performance
date: 2025-06-14 00:00:01 +0000
categories: [hardening, firewall]
tags: [hardening, firewall, iptables, nftables]
---

Learn the essential steps to migrate your Linux firewall configurations from iptables to nftables.
This guide covers syntax, commands, best practices for a smooth transition and offers a good set of essential rules.

## Why?

A while back, due to some changes in my home networks, I decided to harden my systems by implementing firewalls with iptables: [iptables essential rules. A starting point to protect your computer](https://rubenhortas.github.io/posts/iptables-essential-rules/).
And, to this day, I'm still very happy with the results.
However, as everything changes and evolves, the currently recommended firewall is nftables.
So I decided to upgrade myself, my systems, and migrate the firewalls to nftables.

## What is nftables?

nftables is developed by Netfilter, the same organization that currently maintains iptables.

nftables is a subsystem of the Linux kernel that provides filtering and classification of network packets/datagrams/frames.

nftables was first publicly presented at Netfilter Workshop 2008 by Patrick McHardy from the Netfilter Core Team.
The first preview release of kernel and userspace implementation was given in March 2009.

On 16 October 2013, Pablo Neira Ayuso submitted a nftables core pull request to the Linux kernel mainline tree.
It was merged into the kernel mainline on 19 January 2014, with the release of Linux kernel version 3.13.

nftables replaces the popular {ip,ip6,arp,eb}tablesreuses the existing Netfilter subsystems, and provides a new in-kernel packet classification framework based on a network-specific Virtual Machine (VM) and a new nft userspace command line tool.
This VM is able to execute bytecode to inspect a network packet and make decisions about how to handle it.

## Why ntables instead iptables?

Iptables is aging very well, but nftables is gaining ground.
The Netfilter project and the community are focused in replacing iptables with nftables.
And, nftables, offers us some advantages over iptables:

* It provides new structures that drastically reduces the number of rules needed to inspect a packet until reaching the final action.
* Faster packet classification.
* Simplified dual stack IPv4/IPv6 management (This avoids code duplication and inconsistencies).
* Nicer and more compact syntax

Basically, it avoids duplication of work and inconsistencies, is easier to configure and mantain and is faster.
I love working less! ;)

## Migrate from iptables to nftables

I'm going to explain how to migrate based on some initial rules that you can see in my post [iptables essential rules. A starting point to protect your computer](https://rubenhortas.github.io/posts/iptables-essential-rules/).
If you don't have any initial rules to work from, you will have to manually create your file and you can skip the next sections and jump to [Test your nftables ruleset](https://rubenhortas.github.io/posts/migrate-iptables-nftables/#test-your-nftables-ruleset).

### Backup iptables rules

```shell
sudo iptables-save  > /home/rubenhortas/backups/iptables/iptables/iptables_rules_v4~
sudo ip6tables-save > /home/rubenhortas/backups/iptables/iptables_rules_v6~
```

### Install nftables

```shell
sudo apt install nftables
```

### Translate iptables rules to nftables rules

```shell
sudo iptables-restore-translate -f /home/rubenhortas/backups/iptables/iptables/iptables_rules_v4~ > /home/rubenhortas/backups/nftables_rules_v4.nft
sudo ip6tables-restore-translate -f /home/rubenhortas/backups/iptables/iptables_rules_v6~ > /home/rubenhortas/backups/nftables_rules_v4.nft
```

### Review the rules (Coffee time! ☕)

This is usually where I go to make a cup of coffee, because I spend a lot of time reviewing, sorting and commenting the generated rules ;)

I do this to make a mental map of the rules and needs, and later organize them.
Also, some complex iptables extensions might not translate perfectly and may require manual adjustment.

There is my nftables starting point:

```shell
# Set Default Policies (DROP everything first)
add table ip filter
add chain ip filter INPUT { type filter hook input priority 0; policy drop; }
add chain ip filter FORWARD { type filter hook forward priority 0; policy drop; }
add chain ip filter OUTPUT { type filter hook output priority 0; policy drop; }

# Allow all traffic on the loopback interface (the internal 127.0.0.1 address. Essential for local processes)
add rule ip filter INPUT iifname "lo" counter accept
add rule ip filter OUTPUT oifname "lo" counter accept

# Blacklisting IP adresses
# 168.254.0.0/16 Zeroconf address range. APIPA range.
# 192.0.2.0/24 TEST-NET-1
# 224.0.0.0/4 Multicast
# 240.0.0.0/4 Reserved/Class E
# 255.255.255.255 Broadcast
# 0.0.0.0/8 Unspecified/Current Network
add set ip filter blacklisted_ips { type ipv4_addr; flags interval; auto-merge; }
add element ip filter blacklisted_ips { 169.254.0.0/16, 192.0.2.0/24, 224.0.0.0/4, 240.0.0.0/4, 255.255.255.255, 0.0.0.0/8 }
add rule ip filter INPUT ip saddr @blacklisted_ips counter drop

# Spoofing
# Deny incoming traffic that has a source address of an IP address assigned to a local interface.
# Incoming traffic with the source address of your system is going to be spoofed traffic because you know it cannot be generated by the host.# 192.168.1.100 Own ip
add rule ip filter INPUT ip saddr 192.168.1.100 counter drop
add rule ip filter OUTPUT ip saddr != 192.168.1.100 counter drop  # Deny outgoing traffic that does not have a source address of an interface on the local host.

# Drop packets with invalid state
add rule ip filter INPUT ct state invalid counter drop

# SSH
# Allow incoming SSH connections
# Deny outgoing connections is covered by the OUTPUT policy drop
add rule ip filter INPUT tcp dport 22 ct state new,established counter accept

# Drop Packet fragments
add rule ip filter INPUT ip frag-off & 0x1fff != 0 counter drop

# Allow Established/Related Connections
add rule ip filter INPUT ct state related,established counter accept
add rule ip filter OUTPUT ct state related,established counter accept

# Bad flags (and another anomalies)
add chain ip filter BAD_FLAGS
add rule ip filter INPUT ip protocol tcp counter jump BAD_FLAGS
add rule ip filter BAD_FLAGS tcp flags fin,syn / fin,syn counter drop
add rule ip filter BAD_FLAGS tcp flags syn,rst / syn,rst counter drop
add rule ip filter BAD_FLAGS tcp flags fin,syn,psh / fin,syn,psh counter drop
add rule ip filter BAD_FLAGS tcp flags fin,syn,rst / fin,syn,rst counter drop
add rule ip filter BAD_FLAGS tcp flags fin,syn,rst,psh / fin,syn,rst,psh counter drop
add rule ip filter BAD_FLAGS tcp flags fin / fin counter drop
add rule ip filter BAD_FLAGS tcp flags 0x0 / fin,syn,rst,psh,ack,urg counter drop
add rule ip filter BAD_FLAGS tcp flags fin,syn,rst,psh,ack,urg / fin,syn,rst,psh,ack,urg counter drop
add rule ip filter BAD_FLAGS tcp flags fin,psh,urg / fin,syn,rst,psh,ack,urg counter drop
add rule ip filter BAD_FLAGS tcp flags fin,syn,rst,ack,urg / fin,syn,rst,psh,ack,urg counter drop

# Port scanners
# Drop packages if the connections are too agressive to avoid port scanning.
add rule ip filter INPUT tcp flags syn / fin,syn,rst,ack ct state new meter port_scanners { ip saddr limit rate over 5/second } counter drop
add rule ip filter INPUT ip protocol udp ct state new meter port_scanners { ip saddr limit rate over 35/second } counter drop

# SYN flood protection
add rule ip filter INPUT tcp flags syn / fin,syn,rst,ack limit rate 100/second burst 200 packets counter accept
add set ip filter connlimit0 { type ipv4_addr; flags dynamic; }
add rule ip filter INPUT tcp flags syn / fin,syn,rst,ack add @connlimit0 { ip saddr ct count over 30 } counter reject with tcp reset
add rule ip filter INPUT tcp flags syn / fin,syn,rst,ack counter drop

# ICMP
# All ICMP responses should be barred except responses to outgoing connections.
# Allow outbound echo messages and indbound echo reply messages -> Allows the use of ping from the host.
# Allow time exceeded and destination unreaachable messages inbound -> Allow the use of tools such traceroute.
# Avoids ICMP flood, ICMP smurf, Ping of death, ICMP nuke...
add chain ip filter ICMP_IN
add chain ip filter ICMP_OUT
add rule ip filter INPUT ip protocol icmp counter jump ICMP_IN
add rule ip filter OUTPUT ip protocol icmp counter jump ICMP_OUT
add rule ip filter ICMP_IN icmp type echo-reply ct state related,established counter accept
add rule ip filter ICMP_IN icmp type destination-unreachable ct state related,established counter accept
add rule ip filter ICMP_IN icmp type echo-request ct state new counter drop
add rule ip filter ICMP_IN icmp type time-exceeded ct state related,established counter accept
add rule ip filter ICMP_OUT icmp type echo-request ct state new,established counter accept

# Allow NFS
add rule ip filter INPUT tcp dport 2049 ct state new,established counter accept
add rule ip filter OUTPUT tcp sport 2049 ct state established counter accept

# IRC
# add rule ip filter INPUT ip protocol tcp tcp sport 6697 ct state established counter accept
# add rule ip filter OUTPUT ip protocol tcp tcp dport 6697 ct state new,established counter accept

# Allow DNS
add rule ip filter INPUT ip protocol tcp tcp sport { 53, 853 } ct state established counter accept
add rule ip filter INPUT ip protocol udp udp sport { 53, 853 } ct state established counter accept
add rule ip filter OUTPUT ip protocol tcp tcp dport { 53, 853 } ct state new,established counter accept
add rule ip filter OUTPUT ip protocol udp udp dport { 53, 853 } ct state new,established counter accept

# Allow HTTP and HTTPS
add rule ip filter INPUT ip protocol tcp tcp dport { 80, 443 } ct state established counter accept
add rule ip filter OUTPUT ip protocol tcp tcp dport { 80, 443 } ct state new,established counter accept

# Allow NTP
add rule ip filter INPUT udp sport 123 ct state established counter accept
add rule ip filter OUTPUT udp dport 123 ct state new,established counter accept
```

> 192.168.1.100 it's the only IP my machine has.
> Adjust according to your needs.
{: .prompt-info}

Now, we can use these rules or we can (manually) create a nftables.conf file.

### Create a nftables.conf (more coffee! ☕☕)

This is often preferred for a cleaner, more organized setup.
You'll define tables and chains explicitly:

```shell
#!/usr/sbin/nft -f

# Clear existing rules
flush ruleset

# Interfaces
define LAN_INTERFACE_IPV4 = 192.168.1.100
define LAN_INTERFACE_IPV6 = dead::beef

# Ports
define SSH_PORT = 22
define NFS_PORT = 2049
define HTTP_PORTS = { 80, 443 }
define DNS_PORTS = { 53, 853 }
define NTP_PORT = 123

# Using 'inet' family for rules that apply to both IPv4 and IPv6,
table inet filter {
    # Blacklisted IP addresses
    set blacklisted_ipv4_ips {
        type ipv4_addr; # You might want to make this 'inet_addr' if you'll add IPv6 blacklist entries
        flags interval;
        auto-merge;
        elements = { 169.254.0.0/16, 192.0.2.0/24, 224.0.0.0/4, 240.0.0.0/4, 255.255.255.255, 0.0.0.0/8 }
    }

    set blacklisted_ipv6_ips {
        type ipv6_addr;
        flags interval;
        auto-merge;
        elements = { fe80::/10, 2001:db8::/32, ff00::/8, ::/128 }
    }

    # Chain for malformed TCP flags (applicable to both IPv4 and IPv6 within inet table)
    chain BAD_FLAGS {
        # Using a single match for common flag combinations
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == syn | rst counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn | psh counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn | rst counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn | rst | psh counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == 0x00 counter drop # No flags (NULL scan)
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn | rst | psh | ack | urg counter drop # XMAS scan
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | psh | urg counter drop
        tcp flags & (fin | syn | rst | psh | ack | urg) == fin | syn | rst | ack | urg counter drop
    }

    # ICMP handling for incoming traffic (IPv4 & IPv6 consolidated)
    chain ICMP_IN {
        # Allow replies for established/related ICMP sessions
        icmp type { echo-reply, destination-unreachable, time-exceeded } ct state { related, established } counter accept
        icmpv6 type { 129, 1, 2, 3, 4 } ct state { related, established } counter accept # echo-reply, dest-unreachable, packet-too-big, time-exceeded, parameter-problem

        # Drop incoming echo-requests (ping) by default
        icmp type echo-request counter drop
        icmpv6 type echo-request counter drop
    }

    # ICMP handling for outgoing traffic (IPv4 & IPv6 consolidated)
    chain ICMP_OUT {
        # Allow outgoing echo-requests (ping) and established/related ICMP
        icmp type echo-request ct state { new, established } counter accept
        icmpv6 type echo-request ct state { new, established } counter accept
    }

    chain INPUT {
        type filter hook input priority 0; policy drop; # Default drop policy for incoming

        # Optimize order for performance:
        # High-volume, low-risk traffic first.
        # Specific drops before general accepts.

        # 1. Allow all traffic on the loopback interface (essential for local processes)
        iif "lo" accept

        # 2. Allow Established/Related Connections (VERY IMPORTANT, handles most ongoing traffic)
        ct state { related, established } counter accept

        # 3. Drop packets with invalid state (e.g., malformed, out-of-sync)
        ct state invalid counter drop

        # 4. Anti-spoofing and blacklisted IPs (fast drops for known bad traffic)
        # IPv4
        ip saddr @blacklisted_ipv4_ips counter drop
        ip saddr $LAN_INTERFACE_IPV4 counter drop # Deny incoming traffic from own IP

        #IPv6
        ip6 saddr @blacklisted_ipv6_ips counter drop
        ip6 saddr $LAN_INTERFACE_IPV6 counter drop

        # 5. Drop fragmented packets that are clearly invalid or cannot be reassembled
        ip frag-off != 0 ct state invalid counter drop

        # 6. Basic port scanner protection (SYN floods, UDP scans)
        tcp flags syn limit rate over 5/second burst 5 packets counter drop # SYN scans
        meta l4proto udp ct state new limit rate over 35/second burst 35 packets counter drop # UDP scans

        # 7. Jump to BAD_FLAGS chain for detailed TCP flag inspection
        tcp flags & (fin | syn | rst | psh | ack | urg) != 0x00 jump BAD_FLAGS

        # 8. Jump to ICMP chain
        meta l4proto { icmp, icmpv6 } counter jump ICMP_IN

        # 9. Whitelist specific services (order by most frequent or most critical for speed)
        # SSH: Allow incoming SSH connections
        tcp dport $SSH_PORT ct state new counter accept

        # NFS: Allow incoming NFS
        tcp dport $NFS_PORT ct state new counter accept

        # DNS: Allow incoming DNS responses
        ip protocol tcp tcp sport $DNS_PORTS ct state new counter accept
        ip protocol udp udp sport $DNS_PORTS ct state new counter accept

        # Allow incoming NTP responses
        udp sport $NTP_PORT ct state new counter accept

        # 10. SYN flood protection
        tcp flags syn limit rate 100/second burst 200 packets counter accept
        tcp flags syn ct state new limit rate 30/second burst 60 packets counter drop
    }

    chain FORWARD {
        type filter hook forward priority 0; policy drop;
    }

    chain OUTPUT {
        type filter hook output priority 0; policy drop; # Default drop policy for outgoing

        # 1. Allow all traffic on the loopback interface
        oif "lo" accept

        # 2. Allow Established/Related Connections
        ct state { related, established } counter accept

        # 3. Anti-spoofing for outgoing traffic
        # Deny outgoing traffic that does not have the source address of the system interface
        ip saddr != $LAN_INTERFACE_IPV4 counter drop
        ip6 addr != $LAN_INTERFACE_IPV6 counter drop

        # Outgoing DNS requests
        tcp dport $DNS_PORTS ct state new counter accept
        udp dport $DNS_PORTS ct state new counter accept

        # Outgoing HTTP and HTTPS requests
        tcp dport $HTTP_PORTS ct state new counter accept

        # Outgoing NTP requests
        udp dport $NTP_PORT ct state new counter accept

        # Outgoing NFS (server-side, uses source port)
        tcp sport $NFS_PORT ct state new counter accept
        udp sport $NFS_PORT ct state new counter accept

        # 5. Jump to ICMP_OUT chain for ICMP/ICMPv6 traffic
        meta l4proto { icmp, icmpv6 } counter jump ICMP_OUT
    }
}
```

> 192.168.1.100 it's the only IP my machine has.
> Adjust IPs and interfaces according to your needs.
{: .prompt-info}

### Test your nftables ruleset

#### Validate syntax

`sudo nft -c -f /home/rubenhortas/configs/nftables.conf`

If there's no output, the syntax is correct.

#### Test in coward mode

`sudo nft -f /home/rubenhortas/configs/nftables.conf && sleep 300 && sudo nft flush ruleset`

If anything goes wrong, the nftables rules will be flushed in 5 minutes.

Verify the rules with:

`sudo nft list ruleset`

You have 5 minutes to test your connections.
You can repeat these steps as many times as you want/need ;)

### Flush iptables rules

I flush my iptables and ip6tables at first, to make sure there are no rules in memory:

```shell
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X
```

```shell
ip6tables -P INPUT ACCEPT
ip6tables -P FORWARD ACCEPT
ip6tables -P OUTPUT ACCEPT

ip6tables -t nat -F
ip6tables -t mangle -F
ip6tables -F
ip6tables -X
```

### Disable and stop iptables services

Once we have verified that everything is working correctly, we can disable iptables services:

```shell
sudo systemctl disable --now iptables
sudo systemctl disable --now ip6tables
sudo systemctl disable --now netfilter-persistent # If you use iptables-persistent
```

### Uninstall iptables and netfilter-persistent

`sudo apt purge iptables netfilter-persistent`

### Delete /etc/iptables

`sudo rm -rf /etc/iptables`

### Find more iptables files (or folders)

You can also check if there are more iptables files on the system:

`sudo find / \( -path /proc -o -path ./usr \) -iname *ip?tables* 2>/dev/null`

### Copy our nftables.conf to /etc

To make the rules persistent, we must save them to a configuration file: `/etc/nftables.conf`.
The nftables service will start automatically on boot and load the rules from `/etc/nftables.conf`

`sudo cp /home/rubenhortas/configs/nftables.conf /etc/`

### Enable and start nftables service

`sudo systemctl enable --now nftables`

### Reboot and check our nftables rules

After reboot, we can check our nftables rules again to ensure they load correctly and are persisted.

## Sources

* [netfilter/nftables](https://netfilter.org/projects/nftables/)
* [wiki.nftables](https://wiki.nftables.org)
* [Wikipedia/nftables](https://en.wikipedia.org/wiki/Nftables)

As always, thanks to [Rodrigo Rega](https://rodrigorega.es/) for the advice :)

*Enjoy! ;)*
