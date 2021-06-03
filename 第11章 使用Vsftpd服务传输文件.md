# [第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/basic-learning-11.html)

**章节简述：**

本章开篇讲解了什么是文件传输协议（File Transfer Protocol，FTP），以及如何部署vsftpd服务程序，然后深度剖析了vsftpd主配置文件中最常用的参数及其作用，并完整演示了vsftpd服务程序三种认证模式（匿名开放模式、本地用户模式、虚拟用户模式）的配置方法。本章还涵盖了可插拔认证模块（Pluggable Authentication Module，PAM）的原理、作用以及实用配置方法。

读者将通过本章介绍的实战内容进一步练习SE[Linux](https://www.linuxprobe.com/)服务的配置方法，掌握简单文件传输协议（Trivial File Transfer Protocol，TFTP）的理论及配置方法，以及学习[刘遄](https://www.linuxprobe.com/)老师在服务部署和排错方面的经验技巧，以便灵活应对生产环境中遇到的各种问题。

本章目录结构

- [11.1 文件传输协议](https://www.linuxprobe.com/basic-learning-11.html#111)
- 11.2 Vsftpd服务程序
  - [11.2.1 匿名访问模式](https://www.linuxprobe.com/basic-learning-11.html#1121)
  - [11.2.2 本地用户模式](https://www.linuxprobe.com/basic-learning-11.html#1122)
  - [11.2.3 虚拟用户模式](https://www.linuxprobe.com/basic-learning-11.html#1123)
- [11.3 TFTP简单文件传输协议](https://www.linuxprobe.com/basic-learning-11.html#113_TFTP)

##### **11.1 文件传输协议**

一般来讲，人们把计算机联网的首要目的就是获取资料，而文件传输是一种非常重要的获取资料的方式。今天的互联网是由几千万台个人计算机、工作站、服务器、小型机、大型机、巨型机等具有不同型号、不同架构的物理设备共同组成的，而且即便是个人计算机，也可能会装有Windows、Linux、UNIX、Mac等不同的操作系统。为了能够在如此复杂多样的设备之间解决问题解决文件传输问题，FTP（File Transfer Protocol）文件传输协议应运而生。

FTP是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用20、21号端口，其中端口20用于进行数据传输，端口21用于接受客户端发出的相关FTP[命令](https://www.linuxcool.com/)与参数。FTP服务器普遍部署于内网中，具有容易搭建、方便管理的特点。而且有些FTP客户端工具还可以支持文件的多点下载以及断点续传技术，因此得到了广大用户的青睐。FTP协议的传输拓扑如图11-1所示。

![第11章 使用Vsftpd服务传输文件第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/wp-content/uploads/2015/07/FTP%E8%BF%9E%E6%8E%A5%E8%BF%87%E7%A8%8B.png)

图11-1  FTP协议的传输拓扑

FTP服务器是按照FTP协议在互联网上提供文件存储和访问服务的主机，FTP客户端则是向服务器发送连接请求，以建立数据传输链路的主机。FTP协议有下面两种工作模式，第8章在学习防火墙服务配置时曾经讲过，防火墙一般是用于过滤从外网进入内网的流量，因此有些时候需要将FTP的工作模式设置为主动模式，才可以传输数据。

> **主动模式**：FTP服务器主动向客户端发起连接请求。
>
> **被动模式**：FTP服务器等待客户端发起连接请求（默认工作模式）。

由于FTP、HTTP、Telnet等协议的数据都是经过明文进行传输，因此从设计的原理上就是不可靠的，但人们又需要解决文件传输的需求，因此便有了vsftpd服务程序。vsftpd（very secure ftp daemon，非常安全的FTP守护进程）是一款运行在Linux操作系统上的FTP服务程序，不仅完全开源而且免费，此外，还具有很高的安全性、传输速度，以及支持虚拟用户验证等其他FTP服务程序不具备的特点。在不影响使用的前提下，能够让管理者自行决定是公开匿名、本地用户还是虚拟用户的验证方式，这样即便被骇客拿到了我们的账号密码，也不见得能登陆的了服务器。

在配置妥当软件仓库之后，就可以安装vsftpd服务程序了，yum与dnf[命令](https://www.linuxcool.com/)都可以，优先选择用dnf命令方式。

```
[root@linuxprobe ~]# dnf install vsftpd
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                               3.1 MB/s | 3.2 kB     00:00    
BaseOS                                                  2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
========================================================================================
 Package           Arch              Version                 Repository            Size
========================================================================================
Installing:
 vsftpd            x86_64            3.0.3-28.el8            AppStream            180 k

Transaction Summary
========================================================================================
Install  1 Package

Total size: 180 k
Installed size: 356 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                1/1 
  Installing       : vsftpd-3.0.3-28.el8.x86_64                                     1/1 
  Running scriptlet: vsftpd-3.0.3-28.el8.x86_64                                     1/1 
  Verifying        : vsftpd-3.0.3-28.el8.x86_64                                     1/1 
Installed products updated.

Installed:
  vsftpd-3.0.3-28.el8.x86_64                                                            

Complete!
```

iptables防火墙管理工具默认禁止了FTP传输协议的端口号，因此在正式配置vsftpd服务程序之前，为了避免这些默认的防火墙策略“捣乱”，还需要清空iptables防火墙的默认策略，并把当前已经被清理的防火墙策略状态保存下来：

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables-save 
```

然后再把FTP协议添加到firewalld服务的允许列表中，前期准备工作一定要做充足：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=ftp 
success
[root@linuxprobe ~]# firewall-cmd --reload 
success
```

vsftpd服务程序的主配置文件（/etc/vsftpd/vsftpd.conf）内容总长度达到127行，但其中大多数参数在开头都添加了井号（#），从而成为注释信息，大家没有必要在注释信息上花费太多的时间。我们可以在grep命令后面添加-v参数，过滤并反选出没有包含井号（#）的参数行（即过滤掉所有的注释信息），然后将过滤后的参数行通过输出重定向符写回原始的主配置文件中，只剩下12行有效参数了，马上就不紧张了：

```
[root@linuxprobe ~]# mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_bak
[root@linuxprobe ~]# grep -v "#" /etc/vsftpd/vsftpd.conf_bak > /etc/vsftpd/vsftpd.conf
[root@linuxprobe ~]# cat /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
```

表11-1中罗列了vsftpd服务程序主配置文件中常用的参数以及作用。当前大家只需要简单了解即可，在后续的实验中将演示这些参数的用法，以帮助大家熟悉并掌握。

表11-1                   vsftpd服务程序常用的参数以及作用

| 参数                                              | 作用                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| listen=[YES\|NO]                                  | 是否以独立运行的方式监听服务                                 |
| listen_address=IP地址                             | 设置要监听的IP地址                                           |
| listen_port=21                                    | 设置FTP服务的监听端口                                        |
| download_enable＝[YES\|NO]                        | 是否允许下载文件                                             |
| userlist_enable=[YES\|NO] userlist_deny=[YES\|NO] | 设置用户列表为“允许”还是“禁止”操作                           |
| max_clients=0                                     | 最大客户端连接数，0为不限制                                  |
| max_per_ip=0                                      | 同一IP地址的最大连接数，0为不限制                            |
| anonymous_enable=[YES\|NO]                        | 是否允许匿名用户访问                                         |
| anon_upload_enable=[YES\|NO]                      | 是否允许匿名用户上传文件                                     |
| anon_umask=022                                    | 匿名用户上传文件的umask值                                    |
| anon_root=/var/ftp                                | 匿名用户的FTP根目录                                          |
| anon_mkdir_write_enable=[YES\|NO]                 | 是否允许匿名用户创建目录                                     |
| anon_other_write_enable=[YES\|NO]                 | 是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限） |
| anon_max_rate=0                                   | 匿名用户的最大传输速率（字节/秒），0为不限制                 |
| local_enable=[YES\|NO]                            | 是否允许本地用户登录FTP                                      |
| local_umask=022                                   | 本地用户上传文件的umask值                                    |
| local_root=/var/ftp                               | 本地用户的FTP根目录                                          |
| chroot_local_user=[YES\|NO]                       | 是否将用户权限禁锢在FTP目录，以确保安全                      |
| local_max_rate=0                                  | 本地用户最大传输速率（字节/秒），0为不限制                   |



##### **11.2 Vsftpd服务程序**

vsftpd作为更加安全的文件传输协议服务程序，允许用户以三种认证模式登录到FTP服务器上。

**匿名开放模式**：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。

**本地用户模式**：是通过[Linux系统](https://www.linuxprobe.com/)本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被骇客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

**虚拟用户模式**：更安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使骇客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

ftp是Linux系统中以命令行界面的方式来管理FTP传输服务的客户端工具。我们首先手动安装这个ftp客户端工具，以便在后续实验中查看结果。

```
[root@linuxprobe ~]# dnf install ftp
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:30 ago on Tue 02 Mar 2021 09:38:50 PM CST.
Dependencies resolved.
========================================================================================
 Package         Arch               Version                 Repository             Size
========================================================================================
Installing:
 ftp             x86_64             0.17-78.el8             AppStream              70 k

Transaction Summary
========================================================================================
Install  1 Package

Total size: 70 k
Installed size: 112 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                1/1 
  Installing       : ftp-0.17-78.el8.x86_64                                         1/1 
  Running scriptlet: ftp-0.17-78.el8.x86_64                                         1/1 
  Verifying        : ftp-0.17-78.el8.x86_64                                         1/1 
Installed products updated.

Installed:
  ftp-0.17-78.el8.x86_64                                                                

Complete!
```

如果一会想用Windows主机测试实验的效果，可以从FileZilla、FireFTP、SmartFTP、WinSCP和Cyberduck中挑一个喜欢的软件从网上下载，会比ftp命令功能更加强大。

###### **11.2.1 匿名访问模式**

前文提到，在vsftpd服务程序中，匿名开放模式是最不安全的一种认证模式。任何人都可以无需密码验证而直接登录到FTP服务器。这种模式一般用来访问不重要的公开文件（在生产环境中尽量不要存放重要文件）。当然，如果采用第8章中介绍的防火墙管理工具（如Tcp_wrappers服务程序）将vsftpd服务程序允许访问的主机范围设置为企业内网，也可以提供基本的安全性。

vsftpd服务程序默认关闭了匿名开放模式，需要做的就是开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。需要注意的是，针对匿名用户放开这些权限会带来潜在危险，我们只是为了在Linux系统中练习配置vsftpd服务程序而放开了这些权限，不建议在生产环境中如此行事。表11-2罗列了可以向匿名用户开放的权限参数以及作用。

表11-2                 向匿名用户开放的权限参数以及作用

| 参数                        | 作用                               |
| --------------------------- | ---------------------------------- |
| anonymous_enable=YES        | 允许匿名访问模式                   |
| anon_umask=022              | 匿名用户上传文件的umask值          |
| anon_upload_enable=YES      | 允许匿名用户上传文件               |
| anon_mkdir_write_enable=YES | 允许匿名用户创建目录               |
| anon_other_write_enable=YES | 允许匿名用户修改目录名称或删除目录 |



```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
  1 anonymous_enable=YES
  2 anon_umask=022
  3 anon_upload_enable=YES
  4 anon_mkdir_write_enable=YES
  5 anon_other_write_enable=YES
  6 local_enable=YES
  7 write_enable=YES
  8 local_umask=022
  9 dirmessage_enable=YES
 10 xferlog_enable=YES
 11 connect_from_port_20=YES
 12 xferlog_std_format=YES
 13 listen=NO
 14 listen_ipv6=YES
 15 pam_service_name=vsftpd
 16 userlist_enable=YES
```

在vsftpd服务程序的主配置文件中正确填写参数，然后保存并退出。还需要重启vsftpd服务程序，让新的配置参数生效。在此需要提醒各位读者，在生产环境中或者在RHCSA、[RHCE](https://www.linuxprobe.com/)、[RHCA](https://www.linuxprobe.com/)认证考试中一定要把配置过的服务程序加入到开机启动项中，以保证服务器在重启后依然能够正常提供传输服务：

```
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

现在就可以在客户端执行ftp命令连接到远程的FTP服务器了。在vsftpd服务程序的匿名开放认证模式下，其账户统一为anonymous，密码为空。而且在连接到FTP服务器后，默认访问的是/var/ftp目录。可以切换到该目录下的pub目录中，然后尝试创建一个新的目录文件，以检验是否拥有写入权限：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Permission denied.
```

系统显示拒绝创建目录！我们明明在前面清空了iptables防火墙策略，而且也在vsftpd服务程序的主配置文件中添加了允许匿名用户创建目录和写入文件的权限啊。建议先不要着急往下看，而是自己思考一下这个问题的解决办法，以锻炼您的Linux系统排错能力。

前文提到，在vsftpd服务程序的匿名开放认证模式下，默认访问的是/var/ftp目录。查看该目录的权限得知，只有root管理员才有写入权限。怪不得系统会拒绝操作呢！下面将目录的所有者身份改成系统账户ftp即可，这样应该可以了吧？

```
[root@linuxprobe ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 2 root root 6 Aug 13 2021 /var/ftp/pub
[root@linuxprobe ~]# chown -R ftp /var/ftp/pub
[root@linuxprobe ~]# ls -ld /var/ftp/pub
drwxr-xr-x. 2 ftp root 6 Aug 13 2021 /var/ftp/pub
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Create directory operation failed.
```

系统再次报错！尽管在使用ftp命令登入FTP服务器后，再创建目录时系统依然提示操作失败，但是报错信息却发生了变化。在没有写入权限时，系统提示“权限拒绝”（Permission denied）所以[刘遄](https://www.linuxprobe.com/)老师怀疑是权限的问题。但现在系统提示“创建目录的操作失败”（Create directory operation failed），想必各位读者也应该意识到是SELinux服务在“捣乱”了吧。

下面使用getsebool命令查看与FTP相关的SELinux域策略都有哪些：

```
[root@linuxprobe ~]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
```

我们可以根据经验（需要长期培养，别无它法）和策略的名称判断出是ftpd_full_access--> off策略规则导致了操作失败。接下来修改该策略规则，并且在设置时使用-P参数让修改过的策略永久生效，确保在服务器重启后依然能够顺利写入文件。

```
[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

等SELinux域策略修改完毕，现在便能够顺利执行文件创建、修改及删除等操作了：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
257 "/pub/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
ftp> exit
221 Goodbye.
```

在上面的操作中，由于权限的不足所以将/var/ftp/pub目录的所有者设置成了ftp用户本身。而除了这种方法外，也可以通过设置权限的方法让其它用户获取到写入权限，例如777这样的权限。但是由于vsftpd服务自身带有安全保护机制，不要对/var/ftp直接修改权限，有可能导致服务被“安全锁定”而不能登录，一定要记得是里面的pub目录哦：

```
[root@linuxprobe ~]# chmod -R 777 /var/ftp
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): anonymous
331 Please specify the password.
Password:此处敲击回车即可
500 OOPS: vsftpd: refusing to run with writable root inside chroot()
Login failed.
421 Service not available, remote server has closed connection
```

### **Tips**

再次提醒各位读者，在进行下一次实验之前，一定记得将虚拟机还原到最初始的状态，以免多个实验相互产生冲突。

###### **11.2.2 本地用户模式**

相较于匿名开放模式，本地用户模式要更安全，而且配置起来也很简单。如果大家之前用的是匿名开放模式，现在就可以将它关了，然后开启本地用户模式。针对本地用户模式的权限参数以及作用如表11-3所示。

表11-3                  本地用户模式使用的权限参数以及作用

| 参数                | 作用                                              |
| ------------------- | ------------------------------------------------- |
| anonymous_enable=NO | 禁止匿名访问模式                                  |
| local_enable=YES    | 允许本地用户模式                                  |
| write_enable=YES    | 设置可写权限                                      |
| local_umask=022     | 本地用户模式创建文件的umask值                     |
| userlist_deny=YES   | 启用“禁止用户名单”，名单文件为ftpusers和user_list |
| userlist_enable=YES | 开启用户作用名单文件功能                          |



默认情况下本地用户所需的参数都已经在了，不需要修改。而umask这个参数还是头一次见到，一般中文被称为权限掩码或权限补码，能够直接影响到新建文件的权限值。例如Linux系统中新建普通文件后权限是644，新建的目录权限是755，虽然用户都习以为常了，但为什么是这个数呢？

首先不得不说到其实普通文件的默认权限应该是666，目录权限会是777，这是写在系统配置文件中的。但默认值不等于最终权限值，根据公式“默认权限 - umask = 实际权限”，而umask值默认是022，所以实际文件到手就剩下644，目录文件剩下755了。

同学们没有明白没关系，再举一个例子。每个人的收入都要交税，税就相当于umask值，如果政府想让每个人到手的收入多一些，那么就减少税（umask），如果想让每个人到手的收入少一些，那么就多加税（umask）。这样大家就明白了，umask实际是权限的反掩码，通过它可以调整文件最终的权限大小。

好啦说的有点远了，先来配置本地用户的参数吧：

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
  1 anonymous_enable=NO
  2 local_enable=YES
  3 write_enable=YES
  4 local_umask=022
  5 dirmessage_enable=YES
  6 xferlog_enable=YES
  7 connect_from_port_20=YES
  8 xferlog_std_format=YES
  9 listen=NO
 10 listen_ipv6=YES
 11 pam_service_name=vsftpd
 12 userlist_enable=YES
```

在vsftpd服务程序的主配置文件中正确填写参数，然后保存并退出。还需要重启vsftpd服务程序，让新的配置参数生效。在执行完上一个实验后还原了虚拟机的读者，还需要将配置好的服务添加到开机启动项中，以便在系统重启自后依然可以正常使用vsftpd服务。

```
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

按理来讲，现在已经完全可以本地用户的身份登录FTP服务器了。但是在使用root管理员登录后，系统提示如下的错误信息：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): root
530 Permission denied.
Login failed.
ftp> 
```

可见，在我们输入root管理员的密码之前，就已经被系统拒绝访问了。这是因为vsftpd服务程序所在的目录中默认存放着两个名为“用户名单”的文件（ftpusers和user_list）。不知道大家是否已看过一部日本电影“死亡笔记”，里面就提到有一个黑色封皮的小本子，只要将别人的名字写进去，这人就会挂掉。vsftpd服务程序目录中的这两个文件也有类似的功能—只要里面写有某位用户的名字，就不再允许这位用户登录到FTP服务器上。

```
[root@linuxprobe ~]# cat /etc/vsftpd/user_list 
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
[root@linuxprobe ~]# cat /etc/vsftpd/ftpusers 
# Users that are not allowed to login via ftp
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
```

果然如此！vsftpd服务程序为了保证服务器的安全性而默认禁止了root管理员和大多数系统用户的登录行为，这样可以有效地避免骇客通过FTP服务对root管理员密码进行暴力破解。如果您确认在生产环境中使用root管理员不会对系统安全产生影响，只需按照上面的提示删除掉root用户名即可，也可以选择ftpusers和user_list文件中没有的一个普通用户尝试登录FTP服务器：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): root
331 Please specify the password.
Password:此处输入该用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

在继续后面实验之前，不知道同学们有没有思考一个问题——为什么同样是禁止用户登录的功能，却要做两个一摸一样的文件呢？

这个小玄机其实在user_list文件上面，如果把上面主配置文件中userlist_deny参数值改成NO，那么user_list列表就变成了强制白名单，功能完全是反过来的，只允许列表内的用户访问，拒绝其他人。

另外在采用本地用户模式登录FTP服务器后，默认访问的是该用户的家目录，而且该目录的默认所有者、所属组都是该用户自己，因此不存在写入权限不足的情况。但是当前的操作仍然被拒绝，是因为我们刚才将虚拟机系统还原到最初的状态了。为此，需要再次开启SELinux域中对FTP服务的允许策略：

```
[root@linuxprobe ~]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

刘遄老师再啰嗦几句。在实验课程和生产环境中设置SELinux域策略时，一定记得添加-P参数，否则服务器在重启后就会按照原有的策略进行控制，从而导致配置过的服务无法使用。

在配置妥当后再使用本地用户尝试登录下FTP服务器，分别执行文件的创建、重命名及删除等命令。操作均成功！

```
[root@linuxprobe vsftpd]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): root
331 Please specify the password.
Password:此处输入该用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
257 "/root/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
ftp> exit
221 Goodbye.
```

**请注意：当您完成本实验后请还原虚拟机快照再进行下一个实验，否则可能导致配置文件冲突而报错。**

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

###### **11.2.3 虚拟用户模式**

最后讲解的虚拟用户模式是这三种模式中最安全的一种认证模式，是专门创建出一个账号来登录FTP传输服务的，而不能用于SSH登录服务器。当然，因为安全性较之于前面两种模式有了提升，所以配置流程也会稍微复杂一些。

**第1步**：重置安装vsftpd服务后。创建用于进行FTP认证的用户数据库文件，其中奇数行为账户名，偶数行为密码。例如，分别创建出zhangsan和lisi两个用户，密码均为redhat：

```
[root@linuxprobe ~]# cd /etc/vsftpd/
[root@linuxprobe vsftpd]# vim vuser.list
zhangsan
redhat
lisi
redhat
```

但是，明文信息既不安全，也不符合让vsftpd服务程序直接加载的格式，因此需要使用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。

```
[root@linuxprobe vsftpd]# db_load -T -t hash -f vuser.list vuser.db
[root@linuxprobe vsftpd]# chmod 600 vuser.db
[root@linuxprobe vsftpd]# rm -f vuser.list
```

**第2步**：创建vsftpd服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户。FTP服务用于存储文件的根目录指的是，当虚拟用户登录后所访问的默认位置。

由于Linux系统中的每一个文件都有所有者、所属组属性，例如使用虚拟账户“张三”新建了一个文件，但是系统中找不到账户“张三”，就会导致这个文件的权限出现错误。为此，需要再创建一个可以映射到虚拟用户的系统本地用户。简单来说，就是让虚拟用户默认登录到与之有映射关系的这个系统本地用户的家目录中，虚拟用户创建的文件的属性也都归属于这个系统本地用户，从而避免Linux系统无法处理虚拟用户所创建文件的属性权限。

为了方便管理FTP服务器上的数据，可以把这个系统本地用户的家目录设置为/var目录（该目录用来存放经常发生改变的数据）。并且为了安全起见，我们将这个系统本地用户设置为不允许登录FTP服务器，这不会影响虚拟用户登录，而且还能够避免骇客通过这个系统本地用户进行登录。

```
[root@linuxprobe ~]# useradd -d /var/ftproot -s /sbin/nologin virtual
[root@linuxprobe ~]# ls -ld /var/ftproot/
drwx------. 3 virtual virtual 74 Jul 14 17:50 /var/ftproot/
[root@linuxprobe ~]# chmod -Rf 755 /var/ftproot/
```

**第3步**：建立用于支持虚拟用户的PAM文件。

PAM可插拔认证模块是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式。要想把PAM功能和作用完全讲透，至少要一个章节的篇幅才行（对该主题感兴趣的读者敬请关注本书的进阶篇，里面会详细讲解PAM）。

通俗来讲，PAM是一组安全机制的模块，系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行任何修改。PAM采取了分层设计的思想，包含应用程序层、应用接口层、鉴别模块层，其结构如图11-2所示。

![第11章 使用Vsftpd服务传输文件第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/wp-content/uploads/2021/03/PAM%E7%9A%84%E5%88%86%E5%B1%82%E8%AE%BE%E8%AE%A1%E7%BB%93%E6%9E%84.jpg)

图11-2 PAM的分层设计结构

新建一个用于虚拟用户认证的PAM文件vsftpd.vu，其中PAM文件内的“db=”参数为使用db_load命令生成的账户密码数据库文件的路径，但不用写数据库文件的后缀：

```
[root@linuxprobe ~]# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```

**第4步**：在vsftpd服务程序的主配置文件中通过pam_service_name参数将PAM认证文件的名称修改为vsftpd.vu，PAM作为应用程序层与鉴别模块层的连接纽带，可以让应用程序根据需求灵活地在自身插入所需的鉴别功能模块。当应用程序需要PAM认证时，则需要在应用程序中定义负责认证的PAM配置文件，实现所需的认证功能。

例如，在vsftpd服务程序的主配置文件中默认就带有参数pam_service_name=vsftpd，表示登录FTP服务器时是根据/etc/pam.d/vsftpd文件进行安全认证的。现在我们要做的就是把vsftpd主配置文件中原有的PAM认证文件vsftpd修改为新建的vsftpd.vu文件即可。该操作中用到的参数以及作用如表11-4所示。

表11-4              利用PAM文件进行认证时使用的参数以及作用

| 参数                       | 作用                                                        |
| -------------------------- | ----------------------------------------------------------- |
| anonymous_enable=NO        | 禁止匿名开放模式                                            |
| local_enable=YES           | 允许本地用户模式                                            |
| guest_enable=YES           | 开启虚拟用户模式                                            |
| guest_username=virtual     | 指定虚拟用户账户                                            |
| pam_service_name=vsftpd.vu | 指定PAM文件                                                 |
| allow_writeable_chroot=YES | 允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求 |



```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf  
  1 anonymous_enable=NO
  2 local_enable=YES
  3 write_enable=YES
  4 guest_enable=YES
  5 guest_username=virtual
  6 allow_writeable_chroot=YES
  7 local_umask=022
  8 dirmessage_enable=YES
  9 xferlog_enable=YES
 10 connect_from_port_20=YES
 11 xferlog_std_format=YES
 12 listen=NO
 13 listen_ipv6=YES
 14 pam_service_name=vsftpd.vu
 15 userlist_enable=YES
```

**第5步**：为虚拟用户设置不同的权限。虽然账户zhangsan和lisi都是用于vsftpd服务程序认证的虚拟账户，但是我们依然想对这两人进行区别对待。比如，允许张三上传、创建、修改、查看、删除文件，只允许李四查看文件。这可以通过vsftpd服务程序来实现。只需新建一个目录，在里面分别创建两个以zhangsan和lisi命名的文件，其中在名为zhangsan的文件中写入允许的相关权限（使用匿名用户的参数）：

```
[root@linuxprobe ~]# mkdir /etc/vsftpd/vusers_dir/
[root@linuxprobe ~]# cd /etc/vsftpd/vusers_dir/
[root@linuxprobe vusers_dir]# touch lisi
[root@linuxprobe vusers_dir]# vim zhangsan
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

然后再次修改vsftpd主配置文件，通过添加user_config_dir参数来定义这两个虚拟用户不同权限的配置文件所存放的路径。为了让修改后的参数立即生效，需要重启vsftpd服务程序并将该服务添加到开机启动项中：

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf
  1 anonymous_enable=NO
  2 local_enable=YES
  3 write_enable=YES
  4 guest_enable=YES
  5 guest_username=virtual
  6 allow_writeable_chroot=YES
  7 local_umask=022
  8 dirmessage_enable=YES
  9 xferlog_enable=YES
 10 connect_from_port_20=YES
 11 xferlog_std_format=YES
 12 listen=NO
 13 listen_ipv6=YES
 14 pam_service_name=vsftpd.vu
 15 userlist_enable=YES
 16 user_config_dir=/etc/vsftpd/vusers_dir
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

**第6步**：设置SELinux域允许策略，然后使用虚拟用户模式登录FTP服务器。相信大家可以猜到，SELinux会继续来捣乱。所以，先按照前面实验中的步骤开启SELinux域的允许策略，以免再次出现操作失败的情况：

```
[root@linuxprobe ~]# getsebool -a | grep ftp
ftpd_anon_write --> off
ftpd_connect_all_unreserved --> off
ftpd_connect_db --> off
ftpd_full_access --> off
ftpd_use_cifs --> off
ftpd_use_fusefs --> off
ftpd_use_nfs --> off
ftpd_use_passive_mode --> off
httpd_can_connect_ftp --> off
httpd_enable_ftp_server --> off
tftp_anon_write --> off
tftp_home_dir --> off
[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

此时，不但可以使用虚拟用户模式成功登录到FTP服务器，还可以分别使用账户zhangsan和lisi来检验他们的权限。李四用户只能登录，没有其余权限：

```
[root@linuxprobe ~]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): lisi
331 Please specify the password.
Password:此处输入虚拟用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
550 Permission denied.
ftp> exit
221 Goodbye.
```

而张三用户不仅可以登录，还可以创建、改名和删除文件，满权限状态。当然，读者在生产环境中一定要根据真实需求来灵活配置参数，不要照搬这里的实验操作。

```
[root@linuxprobe vusers_dir]# ftp 192.168.10.10
Connected to 192.168.10.10 (192.168.10.10).
220 (vsFTPd 3.0.3)
Name (192.168.10.10:root): zhangsan
331 Please specify the password.
Password:此处输入虚拟用户的密码
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> mkdir files
257 "/files" created
ftp> rename files database
350 Ready for RNTO.
250 Rename successful.
ftp> rmdir database
250 Remove directory operation successful.
```

最后总结下FTP文件传输服务登陆后默认所在的位置，如表11-5所示，这样登录后心里总是有底的，不必担心把文件传错了目录。

表11-5              vsftpd服务程序登陆后所在目录

| 登录方式 | 默认目录             |
| -------- | -------------------- |
| 匿名公开 | /var/ftp             |
| 本地用户 | 该用户的家目录       |
| 虚拟用户 | 对应映射用户的家目录 |



##### **11.3 TFTP简单文件传输协议**

简单文件传输协议（Trivial File Transfer Protocol，TFTP）是一种基于UDP协议在客户端和服务器之间进行简单文件传输的协议。顾名思义，它提供不复杂、开销不大的文件传输服务，可将其当作FTP协议的简化版本。

TFTP的命令功能不如FTP服务强大，甚至不能遍历目录，在安全性方面也弱于FTP服务。而且，由于TFTP在传输文件时采用的是UDP协议，占用的端口号为69，因此文件的传输过程也不像FTP协议那样可靠。但是，因为TFTP不需要客户端的权限认证，也就减少了无谓的系统和网络带宽消耗，因此在传输琐碎（trivial）不大的文件时，效率更高。

接下来在系统上安装相关的软件包，进行体验。tftp-server是服务程序，tftp是用于连接测试的客户端工具，xinetd是管理服务，一会咱们讲到：

```
[root@linuxprobe ~]# dnf install tftp-server tftp xinetd
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                               3.1 MB/s | 3.2 kB     00:00    
BaseOS                                                  2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
========================================================================================
 Package              Arch            Version                  Repository          Size
========================================================================================
Installing:
 tftp                 x86_64          5.2-24.el8               AppStream           42 k
 tftp-server          x86_64          5.2-24.el8               AppStream           50 k
 xinetd               x86_64          2:2.3.15-23.el8          AppStream          135 k

Transaction Summary
========================================================================================
Install  3 Packages

Total size: 227 k
Installed size: 397 k
Is this ok [y/N]: y
Downloading Packages:
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                1/1 
  Installing       : xinetd-2:2.3.15-23.el8.x86_64                                  1/3 
  Running scriptlet: xinetd-2:2.3.15-23.el8.x86_64                                  1/3 
  Installing       : tftp-server-5.2-24.el8.x86_64                                  2/3 
  Running scriptlet: tftp-server-5.2-24.el8.x86_64                                  2/3 
  Installing       : tftp-5.2-24.el8.x86_64                                         3/3 
  Running scriptlet: tftp-5.2-24.el8.x86_64                                         3/3 
  Verifying        : tftp-5.2-24.el8.x86_64                                         1/3 
  Verifying        : tftp-server-5.2-24.el8.x86_64                                  2/3 
  Verifying        : xinetd-2:2.3.15-23.el8.x86_64                                  3/3 
Installed products updated.

Installed:
  tftp-5.2-24.el8.x86_64  tftp-server-5.2-24.el8.x86_64  xinetd-2:2.3.15-23.el8.x86_64 

Complete!
```

在Linux系统中，TFTP服务是使用xinetd服务程序来管理的。xinetd服务可以用来管理多种轻量级的网络服务，而且具有强大的日志功能，专门用于管理那些比较小的应用程序的开关工作，有点类似于有独立控制的插线板那样，如图11-3所示，想开启那个服务就编辑对应的xinetd配置文件的开关参数。

![第11章 使用Vsftpd服务传输文件第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%9C%AA%E6%A0%87%E9%A2%98-1-1-1.jpg)

图11-3 一个带有独立开关的插线板

简单来说，在安装TFTP软件包后，还需要在xinetd服务程序中将其开启。在RHEL 8版本系统中tftp所对应的配置文件默认不存在，需要用户根据示例文件（/usr/share/doc/xinetd/sample.conf）自行创建。读者们直接复制下面内容到文件中，可直接使用：

```
[root@linuxprobe ~]# vim /etc/xinetd.d/tftp
service tftp
{
        socket_type             = dgram
        protocol                = udp
        wait                    = yes
        user                    = root
        server                  = /usr/sbin/in.tftpd
        server_args             = -s /var/lib/tftpboot
        disable                 = no
        per_source              = 11
        cps                     = 100 2
        flags                   = IPv4
}
```

然后，重启xinetd服务并将它添加到系统的开机启动项中，以确保TFTP服务在系统重启后依然处于运行状态。考虑到有些系统的防火墙默认没有允许UDP协议的69端口，因此需要手动将该端口号加入到防火墙的允许策略中：

```
[root@linuxprobe ~]# systemctl restart xinetd
[root@linuxprobe ~]# systemctl enable xinetd
[root@linuxprobe ~]# firewall-cmd --zone=public --permanent --add-port=69/udp
success
[root@linuxprobe ~]# firewall-cmd --reload 
success
```

TFTP的根目录为/var/lib/tftpboot。我们可以使用刚安装好的tftp命令尝试访问其中的文件，亲身体验TFTP服务的文件传输过程。在使用tftp命令访问文件时，可能会用到表11-5中的参数。

表11-5                     tftp命令中可用的参数以及作用

| 参数    | 作用                |
| ------- | ------------------- |
| ?       | 帮助信息            |
| put     | 上传文件            |
| get     | 下载文件            |
| verbose | 显示详细的处理信息  |
| status  | 显示当前的状态信息  |
| binary  | 使用二进制进行传输  |
| ascii   | 使用ASCII码进行传输 |
| timeout | 设置重传的超时时间  |
| quit    | 退出                |



```
[root@linuxprobe ~]# echo "i love linux" > /var/lib/tftpboot/readme.txt
[root@linuxprobe ~]# tftp 192.168.10.10
tftp> get readme.txt
tftp> quit
[root@linuxprobe ~]# ls
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  readme.txt  Videos
Desktop          Downloads  Music                 Public    Templates
[root@linuxprobe ~]# cat readme.txt 
i love linux
```

当然，TFTP服务的玩法还不止于此，第19章会将TFTP服务与其他软件相搭配，组合出一套完整的自动化部署系统方案。

大家继续加油！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．简述FTP协议的功能作用以及所占用的端口号。

**答：**FTP是一种在互联网中进行文件传输的协议，默认使用20、21号端口，其中端口20（数据端口）用于进行数据传输，端口21（命令端口）用于接受客户端发起的相关FTP命令与参数。

2．vsftpd服务程序提供的三种用户认证模式各自有什么特点？

**答：**匿名开放模式是任何人都可以无需密码认证即可直接登录到FTP服务器的验证方式；本地用户模式是通过系统本地的账户密码信息登录到FTP服务器的认证方式；虚拟用户模式是通过创建独立的FTP用户数据库文件来进行认证并登录到FTP服务器的认证方式，相较来说它也是最安全的认证模式。

3． 使用匿名开放模式登录到一台用vsftpd服务程序部署的FTP服务器上时，默认的FTP根目录是什么？

**答：**使用匿名开放模式登录后的FTP根目录是/var/ftp目录，该目录内默认还会有一个名为pub的子目录。

4．简述PAM的功能作用。

**答：**PAM是一组安全机制的模块（插件），系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行过多修改。

5．使用虚拟用户模式登录FTP服务器的所有用户的权限都是一样的吗？

**答：**不一定，可以通过分别定义用户权限文件来为每一位用户设置不同的权限。

6．TFTP协议与FTP协议有什么不同？

**答：**TFTP协议提供不复杂、开销不大的文件传输服务（可将其当作FTP协议的简化版本）。