#                             在两大流行平台部署各种类型服务器

------

#                 Linux部署各种类型服务器

##                                                                                                        （适用于CentOS8 | RHEL8）

------

## RHEL8|Centos8部署服务器

### RAID与LVM技术

章节简述：

在学习了第6章讲解的硬盘设备分区、格式化、挂载等知识后，本章将深入讲解各个常用RAID（Redundant Array of Independent Disks，独立冗余磁盘阵列）技术方案的特性，并通过实际部署RAID 10、RAID 5+备份盘等方案来更直观地查看RAID的强大效果，以便进一步满足生产环境对硬盘设备的IO读写速度和数据冗余备份机制的需求。同时，考虑到用户可能会动态调整存储资源，本章还将介绍LVM（Logical Volume Manager，逻辑卷管理器）的部署、扩容、缩小、快照以及卸载删除的相关知识。相信读者在学完本章内容后，便可以在企业级生产环境中灵活运用RAID和LVM来满足对存储资源的高级管理需求。

本章目录结构

- 7.1 RAID磁盘冗余阵列
  - [7.1.1 部署磁盘阵列](https://www.linuxprobe.com/basic-learning-07.html#711)
  - [7.1.2 损坏磁盘阵列及修复](https://www.linuxprobe.com/basic-learning-07.html#712)
  - [7.1.3 磁盘阵列+备份盘](https://www.linuxprobe.com/basic-learning-07.html#713)
  - [7.1.4 删除磁盘阵列](https://www.linuxprobe.com/basic-learning-07.html#714)
- 7.2 LVM逻辑卷管理器
  - [7.2.1 部署逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#721)
  - [7.2.2 扩容逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#722)
  - [7.2.3 缩小逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#723)
  - [7.2.4 逻辑卷快照](https://www.linuxprobe.com/basic-learning-07.html#724)
  - [7.2.5 删除逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#725)

##### **7.1 RAID磁盘冗余阵列**

近年来， CPU的处理性能保持着高速增长，即便是被称为“牙膏厂”的英特尔公司也在2017年发布了i9-7980XE处理器芯片，率先让家用电脑达到了18核心36线程。2020年末，AMD公司又推出了线程撕裂者系统处理器3990X，让家用电脑也可以驾驭的了64核心128线程的处理器小怪兽了。但与此同时，硬盘设备的性能提升却不是很大，逐渐成为当代计算机整体性能的瓶颈。而且，由于硬盘设备需要进行持续、频繁、大量的IO操作，相较于其他设备，其损坏几率也大幅增加，导致重要数据丢失的几率也随之增加。

硬盘设备是计算机中较容易故障的元器件之一，加之由于其需要存储数据的特殊性质，不能像CPU、内存、电源甚至主板故障后更换新的就好，所以生产环境中一定要未雨绸缪，提前做好数据的冗余及异地备份等工作。

1988年，美国加利福尼亚大学伯克利分校首次提出并定义了**R**edundant **A**rray of **I**ndependent **D**isks技术的概念，中文名是磁盘冗余阵列，简称RAID。RAID技术通过把多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个区段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果。

任何事物都有它的两面性。RAID技术确实具有非常好的数据冗余备份功能，但是它也相应地提高了成本支出。就像原本我们只有一个电话本，但是为了避免遗失，把联系人号码信息写成了两份，自然要为此多买一个电话本，这也就相应地提升了成本支出。RAID技术的设计初衷是减少因为采购硬盘设备带来的费用支出，但是与数据本身的价值相比较，现代企业更看重的则是RAID技术所具备的冗余备份机制以及带来的硬盘吞吐量的提升。也就是说，RAID不仅降低了硬盘设备损坏后丢失数据的几率，还提升了硬盘设备的读写速度，所以它在绝大多数运营商或大中型企业中得以广泛部署和应用。

出于成本和技术方面的考虑，需要针对不同的需求在数据可靠性及读写性能上作出权衡，制定出满足各自需求的不同方案。目前已有的RAID磁盘阵列的方案至少有十几种，而[刘遄](https://www.linuxprobe.com/)老师接下来会详细讲解RAID 0、RAID 1、RAID 5与RAID 10这4种最常见的方案，这四种方案的对比如表7-1所示，其中n代表硬盘总数。

表7-3                          RAID 0、1、5、10方案技术对比

| RAID级别 | 最少硬盘 | 可用容量 | 读写性能 | 安全性 | 特点                                                         |
| -------- | -------- | -------- | -------- | ------ | ------------------------------------------------------------ |
| 0        | 2        | n        | n        | 低     | 追求最大容量和速度，任何一块盘损坏，数据全部异常。           |
| 1        | 2        | n/2      | n        | 高     | 追求最大安全性，只要阵列组中有一块硬盘可用，数据不受影响。   |
| 5        | 3        | n-1      | n-1      | 中     | 在控制成本的前提下，追求硬盘的最大容量、速度及安全性，允许有一块硬盘异常，数据不受影响。 |
| 10       | 4        | n/2      | n/2      | 高     | 综合RAID1和RAID0的优点，追求硬盘的速度和安全性，允许有一半硬盘异常（不可同组），数据不受影响 |



**1. RAID 0**

RAID 0技术把多块物理硬盘设备（至少两块）通过硬件或软件的方式串联在一起，组成一个大的卷组，并将数据依次写入到各个物理硬盘中。这样一来，在最理想的状态下，硬盘设备的读写性能会提升数倍，但是若任意一块硬盘发生故障将导致整个系统的数据都受到破坏。通俗来说，RAID 0技术能够有效地提升硬盘数据的吞吐速度，但是不具备数据备份和错误修复能力。如图7-1所示，数据被分别写入到不同的硬盘设备中，即硬盘A和硬盘B设备会分别保存数据资料，最终实现提升读取、写入速度的效果。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-0-1.jpg)

图7-1 RAID 0技术示意图

**2. RAID 1**

尽管RAID 0技术提升了硬盘设备的读写速度，但是它是将数据依次写入到各个物理硬盘中，也就是说，它的数据是分开存放的，其中任何一块硬盘发生故障都会损坏整个系统的数据。因此，如果生产环境对硬盘设备的读写速度没有要求，而是希望增加数据的安全性时，就需要用到RAID 1技术了。

在图7-2所示的RAID 1技术示意图中可以看到，它是把两块以上的硬盘设备进行绑定，在写入数据时，是将数据同时写入到多块硬盘设备上（可以将其视为数据的镜像或备份）。当其中某一块硬盘发生故障后，一般会立即自动以热交换的方式来恢复数据的正常使用。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-1-1.jpg)

图7-2 RAID 1技术示意图

考虑到写入操作时CPU切换硬盘的开销，速度会比RAID 0有微弱的降低，但在读取数据的时候，操作系统可以分别从两块硬盘中读取信息，理论读取速度的峰值可以是硬盘数量的倍数。另外平时只要保证有一块硬盘稳定运行，数据就不会出现损坏的情况，可靠性较高。

另外RAID 1技术虽然十分注重数据的安全性，但是因为是在多块硬盘设备中写入了相同的数据，因此硬盘设备的利用率得以下降，从理论上来说，图7-2所示的硬盘空间的真实可用率只有50%，由三块硬盘设备组成的RAID 1磁盘阵列的可用率只有33%左右，以此类推。而且，由于需要把数据同时写入到两块以上的硬盘设备，这无疑也在一定程度上增大了系统计算功能的负载。

那么，有没有一种RAID方案既考虑到了硬盘设备的读写速度和数据安全性，还兼顾了成本问题呢？实际上，单从数据安全和成本问题上来讲，就不可能在保持原有硬盘设备的利用率且还不增加新设备的情况下，能大幅提升数据的安全性。[刘遄](https://www.linuxprobe.com/)老师也没有必要忽悠各位读者，下面将要讲解的RAID 5技术虽然在理论上兼顾了三者（读写速度、数据安全性、成本），但实际上更像是对这三者的“相互妥协”。

**3. RAID 5**

如图7-3所示，RAID5技术是把硬盘设备的数据奇偶校验信息保存到其他硬盘设备中。RAID 5磁盘阵列组中数据的奇偶校验信息并不是单独保存到某一块硬盘设备中，而是存储到除自身以外的其他每一块硬盘设备上，这样的好处是其中任何一设备损坏后不至于出现致命缺陷；图7-3中parity部分存放的就是数据的奇偶校验信息，换句话说，就是RAID 5技术实际上没有备份硬盘中的真实数据信息，而是当硬盘设备出现问题后通过奇偶校验信息来尝试重建损坏的数据。RAID这样的技术特性“妥协”地兼顾了硬盘设备的读写速度、数据安全性与存储成本问题。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-5-1.jpg)

图7-3 RAID5技术示意图

RAID 5最少由三块硬盘组成，使用的是Disk Striping硬盘切割技术。比RAID 1级别好处就在于保存的是奇偶校验信息而不是一模一样的文件内容，所以当重复写入某个文件时，RAID 5级别的磁盘阵列组只需要对应一个奇偶校验信息就可以，效率更高，存储成本也会随之降低。

**4.  RAID 10**

鉴于RAID 5技术是因为硬盘设备的成本问题对读写速度和数据的安全性能而有了一定的妥协，但是大部分企业更在乎的是数据本身的价值而非硬盘价格，因此生产环境中主要使用RAID 10技术。

顾名思义，RAID 10技术是RAID 1+RAID 0技术的一个“组合体”。如图7-4所示，RAID 10技术需要至少4块硬盘来组建，其中先分别两两制作成RAID 1磁盘阵列，以保证数据的安全性；然后再对两个RAID 1磁盘阵列实施RAID 0技术，进一步提高硬盘设备的读写速度。这样从理论上来讲，只要坏的不是同一组中的所有硬盘，那么最多可以损坏50%的硬盘设备而不丢失数据。由于RAID 10技术继承了RAID 0的高读写速度和RAID 1的数据安全性，在不考虑成本的情况下RAID 10的性能都超过了RAID 5，因此当前成为广泛使用的一种存储技术。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-10-2.jpg)

图7-4 RAID 10技术示意图

由于RAID 10是由RAID 1和RAID 0组成的，因此正确叫法是“RAID 一零”，而不是“RAID 十”。

细看上图7-4可以分析出，RAID 10是先对信息进行分割，然后再两两一组做的镜像。也就是将RAID 1作为最低级别的组合，再使用RAID 0技术组合到一起，将它们视为“一整块”硬盘。而RAID 01则是相反的，它回先将硬盘分为两组，使用RAID 0作为最低级别的组合，再将两组硬盘通过RAID 1技术组合到一起。

但区别非常明显，RAID 10级别中任何一块硬盘损坏都不会影响到数据安全性，其余硬盘均会正常运作。但RAID 01只要有任何一块盘损坏，最低级别的RAID 0硬盘组马上会停止运作，可能造成严重隐患。所以RAID 10远比RAID 01常见，很多主板甚至不支持RAID 01。

###### **7.1.1 部署磁盘阵列**

在具备了第六章中管理硬盘设备的基础知识后，再来部署RAID和LVM就变得十分轻松了。首先，需要在虚拟机中添加4块硬盘设备来制作一个RAID 10磁盘阵列，如图7-5所示。添加硬盘的步骤省略大家自己操作就行，记得要用SCSI或SATA接口类型，大小默认20GB就可以。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E5%9B%9B%E5%9D%97%E7%A1%AC%E7%9B%98%E8%AE%BE%E5%A4%87-1.png)

图7-5 添加四块硬盘设备

这几块硬盘设备是模拟出来的，不需要特意去买几块真实的物理硬盘插到电脑上。需要注意的是，一定要记得在关闭系统之后，再在虚拟机中添加硬盘设备，否则可能会因为计算机架构的不同而导致虚拟机系统无法识别新添加的硬盘设备。

当前，生产环境中用到的服务器一般都配备RAID阵列卡，尽管服务器的价格越来越便宜，但是我们没有必要为了做一个实验而去单独购买一台服务器，而是可以学会用mdadm[命令](https://www.linuxcool.com/)在[Linux系统](https://www.linuxprobe.com/)中创建和管理软件RAID磁盘阵列，而且它涉及的理论知识的操作过程与生产环境中的完全一致。mdadm[命令](https://www.linuxcool.com/)的常用参数以及作用如表7-2所示。

mdadm命令用于创建、调整、监控和管理RAID设备，英文全称为：“multiple devices admin”，语法格式为：“mdadm 参数 硬盘名称”。

表7-2                      mdadm命令的常用参数和作用

| 参数 | 作用             |
| ---- | ---------------- |
| -a   | 检测设备名称     |
| -n   | 指定设备数量     |
| -l   | 指定RAID级别     |
| -C   | 创建             |
| -v   | 显示过程         |
| -f   | 模拟设备损坏     |
| -r   | 移除设备         |
| -Q   | 查看摘要信息     |
| -D   | 查看详细信息     |
| -S   | 停止RAID磁盘阵列 |



接下来，使用mdadm命令创建RAID 10，名称为“/dev/md0”。

第6章中讲到，udev是[Linux](https://www.linuxprobe.com/)系统内核中用来给硬件命名的服务，其命名规则也非常简单。我们可以通过命名规则猜测到第二个SCSI存储设备的名称会是/dev/sdb，然后依此类推。使用硬盘设备来部署RAID磁盘阵列很像是将几位同学组成一个班级，但总不能将班级命名为/dev/sdbcde吧。尽管这样可以一眼看出它是由哪些元素组成的，但是并不利于记忆和阅读。更何况如果是使用10、50、100个硬盘来部署RAID磁盘阵列呢？

此时，就需要使用mdadm中的参数了。其中，-C参数代表创建一个RAID阵列卡；-v参数显示创建的过程，同时在后面追加一个设备名称/dev/md0，这样/dev/md0就是创建后的RAID磁盘阵列的名称；-n 4参数代表使用4块硬盘来部署这个RAID磁盘阵列；而-l 10参数则代表RAID 10方案；最后再加上4块硬盘设备的名称就搞定了。

```shell
[root@linuxprobe ~]# mdadm -Cv /dev/md0 -n 4 -l 10 /dev/sdb /dev/sdc /dev/sdd /dev/sde
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 20954112K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

初始化过程可以用-D参数进行查看，大约需要1分钟左右，也可以用-Q参数查看简要信息：

```shell
[root@linuxprobe ~]# mdadm -Q /dev/md0
/dev/md0: 39.97GiB raid10 4 devices, 0 spares. Use mdadm --detail for more detail.
```

同学们会好奇，为什么四块20G大小的硬盘组成的磁盘阵列组，可用空间只有39.97G呢？

这还是不得不要提到RAID 10级别的原理，通过两两一组硬盘组成的RAID 1保证了数据的可靠性，每一份数据都会被保存两次，50%的使用率，50%的冗余率，因此80G的容量显示也就是只有一半了。

等两三分钟后，把制作好的RAID磁盘阵列格式化为ext4格式：

```shell
[root@linuxprobe ~]# mkfs.ext4 /dev/md0
mke2fs 1.44.3 (10-July-2018)
Creating filesystem with 10477056 4k blocks and 2621440 inodes
Filesystem UUID: d1c68318-a919-4211-b4dc-c4437bcfe9da
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   
```

随后，创建挂载点后把硬盘设备进行挂载操作。

```shell
[root@linuxprobe ~]# mkdir /RAID
[root@linuxprobe ~]# mount /dev/md0 /RAID
[root@linuxprobe ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               969M     0  969M   0% /dev
tmpfs                  984M     0  984M   0% /dev/shm
tmpfs                  984M  9.6M  975M   1% /run
tmpfs                  984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.9G   14G  23% /
/dev/sr0               6.7G  6.7G     0 100% /media/cdrom
/dev/sda1             1014M  152M  863M  15% /boot
tmpfs                  197M   16K  197M   1% /run/user/42
tmpfs                  197M  3.5M  194M   2% /run/user/0
/dev/md0                40G   49M   38G   1% /RAID
```

再来查看/dev/md0磁盘阵列组设备的详细信息，确认下RAID级别（Raid Level）、大小（Array Size）和总硬盘数（Total Devices）都是否正确：

```shell
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jan 13 08:24:58 2021
        Raid Level : raid10
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 04:49:57 2021
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host linuxprobe.com)
              UUID : 289f501b:3f5f70f9:79189d77:f51ca11a
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```

如果想让创建好的RAID磁盘阵列组能够一直为我们服务，不会因每次的重启操作而取消，那么一定要记得将信息添加到/etc/fstab文件中，这样每次重启后还都是有效的。

```shell
[root@linuxprobe ~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
[root@linuxprobe ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /                 xfs         defaults      0 0
UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot             xfs         defaults      0 0
/dev/mapper/rhel-swap                       swap              swap        defaults      0 0
/dev/cdrom                                  /media/cdrom      iso9660     defaults      0 0 
/dev/md0                                    /RAID             ext4        defaults      0 0
```

###### **7.1.2 损坏磁盘阵列及修复**

咱们在生产环境中部署RAID 10磁盘阵列组目的就是为了提高存储设备的IO读写速度及数据的安全性，但因为这次是在本机电脑上模拟出来的硬盘设备所以对于读写速度的改善可能并不直观，因此决定给同学们讲解下RAID磁盘阵列组损坏后的处理方法，这样以后步入了运维岗位后不会因为突发事件而手忙脚乱。首先确认有一块物理硬盘设备出现损坏不能再继续正常使用后，应该使用mdadm命令来予以移除之后查看下RAID磁盘阵列组的状态已经被改变。

在确认有一块物理硬盘设备出现损坏而不能继续正常使用后，应该使用mdadm命令将其移除，然后查看RAID磁盘阵列的状态，可以发现状态已经改变。

```shell
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Jan 14 05:12:20 2021
        Raid Level : raid10
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 05:33:06 2021
             State : clean, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 81ee0668:7627c733:0b170c41:cd12f376
            Events : 19

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       0       8       16        -      faulty   /dev/sdb
```

刚刚使用的-f参数是让硬盘模拟损坏，为了能够彻底的将故障盘移除，还要再来一步操作：

```shell
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
```

在RAID 10级别的磁盘阵列中，当RAID 1磁盘阵列中存在一个故障盘时并不影响RAID 10磁盘阵列的使用。当购买了新的硬盘设备后再使用mdadm命令来予以替换即可，在此期间可以在/RAID目录中正常地创建或删除文件。由于我们是在虚拟机中模拟硬盘，所以先重启系统，然后再把新的硬盘添加到RAID磁盘阵列中。

更换硬盘后再次使用-a参数进行添加操作，默认会自动开始数据的同步工作，使用-D参数即可看到整个过程和百分比进度：

```shell
[root@linuxprobe ~]# mdadm /dev/md0 -a /dev/sdb
mdadm: added /dev/sdb
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Jan 14 05:12:20 2021
        Raid Level : raid10
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 05:37:32 2021
             State : clean, degraded, recovering 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

    Rebuild Status : 77% complete

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 81ee0668:7627c733:0b170c41:cd12f376
            Events : 34

    Number   Major   Minor   RaidDevice State
       4       8       16        0      spare rebuilding    /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```

这时候会有小可爱学生举手提问了：“老师，我们公司机房的阵列卡上带了三十几块硬盘呢，就算看到了/dev/sdb硬盘故障了，我也不知道该替换那一块啊，怕错拔了好设备”。其实这到不用担心，因为一旦硬盘故障了，服务器上指示灯也会转成红灯（或者闪烁黄灯），如图7-6所示，对照着处理即可。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%A1%AC%E7%9B%98%E6%95%85%E9%9A%9C.png)

###### **7.1.3 磁盘阵列+备份盘**

RAID 10磁盘阵列中最多允许50%的硬盘设备发生故障，但是存在这样一种极端情况，即同一RAID 1磁盘阵列中的硬盘设备若全部损坏，也会导致数据丢失。换句话说，在RAID 10磁盘阵列中，如果RAID 1中的某一块硬盘出现了故障，而我们正在前往修复的路上，恰巧该RAID1磁盘阵列中的另一块硬盘设备也出现故障，那么数据就被彻底丢失了。刘遄老师可真不是乌鸦嘴，这种RAID 1磁盘阵列中的硬盘设备同时损坏的情况还真被我的学生遇到过。

在这样的情况下，该怎么办呢？其实，完全可以使用RAID备份盘技术来预防这类事故。该技术的核心理念就是准备一块足够大的硬盘，这块硬盘平时处于闲置状态，一旦RAID磁盘阵列中有硬盘出现故障后则会马上自动顶替上去。这样很棒吧！

为了避免多个实验之间相互发生冲突，我们需要保证每个实验的相对独立性，为此需要大家自行将虚拟机还原到初始状态。另外，由于刚才已经演示了RAID 10磁盘阵列的部署方法，现在来看一下RAID 5的部署效果吧。部署RAID 5磁盘阵列时，至少需要用到3块硬盘，还需要再加一块备份硬盘（也叫热备盘），所以总计需要在虚拟机中模拟4块硬盘设备，如图7-7所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E5%9B%9B%E5%9D%97%E7%A1%AC%E7%9B%98%E8%AE%BE%E5%A4%87-1.png)

图7-7 重置虚拟机后，再添加四块硬盘设备

现在创建一个RAID 5磁盘阵列+备份盘。在下面的命令中，参数-n 3代表创建这个RAID 5磁盘阵列所需的硬盘数，参数-l 5代表RAID的级别，而参数-x 1则代表有一块备份盘。当查看/dev/md0（即RAID 5磁盘阵列的名称）磁盘阵列的时候就能看到有一块备份盘在等待中了。

```shell
[root@linuxprobe ~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sdd /dev/sde
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 20954112K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Jan 14 06:12:32 2021
        Raid Level : raid5
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 06:14:16 2021
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf0c34b6:3b08edfb:85dfa14f:e2bffc1e
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       4       8       48        2      active sync   /dev/sdd

       3       8       64        -      spare   /dev/sde
```

现在将部署好的RAID 5磁盘阵列格式化为ext4文件格式，然后挂载到目录上，之后就能够使用了：

```shell
[root@linuxprobe ~]# mkfs.ext4 /dev/md0
mke2fs 1.44.3 (10-July-2018)
Creating filesystem with 10477056 4k blocks and 2621440 inodes
Filesystem UUID: ff016386-1126-4799-8a5b-d716242276ec
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   
[root@linuxprobe ~]# mkdir /RAID
[root@linuxprobe ~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
```

由三块硬盘组成的RAID 5级别磁盘阵列，它对应的可用空间是n-1，也就是40G。热备盘的空间是不算入内的，平时完全就是在“睡觉”中，只有意外出现时才会开始工作。

```shell
[root@linuxprobe ~]# mount -a
[root@linuxprobe ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               969M     0  969M   0% /dev
tmpfs                  984M     0  984M   0% /dev/shm
tmpfs                  984M  9.6M  974M   1% /run
tmpfs                  984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.9G   14G  23% /
/dev/sr0               6.7G  6.7G     0 100% /media/cdrom
/dev/sda1             1014M  152M  863M  15% /boot
tmpfs                  197M   16K  197M   1% /run/user/42
tmpfs                  197M  3.5M  194M   2% /run/user/0
/dev/md0                40G   49M   38G   1% /RAID
```

最后是见证奇迹的时刻！我们再次把硬盘设备/dev/sdb移出磁盘阵列，然后迅速查看/dev/md0磁盘阵列的状态，就会发现备份盘已经被自动顶替上去并开始了数据同步。RAID中的这种备份盘技术非常实用，可以在保证RAID磁盘阵列数据安全性的基础上进一步提高数据可靠性，所以，如果公司不差钱的话还是再买上一块备份盘以防万一。

```shell
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Jan 14 06:12:32 2021
        Raid Level : raid5
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 06:24:38 2021
             State : clean 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : cf0c34b6:3b08edfb:85dfa14f:e2bffc1e
            Events : 37

    Number   Major   Minor   RaidDevice State
       3       8       64        0      active sync   /dev/sde
       1       8       32        1      active sync   /dev/sdc
       4       8       48        2      active sync   /dev/sdd

       0       8       16        -      faulty   /dev/sdb
```

是不是感觉很有意思呢，另外考虑到不想让篇幅过长，所以我们刚刚一直没有复制粘贴/RAID目录中文件的信息，有兴趣的同学可以自己动手试一下，里面的文件内容非常安全，不会出现丢失的情况。如果后面像再添加进去一块热备盘，使用-a参数就可以啦。

###### **7.1.4 删除磁盘阵列**

生产环境中，RAID磁盘阵列组部署后一般就不会轻易被停用了，但万一赶上了，还是要知道怎么样删除的。上面这种RAID 5+热备盘损坏的情况是比较复杂的，就以这种情形来进行讲解是再好不过的了。

首先需要将所有的磁盘都设置成停用状态：

```shell
[root@linuxprobe ~]# umount /RAID
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdd
mdadm: set /dev/sdd faulty in /dev/md0
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sde
mdadm: set /dev/sde faulty in /dev/md0
```

然后再逐一的移除出去：

```shell
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md0
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdd
mdadm: hot removed /dev/sdd from /dev/md0
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sde
mdadm: hot removed /dev/sde from /dev/md0
```

如果着急的同学也可以用“mdadm /dev/md0 -f /dev/sdb -r /dev/sdb”一条命令搞定。但由于这个命令在早期版本的服务器中不能一起使用，因此还是保守起见一步步的操作吧。

将所有的硬盘都移除后，再来看下磁盘阵列组的状态：

```shell
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Fri Jan 15 08:53:41 2021
        Raid Level : raid5
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 3
     Total Devices : 0
       Persistence : Superblock is persistent

       Update Time : Fri Jan 15 09:00:57 2021
             State : clean, FAILED 
    Active Devices : 0
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       -       0        0        1      removed
       -       0        0        2      removed
```

很棒，继续再停用整个RAID磁盘组，咱们的工作就彻底的完成了：

```shell
[root@linuxprobe ~]# mdadm --stop /dev/md0
mdadm: stopped /dev/md0
[root@linuxprobe ~]# ls /dev/md0
ls: cannot access '/dev/md0': No such file or directory
```

有一些老版本的服务器，使用完--stop参数后依然会保留设备文件，很明显是没有处理干净，这时再执行一下：“mdadm --remove /dev/md0”命令即可，同学们可以记一下，以备不时之需。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **7.2 LVM逻辑卷管理器**

前面学习的硬盘设备管理技术虽然能够有效地提高硬盘设备的读写速度以及数据的安全性，但是在硬盘分好区或者部署为RAID磁盘阵列之后，再想修改硬盘分区大小就不容易了。换句话说，当用户想要随着实际需求的变化调整硬盘分区的大小时，会受到硬盘“灵活性”的限制。这时就需要用到另外一项非常普及的硬盘设备资源管理技术了— Logical Volume Manager（逻辑卷管理器，简称LVM）。LVM允许用户对硬盘资源进行动态调整。

逻辑卷管理器是Linux系统用于对硬盘分区进行管理的一种机制，理论性较强，其创建初衷是为了解决硬盘设备在创建分区后不易修改分区大小的缺陷。尽管对传统的硬盘分区进行强制扩容或缩容从理论上来讲是可行的，但是却可能造成数据的丢失。而LVM技术是在硬盘分区和文件系统之间添加了一个逻辑层，它提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。这样一来，用户不必关心物理硬盘设备的底层架构和布局，就可以实现对硬盘分区的动态调整。LVM的技术架构如图7-8所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/LVM%E9%80%BB%E8%BE%91%E5%8D%B7%E7%AE%A1%E7%90%86%E5%99%A8-1.jpg)

图7-8 逻辑卷管理器的技术结构

为了帮助大家理解，来举一个吃货的例子吧。比如小明家里想吃馒头但是面粉不够了，于是妈妈从隔壁老王家、老李家、老张家分别借来一些面粉，准备蒸馒头吃。首先需要把这些面粉（物理卷[PV，Physical Volume]）揉成一个大面团（卷组[VG，Volume Group]），然后再把这个大面团分割成一个个小馒头（逻辑卷[LV，Logical Volume]），而且每个小馒头的重量必须是每勺面粉（基本单元[PE，Physical Extent]）的倍数。

在日常的使用中，如果VG卷组的剩余容量不足了，可以随时将新的PV物理卷加入到里面去，不断的扩容。怕同学们还是不好理解，又准备了一张逻辑卷管理器的使用流程示意图，如图7-9所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/LVM%E9%80%BB%E8%BE%91%E5%8D%B7%E7%AE%A1%E7%90%86%E6%9C%9F%E4%BD%BF%E7%94%A8%E6%B5%81%E7%A8%8B%E7%A4%BA%E6%84%8F%E5%9B%BE-1.jpg)

图7-9 逻辑卷管理器使用流程图

物理卷处于LVM中的最底层，可以将其理解为物理硬盘、硬盘分区或者RAID磁盘阵列。卷组建立在物理卷之上，一个卷组能够包含多个物理卷，而且在卷组创建之后也可以继续向其中添加新的物理卷。逻辑卷是用卷组中空闲的资源建立的，并且逻辑卷在建立后可以动态地扩展或缩小空间。这就是LVM的核心理念。

###### **7.2.1 部署逻辑卷**

一般而言，在生产环境中无法在最初时就精确地评估每个硬盘分区在日后的使用情况，因此会导致原先分配的硬盘分区不够用。比如，伴随着业务量的增加，用于存放交易记录的数据库目录的体积也随之增加；因为分析并记录用户的行为从而导致日志目录的体积不断变大，这些都会导致原有的硬盘分区在使用上捉襟见肘。而且，还存在对较大的硬盘分区进行精简缩容的情况。

我们可以通过部署LVM来解决上述问题。部署时需要逐个配置物理卷、卷组和逻辑卷，常用的部署命令如表7-3所示。

表7-3                          常用的LVM部署命令

| 功能/命令 | 物理卷管理 | 卷组管理  | 逻辑卷管理 |
| --------- | ---------- | --------- | ---------- |
| 扫描      | pvscan     | vgscan    | lvscan     |
| 建立      | pvcreate   | vgcreate  | lvcreate   |
| 显示      | pvdisplay  | vgdisplay | lvdisplay  |
| 删除      | pvremove   | vgremove  | lvremove   |
| 扩展      |            | vgextend  | lvextend   |
| 缩小      |            | vgreduce  | lvreduce   |



为了避免实验之间相互发生冲突，请大家自行将虚拟机还原到初始状态，并重新添加两块新硬盘设备，如图7-10所示，然后开机。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E4%B8%A4%E5%9D%97%E6%96%B0%E7%A1%AC%E7%9B%98-1.png)

图7-10 在虚拟机中添加两块新的硬盘设备

在虚拟机中添加两块新硬盘设备的目的，是为了更好地演示LVM理念中用户无需关心底层物理硬盘设备的特性。我们先对这两块新硬盘进行创建物理卷的操作，可以将该操作简单理解成让硬盘设备支持LVM技术，或者理解成是把硬盘设备加入到LVM技术可用的硬件资源池中，然后对这两块硬盘进行卷组合并，卷组的名称允许由用户来自定义。接下来，根据需求把合并后的卷组切割出一个约为150MB的逻辑卷设备，最后把这个逻辑卷设备格式化成EXT4文件系统后挂载使用。在下文中，将对每一个步骤再作一些简单的描述。

**第1步**：让新添加的两块硬盘设备支持LVM技术。

```shell
[root@linuxprobe ~]# pvcreate /dev/sdb /dev/sdc
  Physical volume "/dev/sdb" successfully created.
  Physical volume "/dev/sdc" successfully created.
```

**第2步**：把两块硬盘设备加入到storage卷组中，然后查看卷组的状态。

```shell
[root@linuxprobe ~]# vgcreate storage /dev/sdb /dev/sdc
 Volume group "storage" successfully created
[root@linuxprobe ~]# vgdisplay
  --- Volume group ---
  VG Name               storage
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               39.99 GiB
  PE Size               4.00 MiB
  Total PE              10238
  Alloc PE / Size       0 / 0   
  Free  PE / Size       10238 / 39.99 GiB
  VG UUID               HPwsm4-lOvI-8O0Q-TG54-BkyI-ONYE-owlGLd
………………省略部分输出信息………………
```

**第3步**：再切割出一个约为150MB的逻辑卷设备。

这里需要注意切割单位的问题。在对逻辑卷进行切割时有两种计量单位。第一种是以容量为单位，所使用的参数为-L。例如，使用-L 150M生成一个大小为150MB的逻辑卷。另外一种是以基本单元的个数为单位，所使用的参数为-l。每个基本单元的大小默认为4MB。例如，使用-l 37可以生成一个大小为37×4MB=148MB的逻辑卷。

```shell
[root@linuxprobe ~]# lvcreate -n vo -l 37 storage
 Logical volume "vo" created.
[root@linuxprobe ~]# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/storage/vo
  LV Name                vo
  VG Name                storage
  LV UUID                AsDGJj-G6Uo-HG4q-auD6-lmyn-aLY0-o36HEj
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2021-01-15 00:47:35 +0800
  LV Status              available
  # open                 0
  LV Size                148.00 MiB
  Current LE             37
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
………………省略部分输出信息………………
```

**第4步**：把生成好的逻辑卷进行格式化，然后挂载使用。

Linux系统会把LVM中的逻辑卷设备存放在/dev设备目录中，实际上就是个快捷方式，同时会以卷组的名称来建立一个目录，其中保存了逻辑卷的设备映射文件，即/dev/卷组名称/逻辑卷名称。

```shell
[root@linuxprobe ~]# mkfs.ext4 /dev/storage/vo 
mke2fs 1.44.3 (10-July-2018)
Creating filesystem with 151552 1k blocks and 38000 inodes
Filesystem UUID: 429cbc28-4463-4a1b-b601-02a7cf81a1b2
Superblock backups stored on blocks: 
	8193, 24577, 40961, 57345, 73729

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done 
[root@linuxprobe ~]# mkdir /linuxprobe
[root@linuxprobe ~]# mount /dev/storage/vo /linuxprobe
```

对了，如果用了LVM逻辑卷管理器的话，不建议用XFS文件系统。因为XFS文件系统自身就可以使用xfs_growfs命令进行磁盘扩容，虽然不比LVM灵活，但起码也是够用的。在实测阶段我们发现，有一些服务器上XFS与LVM兼容性并不好。

**第5步**：查看挂载状态，并写入到配置文件，使其永久生效。

```shell
[root@linuxprobe ~]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sr0                6.7G  6.7G     0 100% /media/cdrom
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
/dev/mapper/storage-vo  140M  1.6M  128M   2% /linuxprobe
[root@linuxprobe ~]# echo "/dev/storage/vo /linuxprobe ext4 defaults 0 0" >> /etc/fstab
[root@linuxprobe ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /                       xfs      defaults        0 0
UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot                   xfs      defaults        0 0
/dev/mapper/rhel-swap                       swap                    swap     defaults        0 0
/dev/cdrom                                  /media/cdrom            iso9660  defaults        0 0 
/dev/storage/vo                             /linuxprobe             ext4     defaults        0 0
```

细心的同学又发现了个小问题，刚刚明明写的是148M，怎么查看到只有140M了呢？这是因为硬件厂商制造标准是1GB=1,000MB、1MB＝1,000KB、1KB＝1,000byte，而计算机系统的算法是1GB=1,024MB、1MB＝1,024KB、1KB＝1,024byte，会有3%左右的“缩水”是正常情况。

###### **7.2.2 扩容逻辑卷**

在前面的实验中，卷组是由两块硬盘设备共同组成的。用户在使用存储设备时感知不到设备底层的架构和布局，更不用关心底层是由多少块硬盘组成的，只要卷组中有足够的资源，就可以一直为逻辑卷扩容。扩展前请一定要记得卸载设备和挂载点的关联。

```shell
[root@linuxprobe ~]# umount /linuxprobe
```

**第1步**：把上一个实验中的逻辑卷vo扩展至290M。

```shell
[root@linuxprobe ~]# lvextend -L 290M /dev/storage/vo
Rounding size to boundary between physical extents: 292.00 MiB.
Size of logical volume storage/vo changed from 148 MiB (37 extents) to 292 MiB (73 extents).
Logical volume storage/vo successfully resized.
```

**第2步**：检查硬盘的完整性，确认目录结构、内容和文件内容没有丢失，一般情况下是要没有报错，均为正常情况。

```shell
[root@linuxprobe ~]# e2fsck -f /dev/storage/vo
e2fsck 1.44.3 (10-July-2018)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/storage/vo: 11/38000 files (0.0% non-contiguous), 10453/151552 blocks
```

**第3步**：重置设备在系统中的容量，刚刚是对LV逻辑卷设备进行了扩容操作，但系统内核还没有同步到这部分新修改的信息，需要手动进行同步。

```shell
[root@linuxprobe ~]# resize2fs /dev/storage/vo
resize2fs 1.44.3 (10-July-2018)
Resizing the filesystem on /dev/storage/vo to 299008 (1k) blocks.
The filesystem on /dev/storage/vo is now 299008 (1k) blocks long.
```

**第4步**：重新挂载硬盘设备并查看挂载状态。

```shell
[root@linuxprobe ~]# mount -a
[root@linuxprobe ~]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sr0                6.7G  6.7G     0 100% /media/cdrom
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
/dev/mapper/storage-vo  279M  2.1M  259M   1% /linuxprobe
```

###### **7.2.3 缩小逻辑卷**

相较于扩容逻辑卷，在对逻辑卷进行缩容操作时，其丢失数据的风险更大。所以在生产环境中执行相应操作时，一定要提前备份好数据。另外Linux系统规定，在对LVM逻辑卷进行缩容操作之前，要先检查文件系统的完整性（当然这也是为了保证数据安全）。在执行缩容操作前记得先把文件系统卸载掉。

```shell
[root@linuxprobe ~]# umount /linuxprobe
```

**第1步**：检查文件系统的完整性。

```shell
[root@linuxprobe ~]# e2fsck -f /dev/storage/vo
e2fsck 1.44.3 (10-July-2018)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/storage/vo: 11/74000 files (0.0% non-contiguous), 15507/299008 blocks
```

**第2步**：通知系统内核将逻辑卷vo的容量减小到120M。

```shell
[root@linuxprobe ~]# resize2fs /dev/storage/vo 120M
resize2fs 1.44.3 (10-July-2018)
Resizing the filesystem on /dev/storage/vo to 122880 (1k) blocks.
The filesystem on /dev/storage/vo is now 122880 (1k) blocks long.
```

**第3步**：将LV逻辑卷的容量修改为120M。

```shell
[root@linuxprobe ~]# lvreduce -L 120M /dev/storage/vo
  WARNING: Reducing active logical volume to 120.00 MiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce storage/vo? [y/n]: y
  Size of logical volume storage/vo changed from 292 MiB (73 extents) to 120 MiB (30 extents).
  Logical volume storage/vo successfully resized.
```

咦？步骤跟扩容是反过来了，缩小操作是先通知系统内核设备要改变成120M，然后再正式进行缩小操作，这是为什么呢？举个例子大家就明白了。小强是初中生，开学后看到班里有位同学纹身了感觉很酷，自己也想纹个身又怕家里责骂，于是他回家了就说：“爸妈，我纹身了”。如果他父母的反应是很平和，那么他就可以放心大胆的去纹身了~如果父母的态度非常强烈的不同意，他马上就可以哈哈一笑说逗着玩呢，也不会挨打。

所以缩小操作也是同样的道理，先通知系统内核自己想缩小逻辑卷，如果resize2fs命令执行后没有报错，系统允许了，再正式操作。

**第4步**：重新挂载文件系统并查看系统状态。

```shell
[root@linuxprobe ~]# mount -a
[root@linuxprobe ~]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sr0                6.7G  6.7G     0 100% /media/cdrom
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
/dev/mapper/storage-vo  113M  1.6M  103M   2% /linuxprobe
```

###### **7.2.4 逻辑卷快照**

LVM还具备有“快照卷”功能，该功能类似于虚拟机软件的还原时间点功能。例如，对某一个逻辑卷设备做一次快照，如果日后发现数据被改错了，就可以利用之前做好的快照卷进行覆盖还原。LVM的快照卷功能有两个特点：

> 快照卷的容量必须等同于逻辑卷的容量；
>
> 快照卷仅一次有效，一旦执行还原操作后则会被立即自动删除。

在正式操作前，先看看VG卷组中的容量是否够用：

```shell
[root@linuxprobe ~]# vgdisplay
  --- Volume group ---
  VG Name               storage
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               39.99 GiB
  PE Size               4.00 MiB
  Total PE              10238
  Alloc PE / Size       30 / 120.00 MiB
  Free  PE / Size       10208 / <39.88 GiB
  VG UUID               k3ZnaP-wGPr-TQJ5-PCtA-0RgO-jvsi-9elZ5M
………………省略部分输出信息………………
```

通过卷组的输出信息可以清晰看到，卷组中已经使用了120MB的容量，空闲容量还有39.88GB。接下来用重定向往逻辑卷设备所挂载的目录中写入一个文件。

```shell
[root@linuxprobe ~]# echo "Welcome to Linuxprobe.com" > /linuxprobe/readme.txt
[root@linuxprobe ~]# ls -l /linuxprobe
total 14
drwx------. 2 root root 12288 Jan 15 01:11 lost+found
-rw-r--r--. 1 root root    26 Jan 15 07:01 readme.txt
```

**第1步**：使用-s参数生成一个快照卷，使用-L参数指定切割的大小，需要与要做快照的设备容量保持一致。另外，还需要在命令后面写上是针对哪个逻辑卷执行的快照操作，稍后数据也会还原到这个对应的设备上。

```shell
[root@linuxprobe ~]# lvcreate -L 120M -s -n SNAP /dev/storage/vo
 Logical volume "SNAP" created
[root@linuxprobe ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/SNAP
  LV Name                SNAP
  VG Name                storage
  LV UUID                qd7l6w-3Iv1-6E3X-RGkC-t5xl-170r-rDZSEf
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2021-01-15 07:02:44 +0800
  LV snapshot status     active destination for vo
  LV Status              available
  # open                 0
  LV Size                120.00 MiB
  Current LE             30
  COW-table size         120.00 MiB
  COW-table LE           30
  Allocated to snapshot  0.01%
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:5
………………省略部分输出信息………………
```

**第2步**：在逻辑卷所挂载的目录中创建一个100MB的垃圾文件，然后再查看快照卷的状态。可以发现存储空间占的用量上升了。

```shell
[root@linuxprobe ~]# dd if=/dev/zero of=/linuxprobe/files count=1 bs=100M
1+0 records in
1+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.312057 s, 336 MB/s
[root@linuxprobe ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/storage/SNAP
  LV Name                SNAP
  VG Name                storage
  LV UUID                qd7l6w-3Iv1-6E3X-RGkC-t5xl-170r-rDZSEf
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2021-01-15 07:02:44 +0800
  LV snapshot status     active destination for vo
  LV Status              available
  # open                 0
  LV Size                120.00 MiB
  Current LE             30
  COW-table size         120.00 MiB
  COW-table LE           30
  Allocated to snapshot  83.71%
  Snapshot chunk size    4.00 KiB
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:5
………………省略部分输出信息………………
```

**第3步**：为了校验SNAP快照卷的效果，需要对逻辑卷进行快照还原操作。在此之前记得先卸载掉逻辑卷设备与目录的挂载。

lvconvert命令用于管理逻辑卷的快照，语法格式为：“lvconvert [参数] 快照卷名称”。

使用lvconvert能够将逻辑卷的快照进行自动恢复，在早期5版本中要写全格式：“--mergesnapshot”，而从6版本开始至8版本，已经允许用户只输入“--merge”参数进行操作了，系统会自动分辨设备的类型。

```shell
[root@linuxprobe ~]# umount /linuxprobe
[root@linuxprobe ~]# lvconvert --merge /dev/storage/SNAP
  Merging of volume storage/SNAP started.
  storage/vo: Merged: 36.41%
  storage/vo: Merged: 100.00%
```

**第4步**：快照卷会被自动删除掉，并且刚刚在逻辑卷设备被执行快照操作后再创建出来的100MB的垃圾文件也被清除了。

```shell
[root@linuxprobe ~]# mount -a
[root@linuxprobe ~]# cd /linuxprobe/
[root@linuxprobe linuxprobe]# ls
lost+found readme.txt
[root@linuxprobe linuxprobe]# cat readme.txt 
Welcome to Linuxprobe.com
```

###### **7.2.5 删除逻辑卷**

当生产环境中想要重新部署LVM或者不再需要使用时，则需要执行LVM的删除操作。为此，需要提前备份好重要的数据信息，然后依次删除逻辑卷、卷组、物理卷设备，这个顺序不可颠倒。

**第1步**：取消逻辑卷与目录的挂载关联，删除配置文件中永久生效的设备参数。

```shell
[root@linuxprobe ~]# umount /linuxprobe
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /                       xfs      defaults        0 0
UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot                   xfs      defaults        0 0
/dev/mapper/rhel-swap                       swap                    swap     defaults        0 0
/dev/cdrom                                  /media/cdrom            iso9660  defaults        0 0 
/dev/storage/vo                             /linuxprobe             ext4     defaults        0 0
```

**第2步**：删除逻辑卷设备，需要输入y来确认操作。

```shell
[root@linuxprobe ~]# lvremove /dev/storage/vo 
Do you really want to remove active logical volume storage/vo? [y/n]: y
  Logical volume "vo" successfully removed
```

**第3步**：删除卷组，此处只写卷组名称即可，不需要设备的绝对路径。

```shell
[root@linuxprobe ~]# vgremove storage
  Volume group "storage" successfully removed
```

**第4步**：删除物理卷设备。

```shell
[root@linuxprobe ~]# pvremove /dev/sdb /dev/sdc
  Labels on physical volume "/dev/sdb" successfully wiped.
  Labels on physical volume "/dev/sdc" successfully wiped.
```

在上述操作执行完毕之后，再执行lvdisplay、vgdisplay、pvdisplay命令来查看LVM的信息时就不会再看到信息了（前提是上述步骤的操作是正确的），干净利落！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1． RAID技术主要是为了解决什么问题呢？

**答：**RAID技术可以解决存储设备的读写速度问题及数据的冗余备份问题。

2． RAID 0和RAID 5哪个更安全？

**答：**RAID 0没有数据冗余功能，因此RAID 5更安全。

3．假设使用4块硬盘来部署RAID 10方案，外加一块备份盘，最多可以允许几块硬盘同时损坏呢？

**答：**最多允许5块硬盘设备中的3块设备同时损坏。

4．位于LVM最底层的是物理卷还是卷组？

**答：**最底层的是物理卷，然后在通过物理卷组成卷组。

5． LVM对逻辑卷的扩容和缩容操作有何异同点呢？

**答：**扩容和缩容操作都需要先取消逻辑卷与目录的挂载关联；扩容操作是先扩容后检查文件系统完整性，而缩容操作为了保证数据的安全，需要先检查文件系统完整性再缩容。

6． LVM的快照卷能使用几次？

**答：**只可使用一次，而且使用后即自动删除。

7． LVM的删除顺序是怎么样的？

**答：**依次移除逻辑卷、卷组和物理卷。



 

## Iptables与firewalld防火墙技术

**章节简述：**

保障数据的安全性是继保障数据的可用性之后最为重要的一项工作。防火墙作为公网与内网之间的保护屏障，在保障数据的安全性方面起着至关重要的作用。考虑到大家还不了解RHEL 7/8中新增的firewalld防火墙与先前版本中iptables防火墙之间的区别，[刘遄](https://www.linuxprobe.com/)老师决定先带领读者从理论层面和实际层面正确地认识在这两款防火墙之间的关系。

本章将分别使用iptables、firewall-cmd、firewall-config和TCP Wrappers等防火墙策略配置服务来完成数十个根据真实工作需求而设计的防火墙策略配置实验。在学习完这些实验之后，各位读者不仅能够熟练地过滤请求的流量、基于服务程序的名称对流量进行允许和拒绝操作，还可以使用Cockpit轻松监控系统运行状态，确保[Linux系统](https://www.linuxprobe.com/)的安全性万无一失。

本章目录结构

- [8.1 防火墙管理工具](https://www.linuxprobe.com/basic-learning-08.html#81)
- 8.2 Iptables
  - [8.2.1 策略与规则链](https://www.linuxprobe.com/basic-learning-08.html#821)
  - [8.2.2 基本的命令参数](https://www.linuxprobe.com/basic-learning-08.html#822)
- 8.3 Firewalld
  - [8.3.1 终端管理工具](https://www.linuxprobe.com/basic-learning-08.html#831)
  - [8.3.2 图形管理工具](https://www.linuxprobe.com/basic-learning-08.html#832)
- [8.4 服务的访问控制列表](https://www.linuxprobe.com/basic-learning-08.html#84)
- [8.5 Cockpit驾驶舱管理工具](https://www.linuxprobe.com/basic-learning-08.html#85_Cockpit)

##### **8.1 防火墙管理工具**

众所周知，相较于企业内网，外部的公网环境更加恶劣，罪恶丛生。在公网与企业内网之间充当保护屏障的防火墙虽然有软件或硬件之分，但主要功能都是依据策略对穿越防火墙自身的流量进行过滤。就像家里安装的防盗门一样，目的是保护亲人和财产安全，如图8-1所示。防火墙策略可以基于流量的源目地址、端口号、协议、应用等信息来定制，然后防火墙使用预先定制的策略规则监控出入的流量，若流量与某一条策略规则相匹配，则执行相应的处理，反之则丢弃。这样一来，就能够保证仅有合法的流量在企业内网和外部公网之间流动了。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E9%98%B2%E7%81%AB%E5%A2%99%E4%BD%9C%E4%B8%BA%E5%85%AC%E7%BD%91%E4%B8%8E%E5%86%85%E7%BD%91%E4%B9%8B%E9%97%B4%E7%9A%84%E4%BF%9D%E6%8A%A4%E5%B1%8F%E9%9A%9C-2.jpg)

图8-1 防火墙作为公网与内网之间的保护屏障

从RHEL 7系统开始，firewalld防火墙正式取代了iptables防火墙。对于接触[Linux](https://www.linuxprobe.com/)系统比较早或学习过RHEL 5/6系统的读者来说，当他们发现曾经掌握的知识在RHEL 7/8中不再适用，需要全新学习firewalld时，难免会有抵触心理。其实，iptables与firewalld都不是真正的防火墙，它们都只是用来定义防火墙策略的防火墙管理工具而已，或者说，它们只是一种服务。iptables服务会把配置好的防火墙策略交由内核层面的netfilter网络过滤器来处理，而firewalld服务则是把配置好的防火墙策略交由内核层面的nftables包过滤框架来处理。换句话说，当前在Linux系统中其实存在多个防火墙管理工具，旨在方便运维人员管理Linux系统中的防火墙策略，我们只需要配置妥当其中的一个就足够了。

虽然这些工具各有优劣，但它们在防火墙策略的配置思路上是保持一致的。大家甚至可以不用完全掌握本章介绍的内容，只要在这多个防火墙管理工具中任选一款并将其学透，就足以满足日常的工作需求了。

##### **8.2 Iptables**

在早期的Linux系统中，默认使用的是iptables防火墙管理服务来配置防火墙。尽管新型的firewalld防火墙管理服务已经被投入使用多年，但是大量的企业在生产环境中依然出于各种原因而继续使用iptables。考虑到iptables在当前生产环境中还具有顽强的生命力，以及为了使大家在求职面试过程中被问到iptables的相关知识时能胸有成竹，[刘遄](https://www.linuxprobe.com/)老师觉得还是有必要在本书中好好地讲解一下这项技术。更何况前文也提到，各个防火墙管理工具的配置思路是一致的，在掌握了iptables后再学习其他防火墙管理工具时，也有借鉴意义。

###### **8.2.1 策略与规则链**

防火墙会从上至下的顺序来读取配置的策略规则，在找到匹配项后就立即结束匹配工作并去执行匹配项中定义的行为（即放行或阻止）。如果在读取完所有的策略规则之后没有匹配项，就去执行默认的策略。一般而言，防火墙策略规则的设置有两种：一种是“通”（即放行），一种是“堵”（即阻止）。当防火墙的默认策略为拒绝时（堵），就要设置允许规则（通），否则谁都进不来；如果防火墙的默认策略为允许时，就要设置拒绝规则，否则谁都能进来，防火墙也就失去了防范的作用。

iptables服务把用于处理或过滤流量的策略条目称之为规则，多条规则可以组成一个规则链，而规则链则依据数据包处理位置的不同进行分类，具体如下：

> 在进行路由选择前处理数据包（PREROUTING）；
>
> 处理流入的数据包（INPUT）；
>
> 处理流出的数据包（OUTPUT）；
>
> 处理转发的数据包（FORWARD）；
>
> 在进行路由选择后处理数据包（POSTROUTING）。

一般来说，从内网向外网发送的流量一般都是可控且良性的，因此使用最多的就是INPUT规则链，该规则链可以增大黑客人员从外网入侵内网的难度。

比如在您居住的社区内，物业管理公司有两条规定：禁止小商小贩进入社区；各种车辆在进入社区时都要登记。显而易见，这两条规定应该是用于社区的正门的（流量必须经过的地方），而不是每家每户的防盗门上。根据前面提到的防火墙策略的匹配顺序，可能会存在多种情况。比如，来访人员是小商小贩，则直接会被物业公司的保安拒之门外，也就无需再对车辆进行登记。如果来访人员乘坐一辆汽车进入社区正门，则“禁止小商小贩进入社区”的第一条规则就没有被匹配到，因此按照顺序匹配第二条策略，即需要对车辆进行登记。如果是社区居民要进入正门，则这两条规定都不会匹配到，因此会执行默认的放行策略。

但是，仅有策略规则还不能保证社区的安全，保安还应该知道采用什么样的动作来处理这些匹配的流量，比如“允许”、“拒绝”、“登记”、“不理它”。这些动作对应到iptables服务的术语中分别是ACCEPT（允许流量通过）、REJECT（拒绝流量通过）、LOG（记录日志信息）、DROP（拒绝流量通过）。“允许流量通过”和“记录日志信息”都比较好理解，这里需要着重讲解的是REJECT和DROP的不同点。就DROP来说，它是直接将流量丢弃而且不响应；REJECT则会在拒绝流量后再回复一条“信息已经收到，但是被扔掉了”信息，从而让流量发送方清晰地看到数据被拒绝的响应信息。

来举一个例子，让各位读者更直观地理解这两个拒绝动作的不同之处。比如有一天您正在家里看电视，突然听到有人敲门，透过防盗门的猫眼一看是推销商品的，便会在不需要的情况下开门并拒绝他们（REJECT）。但如果看到的是债主带了十几个小弟来讨债，此时不仅要拒绝开门，还要默不作声，伪装成自己不在家的样子（DROP）。

[红帽](https://www.linuxprobe.com/)考试中必须用REJECT进行拒绝，明确让判分[脚本](https://www.linuxcool.com/)得到反应才有分值。而工作中更多建议用DROP进行拒绝，对隐藏服务器运行状态有好处。

当把Linux系统中的防火墙策略设置为REJECT拒绝动作后，流量发送方会看到端口不可达的响应：

```
[root@linuxprobe ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
From 192.168.10.10 icmp_seq=1 Destination Port Unreachable
From 192.168.10.10 icmp_seq=2 Destination Port Unreachable
From 192.168.10.10 icmp_seq=3 Destination Port Unreachable
From 192.168.10.10 icmp_seq=4 Destination Port Unreachable
--- 192.168.10.10 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3002ms
```

而把Linux系统中的防火墙策略修改成DROP拒绝动作后，流量发送方会看到响应超时的提醒。但是流量发送方无法判断流量是被拒绝，还是接收方主机当前不在线：

```
[root@linuxprobe ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.

--- 192.168.10.10 ping statistics ---
4 packets transmitted, 0 received, 100% packet loss, time 3000ms
```

###### **8.2.2 基本的[命令](https://www.linuxcool.com/)参数**

iptables是一款基于[命令](https://www.linuxcool.com/)行的防火墙策略管理工具，具有大量参数，学习难度较大。好在对于日常的防火墙策略配置来讲，大家无需深入了解诸如“四表五链”的理论概念，只需要掌握常用的参数并做到灵活搭配即可，这就足以应对日常工作了。

根据OSI七层模型的定义，iptables属于是数据链路层的服务，所以可以根据流量的源地址、目的地址、传输协议、服务类型等信息进行匹配，一旦匹配成功，iptables就会根据策略规则所预设的动作来处理这些流量。另外，再次提醒一下，防火墙策略规则的匹配顺序是从上至下的，因此要把较为严格、优先级较高的策略规则放到前面，以免发生错误。表8-1总结归纳了常用的iptables命令参数。再次强调，无需死记硬背这些参数，只需借助下面的实验来理解掌握即可。

表8-1                      iptables中常用的参数以及作用

| 参数        | 作用                                         |
| ----------- | -------------------------------------------- |
| -P          | 设置默认策略                                 |
| -F          | 清空规则链                                   |
| -L          | 查看规则链                                   |
| -A          | 在规则链的末尾加入新规则                     |
| -I num      | 在规则链的头部加入新规则                     |
| -D num      | 删除某一条规则                               |
| -s          | 匹配来源地址IP/MASK，加叹号“!”表示除这个IP外 |
| -d          | 匹配目标地址                                 |
| -i 网卡名称 | 匹配从这块网卡流入的数据                     |
| -o 网卡名称 | 匹配从这块网卡流出的数据                     |
| -p          | 匹配协议，如TCP、UDP、ICMP                   |
| --dport num | 匹配目标端口号                               |
| --sport num | 匹配来源端口号                               |



**1．在iptables命令后添加-L参数查看已有的防火墙规则链：**

```
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:domain
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:domain
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootps
ACCEPT     tcp  --  anywhere             anywhere             tcp dpt:bootps

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  anywhere             192.168.122.0/24     ctstate RELATED,ESTABLISHED
ACCEPT     all  --  192.168.122.0/24     anywhere            
ACCEPT     all  --  anywhere             anywhere            
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable
REJECT     all  --  anywhere             anywhere             reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     udp  --  anywhere             anywhere             udp dpt:bootpc
```

**2．在iptables命令后添加-F参数清空已有的防火墙规则链：**

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

**3．把INPUT规则链的默认策略设置为拒绝：**

```
[root@linuxprobe ~]# iptables -P INPUT DROP
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy DROP)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

如前面所提到的防火墙策略设置无非有两种方式，一种是“通”，一种是“堵”，当把INPUT链设置为默认拒绝后，就要往里面写入允许策略了，否则所有流入的数据包都会被默认拒绝掉，同学们需要留意规则链的默认策略拒绝动作只能是DROP，而不能是REJECT。

**4．向INPUT链中添加允许ICMP流量进入的策略规则：**

在日常运维工作中，经常会使用ping命令来检查对方主机是否在线，而向防火墙的INPUT规则链中添加一条允许ICMP流量进入的策略规则就默认允许了这种ping命令检测行为。

```
[root@linuxprobe ~]# iptables -I INPUT -p icmp -j ACCEPT
[root@linuxprobe ~]# ping -c 4 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.154 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.041 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.038 ms
64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.046 ms

--- 192.168.10.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 104ms
rtt min/avg/max/mdev = 0.038/0.069/0.154/0.049 ms
```

**5．删除INPUT规则链中刚刚加入的那条策略（允许ICMP流量），并把默认策略设置为允许。**

使用-F会清空已有的所有防火墙策略，使用-D参数可以指定删除某一条策略，更加安全和准确。

```
[root@linuxprobe ~]# iptables -D INPUT 1
[root@linuxprobe ~]# iptables -P INPUT ACCEPT
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

**6．将INPUT规则链设置为只允许指定网段的主机访问本机的22端口，拒绝来自其他所有主机的流量。**

要对某台主机进行匹配，则直接写IP地址即可，如需对网段进行匹配，需要写子网掩码的格式，例如：192.168.10.0/24的形式。

```
[root@linuxprobe ~]# iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j ACCEPT
[root@linuxprobe ~]# iptables -A INPUT -p tcp --dport 22 -j REJECT
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination
 ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh 
 REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

再次重申，防火墙策略规则是按照从上到下的顺序匹配的，因此一定要把允许动作放到拒绝动作前面，否则所有的流量就将被拒绝掉，从而导致任何主机都无法访问我们的服务。另外，这里提到的22号端口是ssh服务使用的（有关ssh服务，请见下一章），先在这里挖坑，等大家学完第9章后可再验证这个实验的效果。

在设置完上述INPUT规则链之后，使用IP地址在192.168.10.0/24网段内的主机访问服务器（即前面提到的设置了INPUT规则链的主机）的22端口，效果如下：

```
[root@Client A ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处输入服务器密码
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Jan 20 16:30:28 2021 from 192.168.10.1
```

然后，再使用IP地址在192.168.20.0/24网段内的主机访问服务器的22端口（虽网段不同，但已确认可以相互通信），效果如下，就会提示连接请求被拒绝了（Connection failed）：

```
[root@Client B ~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection failed.
```

**7．向INPUT规则链中添加拒绝所有人访问本机12345端口的策略规则**：

```
[root@linuxprobe ~]# iptables -I INPUT -p tcp --dport 12345 -j REJECT
[root@linuxprobe ~]# iptables -I INPUT -p udp --dport 12345 -j REJECT
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination 
 REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
 REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
 ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
 REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

**8．向INPUT规则链中添加拒绝192.168.10.5主机访问本机80端口（Web服务）的策略规则**：

```
[root@linuxprobe ~]# iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination 
 REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable
 REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
 REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
 ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
 REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

**9．向INPUT规则链中添加拒绝所有主机访问本机1000**～**1024****端口的策略规则**：

刚刚添加防火墙策略时用的是-I参数，它默认会把规则添加到最上面位置，优先级是最高的。如果工作中需要添加一条最后“兜底”的规则 ，那就用-A参数吧，效果差别很大呦：

```
[root@linuxprobe ~]# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT
[root@linuxprobe ~]# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT
[root@linuxprobe ~]# iptables -L
Chain INPUT (policy ACCEPT)
target prot opt source destination 
 REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable
 REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable
 REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable
 ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh
 REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable
 REJECT tcp -- anywhere anywhere tcp dpts:cadlock2:1024 reject-with icmp-port-unreachable
 REJECT udp -- anywhere anywhere udp dpts:cadlock2:1024 reject-with icmp-port-unreachable
………………省略部分输出信息………………
```

有关iptables命令的知识讲解到此就结束了，大家是不是意犹未尽？考虑到Linux防火墙的发展趋势，大家只要能把上面的实例吸收消化，就可以完全搞定日常的iptables配置工作了。但是请特别注意，使用iptables命令配置的防火墙规则默认会在系统下一次重启时失效，如果想让配置的防火墙策略永久生效，还要执行保存命令：

```
[root@linuxprobe ~]# iptables-save 
# Generated by xtables-save v1.8.2 on Wed Jan 20 16:56:27 2021
………………省略部分输出信息………………
```

对了，如果公司服务器是5/6/7版本的话，对应的保存命令应该是：

```
[root@linuxprobe ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables: [ OK ]
```

##### **8.3 Firewalld**

RHEL 8系统中集成了多款防火墙管理工具，其中firewalld（Dynamic Firewall Manager of Linux systems）服务是默认的防火墙配置管理工具，也叫Linux系统的动态防火墙管理器，它拥有基于CLI（命令行界面）和基于GUI（图形用户界面）的两种管理方式。

相较于传统的防火墙管理配置工具，firewalld支持动态更新技术并加入了区域（zone）的概念。简单来说，区域就是firewalld预先准备了几套防火墙策略集合（策略模板），用户可以根据生产场景的不同而选择合适的策略集合，从而实现防火墙策略之间的快速切换。例如，我们有一台笔记本电脑，每天都要在办公室、咖啡厅和家里使用。按常理来讲，这三者的安全性按照由高到低的顺序来排列，应该是家庭、公司办公室、咖啡厅。当前，希望为这台笔记本电脑指定如下防火墙策略规则：在家中允许访问所有服务；在办公室内仅允许访问文件共享服务；在咖啡厅仅允许上网浏览。在以往，我们需要频繁地手动设置防火墙策略规则，而现在只需要预设好区域集合，然后只需轻点鼠标就可以自动切换了，从而极大地提升了防火墙策略的应用效率。firewalld中常见的区域名称（默认为public）以及相应的策略规则如表8-2所示。

表8-2                   firewalld中常用的区域名称及策略规则

| 区域     | 默认规则策略                                                 |
| -------- | ------------------------------------------------------------ |
| trusted  | 允许所有的数据包                                             |
| home     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量 |
| internal | 等同于home区域                                               |
| work     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量 |
| public   | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量 |
| external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| dmz      | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量 |
| block    | 拒绝流入的流量，除非与流出的流量相关                         |
| drop     | 拒绝流入的流量，除非与流出的流量相关                         |



###### **8.3.1 终端管理工具**

第2章在讲解Linux命令时曾经看到过，命令行终端是一种极富效率的工作方式，firewall-cmd是firewalld防火墙配置管理工具的CLI命令行界面版本。它的参数一般都是以“长格式”来提供的，大家不要一听到长格式就头大，因为RHEL 8系统支持部分命令的参数补齐，其中就包含这条命令（很酷吧）。也就是说，现在除了能用Tab键自动补齐命令或文件名等内容之外，还可以用Tab键来补齐表8-3中所示的长格式参数了（这太棒了）。

表8-3                  firewall-cmd命令中使用的参数以及作用

| 参数                          | 作用                                                 |
| ----------------------------- | ---------------------------------------------------- |
| --get-default-zone            | 查询默认的区域名称                                   |
| --set-default-zone=<区域名称> | 设置默认的区域，使其永久生效                         |
| --get-zones                   | 显示可用的区域                                       |
| --get-services                | 显示预先定义的服务                                   |
| --get-active-zones            | 显示当前正在使用的区域与网卡名称                     |
| --add-source=                 | 将源自此IP或子网的流量导向指定的区域                 |
| --remove-source=              | 不再将源自此IP或子网的流量导向某个指定区域           |
| --add-interface=<网卡名称>    | 将源自该网卡的所有流量都导向某个指定区域             |
| --change-interface=<网卡名称> | 将某个网卡与区域进行关联                             |
| --list-all                    | 显示当前区域的网卡配置参数、资源、端口以及服务等信息 |
| --list-all-zones              | 显示所有区域的网卡配置参数、资源、端口以及服务等信息 |
| --add-service=<服务名>        | 设置默认区域允许该服务的流量                         |
| --add-port=<端口号/协议>      | 设置默认区域允许该端口的流量                         |
| --remove-service=<服务名>     | 设置默认区域不再允许该服务的流量                     |
| --remove-port=<端口号/协议>   | 设置默认区域不再允许该端口的流量                     |
| --reload                      | 让“永久生效”的配置规则立即生效，并覆盖当前的配置规则 |
| --panic-on                    | 开启应急状况模式                                     |
| --panic-off                   | 关闭应急状况模式                                     |



与Linux系统中其他的防火墙策略配置工具一样，使用firewalld配置的防火墙策略默认为运行时（Runtime）模式，又称为当前生效模式，而且随着系统的重启会失效。如果想让配置策略一直存在，就需要使用永久（Permanent）模式了，方法就是在用firewall-cmd命令正常设置防火墙策略时添加--permanent参数，这样配置的防火墙策略就可以永久生效了。但是，永久生效模式有一个“不近人情”的特点，就是使用它设置的策略只有在系统重启之后才能自动生效。如果想让配置的策略立即生效，需要手动执行firewall-cmd --reload命令。

Runtime：当前立即生效，重启后失效。

Permanent：当前不生效，重启后生效。

接下来的实验都很简单，但是提醒大家一定要仔细查看书中使用的是Runtime模式还是Permanent模式。如果不关注这个细节，就算是正确配置了防火墙策略，也可能无法达到预期的效果。

**1．查看firewalld服务当前所使用的区域：**

这是一步非常重要的操作，在配置防火墙策略前，必须查看当前生效的是那个区域，否则配置的防火墙策略将不会立即生效。

```
[root@linuxprobe ~]# firewall-cmd --get-default-zone
public
```

**2．查询指定网卡在firewalld服务中绑定的区域：**

生产环境中，服务器大多不止有一块网卡，假如作为网关的服务器有两块网卡，一块对公网，一块对内网，那么这两块网卡对审查流量时的策略肯定也是不一致的。于是可以针对不同的来源方向的网卡绑定上不同的区域，实现对防火墙策略的灵活管控。

```
[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=ens160
public
```

**3．把网卡默认区域修改为external，并在系统重启后生效：**

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=external --change-interface=ens160
The interface is under control of NetworkManager, setting zone to 'external'.
success
[root@linuxprobe ~]# firewall-cmd --permanent --get-zone-of-interface=ens160
external
```

**4．把firewalld服务的默认区域设置为public：**

默认区域也叫全局配置，指的是对所有网卡都生效的配置，优先级较低。我们可以看到当前默认区域为public，而ens160网卡的区域为external，这种情况便是以网卡的区域名称为准。

通俗来说默认区域就是一种通用的政策，例如食堂为所有人准备了一次性餐具，而如果是环保主义者，自己携带了筷子勺子，那么以您实际情况为准，如果没有带的话才用餐厅统一提供的。

```
[root@linuxprobe ~]# firewall-cmd --set-default-zone=public
Warning: ZONE_ALREADY_SET: public
success
[root@linuxprobe ~]# firewall-cmd --get-default-zone 
public
[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=ens160
externa
```

**5．启动和关闭firewalld防火墙服务的应急状况模式：**

如果想一秒钟阻断一切网络连接有什么好办法呢？大家潜意识一定马上会说：“拔掉网线！”，确实是个物理级别的高招~但如果人在北京，服务器却在异地呢？这时就用panic紧急模式吧。使用“--panic-on”参数会立即切断一切网络连接，而“--panic-off”则是恢复网络连接。切记紧急模式会切断一切网络连接，包括ping都是不通的，因此在远程管理服务器时，在按下回车键前一定要三思而后行。

```
[root@linuxprobe ~]# firewall-cmd --panic-on
success
[root@linuxprobe ~]# firewall-cmd --panic-off
success
```

**6．查询ssh和https协议的流量是否允许放行：**

虽然工作中可以不使用“--zone”参数指定区域名称，firewall-cmd命令会自动依据默认区域进行查询，减少用户输入量。但如果默认区域与网卡所绑定的不一致时，就会发生冲突的情况，因此规范写法是一定要加的。

```
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=ssh
yes
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=https
no
```

**7．把https协议的流量设置为永久允许放行，并立即生效：**

默认情况下做的修改都属于Runtime模式，即当前生效而重启后失效，在工作和考试中尽量避免使用。而使用“--permanent”参数时则当前不会立即看到效果，而是重启或重新加载后方可生效。于是，当添加了允许放行https协议的流量后，查询当前模式策略依然是不允许：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=https
success
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=https
no
```

不想重启服务器的话，我们就用“--reload”参数吧：

```
[root@linuxprobe ~]# firewall-cmd --reload
success
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=https
yes
```

**8．把http协议的流量设置为永久拒绝，并立即生效：**

由于默认来讲http协议流量就没有被允许，所以会有：“Warning: NOT_ENABLED: http”这样的提示信息，对实际操作没有影响。

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --remove-service=http
Warning: NOT_ENABLED: http
success
[root@linuxprobe ~]# firewall-cmd --reload 
success
```

**9．把访问8080和8081端口的流量策略设置为允许，但仅限当前生效：**

```
[root@linuxprobe ~]# firewall-cmd --zone=public --add-port=8080-8081/tcp
success
[root@linuxprobe ~]# firewall-cmd --zone=public --list-ports
8080-8081/tcp
```

**10．把原本访问本机888端口的流量转发到22端口，要且求当前和长期均有效：**

下一章节咱们会讲到的ssh远程控制协议是基于22/tcp端口号传输控制指令的，如果想让用户通过其他端口号也能访问到ssh服务，就可以试试端口号转发技术了。通过这项技术，新的端口号收到了用户请求后会自动转发到原本服务的端口上，使得用户能够通过新的端口号访问到原本的服务。

来举个例子帮助同学们理解，假如小强是电子厂的工人，喜欢上了三号流水线的工人小花，但不好意思表白，于是写了一封情书交给了门卫张大爷，再由张大爷转交给小花。信息的传输从小强到小花，变成了小强到张大爷再到小花，信息也能顺利送达。

使用firewall-cmd命令实现端口号转发的格式有点长，为同学们总结好了：

> firewall-cmd --permanent --zone=**<区域>** --add-forward-port=port=<源端口号>:proto=**<协议>**:toport=**<目标端口号>**:toaddr=**<目标IP地址>**

命令中的目标IP地址一般写服务器本机：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

在客户端使用ssh命令尝试访问192.168.10.10主机的888端口，顺利成功：

```
[root@client A ~]# ssh -p 888 192.168.10.10
The authenticity of host '[192.168.10.10]:888 ([192.168.10.10]:888)' can't be established.
ECDSA key fingerprint is b8:25:88:89:5c:05:b6:dd:ef:76:63:ff:1a:54:02:1a.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.10.10]:888' (ECDSA) to the list of known hosts.
root@192.168.10.10's password:此处输入远程root管理员的密码
Last login: Sun Jul 19 21:43:48 2021 from 192.168.10.10
```

**11．富规则的设置：**

富规则也叫复规则。表示更细致、更详细的防火墙策略配置，它可以针对系统服务、端口号、源地址和目标地址等诸多信息进行更有针对性的策略配置。它的优先级在所有的防火墙策略中也是最高的。比如，我们可以在firewalld服务中配置一条富规则，使其拒绝192.168.10.0/24网段的所有用户访问本机的ssh服务（22端口）：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

在客户端使用ssh命令尝试访问192.168.10.10主机的ssh服务（22端口）：

```
[root@client A ~]# ssh 192.168.10.10
Connecting to 192.168.10.10:22...
Could not connect to '192.168.10.10' (port 22): Connection faile
d.
```

例如“一个男人”和“一个高个子、大眼睛、右面脸有个酒窝的男人”相比，很明显后者的描述更加精准，减少了错误匹配的概率，应该优先匹配。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

###### **8.3.2 图形管理工具**

在各种版本的Linux系统中，几乎没有能让刘遄老师欣慰并推荐的图形化工具，但是firewall-config做到了。它是firewalld防火墙配置管理工具的GUI图形用户界面版本，几乎可以实现所有以命令行来执行的操作。毫不夸张的说，即使读者没有扎实的Linux命令基础，也完全可以通过它来妥善配置RHEL 8中的防火墙策略。但在系统默认情况下firewall-config命令是没有被提供的，我们需要自行用dnf命令进行安装，所以需要先配置下软件仓库才行。

首先将虚拟机的CD/DVD光盘选项设置为“使用ISO镜像/映像文件”，并选择已经下载好的系统镜像，如图8-2所示。

> **随书配套的软件资源请在这里下载：**https://www.linuxprobe.com/tools/
>
> **RedHatEnterpriseLinux [RHEL]8.0——红帽操作系统（必需）：**
>
> 由开源软件及全球服务性系统开发商红帽公司出品，最稳定出色的Linux操作系统。
>
> **培训课程介绍视频：https://www.linuxprobe.com/training**

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%8C%82%E8%BD%BD%E7%B3%BB%E7%BB%9F%E9%95%9C%E5%83%8F.png)

8-2  将虚拟机的光盘设备指向ISO镜像

系统镜像下载后就是iso格式结尾的文件，选中即可，无需解压。

然后，把光盘设备中的系统镜像挂载到/media/cdrom目录。

```
[root@linuxprobe ~]# mkdir -p /media/cdrom
[root@linuxprobe ~]# mount /dev/cdrom /media/cdrom
mount: /media/cdrom: WARNING: device write-protected, mounted read-only.
```

为了能够一直为用户提供服务，更加严谨的做法是写入到/etc/fstab文件中，保证万无一失：

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /                  xfs       defaults        0 0
UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot              xfs       defaults        0 0
/dev/mapper/rhel-swap                       swap               swap      defaults        0 0
/dev/cdrom                                  /media/cdrom       iso9660   defaults        0 0 
```

最后，使用Vim文本编辑器创建软件仓库的配置文件。与过往版本的系统不同，RHEL8需要配置两个软件仓库，即[BaseOS]与[AppStream]缺一不可。下述命令中用到的具体参数的含义，可参考4.1.4小节。

```
[root@linuxprobe ~]# vim /etc/yum.repos.d/rhel8.repo
[BaseOS]
name=BaseOS。
baseurl=file:///media/cdrom/BaseOS
enabled=1
gpgcheck=0
[AppStream]
name=AppStream
baseurl=file:///media/cdrom/AppStream
enabled=1
gpgcheck=0
```

正确的配置软件仓库文件后，就可以开始用yum或dnf命令安装软件了。这两个命令在实际操作中除了名字不同外，执行方法完全一致，可随时用yum替代dnf命令。安装firewalld图形化界面工具：

```
[root@linuxprobe ~]# dnf install firewall-config
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                       3.1 MB/s | 3.2 kB     00:00    
BaseOS                                          2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
================================================================================
 Package                Arch          Version            Repository        Size
================================================================================
Installing:
 firewall-config        noarch        0.6.3-7.el8        AppStream        157 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 157 k
Installed size: 1.1 M
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : firewall-config-0.6.3-7.el8.noarch                     1/1 
  Running scriptlet: firewall-config-0.6.3-7.el8.noarch                     1/1 
  Verifying        : firewall-config-0.6.3-7.el8.noarch                     1/1 
Installed products updated.

Installed:
  firewall-config-0.6.3-7.el8.noarch                                            

Complete!
```

安装成功后，firewall-config工具的界面如图8-3所示，其功能具体如下：

> **1：**选择运行时（Runtime）或永久（Permanent）模式的配置
>
> **2**：可选的策略集合区域列表
>
> **3**：常用的系统服务列表
>
> **4：**主机地址的黑白名单
>
> **5**：当前正在使用的区域
>
> **6**：管理当前被选中区域中的服务
>
> **7**：管理当前被选中区域中的端口
>
> **8：**设置允许被访问的协议
>
> **9：**设置允许被访问的端口
>
> **10**：开启或关闭SNAT源地址转换协议技术
>
> **11**：设置端口转发策略
>
> **12**：控制请求icmp服务的流量
>
> **13**：管理防火墙的富规则
>
> **14**：被选中区域的服务，若勾选了相应服务前面的复选框，则表示允许与之相关的流量
>
> **15**：firewall-config工具的运行状态

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/firewall-config%E7%9A%84%E5%9B%BE%E5%BD%A2%E7%95%8C%E9%9D%A2-1.jpg)

图8-3 firewall-config的图形界面

除上图中所列出的功能，还有用于将网卡与区域绑定的“Interfaces”和用于将IP地址与区域绑定的“Sources”选项。另外再啰嗦一句。在使用firewall-config工具配置完防火墙策略之后，无须进行二次确认，因为只要有修改内容，它就自动进行保存。

**下面进行动手实践环节**

先将当前区域中请求http服务的流量设置为允许放行，但仅限当前生效。具体配置如图8-4所示。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%94%BE%E8%A1%8C%E8%AF%B7%E6%B1%82http%E6%9C%8D%E5%8A%A1%E7%9A%84%E6%B5%81%E9%87%8F-1024x591.png)

图8-4 允许放行请求http服务的流量

尝试添加一条防火墙策略规则，使其放行访问8080～8088端口（TCP协议）的流量，并将其设置为永久生效，以达到系统重启后防火墙策略依然生效的目的。在按照图8-5所示的界面配置完毕之后，还需要在Options菜单中单击Reload Firewalld命令，让配置的防火墙策略立即生效（见图8-6）。这与在命令行中执行“--reload”参数的效果一样。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%94%BE%E8%A1%8C%E8%AE%BF%E9%97%AE8080%EF%BD%9E8088%E7%AB%AF%E5%8F%A3%E7%9A%84%E6%B5%81%E9%87%8F-1024x589.png)

图8-5 放行访问8080～8088端口的流量

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%A9%E9%85%8D%E7%BD%AE%E7%9A%84%E9%98%B2%E7%81%AB%E5%A2%99%E7%AD%96%E7%95%A5%E8%A7%84%E5%88%99%E7%AB%8B%E5%8D%B3%E7%94%9F%E6%95%88-1024x590.png)

图8-6 让配置的防火墙策略规则立即生效

前面在讲解firewall-config工具的功能时，曾经提到了SNAT（Source Network Address Translation）源网络地址转换技术。SNAT是一种为了解决IP地址匮乏而设计的技术，它可以使得多个内网中的用户通过同一个外网IP接入Internet。该技术的应用非常广泛，甚至可以说我们每天都在使用，只不过没有察觉到罢了。比如，当通过家中的网关设备（无线路由器）访问本书配套站点[www.linuxprobe.com](https://www.linuxprobe.com/)时，就用到了SNAT技术。

请读者们看一下在网络中不使用SNAT技术（见图8-7）和使用SNAT技术（见图8-8）时的情况。在图8-7所示的局域网中有多台PC，如果网关服务器没有应用SNAT技术，则互联网中的网站服务器在收到PC的请求数据包，并回送响应数据包时，将无法在网络中找到这个私有网络的IP地址，所以PC也就收不到响应数据包了。在图8-8所示的局域网中，由于网关服务器应用了SNAT技术，所以互联网中的网站服务器会将响应数据包发给网关服务器，再由后者转发给局域网中的PC。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2015/03/%E6%9C%AA%E7%94%A8SNAT1.png)

图8-7 没有使用SNAT技术的网络

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2015/03/%E4%BD%BF%E7%94%A8SNAT1.png)

图8-8 使用SNAT技术处理过的网络

使用iptables命令实现SNAT技术是一件很麻烦的事情，但是在firewall-config中却是小菜一碟了。用户只需按照图8-9进行配置，并选中Masquerade zone复选框，就自动开启了SNAT技术。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%BC%80%E5%90%AF%E9%98%B2%E7%81%AB%E5%A2%99%E7%9A%84SNAT%E6%8A%80%E6%9C%AF.png)

图8-9 开启防火墙的SNAT技术

为了让大家直观查看不同工具在实现相同功能的区别，这里使用firewall-config工具重新演示了前面使用firewall-cmd来配置防火墙策略规则，将本机888端口的流量转发到22端口，且要求当前和长期均有效，具体如图8-10和图8-11所示。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E9%85%8D%E7%BD%AE%E6%9C%AC%E5%9C%B0%E7%9A%84%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91-1024x590.png)

图8-10 配置本地的端口转发

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%A9%E9%98%B2%E7%81%AB%E5%A2%99%E6%95%88%E7%AD%96%E7%95%A5%E8%A7%84%E5%88%99%E7%AB%8B%E5%8D%B3%E7%94%9F%E6%95%88-1024x591.png)

图8-11 让防火墙效策略规则立即生效

用命令配置富规则可真辛苦，好在有了图形化界面的工具。设置让192.168.10.20主机访问到本机的1234端口号，如图8-12所示。其中Element选项能够根据服务名称、端口号、协议等信息进行匹配；Source与Destination选项后的inverted参数代表反选功能，勾选后代表对已填写信息进行反选，即除了填写信息以外的主机地址；Log日志选项选中后不仅会被记录到日志文件中，还可以选择重要级别，便于后续的筛查。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E9%85%8D%E7%BD%AE%E9%98%B2%E7%81%AB%E5%A2%99%E5%AF%8C%E8%A7%84%E5%88%99%E7%AD%96%E7%95%A5-1-1024x590.png)

图8-12 配置防火墙富规则策略

如果生产环境中的服务器有多块网卡在同时提供服务（这种情况很常见），则对内网和对外网提供服务的网卡要选择的防火墙策略区域也是不一样的。也就是说，可以把网卡与防火墙策略区域进行绑定，见图8-13和图8-14，这样就可以使用不同的防火墙区域策略，对源自不同网卡的流量进行针对性的监控，效果会更好。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%8A%8A%E7%BD%91%E5%8D%A1%E4%B8%8E%E9%98%B2%E7%81%AB%E5%A2%99%E7%AD%96%E7%95%A5%E5%8C%BA%E5%9F%9F%E8%BF%9B%E8%A1%8C%E7%BB%91%E5%AE%9A-1024x591.png)

图8-13 把网卡与防火墙策略区域进行绑定![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%BD%91%E5%8D%A1%E4%B8%8E%E7%AD%96%E7%95%A5%E5%8C%BA%E5%9F%9F%E7%BB%91%E5%AE%9A%E5%AE%8C%E6%88%90-1024x590.png)

图8-14 网卡与策略区域绑定完成

最后再提一句，firewall-config工具真的非常实用，很多原本复杂的长命令被用图形化按钮替代，设置规则也简单明了，足以应对日常工作。所以再次向大家强调配置防火墙策略的原则—只要能实现所需的功能，用什么工具依据自己习惯就好。

##### **8.4 服务的访问控制列表**

TCP Wrappers是RHEL 6/7系统中默认启用的一款流量监控程序，它能够根据来访主机的地址与本机的目标服务程序作出允许或拒绝的操作，当前8版本中已经被Firewalld正式替代。换句话说，Linux系统中其实有两个层面的防火墙，第一种是前面讲到的基于TCP/IP协议的流量过滤工具，而TCP Wrappers服务则是能允许或禁止Linux系统提供服务的防火墙，从而在更高层面保护了Linux系统的安全运行。

TCP Wrappers服务的防火墙策略由两个控制列表文件所控制，用户可以编辑允许控制列表文件来放行对服务的请求流量，也可以编辑拒绝控制列表文件来阻止对服务的请求流量。控制列表文件修改后会立即生效，系统将会先检查允许控制列表文件（/etc/hosts.allow），如果匹配到相应的允许策略则放行流量；如果没有匹配，则去进一步匹配拒绝控制列表文件（/etc/hosts.deny），若找到匹配项则拒绝该流量。如果这两个文件全都没有匹配到，则默认放行流量。

由于8版本中已不再支持TCP Wrappers服务程序，因此接下来的操作我们选择在一台老版本服务器上进行实验。控制列表文件配置起来并不复杂，常用的参数如表8-4所示。

表8-4              TCP Wrappers服务的控制列表文件中常用的参数

| 客户端类型     | 示例                       | 满足示例的客户端列表               |
| -------------- | -------------------------- | ---------------------------------- |
| 单一主机       | 192.168.10.10              | IP地址为192.168.10.10的主机        |
| 指定网段       | 192.168.10.                | IP段为192.168.10.0/24的主机        |
| 指定网段       | 192.168.10.0/255.255.255.0 | IP段为192.168.10.0/24的主机        |
| 指定DNS后缀    | .linuxprobe.com            | 所有DNS后缀为.linuxprobe.com的主机 |
| 指定主机名称   | www.linuxprobe.com         | 主机名称为www.linuxprobe.com的主机 |
| 指定所有客户端 | ALL                        | 所有主机全部包括在内               |



在配置TCP Wrappers服务时需要遵循两个原则：

1. 编写拒绝策略规则时，填写的是服务名称，而非协议名称；
2. 建议先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果。

下面编写拒绝策略规则文件，禁止访问本机sshd服务的所有流量（无需修改/etc/hosts.deny文件中原有的注释信息）：

```
[root@linuxprobe ~]# vim /etc/hosts.deny
#
# hosts.deny This file contains access rules which are used to
# deny connections to network services that either use
# the tcp_wrappers library or that have been
# started through a tcp_wrappers-enabled xinetd.
#
# The rules in this file can also be set up in
# /etc/hosts.allow with a 'deny' option instead.
#
# See 'man 5 hosts_options' and 'man 5 hosts_access'
# for information on rule syntax.
# See 'man tcpd' for information on tcp_wrappers
sshd:*
[root@linuxprobe ~]# ssh 192.168.10.10
ssh_exchange_identification: read: Connection reset by peer
```

接下来，在允许策略规则文件中添加一条规则，使其放行源自192.168.10.0/24网段，访问本机sshd服务的所有流量。可以看到，服务器立刻就放行了访问sshd服务的流量，效果非常直观：

```
[root@linuxprobe ~]# vim /etc/hosts.allow
#
# hosts.allow This file contains access rules which are used to
# allow or deny connections to network services that
# either use the tcp_wrappers library or that have been
# started through a tcp_wrappers-enabled xinetd.
#
# See 'man 5 hosts_options' and 'man 5 hosts_access'
# for information on rule syntax.
# See 'man tcpd' for information on tcp_wrappers
sshd:192.168.10.

[root@linuxprobe ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is 70:3b:5d:37:96:7b:2e:a5:28:0d:7e:dc:47:6a:fe:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 
Last login: Wed May 4 07:56:29 2021
[root@linuxprobe ~]# 
```

##### **8.5 Cockpit驾驶舱管理工具**

首先，Cockpit是一个英文单词，即飞机驾驶舱的意思，它用名字传达出了功能丰富的特性。其次，Cockpit是一个基于网页的图形化工具，天然具备很好的跨平台性，被 广泛使用于服务器、容器、虚拟机等等多种管理场景，即便是新手都可以直接上手操作。最后要说的是红帽公司对Cockpit十分的看重，从最开始就默认安装到了RHEL 8系统中，衍生的CentOS和Fedora也都是标配它。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/Cockpit.jpg)

默认情况下就已经被安装过了，执行下dnf命令确认下也好：

```
[root@linuxprobe ~]# dnf install cockpitUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                       3.1 MB/s | 3.2 kB     00:00    BaseOS                                          2.7 MB/s | 2.7 kB     00:00    Package cockpit-185-2.el8.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

Cockpit服务程序在RHEL 8.0版本中没有自动运行，将它开启并加入到开机启动项中：

```
[root@linuxprobe ~]# systemctl start cockpit[root@linuxprobe ~]# systemctl enable cockpit.socketCreated symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/systemd/system/cockpit.socket.
```

服务启动后打开系统自带的浏览器，访问“本机地址:9090”即可访问。但是由于https加密协议，但证书又是本地签发的，因此需要再多一个信任本地证书的操作，如图8-14与图8-15所示。进入到登录界面后，输入root管理员账号与系统密码，点击确认后即可进入，如图8-16所示。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/1-58-1024x593.png)

图8-14 添加额外允许的证书

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/2-37.png)

图8-15 确认信任本地证书

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/3-14-1024x588.png)

图8-16 输入登录账号与系统密码

进入到Cockpit网页界面后可谓是别有洞天，总共分为十三个功能模块，即：系统状态、日志信息、硬盘存储、网卡网络、账户安全、服务程序、软件仓库、报告分析、内核排错、SElinux、更新软件、订阅服务、终端界面。逐一为同学们进行讲解。

**1．系统状态**

默认进入后映入眼帘的便是系统状态界面，能够看到系统架构、版本、主机名与时间等信息，还能够动态的展现出CPU、硬盘、内存和网络的复杂情况，类似于网页版的“Winodws系统任务管理器”，属实好用，如图8-17所示：

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/1-59-1024x502.png)

图8-17 系统状态界面

**2．日志信息**

在这个模块中能够提供系统的全部日志，但是同学们会好奇，为什么如图8-18所示的内容如此有限呢。实际是原因出在上面两个选项中，其一是指定时间，其二是日志的级别，通过这两个选项可以让用户更快的找到所需信息，而不是像/var/log/message文件那样一股脑都抛给用户。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/2-38-1024x502.png)

图8-18 日志信息界面

**3．硬盘存储**

这是上课时同学们最最喜欢的功能模块之一，重点不是显示了硬盘的I/O读写负载情况，而是让用户能够通过该界面，用鼠标创建出RAID、LVM、VDO和iSCSI等存储设备，如图8-19所示。是的，没有看错，RAID和LVM都用鼠标可以创建了，是不是很开心~

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/3-15-1024x504.png)

图8-19 硬盘存储界面

**4．网卡网络**

既然称为网卡网络模块，那么动态看网卡的输出和接收值肯定是标配功能了。如图8-20所示，我们不仅可以在这里进行网卡的绑定（Bonding）和聚合（Team），还可以创建桥接网卡及添加VLAN，最下方会单独列出与网卡相关的日志信息。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/4-9-1024x498.png)

8-20 网卡网络界面

**5．账户安全**

千万别小看账户安全模块，虽然如图8-21所示功能界面有些简陋，只有一个创建账户的按钮，但实际上只要点击进入到某用户的管理界面中，马上会发现别有洞天。如图8-22所示，我们在这里可以对用户进行重命名、设置权限、锁定、修改密码及创建SSH密钥信息，功能非常丰富。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/5-9-1024x502.png)

图8-21 账户安全界面

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/5B-1024x502.png)

图8-22 账户管理界面

**6．服务程序**

如图8-23与图8-24所示，在这里可以查看到系统中已有的服务列表和运行状态，点击进入后可以对具体的服务进行开启、关闭操作。设置服务加入到开机启动项中，重启后也会依然为用户进行服务。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/6-3-1024x501.png)

图8-23 服务程序界面

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/6B-1024x502.png)

图8-24 服务管理界面

**7．软件仓库**

后期如果有用Cockpit或红帽订阅服务安装的软件，将会显示在这里，如图8-25所示。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/7-3-1024x499.png)

图8-25 软件仓库界面

**8．报告分析**

报告分析模块的功能是帮助用户收集及分析系统的信息，找到系统出现问题的原因，界面如图8-26所示，点击创建报告后大约两分钟左右，会出现如图8-27所示的成功界面。好吧摊牌了，这个页面其实功能很鸡肋，就是将sosreport命令做成了一个网页按钮。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/8-1024x502.png)

图8-26 报告分析界面

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/8B-1024x502.png)

图8-27 报告生成完毕

**9．内核排错**

Kdump是一个用于收集系统崩溃、死锁或死机时内核参数的一个服务。举例来说，如果有一天系统崩溃了，在这时Kdump服务就会开始工作，将系统的运行状态和内核数据收集到一个“dump core”的文件中，便于后续让运维人员分析找出问题所在。由于我们安装系统时没有启用它，因此该界面可以等后续再使用，如图8-28所示。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/9-2-1024x501.png)

图8-28 内核排错界面

**10．SElinux**

如图8-29所示为SELinux服务的控制按钮和警告信息界面，书籍将在第十章节详细的讲解SELinux安全子系统，此时暂不必了解。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/10-1024x500.png)

图8-29 SElinux管理界面

**11．更新软件**

软件更新界面如图8-30所示，但这里所提到的“Software Updates”实际不是日常更新软件的工作区域，而是红帽客户订阅的服务界面。用户需要购买红帽第三方服务后才能使用这里面的功能。购买红帽订阅服务后，便可以在这里下载到最新版本和稳定版本的服务程序，企业可自行选择是否购买，个人用户可以忽视或等待后续免费体验活动。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/11-5-1024x502.png)

图8-30 更新软件界面

**12．订阅服务**

订阅服务界面如图8-31所示，依然为红帽“小广告”一则，如果想成为尊贵的红帽服务使用者，要付费购买订阅服务，个人用户无需购买，对后续实验无任何影响。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/12-3-1024x500.png)

图8-31 订阅服务界面

**12．终端界面**

压轴的总在最后，终端管理界面如图8-32所示，Cockpit服务提供了Shell终端的在线控制平台，方便用户通过网页管理服务器，这个功能深受运维人员喜爱。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/13-2-1024x500.png)

图8-32 终端管理界面

至此，读者们就充分掌握了对防火墙的管理能力，多种防火墙管理工具任选其一即可。在配置后续的服务前，也要记得检查网络和防火墙的状态，避免出现服务明明配置正确，但从外部无法访问的情况，影响验证实验效果~好啦休息一下吧，一会继续学习！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．在RHEL 8系统中，iptables是否已经被firewalld服务彻底取代？

**答：**没有，iptables和firewalld服务均可用于RHEL 8系统。

2．请简述防火墙策略规则中DROP和REJECT的不同之处。

**答：**DROP的动作是丢包，不响应；REJECT是拒绝请求，同时向发送方回送拒绝信息。

3．如何把iptables服务的INPUT规则链默认策略设置为DROP？

**答：**执行命令iptables -P INPUT DROP即可。

4．怎样编写一条防火墙策略规则，使得iptables服务可以禁止源自192.168.10.0/24网段的流量访问本机的sshd服务（22端口）？

**答：**执行命令iptables -I INPUT -s 192.168.10.0/24 -p tcp --dport 22 -j REJECT即可。

5．请简述firewalld中区域的作用。

**答：**可以依据不同的工作场景来调用不同的firewalld区域，实现大量防火墙策略规则的快速切换。

6．如何在firewalld中把默认的区域设置为dmz？

**答：**执行命令firewall-cmd --set-default-zone=dmz即可。

7．如何让firewalld中以永久（Permanent）模式配置的防火墙策略规则立即生效？

**答：**执行命令firewall-cmd --reload。

8．使用SNAT技术的目的是什么？

**答：**SNAT是一种为了解决IP地址匮乏而设计的技术，它可以使得多个内网中的用户通过同一个外网IP接入Internet（互联网）。

9． TCP Wrappers服务分别有允许策略配置文件和拒绝策略配置文件，请问匹配顺序是怎么样的？

**答：**TCP Wrappers会先依次匹配允许策略配置文件，然后再依次匹配拒绝策略配置文件，如果都没有匹配到，则默认放行流量。

10．默认情况下Cockpit驾驶舱控制服务怎么使用？

**答：**Cockpit服务默认占用9090的端口号，可直接用浏览器访问Web界面。



### Web服务器

#### 静态web服务器部署

- ##### Apache服务器

  **章节简述：**

  本章先向读者科普什么是Web服务程序，以及Web服务程序的用处，然后通过对比当前主流的Web服务程序来使读者更好地理解其各自的优势及特点，最后通过对httpd服务程序中“全局配置参数”、“区域配置参数”及“注释信息”的理论讲解和实战部署，确保读者学会Web服务程序的配置方法，并真正掌握在[Linux系统](https://www.linuxprobe.com/)中配置服务的技巧。

  [刘遄](https://www.linuxprobe.com/)老师还会在本章讲解SE[Linux](https://www.linuxprobe.com/)服务的作用、三种工作模式以及策略管理方法，确保读者掌握SELinux域和SELinux安全上下文的配置方法，并依次完成多个基于httpd服务程序实用功能的部署实验，其中包括httpd服务程序的基本部署、个人用户主页功能和口令加密认证方式的实现，以及分别基于IP地址、主机名（域名）、端口号部署虚拟主机网站功能。

  本章目录结构

  - [10.1 网站服务程序](https://www.linuxprobe.com/basic-learning-10.html#101)
  - [10.2 配置服务文件参数](https://www.linuxprobe.com/basic-learning-10.html#102)
  - [10.3 SELinux安全子系统](https://www.linuxprobe.com/basic-learning-10.html#103SELinux)
  - [10.4 个人用户主页功能](https://www.linuxprobe.com/basic-learning-10.html#104)
  - 10.5 虚拟网站主机功能
    - [10.5.1 基于IP地址](https://www.linuxprobe.com/basic-learning-10.html#1051_IP)
    - [10.5.2 基于主机域名](https://www.linuxprobe.com/basic-learning-10.html#1052)
    - [10.5.3 基于端口号](https://www.linuxprobe.com/basic-learning-10.html#1053)
  - [10.6 Apache的访问控制](https://www.linuxprobe.com/basic-learning-10.html#106_Apache)

  ##### **10.1 网站服务程序**

  1970年，作为互联网前身的ARPANET（阿帕网）已初具雏形，并开始向非军用部门开放，许多大学和商业部门开始陆续接入。虽然彼时阿帕网的规模（只有4台主机联网运行）还不如现在的局域网成熟，但是它依然为网络技术的进步打下了扎实的基础。

  想必大多数人都是通过访问网站而开始接触互联网的吧。我们平时访问的网站服务就是Web网络服务，一般是指允许用户通过浏览器访问到互联网中各种资源的服务。如图10-1所示，Web网络服务是一种被动访问的服务程序，即只有接收到互联网中其他主机发出的请求后才会响应，最终用于提供服务程序的Web服务器会通过HTTP（超文本传输协议）或HTTPS（安全超文本传输协议）把请求的内容传送给用户。

  目前能够提供Web网络服务的程序有IIS、Nginx和Apache等。其中，IIS（Internet Information Services，互联网信息服务）是Windows系统中默认的Web服务程序，这是一款图形化的网站管理工具，不仅可以提供Web网站服务，还可以提供FTP、NMTP、SMTP等服务。但是，IIS只能在Windows系统中使用，暂时不在我们的学习范围之内。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2015/05/%E9%A1%B5%E9%9D%A2%E8%AF%B7%E6%B1%82%E8%BF%87%E7%A8%8B.png)

  图10-1 主机与Web服务器之间的通信

  2004年10月4日，为俄罗斯知名门户站点而开发的Web服务程序Nginx横空出世。Nginx程序作为一款轻量级的网站服务软件，因其稳定性和丰富的功能而快速占领服务器市场，但Nginx最被认可的还当是系统资源消耗低且并发能力强，因此得到了国内诸如新浪、网易、腾讯等门户站的青睐。本书将在第20章讲解Nginx服务程序。

  Apache程序是目前拥有很高市场占有率的Web服务程序之一，其跨平台和安全性广泛被认可且拥有快速、可靠、简单的API扩展。图10-2所示为Apache服务基金会的著名Logo，它的名字取自美国印第安人的土著语，寓意着拥有高超的作战策略和无穷的耐性。Apache服务程序可以运行在Linux系统、UNIX系统甚至是Windows系统中，支持基于IP、域名及端口号的虚拟主机功能，支持多种认证方式，集成有代理服务器模块、安全Socket层（SSL），能够实时监视服务状态与定制日志消息，并有着各类丰富的模块支持。

  > Apache程序是在RHEL 5、6、7、8系统的默认Web服务程序，其相关知识点一直也是RHCSA和[RHCE](https://www.linuxprobe.com/)认证考试的重点内容。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2015/05/Apache.jpg)

  图10-2 Apache软件基金会著名的Logo

  总结来说，Nginx服务程序作为后起之秀，已经通过自身的优势与努力赢得了大批站长的信赖。本书配套的在线学习站点[https://www.linuxprobe.com](https://www.linuxprobe.com/)就是基于Nginx服务程序部署的，不得不说Nginx也真的很棒！

  但是，Apache程序作为老牌的Web服务程序，一方面在Web服务器软件市场具有相当高的占有率，另一方面Apache也是RHEL 8系统中默认的Web服务程序，而且还是RHCSA和[RHCE](https://www.linuxprobe.com/)认证考试的必考内容，因此无论从实际应用角度还是从应对[红帽](https://www.linuxprobe.com/)认证考试的角度，我们都有必要好好学习Apache服务程序的部署，并深入挖掘其可用的丰富功能。

  **再来回忆下软件仓库的配置过程吧：**

  **第1步**：把系统镜像挂载到/media/cdrom目录。

  ```
  [root@linuxprobe ~]# mkdir -p /media/cdrom
  [root@linuxprobe ~]# mount /dev/cdrom /media/cdrom
  mount: /media/cdrom: WARNING: device write-protected, mounted read-only.
  ```

  **第2步**：使用Vim文本编辑器创建软件仓库的配置文件，下述[命令](https://www.linuxcool.com/)中具体参数的含义可参考[4.1.4小节](https://www.linuxprobe.com/basic-learning-04.html#414)。

  ```
  [root@linuxprobe ~]# vim /etc/yum.repos.d/rhel8.repo
  [BaseOS]
  name=BaseOS
  baseurl=file:///media/cdrom/BaseOS
  enabled=1
  gpgcheck=0
  [AppStream]
  name=AppStream
  baseurl=file:///media/cdrom/AppStream
  enabled=1
  gpgcheck=0
  ```

  **第3步**：动手安装Apache服务程序。注意，使用dnf[命令](https://www.linuxcool.com/)进行安装时，跟在命令后面的Apache服务的软件包名称为httpd。

  ```
  [root@linuxprobe ~]# dnf install httpd
  Updating Subscription Management repositories.
  Unable to read consumer identity
  This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
  AppStream                                       3.1 MB/s | 3.2 kB     00:00    
  BaseOS                                          2.7 MB/s | 2.7 kB     00:00    
  Dependencies resolved.
  ================================================================================
   Package            Arch   Version                   Repository           Size
  ================================================================================
  Installing:
   httpd              x86_64 2.4.37-10.module+el8+2764+7127e69e   AppStream 1.4 M
  Installing dependencies:
   apr                x86_64 1.6.3-9.el8                          AppStream 125 k
   apr-util           x86_64 1.6.1-6.el8                          AppStream 105 k
   httpd-filesystem   noarch 2.4.37-10.module+el8+2764+7127e69e   AppStream  34 k
   httpd-tools        x86_64 2.4.37-10.module+el8+2764+7127e69e   AppStream 101 k
   mod_http2          x86_64 1.11.3-1.module+el8+2443+605475b7    AppStream 156 k
   redhat-logos-httpd noarch 80.7-1.el8                           BaseOS     25 k
  Installing weak dependencies:
   apr-util-bdb       x86_64 1.6.1-6.el8                          AppStream  25 k
   apr-util-openssl   x86_64 1.6.1-6.el8                          AppStream  27 k
  Enabling module streams:
   httpd                     2.4                                                 
  
  Transaction Summary
  ================================================================================
  Install  9 Packages
  
  Total size: 2.0 M
  Installed size: 5.4 M
  Is this ok [y/N]: y
  Downloading Packages:
  Running transaction check
  Transaction check succeeded.
  Running transaction test
  Transaction test succeeded.
  ………………省略部分输出信息………………                                    
  Complete!
  ```

  **第4步**：启用httpd服务程序并将其加入到开机启动项中，使其能够随系统开机而运行，从而持续为用户提供Web服务：

  ```
  [root@linuxprobe ~]# systemctl start httpd
  [root@linuxprobe ~]# systemctl enable httpd
  Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
  ```

  大家在浏览器（这里以Firefox浏览器为例）的地址栏中输入http://127.0.0.1并按回车键，就可以看到用于提供Web服务的默认页面了，如图10-3所示。

  ```
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%BD%91%E7%AB%99%E6%9C%8D%E5%8A%A1%E9%BB%98%E8%AE%A4%E9%A1%B5%E9%9D%A2-1-1024x614.png)

  图10-3 httpd服务程序的默认页面

  ##### **10.2 配置服务文件参数**

  需要提醒大家的是，前文介绍的httpd服务程序的安装和运行，仅仅是httpd服务程序的一些皮毛，我们依然有很长的道路要走。在Linux系统中配置服务，其实就是修改服务的配置文件，因此，还需要知道这些配置文件的所在位置以及用途，httpd服务程序的主要配置文件及存放位置如表10-1所示。

  表10-1                        Linux系统中的配置文件

  | 作用         | 文件名称                   |
  | ------------ | -------------------------- |
  | 服务目录     | /etc/httpd                 |
  | 主配置文件   | /etc/httpd/conf/httpd.conf |
  | 网站数据目录 | /var/www/html              |
  | 访问日志     | /var/log/httpd/access_log  |
  | 错误日志     | /var/log/httpd/error_log   |

  主配置文件中保存的是最重要的服务参数，一般名称都是以在/etc中，以软件名称命名的一个文件夹，里面叫做“服务名称.conf”，例如这里的“/etc/httpd/conf/httpd.conf”熟练后就能记住了。

  大家在首次打开httpd服务程序的主配置文件，可能会吓一跳—竟然有356行！这得至少需要一周的时间才能看完吧？！但是，大家只要仔细观看就会发现刘遄老师在这里调皮了。因为在这个配置文件中，所有以井号（#）开始的行都是注释行，其目的是对httpd服务程序的功能或某一行参数进行介绍，不需要逐行研究这些内容。

  在httpd服务程序的主配置文件中，存在三种类型的信息：注释行信息、全局配置、区域配置，如图10-4所示。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2015/05/httpd%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6%E5%88%86%E6%9E%90.png)

  图10-4 httpd服务主配置文件的参数结构

  各位读者在学习第四章时已经接触过注释信息，因此这里主要讲解全局配置参数与区域配置参数的区别。顾名思义，全局配置参数就是一种全局性的配置参数，可作用于对所有的子站点，既保证了子站点的正常访问，也有效减少了频繁写入重复参数的工作量。区域配置参数则是单独针对于每个独立的子站点进行设置的。就像在大学食堂里面打饭，食堂负责打饭的阿姨先给每位同学来一碗标准大小的白饭（全局配置），然后再根据每位同学的具体要求盛放他们想吃的菜（区域配置）。在httpd服务程序主配置文件中，最为常用的参数如表10-2所示。

  表10-2             配置httpd服务程序时最常用的参数以及用途描述

  | 参数           | 作用                      |
  | -------------- | ------------------------- |
  | ServerRoot     | 服务目录                  |
  | ServerAdmin    | 管理员邮箱                |
  | User           | 运行服务的用户            |
  | Group          | 运行服务的用户组          |
  | ServerName     | 网站服务器的域名          |
  | DocumentRoot   | 网站数据目录              |
  | Listen         | 监听的IP地址与端口号      |
  | DirectoryIndex | 默认的索引页页面          |
  | ErrorLog       | 错误日志文件              |
  | CustomLog      | 访问日志文件              |
  | Timeout        | 网页超时时间，默认为300秒 |

  从表10-2中可知，DocumentRoot参数用于定义网站数据的保存路径，其参数的默认值是把网站数据存放到/var/www/html目录中；而当前网站普遍的首页面名称是index.html，因此可以向/var/www/html/index.html文件中写入一段内容，替换掉httpd服务程序的默认首页面，该操作会立即生效。

  ```
  [root@linuxprobe ~]# echo "Welcome To LinuxProbe.Com" > /var/www/html/index.html 
  [root@linuxprobe ~]# firefox
  ```

  在执行上述操作之后，再在Firefox浏览器中刷新httpd服务程序，可以看到该程序的首页面内容已经发生了改变，如图10-5所示。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E7%BD%91%E7%AB%99%E6%95%88%E6%9E%9C-1024x140.png)

  图10-5 首页面内容已经被修改

  大家在完成这个实验之后，是不是信心爆棚了呢？！在默认情况下，网站数据是保存在/var/www/html目录中，而如果想把保存网站数据的目录修改为/home/wwwroot目录，该怎么操作呢？且看下文。

  **第1步**：建立网站数据的保存目录，并创建首页文件。

  ```
  [root@linuxprobe ~]# mkdir /home/wwwroot
  [root@linuxprobe ~]# echo "The New Web Directory" > /home/wwwroot/index.html
  ```

  **第2步**：打开httpd服务程序的主配置文件，将约第122行用于定义网站数据保存路径的参数DocumentRoot修改为/home/wwwroot，同时还需要将约第127行与134行用于定义目录权限的参数Directory后面的路径也修改为/home/wwwroot。配置文件修改完毕后即可保存并退出。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf 
  ………………省略部分输出信息………………
  117 #
  118 # DocumentRoot: The directory out of which you will serve your
  119 # documents. By default, all requests are taken from this directory, but
  120 # symbolic links and aliases may be used to point to other locations.
  121 #
  122 DocumentRoot "/home/wwwroot"
  123 
  124 #
  125 # Relax access to content within /var/www.
  126 #
  127 <Directory "/home/wwwroot">
  128     AllowOverride None
  129     # Allow open access:
  130     Require all granted
  131 </Directory>
  132 
  133 # Further relax access to the default document root:
  134 <Directory "/home/wwwroot">
  ………………省略部分输出信息………………
  ```

  **第3步**：重新启动httpd服务程序并验证效果，浏览器刷新页面后的内容如图10-6所示。奇怪！怎么提示权限不足了？

  ```
  [root@linuxprobe ~]# systemctl restart httpd
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%9D%83%E9%99%90%E4%B8%8D%E8%B6%B3-1024x207.png)

  图10-6 Web页面提示权限不足

  ##### **10.3 SELinux****安全子系统**

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2015/05/selinux.png)

  SELinux（Security-Enhanced Linux）是美国国家安全局在Linux开源社区的帮助下开发的一个强制访问控制（MAC，Mandatory Access Control）的安全子系统。Linux系统使用SELinux技术的目的是为了让各个服务进程都受到约束，使其仅获取到本应获取的资源。

  例如，您在自己的电脑上下载了一个美图软件，正全神贯注地使用它给照片进行美颜的时候，它却在后台默默监听着浏览器中输入的密码信息，而这显然不应该是它应做的事情（哪怕是访问电脑中的图片资源，都情有可原）。SELinux安全子系统就是为了杜绝此类情况而设计的，它能够从多方面监控违法行为：对服务程序的功能进行限制——SELinux域限制可以确保服务程序做不了出格的事情；对文件资源的访问限制——SELinux安全上下文确保文件资源只能被其所属的服务程序进行访问。

  如果一般权限和防火墙是门窗，那么SELinux便是在外面安装的防护栏，让系统内部更加安全。

  刘遄老师经常会把“SELinux域”和“SELinux安全上下文”称为是Linux系统中的双保险，系统内的服务程序只能规规矩矩地拿到自己所应该获取的资源，这样即便黑客入侵了系统，也无法利用系统内的服务程序进行越权操作。但是，非常可惜的是，SELinux服务比较复杂，配置难度也很大，加之很多运维人员对这项技术理解不深，从而导致很多服务器在部署好Linux系统后直接将SELinux禁用了；这绝对不是明智的选择。

  SELinux服务有三种配置模式，具体如下。

  > enforcing：强制启用安全策略模式，将拦截服务的不合法请求。
  >
  > permissive：遇到服务越权访问时，只发出警告而不强制拦截。
  >
  > disabled：对于越权的行为不警告也不拦截。

  本书中的所有实验都是在强制启用安全策略模式下进行的，虽然在禁用SELinux服务后确实能够减少报错几率，但这在生产环境中相当不推荐。建议大家检查一下自己的系统，查看SELinux服务主配置文件中定义的默认状态。如果是permissive或disabled，建议赶紧修改为enforcing。

  ```
  [root@linuxprobe ~]# vim /etc/selinux/config
  # This file controls the state of SELinux on the system.
  # SELINUX= can take one of these three values:
  #     enforcing - SELinux security policy is enforced.
  #     permissive - SELinux prints warnings instead of enforcing.
  #     disabled - No SELinux policy is loaded.
  SELINUX=enforcing
  # SELINUXTYPE= can take one of these three values:
  #     targeted - Targeted processes are protected,
  #     minimum - Modification of targeted policy. Only selected processes are protected. 
  #     mls - Multi Level Security protection.
  SELINUXTYPE=targeted
  ```

  SELinux服务的主配置文件中，定义的是SELinux的默认运行状态，可以将其理解为系统重启后的状态，因此它不会在更改后立即生效。可以使用getenforce命令获得当前SELinux服务的运行模式：

  ```
  [root@linuxprobe ~]# getenforce 
  Enforcing
  ```

  为了确认图10-6所示的结果确实是因为SELinux而导致的，可以用setenforce [0|1]命令修改SELinux当前的运行模式（0为禁用，1为启用）。注意，这种修改只是临时的，在系统重启后就会失效：

  ```
  [root@linuxprobe ~]# setenforce 0
  [root@linuxprobe ~]# getenforce
  Permissive
  ```

  再次刷新网页，就会看到正常的网页内容了，如图10-7所示。可见，问题确实是出在了SELinux服务上面。

  ```
  [root@linuxprobe wwwroot]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%81%A2%E5%A4%8D%E6%AD%A3%E5%B8%B8-1024x160.png)

  图10-7 页面内容按照预期显示

  现在，回忆下前面的操作中到底是哪里出问题了呢？

  httpd服务程序的功能是允许用户访问网站内容，因此SELinux肯定会默认放行用户对网站的请求操作。但是，我们将网站数据的默认保存目录修改为了/home/wwwroot，而这就产生问题了。在6.1小节中讲到，/home目录是用来存放普通用户的家目录数据的，而现在，httpd提供的网站服务却要去获取普通用户家目录中的数据了，这显然违反了SELinux的监管原则。

  现在，把SELinux服务恢复到强制启用安全策略模式，然后分别查看原始网站数据的保存目录与当前网站数据的保存目录是否拥有不同的SELinux安全上下文值。ls命令中“-Z”参数用于查看文件的安全上下文值，而“-d”参数代表对象是个文件夹。

  ```
  [root@linuxprobe ~]# setenforce 1
  [root@linuxprobe ~]# ls -Zd /var/www/html
  drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html
  [root@linuxprobe ~]# ls -Zd /home/wwwroot
  drwxrwxrwx. root root unconfined_u:object_r:home_root_t:s0 /home/wwwroot
  ```

  在文件上设置的SELinux安全上下文是由用户段、角色段以及类型段等多个信息项共同组成的。其中，用户段system_u代表系统进程的身份，角色段object_r代表文件目录的角色，类型段httpd_sys_content_t代表网站服务的系统文件。由于SELinux服务实在太过复杂，现在大家只需要简单熟悉SELinux服务的作用就可以，刘遄老师未来会在本书的进阶篇中单独拿出一个章节仔细讲解SELinux服务。

  针对当前这种情况，我们只需要使用semanage命令，将当前网站目录/home/wwwroot的SELinux安全上下文修改为跟原始网站目录的一样就行了。

  semanage命令用于管理SELinux的策略，英文全称为：“SELinux manage”，语法格式为：“semanage [参数] [文件]”。

  SELinux服务极大地提升了Linux系统的安全性，将用户权限牢牢地锁在笼子里。semanage命令不仅能够像传统chcon命令那样—设置文件、目录的策略，还能够管理网络端口、消息接口（这些新特性将在本章后文中涵盖）。使用semanage命令时，经常用到的几个参数及其作用如表10-3所示：

  表10-3            semanage命令中常用参数以及作用

  | 参数 | 作用 |
  | ---- | ---- |
  | -l   | 查询 |
  | -a   | 添加 |
  | -m   | 修改 |
  | -d   | 删除 |

  

  例如，向新的网站数据目录中新添加一条SELinux安全上下文，让这个目录以及里面的所有文件能够被httpd服务程序所访问到：

  ```
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/*
  ```

  注意，执行上述设置之后，还无法立即访问网站，还需要使用restorecon命令将设置好的SELinux安全上下文立即生效。在使用restorecon命令时，可以加上-Rv参数对指定的目录进行递归操作，以及显示SELinux安全上下文的修改过程。最后，再次刷新页面，就可以正常看到网页内容了，结果如图10-8所示。

  ```
  [root@linuxprobe ~]# restorecon -Rv /home/wwwroot/
  Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%86%8D%E6%AC%A1%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-1024x132.png)

  图10-8 正常看到网页内容

  真可谓是一波三折！原本认为只要把httpd服务程序配置妥当就可以大功告成，结果却反复受到了SELinux安全上下文的限制。所以，建议大家在配置httpd服务程序时，一定要细心、耐心。一旦成功配妥httpd服务程序之后，就会发现SELinux服务并没有那么难。

  由于在RHCSA、RHCE或RHCA考试中，都需要先重启您的机器然后再执行判分脚本。因此，建议读者在日常工作中要养成将所需服务添加到开机启动项中的习惯，比如这里就需要添加systemctl enable httpd命令。

  ##### **10.4 个人用户主页功能**

  如果想在系统中为每位用户建立一个独立的网站，通常的方法是基于虚拟网站主机功能来部署多个网站。但这个工作会让管理员苦不堪言（尤其是用户数量很庞大时），而且在用户自行管理网站时，还会碰到各种权限限制，需要为此做很多额外的工作。其实，httpd服务程序提供的个人用户主页功能完全可以胜任这个工作。该功能可以让系统内所有的用户在自己的家目录中管理个人的网站，而且访问起来也非常容易。

  **第1步**：在httpd服务程序中，默认没有开启个人用户主页功能。为此，我们需要编辑下面的配置文件，然后在第17行的UserDir disabled参数前面加上井号（#），表示让httpd服务程序开启个人用户主页功能；同时再把第24行的UserDir public_html参数前面的井号（#）去掉（UserDir参数表示网站数据在用户家目录中的保存目录名称，即public_html目录）。最后，在修改完毕后记得保存。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf.d/userdir.conf 
    1 #
    2 # UserDir: The name of the directory that is appended onto a user's home
    3 # directory if a ~user request is received.
    4 #
    5 # The path to the end user account 'public_html' directory must be
    6 # accessible to the webserver userid.  This usually means that ~userid
    7 # must have permissions of 711, ~userid/public_html must have permissions
    8 # of 755, and documents contained therein must be world-readable.
    9 # Otherwise, the client will only receive a "403 Forbidden" message.
   10 #
   11 <IfModule mod_userdir.c>
   12     #
   13     # UserDir is disabled by default since it can confirm the presence
   14     # of a username on the system (depending on home directory
   15     # permissions).
   16     #
   17     # UserDir disabled
   18 
   19     #
   20     # To enable requests to /~user/ to serve the user's public_html
   21     # directory, remove the "UserDir disabled" line above, and uncomment
   22     # the following line instead:
   23     # 
   24       UserDir public_html
   25 </IfModule>
   26 
   27 #
   28 # Control access to UserDir directories.  The following is an example
   29 # for a site where these directories are restricted to read-only.
   30 #
   31 <Directory "/home/*/public_html">
   32     AllowOverride FileInfo AuthConfig Limit Indexes
   33     Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
   34     Require method GET POST OPTIONS
   35 </Directory>
  ```

  **第2步**：在用户家目录中建立用于保存网站数据的目录及首页面文件。另外，还需要把家目录的权限修改为755，保证其他人也有权限读取里面的内容。

  ```
  [root@linuxprobe home]# su - linuxprobe
  [linuxprobe@linuxprobe ~]$ mkdir public_html
  [linuxprobe@linuxprobe ~]$ echo "This is linuxprobe's website" > public_html/index.html
  [linuxprobe@linuxprobe ~]$ chmod -R 755 /home/linuxprobe
  ```

  **第3步**：重新启动httpd服务程序，在浏览器的地址栏中输入网址，其格式为“网址/~用户名”（其中的波浪号是必需的，而且网址、波浪号、用户名之间没有空格），从理论上来讲就可以看到用户的个人网站了。不出所料的是，系统显示报错页面，如图10-9所示。这一定还是SELinux惹的祸。

  ```
  [linuxprobe@linuxprobe ~]$ exit
  logout
  [root@linuxprobe ~]# systemctl restart httpd
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/20210208134750-1024x184.png)

  图10-9 禁止访问用户的个人网站

  **第4步**：思考这次报错的原因是什么。httpd服务程序在提供个人用户主页功能时，该用户的网站数据目录本身就应该是存放到与这位用户对应的家目录中的，所以应该不需要修改家目录的SELinux安全上下文。但是，前文还讲到了SELinux域的概念。SELinux域确保服务程序不能执行违规的操作，只能本本分分地为用户提供服务。httpd服务中突然开启的这项个人用户主页功能到底有没有被SELinux域默认允许呢？

  接下来使用getsebool命令查询并过滤出所有与HTTP协议相关的安全策略。其中，off为禁止状态，on为允许状态。

  ```
  [root@linuxprobe ~]# getsebool -a | grep http
  httpd_anon_write --> off
  httpd_builtin_scripting --> on
  httpd_can_check_spam --> off
  httpd_can_connect_ftp --> off
  httpd_can_connect_ldap --> off
  httpd_can_connect_mythtv --> off
  httpd_can_connect_zabbix --> off
  httpd_can_network_connect --> off
  httpd_can_network_connect_cobbler --> off
  httpd_can_network_connect_db --> off
  httpd_can_network_memcache --> off
  httpd_can_network_relay --> off
  httpd_can_sendmail --> off
  httpd_dbus_avahi --> off
  httpd_dbus_sssd --> off
  httpd_dontaudit_search_dirs --> off
  httpd_enable_cgi --> on
  httpd_enable_ftp_server --> off
  httpd_enable_homedirs --> off
  httpd_execmem --> off
  httpd_graceful_shutdown --> off
  httpd_manage_ipa --> off
  httpd_mod_auth_ntlm_winbind --> off
  httpd_mod_auth_pam --> off
  httpd_read_user_content --> off
  httpd_run_ipa --> off
  httpd_run_preupgrade --> off
  httpd_run_stickshift --> off
  httpd_serve_cobbler_files --> off
  httpd_setrlimit --> off
  httpd_ssi_exec --> off
  httpd_sys_script_anon_write --> off
  httpd_tmp_exec --> off
  httpd_tty_comm --> off
  httpd_unified --> off
  httpd_use_cifs --> off
  httpd_use_fusefs --> off
  httpd_use_gpg --> off
  httpd_use_nfs --> off
  httpd_use_openstack --> off
  httpd_use_sasl --> off
  httpd_verify_dns --> off
  mysql_connect_http --> off
  named_tcp_bind_http_port --> off
  prosody_bind_http_port --> off
  ```

  面对如此多的SELinux域安全策略规则，实在没有必要逐个理解它们，我们只要能通过名字大致猜测出相关的策略用途就足够了。比如，想要开启httpd服务的个人用户主页功能，那么用到的SELinux域安全策略应该是httpd_enable_homedirs吧？大致确定后就可以用setsebool命令来修改SELinux策略中各条规则的布尔值了。大家一定要记得在setsebool命令后面加上-P参数，让修改后的SELinux策略规则永久生效且立即生效。随后刷新网页，其效果如图10-10所示。

  ```
  [root@linuxprobe ~]# setsebool -P httpd_enable_homedirs=on
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-1024x137.png)

  图10-10 正常看到个人用户主页面中的内容

  有时，网站的拥有者并不希望直接将网页内容显示出来，只想让通过身份验证的用户访客看到里面的内容。

  **第1步**：先使用htpasswd命令生成密码数据库。-c参数表示第一次生成；后面再分别添加密码数据库的存放文件，以及验证要用到的用户名称（该用户不必是系统中已有的本地账户）。

  ```
  [root@linuxprobe ~]# htpasswd -c /etc/httpd/passwd linuxprobe
  New password:此处输入用于网页验证的密码
  Re-type new password:再输入一遍进行确认
  Adding password for user linuxprobe
  ```

  **第2步**：继续编辑个人用户主页功能的配置文件。把第31～37行的参数信息修改成下列内容，其中井号（#）开头的内容为添加的注释信息，可将其忽略。随后保存并退出配置文件，重启httpd服务程序即可生效。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf.d/userdir.conf
  ………………省略部分输出信息………………
   27 #
   28 # Control access to UserDir directories.  The following is an example
   29 # for a site where these directories are restricted to read-only.
   30 #
   31 <Directory "/home/*/public_html">
   32     AllowOverride all 
          #刚刚生成出的密码验证文件保存路径
   33     authuserfile "/etc/httpd/passwd" 
          #当用户访问网站时的提示信息
   34     authname "My privately website"
          #验证方式为口令模式
   35     authtype basic
          #访问网站时需要验证的用户名称
   36     require user linuxprobe
   37 </Directory>
  [root@linuxprobe ~]# systemctl restart httpd
  ```

  此后，当用户再想访问某个用户的个人网站时，就必须要输入账户和密码才能正常访问了。另外，验证时使用的账户和密码是用htpasswd命令生成的专门用于网站登录的口令密码，而不是系统中的用户密码，请不要搞错了。登录界面如图10-11与图10-12所示。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E9%AA%8C%E8%AF%81%E8%B4%A6%E5%8F%B7%E5%AF%86%E7%A0%81-1024x469.png)

  图10-11 需要输入账户和密码才能访问

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-1-1024x291.png)

  图10-12 口令验证成功

  **出现问题?大胆提问!**

  > 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
  >
  > Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
  >
  > *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

  ##### **10.5 虚拟网站主机功能**

  如果每台运行Linux系统的服务器上只能运行一个网站，那么人气低、流量小的草根站长就要被迫承担着高昂的服务器租赁费用了，这显然也会造成硬件资源的浪费。在虚拟专用服务器（Virtual Private Server，VPS）与云计算技术诞生以前，IDC服务供应商为了能够更充分地利用服务器资源，同时也为了降低购买门槛，于是纷纷启用了虚拟主机功能。

  利用虚拟主机功能，可以把一台处于运行状态的物理服务器分割成多个“虚拟的服务器”。但是，该技术无法实现目前云主机技术的硬件资源隔离，让这些虚拟的服务器共同使用物理服务器的硬件资源，供应商只能限制硬盘的使用空间大小。出于各种考虑的因素（主要是价格低廉），目前依然有很多企业或个人站长在使用虚拟主机的形式来部署网站。

  Apache的虚拟主机功能是服务器基于用户请求的不同IP地址、主机域名或端口号，实现提供多个网站同时为外部提供访问服务的技术，如图10-13所示，用户请求的资源不同，最终获取到的网页内容也各不相同。如果大家之前没有做过网站，可能不太理解其中的原理，等一会儿搭建出实验环境并看到实验效果之后，您一定就会明白了。

  > 再次提醒大家，在做每个实验之前请先将虚拟机还原到最初始状态，以免多个实验之间相互产生冲突。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/Apache%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E5%8A%9F%E8%83%BD%E6%8B%93%E6%89%91-1.jpg)

  图10-13 用户请求网站资源

  ###### **10.5.1 基于IP地址**

  如果一台服务器有多个IP地址，而且每个IP地址与服务器上部署的每个网站一一对应，这样当用户请求访问不同的IP地址时，会访问到不同网站的页面资源。而且，每个网站都有一个独立的IP地址，对搜索引擎优化也大有裨益。因此以这种方式提供虚拟网站主机功能不仅最常见，也受到了网站站长的欢迎（尤其是草根站长）。

  在第4章和第9章分别讲解了用于配置网络的两种方法，大家在实验中和工作中可随意选择。就当前的实验来讲，需要配置的IP地址如图10-14所示。在配置完毕并重启网卡服务之后，记得检查网络的连通性，确保三个IP地址均可正常访问，如图10-15所示（这很重要，一定要测试好，然后再进行下一步!）。

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E9%85%8D%E7%BD%AEIP%E5%9C%B0%E5%9D%80-1024x647.png)

  图10-14 使用nmtui命令配置网络参数

  ```
  [root@linuxprobe ~]# nmcli connection up ens160 
  Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1-1024x612.png)

  图10-15 分别检查3个IP地址的连通性

  **第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

  ```
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/10
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/20
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/30
  [root@linuxprobe ~]# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html
  [root@linuxprobe ~]# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html
  [root@linuxprobe ~]# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
  ```

  **第2步**：在httpd服务的配置文件中大约132行处开始，分别追加写入三个基于IP地址的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
  ………………省略部分输出信息………………
  132 <VirtualHost 192.168.10.10>
  133     DocumentRoot /home/wwwroot/10
  134     ServerName www.linuxprobe.com
  135     <Directory /home/wwwroot/10>
  136     AllowOverride None
  137     Require all granted
  138     </Directory>
  139 </VirtualHost>
    
  140 <VirtualHost 192.168.10.20>
  141     DocumentRoot /home/wwwroot/20
  142     ServerName www.linuxcool.com
  143     <Directory /home/wwwroot/20>
  144     AllowOverride None
  145     Require all granted
  146     </Directory>
  147 </VirtualHost>
    
  148 <VirtualHost 192.168.10.30>
  149     DocumentRoot /home/wwwroot/30
  150     ServerName www.linuxdown.com
  151     <Directory /home/wwwroot/30>
  152     AllowOverride None
  153     Require all granted
  154     </Directory>
  155 </VirtualHost>
  ………………省略部分输出信息………………
  [root@linuxprobe ~]# systemctl restart httpd
  ```

  **第3步**：此时访问网站，则会看到httpd服务程序的默认首页面，提示权限不足。大家现在应该立刻就反应过来—这是SELinux在捣鬼。由于当前的/home/wwwroot目录及里面的网站数据目录的SELinux安全上下文与网站服务不吻合，因此httpd服务程序无法获取到这些网站数据目录。我们需要手动把新的网站数据目录的SELinux安全上下文设置正确（见前文的实验），并使用restorecon命令让新设置的SELinux安全上下文立即生效，这样就可以立即看到网站的访问效果了，如图10-16所示。

  ```
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30/*
  [root@linuxprobe ~]# restorecon -Rv /home/wwwroot
  Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/10 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/10/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/20 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/20/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/30 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/30/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E4%B8%8D%E5%90%8C%E7%9A%84IP%E5%9C%B0%E5%9D%80%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x352.png)

  图10-16 基于不同的IP地址访问虚拟主机网站

  ###### **10.5.2 基于主机域名**

  当服务器无法为每个网站都分配一个独立IP地址的时候，可以尝试让Apache自动识别用户请求的域名，从而根据不同的域名请求来传输不同的内容。在这种情况下的配置更加简单，只需要保证位于生产环境中的服务器上有一个可用的IP地址（这里以192.168.10.10为例）就可以了。由于当前还没有介绍如何配置DNS解析服务，因此需要手工定义IP地址与域名之间的对应关系。/etc/hosts是Linux系统中用于强制把某个主机域名解析到指定IP地址的配置文件。简单来说，只要这个文件配置正确，即使网卡参数中没有DNS信息也依然能够将域名解析为某个IP地址。

  **第1步**：手工定义IP地址与域名之间对应关系的配置文件，保存并退出后会立即生效。可以通过分别ping这些域名来验证域名是否已经成功解析为IP地址。

  ```
  [root@linuxprobe ~]# vim /etc/hosts
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  192.168.10.10   www.linuxprobe.com www.linuxcool.com www.linuxdown.com
  [root@linuxprobe ~]# ping -c 4 www.linuxprobe.com
  PING www.linuxprobe.com (192.168.10.10) 56(84) bytes of data.
  64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=1 ttl=64 time=0.070 ms
  64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=2 ttl=64 time=0.077 ms
  64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=3 ttl=64 time=0.061 ms
  64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=4 ttl=64 time=0.069 ms
  --- www.linuxprobe.com ping statistics ---
  4 packets transmitted, 4 received, 0% packet loss, time 2999ms
  rtt min/avg/max/mdev = 0.061/0.069/0.077/0.008 ms
  [root@linuxprobe ~]# 
  ```

  **第2步**：分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

  ```
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxprobe
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxcool
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxdown
  [root@linuxprobe ~]# echo "www.linuxprobe.com" > /home/wwwroot/linuxprobe/index.html
  [root@linuxprobe ~]# echo "www.linuxcool.com" > /home/wwwroot/linuxcool/index.html
  [root@linuxprobe ~]# echo "www.linuxdown.com" > /home/wwwroot/linuxdown/index.html
  ```

  **第3步**：在httpd服务的配置文件中大约132行处开始，分别追加写入三个基于主机名的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
  ………………省略部分输出信息………………
  132 <VirtualHost 192.168.10.10>
  133     Documentroot /home/wwwroot/linuxprobe
  134     ServerName www.linuxprobe.com
  135     <Directory /home/wwwroot/linuxprobe>
  136     AllowOverride None
  137     Require all granted
  138     </Directory>
  139 </VirtualHost>
   
  140 <VirtualHost 192.168.10.10>
  141     Documentroot /home/wwwroot/linuxcool
  142     ServerName www.linuxcool.com
  143     <Directory /home/wwwroot/linuxcool>
  144     AllowOverride None
  145     Require all granted
  146     </Directory>
  147 </VirtualHost>
   
  148 <VirtualHost 192.168.10.10>
  149     Documentroot /home/wwwroot/linuxdown
  150     ServerName www.linuxdown.com
  151     <Directory /home/wwwroot/linuxdown>
  152     AllowOverride None
  153     Require all granted
  154     </Directory>
  155 </VirtualHost>
  ………………省略部分输出信息………………
  [root@linuxprobe ~]# systemctl restart httpd
  ```

  **第4步**：因为当前的网站数据目录还是在/home/wwwroot目录中，因此还是必须要正确设置网站数据目录文件的SELinux安全上下文，使其与网站服务功能相吻合。最后记得用restorecon命令让新配置的SELinux安全上下文立即生效，这样就可以立即访问到虚拟主机网站了，效果如图10-17所示。

  ```
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown/*
  [root@linuxprobe ~]# restorecon -Rv /home/wwwroot
  Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxprobe from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxprobe/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxcool from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxcool/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxdown from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/linuxdown/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  [root@linuxprobe ~]# firefox 
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E4%B8%BB%E6%9C%BA%E5%9F%9F%E5%90%8D%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x383.png)

  图10-17 基于主机域名访问虚拟主机网站

  ###### **10.5.3 基于端口号**

  基于端口号的虚拟主机功能可以让用户通过指定的端口号来访问服务器上的网站资源。在使用Apache配置虚拟网站主机功能时，基于端口号的配置方式是最复杂的。因此我们不仅要考虑httpd服务程序的配置因素，还需要考虑到SELinux服务对新开设端口的监控。一般来说，使用80、443、8080等端口号来提供网站访问服务是比较合理的，如果使用其他端口号则会受到SELinux服务的限制。

  在接下来的实验中，我们不但要考虑到目录上应用的SELinux安全上下文的限制，还需要考虑SELinux域对httpd服务程序的管控。

  **第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

  ```
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/6111
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/6222
  [root@linuxprobe ~]# mkdir -p /home/wwwroot/6333
  [root@linuxprobe ~]# echo "port:6111" > /home/wwwroot/6111/index.html
  [root@linuxprobe ~]# echo "port:6222" > /home/wwwroot/6222/index.html
  [root@linuxprobe ~]# echo "port:6333" > /home/wwwroot/6333/index.html
  ```

  **第2步**：在httpd服务配置文件的第46行至48行分别添加用于监听6111、6222和6333端口的参数。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf 
  ………………省略部分输出信息……………… 
   37 # Listen: Allows you to bind Apache to specific IP addresses and/or
   38 # ports, instead of the default. See also the 
   39 # directive.
   40 #
   41 # Change this to Listen on specific IP addresses as shown below to 
   42 # prevent Apache from glomming onto all bound IP addresses.
   43 #
   44 #Listen 12.34.56.78:80
   45 Listen 80
   46 Listen 6111
   47 Listen 6222
   48 Listen 6333
  ………………省略部分输出信息……………… 
  ```

  **第3步**：在httpd服务的配置文件中大约134行处开始，分别追加写入三个基于端口号的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
  ………………省略部分输出信息……………… 
  134 <VirtualHost 192.168.10.10:6111>
  135     DocumentRoot /home/wwwroot/6111
  136     ServerName www.linuxprobe.com
  137     <Directory /home/wwwroot/6111>
  138     AllowOverride None
  139     Require all granted
  140     </Directory> 
  141 </VirtualHost>
  
  142 <VirtualHost 192.168.10.10:6222>
  143     DocumentRoot /home/wwwroot/6222
  144     ServerName www.linuxcool.com
  145     <Directory /home/wwwroot/6222>
  146     AllowOverride None
  147     Require all granted
  148     </Directory>
  149 </VirtualHost>
  
  150 <VirtualHost 192.168.10.10:6333>
  151     DocumentRoot /home/wwwroot/6333
  152     ServerName www.linuxdown.com
  153     <Directory /home/wwwroot/6333>
  154     AllowOverride None
  155     Require all granted
  156     </Directory>
  157 </VirtualHost>
  ………………省略部分输出信息………………
  ```

  **第4步**：因为我们把网站数据目录存放在/home/wwwroot目录中，因此还是必须要正确设置网站数据目录文件的SELinux安全上下文，使其与网站服务功能相吻合。最后记得用restorecon命令让新配置的SELinux安全上下文立即生效。

  ```
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222/*
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333
  [root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333/*
  [root@linuxprobe ~]# restorecon -Rv /home/wwwroot/
  Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6111 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6111/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6222 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6222/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6333 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  Relabeled /home/wwwroot/6333/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0
  [root@linuxprobe ~]# systemctl restart httpd
  Job for httpd.service failed because the control process exited with error code.
  See "systemctl status httpd.service" and "journalctl -xe" for details.
  ```

  见鬼了！在妥当配置httpd服务程序和SELinux安全上下文并重启httpd服务后，竟然出现报错信息。这是因为SELinux服务检测到6111、6222和6333端口原本不属于Apache服务应该需要的资源，但现在却以httpd服务程序的名义监听使用了，所以SELinux会拒绝使用Apache服务使用这三个端口。我们可以使用semanage命令查询并过滤出所有与HTTP协议相关且SELinux服务允许的端口列表。

  ```
  [root@linuxprobe ~]# semanage port -l | grep http
  http_cache_port_t            tcp      8080, 8118, 8123, 10001-10010
  http_cache_port_t            udp      3130
  http_port_t                  tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
  pegasus_http_port_t          tcp      5988
  pegasus_https_port_t         tcp      5989
  ```

  **第5步**：SELinux允许的与HTTP协议相关的端口号中默认没有包含6111、6222和6333，因此需要将这三个端口号手动添加进去。该操作会立即生效，而且在系统重启过后依然有效。设置好后再重启httpd服务程序，然后就可以看到网页内容了，结果如图10-18所示。

  ```
  [root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6111
  [root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6222
  [root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6333
  [root@linuxprobe ~]# semanage port -l | grep http
  http_cache_port_t            tcp      8080, 8118, 8123, 10001-10010
  http_cache_port_t            udp      3130
  http_port_t                  tcp      6333, 6222, 6111, 80, 81, 443, 488, 8008, 8009, 8443, 9000
  pegasus_http_port_t          tcp      5988
  pegasus_https_port_t         tcp      5989
  [root@linuxprobe ~]# systemctl restart httpd
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E7%AB%AF%E5%8F%A3%E5%8F%B7%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x369.png)

  图10-18 基于端口号访问虚拟主机网站

  ##### **10.6 Apache的访问控制**

  Apache可以基于源主机名、源IP地址或源主机上的浏览器特征等信息对网站上的资源进行访问控制。它通过Allow指令允许某个主机访问服务器上的网站资源，通过Deny指令实现禁止访问。在允许或禁止访问网站资源时，还会用到Order指令，这个指令用来定义Allow或Deny指令起作用的顺序，其匹配原则是按照顺序进行匹配，若匹配成功则执行后面的默认指令。比如“Order Allow, Deny”表示先将源主机与允许规则进行匹配，若匹配成功则允许访问请求，反之则拒绝访问请求。

  **第1步**：先在服务器上的网站数据目录中新建一个子目录，并在这个子目录中创建一个包含Successful单词的首页文件。

  ```
  [root@linuxprobe ~]# mkdir /var/www/html/server
  [root@linuxprobe ~]# echo "Successful" > /var/www/html/server/index.html
  ```

  **第2步**：打开httpd服务的配置文件，在第161行后面添加下述规则来限制源主机的访问。这段规则的含义是允许使用Firefox浏览器的主机访问服务器上的首页文件，除此之外的所有请求都将被拒绝。使用Firefox浏览器的访问效果如图10-19所示，而其它浏览器访问效果如图10-20所示。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
  ………………省略部分输出信息………………
  161 <Directory "/var/www/html/server">
  162     SetEnvIf User-Agent "Firefox" ff=1
  163     Order allow,deny
  164     Allow from env=ff
  165 </Directory>
  ………………省略部分输出信息………………
  [root@linuxprobe ~]# systemctl restart httpd
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-2-1024x130.png)

  图10-19 火狐浏览器成功访问

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E5%A4%B1%E8%B4%A5-1024x115.png)

  图10-20 其它浏览器访问失败

  除了匹配源主机的浏览器特征之外，还可以通过匹配源主机的IP地址进行访问控制。例如，我们只允许IP地址为192.168.10.20的主机访问网站资源，那么就可以在httpd服务配置文件的第161行后面添加下述规则。这样在重启httpd服务程序后再用本机（即服务器，其IP地址为192.168.10.10）来访问网站的首页面时就会提示访问被拒绝了，如图10-21所示。

  ```
  [root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
  ………………省略部分输出信息………………
  161 <Directory "/var/www/html/server">
  162     Order allow,deny 
  163     Allow from 192.168.10.20
  164 </Directory>
  ………………省略部分输出信息………………
  [root@linuxprobe ~]# systemctl restart httpd
  [root@linuxprobe ~]# firefox
  ```

  ![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E8%A2%AB%E6%8B%92%E7%BB%9D-1-1024x174.png)图10-20 因IP地址不符合要求而被拒绝访问

  **出现问题?大胆提问!**

  > 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
  >
  > Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
  >
  > *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

  **本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

  1．什么是Web网络服务？

  **答：**一种允许用户通过浏览器访问到互联网中各种资源的服务。

  2．相较于Nginx服务程序，Apache服务程序最大的优势是什么？

  **答：**Apache服务程序具备跨平台特性、安全性，而且拥有快速、可靠、简单的API扩展。

  3．httpd服务程序没有检查到首页文件，会提示报错信息吗？

  **答：**不会，httpd服务在未找到网站首页文件时，会向访客显示一个默认页面。

  4．简述Apache服务主配置文件中全局配置参数、区域配置参数和注释信息的作用。

  **答：**全局配置参数是一种全局性的配置参数，可作用于对所有的子站点；区域配置参数则是单独针对于每个独立的子站点进行设置的；而注释信息一般是对服务程序的功能或某一行参数进行介绍。

  5．简述SELinux服务的作用。

  **答：**为了让各个服务进程都受到约束，使其仅获取到本应获取的资源。

  6．在使用getenforce命令查看SELinux服务模式时，发现其配置模式为permissive，这代表强制开启模式吗？

  **答：**不是，强制开启模式是enforcing，而permissive是只发出警告而不强制拦截的模式。

  7．在使用semanage命令修改了文件上应用的SELinux安全上下文后，还需要执行什么命令才可以让更改立即生效？

  **答：**还需要restorecon命令即可让新的SELinux安全上下文参数立即生效。

  8．要想查询并过滤出所有与HTTP协议相关的SELinux域策略有哪些，应该怎么做呢？

  **答：**可以结合管道符来实现，即执行getsebool -a | grep http命令。

  9． Apache服务程序可以基于哪些资源来创建虚拟主机网站呢？

  **答：**可以基于IP地址、主机名（域名）或者端口号创建虚拟主机网站。

  10．相对于基于IP地址和基于主机名（域名）配置的虚拟主机网站来说，使用端口号配置虚
  拟主机网站有哪些特点？

  **答：**在使用端口号来配置虚拟主机网站时，必须要考虑到SELinux域对httpd服务程序所用端口号的控制策略，还要在httpd服务程序的主配置文件中使用Listen参数来开启要监听的端口号。

#### 动态web服务器部署

- ##### LNMP架构

  **章节概述：**

  LNMP动态网站部署架构是一套由[Linux](https://www.linuxprobe.com/) + Nginx + MySQL + PHP组成的动态网站系统解决方案，具有免费、高效、扩展性强且资源消耗低等优良特性，目前正在被广泛使用。本章首先对比了使用源码包安装服务程序与使用RPM软件包安装服务程序的区别，然后讲解了如何手工编译源码包并安装各个服务程序，以及使用最受欢迎的WordPress博客系统验证架构环境。

  本章是本书的最后一章内容，[刘遄](https://www.linuxprobe.com/)老师不仅希望各位读者在学完本书之后，能够顺利找到满意的高薪工作，也希望您能利用书中所学知识搭建自己的博客或论坛系统，并以此为平台，将自己工作中积攒的Linux经验以及技巧分享给更多人，为美好的开源世界贡献自己的力量。**See you later!**

  本章目录结构

  - [20.1 源码包程序](https://www.linuxprobe.com/basic-learning-20.html#201)
  - 20.2 LNMP动态网站架构
    - [20.2.1 配置Nginx服务](https://www.linuxprobe.com/basic-learning-20.html#2021_Nginx)
    - [20.2.2 配置Mysql服务](https://www.linuxprobe.com/basic-learning-20.html#2022_Mysql)
    - [20.2.3 配置php服务](https://www.linuxprobe.com/basic-learning-20.html#2023_php)
  - [20.3 搭建Discuz论坛](https://www.linuxprobe.com/basic-learning-20.html#203_Discuz)
  - [20.4 选购服务器主机](https://www.linuxprobe.com/basic-learning-20.html#204)

  ##### **20.1 源码包程序**

  本书第1章中曾经讲到，在RPM[红帽](https://www.linuxprobe.com/)软件包管理器技术出现之前，[Linux系统](https://www.linuxprobe.com/)运维人员只能通过源码包的方式来安装各种服务程序，这是一件非常繁琐且极易消耗时间与耐心的事情；而且在安装、升级、卸载程序时还要考虑到与其他程序或函数库的相互依赖关系，这就要求运维人员不仅要掌握更多的Linux系统理论知识以及高超的实操技能，还需要有极好的耐心才能安装好一个源码软件包。考虑到本书的读者都是刚入门或准备入门的运维新人，因为本书在前面的章节中一直都是采用软件仓库的方式来安装服务程序。但是，现在依然有很多软件程序只有源码包的形式，如果我们只会使用dnf[命令](https://www.linuxcool.com/)来安装程序，则面对这些只有源码包的软件程序时，将充满无力感，要么需要等到第三方组织将这些软件程序编写成RPM软件包之后再行使用，要么就只能寻找相关软件程序的替代品了（而且替代软件还必须具备RPM软件包的形式）。由此可见，如果运维人员只会使用软件仓库来安装服务程序，将会形成知识短板，对日后的运维工作带来不利。

  本着不能让自己的读者在运维工作中吃亏的想法，[刘遄](https://www.linuxprobe.com/)老师接下来会详细讲解如何使用源码包的方式来安装服务程序。

  其实，使用源码包来安装服务程序具有两个优势。

  > 源码包的可移植性非常好，几乎可以在任何Linux系统中安装使用，而RPM软件包是针对特定系统和架构编写的指令集，必须严格地符合执行环境才能顺利安装（即只会去“生硬地”安装服务程序）。
  >
  > 使用源码包安装服务程序时会有一个编译过程，因此能够更好地适应安装主机的系统环境，运行效率和优化程度都会强于使用RPM软件包安装的服务程序。也就是说，可以将采用源码包安装服务程序的方式看作是针对系统的“量体裁衣”。

  一般来讲，在安装软件时，如果能通过软件仓库来安装，就用dnf[命令](https://www.linuxcool.com/)搞定它；反之则去寻找合适的RPM软件包来安装；如果是在没有资源可用，那就只能使用源码包来安装了。使用源码包安装服务程序的过程看似复杂，其实在归纳汇总后只需要4～5个步骤即可完成安装。接下来会对每一个步骤进行详解。

  需要提前说明的是，在使用源码包安装程序时，会输出大量的过程信息，这些信息的意义并不大，因此本章会省略这部分输出信息而不作特殊备注，请大家在具体操作时以实际为准。

  **第1步**：下载及解压源码包文件。为了方便在网络中传输，源码包文件通常会在归档后使用gzip或bzip2等格式进行压缩，因此一般会具有.tar.gz与.tar.bz2的后缀。要想使用源码包安装服务程序，必须先把里面的内容解压出来，然后再切换到源码包文件的目录中：

  > [root@linuxprobe ~]# tar xzvf FileName**.tar.gz**
  >
  > [root@linuxprobe ~]# cd FileDirectory

  **第2步**：编译源码包代码。在正式使用源码包安装服务程序之前，还需要使用编译[脚本](https://www.linuxcool.com/)针对当前系统进行一系列的评估工作，包括对源码包文件、软件之间及函数库之间的依赖关系、编译器、汇编器及连接器进行检查。我们还可以根据需要来追加--prefix参数，以指定稍后源码包程序的安装路径，从而对服务程序的安装过程更加可控。当编译工作结束后，如果系统环境符合安装要求，一般会自动在当前目录下生成一个Makefile安装文件。

  > [root@linuxprobe ~]# ./configure --prefix=/usr/local/program

  **第3步**：生成二进制安装程序。刚刚生成的Makefile文件中会保存有关系统环境、软件依赖关系和安装规则等内容，接下来便可以使用make命令来根据Makefile文件内容提供的合适规则编译生成出真正可供用户安装服务程序的二进制可执行文件了。

  > [root@linuxprobe ~]# make

  **第4步**：运行二进制的服务程序安装包。由于不需要再检查系统环境，也不需要再编译代码，因此运行二进制的服务程序安装包应该是速度最快的步骤。如果在源码包编译阶段使用了--prefix参数，那么此时服务程序就会被安装到那个目录，如果没有自行使用参数定义目录的话，一般会被默认安装到/usr/local/bin目录中。

  > [root@linuxprobe ~]# make install

  **第5步**：清理源码包临时文件。由于在安装服务程序的过程中进行了代码编译的工作，因此在安装后目录中会遗留下很多临时垃圾文件，本着尽量不要浪费磁盘存储空间的原则，可以使用make clean命令对临时文件进行彻底的清理工作。

  > [root@linuxprobe ~]# make clean

  估计有读者会有疑问，为什么通常是安装一个服务程序，源码包的编译工作（configure）与生成二进制文件的工作（make）会使用这么长的时间，而采用RPM软件包安装就特别有效率呢？其实原因很简单，在RHCA认证的RH401考试中，会要求考生写一个RPM软件包。刘遄老师会在本书的进阶篇中讲到，其实RPM软件包就是把软件的源码包和一个针对特定系统、架构、环境编写的安装规定打包成一起的指令集，因此为了让用户都能使用这个软件包来安装程序，通常一个软件程序会发布多种格式的RPM软件包（例如i386、x86_64等架构）来让用户选择。而源码包的软件作者肯定希望自己的软件能够被安装到更多的系统上面，能够被更多的用户所了解、使用，因此便会在编译阶段（configure）来检查用户当前系统的情况，然后制定出一份可行的安装方案，所以会占用很多的系统资源，需要更长的等待时间。

  ##### **20.2 LNMP动态网站架构**

  LNMP动态网站部署架构是一套由Linux + Nginx + MySQL + PHP组成的动态网站系统解决方案。LNMP中的字母L是Linux系统的意思，不仅可以是RHEL、CentOS、Fedora，还可以是Debian、Ubuntu等系统。本书的配套站点[https://www.linuxprobe.com](https://www.linuxprobe.com/)就是基于LNMP部署出来的，目前的运行一直很稳定，访问速度也很快。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2015/11/lnmp%E6%9E%B6%E6%9E%84%E5%9B%BE%E7%89%87.jpg)

  在使用源码包安装服务程序之前，首先要让安装主机具备编译程序源码的环境，他需要具备C语言、C++语言、Perl语言的编译器，以及各种常见的编译支持函数库程序。因此请先配置妥当软件仓库，然后把下面列出的这些软件包都统统安装上：

  ```
  [root@linuxprobe ~]# dnf -y install apr* autoconf automake numactl bison bzip2-devel cpp curl-devel fontconfig-devel freetype-devel gcc gcc-c++ gd-devel gettext-devel kernel-headers keyutils-libs-devel krb5-devel libcom_err-devel  libpng-devel  libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* libxml2-devel libXpm* libxml* libXaw-devel libXmu-devel libtiff* make openssl-devel patch pcre-devel perl php-common php-gd telnet zlib-devel libtirpc-devel gtk* ntpstat na* bison* lrzsz cmake ncurses-devel libzip-devel libxslt-devel gdbm-devel readline-devel gmp-devel
  
  Updating Subscription Management repositories.
  Unable to read consumer identity
  AppStream                                       3.1 MB/s | 3.2 kB     00:00    
  BaseOS                                          2.0 MB/s | 2.7 kB     00:00    
  ………………省略部分输出信息………………
    Running scriptlet: mariadb-connector-c-3.0.7-1.el8.x86_64                 1/1 
    Preparing        :                                                        1/1 
    Installing       : xorg-x11-proto-devel-2018.4-1.el8.noarch             1/261 
    Installing       : perl-version-6:0.99.24-1.el8.x86_64                  2/261 
    Installing       : zlib-devel-1.2.11-10.el8.x86_64                      3/261 
    Installing       : perl-Time-HiRes-1.9758-1.el8.x86_64                  4/261 
    Installing       : libpng-devel-2:1.6.34-5.el8.x86_64                   5/261 
    Installing       : perl-CPAN-Meta-Requirements-2.140-396.el8.noarch     6/261 
    Installing       : perl-ExtUtils-ParseXS-1:3.35-2.el8.noarch            7/261 
    Installing       : perl-ExtUtils-Manifest-1.70-395.el8.noarch           8/261 
    Installing       : cmake-filesystem-3.11.4-3.el8.x86_64                 9/261 
    Installing       : perl-Test-Harness-1:3.42-1.el8.noarch               10/261 
    Installing       : perl-Module-CoreList-1:5.20181130-1.el8.noarch      11/261 
    Installing       : perl-Module-Metadata-1.000033-395.el8.noarch        12/261 
    Installing       : perl-SelfLoader-1.23-416.el8.noarch                 13/261 
    Installing       : perl-Perl-OSType-1.010-396.el8.noarch               14/261 
    Installing       : perl-Module-Load-1:0.32-395.el8.noarch              15/261 
    Installing       : perl-JSON-PP-1:2.97.001-3.el8.noarch                16/261 
    Installing       : perl-Filter-2:1.58-2.el8.x86_64                     17/261 
    Installing       : perl-Compress-Raw-Zlib-2.081-1.el8.x86_64           18/261 
    Installing       : perl-encoding-4:2.22-3.el8.x86_64                   19/261 
    Installing       : perl-Text-Balanced-2.03-395.el8.noarch              20/261 
  ………………省略部分输出信息………………
  Complete!
  ```

  如果条件允许，建议适当增加虚拟机的内存上限，让稍后的编译过程快一些。并且由于接下来还需要从外部网络中获取Nginx、Mysql、PHP及WordPress等一系列的安装包，因此需要配置虚拟机联网。

  将已经调整为桥接模式的网卡，通过nmtui或nm-connection-editor命令修改为DHCP自动获取网卡信息模式，大多数情况下就可以接入互联网了，如图20-1所示。如若不可访问互联网，则优先考虑是否外部环境有特殊的限制，则将虚拟机内网卡配置成跟物理机一致即可。

  ```
  [root@linuxprobe ~]# ping -c 4 www.linuxprobe.com
  PING www.linuxprobe.com.w.kunlunno.com (202.97.231.16) 56(84) bytes of data.
  64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=1 ttl=55 time=27.5 ms
  64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=2 ttl=55 time=27.10 ms
  64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=3 ttl=55 time=27.4 ms
  64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=4 ttl=55 time=28.9 ms
  
  --- www.linuxprobe.com.w.kunlunno.com ping statistics ---
  4 packets transmitted, 4 received, 0% packet loss, time 8ms
  rtt min/avg/max/mdev = 27.354/27.913/28.864/0.593 ms
  ```

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%8D%A1%E6%A8%A1%E5%BC%8F.png)

  图20-1 设置网卡为DHCP自动获取模式

  刘遄老师已经把安装LNMP动态网站部署架构所需的4个软件源码包和1个用于检查效果的论坛网站系统软件包上传到与本书配套的站点服务器上。大家可以在Windows系统中下载后通过ssh服务传送到打算部署LNMP动态网站架构的Linux服务器中，也可以直接在Linux服务器中使用wget命令下载这些源码包文件。为了更好的找到它们，统一放到/lnmp目录下保存：

  ```
  [root@linuxprobe ~]# mdir /lnmp
  [root@linuxprobe ~]# cd /lnmp
  [root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/rpcsvc-proto-1.4.tar.gz
  [root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/nginx-1.16.0.tar.gz
  [root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/mysql-8.0.18.tar.xz
  [root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/php-7.3.5.tar.gz
  [root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/wordpress.tar.gz
  [root@linuxprobe lnmp]# ls
  rpcsvc-proto-1.4.tar.gz       nginx-1.16.0.tar.gz   mysql-8.0.18.tar.xz
  php-7.3.5.tar.gz              wordpress.tar.gz
  ```

  将相关软件上传至书籍配套网站的目地是保证读者100%可找到它们，顺利的进行实验。但由于受到服务器或网络的限制，下载速度若不能达到读您的预期，亦可从互联网中下载同版本软件。

  百度网盘打包下载链接：https://pan.baidu.com/s/1GRXh4E92OEi19-caz3QJ_w  提取码：**ghi0**

  小试牛刀，rpcsvc-proto是一款用于包含rcpsvc协议文件的支持软件包名称，在后续Nginx与Mysql服务程序的部署过程中都需要被调用到。而要想通过源码包安装服务程序，就一定要严格遵守上面总结的安装步骤—下载及解压源码包文件、编译源码包代码、生成二进制安装程序、运行二进制的服务程序安装包。接下来在解压、编译各个软件包源码程序时，都会生成大量的输出信息，下文中将其省略，请读者以实际操作为准。

  ```
  [root@linuxprobe lnmp]# tar xzvf rpcsvc-proto-1.4.tar.gz 
  [root@linuxprobe lnmp]# cd rpcsvc-proto-1.4/
  [root@linuxprobe rpcsvc-proto-1.4]# ./configure
  [root@linuxprobe rpcsvc-proto-1.4]# make 
  [root@linuxprobe rpcsvc-proto-1.4]# make install
  [root@linuxprobe rpcsvc-proto-1.4]# cd ..
  [root@linuxprobe lnmp]#
  ```

  本章节内涉及软件较多，频繁切换工作目录在所难免，一方面我们会在每次操作后尽可能的返回到/lnmp目录下待命，一方面也请读者们仔细看清所在目录路径，避免找不到文件而影响学习心情~

  ###### **20.2.1 配置Nginx服务**

  Nginx是一款相当优秀的用于部署动态网站的轻量级服务程序，它最初是为俄罗斯门户站点而开发的，因其稳定性、功能丰富、占用内存少且并发能力强而备受用户的信赖。目前国内诸如新浪、网易、腾讯等门户站点均已使用了此服务。

  Nginx服务程序的稳定性源自于采用了分阶段的资源分配技术，降低了CPU与内存的占用率，所以使用Nginx程序部署的动态网站环境不仅十分稳定、高效，而且消耗的系统资源也很少。此外，Nginx具备的模块数量与Apache具备的模块数量几乎相同，而且现在已经完全支持proxy、rewrite、mod_fcgi、ssl、vhosts等常用模块。更重要的是，Nginx还支持热部署技术，7×24不间断提供服务，还可以在不暂停服务的情况下直接对Nginx服务程序进行升级。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2015/11/Nginx%E4%B8%8EApache.png)

  坦白来讲，虽然Nginx程序的代码质量非常高，代码很规范，技术成熟，模块扩展也很容易，但依然存在不少问题，比如是由俄罗斯人开发的，所以在资料文档方面还并不完善，中文资料的质量更是鱼龙混杂。但是Nginx服务程序在近年来增长势头迅猛，相信会在轻量级Web服务器市场具有不错的未来。

  第1步：创建用于管理网站服务的系统账户。这是在Linux系统创建之初就刻入的基因片段，为了能够让操作系统更加安全，让不同的系统用户管理对应的服务程序。这样即便有骇客通过网站服务侵入了服务器，也无法提权到更高权限，或是对系统进行更大的破坏，甚至都无法登陆ssh协议，因为仅仅拿到的是一个系统账号而已。不同于以往，此次新建的账户应使用“-M”参数不创建对应的家目录，以及用“-s”指定登录Shell解释器为/sbin/nologin，让任何人都不能通过这个账号登陆到主机。

  ```
  [root@linuxprobe lnmp]# useradd nginx -M -s /sbin/nologin
  [root@linuxprobe lnmp]# id nginx
  uid=1001(nginx) gid=1001(nginx) groups=1001(nginx)
  ```

  第2步：编译安装Nginx网站服务程序。为了能够让网站服务支持更多的功能，我们会在编译过程中添加额外的参数，较为重要的是用“prefix”参数指定服务将被安装到哪个目录，方便后面找到和调用它。其次是考虑到HTTPS协议越来越被广泛使用，所以用“with-http_ssl_module”参数来开启Nginx服务的SSL加密模块，便于日后开启HTTPS协议功能：

  ```
  [root@linuxprobe lnmp]# tar zxvf nginx-1.16.0.tar.gz
  [root@linuxprobe lnmp]# cd nginx-1.16.0/
  [root@linuxprobe nginx-1.16.0]# ./configure --prefix=/usr/local/nginx --with-http_ssl_module 
  [root@linuxprobe nginx-1.16.0]# make 
  [root@linuxprobe nginx-1.16.0]# make install
  [root@linuxprobe nginx-1.16.0]# cd ..
  ```

  相对来说，configure编译脚本文件比make生成二进制文件命令要快，而make install安装程序则一般是最快的，相当于双击运行二进制安装包的操作。在编译、生成、安装三阶段中，屏幕上会输出各式各样的信息，主要包含软件包的概要情况、当前系统的软件依赖关系、及是否有条件进行安装操作。但只要进程没有被强制终止，或是输出明显报错信息，全是正常情况。

  第3步：安装完毕后进行最终配置阶段。既然在编译环境中使用“prefix”参数指定了安装路径，那么Nginx服务程序配置文件一定会乖乖的在/usr/local/nginx目录中等着您的。

  我们总共要进行三处修改，首先是把第2行的注释符（#）删除，然后在后面写上负责运行网站服务程序的账户名称和用户组名称，设定由nginx用户及nginx用户组负责管理网站服务，让网站服务能够顺利的被系统所读取：

  ```
  [root@linuxprobe lnmp]# vim /usr/local/nginx/conf/nginx.conf 
    1 
    2 user  nginx nginx;
  ```

  其次是修改第45行的首页文件名称，添加上index.php的名字，也就是让用户浏览网站时第一眼所看到的文件，也叫首页文件。

  ```
   43         location / {
   44             root   html;
   45             index  index.php index.html index.htm;
   46         }
  ```

  最后再删除第65至71行前面的注释符（#）来启用虚拟主机功能，将第69行后面对应的网站根目录修改为“/usr/local/nginx/html”，fastcgi_script_name参数用于指代脚本名称，也就是用户请求的URL。正确填写方能使Nginx服务解析用户请求，否则访问的页面都会提示404未找到的错误。

  ```
   63         # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000 64         # 65         location ~ \.php$ { 66             root           html; 67             fastcgi_pass   127.0.0.1:9000; 68             fastcgi_index  index.php; 69             fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html$fastcgi_script_name; 70             include        fastcgi_params; 71         }
  ```

  第4步：通过编译源码方式安装的服务默认不能够被systemctl命令所管理，而要用Nginx服务本身的管理工具进行操作，对应的命令目录是/usr/local/nginx/sbin。但使用绝对路径的形式输入命令未免会显得麻烦，建议将/usr/local/nginx/sbin路径加入到PATH变量中，让Bash解释器在后续执行命令时自动的搜索到它。source命令加载配置文件让参数立即生效，下次就只需要输入“nginx”命令即可启动网站服务了，很方便呢~

  ```
  [root@linuxprobe lnmp]# vim ~/.bash_profile# .bash_profile# Get the aliases and functionsif [ -f ~/.bashrc ]; then	. ~/.bashrcfi# User specific environment and startup programsPATH=$PATH:$HOME/bin:/usr/local/nginx/sbinexport PATH[root@linuxprobe lnmp]# source ~/.bash_profile[root@linuxprobe lnmp]# nginx 
  ```

  操作完毕！重启服务程序~并在浏览器中输入本机的IP地址，即可浏览到Nginx网站服务程序的默认界面，如图20-2所示。相较于Apache服务程序的红色默认页面，Nginx服务程序的默认页面显得更加简洁。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/%E8%AE%BF%E9%97%AE%E7%BD%91%E7%AB%99%E6%88%90%E5%8A%9F-1.png)

  图20-2 Nginx服务程序的默认页面

  **出现问题?大胆提问!**

  > 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
  >
  > Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
  >
  > *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

  ###### **20.2.2 配置Mysql服务**

  本书在第18章讲解过MySQL和MariaDB数据库管理系统之间的因缘和特性，也狠狠地夸奖了MariaDB数据库，但是MySQL数据库当前依然是生产环境中最常使用的关系型数据库管理系统之一，坐拥极大的市场份额，并且已经通过十几年不断的发展向业界证明了自身的稳定性和安全性。另外，虽然第18章已经讲解了基本的数据库管理知识，但是为了进一步帮助大家夯实基础，本章依然在这里整合了MySQL数据库内容，使大家在温故的同时知新。
  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/mysql-300x155.png)

  在使用软件仓库安装服务程序时，系统会自动根据RPM软件包中的指令集完整软件配置等工作。但是一旦选择使用源码包的方式来安装，这一切就需要自己来完成了。针对MySQL数据库来讲，我们需要在系统中创建一个名为mysql的用户，专门用于负责运行MySQL数据库。请记得要把这类账户的Bash终端设置成nologin解释器，避免黑客通过该用户登录到服务器中，从而提高系统安全性。

  ```
  [root@linuxprobe lnmp]# useradd mysql -M -s /sbin/nologin
  ```

  第1步：解压MySQL安装软件包。将解压出的程序目录改名并移动到/usr/local目录下，对其进行初始化操作后便可使用。但请注意.tar.xz结尾的压缩包软件，不应用“z”参数进行解压。

  ```
  [root@linuxprobe lnmp]# tar xvf mysql-8.0.18.tar.xz[root@linuxprobe lnmp]# mv mysql-8.0.18-linux-glibc2.12-x86_64 mysql[root@linuxprobe lnmp]# mv mysql /usr/local
  ```

  第2步：生产环境中管理MySQL数据库时，有两个比较常用的目录。其一是/usr/local/mysql目录，这是用于保存MySQL数据库程序文件的路径。还有一个是/usr/local/mysql/data目录，它用于存储数据库的具体内容，每个数据库的内容会被单独存放到一个目录内。对于存放实际数据库文件的data目录，用户需要先手动创建出来：

  ```
  [root@linuxprobe lnmp]# cd /usr/local/mysql[root@linuxprobe mysql]# mkdir data
  ```

  第3步：初始化MySQL服务程序，对目录进行授权，保证数据能够被mysql系统用户所读取。在初始化阶段，应使用mysqld命令确认管理MySQL数据库服务的用户名称、数据保存目录及编码信息。信息确认无误后会初始化完毕，在最后一段会给予用户初始化临时密码，一定要保存好，例如本书所分配的密码为qfroRs,Ei4Ls。

  ```
  [root@linuxprobe mysql]# chown -R mysql:mysql /usr/local/mysql[root@linuxprobe mysql]# cd bin[root@linuxprobe bin]# ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data2021-05-06T07:07:06.243270Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.18) initializing of server in progress as process 76062021-05-06T07:07:08.116268Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qfroRs,Ei4Ls
  ```

  第4步：与Nginx服务相似，MySQL数据库的二进制可执行命令也是单独存放在自身程序目录中的，每一次都切换到/usr/local/mysql/bin目录中再执行着实有些麻烦，要能也加入到PATH变量中可就方便太多了。说干就干：

  ```
  [root@linuxprobe bin]# vim ~/.bash_profile# .bash_profile# Get the aliases and functionsif [ -f ~/.bashrc ]; then        . ~/.bashrcfi# User specific environment and startup programsPATH=$PATH:$HOME/bin:/usr/local/nginx/sbin:/usr/local/mysql/binexport PATH[root@linuxprobe bin]# source ~/.bash_profile
  ```

  这样设置后，即便返回到了源码目录，也可以继续执行MySQL数据库的管理命令。不过先别着急！既然是手动安装服务，那么让文件“归位”的重任就只得亲力亲为了——将启动脚本mysql.server放入到/etc/init.d目录中，让服务器每次重启后都能自动启动数据库，并给予可执行权限。

  libtinfo.so.5文件是MySQL数据库在8.0版本后重要的函数库文件，需要将libtinfo.so.6.1文件复制或者作为链接文件才能正常启动：

  ```
  [root@linuxprobe bin]# cd /usr/local/mysql[root@linuxprobe mysql]# cp -a support-files/mysql.server /etc/init.d/[root@linuxprobe mysql]# chmod a+x /etc/init.d/mysql.server[root@linuxprobe mysql]# ln -s /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5 
  ```

  第5步：执行MySQL数据库服务启动文件，并进行初始化工作。8.0版本以后为了安全着想，不再允许用户使用临时密码管理数据库内容，也不能远程控制，用户必须修改初始化密码后才能使用。数据库作为系统重要的组成服务，密码不建议少于20位，例如想修改为“PObejCBeDzTRCncXwgBy”。

  ```
  [root@linuxprobe mysql]# /etc/init.d/mysql.server start Starting MySQL.Logging to '/usr/local/mysql/data/linuxprobe.com.err'.. SUCCESS! [root@linuxprobe mysql]# mysql -u root -pEnter password: 输入初始化时给的原始密码Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 8Server version: 8.0.18Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.mysql> alter user 'root'@'localhost' identified by 'PObejCBeDzTRCncXwgBy'; Query OK, 0 rows affected (0.01 sec)mysql> 
  ```

  但这样还是不行，需要继续切换至mysql数据库中，修改user表单的密码值。这也是从MySQL数据库8.0版本后才开始的新安全要求，看过《Linux就该这么学》RHEL7版本的老读者应该记得5.6版本时就没有这么麻烦。

  ```
  mysql> use mysql;Reading table information for completion of table and column namesYou can turn off this feature to get a quicker startup with -ADatabase changedmysql> show tables;+---------------------------+| Tables_in_mysql           |+---------------------------+| columns_priv              || tables_priv               || time_zone                 || time_zone_leap_second     || time_zone_name            || time_zone_transition      || time_zone_transition_type || user                      || …………省略部分输出信息…………  |+---------------------------+33 rows in set (0.00 sec)mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PObejCBeDzTRCncXwgBy';Query OK, 0 rows affected (0.01 sec)
  ```

  由于稍后将在20.4小节安装部署WordPress网站系统，因此现在需要提前把数据库新建出来：

  ```
  mysql> create database linuxcool;Query OK, 1 row affected (0.00 sec)mysql> exitBye
  ```

  ###### **20.2.3 配置php服务**

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/PHP.png)PHP（Hypertxt Preprocessor，超文本预处理器）是一种通用的开源脚本语言，发明于1995年，它吸取了C语言、Java语言及Perl语言的很多优点，具有开源、免费、快捷、跨平台性强、效率高等优良特性，是目前Web开发领域最常用的语言之一。使用源码包的方式编译安装PHP语言环境其实并不复杂，难点在于解决PHP的程序包和其他软件的依赖关系。

  第1步：解压php安装包软件并编译安装。编译中需要使用“prefix”参数指定安装路径，使用“--with-mysqli”等命令开启对数据库的支持模块，为后面在线安装网站夯实基础。

  ```
  [root@linuxprobe mysql]# cd /lnmp[root@linuxprobe lnmp]# tar xvf php-7.3.5.tar.gz[root@linuxprobe lnmp]# cd php-7.3.5/[root@linuxprobe php-7.3.5]# ./configure --prefix=/usr/local/php --enable-fpm --with-mysqli --with-curl --with-pdo_mysql --with-pdo_sqlite --enable-mysqlnd --enable-mbstring --with-gd
  ```

  生成二进制文件和安装依然为下述命令，时间大约为10~20分钟，耐心等待即可：

  ```
  [root@linuxprobe php-7.3.5]# make[root@linuxprobe php-7.3.5]# make install
  ```

  第2步：将生成出的php服务配置文件复制到安装目录中（/usr/local/php/），让其生效。主配置文件有了，接下来还需要php-fpm的配置文件，好在也已经提供，只需要复制模板即可：

  ```
  [root@linuxprobe php-7.3.5]# cp php.ini-development /usr/local/php/lib/php.ini[root@linuxprobe php-7.3.5]# cd /usr/local/php/etc/[root@linuxprobe etc]# mv php-fpm.conf.default php-fpm.conf
  ```

  复制一个模板文件到php-fpm.d的目录中，用于后续控制网站的连接性能：

  ```
  [root@linuxprobe etc]# mv php-fpm.d/www.conf.default php-fpm.d/www.conf
  ```

  第3步：把php服务加入到启动项中，使其重启后依然生效：

  ```
  [root@linuxprobe etc]# cd /lnmp/php-7.3.5[root@linuxprobe php-7.3.5]# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm[root@linuxprobe php-7.3.5]# chmod 755 /etc/init.d/php-fpm
  ```

  第4步：由于php服务程序的配置参数直接会影响到Web服务服务的运行环境，如果默认开启了一些不必要且高危的功能（如允许用户在网页中执行Linux命令），则会降低网站被入侵的难度，入侵人员甚至可以拿到整台Web服务器的管理权限。因此我们需要编辑php.ini配置文件，在310行的disable_functions参数后面追加上要禁止的功能。下面的禁用功能名单是刘遄老师依据网站运行的经验而定制的，不见得适合每个生产环境，建议大家在此基础上根据自身工作需求酌情删减：

  ```
  [root@linuxprobe php-7.3.5]# vim /usr/local/php/lib/php.ini 307 ; This directive allows you to disable certain functions for security reasons. 308 ; It receives a comma-delimited list of function names. 309 ; http://php.net/disable-functions 310 disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
  ```

  第5步：LNMP架构源码编译工作就此结束~享受胜利成果吧：

  ```
  [root@linuxprobe php-7.3.5]# /etc/init.d/php-fpm startStarting php-fpm done
  ```

  ##### **20.3 搭建Discuz论坛**

  为了检验LNMP动态网站架构环境是否配置妥当，可以在上面部署WordPress博客系统，然后查看效果。如果能够在LNMP动态网站环境中成功安装使用WordPress网站系统，也就意味着这套架构是可用的。WordPress是一个以PHP和MySQL为平台的开源博客软件，具有丰富的插件和模板系统，截止于2021年5月，全球排名前1000万的网站中已有超过41%使用了WordPress，是当前最受欢迎的网站内容管理系统。

  把Nginx服务程序根目录的内容清空后，将WordPress解压后的网站文件复制进去：

  ```
  [root@linuxprobe php-7.3.5]# cd ..[root@linuxprobe lnmp]# rm -f /usr/local/nginx/html/*[root@linuxprobe lnmp]# tar xzvf wordpress.tar.gz [root@linuxprobe lnmp]# mv wordpress/* /usr/local/nginx/html/
  ```

  为了能够让网站文件被Nginx服务程序顺利读取，应设置目录所有身份及可读写的权限：

  ```
  [root@linuxprobe lnmp]# chown -Rf nginx:nginx /usr/local/nginx/html [root@linuxprobe lnmp]# chmod -Rf 777 /usr/local/nginx/html
  ```

  随后输入本机IP地址访问网站首页，如图20-3所示，WordPress网站系统的安装界面提醒了用户稍后需要的安装信息。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/1-1-1024x624.png)

  图20-3 WordPress网站系统安装首页面

  在安装界面应依次输入刚刚建立的数据库名称、账号及重置过的密码值，WordPress要求用户自行创建好数据库，因此请确保网页中填写的数据库名称与刚刚创建的一致，如图20-4所示。点击“提交”按钮进行确认后，便可进行最终的安装阶段，如图20-5所示。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/2-1-1024x624.png)

  图20-4 填写安装信息

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/3-1-1024x624.png)

  图20-5 确认安装WordPress网站系统

  顺利安装完毕后，WordPress网站系统会向用户寻求站点标题、用户名及密码等信息，如图20-6所示。这些信息均可自行填写，密码稍微复杂一些为好，检查无误后即可点击“安装WordPress”按钮进行确认。成功的界面如图20-7所示。

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/4-1024x624.png)

  图20-6 填网站标题及管理员名称

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/5-1024x624.png)

  图20-7 安装完成界面

  WordPress的登录界面将在用户点击登录后自动显现，如图20-8所示以填写账号及密码吗，然后点击“登录”按钮：

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/6-1024x624.png)

  图20-8 填写网站账号和密码

  顺利进入到管理后台中，如图20-9所示。读者一定会很好奇，WordPress作为最热门的网站内容管理系统，都能做出什么样的网站呢~感兴趣的同学可以进一步访问我们的随书配套站点，三个风格迥异的站点，但都是基于WordPress搭建的呢~

  > 《Linux就该这么学》最新版电子书：[https://www.linuxprobe.com](https://www.linuxprobe.com/)
  >
  > Linux命令大全：[https://www.linuxcool.com](https://www.linuxcool.com/)
  >
  > Linux系统大全：[https://www.linuxdown.com](https://www.linuxdown.com/)

  ![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/7-1024x624.png)图20-9 WordPress管理后台

  看到这个界面，刘遄老师心中真是五味杂陈，从2015年写书创业至今，它见证了我们梦想成真。建议对未来有期许的同学也动手搭建出一个属于自己的网站，不仅可以让好内容帮助到更多人，也或许能够帮助您实现梦想呢~

  ##### **20.4 选购服务器主机**

  我们日常访问的网站是由域名、网站源程序和主机共同组成的，其中，主机则是用于存放网页源代码并能够把网页内容展示给用户的服务器。在本书即将结束之际，再啰嗦几句有关服务器主机的知识以及选购技巧，这些技巧都是在近几年做网站时总结出来的，希望能对大家有所帮助。

  **虚拟主机**：在一台服务器中划分一定的磁盘空间供用户放置网站信息、存放数据等；仅提供基础的网站访问、数据存放与传输功能；能够极大地降低用户费用，也几乎不需要用户来维护网站以外的服务；适合小型网站。

  **VPS（Virtual Private Server，虚拟专用服务器）**：在一台服务器中利用OpenVZ、Xen或KVM等虚拟化技术模拟出多台“主机”（即VPS），每个主机都有独立的IP地址、操作系统；不同VPS之间的磁盘空间、内存、CPU、进程与系统配置完全隔离，用户可自由使用分配到的主机中的所有资源，为此需要具备一定的维护系统的能力；适合小型网站。

  **ECS（Elastic Compute Service，云服务器）**：是一种整合了计算、存储、网络，能够做到弹性伸缩的计算服务；使用起来与VPS几乎一样，差别是云服务器是建立在一组集群服务器中，每个服务器都会保存一个主机的镜像（备份），从而大大提升了安全性和稳定性；另外还具备灵活性与扩展性；用户只需按使用量付费即可；适合大中小型网站。

  **独立服务器**：这台服务器仅提供给用户一个人使用，其使用方式分为租用方式与托管方式。租用方式是用户将服务器的硬件配置要求告知IDC服务商，按照月、季、年为单位来租用它们的硬件设备。这些硬件设备由IDC服务商的机房负责维护，用户一般需要自行安装相应的软件并部署网站服务，这减轻了用户在硬件设备上的投入，适合大中型网站。托管方式则是用户需要自行购置服务器硬件设备，并将其交给IDC服务供应商进行管理（需要缴纳管理服务费）。用户对服务器硬件配置有完全的控制权，自主性强，但需要自行维护、修理服务器硬件设备，适合大中型网站。

  另外需要提醒读者的是，在选择服务器主机供应商时请一定要注意查看口碑，并在综合分析后再决定购买。某些供应商会有限制功能、强制添加广告、隐藏扣费或强制扣费等恶劣行为，请各位读者一定擦亮眼睛，不要上当!

  **出现问题?大胆提问!**

  > 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
  >
  > Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
  >
  > *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

  **本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

  1．使用源码包安装服务程序的最大优点和缺点是什么？

  **答：**使用源码包安装服务程序的最大优势是，服务程序的可移植性好，而且能更好地提升服务程序的运行效率；缺点是源码包程序的安装、管理、卸载和维护都比较麻烦。

  2．使用源码包的方式来安装软件服务的大致步骤是什么？

  **答：**基本分为4个步骤，分别为下载及解压源码包文件、编译源码包代码、生成二进制安装程序、运行二进制的服务程序安装包。

  3．LNMP动态网站部署架构通常包含了哪些服务程序？

  **答：**LNMP动态网站部署架构通常包含Linux系统、Nginx网站服务、MySQL数据库管理系统，以及PHP脚本语言。

  4．在MySQL数据库服务程序中，/usr/local/mysql与/usr/local/mysql/data目录的作用是什么？

  **答：**/usr/local/mysql用于保存MySQL数据库服务程序的目录，/usr/local/mysql/var则用于保存真实数据库文件的目录。

  5．较之于Apache服务程序，Nginx最显著的优势是什么？

  **答：**Nginx服务程序比较稳定，原因是采用了的资源分配技术，降低了CPU与内存的占用率，所以使用Nginx程序部署的动态网站环境不仅十分稳定、高效，而且消耗的系统资源也很少。

  6．如何禁止php服务程序中不安全的功能？

  **答：**编辑php服务程序的配置文件（/usr/local/php/etc/php.ini），把要禁用的功能追加到disable_functions参数之后即可。

  7． 对于处于创业阶段的小站长群体来说，适合购买哪种服务器类型呢？

  **答：**刘遄老师建议他们选择云服务器类型，不但费用便宜（每个月费用不超过100元人民币），而且性能也十分强劲。

