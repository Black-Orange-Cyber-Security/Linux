# [第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/basic-learning-12.html)

**章节简述：**

本章首先通过比较文件传输和文件共享这两种资源交换方式来引入Samba服务的理论知识，并介绍SMB协议与Samba服务程序的起源和发展过程。然后通过实验的方式部署文件共享服务来深入了解Samba服务中相关参数的作用，并在实验最后分别使用Windows系统和[Linux系统](https://www.linuxprobe.com/)访问共享的文件资源，确保读者彻底掌握文件共享服务的配置方法

本章还讲解了如何配置网络文件系统（Network File System，NFS）服务来简化[Linux](https://www.linuxprobe.com/)系统之间的文件共享工作，以及通过部署NFS服务在多台Linux系统之间挂载并使用资源。在管理设备挂载信息时，使用autofs服务不仅可以正常满足设备挂载的使用需求，还能进一步提高服务器硬件资源和网络带宽的利用率。

[刘遄](https://www.linuxprobe.com/)老师相信，当各位读者认真学习完本章内容之后，一定会深刻理解在Linux系统之间共享文件资源以及在Linux系统与Windows系统之间共享文件资源的工作机制，并彻底掌握相应的配置方法。

本章目录结构

- 12.1 SAMBA文件共享服务
  - [12.1.1 配置共享资源](https://www.linuxprobe.com/basic-learning-12.html#1211)
  - [12.1.2 Windows挂载共享](https://www.linuxprobe.com/basic-learning-12.html#1212_Windows)
  - [12.1.3 Linux挂载共享](https://www.linuxprobe.com/basic-learning-12.html#1213_Linux)
- [12.2 NFS网络文件系统](https://www.linuxprobe.com/basic-learning-12.html#122_NFS)
- [12.3 AutoFs自动挂载服务](https://www.linuxprobe.com/basic-learning-12.html#123_AutoFs)

##### **12.1 SAMBA文件共享服务**

上一章讲解的FTP文件传输服务确实可以让主机之间的文件传输变得简单方便，但是FTP协议的本质是传输文件，而非共享文件，因此要想通过客户端直接在服务器上修改文件内容还是一件比较麻烦的事情。

1987年，微软公司和英特尔公司共同制定了SMB（Server Messages Block）服务器消息块协议，旨在解决局域网内的文件或打印机等资源的共享问题，这也使得在多个主机之间共享文件变得越来越简单。到了1991年，当时还在读大学的Tridgwell为了解决Linux系统与Windows系统之间的文件共享问题，基于SMB协议开发出了SMBServer服务程序。这是一款开源的文件共享软件，经过简单配置就能够实现Linux系统与Windows系统之间的文件共享工作。当时，Tridgwell想把这款软件的名字SMBServer注册成为商标，但却被商标局以SMB是没有意义的字符而拒绝了申请。后来Tridgwell不断翻看词典，突然看到一个拉丁舞蹈的名字—Samba，而且这个热情洋溢的舞蹈名字中又恰好包含了“SMB”，于是Samba服务程序的名字由此诞生（见图12-1）。Samba服务程序现在已经成为在Linux系统与Windows系统之间共享文件的最佳选择。

![第12章 使用Samba或NFS实现文件共享第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/wp-content/uploads/2015/06/samba%E6%9C%8D%E5%8A%A1logo.jpg)

图12-1 Samba服务程序的logo

Samba服务程序的配置方法与之前讲解的很多服务的配置方法类似，首先需要先通过软件仓库来安装Samba服务程序（Samba服务程序的名字也恰巧是软件包的名字），顺手再安装一个samba-client软件包，这是用于一会测试共享目录的客户端程序：

```
[root@linuxprobe ~]# dnf install samba samba-client
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:17 ago on Fri 05 Mar 2021 04:54:36 PM CST.
Dependencies resolved.
========================================================================================
 Package                     Arch            Version              Repository       Size
========================================================================================
Installing:
 samba                       x86_64          4.9.1-8.el8          BaseOS          708 k
 samba-client                x86_64          4.9.1-8.el8          BaseOS          636 k
Installing dependencies:
 samba-common-tools          x86_64          4.9.1-8.el8          BaseOS          461 k
 samba-libs                  x86_64          4.9.1-8.el8          BaseOS          177 k

Transaction Summary
========================================================================================
Install  4 Packages
………………省略部分输出信息………………
Installed:
  samba-4.9.1-8.el8.x86_64                      samba-client-4.9.1-8.el8.x86_64        
  samba-common-tools-4.9.1-8.el8.x86_64         samba-libs-4.9.1-8.el8.x86_64          

Complete!
```

安装完毕后打开Samba服务程序的主配置文件，好在参数并不多，只有37行。其中第17至22行代表共享该登录用户的家目录内容，虽然某些情况下可以更方便的共享文件，但这个默认操作着实有些危险，建议删除掉不要共享；而24-29行是用SMB协议共享本地的打印机设备，方便局域网内的用户可以远程使用，当前我们没有打印机设备，因此也删除掉不共享；最后的31至37行依然为共享打印机设备的参数，同样建议予以删除。

```
[root@linuxprobe ~]# vim /etc/samba/smb.conf
  1 # See smb.conf.example for a more detailed config file or
  2 # read the smb.conf manpage.
  3 # Run 'testparm' to verify the config is correct after
  4 # you modified it.
  5 
  6 [global]
  7         workgroup = SAMBA
  8         security = user
  9 
 10         passdb backend = tdbsam
 11 
 12         printing = cups
 13         printcap name = cups
 14         load printers = yes
 15         cups options = raw
 16 
 17 [homes]
 18         comment = Home Directories
 19         valid users = %S, %D%w%S
 20         browseable = No
 21         read only = No
 22         inherit acls = Yes
 23 
 24 [printers]
 25         comment = All Printers
 26         path = /var/tmp
 27         printable = Yes
 28         create mask = 0600
 29         browseable = No
 30 
 31 [print$]
 32         comment = Printer Drivers
 33         path = /var/lib/samba/drivers
 34         write list = @printadmin root
 35         force group = @printadmin
 36         create mask = 0664
 37         directory mask = 0775
```

对着Samba服务的主配置文件一顿删减操作，最后的有效配置参数只剩下了8行。所剩不多的参数中，我们还能继续删除不需要的参数，例如5-8行参数中所提到的cups全称叫做Common UNIX Printing System，中文名叫通用UNIX打印系统服务，依然是用于打印机或打印服务器的，继续予以删除。

```
[root@linuxprobe ~]# cat /etc/samba/smb.conf
  1 [global]
  2         workgroup = SAMBA
  3         security = user
  4         passdb backend = tdbsam
  5         printing = cups
  6         printcap name = cups
  7         load printers = yes
  8         cups options = raw
```

### **Tips**

删除掉不需要的代码是常规操作，能够让服务程序“轻装前进”，关闭非必要功能，实现更好的性能，把硬件资源用到刀刃上。其次还能让运维人员更好的找到所需的代码，对比一百行代码来讲，从十行代码中找到一个参数要容易很多。所以只要对参数有正确的认识，那么就大胆的操作吧！

为了避免工作中使用到了打印机服务而不知如何配置，我们对上述代码进行了详细的注释说明，如表12-1所示，供读者们留存备查。

表12-1                   Samba服务程序中的参数以及作用

| 行数 | 参数                                                      | 作用                               |
| ---- | --------------------------------------------------------- | ---------------------------------- |
| 1    | # See smb.conf.example for a more detailed config file or | 注释信息                           |
| 2    | # read the smb.conf manpage.                              |                                    |
| 3    | # Run 'testparm' to verify the config is correct after    |                                    |
| 4    | # you modified it.                                        |                                    |
| 5    | [global]                                                  | 全局参数                           |
| 6    | workgroup = SAMBA                                         | 工作组名称                         |
| 7    |                                                           |                                    |
| 8    | security = user                                           | 安全验证的方式，总共有4种          |
| 9    |                                                           |                                    |
| 10   | passdb backend = tdbsam                                   | 定义用户后台的类型，总共有3种      |
| 11   |                                                           |                                    |
| 12   | printing = cups                                           | 打印服务协议                       |
| 13   | printcap name = cups                                      | 打印服务名称                       |
| 14   | load printers = yes                                       | 是否加载打印机                     |
| 15   | cups options = raw                                        | 打印机的选项                       |
| 16   |                                                           |                                    |
| 17   | [homes]                                                   | 共享名称                           |
| 18   | comment = Home Directories                                | 描述信息                           |
| 19   | valid users = %S, %D%w%S                                  | 可用账户                           |
| 20   | browseable = No                                           | 指定共享信息是否在“网上邻居”中可见 |
| 21   | read only = No                                            | 是否只读                           |
| 22   | inherit acls = Yes                                        | 是否继承访问控制列表               |
| 23   |                                                           |                                    |
| 24   | [printers]                                                | 共享名称                           |
| 25   | comment = All Printers                                    | 描述信息                           |
| 26   | path = /var/tmp                                           | 共享路径                           |
| 27   | printable = Yes                                           | 是否可打印                         |
| 28   | create mask = 0600                                        | 文件权限                           |
| 29   | browseable = No                                           | 指定共享信息是否在“网上邻居”中可见 |
| 30   |                                                           |                                    |
| 31   | [print$]                                                  | 共享名称                           |
| 32   | comment = Printer Drivers                                 | 描述信息                           |
| 33   | path = /var/lib/samba/drivers                             | 共享路径                           |
| 34   | write list = @printadmin root                             | 可写入文件的用户列表               |
| 35   | force group = @printadmin                                 | 用户组列表                         |
| 36   | create mask = 0664                                        | 文件权限                           |
| 37   | directory mask = 0775                                     | 目录权限                           |



上面代码中security参数代表用户登录samba服务时的验证方式，总共有4种可用参数：“share”代表主机无需验证口令，相当于vsftpd服务的匿名公开访问模式，比较方便，但安全性很差；“user”代表登录samba服务时需要使用账号密码进行验证，通过后才能获取到文件，这是默认的验证方式，最为常用；“domain”代表通过域控制器进行身份验证，限制用户的来源域；“server”代表使用独立主机验证来访用户的提供的口令，相当于是集中管理账号，并不常用。

在最早期的Linux系统中，samba服务使用的是pam模块调用本地账号和密码信息，后来在5、6版本时替。换成了用smbpasswd[命令](https://www.linuxcool.com/)设置独立的samba服务账号和密码口令。到了RHEL 7/8版本时则又进行了一次改革，将传统的验证方式换成了tdbsam数据库，这是一个专门用于保存samba服务账号口令的数据库，用户需要用pdbedit[命令](https://www.linuxcool.com/)进行独立的添加操作，下面章节中会有实战演示。

###### **12.1.1 配置共享资源**

Samba服务程序的主配置文件与前面学习过的Apache服务很相似，包括全局配置参数和区域配置参数。全局配置参数用于设置整体的资源共享环境，对里面的每一个独立的共享资源都有效。区域配置参数则用于设置单独的共享资源，且仅对该资源有效。创建共享资源的方法很简单，只要将表12-2中的参数写入到Samba服务程序的主配置文件中，然后重启该服务即可。

表12-2  用于设置Samba服务程序的参数以及作用

| 参数                                                  | 作用                       |
| ----------------------------------------------------- | -------------------------- |
| [database]                                            | 共享名称为database         |
| comment = Do not arbitrarily modify the database file | 警告用户不要随意修改数据库 |
| path = /home/database                                 | 共享目录为/home/database   |
| public = no                                           | 关闭“所有人可见”           |
| writable = yes                                        | 允许写入操作               |



**第1步**：创建用于访问共享资源的账户信息。在RHEL 8系统中，Samba服务程序默认使用的是用户口令认证模式（user）。这种认证模式可以确保仅让有密码且受信任的用户访问共享资源，而且验证过程也十分简单。不过，只有建立账户信息数据库之后，才能使用用户口令认证模式。另外，Samba服务程序的数据库要求账户必须在当前系统中已经存在，否则日后创建文件时将导致文件的权限属性混乱不堪，由此引发错误。

pdbedit命令用于管理samba服务程序的账户信息数据库，格式为“pdbedit [选项] 账户”。在第一次把账户信息写入到数据库时需要使用-a参数，以后在执行修改密码、删除账户等操作时就不再需要该参数了。pdbedit命令中使用的参数以及作用如表12-3所示。

表12-3                    用于pdbedit命令的参数以及作用

| 参数      | 作用                   |
| --------- | ---------------------- |
| -a 用户名 | 建立Samba用户          |
| -x 用户名 | 删除Samba用户          |
| -L        | 列出用户列表           |
| -Lv       | 列出用户详细信息的列表 |



```
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
[root@linuxprobe ~]# pdbedit -a -u linuxprobe
new password:此处输入该账户在Samba服务数据库中的密码
retype new password:再次输入密码进行确认
Unix username:        linuxprobe
NT username:          
Account Flags:        [U          ]
User SID:             S-1-5-21-650031181-3622628401-3290108334-1000
Primary Group SID:    S-1-5-21-650031181-3622628401-3290108334-513
Full Name:            linuxprobe
Home Directory:       \\linuxprobe\linuxprobe
HomeDir Drive:        
Logon Script:         
Profile Path:         \\linuxprobe\linuxprobe\profile
Domain:               LINUXPROBE
Account desc:         
Workstations:         
Munged dial:          
Logon time:           0
Logoff time:          Wed, 06 Feb 2036 23:06:39 CST
Kickoff time:         Wed, 06 Feb 2036 23:06:39 CST
Password last set:    Fri, 05 Mar 2021 18:52:35 CST
Password can change:  Fri, 05 Mar 2021 18:52:35 CST
Password must change: never
Last bad password   : 0
Bad password count  : 0
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF
```

**第2步**：创建用于共享资源的文件目录。在创建时，不仅要考虑到文件读写权限的问题，而且由于/home目录是系统中普通用户的家目录，因此还需要考虑应用于该目录的SELinux安全上下文所带来的限制。在samba的帮助手册中告诉用户正确的文件上下文值应该是samba_share_t，所以只需要修改完毕后执行restorecon命令，就能让应用于目录的新SELinux安全上下文立即生效。

```
[root@linuxprobe ~]# mkdir /home/database
[root@linuxprobe ~]# chown -Rf linuxprobe:linuxprobe /home/database
[root@linuxprobe ~]# semanage fcontext -a -t samba_share_t /home/database
[root@linuxprobe ~]# restorecon -Rv /home/database
Relabeled /home/database from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:samba_share_t:s0
```

**第3步**：设置SELinux服务与策略，使其允许通过Samba服务程序访问普通用户家目录。执行getsebool命令，筛选出所有与Samba服务程序相关的SELinux域策略，根据策略的名称（和经验）选择出正确的策略条目进行开启即可：

```
[root@linuxprobe ~]# getsebool -a | grep samba
samba_create_home_dirs --> off
samba_domain_controller --> off
samba_enable_home_dirs --> off
samba_export_all_ro --> off
samba_export_all_rw --> off
samba_load_libgfapi --> off
samba_portmapper --> off
samba_run_unconfined --> off
samba_share_fusefs --> off
samba_share_nfs --> off
sanlock_use_samba --> off
tmpreaper_use_samba --> off
use_samba_home_dirs --> off
virt_use_samba --> off
[root@linuxprobe ~]# setsebool -P samba_enable_home_dirs on
```

**第4步**：在Samba服务程序的主配置文件中，根据表12-2所提到的格式写入共享信息。

```
[root@linuxprobe ~]# vim /etc/samba/smb.conf 
[global]
        workgroup = SAMBA
        security = user
        passdb backend = tdbsam
[database]
        comment = Do not arbitrarily modify the database file
        path = /home/database
        public = no
        writable = yes
```

**第5步**：Samba服务程序的配置工作基本完毕。Samba服务程序在Linux系统中的名字为smb，所以重启并加入到启动项中，保证在重启服务器后依然能够为用户持续提供服务。

```
[root@linuxprobe ~]# systemctl restart smb 
[root@linuxprobe ~]# systemctl enable smb 
Created symlink /etc/systemd/system/multi-user.target.wants/smb.service → /usr/lib/systemd/system/smb.service.
```

避免防火墙会限制用户访问，因此决定将iptables防火墙清空，再把samba服务添加到firewalld防火墙中，确保万无一失。

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables-save 
[root@linuxprobe ~]# firewall-cmd --zone=public --permanent --add-service=samba
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**第6步：**在服务器本地检查samba服务是否启动可以用“systemctl status smb”进行查看，而如果想进一步看samba服务都共享出去了哪些共享目录，则可以用smbclient命令来查看共享详情，-U参数指定了用户名称，建议一会用哪位用户进行挂载，就用哪位用户身份进行查看；-L参数列举共享清单。

```
[root@linuxprobe ~]# smbclient -U linuxprobe -L 192.168.10.10
Enter SAMBA\linuxprobe's password: 此处输入该账户在Samba服务数据库中的密码

	Sharename       Type      Comment
	---------       ----      -------
	database        Disk      Do not arbitrarily modify the database file
	IPC$            IPC       IPC Service (Samba 4.9.1)
Reconnecting with SMB1 for workgroup listing.

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
```

###### **12.1.2 Windows挂载共享**

无论Samba共享服务是部署Windows系统上还是部署在Linux系统上，通过Windows系统进行访问时，其步骤和方法都是一样的。下面假设Samba共享服务部署在Linux系统上，并通过Windows系统来访问Samba服务。Samba共享服务器和Windows客户端的IP地址可以根据表12-4来设置。

表12-4        Samba服务器和Windows客户端使用的操作系统以及IP地址



| 主机名称        | 操作系统   | IP地址        |
| --------------- | ---------- | ------------- |
| Samba共享服务器 | RHEL 8     | 192.168.10.10 |
| Linux客户端     | RHEL 8     | 192.168.10.20 |
| Windows客户端   | Windows 10 | 192.168.10.30 |



要在Windows系统中访问共享资源，只需要点击开始按钮后输入两个反斜杠，然后再加服务器的IP地址即可，如图12-2所示。

![第12章 使用Samba或NFS实现文件共享第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%9C%A8Windows%E7%B3%BB%E7%BB%9F%E4%B8%AD%E8%AE%BF%E9%97%AE%E5%85%B1%E4%BA%AB%E8%B5%84%E6%BA%90.png)

图12-2 在Windows系统中访问共享资源

现在就应该能看到Samba共享服务的登录界面了。[刘遄](https://www.linuxprobe.com/)老师在这里先使用linuxprobe账户的系统本地密码尝试登录，结果出现了如图12-3所示的报错信息。由此可以验证，在RHEL 8系统中，Samba服务程序使用的果然是独立的账户信息数据库。所以，即便在Linux系统中有一个linuxprobe账户，Samba服务程序使用的账户信息数据库中也有一个同名的linuxprobe账户，大家也一定要弄清楚它们各自所对应的密码。

![第12章 使用Samba或NFS实现文件共享第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BF%E9%97%AESamba%E5%85%B1%E4%BA%AB%E6%9C%8D%E5%8A%A1%E6%8F%90%E7%A4%BA%E5%87%BA%E9%94%99.png)

图12-3 访问Samba共享服务提示出错

正确输入linuxprobe账户名以及使用pdbedit命令设置的密码后，就可以登录到共享界面中了，如图12-4所示。此时，我们可以尝试执行查看、写入、更名、删除文件等操作。

![第12章 使用Samba或NFS实现文件共享第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%88%90%E5%8A%9F%E8%AE%BF%E9%97%AESamba%E5%85%B1%E4%BA%AB%E6%9C%8D%E5%8A%A1-1.png)

图12-4 成功访问Samba共享服务

由于Windows系统的缓存原因，有可能您在第二次登录时提供了正确的账户和密码，依然会报错，这时只需要重新启动一下Windows客户端就没问题了（如果Windows系统依然报错，请检查上述步骤是否有做错的地方）。

###### **12.1.3 Linux挂载共享**

上面的实验操作可能会让各位读者误以为Samba服务程序只是为了解决Linux系统和Windows系统的资源共享问题而设计的。其实，Samba服务程序还可以实现Linux系统之间的文件共享。请各位读者按照表12-5来设置Samba服务程序所在主机（即Samba共享服务器）和Linux客户端使用的IP地址，然后在客户端安装支持文件共享服务的软件包（cifs-utils）。

表12-5      Samba共享服务器和Linux客户端各自使用的操作系统以及IP地址

| 主机名称        | 操作系统   | IP地址        |
| --------------- | ---------- | ------------- |
| Samba共享服务器 | RHEL 8     | 192.168.10.10 |
| Linux客户端     | RHEL 8     | 192.168.10.20 |
| Windows客户端   | Windows 10 | 192.168.10.30 |



```
[root@linuxprobe ~]# dnf install cifs-utils
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                                    3.1 MB/s | 3.2 kB     00:00    
BaseOS                                                       2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
=============================================================================================
 Package                Arch               Version                  Repository          Size
=============================================================================================
Installing:
 cifs-utils             x86_64             6.8-2.el8                BaseOS              93 k

Transaction Summary
=============================================================================================
Install  1 Package         
………………省略部分输出信息………………    
Installed:
  cifs-utils-6.8-2.el8.x86_64                                                     
Complete!
```

安装好软件包后，在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，可以与服务器上的共享名称同名，这样便于日后查找。mount命令的-t参数指定协议类型，-o参数指定用户命和密码，最后追加上服务器IP地址和共享名称和本地挂载目录即可。服务器IP地址后面的共享名称指的是配置文件中[database]的值，而不是服务器本地挂载的目录名称，虽然这两个值可能一样，但读者们应该认出它们的区别。

```
[root@linuxprobe ~]# mkdir /database
[root@linuxprobe ~]# mount -t cifs -o username=linuxprobe,password=redhat //192.168.10.10/database /database
[root@linuxprobe ~]# df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  969M     0  969M   0% /dev
tmpfs                     984M     0  984M   0% /dev/shm
tmpfs                     984M  9.6M  974M   1% /run
tmpfs                     984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root      17G  3.9G   14G  23% /
/dev/sr0                  6.7G  6.7G     0 100% /media/cdrom
/dev/sda1                1014M  152M  863M  15% /boot
tmpfs                     197M   16K  197M   1% /run/user/42
tmpfs                     197M  3.4M  194M   2% /run/user/0
//192.168.10.10/database   17G  3.9G   14G  23% /database
```

如果说每次重启电脑后都需要再手动的mount挂载一下远程共享目录，是不是觉得很麻烦呢？其实我们可以按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中，然后让/etc/fstab文件和系统自动的加载它。为了保证不被其他人随意看到，最后把这个认证文件的权限修改为仅root管理员才能够读写：

```
[root@linuxprobe ~]# vim auth.smb
username=linuxprobe
password=redhat
domain=MYGROUP
[root@linuxprobe ~]# chmod 600 auth.smb
```

挂载信息写入到/etc/fstab文件中，以确保共享挂载信息在服务器重启后依然生效：

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Thu Feb 25 10:42:11 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                      /             xfs     defaults                    0 0
UUID=37d0bdc6-d70d-4cc0-b356-51195ad90369  /boot         xfs     defaults                    1 0
/dev/mapper/rhel-swap                      swap          swap    defaults                    0 0
/dev/cdrom                                 /media/cdrom  iso9660 defaults                    0 0 
//192.168.10.10/database                   /database     cifs    credentials=/root/auth.smb  0 0
[root@linuxprobe ~]# mount -a
```

Linux客户端成功地挂载了Samba服务的共享资源。进入到挂载目录/database后就可以看到Windows系统访问Samba服务程序时留下来的文件了（即文件Memo.txt）。当然，也可以对该文件进行读写操作并保存。

```
[root@linuxprobe ~]# cat /database/Memo.txt
i can edit it .
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **12.2 NFS网络文件系统**

如果读者们觉得Samba服务程序的配置太麻烦，而且恰巧需要共享文件的主机都是Linux系统，非常推荐大家在客户端部署NFS服务来共享文件。NFS网络文件系统服务可以将远程Linux系统上的文件共享资源挂载到本地主机的目录上，从而使得本地主机（Linux客户端）基于TCP/IP协议，像使用本地主机上的资源那样读写远程Linux系统上的共享文件。

![第12章 使用Samba或NFS实现文件共享第12章 使用Samba或NFS实现文件共享](https://www.linuxprobe.com/wp-content/uploads/2021/03/NFS.png)

由于RHEL 8系统中默认已经安装了NFS服务，外加NFS服务的配置步骤也很简单，因此刘遄老师在授课时会戏称为Need For Speed极品飞车。接下来，准备配置NFS服务。首先请使用软件仓库检查自己的RHEL 8系统中是否已经安装了NFS软件包：

```
[root@linuxprobe ~]# dnf install nfs-utils
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:12 ago on Sat 06 Mar 2021 04:48:38 AM CST.
Package nfs-utils-1:2.3.3-14.el8.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

**第1步**：为了检验NFS服务配置的效果，我们需要使用两台Linux主机（一台充当NFS服务器，一台充当NFS客户端），并按照表12-6来设置它们所使用的IP地址。

表12-6               两台Linux主机所使用的操作系统以及IP地址

| 主机名称  | 操作系统 | IP地址        |
| --------- | -------- | ------------- |
| NFS服务器 | RHEL 8   | 192.168.10.10 |
| NFS客户端 | RHEL 8   | 192.168.10.20 |



另外，不要忘记配置好防火墙，以免默认的防火墙策略禁止正常的NFS共享服务。

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables-save
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=nfs
success
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=rpc-bind
success
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=mountd
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**第2步**：在NFS服务器上建立用于NFS文件共享的目录，并设置足够的权限确保其他人也有写入权限。

```
[root@linuxprobe ~]# mkdir /nfsfile
[root@linuxprobe ~]# chmod -R 777 /nfsfile
[root@linuxprobe ~]# echo "welcome to linuxprobe.com" > /nfsfile/readme
```

**第3步**：NFS服务程序的配置文件为/etc/exports，默认情况下里面没有任何内容。我们可以按照“共享目录的路径 允许访问的NFS客户端（共享权限参数）”的格式，定义要共享的目录与相应的权限。

例如，如果想要把/nfsfile目录共享给192.168.10.0/24网段内的所有主机，让这些主机都拥有读写权限，在将数据写入到NFS服务器的硬盘中后才会结束操作，最大限度保证数据不丢失，以及把来访客户端root管理员映射为本地的匿名用户等，则可以按照下面命令中的格式，将表12-7中的参数写到NFS服务程序的配置文件中。

表12-7                 用于配置NFS服务程序配置文件的参数

| 参数           | 作用                                                         |
| -------------- | ------------------------------------------------------------ |
| ro             | 只读                                                         |
| rw             | 读写                                                         |
| root_squash    | 当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户     |
| no_root_squash | 当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员   |
| all_squash     | 无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户   |
| sync           | 同时将数据写入到内存与硬盘中，保证不丢失数据                 |
| async          | 优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据 |



请注意，NFS客户端地址与权限之间没有空格。

```
[root@linuxprobe ~]# vim /etc/exports
/nfsfile 192.168.10.*(rw,sync,root_squash)
```

在NFS服务的配置文件中巧用通配符能够实现很多便捷功能，就比如匹配IP地址就有三种方法——第一种是直接写*号，代表任何主机都可以访问；第二种则是实验中采用的192.168.10.*通配格式，代表来自192.168.10.0/24网段的主机；第三种则是直接写对方的IP地址，如192.168.10.20，代表仅允许某个主机进行访问。

**第4步**：启动和启用NFS服务程序。由于在使用NFS服务进行文件共享之前，需要使用RPC（Remote Procedure Call，远程过程调用）服务将NFS服务器的IP地址和端口号等信息发送给客户端。因此，在启动NFS服务之前，还需要顺带重启并启用rpcbind服务程序，并将这两个服务一并加入开机启动项中。

```
[root@linuxprobe ~]# systemctl restart rpcbind
[root@linuxprobe ~]# systemctl enable rpcbind
[root@linuxprobe ~]# systemctl start nfs-server
[root@linuxprobe ~]# systemctl enable nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```

NFS客户端的配置步骤也十分简单。先使用showmount命令查询NFS服务器的远程共享信息，必要的参数见表12-8，其输出格式为“共享的目录名称 允许使用客户端地址”。

表12-8                 showmount命令中可用的参数以及作用

| 参数 | 作用                                      |
| ---- | ----------------------------------------- |
| -e   | 显示NFS服务器的共享列表                   |
| -a   | 显示本机挂载的文件资源的情况NFS资源的情况 |
| -v   | 显示版本号                                |



```
[root@linuxprobe ~]# showmount -e 192.168.10.10
Export list for 192.168.10.10:
/nfsfile 192.168.10.*
```

然后在NFS客户端创建一个挂载目录。使用mount命令并结合-t参数，指定要挂载的文件系统的类型，并在命令后面写上服务器的IP地址、服务器上的共享目录以及要挂载到本地系统（即客户端）的目录。

```
[root@linuxprobe ~]# mkdir /nfsfile
[root@linuxprobe ~]# mount -t nfs 192.168.10.10:/nfsfile /nfsfile
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
192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile
```

挂载成功后就应该能够顺利地看到在执行前面的操作时写入的文件内容了。如果希望NFS文件共享服务能一直有效，则需要将其写入到fstab文件中：

```
[root@linuxprobe ~]# cat /nfsfile/readme
welcome to linuxprobe.com
[root@linuxprobe ~]# vim /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Thu Feb 25 10:42:11 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                     /                       xfs     defaults        0 0
UUID=37d0bdc6-d70d-4cc0-b356-51195ad90369 /boot                   xfs     defaults        0 0
/dev/mapper/rhel-swap                     swap                    swap    defaults        0 0
/dev/cdrom                                /media/cdrom            iso9660 defaults        0 0 
192.168.10.10:/nfsfile                    /nfsfile                nfs     defaults        0 0   
```

##### **12.3 AutoFs自动挂载服务**

无论是Samba服务还是NFS服务，都要把挂载信息写入到/etc/fstab中，这样远程共享资源就会自动随服务器开机而进行挂载。虽然这很方便，但是如果挂载的远程资源太多，则会给网络带宽和服务器的硬件资源带来很大负载。如果在资源挂载后长期不使用，也会造成服务器硬件资源的浪费。可能会有读者说，“可以在每次使用之前执行mount命令进行手动挂载”。这是一个不错的选择，但是每次都需要先挂载再使用，您不觉得麻烦吗？

autofs自动挂载服务可以帮我们解决这一问题。与mount命令不同，autofs服务程序是一种Linux系统守护进程，当检测到用户试图访问一个尚未挂载的文件系统时，将自动挂载该文件系统。换句话说，将挂载信息填入/etc/fstab文件后，系统在每次开机时都自动将其挂载，而autofs服务程序则是在用户需要使用该文件系统时才去动态挂载，从而节约了网络资源和服务器的硬件资源。

需要自行安装下autofs服务程序：

```
[root@linuxprobe ~]# dnf install autofs
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:28:58 ago on Sat 06 Mar 2021 04:57:01 AM CST.
Dependencies resolved.
========================================================================================
 Package           Arch              Version                    Repository         Size
========================================================================================
Installing:
 autofs            x86_64            1:5.1.4-29.el8             BaseOS            755 k

Transaction Summary
========================================================================================
Install  1 Package
………………省略部分输出信息………………
Installed:
  autofs-1:5.1.4-29.el8.x86_64                                                          

Complete!
```

处于生产环境中的Linux服务器，一般会同时管理许多设备的挂载操作。如果把这些设备挂载信息都写入到autofs服务的主配置文件中，无疑会让主配置文件臃肿不堪，不利于服务执行效率，也不利于日后修改里面的配置内容，因此在autofs服务程序的主配置文件中需要按照“挂载目录 子配置文件”的格式进行填写。挂载目录是设备挂载位置的上一级目录。例如，光盘设备一般挂载到/media/cdrom目录中，那么挂载目录写成/media即可。对应的子配置文件则是对这个挂载目录内的挂载设备信息作进一步的说明。子配置文件需要用户自行定义，文件名字没有严格要求，但后缀建议以.misc结束。具体的配置参数如第7行的加粗字所示。

```
[root@linuxprobe ~]# vim /etc/auto.master
#
# Sample auto.master file
# This is a 'master' automounter map and it has the following format:
# mount-point [map-type[,format]:]map [options]
# For details of the format look at auto.master(5).
#
/media  /etc/iso.misc
/misc   /etc/auto.misc
#
# NOTE: mounts done from a hosts map will be mounted with the
#       "nosuid" and "nodev" options unless the "suid" and "dev"
#       options are explicitly given.
#
/net    -hosts
#
# Include /etc/auto.master.d/*.autofs
# The included files must conform to the format of this file.
#
+dir:/etc/auto.master.d
#
# If you have fedfs set up and the related binaries, either
# built as part of autofs or installed from another package,
# uncomment this line to use the fedfs program map to access
# your fedfs mounts.
#/nfs4  /usr/sbin/fedfs-map-nfs4 nobind
#
# Include central master map if it can be found using
# nsswitch sources.
#
# Note that if there are entries for /net or /misc (as
# above) in the included master map any keys that are the
# same will not be seen as the first read key seen takes
# precedence.
#
+auto.master
```

在子配置文件中，应按照“挂载目录 挂载文件类型及权限 :设备名称”的格式进行填写。例如，要把光盘设备挂载到/media/iso目录中，可将挂载目录写为iso，而-fstype为文件系统格式参数，iso9660为光盘设备格式，ro、nosuid及nodev为光盘设备具体的权限参数，/dev/cdrom则是定义要挂载的设备名称。配置完成后再顺手将autofs服务程序启动并加入到系统启动项中：

```
[root@linuxprobe ~]# vim /etc/iso.misc
iso   -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom
[root@linuxprobe ~]# systemctl start autofs 
[root@linuxprobe ~]# systemctl enable autofs 
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.
```

接下来将发生一件非常有趣的事情。先查看当前的光盘设备挂载情况，确认光盘设备没有被挂载上，而且/media目录中根本就没有iso子目录：

```
[root@linuxprobe ~]# umount /dev/cdrom
[root@linuxprobe ~]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile
[root@linuxprobe ~]# cd /media
[root@linuxprobe media]# ls
[root@linuxprobe media]#
```

但是，我们却可以使用cd命令切换到这个iso子目录中，而且光盘设备会被立即自动挂载上，也就能顺利查看光盘内的内容了。

```
[root@linuxprobe media]# cd iso
[root@linuxprobe iso]# ls
AppStream  EULA              images      RPM-GPG-KEY-redhat-beta
BaseOS     extra_files.json  isolinux    RPM-GPG-KEY-redhat-release
EFI        GPL               media.repo  TRANS.TBL
[root@linuxprobe iso]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile
/dev/sr0                6.7G  6.7G     0 100% /media/iso
```

### **Tips**

咦？怎么光盘设备的名称变成了/dev/sr0呢？实际上和/dev/cdrom是连接方式，只是名称不同而已。

```
[root@linuxprobe ~]# ls -l /dev/cdrom
lrwxrwxrwx. 1 root root 3 Feb 26 00:09 /dev/cdrom -> sr0
```

是不是很有方便，趁着刚学完的知识还没忘，再对NFS服务动手试试吧。

首先应该把NFS共享目录先卸载掉，然后在autofs服务程序的主配置文件中会有一条“/misc /etc/auto.misc”参数，这个auto.misc相当于自动挂载的参考文件，而它默认就已经存在，所以这步我们不需要进行任何操作：

```
[root@linuxprobe ~]# umount /nfsfile
[root@linuxprobe ~]# vim /etc/auto.master
#
# Sample auto.master file
# This is a 'master' automounter map and it has the following format:
# mount-point [map-type[,format]:]map [options]
# For details of the format look at auto.master(5).
#
/media   /etc/iso.misc
/misc    /etc/auto.misc
………………省略部分输出信息………………
```

接下来找到这个对应的auto.misc文件，填写本地挂载的路径和NFS服务器的挂载信息，如下所示：

```
[root@linuxprobe ~]# vim /etc/auto.misc
#
# This is an automounter map and it has the following format
# key [ -mount-options-separated-by-comma ] location
# Details may be found in the autofs(5) manpage
nfsfile         192.168.10.10:/nfsfile
cd              -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom

# the following entries are samples to pique your imagination
#linux          -ro,soft,intr           ftp.example.org:/pub/linux
#boot           -fstype=ext2            :/dev/hda1
#floppy         -fstype=auto            :/dev/fd0
#floppy         -fstype=ext2            :/dev/fd0
#e2floppy       -fstype=ext2            :/dev/fd0
#jaz            -fstype=ext2            :/dev/sdc1
#removable      -fstype=ext2            :/dev/hdd
```

这样填写信息后重启autofs服务程序，进入到/misc/nfsfile目录时，共享信息便会自动挂载：

```
[root@linuxprobe ~]# systemctl restart autofs
[root@linuxprobe ~]# cd /misc/nfsfile
[root@linuxprobe nfsfile]# df -h
Filesystem              Size  Used Avail Use% Mounted on
devtmpfs                969M     0  969M   0% /dev
tmpfs                   984M     0  984M   0% /dev/shm
tmpfs                   984M  9.6M  974M   1% /run
tmpfs                   984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root    17G  3.9G   14G  23% /
/dev/sda1              1014M  152M  863M  15% /boot
tmpfs                   197M   16K  197M   1% /run/user/42
tmpfs                   197M  3.4M  194M   2% /run/user/0
192.168.10.10:/nfsfile   17G  3.9G   14G  23% /misc/nfsfile
/dev/sr0                6.7G  6.7G     0 100% /media/iso
```

真棒！又get到了一个全新的技能，有了autofs服务就能让工作更加的便捷了呢，不用总想着挂载设备的问题，它就能帮我们自动完成了~稍作休息，继续前进！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．要想实现Linux系统与Windows系统之间的文件共享，能否使用NFS服务？

**答：**不可以，应该使用Samba服务程序，NFS服务仅能实现Linux系统之间的文件共享。

2．用于管理Samba服务程序的独立账户信息数据库的命令是什么？

**答：**pdbedit命令用于管理Samba服务程序的账户信息数据库。

3．简述在Windows系统中使用Samba服务程序来共享资源的方法。

**答：**在开始菜单的输入框中按照\\192.168.10.10的格式输入访问命令并回车执行即可。在Windows的“运行”命令框中按照“\\192.168.10.10”的格式输入访问命令并按回车键即可。

4．简述在Linux系统中使用Samba服务程序来共享资源的步骤方法。

**答：**首先应创建密码认证文件以及挂载目录，然后把挂载信息写入到/etc/fstab文件中，最后执行mount -a命令挂载使用。

5．如果在Linux系统中默认没有安装NFS服务程序，则需要安装什么软件包呢？

**答：**NFS服务程序的软件包名字为nfs-utils，因此执行yum install nfs-utils命令即可。

6．在使用NFS服务共享资源时，若希望无论NFS客户端使用什么帐户来访问共享资源，都会被映射为本地匿名用户，则需要添加哪个参数。

**答：**需要添加all_squash参数，以便更好地保证服务器的安全。

7．客户端在查看到远程NFS服务器上的共享资源列表时，需要使用哪个命令？

**答：**使用showmount命令即可看到NFS服务器上的资源共享情况。

8．简述autofs服务程序的作用。

**答：**实现动态灵活的设备挂载操作，而且只有检测到用户试图访问一个尚未挂载的文件系统时，才自动挂载该文件系统。

