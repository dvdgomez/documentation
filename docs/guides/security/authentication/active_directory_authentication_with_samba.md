---
title: Active Directory Authentication with Samba
author: Neel Chauhan
contributors: Steven Spencer, Ganna Zhyrnova
tested_with: 9.4
---

## Prerequisites

- Some understanding of Active Directory
- Some understanding of LDAP

## Introduction

In most enterprises, Microsoft's Active Directory (AD) is the default authentication system for Windows systems and for external, LDAP-connected services. It allows you to configure users and groups, access control, permissions, auto-mounting, and more.

While connecting Linux to an AD cluster cannot support _all_ of the features mentioned, it can handle users, groups, and access control. It is possible (through some configuration tweaks on the Linux side and some advanced options on the AD side) to distribute SSH keys using AD.

The default way of using Active Directory on Rocky Linux is using SSSD, but Samba is a more full-featured alternative. For instance, file sharing can be done with Samba but not SSSD. This guide, however, will cover configuring authentication against Active Directory using Samba and will not include any extra configuration on the Windows side.

## Discovering and joining AD using Samba

!!! Note

    The domain name `ad.company.local` throughout this guide will represent the Active Directory domain. To follow this guide, replace it with your AD domain's name.

The first step to joining a Linux system into AD is to discover your AD cluster, to ensure the network configuration is correct on both sides.

### Preparation

- Ensure the following ports are open to your Linux host on your domain controller:

  | Service  | Port(s)           | Notes                                                       |
  |----------|-------------------|-------------------------------------------------------------|
  | DNS      | 53 (TCP+UDP)      |                                                             |
  | Kerberos | 88, 464 (TCP+UDP) | Used by `kadmin` for setting & updating passwords           |
  | LDAP     | 389 (TCP+UDP)     |                                                             |
  | LDAP-GC  | 3268 (TCP)        | LDAP Global Catalog - allows you to source user IDs from AD |

- Ensure you have configured your AD domain controller as a DNS server on your Rocky Linux host:

  **With NetworkManager:**

  ```sh
  # where your primary NetworkManager connection is 'System eth0' and your AD
  # server is accessible on the IP address 10.0.0.2.
  [root@host ~]$ nmcli con mod 'System eth0' ipv4.dns 10.0.0.2
  ```
  
- Ensure that the time on both sides (AD host and Linux system) is synchronized (see chronyd)

  **To check the time on Rocky Linux:**

  ```sh
  [user@host ~]$ date
  Wed 22 Sep 17:11:35 BST 2021
  ```

- Install the required packages for AD connection on the Linux side:

  ```sh
  [user@host ~]$ sudo dnf install samba samba-winbind samba-client
  ```

### Discovery

You should now be able to successfully discover your AD server(s) from your Linux host.

```sh
[user@host ~]$ realm discover ad.company.local
ad.company.local
  type: kerberos
  realm-name: AD.COMPANY.LOCAL
  domain-name: ad.company.local
  configured: no
  server-software: active-directory
  client-software: sssd
  required-package: oddjob
  required-package: oddjob-mkhomedir
  required-package: sssd
  required-package: adcli
  required-package: samba-common
```

The relevant SRV records stored in your Active Directory DNS service will allow discovery.

### Joining

Once you have successfully discovered your Active Directory installation from the Linux host, you should be able to use `realmd` to join the domain, which will orchestrate the configuration of Samba using `adcli` and some other such tools.

```sh
[user@host ~]$ sudo realm join -v --membership-software=samba --client-software=winbind ad.company.local
```

You will be prompted to enter your domain's administrator password so input it.

If this process complains about encryption with `KDC has no support for encryption type`, try updating the global crypto policy to allow older encryption algorithms:

```sh
[user@host ~]$ sudo update-crypto-policies --set DEFAULT:AD-SUPPORT
```

If this process succeeds, you should now be able to pull `passwd` information for an Active Directory user.

```sh
[user@host ~]$ sudo getent passwd administrator@ad.company.local
AD\administrator:*:1450400500:1450400513:Administrator:/home/administrator@ad.company.local:/bin/bash
```

!!! Note

    `getent` get entries from Name Service Switch libraries (NSS). It means that, contrary to `passwd` or `dig` for example, it will query different databases, including `/etc/hosts` for `getent hosts` or from `samba` in the `getent passwd` case.

`realm` provides some interesting options that you can use:

| Option                                                     | Observation                              |
|------------------------------------------------------------|------------------------------------------|
| --computer-ou='OU=LINUX,OU=SERVERS,dc=ad,dc=company.local' | The OU where to store the server account |
| --os-name='rocky'                                          | Specify the OS name stored in the AD     |
| --os-version='8'                                           | Specify the OS version stored in the AD  |
| -U admin_username                                          | Specify an admin account                 |

### Attempting to authenticate

Now your users should be able to authenticate to your Linux host against Active Directory.

**On Windows 10:** (which provides its own copy of OpenSSH)

```dos
C:\Users\John.Doe> ssh -l john.doe@ad.company.local linux.host
Password for john.doe@ad.company.local:

Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Sep 15 17:37:03 2021 from 10.0.10.241
[john.doe@ad.company.local@host ~]$
```

If this succeeds, you have successfully configured Linux to use Active Directory as an authentication source.

### Eliminating the domain name in usernames

In a completely default setup, you will need to log in with your AD account by specifying the domain in your username (e.g., `john.doe@ad.company.local`). If this is not the desired behavior and you instead want to be able to omit the default domain name at authentication time you can configure Samba to default to a specific domain.

This is a relatively straightforward process, requiring a configuration tweak in your `smb.conf` configuration file.

```sh
[user@host ~]$ sudo vi /etc/samba/smb.conf
[global]
...
winbind use default domain = yes
```

By adding the `winbind use default domain`, you are instructing Samba to infer that the user is trying to authenticate as a user from the `ad.company.local` domain. This allows you to authenticate as something like `john.doe` instead of `john.doe@ad.company.local`.

To make this configuration change take effect, you must restart the `smb` and `winbind` services with `systemctl`.

```sh
[user@host ~]$ sudo systemctl restart smb winbind
```

In the same way, if you do not want your home directories suffixed with the domain name, you can add those options into your configuration file `/etc/samba/smb.conf`:

```bash
[global]
template homedir = /home/%U
```

Do not forget to restart the `smb` and `winbind` services.