- ##### LAMP架构

### 文件服务器

### Vsftpd传输服务器

### Samba|NFS文件共享服务器

### TFP服务器

### DHCP服务器

### DNS服务器

#### Bind域名解析服务

### 邮件系统服务器

#### Postfix部署

#### Dovecot部署

### 网络存储服务器

### iSCSI服务器

### 数据库服务器

#### MariaDB数据库

#### MySQL数据库

#### Redis数据库

### 自动化安装服务器部署

#### PEX+Kickstart

### 自动化运维实现

#### Ansible

### SSH服务器

**章节简述：** 

本章讲解了如何使用nmtui[命令](https://www.linuxcool.com/)配置网卡参数，以及通过nmcli[命令](https://www.linuxcool.com/)查看网络信息并管理网络会话服务，从而让读者能够在不同工作场景中快速地切换网络运行参数；还讲解了如何手工绑定round-robin轮询模式双网卡，实现网络的负载均衡。

本章深入介绍了SSH协议与sshd服务程序的理论知识、[Linux系统](https://www.linuxprobe.com/)的远程管理方法以及在系统中配置服务程序的方法，并采用实验的形式演示了使用基于密码与密钥验证的sshd服务程序进行远程访问，以及使用登录tmux服务程序远程管理[Linux](https://www.linuxprobe.com/)系统的不间断会话等技术。细致讲解日志系统的理论知识，使用journalctl命令基于各种条件进行日志信息的检索，快速定位工作中的故障点。

当读者掌握了本章的内容之后，也就完全具备了对Linux系统进行配置管理的知识。而且后续章节中将陆续引入大量实用服务的配置内容，读者将用到本章学习的知识进行配置，这样一方面能够让读者对生产环境中用到的大多数热门服务程序有一个广泛且深入的认识，另一方面也可以掌握相应的配置方法。

本章目录结构

- 配置网卡服务
  - [ 配置网卡参数](https://www.linuxprobe.com/basic-learning-09.html#911)
  - [ 创建网络会话](https://www.linuxprobe.com/basic-learning-09.html#912)
  - [ 绑定两块网卡](https://www.linuxprobe.com/basic-learning-09.html#913)
- 远程控制服务
  - [ 配置sshd服务](https://www.linuxprobe.com/basic-learning-09.html#921_sshd)
  - [ 安全密钥验证](https://www.linuxprobe.com/basic-learning-09.html#922)
  - [ 远程传输命令](https://www.linuxprobe.com/basic-learning-09.html#923)
- 不间断会话服务
  - [ 管理远程会话](https://www.linuxprobe.com/basic-learning-09.html#931)
  - [ 管理多窗格](https://www.linuxprobe.com/basic-learning-09.html#932)
  - [ 会话共享功能](https://www.linuxprobe.com/basic-learning-09.html#933)
- [检索日志信息](https://www.linuxprobe.com/basic-learning-09.html#94)

##### 配置网卡服务

###### 配置网卡参数

截至目前，读者已经完全可以利用当前所学的知识来管理Linux系统了。当然，大家的水平完全可以更进一步，当有朝一日登顶技术巅峰时，您一定会感谢现在正在努力学习的自己。

我们接下来将学习如何在Linux系统上配置服务。但是在此之前，必须先保证主机之间能够顺畅地通信。如果网络不通，即便服务部署得再正确用户也无法顺利访问，所以，配置网络并确保网络的连通性是学习部署Linux服务之前的最后一个重要知识点。

在4.1.3小节讲解了如何使用Vim文本编辑器来配置网络参数，其实，在RHEL 8系统中有至少5种网络的配置方法，[刘遄](https://www.linuxprobe.com/)老师尽量在本书中为大家逐一演示。这里教给大家的是使用nmtui命令来配置网络，其具体的配置步骤如图9-1至图9-8所示。当遇到不容易理解的内容时，书中会额外进行解释说明。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/1-24.png)

图9-1 执行nmtui命令运行网络配置工具

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/2-13.png)

图9-2 选中配置网卡按钮并按下回车键

在RHEL 5、RHEL 6系统及其他大多数早期的Linux系统中，网卡的名称一直都是eth0、eth1、eth2、……；在RHEL 7中则变成了类似于eno16777736、eno33554968这样的名字；而RHEL 8系统最新的名称是ens160、ens192等等诸如此类，所以经常丰富的运维老手光看网卡名称大致就能猜出系统版本了。不过除了网卡的名称发生变化之外，其他几乎一切照旧，因此这里演示的网络配置实验完全可以适用于各种版本的Linux系统。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/3-12.png)

图9-3 选中要配置的网卡名称，然后按下编辑按钮

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/4-8-1024x647.png)

图9-4 把网卡IPv4的配置方式改成手动模式

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/5-8.png)

图9-5 按下显示详细信息按钮

现在，在服务器主机的网络配置信息中填写IP地址192.168.10.10/24。24代表子网掩码中前24位为网络号，后8位是主机号，和写成255.255.255.0的效果一样。网关、DNS等信息暂可不必填写，有用到时会继续编辑网卡文件。

再多提一句，我们的这本《Linux就该这么学》不仅学习门槛低、简单易懂，而且还有一个潜在的优势——书中所有的服务器主机IP地址均为192.168.10.10，而客户端主机均为192.168.10.20及192.168.10.30。这样的好处就是，在后面部署Linux服务的时候，不用每次都要考虑IP地址变化的问题，从而可以心无旁骛地关注配置细节。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/6-8.png)

图9-6 填写IP地址和子网掩码

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/7-6.png)

图9-7 单击OK按钮保存配置

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/8-4.png)

图9-8 单击Back按钮结束配置工作

至此，在Linux系统中配置网络的步骤就结束了。

[刘遄](https://www.linuxprobe.com/)老师在培训时经常会发现，很多学员在安装RHEL 8系统时默认没有激活网卡。如果各位读者有同样的情况也不用担心，只需使用Vim编辑器将网卡配置文件中的ONBOOT参数修改成yes，这样在系统重启后网卡就被激活了。

```shell
[root@linuxprobe ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens160
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
UUID=97486c86-6d1e-4e99-9aa2-68d3172098b2
DEVICE=ens160
ONBOOT=yes
HWADDR=00:0C:29:7D:27:BF
IPADDR=192.168.10.10
PREFIX=24
IPV6_PRIVACY=no
```

当修改完Linux系统中的服务配置文件后，并不会对服务程序立即产生效果。要想让服务程序获取到最新的配置文件，需要手动重启相应的服务，之后就可以看到网络畅通了：

```shell
[root@linuxprobe ~]# nmcli connection reload ens160
[root@linuxprobe ~]# nmcli connection up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
[root@linuxprobe ~]# ping 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.122 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.048 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.106 ms
64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.043 ms

--- 192.168.10.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 61ms
rtt min/avg/max/mdev = 0.043/0.079/0.122/0.036 ms
```

######  创建网络会话

RHEL和[CentOS](https://www.linuxprobe.com/)系统默认使用NetworkManager来提供网络服务，这是一种动态管理网络配置的守护进程，能够让网络设备保持连接状态。可以使用nmcli命令来管理NetworkManager服务程序。nmcli是一款基于命令行的网络配置工具，功能丰富，参数众多。它可以轻松地查看网络信息或网络状态：

```shell
[root@linuxprobe ~]# nmcli connection show
NAME    UUID                                  TYPE      DEVICE 
ens160  97486c86-6d1e-4e99-9aa2-68d3172098b2  ethernet  ens160 
virbr0  e5fca1ee-7020-4c21-a65b-259d0f993b44  bridge    virbr0
[root@linuxprobe ~]# nmcli connection show ens160 
connection.id:                          ens160
connection.uuid:                        97486c86-6d1e-4e99-9aa2-68d3172098b2
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens160
connection.autoconnect:                 yes
………………省略部分输出信息………………
```

另外，RHEL 8系统支持网络会话功能，允许用户在多个配置文件中快速切换（非常类似于firewalld防火墙服务中的区域技术）。如果在公司网络中使用笔记本电脑时需要手动指定网络的IP地址，而回到家中则是使用DHCP自动分配IP地址。这就需要麻烦地频繁修改IP地址，但是使用了网络会话功能后一切就简单多了——只需在不同的使用环境中激活相应的网络会话，就可以实现网络配置信息的自动切换了。

使用nmcli命令并按照“connection add con-name type ifname”的格式来创建网络会话。假设将公司网络中的网络会话称之为company，将家庭网络中的网络会话称之为house，现在依次创建各自的网络会话。

使用con-name参数指定公司所使用的网络会话名称company，然后依次用ifname参数指定本机的网卡名称（千万要以实际环境为准，不要照抄书上的ens160），用autoconnect no参数设置该网络会话默认不被自动激活，以及用ip4及gw4参数手动指定网络的IP地址：

```shell
[root@linuxprobe ~]# nmcli connection add con-name company ifname ens160 autoconnect no type ethernet ip4 192.168.10.10/24 gw4 192.168.10.1
Connection 'company' (6ac8f3ad-0846-42f4-819a-e1ae84f4da86) successfully added.
```

使用con-name参数指定家庭所使用的网络会话名称house。因为要从外部DHCP服务器自动获得IP地址，所以这里不需要进行手动指定。

```shell
[root@linuxprobe ~]# nmcli connection add con-name house type ethernet ifname ens160
Connection 'house' (d848242a-4bdf-4446-9079-6e12ab5d1f15) successfully added.
```

在成功创建网络会话后，可以使用nmcli命令查看创建的所有网络会话：

```shell
[root@linuxprobe ~]# nmcli connection show
NAME     UUID                                  TYPE      DEVICE 
ens160   97486c86-6d1e-4e99-9aa2-68d3172098b2  ethernet  ens160 
virbr0   e5fca1ee-7020-4c21-a65b-259d0f993b44  bridge    virbr0 
company  6ac8f3ad-0846-42f4-819a-e1ae84f4da86  ethernet  --     
house    d848242a-4bdf-4446-9079-6e12ab5d1f15  ethernet  --  
```

使用nmcli命令配置过的网络会话是永久生效的，这样当我们上班后，顺手启动company网络会话，网卡信息就自动配置好了：

```shell
[root@linuxprobe ~]# nmcli connection up company 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
[root@linuxprobe ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.88  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::320e:a005:dfa1:431c  prefixlen 64  scopeid 0x20
        ether 00:0c:29:7d:27:bf  txqueuelen 1000  (Ethernet)
        RX packets 66  bytes 5469 (5.3 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 99  bytes 11255 (10.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
………………省略部分输出信息………………
```

如果大家使用的是虚拟机，请把虚拟机系统的网络适配器切换成桥接模式，如图9-9所示，然后再重启下虚拟机。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%8D%A1%E4%B8%BA%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F.png)

图9-9 设置虚拟机网卡的模式

这样操作过后就能使用家庭中的路由器设备了，启动house家庭会话看下效果吧：

```shell
[root@linuxprobe ~]# nmcli connection up house
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[root@linuxprobe ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.0.107  netmask 255.255.255.0  broadcast 192.168.0.255
        inet6 fe80::f209:dc47:4004:3868  prefixlen 64  scopeid 0x20
        ether 00:0c:29:7d:27:bf  txqueuelen 1000  (Ethernet)
        RX packets 22  bytes 6924 (6.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 82  bytes 10582 (10.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
………………省略部分输出信息………………
```

如果启用company会话成功，但启用house会话失败且不能获取到DHCP动态地址的话，证明您的配置是正确的，问题出在了外部网络环境。有三种常见的情况，首先，读者家中的设备没有连接路由器，通过拨号网络或共享WIFI的方式上网；其次，还在上学或上班的同学在浏览网页前必须通过学校或公司的验证页面才能访问互联网；最后，检查物理机防火墙设置，可暂时关闭后再重试。

后续不需要网卡会话了，直接用delete删除就能删除，特别简单：

```shell
[root@linuxprobe ~]# nmcli connection delete house
Connection 'house' (d848242a-4bdf-4446-9079-6e12ab5d1f15) successfully deleted.
[root@linuxprobe ~]# nmcli connection delete company 
Connection 'company' (6ac8f3ad-0846-42f4-819a-e1ae84f4da86) successfully deleted.
```

###### 绑定两块网卡

一般来讲，生产环境必须提供7×24小时的网络传输服务。借助于网卡绑定技术，不仅能够提高网络传输速度，更重要的是，还可以确保在其中一块网卡出现故障时，依然可以正常提供网络服务。假设我们对两块网卡实施了绑定技术，这样在正常工作中它们会共同传输数据，使得网络传输的速度变得更快；而且即使有一块网卡突然出现了故障，另外一块网卡便会立即自动顶替上去，保证数据传输不会中断。

下面来看一下如何绑定网卡。

在虚拟机系统中再添加一块网卡设备，请确保两块网卡都处在同一种网络连接模式中，如图9-10和图9-11所示。处于相同模式的网卡设备才被允许进行网卡绑定，否则这两块网卡无法互相传送数据。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E5%9C%A8%E8%99%9A%E6%8B%9F%E6%9C%BA%E4%B8%AD%E5%86%8D%E6%B7%BB%E5%8A%A0%E4%B8%80%E5%9D%97%E7%BD%91%E5%8D%A1%E8%AE%BE%E5%A4%87-1-1024x711.png)

图9-10 在虚拟机中再添加一块网卡设备

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E7%A1%AE%E4%BF%9D%E4%B8%A4%E5%9D%97%E7%BD%91%E5%8D%A1%E5%A4%84%E5%9C%A8%E5%90%8C%E4%B8%80%E4%B8%AA%E7%BD%91%E7%BB%9C%E8%BF%9E%E6%8E%A5%E4%B8%AD%EF%BC%88%E5%8D%B3%E7%BD%91%E5%8D%A1%E6%A8%A1%E5%BC%8F%E7%9B%B8%E5%90%8C%EF%BC%89-2.png)

图9-11 确保两块网卡处在同一个网络连接中（即网卡模式相同）

上面章节用的是nmtui命令配置的网卡信息，这次对应的就使用nmcli命令来配置网卡设备的绑定参数吧，这样读者们两个命令都可以学习到啦，不会太依赖于某个特定命令。网卡绑定的理论知识类似于前面学习的RAID硬盘组，我们需要对参与绑定的网卡设备逐个进行“初始设置”。需要注意的是，如图9-12所示，这些原本独立的网卡设备此时需要被配置成为一块“从属”网卡，服务于“主”网卡，不应该再有自己的IP地址等信息。在进行了初始设置之后，它们就可以支持网卡绑定。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/bond%E7%BD%91%E5%8D%A1%E7%BB%91%E5%AE%9A-3.jpg)

图9-12 网卡绑定信息示意图

**1．创建出一个bond网卡**

nmcli命令配置网卡信息有一定的难度，所以咱们放到了第九章才开始讲解。首先创建一个bond网卡，命令与参数的意思是创建了一个类型为bond（绑定）、名称为bond0、网卡名为bond0的绑定设备，模式为balance-rr：

```shell
[root@linuxprobe ~]# nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=balance-rr"
Connection 'bond0' (b37b720d-c5fa-43f8-8578-820d19811f32) successfully added.
```

这里使用的是balance-rr网卡绑定模式，rr是round-robin的缩写，全称也叫轮循模式。round-robin的特点是会根据设备顺序依次传输数据包，提供负载均衡的效果，让带宽变得更大，一旦某个网卡故障马上切换到另外一台设备上，保证网络传输不被中断。而active-backup网卡绑定模式也比较常用，特点是平时只有一块网卡正常工作，另一个设备待命，一旦出现损坏时自动顶替上去，冗余能力比较强，因此也被叫做主备模式。

比如有一台用于提供NFS或者samba服务的文件服务器，它所能提供的最大网络传输速度为100Mbit/s，但是访问该服务器的用户数量特别多，那么它的访问压力一定很大。在生产环境中，网络的可靠性是极为重要的，而且网络的传输速度也必须得以保证。针对这样的情况，比较好的选择就是balance-rr网卡绑定模式了。因为balance-rr能够让两块网卡同时一起工作，当其中一块网卡出现故障后能自动备援，且无需交换机设备支援，从而提供了可靠的网络传输保障。

**2．向bond0添加从属网卡**

刚刚创建成功的bond0设备当前仅仅是个名称，里面并没有真正能为用户传输数据的网卡设备，接下来要把ens160和ens192网卡添加进去。其中“con-name”参数后面接的是从属网卡名称，可以随时设置，而“ifname”参数后面接的是两块网卡名称，一定要以同学们实际为准，不要直接复制：

```shell
[root@linuxprobe ~]# nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname ens160 master bond0
Connection 'bond0-port1' (8a2f77ee-cc92-4c11-9292-d577ccf8753d) successfully added.
[root@linuxprobe ~]# nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname ens192 master bond0
Connection 'bond0-port2' (b1ca9c47-3051-480a-9623-fbe4bf731a89) successfully added.
```

**3．配置bond0设备的网卡信息**

配置网卡参数的方法有很多，为了让咱们这个实验的配置过程更加具有一致性，所以决定还是用nmcli命令依次配置网卡的IP地址及子网掩码、网关、DNS、搜索域和手动配置等参数。如果同学们不习惯这个命令，也可以用编辑网卡配置文件或nmtui命令的方式完成下面操作：

```shell
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.addresses 192.168.10.10/24
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.gateway 192.168.10.1
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.dns 192.168.10.1
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.dns-search linuxprobe.com
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.method manual
```

**4．启动它！**

接下来就是最激动人心的时刻了，启动它吧！再顺便看下设备详细的列表：

```shell
[root@linuxprobe ~]# nmcli connection up bond0
Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/22)
[root@linuxprobe ~]# nmcli device status
DEVICE      TYPE      STATE      CONNECTION  
bond0       bond      connected  bond0       
ens160      ethernet  connected  ens160      
virbr0      bridge    connected  virbr0      
ens192      ethernet  connected  bond0-port2 
lo          loopback  unmanaged  --          
virbr0-nic  tun       unmanaged  --
```

当接下来用户访问192.168.10.10这个主机IP地址的时候，实际上是由两块网卡设备在共同提供服务。 可以在本地主机执行ping 192.168.10.10命令检查网络的连通性。为了检验网卡绑定技术的自动备援功能，我们突然在虚拟机硬件配置中随机移除一块网卡设备，如图9-13所示。可以非常清晰地看到网卡切换的过程（一般只有1个数据丢包），然后另外一块网卡会继续为用户提供服务。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E7%A7%BB%E9%99%A4%E6%8E%89%E4%B8%80%E5%9D%97%E7%BD%91%E5%8D%A1-1.png)

图9-13 随机移除掉任意一块网卡

```shell
[root@linuxprobe ~]# ping 192.168.10.10
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.109 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.102 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.066 ms
ping: sendmsg: Network is unreachable
64 bytes from 192.168.10.10: icmp_seq=5 ttl=64 time=0.065 ms
64 bytes from 192.168.10.10: icmp_seq=6 ttl=64 time=0.048 ms
64 bytes from 192.168.10.10: icmp_seq=7 ttl=64 time=0.042 ms
64 bytes from 192.168.10.10: icmp_seq=8 ttl=64 time=0.079 ms
^C
--- 192.168.10.10 ping statistics ---
8 packets transmitted, 7 received, 12% packet loss, time 7006ms
rtt min/avg/max/mdev = 0.042/0.073/0.109/0.023 ms
```

由于在RHEL8系统中网卡绑定切换间隔为1毫秒，也就是1/1000秒，因此上面一个丢包情况大概率都不会出现。

##### 远程控制服务

######  配置sshd服务

SSH（Secure [Shell](https://www.linuxcool.com/)）是一种能够以安全的方式提供远程登录的协议，也是目前远程管理Linux系统的首选方式。在此之前，一般使用FTP或Telnet来进行远程登录。但是因为它们以明文的形式在网络中传输账户密码和数据信息，因此很不安全，很容易受到黑客发起的中间人攻击，这轻则篡改传输的数据信息，重则直接抓取服务器的账户密码。

想要使用SSH协议来远程管理Linux系统，则需要部署配置sshd服务程序。sshd是基于SSH协议开发的一款远程管理服务程序，不仅使用起来方便快捷，而且能够提供两种安全验证的方法：

> 基于口令的验证—用账户和密码来验证登录；
>
> 基于密钥的验证—需要在本地生成密钥对，然后把密钥对中的公钥上传至服务器，并与服务器中的公钥进行比较；该方式相较来说更安全。

前文曾多次强调“Linux系统中的一切都是文件”，因此在Linux系统中修改服务程序的运行参数，实际上就是在修改程序配置文件的过程。sshd服务的配置信息保存在/etc/ssh/sshd_config文件中。运维人员一般会把保存着最主要配置信息的文件称为主配置文件，而配置文件中有许多以井号开头的注释行，要想让这些配置参数生效，需要在修改参数后再去掉前面的井号。sshd服务配置文件中包含的重要参数如表9-1所示。

表9-1                 sshd服务配置文件中包含的参数以及作用

| 参数                              | 作用                                |
| --------------------------------- | ----------------------------------- |
| Port 22                           | 默认的sshd服务端口                  |
| ListenAddress 0.0.0.0             | 设定sshd服务器监听的IP地址          |
| Protocol 2                        | SSH协议的版本号                     |
| HostKey /tc/ssh/ssh_host_key      | SSH协议版本为1时，DES私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_rsa_key | SSH协议版本为2时，RSA私钥存放的位置 |
| HostKey /etc/ssh/ssh_host_dsa_key | SSH协议版本为2时，DSA私钥存放的位置 |
| PermitRootLogin yes               | 设定是否允许root管理员直接登录      |
| StrictModes yes                   | 当远程用户的私钥改变时直接拒绝连接  |
| MaxAuthTries 6                    | 最大密码尝试次数                    |
| MaxSessions 10                    | 最大终端数                          |
| PasswordAuthentication yes        | 是否允许密码验证                    |
| PermitEmptyPasswords no           | 是否允许空密码登录（很不安全）      |



接下来的实验种会使用两台虚拟机，IP地址及作用如表9-2所示。

表9-2                 sshd服务实验机器简介

| 主机地址      | 操作系统 | 作用   |
| ------------- | -------- | ------ |
| 192.168.10.10 | Linux    | 服务器 |
| 192.168.10.20 | Linux    | 客户端 |



在RHEL 8系统中，已经默认安装并启用了sshd服务程序。接下来在客户端使用ssh命令进行远程连接服务器，其格式为“ssh [参数] 主机IP地址”，要退出登录则执行exit命令。第一次访问时需要输入yes来确认对方主机的指纹信息：

```shell
[root@Client ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处输入服务器管理员密码
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Jul 24 06:26:58 2020
[root@Server ~]# 
[root@Server ~]# exit
logout
Connection to 192.168.10.10 closed.
```

如果禁止以root管理员的身份远程登录到服务器，则可以大大降低被黑客暴力破解密码的几率。下面进行相应配置。首先使用Vim文本编辑器打开服务器上的sshd服务主配置文件，然后把第46行#PermitRootLogin yes参数前的井号（#）去掉，并把参数值yes改成no，这样就不再允许root管理员远程登录了。记得最后保存文件并退出。

```shell
[root@Server ~]# vim /etc/ssh/sshd_config 
 ………………省略部分输出信息………………
 43 # Authentication:
 44 
 45 #LoginGraceTime 2m
 46 PermitRootLogin no
 47 #StrictModes yes
 48 #MaxAuthTries 6
 49 #MaxSessions 10
 ………………省略部分输出信息………………
```

再次提醒的是，一般的服务程序并不会在配置文件修改之后立即获得最新的参数。如果想让新配置文件生效，则需要手动重启相应的服务程序。最好也将这个服务程序加入到开机启动项中，这样系统在下一次启动时，该服务程序便会自动运行，继续为用户提供服务。

```shell
[root@Server ~]# systemctl restart sshd
[root@Server ~]# systemctl enable sshd
```

这样一来，当root管理员再来尝试访问sshd服务程序时，系统会提示不可访问的错误信息。虽然sshd服务程序的参数相对比较简单，但这就是在Linux系统中配置服务程序的正确方法。大家要做的是举一反三、活学活用，这样即便以后遇到了陌生的服务，也一样可以搞定了。

```shell
[root@Client ~]# ssh 192.168.10.10
root@192.168.10.10's password:此处输入服务器管理员密码
Permission denied, please try again.
```

为了避免后续实验不能用root管理员账号登录了，请同学们再动手把上面的参数修改回来吧：

```shell
[root@Server ~]# vim /etc/ssh/sshd_config 
………………省略部分输出信息……………… 
 43 # Authentication:
 44 
 45 #LoginGraceTime 2m
 46 PermitRootLogin yes
 47 #StrictModes yes
 48 #MaxAuthTries 6
 49 #MaxSessions 10 
………………省略部分输出信息………………
[root@Server ~]# systemctl restart sshd 
[root@Server ~]# systemctl enable sshd
```

如果读者想使用Windows系统进行访问也没问题，首先确保网络是可以通信的，随后在Xshell、Putty、SecureCRT、Secure Shell Client等工具中选择一个喜欢的，远程连接试试看吧~如图9-14、9-15与9-16所示。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/1-25.png)

图9-14 远程登录输入账号名称

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/2-15.png)

图9-15 远程登录输入账号密码

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/3-14.png)

图9-16 远程登录成功界面

######  安全密钥验证

加密是对信息进行编码和解码的技术，它通过一定的算法（密钥）将原本能被直接阅读的明文信息转换成密文形式。密钥即是密文的钥匙，有私钥和公钥之分。在传输数据时，如果担心被他人监听或截获，就可以在传输前先使用公钥对数据加密处理，然后再行传送。这样，只有掌握私钥的用户才能解密这段数据，除此之外的其他人即便截获了数据，一般也很难将其破译为明文信息。

一言以蔽之，在生产环境中使用密码进行口令验证终归存在着被暴力破解或嗅探截获的风险。如果正确配置了密钥验证方式，那么sshd服务程序将更加安全。我们下面进行具体的配置，其步骤如下。

**第1步**：在客户端主机中生成“密钥对”，记住是客户端。

```shell
[root@Client ~]# ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 按回车键或设置密钥的存储路径
Enter passphrase (empty for no passphrase): 直接按回车键或设置密钥的密码
Enter same passphrase again: 再次按回车键或设置密钥的密码
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:kHa7B8V0nk63evABRrfZhxUpLM5Hx0I6gb7isNG9Hkg root@linuxprobe.com
The key's randomart image is:
+---[RSA 2048]----+
|          o.=.o.+|
|       . + =oB X |
|      + o =oO O o|
|     . o + *.+ ..|
|      .ES . + o  |
|     o.o.=   + . |
|      =.o.o . o  |
|     . . o.  .   |
|        ..       |
+----[SHA256]-----+
```

**第2步**：把客户端主机中生成的公钥文件传送至远程服务器：

```shell
[root@Client ~]# ssh-copy-id 192.168.10.10
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.10.10's password: 此处输入服务器管理员密码

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh '192.168.10.10'"
and check to make sure that only the key(s) you wanted were added.
```

**第3步**：对服务器进行设置，使其只允许密钥验证，拒绝传统的口令验证方式。记得在修改配置文件后保存并重启sshd服务程序。

```shell
[root@Server ~]# vim /etc/ssh/sshd_config 
 ………………省略部分输出信息………………
 70 # To disable tunneled clear text passwords, change to no here!
 71 #PasswordAuthentication yes
 72 #PermitEmptyPasswords no
 73 PasswordAuthentication no
 74 
 ………………省略部分输出信息………………
[root@Server ~]# systemctl restart sshd
```

**第4步**：在客户端尝试登录到服务器，此时无须输入密码也可成功登录，特别方便：

```shell
[root@Client ~]# ssh 192.168.10.10
Activate the web console with: systemctl enable --now cockpit.socket

Last failed login: Thu Jan 28 13:44:09 CST 2021 from 192.168.10.20 on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Thu Jan 28 13:22:34 2021 from 192.168.10.20
```

但如果用户没有密钥信息，即便有口令密码也会被拒绝，连输入密码的机会都不给哦，如图9-17所示：

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E6%97%A0%E5%AF%86%E9%92%A5%E8%AE%BF%E9%97%AE%E8%BF%9C%E7%A8%8B%E6%9C%8D%E5%8A%A1%E5%99%A8%E8%A2%AB%E6%8B%92.png)

