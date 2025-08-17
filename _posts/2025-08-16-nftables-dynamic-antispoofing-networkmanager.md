---
title: How to prevent IP spoofing with nftables and NetworkManager
date: 2025-08-16 00:00:01 +0000
categories: [hardening, firewall]
tags: [hardening, firewall, iptables, nftables, antispoofing, spoofing]
---

Add dynamic IP antispoofing rules to your [nftables](https://netfilter.org/projects/nftables/) firewall with [NetworkManager](https://networkmanager.dev/).
Prevent spoofing of your computer's network interfaces using dynamic [nftables](https://netfilter.org/projects/nftables/) rules.

## Why?

I while ago I wrote my article [Migrate from iptables to nftables](https://rubenhortas.github.io/posts/migrate-iptables-nftables/), in which I explained how to migrate from [iptables](https://netfilter.org/projects/iptables/index.html) to [nftables](https://netfilter.org/projects/nftables/) and why.
I guess many of you have noticed that I included a static antispoofing rule.

The computer on which I configured that firewall acts as a server, it only has that network interface and doesn't change networks.
So, configuring a static antispoofing rule on that computer, is very simple and very easy to mantain.

But, what happens when the computer it's a laptop (for example)?
I latop typically has two network interfaces, and may frequently switch between networks.
Mantaining static antispoofing filtering rules in this scenario would be horrible.

## What is IP spoofing?

Spoofing is the act of faking your identity in a digital environment to impersonate a trusted source.
There are many types of spoofing: email spofing, web spoofing, caller ID spoofing, IP spoofing...
For brevity, In this post, I will focus only on IP spoofing.
Focusing on IP spoofing of our computer interfaces using dynamc rules with [nftables](https://netfilter.org/projects/nftables/) and [NetworkManager](https://networkmanager.dev/).

IP spoofing is a type of spoofing where an attacker creates IP packets with a false source IP address.
This is possible because the design of the internet's communication protocols, specifically TCP/IP, doesn't always verify the source IP address of a packet.

## Why is IP spoofing dangerous

IP spoofing is dangerous because it allows attackers to bypass security measures and launch powerful and stealthy attacks, for example:

* **DDoS**

  Attackers floods a server with an overwhelming amount of traffic.
  By using spoofed IP addresses, they make it nearly impossible to block the malicious traffic, since the packets may appear to come from different random IPs, trusted IPs, or, in our case, from legitimate IPs from our own machine.

* **Bypass authentication**

  Some systems rely on IP address authentication.
  If an attacker can spoof that IP, he will be able to access the "protected" resources.

* **Masking identity**

  By spoofing the source IP, an attacker remains anonymous thorough the attack.
  This makes incredibly difficult for sysadmins trace the attack back to its origin.

## Adding dynamic IP spoofing rules to [nftables](https://netfilter.org/projects/nftables/) to avoid IP spoofing

> For this to work we need to have the "INPUT" table and the "filter" chanin created.
{: .prompt-info}

We create the script `10-nftables-antispoofing` in `/etc/NetworkManager/dispatcher.d/` (as root) with the following content (the script is self explainatory):

```bash
#!/usr/bin/env bash

# $1 = interface name
# $2 = connection status

delete_rules() {
    local interface="$1"
    local family="$2"

    nft -a list ruleset | grep "iifname \"$interface\" $family" | while read -r line; do
        handle=$(echo "$line" | sed -n 's/.* handle \([0-9]*\).*/\1/p')

        if [ -n "$handle" ]; then
            nft delete rule inet filter INPUT handle "$handle"
        fi
    done
}

# When an interface is connected or its configuration changed
if [ "$2" = "dhcp4-change" ]; then
	ipv4=$(ip addr show $1 | awk '$1=="inet"{gsub("/.*","",$2); print $2; next}')
	ipv6=$(ip addr show $1 | awk '$1=="inet6"{gsub("/.*","",$2); print $2; next}')

    	if [ -n "$ipv4" ]; then
    		delete_rules "$1" "ip" # Delete old antispoofing rules for the interface and family
    	  nft add rule inet filter INPUT iifname "$1" ip saddr "$ipv4" drop # Add new antispoofing rules
    	fi

    	if [ -n "$ipv6" ]; then
    		delete_rules "$1" "ip6" # Delete old antispoofing rules for the interface and family
    	  nft add rule inet filter INPUT iifname "$1" ip6 saddr "$ipv6" drop # Add new antispoofing rules
    	fi
fi
```

Scripts placed in `/etc/NetworkManager/dispatcher.d/` should be named with a number (between 00 and 99) followed by a descriptive number.
The naming convention it's `XX-name`.
[NetworkManager](https://networkmanager.dev/).executes these scripts in numerical order from lowest to highest.

We give execution permissions to the script:

`sudo chmod +x /etc/NetworkManager/dispatcher.d/10-nftables-antispoofing`

Now, the script will be executed automatically when we connect or disconnect a network interface or whe we change its configuration.

*Enjoy! ;)*
