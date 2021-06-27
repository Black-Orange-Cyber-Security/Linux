# [第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/basic-learning-07.html)

**章节简述：**

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



 