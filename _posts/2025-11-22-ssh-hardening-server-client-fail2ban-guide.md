---
title: Essential SSH hardening guide (Server, client and fail2ban)
date: 2025-11-22 00:00:01 +0000
categories: [hardening, ssh]
tags: [hardening, ssh, fail2ban]
---

Harden the default SSH configuration settings to reduce your system's attack surface on both your server and client.
Also, install and configure [fail2ban](https://github.com/fail2ban/fail2ban) in your server.

## Why?

SSH is the privileged gateway for remote administration of almost any server or network device (in many cases, this includes personal computers and/or gadgets), which makes it a prime target for attackers.

We must harden SSH to prevent attackers from exploiting weak configurations and gaining access to our devices, thereby protecting both our devices and the rest of our network.

The goal of hardening is to eliminate (or reduce) inherent vulnerabilities that may exist in the default configuration of a service.

## Keep your system updated ;)

Keeping your system updated is the primary way to patch security vulnerabilities.
Therefore, the first step, and one of the most important steps in hardening (both on the server side and on the client side) is to keep your system updated.

## Server hardening

### The configuration file

To harden SSH, on the server side, we are going to change certain settings in the SSH configuration file: `/etc/ssh/sshd_config`.

To configure the values, we can uncomment the lines (commented out) and edit the values.
However, I prefer to leave the original line commented out and place the line with the new value directly below it.
This is a quick way to see if we have modified a default configuration (and what its default value was) when we review the file, when we add new configurations, or when we modify existing settings.
If you prefer to edit the values directly, I recommend you make a backup of the original configuration file.

After changing the settings, we'll need to restart the SSH service for the changes to take effect.

### Change the default SSH port

Security by obscurity is never the solution, and **port change is hardly a silver bullet**, but it will deter automated attacks.

`Port 1023` # Generally used for testing

> Changing the port can create a security issue.
>The catch here is that changing the port can create a security issue called "Non-Privileged Port Binding" or "Ephemeral Port Hijacking".
While Linux restricts privileged ports (1-1023) to the root user, any unprivileged local user is able to bind and listen to unprivileged ports (1024-65535).
Therefore, if our SSH server is running on, say, port 2222, and it crashes, a local user could start a fake SSH server on this port.
{: .prompt-warning}

My choice here is to implement Port Forwarding from the external firewall.
I would open port 22222 in the firewall and redirect that traffic to port 22 of the machine running the SSH server.

### Reduce MaxAuthTries

Reduce the maximum number of authentication attempts permitted per network connection.
Once the limit is reached, the server drops the connection.
This will help us mitigate possible brute-force attacks.

`MaxAuthTries 3`

### Reduce MaxSessions

Reduce the number of "sessions" (like shell windows or file transfers) that can be opened over a single network connection (multiplexing).
This will help us to prevent resource exhaustion and minimize the potential damage of a session hijacking (since the attacker cannot open new sessions without re-authenticating).

`MaxSessions 2`

>SSH clients often try several authentication methods (like different SSH keys) automatically. If you have 5 keys in your ssh-agent, you might hit a limit of 3 before you even get to type a password or login.
{: .prompt-warning}

### Disable X11 forwarding

If we are not going to use graphical applications via SSH, it is best to disable it.
The main reason for disabling X11 forwarding is the possibility of an attacker gaining access to your local X server session if the remote SSH server is compromised.

`X11Forwarding no`

### Disable TCP Forwarding

By disabling TCP forwarding prevents a compromised SSH session from being used as a bridge to attack other systems.
If an attacker gains access to a limited user account, they can use TCP forwarding to bypass firewalls and poking around your internal network.
Disabling TCP forwarding an attacker cannot tunner network traffic through the SSH connection.

`AllowTcpForwarding no`

### Disable agent forwarding

We will disable agent forwarding because it creates a security risk by allowing the compromissed remote machine to use our local SSH private keys for *any* subsequent connection.

`AllowAgentForwarding no`

### Restrict root login

On a well-configured system, the `root` account should be disabled already.
Disabling the `root` account and elevating privileges using `sudo` is a good security practice.
Anyway, we are going to disable it:

`PermitRootLogin no`

### Limit users and/or groups that can log in:

We will specify which users and/or groups are allowed to access through SSH.

```
AllowUsers rubenhortas rhortas
AllowGroups rubenhortas rhortas ssh
```

> The users and groups are examples.
> You have to add your own users and groups.
{: .promt-warning}

If a user is listed in `AllowUsers` or belongs to a group listed in `AllowGroups`, they are allowed access.
If neither `AllowUsers` nor `AllowGroups` is specified, all users are permitted to log in (assuming they have credentials).

> You should consider using only one approach (`AllowUsers` or `AllowGroups`) if possible.
> My personal approach is to use only `AllowUsers` because I have few users to manage.
{: .prompt-info}

### Configure the idle timeout

We are going to set up an inactiviy timeout to prevent sessions from remaining open indefinitely.

```
ClientAliveInterval 300 # The server will send the client a message every 300 seconds if there is no activity
ClientAliveCountMax 2   # If the client does not respond to 2 messages in 10 minutes (300 x 2 = 600 seconds), the connection is terminated
```

> At this point, we have to be careful not to set values so restrictive that they cut off our connections.
{: .prompt-warning}

> Alternatively, for non-SSH specific shell timeouts, consider using the TMOUT environment variable in the shell profile.
{: .prompt-info}

### Disable password authentication.

Use only public/private key authentication.

You can learn how to use RSA keys as authentication method here: [SSH without password, using RSA keys](https://rubenhortas.github.io/posts/ssh-without-password-using-rsa-keys/)

After we have RSA key authentication set up and working, we are going to disable `PasswordAuthentication`:

`PasswordAuthentication no`

And we will to enable `PubkeyAuthentication`:

`PubkeyAuthentication yes`

### Explicity enforce Protocol 2

While modern SSH server software defaults to using the secure `Protocol 2`, and the modern and secure approach is to simply *not* specify the protocol, explicitly setting the directive could be a critical step in hardening our server.
SSH `Protocol 1`is insecure and has known severe vulnerabilities.
Explicty setting the configuration ensures that `Protocol 1`is never accepted, even if a client attempts to negotiate it.

`Protocol 2`

### Force the use of modern algorithms

We will define a strict list of preferred modern algorithms.
The server's list will determine the algorithms used if the client supports them.

#### Key Exchange Algorithms

We need to harden our Key Exchange Algorithms using algorithms that uses modern elliptic curve cryptography (ECC) and strong Diffie-Hellman methods, offering excellent security and performance.

`KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256`

#### MACs (integrity)

MACs (Message Authentication Codes) ensure the data has not been tampered with in transit.
We should prioritize **Encrypt-then-MAC (EtM)** algorithms.

`MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com`

#### Ciphers (encryption)

Ciphers encrypt the data channel.
It's best to prioritize **Authenticated Encryption with Associated Data (AEAD)** ciphers like `chacha20-poly1305` and `aes-gcm`.

`Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr`

### Explicity disable rhosts

rhosts was a weak method to authenticate systems. 
It defines a way to trust another system simply by its IP address.
By default is already disabled, but it's better to disable it explicitly.

`IgnoreRhosts yes`

### Reboot SSH service

We have to restart the SSH service for the changes to take effect.

`sudo systemctl restart ssh`

### Bonus: fail2ban

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

> There was a time when I had the ban set to last forever, but it ended up with so many IPs that it degraded performance.
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

### The configuration files

To harden SSH, on the client side, we are going to change certain settings in one of the two SSH configuration files: `/etc/ssh/ssh_config` or `~/.ssh/config`.

* `/etc/ssh/ssh_config`

    This file holds the system-wide configuration.
    It defines the default configuration for all users on the system.

    To configure the values, we can uncomment the lines (if commented out) and edit the values.
    However, as I said before, I prefer to leave the original line commented out and place the line with the new value directly below it.
    If you prefer to edit the values directly, I recommend you make a backup of the original configuration file.

* `~/.ssh/config`

    This file defines custom configurations for the user and specific hosts or groups of hosts:

    * Specific host configuration

    To add a specific host, you create a new configuration block starting with the `Host` option and a unique alias for that host:

    ```
    Host rubenhortas-dev
        HostName 192.168.1.1
        PreferredAuthentications publickey
        PasswordAuthentication no
        HostBasedAuthentication no
        StrictHostKeyChecking ask
        KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
        MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com
        Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
        ForwardX11 no
        ForwardAgent no
        ...
    ```

    * Default configuration for all hosts

    The `Host *` entry applies to all hosts, but its settings are applied hierarchically.
    The `Host *` settings will be applied to every single connection, unless they are overridden by a more specific `Host` entry.
    We will add the hardening settings in this block.

    ```
    Host *
        PreferredAuthentications publickey
        PasswordAuthentication no
        HostBasedAuthentication no
        StrictHostKeyChecking ask
        KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
        MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com
        Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
        ForwardX11 no
        ForwardAgent no
        ...
    ```

### Restrict key files permissions

```
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_*
chmod 64 ~/.ssh/known_hosts
```

### Private keys

Better with passphrase? The passphrase prevents the attacker from using the keys if they fall into the wrong hands.

### Prefer public key authentication

`PreferredAuthentications publickey`

This forces the use of public keys as form of authentication.

### Prioritizing Public Keys

`PasswordAuthentication no`

This ensures that our client always prioritizes the use of public keys above all other password-based methods.

### Disable host based authentication

`HostBasedAuthentication no`

Host based authentication is less secure.

### Key verification

We must force the client to verify that the server's key matches the key stored in the `known_hosts` file to prevent Man-In-The-Middle (MITM) attacks.
The relevant configuration options are setting `StrictHostKeyChecking` to `ask` or `yes`.

We need a balance between security and convenience.
My preferred approach is to configure `StrictHostKeyChecking ask` for all hosts by default, and set `StrictHostKeyChecking yes` to sensitive hosts (like my own).

A change in a host key is usually due to legitimate administrative reasons (e.g. reinstallations, replacement, changes in key/type algorithms...).
However, althoug less common, it could be due to malicious acts.

There are certain types of hosts where key changes are not uncommon, such as Capture The Flag (CTF) challenge servers or frequently rebuild virtual machines.
But, on my own hosts, a change in keys should never happen without my knowledge.

By setting `StrickHostKeyChecking ask` globally, as default, and `StrictHostKeyChecking yes` for critical hosts, I can connect to a host susceptible to key changes and, if the key had changed, I can decide whether to accept the new key. Crucially, if the key had changed on a host where it should not, the strict setting will prevent the connection entirelly, inmediately alerting me to a potential security issue.

if the key had changed on a host where should not, I will find out something is wrong when I connect.

`StrictHostKeyChecking ask`

    * First connection: the SSH client will prompt you to manually confirm if you trust the server's key before adding it to your `known_hosts` file.
    * Server key changed: The connection will be refused.

`StrictHostKeyChecking yes`

    * First connection: The SSH client will refuse the connection immediately if the host's key is not already present in your `known_hosts` file.
    You must manually add the host key before attempting to connect.
    * Server key changed: The connection will be refused.

### Force the use of modern algorithms

On the client side, we will enforce the same strong ciphers that we chose for the server.
This will ensure that, even if we connect to a legacy server, we will use the best available options.

#### Key Exchange Algorithms

`KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256`

#### MACs (integrity)

`MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com`

#### Ciphers (Encryption)

`Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com`

### Disable X11 forwarding

As on the server side, we will disable X11 forwarding to prevent exposing our graphical session.

`ForwardX11 no`

### Disable agent forwarding

As on the server side, we will disable agent forwarding to prevent that a compromised server from using our keys to (potentially connect to other servers.

`ForwardAgent no`

## Acknowledgments

*Thanks to [Rodrigo Rega](https://rodrigorega.es/) and [Alberto Mouriz](https://github.com/AlbertoMouriz) for the advice.*

*Thanks for reading! ;)*
