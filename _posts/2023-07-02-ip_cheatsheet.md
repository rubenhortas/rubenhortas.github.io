---
title: ip cheatsheet 
date: 2023-07-2 00:00:01 +0000
categories: [cheatsheets, gnu/linux]
tags: [cheatsheet, gnux/linux, ip] 
---

I'm an old school guy. 
I lerned to use `ifconfig`in the old timems and whith `ifconfig`I continue until today.

But, for a while, there is a new contender `ip`. 
`ip` is a network tool called to replace `ipconfig`, is installed on all modern gnu/linux distributions, and it's more powerful than `ipconfig`.
So, it's time to learn `ip`, and, to do it, what better way to do it than by keeping a cheeatsheet with the most used commands? 

# Syntax

```
ip [OPTIONS] OBJECT {COMMAND}  
```

# Display network interfaces information

```
ip addr [show]
```

```
ip link show 
```

# Display IPv4 addresses

```
ip -4 addr
```

# Display IPv6 addresses

```
ip -6 addr
```

# Display information from a single network interface

```
ip addr show dev eth0
```

```
ip link show eth0 
```

# Bring down a network interface

```
ip link set eth0 up
```

# Bring down a network interface

```
ip link set eth0 down
```

# Assing an IP address to a network interface

```
sudo ip address add 10.0.0.1/24 dev eth0
```

# Deleting an IP address

```
sudo ip address del 10.0.0.1/24 dev eth0
```

*Enjoy! ;)*