图9-17 无密钥访问远程服务器被拒

###### 远程传输命令

不知道读者有没有考虑一个问题，既然SSH协议可以让用户远程控制服务器，传输命令信息，那是不是也能传输文件呢？

scp（secure copy）是一个基于SSH协议在网络之间进行安全传输的命令，与第2章讲解的cp命令不同，cp命令只能在本地硬盘中进行文件复制，而scp不仅能够通过网络传送数据，而且所有的数据都将进行加密处理。

其格式为：“scp [参数] 本地文件 远程帐户@远程IP地址:远程目录”

例如，如果想把一些文件通过网络从一台主机传递到其他主机，这两台主机又恰巧是Linux系统，这时使用scp命令就可以轻松完成文件的传递了。scp命令中可用的参数以及作用如表9-3所示。

表9-3                       scp命令中可用的参数及作用

| 参数 | 作用                     |
| ---- | ------------------------ |
| -v   | 显示详细的连接进度       |
| -P   | 指定远程主机的sshd端口号 |
| -r   | 用于传送文件夹           |
| -6   | 使用IPv6协议             |



在使用scp命令把文件从本地复制到远程主机时，首先需要以绝对路径的形式写清本地文件的存放位置。如果要传送整个文件夹内的所有数据，还需要额外添加参数-r进行递归操作。然后写上要传送到的远程主机的IP地址，远程服务器便会要求进行身份验证了。当前用户名称为root，而密码则为远程服务器的密码。如果想使用指定用户的身份进行验证，可使用用户名@主机地址的参数格式。最后需要在远程主机的IP地址后面添加冒号，并在后面写上要传送到远程主机的哪个文件夹中。只要参数正确并且成功验证了用户身份，即可开始传送工作。由于scp命令是基于SSH协议进行文件传送的，而9.2.2小节又设置好了密钥验证，因此当前在传输文件时，并不需要账户和密码。

