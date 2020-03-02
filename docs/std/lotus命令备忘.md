---
title: lotus命令备忘
slug: filecoin-lotus-cmd
date: "2019-11-24"
description: lotus命令备忘
categories: 
- filecoin 
tags: 
- filecoin 
---


lotus命令备忘。
<!--more-->

# lotus技术备忘

有mac上面安装

```bash
ping emacsvi.mynetgear.com
telnet emacsvi.mynetgear.com 28080
pxteprl.com/

远程管理访问:
https://182.151.172.116:8443

- 自动产生指定大小文件工具
- 自动监控程序状态
- 自动为地址转账

sudo dpkg-reconfigure tzdata

brew install rust
rustc --version
cargo --version
brew search bazaar
brew install bazaar
brew install jq
brew install pkg-config
cd lotus/
make
sudo make install


dd if=/dev/urandom of=./tmp bs=$((1024*1024)) count=1022
```

go交叉编译：

```bash
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -o test_win_x64.exe test.go
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o test_linux_x64 test.go
CGO_ENABLED = 0 表示设置CGO工具不可用
GOOS 程序构建环境的目标操作系统
GOARCH 表示程序构建环境的目标计算架构
```



编译结果：

```bash
go build -o lotus ./cmd/lotus
go run github.com/GeertJohan/go.rice/rice append --exec lotus -i ./build
rm -f lotus-storage-miner
go build -o lotus-storage-miner ./cmd/lotus-storage-miner
go run github.com/GeertJohan/go.rice/rice append --exec lotus-storage-miner -i ./build
 ~/coding/go/filecoin/lotus   master  sudo make install
Password:
install -C ./lotus /usr/local/bin/lotus
install -C ./lotus-storage-miner /usr/local/bin/lotus-storage-miner
```



```bash
https://www.mls-tech.info/linux/ubuntu-18-mirrors-in-cn/
sudo tar -C /usr/local -xzf go1.13.4.linux-amd64.tar.gz
vim ~/.bashrc
export GOROOT=/usr/local/go
export GOPATH=/home/liwei/go
export PATH=$PATH:$GOPATH:/usr/local/go/bin
source ~/.bashrc
```

docker环境安装：

https://docs.docker.com/install/linux/docker-ce/ubuntu/



```bash
# 安装docker
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
   
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo groupadd docker
sudo usermod -aG docker ${USER}
# 退出终端之后再进入一次

# 安装docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

#下载网址： https://github.com/docker/compose/releases
#下载完成后移动到 /usr/local/bin/下面

sudo chmod +x /usr/local/bin/docker-compose


docker-compose --version


```



shell命令：

```bash
# ubuntu脚本自动输入sudo密码
# 不管用哪种方法sudo后面都有用到参数-S,这个参数是让sudo从标准输入流读取而不是终端设备
# 使用echo和管道命令
echo password | sudo -S service runtime* status
# 使用文本块输入重定向
sudo -S service talend-runtime* status<<EOF
password
EOF
```







- 挖矿环境初始化
- 单机挖矿管理软件
- 压测软件脚本
- 持续跟踪lotus和gofilecoin的最新代码与状态。
- 灌数据功能
- 万总强调的大屏功能展示界面以及后端提供数据支持





```bash
pstree 常用命令
pstrr -ulhs pid

-a：显示每个程序的完整指令，包含路径，参数或是常驻服务的标示；
-c：不使用精简标示法；
-G：使用VT100终端机的列绘图字符；
-h：列出树状图时，特别标明现在执行的程序；
-H<程序识别码pid>：此参数的效果和指定”-h”参数类似，但特别标明指定的程序；
-l：采用长列格式显示树状图；
-n：用程序识别码排序。预设是以程序名称来排序；
-p：显示程序识别码pid；
-u：显示用户名称；
-U：使用UTF-8列绘图字符；
-V：显示版本信息。
```



