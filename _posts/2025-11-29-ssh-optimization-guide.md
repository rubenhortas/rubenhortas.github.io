---
title: SSH optimization guide
date: 2025-11-29 00:00:01 +0000
categories: [ssh, performance]
tags: [ssh, performance, optimization, multiplexing, rsync]
---

Speed up SSH for quicker data transfer.

## Why?

With current internet speeds and file sizes, it's common to find ourselves frequently transferring (increasingly) large files.
This is good, but it's better if we can save a few minutes on each transfer.

A number of factors affect SSH speed, such as network latency and bandwidth, but also client and server configurations.
We can speed up our transfers by changing a few settings.

## Optimize SSH server

### The configuration file

To optimize SSH, on the server side, we are going to tweak the SSH server configuration file: /etc/ssh/sshd_config.

To configure the values, we can uncomment the lines (commented out) and edit the values. However, I prefer to leave the original line commented out and place the line with the new value directly below it.
This is a quick way to see if we have modified a default configuration (and what its default value was) when we review the file, when we add new configurations, or when we modify existing settings.
If you prefer to edit the values directly, I recommend you make a backup of the original configuration file.

After changing the settings, we’ll need to restart the SSH service for the changes to take effect.

### Avoid reverse DNS lookups

If the server can't resolve the hostnames of the IPs that attempt to connect, it could introduce a noticeable lag or latency.

`UseDNS no`

### Disable GSSAPIAuthentication

GSSAPI can cause delays if it isn't properly configured or if it isn't used.

`GSSAPIAuthentication no`

### KexAlgorithms and Ciphers

SSH security relies on cryptography, but the algorithms used for key exchange (KexAlgorithms) and encryption (Ciphers) can introduce significant latency, especially on older hardware or high-latency networks.
By prioritizing modern, high-speed algorithms, we can minimize the CPU time spent on encryption/decryption and speed up transfers.
This is especially useful in systems with a limited CPU.

> For this to take effect, the client and the server must be configured identically.
{: .prompt-info}

#### Optimizing KexAlgorithms

The Key Exchange (KEX) Algorithm determines how the client and server agree upon a shared secret key for the session.
Older algorithms, such as those based on traditional Diffie-Hellman, are often slower.

The best practice is to select modern, high-speed algorithms like those based on Elliptic Curve Cryptography (ECC).
The curve25519-sha256@libssh.org algorithm is the current gold standard for speed and security.

`KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256`

#### Optimizing Ciphers

The Cipher is the encryption algorithm used to scramble the actual data being transferred.
The fastest Ciphers are generally those that are designed to be efficiently processed on modern CPUs.

A high-performance choice is `ChaCha20-Poly1305`, which is implemented in OpenSSH as `chacha20-poly1305@openssh.com`.
This algorithm is often significantly faster than legacy `AES-CBC` modes, particularly on systems without `AES` acceleration (common in cloud VMs).

`Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com`

> The order matters.
> SSH client and server will choose the first common algorithm they both support from their respective lists.
> By placing the fastest algorithms at the beginning of the list, you ensure they are prioritized for maximum speed.
{: .prompt-warning}

## Optimize SSH client

### The configuration file

To optimize SSH, on the client side, we are going to change certain settings in the SSH configuration file `~/.ssh/config`.

This file defines custom configurations for the user and specific hosts or groups of hosts:

* Specific host configuration

To add a specific host, you simply create a new configuration block starting with the `Host` option and a unique alias for that host:


```
Host rubenhortas-dev
    HostName 192.168.1.1
    Compression yes
    ControlMaster auto
    ControlPath ~/.ssh/control_sockets/%r@%h:%p
    ControlPersist 600
    KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
    LogLevel ERROR
    ...
```

* Default configuration for all hosts

The `Host *` entry applies to all hosts, but its settings are applied hierarchically.
The `Host *` settings will be applied to every single connection, unless they are overridden by a more specific `Host` entry.

```
Host *
    Compression yes
    ControlMaster auto
    ControlPath ~/.ssh/control_sockets/%r@%h:%p
    ControlPersist 600
    KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
    LogLevel ERROR
    ...
```

As we want to speed up all our connections, we will add the options in the `Host *` section.

### Use compression

Compression can speed transfers, especially in connections with low bandwidth or when we transfer many small files.

>Compression is counterproductive in environments where the CPU is limited.
{: .prompt-warning}

`Compression yes`

### Connection re-use (or multiplexing)

This is one of the best ways to speed up repeated SSH connections.
It opens an initial connection and, then, reuses the same connection for subsequent commands or file transfers.

```
    ControlMaster auto
    ControlPath ~/.ssh/control_sockets/%r@%h:%p
    ControlPersist 600
```

* ControlMaster auto: Opens a new connection if there is none available, or uses an existing connection.
* ControlPath: The directory path where the control socket will be stored.
* ControlPersist: Keeps the master connection open for a specified time.

>Make sure that the `ControlPath` directory exists
{: .prompt-warning}

### KexAlgorithms and Ciphers

We will configure the algorithms and ciphers the same way as on the server.
We are going to prioritize modern, high-speed algorithms to minimize the CPU time spent on encryption/decryption and speed up transfers.

```
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
```

### Reduce verbosity

Although it does not directly affect connection speed, a lower level of logging on the client can make the interface feel smoother.

`LogLevel ERROR`

### rsync instead of scp

This is my personal favorite.
*Always* use `rsync` instead of `scp`, not only for big or resumable transfers.
The key benefit of `rsync` is its delta-transfer algorithm.
`rsync` only transfers the differences between files, which is a massive speed win on repeated transfers of large files.
`rsync`also uses SSH under the hood, so the SSH optimizations are applied also to rsync transfers.

We will define a bash alias in our `~/.bashrc` with our favorite parameters, and then we will use `rsync` instead of `scp`:

` alias rsync='rsync --recursive --partial --progress --human-readable --verbose'`

> With the `--progress` option we can see the file transfer progress.
> Something we can't see using `scp`
{: .prompt-info}

*Thanks to [Rodrigo Rega](https://rodrigorega.es/) for the advices.*

**Thanks for reading! :)*
