---
title: DevOp cheatsheet
categories:
 - DevOp
tags:
 - DevOp
 - Linux
---

## Meaning of Linux directries

[source link](https://serverfault.com/questions/24523/meaning-of-directories-on-unix-and-unix-like-systems) 

- /bin - Binaries.
- /boot - Files required for booting.
- /dev - Device files.
- /etc - Et cetera. The name is inherited from the earliest Unixes, which is when it became the spot to put config-iles.
- /home - Where home directories are kept.
- /lib - Where code libraries are kept.
- /media - A more modern directory, but where removable media gets mounted.
- /mnt - Where temporary file-systems are mounted.
- /opt - Where optional add-on software is installed. This is discrete from /usr/local/ for reasons I'll get to later.
- /run - Where runtime variable data is kept.
- /sbin - Where super-binaries are stored. These usually only work with root.
- /srv - Stands for "serve". This directory is intended for static files that are served out. /srv/http would be for static websites, /srv/ftp for an FTP server.
- /tmp - Where temporary files may be stored.
- /usr - Another directory inherited from the Unixes of old, it stands for "UNIX System Resources". It does not stand for "user" (see the Debian Wiki). This directory should be sharable between hosts, and can be NFS mounted to multiple hosts safely. It can be mounted read-only safely.
- /var - Another directory inherited from the Unixes of old, it stands for "variable". This is where system data that varies may be stored. Such things as spool and cache directories may be located here. If a program needs to write to the local file-system and isn't serving that data to someone directly, it'll go here.


## Create group and user
```sh
groupadd -g 505 groupname
useradd -g 505 -u 505 username
```

## disable firewall
```sh
systemctl stop firewalld
systemctl disable firewalld
```

## setup local yum source
upload CentOS-7-x86_64-DVD-1611 to /opt

```sh
mkdir /opt/yum-local /opt/yum-temp
mount /opt/CentOS-7_x86_64-DVD-1611 /opt/yum-temp
cp -rf /opt/yum-temp/* /opt/yum-local/
umount /opt/yum-temp
```

and then update yum repo file
```sh
tar czvf yum_repo-backup.tgz /etc/yum.repos.d
rm -rf /etc/yum.repos.d/*
echo "[Local]
name=CentOS-Local
baseurl=file:///opt/yum-local/
gpgcheck=0
enabled=1" >> /etc/yum.repos.d/CentOS-Local.repo
yum clean all
yum makecache
```

# todo: 

## ssh-keygen and add public key to authorized_keys

## setup nfs

## make file-system using gdisk for over 2T disk