```mysql
测试环境ubuntu18.04 需安装mysql并设置好密码
sudo apt-get install mysql-server mysql-common mysql-client -y
sudo /etc/init.d/mysql restart
登录
sudo mysql -u root
删除并重建root账号
mysql> DROP USER 'root'@'localhost';
mysql> CREATE USER 'root'@'%' IDENTIFIED BY '123258';
授权远程登录
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
完成
mysql> FLUSH PRIVILEGES;

退出mysql

mysql>q 

现在设置mysql允许远程访问，首先编辑文件/etc/mysql/mysql.conf.d/mysqld.cnf：
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
注释掉bind-address = 127.0.0.1

完成


# https://wangxin1248.github.io/linux/2018/07/ubuntu18.04-install-redis.html
sudo apt install redis-server
sudo vi /etc/redis/redis.conf
改 supervised systemd
sudo service redis restart
sudo systemctl status redis


create database xjgw
  DEFAULT CHARACTER SET utf8
  DEFAULT COLLATE utf8_general_ci;
USE xjgw;
SET NAMES utf8;

DROP TABLE if exists funds;
CREATE TABLE `funds` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `address` varchar(128) NOT NULL,
  `period` int(10) unsigned not NULL default '0',
  `status` int(10) unsigned not NULL default '0',
  `create_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_address` (`address`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

DROP TABLE if exists actors;
CREATE TABLE `actors` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `machine_id` varchar(64) NOT NULL default '',
  `actor_id` varchar(64) NOT NULL,
  `port` int(10) unsigned NOT NULL,
  `sector_size` varchar(64) NOT NULL,
  `owner` varchar(128) NOT NULL,
  `user` varchar(64) NOT NULL,
  `user_home` varchar(64) NOT NULL,
  `storage_path` varchar(1024) NOT NULL,
  `lotus_path` varchar(1024) NOT NULL,
  `env` varchar(1024) NOT NULL default '',
  `status` int(10) unsigned not NULL default '0',
  `create_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_actor` (`actor_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

insert into `funds`(`address`) values ('');

LOTUS_PATH=/home/liwei/.lotus LOTUS_STORAGE_PATH=/home/liwei/.lotusstorage lotus-storage-miner info
lotus-storage-miner init --actor=t01124 --owner=t3wuqppkjzuxnkje3eawqh6twkc7dyikoni7alm2dbg743ofirl3czvdwdnvfvesyfsgghifija4ino3c5oqkq
insert into `actors`(`machine_id`,`actor_id`,`port`,`owner`,`user`,`user_home`,`storage_path`,`lotus_path`,`env`) values("cd-001-001", "t01124", "2237", "t3wuqppkjzuxnkje3eawqh6twkc7dyikoni7alm2dbg743ofirl3czvdwdnvfvesyfsgghifija4ino3c5oqkq", "liwei", "/home/liwei", "/home/liwei/.lotusstorage", "/root/.lotus", 'LOTUS_PATH="/home/liwei/.lotus",LOTUS_STORAGE_PATH="/home/liwei/.lotusstorage"');