```shell
[root@Client ~]# echo "Welcome to LinuxProbe.Com" > readme.txt
[root@Client ~]# scp /root/readme.txt 192.168.10.10:/home
readme.txt                                    100%   26    13.6KB/s   00:00
```

此外，还可以使用scp命令把远程服务器上的文件下载到本地主机，这样就无须先登录远程主机再进行文件传送了，也就省去了很多周折。例如，可以把远程主机的系统版本信息文件下载过来。

其命令格式为“scp [参数] 远程用户@远程IP地址:远程文件 本地目录”。

```shell
[root@Client ~]# scp 192.168.10.10:/etc/redhat-release /root
[root@Client ~]# scp 192.168.10.10:/etc/redhat-release /root
redhat-release                                100%   45    23.6KB/s   00:00    
[root@Client ~]# cat redhat-release 
Red Hat Enterprise Linux release 8.0 (Ootpa)
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **9.3 不间断会话服务**

大家在学习sshd服务时，不知有没有注意到这样一个事情：当与远程主机的会话被关闭时，在远程主机上运行的命令也随之被中断。

如果正在使用命令来打包文件，或者正在使用脚本安装某个服务程序，中途是绝对不能关闭在本地打开的终端窗口或断开网络链接的，甚至是网速的波动都有可能导致任务中断，此时只能重新进行远程链接并重新开始任务。还有些时候，我们正在执行文件打包操作，同时又想用脚本来安装某个服务程序，这时会因为打包操作的输出信息占满用户的屏幕界面，而只能再打开一个执行远程会话的终端窗口，时间久了，难免会忘记这些打开的终端窗口是做什么用的了。

Terminal multiplexer是一款能够实现多窗口远程控制的开源服务程序，也叫终端复用器，简称Tmux。简单来说就是为了解决网络异常中断或为了同时控制多个远程终端窗口而设计的程序。用户还可以使用Tmux服务程序同时在多个远程会话中自由切换，能够做到实现如下功能。

1. **会话恢复**：即便网络中断，也可让会话随时恢复，确保用户不会失去对远程会话的控制。
2. **多窗口**：每个会话都是独立运行的，拥有各自独立的输入输出终端窗口，终端窗口内显示过的信息也将被分开隔离保存，以便下次使用时依然能看到之前的操作记录。
3. **会话共享**：当多个用户同时登录到远程服务器时，便可以使用会话共享功能让用户之间的输入输出信息共享。

在RHEL 8系统中，默认没有安装Tmux服务程序，因此需要配置软件仓库来安装它，此步骤请见8.3.2小节。这里直接开始安装：

```shell
[root@linuxprobe ~]# dnf install tmux
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                       3.1 MB/s | 3.2 kB     00:00    
BaseOS                                          2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
================================================================================
 Package         Arch              Version              Repository         Size
