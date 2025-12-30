---
title: Dynamic firewall rules with nftables and NetworkManager
date: 2025-08-30 00:00:01 +0000
categories: [hardening, firewall]
tags: [hardening, firewall, iptables, nftables]
---

Dynamically implement IPv4 and IPv6 firewall rules for each network interface with [nftables](https://netfilter.org/projects/nftables/) and [NetworkManager](https://networkmanager.dev/).
If the interface comes online or changes IP, the rules are automatically updated.
If the interface goes offline, the rules are automatically deleted.

## Why?

I while ago I wrote about [Migrate from iptables to nftables](https://rubenhortas.github.io/posts/migrate-iptables-nftables/).
More recently I wrote about [How to prevent IP spoofing with nftables and NetworkManager](https://rubenhortas.github.io/posts/nftables-dynamic-anti-spoofing-networkmanager/).
Yep, I'm into firewalling lately... :P

The thing is, as the [iptables to nftables migration post](https://rubenhortas.github.io/posts/migrate-iptables-nftables/) is written in a single file (for simplicity and to use a server as an example), and IP spoofing (althoug is implemented dynamically) is only a part of protection, I decided to combine both, and explain how to dynamically manage firewall rules with [nftables](https://netfilter.org/projects/nftables/) and [NetworkManager](https://networkmanager.dev/).

This implementation is geared toward everyday devices (desktops and/or laptops) where network interfaces may be available (or unavailable) and their IP addresses may change.
Managing the firewall manually in these environments would be a horrible job, if not impossible.

## The strategy

The strategy, for me, in these scenarios is simple: Keep the `/etc/nftables.conf` file as simple as possible and, then, implement rules dynamically for the interfaces with [NetworkManager](https://networkmanager.dev/).

### nftables.conf

This file should be kept as simple as possible.
To achieve this, this file will only contain:

* Deny all rules: We will start from a deny all by default.
* Rules to allow all traffic for the loopback interface (lo): This is necessary for local applications and services to communicate with each other.
* ICMP rules: Since this rules will be applied per protocol and (in my case) the same ones will be applied to all interfaces.

>If you don't want all ICMP rules to apply to all interfaces, or you want different ICMP rules to apply to different interfaces, don't add these rules to the /etc/nftables.conf file and add them to the interface file(s).
{: .prompt-info}

```bash
#!/usr/sbin/nft -f

#/etc/nftables.conf

# Clear existing rules
flush ruleset

# Using 'inet' family for rules that apply to both IPv4 and IPv6
table inet filter {
    # Blacklisted IP addresses
    set blacklist_ipv4_static {
        type ipv4_addr;
        flags interval;
        auto-merge;
        elements = { 169.254.0.0/16, 192.0.2.0/24, 224.0.0.0/4, 240.0.0.0/4, 255.255.255.255, 0.0.0.0/8 };
    }

    set blacklist_ipv4 {
        type ipv4_addr;
        flags dynamic;
    }

    set blacklist_ipv6_static {
        type ipv6_addr;
        flags interval;
        auto-merge;
        elements = { fe80::/10, 2001:db8::/32, ff00::/8, ::/128 };
    }

    set blacklist_ipv6 {
        type ipv6_addr;
        flags dynamic;
    }

    # ICMP handling for incoming traffic (IPv4 & IPv6)
    chain ICMP_IN {
        # Allow replies for established/related ICMP sessions
        icmp type { echo-reply, destination-unreachable, time-exceeded } ct state { related, established } accept
        icmpv6 type { 129, 1, 2, 3, 4 } ct state { related, established } accept # echo-reply, dest-unreachable, packet-too-big, time-exceeded, parameter-problem

        # Drop incoming echo-requests (ping) by default
        icmp type echo-request drop
        icmpv6 type echo-request drop
    }

    # ICMP handling for outgoing traffic (IPv4 & IPv6)
    chain ICMP_OUT {
        # Allow outgoing echo-requests (ping) and established/related ICMP
        icmp type echo-request ct state { new, established } accept
        icmpv6 type echo-request ct state { new, established } accept
    }

    chain INPUT {
        type filter hook input priority 0; policy drop; # Default drop policy for incoming

        # Allow all traffic on the loopback interface (essential for local processes)
        iif "lo" accept
    }

    chain FORWARD {
        type filter hook forward priority 0; policy drop;
    }

    chain OUTPUT {
        type filter hook output priority 0; policy drop; # Default drop policy for outgoing

        # Allow all traffic on the loopback interface
        oif "lo" accept
    }
}
```

## The rules for interfaces

We create the script `10-nftables-hotplug` in `/etc/NetworkManager/dispatcher.d/` (as root) with the following content (the script is self explainatory):

At this point, we can create one file for all interfaces or one file for each interface.
To maintain a single file, if I needed other rules depending on the interface, I would handle the cases through conditionals in the code of this script.
I prefer to keep a single file, but this, as always, will depend on each user's preferences and needs.

```bash
#!/usr/bin/env bash

# /etc/NetworkManager/dispatcher.d/10-nftables-hotplug
# $1 = interface name
# $2 = connection status

delete_rules() {
    interface=$1
    chain=$2
    handles=$(sudo nft -a list chain inet filter $chain | grep "ifname \"$interface\"" | awk '{print $NF}')

    if [ -n "$handles" ]; then
        for handle in $handles; do
            sudo nft delete rule inet filter $chain handle $handle
        done
    fi
}

# Delete old rules
delete_rules "$1" INPUT
delete_rules "$1" OUTPUT

if [[ "$1" != lo && ("$2" = "up" || "$2" = "dhcp4-change" || "$2" = "dhcp6-change") ]]; then
    # Add rules to the INPUT chain

    # 1. Anti-spofing
    ipv4=$(ip addr show $1 | awk '$1=="inet"{gsub("/.*","",$2); print $2; next}')

    if [ -n "$ipv4" ]; then
        nft add rule inet filter INPUT iifname "$1" ip saddr "$ipv4" drop
    fi

    ipv6=$(ip addr show $1 | awk '$1=="inet6"{gsub("/.*","",$2); print $2; next}')

    if [ -n "$ipv6" ]; then
        nft add rule inet filter INPUT iifname "$1" ip6 saddr "$ipv6" drop
    fi

    # 2. Accept established and related traffic
    nft add rule inet filter INPUT iifname "$1" ct state { established, related } accept

    # 3. Drop traffic from known blacklisted ips
    nft add rule inet filter INPUT iifname "$1" ip saddr @blacklist_ipv4 drop
    nft add rule inet filter INPUT iifname "$1" ip6 saddr @blacklist_ipv6 drop

    # 4. Drop invalid traffic
    nft add rule inet filter INPUT iifname "$1" ct state invalid drop

    # 5. Drop invalid fragmented packets
    nft add rule inet filter INPUT iifname "$1" ip frag-off != 0 tcp dport 22 ct state new accept

    # 6. Jump to an ICMP chain
    nft add rule inet filter INPUT iifname "$1" meta l4proto { icmp, icmpv6 } jump ICMP_IN

    # 7. Limit and drop new connections to prevent floods and scans
    nft add rule inet filter INPUT iifname "$1" tcp limit rate 5/second burst 15 packets counter add @blacklist_ipv4 { ip saddr } drop
    nft add rule inet filter INPUT iifname "$1" tcp limit rate 5/second burst 15 packets counter add @blacklist_ipv6 { ip6 saddr } drop
    nft add rule inet filter INPUT iifname "$1" udp limit rate 5/second burst 15 packets counter add @blacklist_ipv4 { ip saddr } drop
    nft add rule inet filter INPUT iifname "$1" udp limit rate 5/second burst 15 packets counter add @blacklist_ipv6 { ip6 saddr } drop

    # 8. Accept all incoming traffic
    nft add rule inet filter INPUT iifname "$1" ct state new accept

    # Add rules to the OUTPUT chain

    # 1. Jump to an ICMP chain
    nft add rule inet filter OUTPUT oifname "$1" meta l4proto { icmp, icmpv6 } jump ICMP_OUT

    # 2. Accept all outgoing traffic
    nft add rule inet filter OUTPUT oifname "$1" accept
fi
```

>Scripts placed in /etc/NetworkManager/dispatcher.d/ should be named with a number (between 00 and 99) followed by a descriptive number.
>The naming convention it’s XX-name.
>[NetworkManager](https://networkmanager.dev/) executes these scripts in numerical order from lowest to highest.
{: .prompt-info}

>Note that, in this example, all incoming and outgoing traffic is allowed.
>You should only allow traffic for your trusted services.
{: .prompt-warning}

With this script, every time an interface connects or changes IP, its old rules will be deleted, and new ones will be added, taking into account the new IP.
If an interface is disconnected, its associated rules will be deleted.

*Enjoy! ;)*
