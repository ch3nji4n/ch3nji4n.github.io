---
title: CentOS DevOp cheatsheet
categories:
 - DevOp
tags:
 - DevOp
 - Linux
---

# Meaning of Linux directries

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


# Create group and user
```sh
groupadd -g 505 groupname
useradd -g 505 -u 505 username
```

# disable firewall
```sh
systemctl stop firewalld
systemctl disable firewalld
```

# setup local yum source
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

# ssh-keygen and add public key to authorized_keys
```sh
ssh-keygen -t rsa -P '' 
```
then insert public key to target server ~/.ssh/authorized_keys

# setup nfs
```sh
yum -y install nfs-utils
yum -y install rpcbind
echo"/public 192.168.3.0/24(rw,async,insecure,no_root_squash,no_subtree_check)" >> /etc/exports
systemctl start nfs
systemctl start rpcbind
systemctl enable nfs
systemctl enable rpcbind
showmount -e localhost # show 
    >Export list for localhost:
    >/public 192.168.3.0/24
```

# make file-system using gdisk for over 2T disk
Using fdisk tool can only process at maximum 2T disk, while using parted tool will recognise the file system as 'Microsoft Basic Data' incorrectly. Therefore should use the gdisk tool. 
```sh
yum -y install gdisk.x86_64
gdisk /dev/vdc #进入gidsk对vdc盘进行操作
Command (? for help): ?
    >b       back up GPT data to a file
    >c       change a partition's name
    >d       delete a partition
    >i       show detailed information on a partition
    >l       list known partition types
    >n       add a new partition
    >o       create a new empty GUID partition table (GPT)
    >p       print the partition table
    >q       quit without saving changes
    >r       recovery and transformation options (experts only)
    >s       sort partitions
    >t       change a partition's type code
    >v       verify disk
    >w       write table to disk and exit
    >x       extra functionality (experts only)
    >?       print this menu

Command (? for help): p #输入p打印硬盘信息
    >Disk /dev/vdc: 20971520000 sectors, 9.8 TiB
    >Logical sector size: 512 bytes
    >Disk identifier (GUID): 6FE0C68F-5637-45E4-97EC-B6A6F722BFE4
    >Partition table holds up to 128 entries
    >First usable sector is 34, last usable sector is 20971519966
    >Partitions will be aligned on 2048-sector boundaries
    >Total free space is 20971519933 sectors (9.8 TiB)

Command (? for help): n #输入n新建分区
Partition number (1-128, default 1): #回车默认1
First sector (34-20971519966, default = 2048) or {+-}size{KMGTP}: #回车默认2048
Last sector (2048-20971519966, default = 20971519966) or {+-}size{KMGTP}: #回车默认20971519966
    >Current type is 'Linux filesystem'
    >Hex code or GUID (L to show codes, Enter = 8300):
    >Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/vdc: 20971520000 sectors, 9.8 TiB
    >Logical sector size: 512 bytes
    >Disk identifier (GUID): 6FE0C68F-5637-45E4-97EC-B6A6F722BFE4
    >Partition table holds up to 128 entries
    >First usable sector is 34, last usable sector is 20971519966
    >Partitions will be aligned on 2048-sector boundaries
    >Total free space is 2014 sectors (1007.0 KiB)

    >Number  Start (sector)    End (sector)  Size       Code  Name
    >   1            2048     20971519966   9.8 TiB     8300  Linux filesystem

Command (? for help): w #将分区表写入索引
    >Final checks complete. About to write GPT data. THIS WILL OVERWRITE >EXISTING PARTITIONS!!
Do you want to proceed? (Y/N): Y
    >OK; writing new GUID partition table (GPT) to /dev/vdc.
    >The operation has completed successfully. #会自动退出gdisk

mkfs.ext4 /dev/vdc #快速格式化为ext4

mkdir /public
mount /dev/vdc /public #挂载至/public
echo“/dev/vdc  /public  ext4  defaults  0 0” >> /etc/fstab #添加开机自动挂载/dev/vdc
```
