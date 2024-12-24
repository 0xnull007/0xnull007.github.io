---
layout: post
title: Demystifying the User Authentication in Linux
tags:
  - Linux
  - /etc/passwd
  - /etc/shadow
author: Abdul Qadir
comments: true
permalink: /posts/demystifying-user-authentication-in-linux
description: Explored the user authentication mechanism in Linux
---

## A Brief History

In early Unix and Linux systems, user account information, including encrypted passwords, was stored in the `/etc/passwd` file. This file was readable by all users, which posed a significant security risk. Malicious users could potentially access the encrypted passwords and attempt to crack them offline. As security concerns grew, the need for a more secure method of storing password information became apparent. This led to the development of the `/etc/shadow` file in the 1980s. The shadow password system was first implemented in SunOS and later adopted by various Unix-like operating systems, including Linux. You can read [Password Security: A Case History](https://www.bell-labs.com/usr/dmr/www/passwd.ps) by Robert Morris and Ken Thompson for insights into the historical evolution of password storage mechanisms.

The introduction of `/etc/shadow` allowed for the separation of sensitive password data from other user account information, significantly enhancing system security. This change marked a crucial evolution in Unix and Linux security practices, addressing a longstanding vulnerability in user authentication systems.

## Prerequisites

In this post we will look at the user authentication mechanism in the modern Linux systems. So, lets look at the format of two user authentication related files `/etc/passwd` and   `/etc/shadow`  from my Ubuntu 22.04 with Linux Kernel v6.8.0. 

```bash
/etc/passwd

null:x:1001:1001:Nothing,,,:/home/null:/bin/bash
```

Each entry in `/etc/passwd` has the following seven fields separated by colon `:`.

1. User name or login name
2. Encrypted password
3. User ID
4. Group ID
5. User details
6. User's home directory
7. User's login shell

```bash
/etc/shadow

null:$y$j9T$0XNlzjOIPFoBIudkyHauJ.$cHyCfoQcxDm9ktgcYr5rbvrdaBcAwj8vwXNgMql2ffA:20081:0:99999:7:::
```

Each line in the `/etc/shadow` file represents an individual user account. It contains the following nine fields separated by colon `:`.

1. Username
2. Encrypted(Hashed) password 
3. Date of last password change
4. Minimum required days between password changes
5. Maximum allowed days between password changes
6. Number of days in advance to display password expiration message
7. Number of days after password expiration to lock the account
8. Account expiration date
9. Reserve field

## The Password Field

The password field further contains some sub-fields separated by dollar sign `$` . The first sub-field is ID of the hashing algorithm which is used to store the password. Following are the some hashing algorithms with their IDs.

1. `$1$` is MD5
2. `$2a$` is Blowfish
3. `$2y$`is Blowfish
4. `$5$` is SHA-256
5. `$6$` is SHA-512
6. `$y$` is yescrypt

According to [openwall](https://www.openwall.com/) 

> yescrypt is the default password hashing scheme on recent [ALT Linux](https://en.altlinux.org/), [Arch Linux](https://archlinux.org/news/changes-to-default-password-hashing-algorithm-and-umask-settings/), [Debian 11+](https://www.debian.org/releases/bullseye/amd64/release-notes/ch-information.en.html#pam-default-password), [Fedora 35+](https://fedoraproject.org/wiki/Changes/yescrypt_as_default_hashing_method_for_shadow), [Kali Linux](https://www.kali.org/) 2021.1+, and Ubuntu 22.04+. It is also supported in Fedora 29+, RHEL 9+, and [Ubuntu 20.04+](https://manpages.ubuntu.com/manpages/focal/en/man5/crypt.5.html), and is recommended for new passwords in [Fedora CoreOS](https://docs.fedoraproject.org/en-US/fedora-coreos/authentication/#_using_password_authentication).

Before yescrypt, mostly the password format was `$id$salt$hash`.  Here the `salt` is a random number generated (by reading the real time clock) at the time of password generation which is concatenated with the user password and then the `hash` of that resultant string is stored as the third field. While in case of yescrypt, the password format is `$id$cost_parameters$salt$hash`. Here the cost parameters define the memory hardness of the algorithm which refers to the requirement that a function needs a large amount of memory to compute its output efficiently. This characteristic is especially important for defending against hardware-accelerated attacks. You can watch [yescrypt: large-scale password hashing (Alexander Peslyak)](https://0x7e1.bsidesljubljana.si/yescrypt-large-scale-password-hashing-alexander-peslyak/index.html) , a presentation on Yescrypt by Alexander Peslyak (Solar Designer) to understand this memory hardening technique in depth.

## Password Verification

Say a user named `null` tries to log into a Linux machine, and the Linux OS looks up the entry for this user in the `/etc/shadow` file. It combines the salt for the user named `null` with the plain password typed in by the user and encrypts them using the specified hashing algorithm. If the result matches the hash stored in the `/etc/shadow` file, the user named `null` typed in the correct password. If the result does not match the encrypted hash, the user named `null` typed in the wrong password, and the login attempt fails. these

Let's breakdown the password field of the above given entry from  `/etc/shadow` file.

```bash
$y$j9T$0XNlzjOIPFoBIudkyHauJ.$cHyCfoQcxDm9ktgcYr5rbvrdaBcAwj8vwXNgMql2ffA
```

- `$y$` --> Hashing Algorithm (Yescrypt)
- `j9T` --> Cost Parameters
- `0XNlzjOIPFoBIudkyHauJ.` --> Salt
- `cHyCfoQcxDm9ktgcYr5rbvrdaBcAwj8vwXNgMql2ffA`  --> Hash

You can generate this stored hash by yourself (if you know the user's password) by using following python's code.

```python
import crypt

# Parameters
password = "133817"
salt = "$y$j9T$0XNlzjOIPFoBIudkyHauJ." # Includes both cost parameters and salt

# Generate the Yescrypt hash
hash = crypt.crypt(password, salt)
print("Regenerated Hash:", hash)

```

Here is the my generated hash and you can verify this by comparing this by the original hash from the `/etc/shadow` file's entry.

![[/assets/images/posts/2024-12-25-Demystifying-User-Authentication-in-Linux/hash.png]]

💡 You can specify the hashing algorithm while changing a user's password using `chpasswd` utility from `whois` package. Read man page of `chpasswd` for all available algorithms.

```bash
sudo apt install whois
echo "<username>:<password>" | sudo chpasswd -c SHA512
```

## References

- https://www.bell-labs.com/usr/dmr/www/passwd.ps
- https://0x7e1.bsidesljubljana.si/yescrypt-large-scale-password-hashing-alexander-peslyak/
- https://www.computernetworkingnotes.com/linux-tutorials/etc-shadow-file-in-linux-explained-with-examples.html
- https://www.cyberciti.biz/faq/understanding-etcshadow-file/
- https://mangolassi.it/topic/8057/unix-the-etc-shadow-file-in-depth
- https://unix.stackexchange.com/questions/542989/when-did-unix-stop-storing-passwords-in-clear-text
- https://www.openwall.com/yescrypt/