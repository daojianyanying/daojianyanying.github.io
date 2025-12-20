---
title: ubuntu24安装
date: 2025-08-11 15:42:58
update: 2025-08-12 15:42:58
tags: [linux]
index_img: /img/bg/ubuntu.jpg
excerpt: ubuntu24系统安装和配置
sticky: 100
category: linux
---
# <center>Ubuntu24安装(Vmware)</center>

#### <div style="background-color:cadetblue">一、环境准备</div>

#### <div style="background-color:cadetblue">一、网络配置</div>
/etc/netplan/目录下的YAML文件（如01-netcfg.yaml）
```sh
devops@devops:~$ ls /etc/netplan/*
/etc/netplan/50-cloud-init.yaml
```
```yaml
# 修改内容为如下内容
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: false
      addresses: [192.168.59.3/24]  # 静态IP地址和子网掩码
    #   gateway4: 192.168.1.2          # 网关地址,使用Vmware时，网关末位是2
      routes:
        - to: default
        via: 192.168.1.1    # 网关 使用Vmware时，网关末位是2   
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]  # DNS服务器地址
```
```sh
sudo netplan apply
```

 <p class="note note-primary"> 新增硬盘</p>
 分区工具：gdisk

 格式化类型：xfs
```sh
root@devops:/home/devops# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   40G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   19G  0 lvm  /
sdb                         8:16   0  200G  0 disk 
sr0                        11:0    1  3.1G  0 rom  
root@devops:/home/devops# gdisk /dev/sdb
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries in memory.

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-419430366, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-419430366, default = 419428351) or {+-}size{KMGTP}: 
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully.
root@devops:/home/devops# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   40G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   19G  0 lvm  /
sdb                         8:16   0  200G  0 disk 
└─sdb1                      8:17   0  200G  0 part 
sr0                        11:0    1  3.1G  0 rom  
root@devops:/home/devops# 
```
```sh
root@devops:/home/devops# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=13107072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=1
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=52428288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=25599, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
```
root@devops:/home/devops# mkdir /data
root@devops:/home/devops# mount /dev/sdb1 /data
root@devops:/home/devops# lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   40G  0 disk 
├─sda1                      8:1    0    1M  0 part 
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part 
  └─ubuntu--vg-ubuntu--lv 252:0    0   19G  0 lvm  /
sdb                         8:16   0  200G  0 disk 
└─sdb1                      8:17   0  200G  0 part /data
sr0                        11:0    1  3.1G  0 rom  
```
```
vi /etc/fstab
/dev/sdb1  /data   xfs  defaults 0 0
```