DROP TABLE if exists new_actors;
CREATE TABLE `new_actors` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `machine_id` varchar(64) NOT NULL default '',
  `newflag` int(10) unsigned not NULL default '0',
  `actor_id` varchar(64) NOT NULL default '',
  `port` int(10) unsigned NOT NULL,
  `sector_size` varchar(64) NOT NULL,
  `owner` varchar(128) NOT NULL,
  `user` varchar(64) NOT NULL,
  `user_home` varchar(64) NOT NULL,
  `storage_path` varchar(1024) NOT NULL,
  `lotus_path` varchar(1024) NOT NULL,
  `env` varchar(1024) NOT NULL default '',
  `status` int(10) unsigned not NULL default '0',
  `create_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP on update CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_actor_owner` (`actor_id`,`owner`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;


insert into `new_actors`(`machine_id`,`port`,`owner`,`user`,`user_home`,`storage_path`,`lotus_path`,`sector_size`) values("cd-001-002", "2239", "t3wxvw2pw4m3delds2ku4komu5apxkjbdgc7zs22zcnga4pjgosdpr4ngqcxftxjiktk63a72lhgtspbazdvga", "dada", "/home/dada", "/home/dada/.lotusstorage", "/root/.lotus", '16777216');


```



一定要改多用户的权限。chmod 777 -R /root; chmod 777 -R /root/.lotus

另外需要改代码：node/repo/fsrepo.go



```bash
https://www.cnblogs.com/yasmi/articles/4835644.html
sudo apt-get install lvm2

sudo fdisk -l
Device           Start        End   Sectors   Size Type
/dev/nvme0n1p1    2048    1050623   1048576   512M EFI System
/dev/nvme0n1p2 1050624 1000214527 999163904 476.4G Linux filesystem


Disk /dev/sda: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes


Disk /dev/sdb: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes


Disk /dev/sdc: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes


lotus@lotus-xjgw:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.31.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

The old ext4 signature will be removed by a write command.

Device does not contain a recognized partition table.
The size of this disk is 3.7 TiB (4000787030016 bytes). DOS partition table format cannot be used on drives for volumes larger than 2199023255040 bytes for 512-byte sectors. Use GUID partition table format (GPT).

Created a new DOS disklabel with disk identifier 0x154dac12.

Command (m for help): m

Help:

  DOS (MBR)
   a   toggle a bootable flag
   b   edit nested BSD disklabel
   c   toggle the dos compatibility flag

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   u   change display/entry units
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty DOS partition table
   s   create a new empty Sun partition table


Command (m for help): d
No partition is defined yet!
Could not delete partition 94732900831001

Command (m for help): g

Created a new GPT disklabel (GUID: 3CE77D14-33F3-4E4D-9BE3-0F22E024E85F).
The old ext4 signature will be removed by a write command.

Command (m for help): n
Partition number (1-128, default 1):
First sector (2048-7814037134, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-7814037134, default 7814037134):

Created a new partition 1 of type 'Linux filesystem' and of size 3.7 TiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

Command (m for help): p
Disk /dev/sdc: 3.7 TiB, 4000787030016 bytes, 7814037168 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 0977529D-7574-A843-AF53-CA3C582CAC00

Device     Start        End    Sectors  Size Type
/dev/sdc1   2048 7814037134 7814035087  3.7T Linux filesystem

Command (m for help): t
Selected partition 1
Partition type (type L to list all types): L
  1 EFI System                     C12A7328-F81F-11D2-BA4B-00A0C93EC93B
  2 MBR partition scheme           024DEE41-33E7-11D3-9D69-0008C781F39F
  3 Intel Fast Flash               D3BFE2DE-3DAF-11DF-BA40-E3A556D89593
  4 BIOS boot                      21686148-6449-6E6F-744E-656564454649
  5 Sony boot partition            F4019732-066E-4E12-8273-346C5641494F
  6 Lenovo boot partition          BFBFAFE7-A34F-448A-9A5B-6213EB736C22
  7 PowerPC PReP boot              9E1A2D38-C612-4316-AA26-8B49521E5A8B
  8 ONIE boot                      7412F7D5-A156-4B13-81DC-867174929325
  9 ONIE config                    D4E6E2CD-4469-46F3-B5CB-1BFF57AFC149
 10 Microsoft reserved             E3C9E316-0B5C-4DB8-817D-F92DF00215AE
 11 Microsoft basic data           EBD0A0A2-B9E5-4433-87C0-68B6B72699C7
 12 Microsoft LDM metadata         5808C8AA-7E8F-42E0-85D2-E1E90434CFB3
 13 Microsoft LDM data             AF9B60A0-1431-4F62-BC68-3311714A69AD
 14 Windows recovery environment   DE94BBA4-06D1-4D40-A16A-BFD50179D6AC
 15 IBM General Parallel Fs        37AFFC90-EF7D-4E96-91C3-2D7AE055B174
 16 Microsoft Storage Spaces       E75CAF8F-F680-4CEE-AFA3-B001E56EFC2D
 17 HP-UX data                     75894C1E-3AEB-11D3-B7C1-7B03A0000000
 18 HP-UX service                  E2A1E728-32E3-11D6-A682-7B03A0000000
 19 Linux swap                     0657FD6D-A4AB-43C4-84E5-0933C84B4F4F
 20 Linux filesystem               0FC63DAF-8483-4772-8E79-3D69D8477DE4
 21 Linux server data              3B8F8425-20E0-4F3B-907F-1A25A76F98E8
 22 Linux root (x86)               44479540-F297-41B2-9AF7-D131D5F0458A
 23 Linux root (ARM)               69DAD710-2CE4-4E3C-B16C-21A1D49ABED3
 24 Linux root (x86-64)            4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709
 25 Linux root (ARM-64)            B921B045-1DF0-41C3-AF44-4C6F280D3FAE
 26 Linux root  (IA-64)             993D8D3D-F80E-4225-855A-9DAF8ED7EA97
 27 Linux reserved                 8DA63339-0007-60C0-C436-083AC8230908
 28 Linux home                     933AC7E1-2EB4-4F13-B844-0E14E2AEF915
 29 Linux RAID                     A19D880F-05FC-4D3B-A006-743F0F84911E
 30 Linux extended boot            BC13C2FF-59E6-4262-A352-B275FD6F7172
 31 Linux LVM                      E6D6D379-F507-44C2-A23C-238F2A3DF928
 32 FreeBSD data                   516E7CB4-6ECF-11D6-8FF8-00022D09712B
 33 FreeBSD boot                   83BD6B9D-7F41-11DC-BE0B-001560B84F0F
 34 FreeBSD swap                   516E7CB5-6ECF-11D6-8FF8-00022D09712B
 35 FreeBSD UFS                    516E7CB6-6ECF-11D6-8FF8-00022D09712B
 36 FreeBSD ZFS                    516E7CBA-6ECF-11D6-8FF8-00022D09712B
 37 FreeBSD Vinum                  516E7CB8-6ECF-11D6-8FF8-00022D09712B
 38 Apple HFS/HFS+                 48465300-0000-11AA-AA11-00306543ECAC
 39 Apple UFS                      55465300-0000-11AA-AA11-00306543ECAC
 40 Apple RAID                     52414944-0000-11AA-AA11-00306543ECAC
 41 Apple RAID offline             52414944-5F4F-11AA-AA11-00306543ECAC
 42 Apple boot                     426F6F74-0000-11AA-AA11-00306543ECAC
 43 Apple label                    4C616265-6C00-11AA-AA11-00306543ECAC
 44 Apple TV recovery              5265636F-7665-11AA-AA11-00306543ECAC
 45 Apple Core storage             53746F72-6167-11AA-AA11-00306543ECAC
 46 Solaris boot                   6A82CB45-1DD2-11B2-99A6-080020736631
 47 Solaris root                   6A85CF4D-1DD2-11B2-99A6-080020736631
 48 Solaris /usr & Apple ZFS       6A898CC3-1DD2-11B2-99A6-080020736631
 49 Solaris swap                   6A87C46F-1DD2-11B2-99A6-080020736631
 50 Solaris backup                 6A8B642B-1DD2-11B2-99A6-080020736631
 51 Solaris /var                   6A8EF2E9-1DD2-11B2-99A6-080020736631
 52 Solaris /home                  6A90BA39-1DD2-11B2-99A6-080020736631
 53 Solaris alternate sector       6A9283A5-1DD2-11B2-99A6-080020736631
 54 Solaris reserved 1             6A945A3B-1DD2-11B2-99A6-080020736631
 55 Solaris reserved 2             6A9630D1-1DD2-11B2-99A6-080020736631
 56 Solaris reserved 3             6A980767-1DD2-11B2-99A6-080020736631
 57 Solaris reserved 4             6A96237F-1DD2-11B2-99A6-080020736631
 58 Solaris reserved 5             6A8D2AC7-1DD2-11B2-99A6-080020736631
 59 NetBSD swap                    49F48D32-B10E-11DC-B99B-0019D1879648
 60 NetBSD FFS                     49F48D5A-B10E-11DC-B99B-0019D1879648
 61 NetBSD LFS                     49F48D82-B10E-11DC-B99B-0019D1879648
 62 NetBSD concatenated            2DB519C4-B10E-11DC-B99B-0019D1879648
 63 NetBSD encrypted               2DB519EC-B10E-11DC-B99B-0019D1879648
 64 NetBSD RAID                    49F48DAA-B10E-11DC-B99B-0019D1879648
 65 ChromeOS kernel                FE3A2A5D-4F32-41A7-B725-ACCC3285A309
 66 ChromeOS root fs               3CB8E202-3B7E-47DD-8A3C-7FF2A13CFCEC
 67 ChromeOS reserved              2E0A753D-9E48-43B0-8337-B15192CB1B5E
 68 MidnightBSD data               85D5E45A-237C-11E1-B4B3-E89A8F7FC3A7
 69 MidnightBSD boot               85D5E45E-237C-11E1-B4B3-E89A8F7FC3A7
 70 MidnightBSD swap               85D5E45B-237C-11E1-B4B3-E89A8F7FC3A7
 71 MidnightBSD UFS                0394EF8B-237E-11E1-B4B3-E89A8F7FC3A7
 72 MidnightBSD ZFS                85D5E45D-237C-11E1-B4B3-E89A8F7FC3A7
 73 MidnightBSD Vinum              85D5E45C-237C-11E1-B4B3-E89A8F7FC3A7
 74 Ceph Journal                   45B0969E-9B03-4F30-B4C6-B4B80CEFF106
 75 Ceph Encrypted Journal         45B0969E-9B03-4F30-B4C6-5EC00CEFF106
 76 Ceph OSD                       4FBD7E29-9D25-41B8-AFD0-062C0CEFF05D
 77 Ceph crypt OSD                 4FBD7E29-9D25-41B8-AFD0-5EC00CEFF05D
 78 Ceph disk in creation          89C57F98-2FE5-4DC0-89C1-F3AD0CEFF2BE
 79 Ceph crypt disk in creation    89C57F98-2FE5-4DC0-89C1-5EC00CEFF2BE
 80 OpenBSD data                   824CC7A0-36A8-11E3-890A-952519AD3F61
 81 QNX6 file system               CEF5A9AD-73BC-4601-89F3-CDEEEEE321A1
 82 Plan 9 partition               C91818F9-8025-47AF-89D2-F030D7000C2C

Partition type (type L to list all types): 31
Changed type of partition 'Linux filesystem' to 'Linux LVM'.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

lotus@lotus-xjgw:~$ sudo pvcreate /dev/sda1 /dev/sdb1 /dev/sdc1
  Physical volume "/dev/sda1" successfully created.
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdc1" successfully created.
lotus@lotus-xjgw:~$ sudo pvdisplay
  "/dev/sda1" is a new physical volume of "<3.64 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sda1
  VG Name
  PV Size               <3.64 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               0xok6c-cXB0-nT06-j3eK-wtWp-gGsj-mCtYUt

  "/dev/sdc1" is a new physical volume of "<3.64 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdc1
  VG Name
  PV Size               <3.64 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               3eZ1PF-eJ8F-tjxG-SQzA-isgQ-zBsr-feF6My

  "/dev/sdb1" is a new physical volume of "<3.64 TiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb1
  VG Name
  PV Size               <3.64 TiB
  Allocatable           NO
  PE Size               0
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               MJDpCD-V2LO-Bnva-YRhg-OSbd-cyt1-HU7rZb
  
  lotus@lotus-xjgw:~$ sudo vgcreate extspace /dev/sda1 /dev/sdb1 /dev/sdc1
  Volume group "extspace" successfully created
  
  lotus@lotus-xjgw:~$ sudo lvcreate --name home_ext -l100%FREE extspace
  Logical volume "home_ext" created.
lotus@lotus-xjgw:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/extspace/home_ext
  LV Name                home_ext
  VG Name                extspace
  LV UUID                dPA5ce-5TEw-afL0-A0Qq-UqnQ-KW03-t02DXD
  LV Write Access        read/write
  LV Creation host, time lotus-xjgw, 2019-12-08 18:18:40 +0800
  LV Status              available
  # open                 0
  LV Size                <10.92 TiB
  Current LE             2861583
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

lotus@lotus-xjgw:~$ sudo mkfs.ext4 /dev/extspace/home_ext
mke2fs 1.44.1 (24-Mar-2018)
Creating filesystem with 2930260992 4k blocks and 366284800 inodes
Filesystem UUID: c7f7e11f-7a8f-410c-8229-9fcacc3e4786
Superblock backups stored on blocks:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848, 512000000, 550731776, 644972544, 1934917632,
	2560000000

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done

lotus@lotus-xjgw:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/extspace/home_ext
  LV Name                home_ext
  VG Name                extspace
  LV UUID                dPA5ce-5TEw-afL0-A0Qq-UqnQ-KW03-t02DXD
  LV Write Access        read/write
  LV Creation host, time lotus-xjgw, 2019-12-08 18:18:40 +0800
  LV Status              available
  # open                 0
  LV Size                <10.92 TiB
  Current LE             2861583
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
  
  sudo mkdir /data
  sudo mount /dev/extspace/home_ext /data
  sudo vim /etc/fstab
  /dev/extspace/home_ext /data ext4 rw,noatime 0 0

```



https://www.cnblogs.com/yasmi/articles/4835644.html



https://wangxin1248.github.io/linux/2018/07/ubuntu18.04-install-redis.html ubuntu



```bash
master:
lotus-storage-miner init --actor=t01957 --owner=t3uqtmmlydiaf6xub4elio3urf2fp5uubp5gkdjvszvd5p7aztllkv7flx4doroc3ps3c4vwm5yelt5tyjbbva
slave:
lotus-storage-miner --repo=/home/miner/.lotus --storagerepo=/data/slave init --actor=t01959 --owner=t3thjuyfzhbh2ypk2rq2t7i6fa3p6jok42ddbdyvuxfif6ocwpbccb3llm47eun4tf2sf7qvgjeoemyjx4o4ba
备用：
lotus-storage-miner init --actor=t01961 --owner=t3r22h4ao2ynp6qrclw22egpeuvg76gl7bgwzmxh3p4caudb6wfahnktxdkaqlp7aqgd5b6sljsfcq64skh2ua

command=lotus-storage-miner --repo=/home/miner/.lotus --storagerepo=/data/slave run --api 2239
environment=BELLMAN_CUSTOM_GPU="GeForce RTX 2080 Ti:4352"
lotus-storage-miner init --actor=t02085 --owner=t3r6532uydzhcw6nrfkbm2t3noakdqbrny7t6q6bokivgvecxejdnw5tv6a6p5mjtkwwq3m3ei3oeywojdvbmq

```



```bash
t3xcgiw5mbi7tz3mmyi6kc5y4ej74axzrpymeqtol3k7oeo7htpu5zgvvte6hkfpk6nexbi2couqlsd63udtyq
7b2254797065223a22626c73222c22507269766174654b6579223a22762b424f6c37435550394463725042362b7557745a38413945626e525a44476969476d58366746706f696f3d227d
lotus-storage-miner init --actor=t02225 --owner=t3xcgiw5mbi7tz3mmyi6kc5y4ej74axzrpymeqtol3k7oeo7htpu5zgvvte6hkfpk6nexbi2couqlsd63udtyq

t3wbopxkvqt4iv36hbmolmwow4noz6cglptojzogd6wn5ifzvsbvqctenduc7kn6brfx5bkh4el2g7ghg65rva
lotus-storage-miner init --actor=t02230 --owner=t3wbopxkvqt4iv36hbmolmwow4noz6cglptojzogd6wn5ifzvsbvqctenduc7kn6brfx5bkh4el2g7ghg65rva
7b2254797065223a22626c73222c22507269766174654b6579223a225766385a6c4342536667554c4e2b416c596b2b37582f2f6b56797974752b672f73424d4a3764762f5643383d227d

32g:
t3qu2l6wmeqzodi6am372c6gtjv6s5kaherbzpj6j4wj4u7fpnoj6oof4ow6zwyffkagxpjhqlmnp3ysif46qq

lotus-storage-miner init --actor=t02235 --owner=t3qu2l6wmeqzodi6am372c6gtjv6s5kaherbzpj6j4wj4u7fpnoj6oof4ow6zwyffkagxpjhqlmnp3ysif46qq
7b2254797065223a22626c73222c22507269766174654b6579223a22516a2f44786f72714f515843476e37796847744f7169426e35463868364a3054333473707a3274324946593d227d
```





https://blog.csdn.net/xin_yu_xin/article/details/46416101

sudo iptables -t nat -A PREROUTING -d 10.0.48.206 -p tcp --dport 2345 -j DNAT --to-destination 10.0.48.80:2345

lotus-storage-miner init --actor=t05991 --owner=t3vmzwau5pfrqwbutofcjrzug7ztdoy4j7l7d4aoqphdw352ybgcckercgkszoobhfjur6zarz2oaa4bvf4wdq



bench:

https://github.com/filecoin-project/lotus/issues/839#



分布式id测试数据：

```bash
# 申请矿工 在b机器上
./lotus-storage-miner   init --sector-size=16777216
# 另外一台机器ini
./lotus-storage-miner init --actor=t01012

# 主节点 nfs查询节点
./lotus-storage-miner run -o=true -t=t01001
# seal节点
./lotus-storage-miner init --actor=t01001
./lotus-storage-miner run -o=true -s=true -t=t01001

# 3970自定义目录
# 如果要压数据，记得修改config.toml
#  WorkerCount = 5
./lotus-storage-miner --storagerepo=/data/r3970 pledge-sector
./lotus-storage-miner --storagerepo=/data/r3970 init --actor=t01001
./lotus-storage-miner --storagerepo=/data/r3970 run -o=true -s=true -t=t01001


# nfs server
sudo apt install nfs-kernel-server -y
mkdir .lotusstorage
sudo vi /etc/exports
/home/xjgwc/.lotusstorage 10.0.48.0/24(rw,sync,no_subtree_check,no_root_squash)
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
# nfs client
cd /mnt/nfs/
mkdir miner-xjgw-lotusstorage
sudo mount -o soft,soft,timeo=30,retry=2  10.0.48.12:/home/miner/.lotusstorage /mnt/nfs/miner-xjgw-lotusstorage


# 自定义目录：
./lotus --repo=/home/xjgwa/seal/lotus daemon --api=9123
./lotus --repo=/home/xjgwa/seal/lotus wallet set-default t3rjx2e2zjiyqk5a3go4ag76xjum2va5yq5mcdi5m2uodterxhkwdhbk4i44jqbu2ojirida4q6ybmlowtgqiq
./lotus --repo=/home/xjgwa/seal/lotus sync wait

# 自定义矿工
./lotus-storage-miner --repo=/home/xjgwa/seal/lotus --storagerepo=/home/xjgwa/seal/data init --actor=t01003
/home/xjgwa/seal/bin/lotus-storage-miner --repo=/home/xjgwa/seal/lotus --storagerepo=/home/xjgwa/seal/data run -o=true -s=true -t=t01003

./lotus-storage-miner init --actor=t01073
```



lsi做raid

```bash
# https://blog.csdn.net/oaa608868/article/details/53523960
# https://www.aikaiyuan.com/11557.html
# 末尾添加
sudo vim /etc/apt/sources.list

deb http://hwraid.le-vert.net/ubuntu precise main

# 更新源
sudo apt-get update  
# 注意：会提示一些警告可忽略，但如果提示 GPG 错误，需要执行如下命令添加证书：
sudo wget -O - http://hwraid.le-vert.net/debian/hwraid.le-vert.net.gpg.key | sudo apt-key add -
sudo apt-get update
# 安装MegaCLI
sudo apt-get install -y megacli megactl megaraid-status
# 显示Raid卡型号，Raid设置，Disk相关信息
sudo megacli -cfgdsply -aALL

# 查看RAID状态
sudo megacli -cfgdsply -aALL
# 注意：State为Optimal，表示正常。更换损坏的硬盘后，机器会自动同步raid数据，此时状态为Degraded降级状态，不需要别的操作，只需等待几个小时待完全同步数据，恢复raid状态。
sudo megacli -cfgdsply -aALL |grep "State"
# 查看物理磁盘信息
sudo megacli -PDList -aALL

# 检测磁盘 ID 注意, 该ID 值用于标注磁盘 Enclosure Device ID: 252
sudo megacli -PDlist -aALL | grep "ID"  | uniq
# 查看当前raid数量
sudo megacli -cfgdsply -aALL |grep "Number of DISK GROUPS:"
# 查看Raid卡信息
sudo megacli -cfgdsply –aALL  | more
# 其他物理信息
sudo megacli -PDList -aALL | more
# 当前raid信息
sudo megacli -LDInfo -LALL –aAll
# raid 控制器个数
sudo megacli  -adpCount
# raid 控制器时间
sudo megacli -AdpGetTime –aALL
# https://blog.51cto.com/hmtk520/2140657
# 创建raid5 创建一个raid5阵列，由物理盘1,2,3,4,5构成，该阵列的热备盘是物理盘6
xj@xjgw3970:~$ sudo megacli -CfgLdAdd -r5 [252:0,252:1,252:2,252:3,252:4] WB Direct -Hsp[252:5] –a0

Adapter 0: Created VD 0
Adapter: 0: Set Physical Drive at EnclId-252 SlotId-5 as Hot Spare Success.

Adapter 0: Configured the Adapter!!

Exit Code: 0x00

# 如果不指定热备
sudo megacli -CfgLdAdd -r5 [252:0,252:1,252:2,252:3,252:4] WB Direct –a0

# 创建一个raid10阵列，由物理盘2,3和4,5分别做raid1，在将两组raid1做raid0
sudo megacli –CfgSpanAdd –r10 –Array0[1:2,1:3] –Array1[1:4,1:5] WB Direct -a0


# 创建分区，格式化，mount
sudo fdisk /dev/sda
g n t 31 w p
sudo mkfs.ext4 /dev/sda1
sudo mount /dev/sda1 /r5
# 写fstab
/dev/sda1 /r5 ext4 rw,noatime 0 0

# 删除radi5
sudo megacli -CfgLdDel -L0 -a0

/dev/sda1 /data/filecoin ext4 rw,noatime 0 0

sudo systemctl set-default multi-user.target
sudo reboot
sudo systemctl set-default multi-user.target
sudo reboot

run("echo '{0}\t{1}\txfs\tdefaults\t0\t0' | sudo tee -a /etc/fstab".format(device, mount))

sudo echo "deb http://hwraid.le-vert.net/ubuntu precise main" | sudo tee -a /etc/apt/sources.list
sudo apt-get update
sudo wget -O - http://hwraid.le-vert.net/debian/hwraid.le-vert.net.gpg.key | sudo apt-key add -
sudo apt-get update
sudo apt-get install -y megacli megactl megaraid-status
sudo megacli -PDlist -aALL | grep "ID"  | uniq
sudo megacli -CfgLdAdd -r5 [26:1,26:2,26:3,26:4,26:5,26:6,26:7,26:8] WB Direct –a0
sudo fdisk /dev/sda
sudo mkfs.ext4 /dev/sda1

sudo mkdir -p /data/filecoin
sudo mount /dev/sda1 /data/filecoin
sudo echo "/dev/sda1 /data/filecoin ext4 rw,noatime 0 0" | sudo tee -a /etc/fstab
sudo chown xjgw:root /data -Rf
sudo systemctl set-default multi-user.target
sudo vi /etc/network/interfaces
cat /etc/network/interfaces
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback
sudo vi /etc/systemd/resolved.conf


sudo mount -t cifs -o user=xjgw,password='xjgw!234' //10.0.20.4/share /mnt/nfs/cd-xx-004-filecoin
sudo chown xjgw:root /data -Rf
sudo chown xjgw:root /data/filecoin -Rf
./sp.sh
sudo cp lotus* /usr/local/bin/
./lotus-storage-miner --repo=/home/xjgw/sealwork/lotus --storagerepo=/data/filecoin init --actor=t02112

./dminer ctl -s http://127.0.0.1:9028 reload
iostat -d sda -m 2
~/sealwork/bin/lotus-storage-miner  --storagerepo=/data/filecoin   --repo=~/sealwork/lotus info

export {http,https}_proxy='http://www.shihuajin.pro:11087'
export {http,https}_proxy='http://lijie.mynetgear.com:11087'

systemctl stop [servicename]
systemctl disable [servicename]
rm /etc/systemd/system/[servicename]
rm /etc/systemd/system/[servicename] symlinks that might be related
systemctl daemon-reload
systemctl reset-failed
```



```bash
# ansible相关
sudo apt install software-properties-common
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
# 使用pip安装
sudo apt install python
sudo apt install python-pip
pip install ansible


# 阶段二 创建一个测试用的环境用来测试
# 下载我的sealwork文件 必须下载老版本，包含了lotus目录的版本，才能在本地不启动lotus的前提下，连接到服务器上而去

# 初始化矿工
/home/lotus/sealwork/bin/lotus-storage-miner --repo=/home/lotus/sealwork/lotus --storagerepo=/home/lotus/sealwork/lwdd init --sector-size=34359738368 --worker=t3tfxx565befknh2gfariuwkitwfbaru66f3buj6aj66h44ntik6sjezr55fb63wcrnryqivyeeevgcg7vj5sq --owner=t3tfxx565befknh2gfariuwkitwfbaru66f3buj6aj66h44ntik6sjezr55fb63wcrnryqivyeeevgcg7vj5sq

t021162

# 运行密封节点
/home/lotus/sealwork/bin/lotus-storage-miner --repo=/home/lotus/sealwork/lotus --storagerepo=/home/lotus/sealwork/lwdd run -o=true -s=true -t=t021162

# garbage数据
/home/lotus/sealwork/bin/lotus-storage-miner --repo=/home/lotus/sealwork/lotus --storagerepo=/home/lotus/sealwork/lwdd pledge-sector

# 查看硬盘读写性能
iostat --human -x 2

# 会完etcd里面写数据
success save to etcd [/xjgw/kvstore/t021162/commP1]=(0x9afbe3cec92dee670c7c39818b289dd870780c415e5ef9741bf03330a2e33526)


# 停一台用来测试
/home/xjgw/sealwork/bin/lotus-storage-miner --storagerepo=/home/xjgw/sealwork/lwdd init --sector-size=34359738368 --worker=t3tfxx565befknh2gfariuwkitwfbaru66f3buj6aj66h44ntik6sjezr55fb63wcrnryqivyeeevgcg7vj5sq --owner=t3tfxx565befknh2gfariuwkitwfbaru66f3buj6aj66h44ntik6sjezr55fb63wcrnryqivyeeevgcg7vj5sq

t021167

rsync -e "ssh -p44022" -avpgolr --progress xjgw@10.0.20.2:/home/xjgw/sealwork .
```

















