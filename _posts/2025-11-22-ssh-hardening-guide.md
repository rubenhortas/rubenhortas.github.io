---
title: Essential SSH hardening guide
date: 2025-11-22 00:00:01 +0000
categories: [hardening, ssh]
tags: [hardening, ssh, fail2ban]
---

Harden the default SSH configuration settings to reduce your system's attack surface in your server and client.
Also, install and configure [fail2ban](https://github.com/fail2ban/fail2ban) in your server.

## Why?

SSH is the privileged gateway for remote administration of almost any server or network device (in many cases, this includes personal computers and/or gadgets), this makes SSH a prime target for attackers.

We must harden SSH to prevent attackers from exploiting weak configurations and gaining access to our devices, thereby protecting both our devices and our network (other devices).

The goal of hardening is to eliminate (or reduce) inherent vulnerabilities that may exist in the default configuration of a service.

## Keep your system updated ;)

Keep your system updated is the primary way to patch security vulnerabilities.
So, the fisrt step, and one of the most importants steps in hardening (both on the server side and on the client side) is to keep our system updated.

## Server hardening

### The configuration file

To harden SSH, on the server side, we are going to change certain settings in the SSH configuration file: `/etc/ssh/sshd_config`.

To configure the values, we can uncomment the lines (if they are commented out) and edit the values.
However, I prefer to leave the original line commented out and place the line with the new value directly below it.
This is a quick way to see if we have modified a default configuration (and what its default value was) when we review the file, when we add new configurations, or when we modify existing settings.
If you prefer to edit the values directly, I recommend you make a backup of the original configuration file.

After changing the settings, we'll need to restart the SSH service for the changes to take effect.

### Change the default SSH port

Security by obscurity is never the solution, and **changing the default port isn't a foolproof security measure**, but it will deter automated attacks.

`Port 22222`

### Disable X11 forwarding

If we aren't going to use graphical applications via SSH, it's best to disable it.
The main reason for disabling X11 forwarding is the possibility of an attacker gaining access to your local X server session if the remote SSH server is compromised.

`X11Forwarding no`

### Disable agent forwarding

We will disable agent forwarding because it creates a security risk by allowing the remote machine to use our local SSH private keys for *any* subsequent connection.

`AllowAgentForwarding no`

### Restrict root login

On our systems, the `root` account should be disabled already.
Disabling the `root` account and elevating privileges using `sudo` is a good security practice.
Anyway, we are going to disable it:

`PermitRootLogin no`

### Limit users and/or groups that can log in:

We will specify which users and/or groups are allowed to access through SSH.

```
AllowUsers rubenhortas rhortas
AllowGroups rubenhortas rhortas ssh
```

>The users and groups are examples.
>You have to add your own users and groups.
{: .promt-warning}

If a user is listed in `AllowUsers` or belongs to a group listed in `AllowGroups`, they are allowed access.
If neither `AllowUsers` nor `AllowGroups`is specified, all users are permitted to log in (assuming they have credentials).

>You should consider using only one approach (`AllowUsers` or `AllowGroups`) if possible.
>My personal approach is use only `AllowUsers` because I have few users to manage.
{: .prompt-info}

### Configure the idle timeout

We are going to set up an inactiviy timeout to prevent sessions from remaining open indefinitely.

```
ClientAliveInterval 300 # The server will send the client a message every 300 seconds if there is no activity
ClientAliveCountMax 2   # If the clinent does not respond to 2 messages, the connection is terminated
```

>At this point, we have to be careful not to set values so restrictive that they cut off our connections.
{: .prompt-warning}

>Alternatively, for non-SSH specific shell timeouts, consider using the TMOUT environment variable in the shell profile.
{: .prompt-info}

### Disable password authentication.

Use only public/private key authentication.

You can learn how to use RSA keys as authentication method here: [SSH without password, using RSA keys](https://rubenhortas.github.io/posts/ssh-without-password-using-rsa-keys/)

After we have RSA key authentication set up and working, we are going to disable `PasswordAuthentication`:

`PasswordAuthentication no`

### Explicity enforce Protocol 2

While modern SSH server sowftware defaults to using the secure `Protocol 2`, explicity setting the directive is a critical step in hardening our server.
SSH `Protocol 1`is insecure and has known severe vulnerabilities.
Explicty setting the configuration ensures that `Protocol 1`is never accepted, even if a client attempts to negotiate it.

`Protocol 2`

### Force the use of modern algorithms

We will define a strict list of preferred modern algorithms.
The server's list will determine the algorithms used if the client supports them.

#### Key Exchange Algorithms

We need to harden our Key Exchange Algorithms using algorithms that uses modern elliptic curve cryptography (ECC) and strong Diffie-Hellman methods, offering excellent security and performance.

`KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256`

#### Ciphers (encryption)

Ciphers encrypt the data channel.
It's best to prioritize **Authenticated Encryption with Associated Data (AEAD)** ciphers like `chacha20-poly1305` and `aes-gcm`.

`Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr`

#### MACs (integrity)

MACs (Message Authentication Codes) ensure the data hasn't been tampered with in transit.
We should prioritize **Encrypt-then-MAC (EtM)** algorithms.

`MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com`

### Reboot SSH service

We have to restart the SSH service for the changes to take effect.

`sudo systemctl restart ssh`

### Bonus track: fail2ban

[fail2ban](https://github.com/fail2ban/fail2ban) monitors failed login attempts in the logs and blocks the IPs addresses they originate from in the firewall.

#### Installation

`sudo apt install fail2ban`

#### Configuration

The default configuration is acceptable, but I preffer to tighten it.
To configure [fail2ban](https://github.com/fail2ban/fail2ban) we will edit the file `/etc/fail2ban/jail.local`.
My `/etc/fail2ban/jail.local` file (commented) as an example:

```
[sshd]                      # Defines a jail targeting the SSH daemon
enabled = true              # Activates this jail
backend = systemd           # Method to monitor the logs
port = ssh
filter = sshd               # Use the predefined sshd filter to detect SSH authentication failures in the logs
maxretry = 1                # Maximum number of failed login attempts allowed before an IP address is banned
findtime = 1m               # Time window during which the maxretry count is calculated
bantime = 1m                # Ban duration
```

>There was a time when I had the ban set to last forever, but it ended up with so many IPs that it degraded performance.
{: .prompt-info}

My `fail2ban` configuration could be quite aggressive.
Since I have disabled password authentication, any failed login attempt is highly unlikely to be a legitimate user.
Therefore, setting `maxretry = 1` and `findtime = 1m` is effective, because any password attempt will be considereded an attack, justifying the aggressive ban.

#### Enable and start fail2ban service

```
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

## Client hardening

### The configuration file

To harden SSH, on the client side, we are going to change certain settings in the SSH configuration file: `~/.ssh/config`.
This file allows us to define custom settings for individual remote hosts or groups of host.

#### Specific host configuration

To add a specific host, you create a new configration block starting with the `Host` option and a unique alias for that host:

```
Host rubenhortas-dev
    HostName 192.168.1.1
    KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    ...
```

#### Default all hosts configurations

The `Host *` will work for all hosts, but it's applied hierarchically.
The `Host *` settings will be applied to every single connection, unless they are overriden by a more specific `Host` entry.
We will add the hardening settings in this block.

```
Host *
    PasswordAuthentication no
    KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    ...
```

### Restrict key files permissions

```
chmod 700 ~/.ssh
chmod 600 ~./ssh/id_*
```

### Private keys

Better with passphrase? The passphrase prevents the attacker from using the keys if they fall into the wrong hands.

### Prefer public key authentication

`PreferredAuthentications publickey`

This forces the use of public keys as form of authentication.

### Disable password authentication

`PasswordAuthentication no`

This disables the login with password.

### Disable host based authentication

`HostBasedAuthentication no`

Host based authentication is less secure.

### Key verification

We will force the client to verify that the server key matches the key stored in the `known_hosts` file to prevent Man In The Middle (MITM) attacks.

`StrictHostKeyChecking yes`

### Force the use of modern algorithms

On the client side, we will enforce the same strong ciphers that we chose for the server.
This will ensure that, even if we connect to a legacy server, we will use the best available options.

#### Key Exchange Algorithms

`KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256`

#### Ciphers (Encryption)

`Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com`

#### MACs (integrity)

`MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com`

### Disable X11 forwarding

As on the server side, we will disable X11 forwarding to prevent exposing our graphical session.

`ForwardX11 no`

### Disable agent forwarding

As on the server side, we will disable agent forwarding to prevent that a compromised server from using our keys to (potentially connect to other servers.

`ForwardAgent no`

*Thanks for reading! ;)*