================================================================================
Installing:
 tmux            x86_64            2.7-1.el8            BaseOS            317 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 317 k
Installed size: 770 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                        1/1 
  Installing       : tmux-2.7-1.el8.x86_64                                  1/1 
  Running scriptlet: tmux-2.7-1.el8.x86_64                                  1/1 
  Verifying        : tmux-2.7-1.el8.x86_64                                  1/1 
Installed products updated.

Installed:
  tmux-2.7-1.el8.x86_64                                                         

Complete!
```



简捷起见，刘遄老师将对后面章节中出现的软件安装信息进行过滤—把重复性高及无意义的非必要信息省略。

###### **9.3.1 管理远程会话**

Tmux服务能做的事情非常多，例如创建不间断会话、恢复离线工作、界面切割窗格、会话共享功能等等，下面可以直接敲击tmux命令进入到会话窗口中，如图9-18所示。不难发现，进入到会话的终端底部出现了一个绿色的状态栏，分别显示的是会话编号、名称、主机名及系统时间。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/tmux.png)

图9-18 Tumx服务程序会话界面

退出会话的命令是exit，敲击后即可返回到正常的终端界面，如图9-19所示：

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E9%80%80%E5%87%BATmux.png)

图9-19 从会话窗口退回到终端界面

会话窗口的编号是从0开始自动排序，即0、1、2、3……，数量少还没关系，数量多的时候区分起来很麻烦。接下来创建一个指定名称为backup的会话窗口吧，请各位读者留心观察，当在命令行中敲下这条命令的一瞬间，屏幕会快速闪动一下，这时就已经进入Tmux服务会话中了，在里面运行的任何操作都会被后台记录下来。

```shell
[root@linuxprobe ~]# tmux new -s backup
```

突然要去忙其他事情了，但是会话窗口中执行的进程还不能被中断，此时便可以用detach参数将会话隐藏到后台。虽然看起来与刚才没有不同，但实际上可以查看到当前的会话正在工作中：

```shell
[root@linuxprobe ~]# tmux detach
[detached (from session backup)]
```

如果觉得每次都要输入detach参数很麻烦，可以直接如图9-20所示关闭中断窗口（这与进行远程连接时突然断网具有相同的效果），Tmux服务程序会自动帮我们进行保存。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E5%85%B3%E9%97%AD%E4%BC%9A%E8%AF%9D-1024x613.png)

图9-20 强行关闭会话窗口

这样执行过后服务和进程都会一直在后台默默运行，不会因为窗口被关闭而造成数据丢失。不放心的话可以查看下后台有哪些会话：

```shell
[root@linuxprobe ~]# tmux ls
backup: 1 windows (created Thu Jan 28 15:57:40 2021) [80x23]
```

由于刚才关闭了会话窗口，这样的操作在传统的远程控制中一定会导致正在运行的命令也突然终止，但在不间断会话服务中则不会这样。我们只需查看一下刚刚离线的会话名称，然后尝试恢复回来就可以继续工作了，回归到刚刚backup中的方法很简单，再直接attach加会话编号或会话名称就可以的。刚刚离开时正在进行的一切工作状态都会被原原本本的被呈现出来，丝毫不影响：

```shell
[root@linuxprobe ~]# tmux attach -t backup
```

不需要使用这个Tmux会话了，不用费事先attach接入，再exit命令退出。其实可以直接用kill进行关闭的：

```shell
[root@linuxprobe ~]# tmux attach -t backup
[exited]
[root@linuxprobe ~]# tmux ls
no server running on /tmp/tmux-0/default
```

在日常的生产环境中，其实并不是必须先创建会话，然后再开始工作。直接使用tmux命令执行要运行的命令，这样在命令中的一切操作也都会被记录下来，当命令执行结束后后台会话也会自动结束。

```shell
[root@linuxprobe ~]# tmux new "vim memo.txt"
```

###### **9.3.2 管理多窗格**

在实际工作中，一个Shell终端窗口总是不够用怎么办呢？Tmux服务有个多窗格的功能，能够把一个终端界面上下或左右进行切割，同时能够做几件事情，而且之间互不打扰，还能保留之前的记录，特别方便。

先创建一个会话。创建上下切割的多窗格终端界面可以用“tmux split-window”命令，如图9-21所示。创建左右切割的多窗格终端界面可以用“tmux split-window -h”命令，如图9-22所示。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%B8%8A%E4%B8%8B%E5%88%87%E5%89%B2-1024x647.png)

图9-21 上下切割多窗格

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E5%B7%A6%E5%8F%B3%E5%88%87%E5%89%B2-1024x638.png)

图9-22 上下切割多窗格

创建多窗口的终端界面后，同时做几件事情都不会乱了，如果觉得两个界面还是不够，那就再执行几次吧，退出就用exit命令即可。

呀！

一不小心创建的太多了，当前正在使用的窗口变得特别小，看不到输入内容了怎么办？不要着急！可以同时按下<ctrl>+<b>+<方向键>调整窗口的尺寸。例如现在创建有些小，想向右扩大一些，则同时如下<ctrl>+<b>+<向右箭头>就行了。

如果需要切换到其他窗格进行工作，但又不能关闭当前的窗格，则使用如表9-4所示的命令进行切换：

表9-4                 Tmux不间断会话多窗格切换命令

| 命令                | 作用             |
| ------------------- | ---------------- |
| tmux select-pane -U | 切换至上方的窗格 |
| tmux select-pane -D | 切换至下方的窗格 |
| tmux select-pane -L | 切换至左方的窗格 |
| tmux select-pane -R | 切换至右方的窗格 |



假如想调整窗格的位置，把上面与下面的窗格位置互换，可以用如表9-5所示的命令进行互换：

表9-4                 Tmux不间断会话多窗格互换命令

| 命令              | 作用                       |
| ----------------- | -------------------------- |
| tmux swap-pane -U | 将当前窗格与上方的窗格互换 |
| tmux swap-pane -D | 将当前窗格与下方的窗格互换 |
| tmux swap-pane -L | 将当前窗格与左方的窗格互换 |
| tmux swap-pane -R | 将当前窗格与右方的窗格互换 |



如图9-23所示，原本执行过uptime命令的窗格在下方，只需要在该窗格中执行“tmux swap-pane -U”命令即可与上方窗格互换位置，效果如图9-24所示。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E7%AA%97%E6%A0%BC%E4%BA%92%E6%8D%A2%E5%89%8D-1024x543.png)

图9-23 切换窗格位置前

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%BA%92%E6%8D%A2%E7%AA%97%E6%A0%BC%E5%90%8E-1024x542.png)
图9-24 切换窗格位置后

工作中切换窗格操作还要输入命令难免有些麻烦，实际上Tmux服务为用户提供了以**<Ctrl>+<b>**同时按下相关的一系列快捷键。记住，是先同时按下<Ctrl>+<b>按键，然后松手后再按下后续按键，不是一起按下的。会话窗格常见的快捷键如表9-5所示：

表9-5                 Tmux会话窗格相关的常用快捷键

| 快捷键   | 作用                           |
| -------- | ------------------------------ |
| %        | 划分左右两个窗格               |
| "        | 划分上下两个窗格               |
| <方向键> | 切换到上下左右相邻的一个窗格   |
| ;        | 切换至上一个窗格               |
| o        | 切换至下一个窗格               |
| {        | 将当前窗格与上一个窗格位置互换 |
| }        | 将当前窗格与下一个窗格位置互换 |
| x        | 关闭窗格                       |
| !        | 将当前窗格拆分成独立窗口       |
| q        | 显示窗格编号                   |



请读者们一定要注意，上面快捷键需要先按下<Ctrl>+<b>按键后再敲，否则不生效。例如图9-25与图9-26所示，将两个窗口的位置互换。操作方法是先同时按下<Ctrl>+<b>按键，松手，再按出}符号，即可实现。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%BA%A4%E6%8D%A2%E5%89%8D-1024x645.png)

图9-25 窗格互换前

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%BA%A4%E6%8D%A2%E5%90%8E-1024x591.png)

图9-26 窗格互换后

基本会了Tmux服务这些操作，日常管理就不成问题了，命令和快捷键不建议死记硬背，工作时可以把书放在案头，随用随查即可。

###### **9.3.3 会话共享功能**

Tmux服务不仅可以确保用户在极端情况下也不丢失对系统的远程控制，保证了生产环境中远程工作的不间断性，而且它还具有会话共享、分屏窗格、会话锁定等实用的功能。其中，会话共享功能是一件很酷的事情，当多个用户同时控制主机的时候，它可以把屏幕内容共享出来，也就是说每个用户都能够看到相同的内容，还能一起同时操作。会话共享功能的流程拓扑如图9-27所示。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%BC%9A%E8%AF%9D%E5%85%B1%E4%BA%AB%E5%8A%9F%E8%83%BD%E6%8A%80%E6%9C%AF%E6%8B%93%E6%89%91-2.jpg)

图9-27 会话共享功能技术拓扑

要实现会话共享功能，首先使用ssh服务将客户端A远程连接到服务器。随后使用tmux服务创建一个新的会话窗口，名称为share：

```shell
[root@client A ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处输入服务器管理员密码
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Jul 24 06:26:58 2020
[root@client A ~]# tmux new -s share
```

然后，使用ssh服务将客户端B也远程连接到服务器，并执行获取远程会话的命令。接下来，两台主机就能看到相同的内容了。

```shell
[root@client B ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处输入服务器管理员密码
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Fri Jul 24 06:26:58 2020
[root@client B ~]# tmux attach-session -t share
```

操作完成后，两台客户端的所有终端信息都会被实时同步，可以一起共享使用一个会话窗口，特别方便。为了让读者们更好的感受会话共享功能的强大之处，我们在同一台电脑上创建出两个窗口，如图9-28所示，更能清晰的看到数据被同步的过程。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E4%BC%9A%E8%AF%9D%E5%90%8C%E6%AD%A5-1024x609.png)

图9-28 终端界面进行会话同步

##### **9.4 检索日志信息**

Linux系统拥有十分强大且灵活的日志系统，保存了几乎日常中所有的操作记录和服务运行状态，并且按照报错、警告、提示和其他等标注进行了分类，让运维管理员可以根据所需的信息进行检索，快速找出想要的信息，对于了解系统运行状态有着不错的帮助作用。

在最新版的RHEL 8系统中日志服务程序默认是rsyslog，相比于此前的syslogd服务，实际上可以理解成是增强版本，更加注重日志的安全性和性能指标。不同的日志信息会被写入到不同的文件中，便于日后的检索，常见的日志文件如表9-6所示。

表9-6                 常见的日志文件保存路径

| 文件路径及命令    | 作用                                       |
| ----------------- | ------------------------------------------ |
| /var/log/boot.log | 系统开机自检事件及引导过程等信息           |
| /var/log/lastlog  | 用户登录成功时间、终端名称及IP地址等信息   |
| /var/log/btmp     | 记录登录失败的时间、终端名称及IP地址等信息 |
| /var/log/messages | 系统及各个服务的运行和报错信息             |
| /var/log/secure   | 系统安全相关的信息                         |
| /var/log/wtmp     | 系统启动与关机等相关信息                   |



日常工作中还是/var/log/message这个综合性的文件用的最多，处理Linux系统出现的各种故障时，症状是最先被发现了，而找到这些故障的原因一定离不开日志信息的帮忙。从理论上讲，日志文件分为三种类型。第一种叫系统日志，主要是记录系统的运行情况和内核信息；第二种叫用户日志，主要记录用户的访问信息，包含用户名、终端名称、时间、来源和执行过的操作等等；第三种叫程序日志，一般稍微大一些的服务都会保存一份与其同名的日志文件，记录着服务运行过程中各种事件的信息，每个服务程序自己有独立的日志文件，但是格式相差的比较大。

只有快速的定位故障点，才能对症下药及时解决各种系统问题。

journalctl命令用于检索和管理系统日志信息，英文全称为：“journal control”，语法格式为：“journalctl 参数”。

由于上面所提及的，每个稍微大一点的服务都会有自己独立的日志文件，为了让用户检索信息不至于特别麻烦，journalctl命令便由此诞生。它可以根据事件、类型、服务名等信息进行信息检索，大大的提高了日常排错的效率，常见参数如表9-7所示，同学们先有个印象再开始实验。

表9-7                journalctl命令中常用按键以及作用

| 参数         | 作用                 |
| ------------ | -------------------- |
| -k           | 内核日志             |
| -b           | 启动日志             |
| -u           | 指定服务             |
| -n           | 指定条数             |
| -p           | 指定类型             |
| -f           | 实时刷新（追踪日志） |
| --since      | 指定时间             |
| --disk-usage | 占用空间             |



动手操练起来吧！~首先查看下系统最后5条日志信息：

```shell
[root@linuxprobe ~]# journalctl -n 5 
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 13:39:51 CS>
Jan 31 13:33:54 localhost.localdomain systemd[1]: Started Fingerprint Authentic>
Jan 31 13:33:55 localhost.localdomain gnome-keyring-daemon[2533]: couldn't init>
Jan 31 13:33:55 localhost.localdomain gdm-password][4983]: gkr-pam: unlocked lo>
Jan 31 13:33:56 localhost.localdomain NetworkManager[1203]:   [1612071236>
Jan 31 13:39:51 localhost.localdomain cupsd[1230]: REQUEST localhost - - "POST >
lines 1-6/6 (END)
```

还可以使用-f参数进行实时刷新日志最新内容，与第二章节学习过的tail -f /var/log/message命令效果相同：

```shell
[root@linuxprobe ~]# journalctl -f 
-- Logs begin at Fri 2020-07-24 05:59:38 CST. --
Jan 31 13:33:54 localhost.localdomain dbus-daemon[1058]: [system] Activating via systemd: service name='net.reactivated.Fprint' unit='fprintd.service' requested by ':1.172' (uid=0 pid=2600 comm="/usr/bin/gnome-shell " label="unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023")
Jan 31 13:33:54 localhost.localdomain systemd[1]: Starting Fingerprint Authentication Daemon...
Jan 31 13:33:54 localhost.localdomain dbus-daemon[1058]: [system] Successfully activated service 'net.reactivated.Fprint'
Jan 31 13:33:54 localhost.localdomain systemd[1]: Started Fingerprint Authentication Daemon.
………………省略部分输出信息………………
```

在rsyslog服务程序中，日志根据重要程度被分为了九个等级，这样我们不必被海啸般的输出内容淹没，可以直接直击最重要的内容。读者们可以将表9-8留存，便于日后工作中进行查阅。

表9-8                 日志信息登记分类

| 日志等级 | 说明介绍                                       |
| -------- | ---------------------------------------------- |
| emerg    | 系统出现严重故障，内核崩溃等情况               |
| alert    | 应立即修复的故障，数据库损坏等情况             |
| crit     | 危险较高的故障，硬盘损坏导致程序运行失败的情况 |
| err      | 一般危险的故障，某个服务启动或运行失败的情况   |
| warning  | 警告信息，某个服务参数或功能错误的情况         |
| notice   | 一般无危险的故障，只是需要处理的情况           |
| info     | 通用性消息，给用户提示一些有用信息             |
| debug    | 调试程序所产生的信息                           |
| none     | 没有优先级，不做日志记录                       |



如上表所示，只想看较高级别报错信息的话，用“-p”参数进行指定即可：

```shell
[root@linuxprobe ~]# journalctl -p crit
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:06:07 CST. --
Jul 24 05:59:38 localhost.localdomain kernel: Detected CPU family 6 model 158 stepping 13
Jul 24 05:59:38 localhost.localdomain kernel: Warning: Intel Processor - this hardware has not undergone testing by Red Hat and might not be certified. Please consult https://hardware.redhat.com for certified hardware.
………………省略部分输出信息………………
```

越用越顺手了，不仅能够根据日志类型进行检索，还可以用“--since”参数按照今日（today），近N小时（hour），指定时间范围的格式进行检索，找出最近的日志数据：

**1. 仅查询今日的日志信息**

```shell
[root@linuxprobe ~]# journalctl --since today
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --
Jan 31 12:48:25 localhost.localdomain systemd[1]: Starting update of the root trust anchor >
Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: sd_journal_get_cursor() fail>
Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: journal reloaded... [v8.3>
Jan 31 12:48:25 localhost.localdomain systemd[1]: Started update of the root trust anchor for>
Jan 31 12:48:25 localhost.localdomain sssd[kcm][2764]: Shutting down
………………省略部分输出信息………………
```

**2. 仅查询最近一小时的日志信息**

```shell
[root@linuxprobe ~]# journalctl --since "-1 hour"
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --
Jan 31 14:25:36 localhost.localdomain systemd[1]: Starting dnf makecache...
Jan 31 14:25:36 localhost.localdomain dnf[5516]: Updating Subscription Management repositories.
Jan 31 14:25:36 localhost.localdomain dnf[5516]: Unable to read consumer identity
Jan 31 14:25:36 localhost.localdomain dnf[5516]: This system is not registered to Red Hat>
Jan 31 14:25:36 localhost.localdomain dnf[5516]: Metadata cache refreshed recently.
Jan 31 14:25:36 localhost.localdomain systemd[1]: Started dnf makecache.
………………省略部分输出信息………………
```

**3. 仅查询从上午8点整到10点整的日志信息**

```shell
[root@linuxprobe ~]# journalctl --since "12:00" --until "14:00"
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --
Jan 31 12:48:25 localhost.localdomain systemd[1]: Starting update of the root trust anchor>
Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: sd_journal_get_cursor()>
Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: journal reloaded... [v8.37>
Jan 31 12:48:25 localhost.localdomain systemd[1]: Started update of the root trust anchor>
Jan 31 12:48:25 localhost.localdomain sssd[kcm][2764]: Shutting down
Jan 31 12:48:30 localhost.localdomain systemd[1]: Starting SSSD Kerberos Cache Manager...
Jan 31 12:48:30 localhost.localdomain systemd[1]: Started SSSD Kerberos Cache Manager.
Jan 31 12:48:30 localhost.localdomain sssd[kcm][3981]: Starting up
………………省略部分输出信息………………
```

**4. 仅查询从2020年7月1日至2020年8月1日的日志信息**

```shell
[root@linuxprobe ~]# journalctl --since "2020-07-01" --until "2020-08-01"
-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --
Jul 24 05:59:38 localhost.localdomain kernel: Linux version 4.18.0-80.el8.x86_64 (mockbuild>
Jul 24 05:59:38 localhost.localdomain kernel: Command line: BOOT_IMAGE=(hd0,msdos1)/vmlinuz>
Jul 24 05:59:38 localhost.localdomain kernel: Disabled fast string operations
Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87>
Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE>
Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX>
Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes>
Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Enabled xstate features 0x7, context>
Jul 24 05:59:38 localhost.localdomain kernel: BIOS-provided physical RAM map:
………………省略部分输出信息………………
```

**5. 查询指定服务的日志信息**

默认情况下所有的日志信息都是综合的、混在一起的，如果想看具体某项服务的日志信息，可以使用“_SYSTEMD_UNIT”参数进行查询，服务名称的后面还有.service，这个是标准服务名称的写法。：

```shell
[root@linuxprobe ~]# journalctl _SYSTEMD_UNIT=sshd.service
-- Logs begin at Mon 2020-09-14 15:35:27 CST, end at Sun 2021-01-31 17:26:15 CST. --
Nov 09 13:50:03 iZuf61gqesu0zmrcsma8x4Z sshd[1218]: Server listening on 0.0.0.0 port 22.
Nov 09 13:58:45 iZuf61gqesu0zmrcsma8x4Z sshd[1218]: Received signal 15; terminating.
-- Reboot --
Nov 09 13:59:29 iZuf61gqesu0zmrcsma8x4Z sshd[1127]: Server listening on 0.0.0.0 port 22.
Nov 09 14:12:12 iZuf61gqesu0zmrcsma8x4Z sshd[1262]: Accepted password for root from 111.196
Nov 09 14:12:12 iZuf61gqesu0zmrcsma8x4Z sshd[1262]: pam_unix(sshd:session): session opened
Nov 09 14:14:31 iZuf61gqesu0zmrcsma8x4Z sshd[1127]: Received signal 15; terminating.
………………省略部分输出信息………………
```

恭喜您！又学习完了一个章节~按照书籍内容结构的划分，从第10章节开始将会学习各种服务的配置方法，随着实验的成功，乐趣也会翻倍提升，一定要坚持下去哦！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．在Linux系统中有多种方法可以配置网络参数，请列举几种。

**答：**配置网卡参数可以使用nmtui命令、nmcli命令、nm-connection-editor命令或者直接编辑网卡配置文件来实现对网卡参数的修改。

2．在RHEL 8系统中使用网卡会话技术的目的是什么？

**答：**使用nmcli命令来管理网卡会话的目的是为了快速切换网卡参数，以便适应不同的工作场景。

3．请简述网卡绑定技术balaner-rr轮循模式的特点。

**答：**平时两块网卡均工作，且自动备援，无须交换机设备提供辅助支持。

4． 在Linux系统中，当通过修改其配置文件中的参数来配置服务程序时，若想要让新配置的参数生效，还需要执行什么操作？

**答：**需要重新启动相关的服务程序，或让服务程序重新加载配置文件，或重启系统。

5．sshd服务的口令验证与密钥验证方式，哪个更安全？

**答：**一般情况下，密钥验证方式更加安全。若用户若认证有更高的安全需求，还可以再对密钥文件进行口令加密，从而实现双重加密。

6． 想要把本地文件/root/out.txt传送到地址为192.168.10.20的远程主机的/home目录下，且本地主机与远程主机均为Linux系统，最为简便的传送方式是什么？

**答：**执行命令scp /root/out.txt root@192.168.10.20:/home，并在进行口令验证后即可开始传送。

7．请简述配置软件仓库的步骤。

**答：**首先应该创建挂载目录并把光盘镜像文件与其关联，然后修改软件仓库的配置文件，填写入相关参数，尤其需要注意挂载目录的存放路径要正确无误，最后便可使用dnf/yum命令来安装相关的服务程序了。

8． tmux服务程序能够让用户实现远程控制的不间断会话服务，即便网络发生中断也不丢失对远程主机的会话控制。那么，当想要恢复到一个名为linux的会话窗口时，应该怎么做呢？

**答：**执行命令tmux attach -t linux即可恢复到这个会话窗口中。

### 集群服务器

### VPN服务器

### 内网穿透服务

#### 基于frp的内网穿透服务

#### 基于Ngrok的内网穿透服务

#### 基于NATAPP的内网穿透服务

### NAT端口转发服务器

#### Firewalld端口转发

#### Firewalld伪装

------

#                                      windows Server2019各种服务器部署

##                                                                                     （ 基于PowerShell命令）

------

### AD RMS（Active  Directory Rights Management Service）

### AD  FS（Active Directory联合身份验证服务器）

### AD  LDS (Active Directory 轻型目录服务)

### AD DS （Active Directory域服务）

一、使用AD域的好处

大企业为了统一管理电脑，公司IT人员都会在公司搭建域控，将公司所有电脑都加入到域中，然后进行权限的集中管理。

公司员工电脑加入域后，可以控制员工的登录权限，文件访问权限，打印权限，电脑配置修改权限等。

阻止一些软件自动升级，用户私自安装软件，电脑的安全得到了一定的防范。

对于用户集中权限管理后，IT管理工作效率高，现在很多企业的ERP软件，加密软件都和AD域进行帐户绑定，分配权限。



![img](http://inews.gtimg.com/newsapp_match/0/10691917660/0)

软件安装，可以进行软件的统一下发等等，减少IT管理工作量。

等等。

二、搭建AD域



![img](http://inews.gtimg.com/newsapp_match/0/10691917661/0)

1、安装windows server 2019操作系统。

2、选中AD域服务器，修改静态IP地址，将DNS设置为127.0.0.1。



![img](http://inews.gtimg.com/newsapp_match/0/10691917663/0)

3、修改AD域服务器的主机名称，自己定义。



![img](http://inews.gtimg.com/newsapp_match/0/10691917664/0)

4、打开服务器管理器，找到右上角“管理”--“添加角色和功能”。



![img](http://inews.gtimg.com/newsapp_match/0/10691917665/0)

5、“开始之前”，启用向导，进行角色或功能添加，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691917666/0)

6、选择默认的“基于角色或基于功能的安装”，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691917667/0)

7、“从服务器池中选择服务器”，现在只有一台，所以默认就选择NJAD01，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921371/0)

8、找到“Active Directory域服务”，勾选。



![img](http://inews.gtimg.com/newsapp_match/0/10691921372/0)

9、勾选后，会跳出如下需要安装的角色或功能，点击“添加功能”。



![img](http://inews.gtimg.com/newsapp_match/0/10691921373/0)

10、现在Active Directory域服务，已经选中，点击下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921374/0)

11、功能，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921375/0)

12、ADDS，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921376/0)

13、选择“安装”。



![img](http://inews.gtimg.com/newsapp_match/0/10691921377/0)

14、正在安装Active Directory 域服务。



![img](http://inews.gtimg.com/newsapp_match/0/10691921378/0)

15、看到已在NJAD01上安装成功，说明AD域的角色添加成功了。那是不是AD域现在就配置完成了呢？



![img](http://inews.gtimg.com/newsapp_match/0/10691921379/0)

16、AD域服务的角色安装成功后，搭建域控的任务还没有结束，还需要进行部署配置。

选择右上角，带感叹号的旗子，点击“将此服务器升级为域控制器”。



![img](http://inews.gtimg.com/newsapp_match/0/10691921380/0)

17、部署配置，如果是公司的首台域控制器，选择“添加新林”，自定义公司的根域名，域名命令规则xxx.com，为什么要用.com结尾呢？一般来说.com注册用户为公司或企业。



![img](http://inews.gtimg.com/newsapp_match/0/10691921381/0)

18、新林和根域的功能级别，默认Windows Server 2016。

指定域控制器功能，域名系统（DNS）服务器，全局编录（GC）都是默认安装在此域控服务器上。



![img](http://inews.gtimg.com/newsapp_match/0/10691921382/0)

19、设置目录服务器叫还原模式（DSRM）密码。Directory Services Restore Mode，简称DSRM，又称目录服务恢复模式。是Windows域控制器的服务器安全模式启动选项。DSRM允许管理员用来修复或还原修复或重建活动目录数据库。



![img](http://inews.gtimg.com/newsapp_match/0/10691921383/0)

20、DNS选项，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921384/0)

21、其它选项，NetBIOS域名，默认，下一步。



![img](http://inews.gtimg.com/newsapp_match/0/10691921385/0)

22、AD DS数据库、日志文件和SYSVOL的文件存储的默认位置。

Ntds.dit：存储活动目录数据库文件，存储域控制器的所有活动目录对象。

Edb*.log：存储日志文件，默认日志文件Edb.log。

SYSVOL，它是用来存储域公共文件服务器副本的共享文件夹，例如我们用得最多的组策略设置、脚本等都是存在这个共享目录中的，script脚本文件，Netlogon共享文件，Sysvol共享文件。



![img](http://inews.gtimg.com/newsapp_match/0/10691921386/0)

23、查看选项中，可以看查看到上面的部署全部信息参数。



![img](http://inews.gtimg.com/newsapp_match/0/10691921387/0)

24、刚才部署了那么多，可以查看刚才的配置脚本。



![img](http://inews.gtimg.com/newsapp_match/0/10691921388/0)

25、“先决条件检查”。



![img](http://inews.gtimg.com/newsapp_match/0/10691921389/0)

发现有一个报错，本地Administrator帐户将成为域Administrator帐户。无法新建域，因为本地的域Administrator帐户密码不符合要求。

这个报错，就是说域服务器的本地Administrator帐户要升级成域Administrator帐户了，之前设置的密码太简单了，需要重新设置一下。



![img](http://inews.gtimg.com/newsapp_match/0/10691921390/0)

打开dos界面，输入命令，命令完成。



![img](http://inews.gtimg.com/newsapp_match/0/10691921391/0)

命令完成后，重置本地管理员密码。



![img](http://inews.gtimg.com/newsapp_match/0/10691921392/0)

26、点击上一步后，然后点下一步，重新检测“先决条件检查”。发现可以正常通过条件检查。点击“安装”。



![img](http://inews.gtimg.com/newsapp_match/0/10691921393/0)

27、正在安装中...



![img](http://inews.gtimg.com/newsapp_match/0/10691921394/0)

28、安装完成后，重新启动域服务器。



![img](http://inews.gtimg.com/newsapp_match/0/10691921395/0)

29、输入密码，登录域服务器。



![img](http://inews.gtimg.com/newsapp_match/0/10691921396/0)

30、在升级成为域服务器后，计算机管理中，就没有用户和组了。



![img](http://inews.gtimg.com/newsapp_match/0/10691921397/0)

31、用户和组都在“工具”--“Active Directory用户和计算机”中。



![img](http://inews.gtimg.com/newsapp_match/0/10691921398/0)

32、进入“Active Directory用户和计算机”后，找到Users可以查找到Administrator等用户。



![img](http://inews.gtimg.com/newsapp_match/0/10691921399/0)

33、域服务器已经搭建完成。



![img](http://inews.gtimg.com/newsapp_match/0/10691921400/0)

### AD  CS (Active Directory证书服务)

### DHCP服务器

### DNS服务器

### Hyper-V服务器

### Web 服务器（IIS）

### Windows Server更新服务

### Windows部署服务

### 传真服务

### 打印和文件服务

### 批量激活服务

### 设备运行状况证明

### 网络策略和访问服务

### 网络控制服器

### 文件和存储服务

#### 存储服务

#### 文件和iSCSI服务

##### 文件服务器

##### DFS复制

##### DFS命名空间

##### iSCSI目标存储提供程序

##### iSCSI目标服务器

##### NFS服务器

##### 工作文件夹

##### 网络文件 BranchCache

##### 文件服务器VSS代理服务

##### 文件服务器资源管理器

##### 重复数据删除

### 远程访问

### 远程桌面服务

### 主机保护者服务

### 内网穿透服务



