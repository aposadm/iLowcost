Using Active Directory to Authenticate Linux Users

```
sudo apt -y update
```

Hostname and DNS Setting

```
hostnamectl 
```
```
root@doe-puppet-001:~# hostnamectl
 Static hostname: doe-puppet-001
       Icon name: computer-vm
         Chassis: vm
      Machine ID: 6a0185b890d64e9c9356ea833908bfcf
         Boot ID: 61ba7e60ffec481ea59bf152581d9982
  Virtualization: vmware
Operating System: Ubuntu 22.04.5 LTS
          Kernel: Linux 5.15.0-124-generic
    Architecture: x86-64
 Hardware Vendor: VMware, Inc.
  Hardware Model: VMware Virtual Platform
root@doe-puppet-001:~#
```

Subsequently, let’s configure our DNS setting. We’ll edit the resolv.conf and append our Active Directory’s IP as the second DNS address:

```
vim /etc/resolv.conf
```

```
nameserver 127.0.0.53
nameserver 192.168.x.x
options edns0 trust-ad
search .
```

Installing the Required Packages

```
sudo apt update
```
```
sudo apt -y install realmd libnss-sss libpam-sss sssd sssd-tools adcli samba-common-bin oddjob oddjob-mkhomedir packagekit
```

sudo apt -y install realmd libnss-sss libpam-sss sssd sssd-tools adcli samba-common-bin oddjob oddjob-mkhomedir packagekit

Discover the Realm

```
sudo realm discover futuresky.local
```

Join Realm

```
sudo realm join --user=aekkalak.phe futuresky.local
```


Let’s view the /etc/sssd/sssd.conf file:

```
cat /etc/sssd/sssd.conf
```
```
[sssd]
domains = futuresky.local
config_file_version = 2
services = nss, pam

[domain/futuresky.local]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = FUTURESKY.LOCAL
realmd_tags = manages-system joined-with-adcli
id_provider = ad
fallback_homedir = /home/%u@%d
ad_domain = futuresky.local
use_fully_qualified_names = True
ldap_id_mapping = True
access_provider = simple
sudo_provider = ad
```

By default, the realm command has already configured this file. It added the pam and nss modules and started the necessary services.


```
pam-auth-update --enable mkhomedir
```


To assign sudoers to groups


```
chmod 640 /etc/sudoers
```
```
vim /etc/sudoers
```
```
%SudoAdmin@futuresky.local ALL=(ALL) ALL
```


TEST command

```
getent passwd aekkalak.phe@futuresky.local
```

```
sudo login
```

Steps to Unjoin (Leave) an Ubuntu System from AD Domain:

```
realm list
```

```
realm leave futuresky.local
```

If you used sssd to authenticate, you may want to remove the configuration:

```
sudo rm /etc/sssd/sssd.conf
```
```
sudo systemctl restart sssd
```
Remove Kerberos Configuration:
```
sudo rm /etc/krb5.conf
```