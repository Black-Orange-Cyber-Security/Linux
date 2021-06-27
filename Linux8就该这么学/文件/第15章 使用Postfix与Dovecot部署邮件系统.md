# [第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/basic-learning-15.html)

**章节概述：**

Email电子邮件系统是我们在日常工作、生活中最常用的一个网络服务，本章将首先介绍电子邮件系统的起源，然后介绍SMTP、POP3、IMAP4等常见的电子邮件协议，以及MUA、MTA、MDA这三种服务角色的作用。本章将完整地演示在[Linux系统](https://www.linuxprobe.com/)中使用Postfix和Dovecot服务程序配置电子邮件系统服务的方法，并重点讲解常用的配置参数，此外还将结合BIND服务程序提供的DNS域名解析服务来验证客户端主机与服务器之间的邮件收发功能。

本章最后还介绍了如何在电子邮件系统中设置用户别名，以帮助大家在生产环境中更好地控制、管理电子邮件账户以及信箱地址。使用Thunderbird客户端完成日常的邮件收发工作，把办公环境迁移到[Linux](https://www.linuxprobe.com/)系统上并不困难。

本章目录结构

- 15.1 电子邮件系统
  - [15.2.1 配置Postfix服务程序](https://www.linuxprobe.com/basic-learning-15.html#1521_Postfix)
  - [15.2.2 配置Dovecot服务程序](https://www.linuxprobe.com/basic-learning-15.html#1522_Dovecot)
  - [15.2.3 客户使用电子邮件系统](https://www.linuxprobe.com/basic-learning-15.html#1523)
- [15.3 设置用户别名邮箱](https://www.linuxprobe.com/basic-learning-15.html#153)
- [15.4 Linux邮件客户端](https://www.linuxprobe.com/basic-learning-15.html#154_Linux)

##### **15.1 电子邮件系统**

20世纪60年代，美苏两国正处于冷战时期。美国军方认为应该在科学技术上保持其领先的地位，这样有助于在未来的战争中取得优势。美国国防部由此发起了一项名为ARPANET的科研项目，即大家现在所熟知的阿帕网计划。阿帕网是当今互联网的雏形，它也是世界上第一个运营的封包交换网络。但是很快在1971年阿帕网遇到了严峻的问题，如图15-1所示，参与阿帕网科研项目的科学家分布在美国不同的地区，甚至还会因为时差的影响而不能及时分享各自的研究成果，因此科学家们迫切需要一种能够借助于网络在计算机之间传输数据的方法。

尽管本书第10章和第11章介绍的Web服务和FTP文件传输服务也能实现数据交换，但是这些服务的数据传输方式就像“打电话”那样，需要双方同时在线才能完成传输工作。如果对方的主机宕机或者科研人员因故离开，就有可能错过某些科研成果了。好在当时麻省理工学院的Ray Tomlinson博士也参与到了阿帕网计划的科研项目中，他觉得有必要设计一种类似于“信件”的传输服务，并为信件准备一个“信箱”，这样即便对方临时离线也能完成数据的接收，等上线后再进行处理即可。于是，Ray Tomlinson博士用了近一年的时间完成了电子邮件（Email）的设计，并在1971年秋天使用SNDMSG软件向自己的另一台计算机发送出了人类历史上第一封电子邮件—电子邮件系统在互联网中由此诞生！

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2015/08/arpanet3_small.gif)

图15-1 1971年阿帕网科研项目运营情况历史资料图片

既然要在互联网中给他人发送电子邮件，那么对方用户用于接收电子邮件的名称必须是唯一的，否则电子邮件可能会同时发给多个重名的用户，也或者干脆大家都收不到邮件了。因此，Ray Tomlinson博士决定选择使用“姓名@计算机主机名称”的格式来规范电子信箱的名称。选择使用@符号作为间隔符的原因其实也很简单，因为Ray Tomlinson博士觉得人类的名字和计算机主机名称中应该不会有这么一个@符号，所以就选择了这个符号。

电子邮件系统基于邮件协议来完成电子邮件的传输，常见的邮件协议有下面这些。

> **简单邮件传输协议（Simple Mail Transfer Protocol，SMTP）**：用于发送和中转发出的电子邮件，占用服务器的25/TCP端口。
>
> **邮局协议版本3（Post Office Protocol 3）**：用于将电子邮件存储到本地主机，占用服务器的110/TCP端口。
>
> **Internet消息访问协议版本4（Internet Message Access Protocol 4）**：用于在本地主机上访问邮件，占用服务器的143/TCP端口。

在电子邮件系统中，为用户收发邮件的服务器名为邮件用户代理（Mail User Agent，MUA）。另外，既然电子邮件系统能够让用户在离线的情况下依然可以完成数据的接收，肯定得有一个用于保存用户邮件的“信箱”服务器，这个服务器的名字为邮件投递代理（Mail Delivery Agent，MDA），其工作职责是把来自于邮件传输代理（Mail Transfer Agent，MTA）的邮件保存到本地的收件箱中。其中，这个MTA的工作职责是转发处理不同电子邮件服务供应商之间的邮件，把来自于MUA的邮件转发到合适的MTA服务器。例如，我们从新浪信箱向谷歌信箱发送一封电子邮件，这封电子邮件的传输过程如图15-2所示。

总的来说，一般的网络服务程序在传输信息时就像拨打电话，需要双方同时保持在线，而在电子邮件系统中，当用户发送邮件后不必等待投递工作完成即可下线。如果对方邮件服务器（MTA）宕机或对方临时离线，则发件服务器（MTA）就会把要发送的内容自动的暂时保存到本地，等检测到对方邮件服务器恢复后会立即再次投递，期间一般无需运维人员维护处理，随后收信人（MUA）就能在自己的信箱中找到这封邮件了。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2015/08/%E9%82%AE%E4%BB%B6%E6%8A%95%E9%80%92%E5%B7%A5%E7%A8%8B.png)

图15-2 电子邮件的传输过程

大家在生产环境中部署企业级的电子邮件系统时，有4个注意事项请留意。

> **添加反垃圾与反病毒模块：**它能够很有效地阻止垃圾邮件或病毒邮件对企业信箱的干扰。
>
> **对邮件加密：**可有效保护邮件内容不被黑客盗取和篡改。
>
> **添加邮件监控审核模块：**可有效地监控企业全体员工的邮件中是否有敏感词、是否有透露企业资料等违规行为。
>
> **保障稳定性：**电子邮件系统的稳定性至关重要，运维人员应做到保证电子邮件系统的稳定运行，并及时做好防范分布式拒绝服务（Distributed Denial of Service，DDoS）攻击的准备。

**15.2 部署基础的电子邮件系统**

一个最基础的电子邮件系统肯定要能提供发件服务和收件服务，为此需要使用基于SMTP协议的Postfix服务程序提供发件服务功能，并使用基于POP3协议的Dovecot服务程序提供收件服务功能。这样一来，用户就可以使用Outlook Express或Foxmail等客户端服务程序正常收发邮件了。电子邮件系统的工作流程如图15-3所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2015/08/%E9%82%AE%E5%B1%80%E7%B3%BB%E7%BB%9F%E6%B5%81%E7%A8%8B%E5%9B%BE.png)

图15-3 电子邮件系统的工作流程

在诸多早期的Linux系统中，默认使用的发件服务是由Sendmail服务程序提供的，而在RHEL 8系统中已经替换为Postfix服务程序。相较于Sendmail服务程序，Postfix服务程序减少了很多不必要的配置步骤，而且在稳定性、并发性方面也有很大改进。

一般而言，我们的信箱地址类似于“root@linuxprobe.com”这样，也就是按照“用户名@主机地址（域名）”格式来规范的。如果您给我一串“root@192.168.10.10”的信息，我可能猜不到这是一个邮箱地址，没准会将它当作SSH协议的连接信息。因此，要想更好地检验电子邮件系统的配置效果，需要先部署bind服务程序，为电子邮件服务器和客户端提供DNS域名解析服务。

**第1步**：配置服务器主机名称，需要保证服务器主机名称与发信域名保持一致：

```
[root@linuxprobe ~]# vim /etc/hostname
mail.linuxprobe.com
[root@linuxprobe ~]# hostname
mail.linuxprobe.com
```

### **Tips**

修改主机名称文件后如果没有立即生效，可以重启下服务器。或者再执行一条“hostnamectl set-hostname mail.linuxprobe.com”[命令](https://www.linuxcool.com/)，让主机名称立即被设置。

**第2步**：清空iptables防火墙默认策略，并保存策略状态，避免因防火墙中默认存在的策略阻止了客户端DNS解析域名及收发邮件：

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables-save
```

还要记得firewalld防火墙，把dns协议加入到允许列表中：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=dns
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**第3步**：为电子邮件系统提供域名解析。由于[第13章](https://www.linuxprobe.com/basic-learning-13.html)已经讲解了bind-chroot服务程序的配置方法，因此这里只提供主配置文件、区域配置文件和域名数据文件的配置内容，其余配置步骤请大家自行完成。

```
 [root@linuxprobe ~]# dnf install bind-chroot
 [root@linuxprobe ~]# cat /etc/named.conf
  1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9 
 10 options {
 11         listen-on port 53 { any; };
 12         listen-on-v6 port 53 { ::1; };
 13         directory       "/var/named";
 14         dump-file       "/var/named/data/cache_dump.db";
 15         statistics-file "/var/named/data/named_stats.txt";
 16         memstatistics-file "/var/named/data/named_mem_stats.txt";
 17         secroots-file   "/var/named/data/named.secroots";
 18         recursing-file  "/var/named/data/named.recursing";
 19         allow-query     { any; };
 20 
 ………………省略部分输出信息………………
[root@linuxprobe ~]# cat /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
        type master;
        file "linuxprobe.com.zone";
        allow-update {none;};
};
```

建议在复制正向解析模板文件时，对cp[命令](https://www.linuxcool.com/)追加-a参数，能够让新文件继承原文件的属性和权限信息：

```
[root@linuxprobe ~]# cp -a /var/named/named.localhost /var/named/linuxprobe.com.zone
[root@linuxprobe ~]# cat /var/named/linuxprobe.com.zone
```

| $TTL 1D |          |                      |                      |             |
| ------- | -------- | -------------------- | -------------------- | ----------- |
| @       | IN SOA   | linuxprobe.com.      | root.linuxprobe.com. | (           |
|         |          |                      |                      | 0;serial    |
|         |          |                      |                      | 1D;refresh  |
|         |          |                      |                      | 1H;retry    |
|         |          |                      |                      | 1W;expire   |
|         |          |                      |                      | 3H);minimum |
|         | NS       | ns.linuxprobe.com.   |                      |             |
| ns      | IN A     | 192.168.10.10        |                      |             |
| @       | IN MX 10 | mail.linuxprobe.com. |                      |             |
| mail    | IN A     | 192.168.10.10        |                      |             |



```
[root@linuxprobe ~]# systemctl restart named
[root@linuxprobe ~]# systemctl enable  named
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /usr/lib/systemd/system/named.service.
```

修改好配置文件后记得重启bind服务程序，这样电子邮件系统所对应的服务器主机名即为mail.linuxprobe.com，而邮件域为@linuxprobe.com。把服务器的DNS地址修改成本地IP地址，如图15-4所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E9%85%8D%E7%BD%AE%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%9A%84DNS%E5%9C%B0%E5%9D%80.png)

图15-4 配置服务器的DNS地址

让新配置的网卡参数立即生效：

```
[root@linuxprobe ~]# nmcli connection up ens160 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
```

最后，能够用ping命令获得主机名所对应的IP地址，证明上述操作全部正确：

```
[root@linuxprobe ~]# ping -c 4  mail.linuxprobe.com 
PING mail.linuxprobe.com (192.168.10.10) 56(84) bytes of data.
64 bytes from mail.linuxprobe.com (192.168.10.10): icmp_seq=1 ttl=64 time=0.040 ms
64 bytes from mail.linuxprobe.com (192.168.10.10): icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from mail.linuxprobe.com (192.168.10.10): icmp_seq=3 ttl=64 time=0.037 ms
64 bytes from mail.linuxprobe.com (192.168.10.10): icmp_seq=4 ttl=64 time=0.052 ms

--- mail.linuxprobe.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 45ms
rtt min/avg/max/mdev = 0.037/0.046/0.057/0.010 ms
```

###### **15.2.1 配置Postfix服务程序**

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/postfix.gif)

Postfix是一款由IBM资助研发的免费开源电子邮件服务程序，能够很好地兼容Sendmail服务程序，可以方便Sendmail用户迁移到Postfix服务上。Postfix服务程序的邮件收发能力强于Sendmail服务，而且能自动增加、减少进程的数量来保证电子邮件系统的高性能与稳定性。另外，Postfix服务程序由许多小模块组成，每个小模块都可以完成特定的功能，因此可在生产工作环境中根据需求灵活搭配它们。

**第1步**：安装Postfix服务程序。

```
[root@linuxprobe ~]# dnf install postfix
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:10:38 ago on Mon 29 Mar 2021 06:40:32 AM CST.
Dependencies resolved.
================================================================================
 Package             Arch          Version             Repository    Size
================================================================================
Installing:
 postfix             x86_64        2:3.3.1-8.el8       BaseOS        1.5 M

Transaction Summary
================================================================================
Install  1 Package
………………省略部分输出信息………………
Installed:
  postfix-2:3.3.1-8.el8.x86_64                                                                                     

Complete!
```

**第2步**：配置Postfix服务程序。大家如果是首次看到Postfix服务程序主配置文件（/etc/postfix/main.cf），估计会被738行的内容给吓到。其实不用担心，这里面绝大多数的内容依然是注释信息。[刘遄](https://www.linuxprobe.com/)老师在本书中一直强调正确学习Linux系统的方法，并坚信“负责任的好老师不应该是书本的搬运工，而应该一名优质内容的提炼者”，因此在翻遍了配置参数的介绍，以及结合多年的运维经验后，最终总结出了7个最应该掌握的参数，如表15-1所示。

表15-1                Postfix服务程序主配置文件中的重要参数

| 参数            | 作用                     |
| --------------- | ------------------------ |
| myhostname      | 邮局系统的主机名         |
| mydomain        | 邮局系统的域名           |
| myorigin        | 从本机发出邮件的域名名称 |
| inet_interfaces | 监听的网卡接口           |
| mydestination   | 可接收邮件的主机名或域名 |
| mynetworks      | 设置可转发哪些主机的邮件 |
| relay_domains   | 设置可转发哪些网域的邮件 |



在Postfix服务程序的主配置文件中，总计需要修改5处。首先是在第95行定义一个名为myhostname的变量，用来保存服务器的主机名称。请大家记住这个变量的名称，下边的参数需要调用它：

```
[root@linuxprobe ~]# vim /etc/postfix/main.cf
 86 
 87 # INTERNET HOST AND DOMAIN NAMES
 88 # 
 89 # The myhostname parameter specifies the internet hostname of this
 90 # mail system. The default is to use the fully-qualified domain name
 91 # from gethostname(). $myhostname is used as a default value for many
 92 # other configuration parameters.
 93 #
 94 #myhostname = host.domain.tld
 95 myhostname = mail.linuxprobe.com
 96 
```

然后在第102行定义一个名为mydomain的变量，用来保存邮件域的名称。大家也要记住这个变量名称，下面将调用它：

```
 96 
 97 # The mydomain parameter specifies the local internet domain name.
 98 # The default is to use $myhostname minus the first component.
 99 # $mydomain is used as a default value for many other configuration
100 # parameters.
101 #
102 mydomain = linuxprobe.com
103 
```

在第118行调用前面的mydomain变量，用来定义发出邮件的域。调用变量的好处是避免重复写入信息，以及便于日后统一修改：

```
105 # 
106 # The myorigin parameter specifies the domain that locally-posted
107 # mail appears to come from. The default is to append $myhostname,
108 # which is fine for small sites.  If you run a domain with multiple
109 # machines, you should (1) change this to $mydomain and (2) set up
110 # a domain-wide alias database that aliases each user to
111 # user@that.users.mailhost.
112 #
113 # For the sake of consistency between sender and recipient addresses,
114 # myorigin also specifies the default domain name that is appended
115 # to recipient addresses that have no @domain part.
116 #
117 #myorigin = $myhostname
118 myorigin = $mydomain
119 
```

第四处修改是在第135行定义网卡监听地址。可以指定要使用服务器的哪些IP地址对外提供电子邮件服务；也可以干脆写成all，代表所有IP地址都能提供电子邮件服务：

```
121 
122 # The inet_interfaces parameter specifies the network interface
123 # addresses that this mail system receives mail on.  By default,
124 # the software claims all active interfaces on the machine. The
125 # parameter also controls delivery of mail to user@[ip.address].
126 #
127 # See also the proxy_interfaces parameter, for network addresses that
128 # are forwarded to us via a proxy or network address translator.
129 #
130 # Note: you need to stop/start Postfix when this parameter changes.
131 #
132 #inet_interfaces = all
133 #inet_interfaces = $myhostname
134 #inet_interfaces = $myhostname, localhost
135 inet_interfaces = all
136 
```

最后一处修改是在第183行定义可接收邮件的主机名或域名列表。这里可以直接调用前面定义好的myhostname和mydomain变量（如果不想调用变量，也可以直接调用变量中的值）：

```
151 
152 # The mydestination parameter specifies the list of domains that this
153 # machine considers itself the final destination for.
154 #
155 # These domains are routed to the delivery agent specified with the
156 # local_transport parameter setting. By default, that is the UNIX
157 # compatible delivery agent that lookups all recipients in /etc/passwd
158 # and /etc/aliases or their equivalent.
159 #
160 # The default is $myhostname + localhost.$mydomain + localhost.  On
161 # a mail domain gateway, you should also include $mydomain.
162 #
163 # Do not specify the names of virtual domains - those domains are
164 # specified elsewhere (see VIRTUAL_README).
165 #
166 # Do not specify the names of domains that this machine is backup MX
167 # host for. Specify those names via the relay_domains settings for
168 # the SMTP server, or use permit_mx_backup if you are lazy (see
169 # STANDARD_CONFIGURATION_README).
170 #
171 # The local machine is always the final destination for mail addressed
172 # to user@[the.net.work.address] of an interface that the mail system
173 # receives mail on (see the inet_interfaces parameter).
174 #
175 # Specify a list of host or domain names, /file/name or type:table
176 # patterns, separated by commas and/or whitespace. A /file/name
177 # pattern is replaced by its contents; a type:table is matched when
178 # a name matches a lookup key (the right-hand side is ignored).
179 # Continue long lines by starting the next line with whitespace.
180 #
181 # See also below, section "REJECTING MAIL FOR UNKNOWN LOCAL USERS".
182 #
183 mydestination = $myhostname, $mydomain
184 #mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
185 #mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,
186 #       mail.$mydomain, www.$mydomain, ftp.$mydomain
187 
```

**第3步**：创建电子邮件系统的登录账户。Postfix与vsftpd服务程序一样，都可以调用本地系统的账户和密码，因此在本地系统创建常规账户即可。最后重启配置妥当的postfix服务程序，并将其添加到开机启动项中。大功告成！

```
[root@linuxprobe ~]# useradd liuchuan
[root@linuxprobe ~]# echo "linuxprobe" | passwd --stdin liuchuan
Changing password for user liuchuan.
passwd: all authentication tokens updated successfully.
[root@linuxprobe ~]# systemctl restart postfix
[root@linuxprobe ~]# systemctl enable  postfix
Created symlink /etc/systemd/system/multi-user.target.wants/postfix.service → /usr/lib/systemd/system/postfix.service.
```

###### **15.2.2 配置Dovecot服务程序**

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/dovecot.png)

Dovecot是一款能够为Linux系统提供IMAP和POP3电子邮件服务的开源服务程序，安全性极高，配置简单，执行速度快，而且占用的服务器硬件资源也较少，因此是一款值得推荐的收件服务程序。

**第1步**：安装Dovecot服务程序软件包。

```
[root@linuxprobe ~]# dnf install -y dovecot
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:49:52 ago on Mon 29 Mar 2021 06:40:32 AM CST.
Dependencies resolved.
=============================================================================================
 Package                   Arch      Version                              Repository    Size
=============================================================================================
Installing:
 dovecot                   x86_64    1:2.2.36-5.el8                       AppStream     4.6 M
Installing dependencies:
 clucene-core              x86_64    2.3.3.4-31.20130812.e8e3d20git.el8   AppStream     590 k

Transaction Summary
===============================================================================================
Install  2 Packages
………………省略部分输出信息………………

Installed:
  dovecot-1:2.2.36-5.el8.x86_64                                                               
  clucene-core-2.3.3.4-31.20130812.e8e3d20git.el8.x86_64                                                              

Complete!
```

**第2步**：配置部署Dovecot服务程序。在Dovecot服务程序的主配置文件中进行如下修改。首先是第24行，把Dovecot服务程序支持的电子邮件协议修改为imap、pop3和lmtp。然后在这一行下面添加一行参数，允许用户使用明文进行密码验证。之所以这样操作，是因为Dovecot服务程序为了保证电子邮件系统的安全而默认强制用户使用加密方式进行登录，而由于当前还没有加密系统，因此需要添加该参数来允许用户的明文登录。

```
[root@linuxprobe ~]# vim /etc/dovecot/dovecot.conf
………………省略部分输出信息………………
 22 
 23 # Protocols we want to be serving.
 24 protocols = imap pop3 lmtp
 25 disable_plaintext_auth = no
 26 
………………省略部分输出信息………………
```

在主配置文件中的第49行，设置允许登录的网段地址，也就是说我们可以在这里限制只有来自于某个网段的用户才能使用电子邮件系统。如果想允许所有人都能使用，则不用修改本参数：

```
 44 
 45 # Space separated list of trusted network ranges. Connections from these
 46 # IPs are allowed to override their IP addresses and ports (for logging and
 47 # for authentication checks). disable_plaintext_auth is also ignored for
 48 # these networks. Typically you'd specify your IMAP proxy servers here.
 49 login_trusted_networks = 192.168.10.0/24
 50 
```

**第3步**：配置邮件格式与存储路径。在Dovecot服务程序单独的子配置文件中，定义一个路径，用于指定要将收到的邮件存放到服务器本地的哪个位置。这个路径默认已经定义好了，只需要将该配置文件中第25行前面的井号（#）删除即可。

```
[root@linuxprobe ~]# vim /etc/dovecot/conf.d/10-mail.conf
  1 ##
  2 ## Mailbox locations and namespaces
  3 ##
  4 
  5 # Location for users' mailboxes. The default is empty, which means that Dovecot
  6 # tries to find the mailboxes automatically. This won't work if the user
  7 # doesn't yet have any mail, so you should explicitly tell Dovecot the full
  8 # location.
  9 #
 10 # If you're using mbox, giving a path to the INBOX file (eg. /var/mail/%u)
 11 # isn't enough. You'll also need to tell Dovecot where the other mailboxes are
 12 # kept. This is called the "root mail directory", and it must be the first
 13 # path given in the mail_location setting.
 14 #
 15 # There are a few special variables you can use, eg.:
 16 #
 17 #   %u - username
 18 #   %n - user part in user@domain, same as %u if there's no domain
 19 #   %d - domain part in user@domain, empty if there's no domain
 20 #   %h - home directory
 21 #
 22 # See doc/wiki/Variables.txt for full list. Some examples:
 23 #
 24 #   mail_location = maildir:~/Maildir
 25     mail_location = mbox:~/mail:INBOX=/var/mail/%u
 26 #   mail_location = mbox:/var/mail/%d/%1n/%n:INDEX=/var/indexes/%d/%1n/%n
 27 #
………………省略部分输出信息………………
```

然后切换到配置Postfix服务程序时创建的boss账户，并在家目录中建立用于保存邮件的目录。记得要重启Dovecot服务并将其添加到开机启动项中。至此，对Dovecot服务程序的配置部署步骤全部结束。

```
[root@linuxprobe ~]# su - liuchuan
[liuchuan@linuxprobe ~]$ mkdir -p mail/.imap/INBOX
[liuchuan@linuxprobe ~]$ exit
logout
[root@linuxprobe ~]# systemctl restart dovecot
[root@linuxprobe ~]# systemctl enable  dovecot
Created symlink /etc/systemd/system/multi-user.target.wants/dovecot.service → /usr/lib/systemd/system/dovecot.service.
```

老读者肯定觉得少了点什么吧~对喽，还要记得把上面所提到的邮件协议在防火墙中的策略予以放行，这样客户端就能正常访问了：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=imap
success
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=pop3
success
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=smtp
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

###### **15.2.3 客户使用电子邮件系统**

如何得知电子邮件系统已经能够正常收发邮件了呢？可以使用Windows操作系统中自带的Outlook软件来进行测试（也可以使用其他电子邮件客户端来测试，比如Foxmail）。请按照表15-2来设置电子邮件系统及DNS服务器和客户端主机的IP地址，以便能正常解析邮件域名。设置后的结果如图15-5所示。

表15-2                  服务器与客户端的操作系统与IP地址



| 主机名称                | 操作系统   | IP地址        |
| ----------------------- | ---------- | ------------- |
| 电子邮件系统及DNS服务器 | RHEL 8     | 192.168.10.10 |
| 客户端主机              | Windows 10 | 192.168.10.30 |



![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E9%85%8D%E7%BD%AEWindows-10%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%BD%91%E7%BB%9C%E5%8F%82%E6%95%B0.png)

图15-5 配置Windows 10系统的网络参数

**第1步**：在Windows 10系统中运行Outlook软件程序。由于各位读者使用的Windows 10系统版本不一定相同，因此本书决定采用Outlook 2010版本为对象进行实验。如果您想要与这里的实验环境尽量保持一致，可在本书配套站点的软件资源库页面（https://www.linuxprobe.com/tools）下载并安装。在初次运行该软件时会出现一个“Microsoft Outlook 2010 启动”页面，引导大家完成该软件的配置过程，如图15-6所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/Outlook-2010%E5%90%AF%E5%8A%A8%E5%90%91%E5%AF%BC.png)

图15-6 Outlook 2010启动向导

**第2步**：配置电子邮件账户。在图15-7所示的“账户设置”页面中单击“是”单选按钮，然后单击“下一步”按钮。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2015/08/%E7%AC%AC2%E6%AD%A5%EF%BC%9A%E9%80%89%E6%8B%A9%E5%BC%80%E5%A7%8B%E9%85%8D%E7%BD%AE%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E5%B8%90%E6%88%B7.jpg)

图15-7 配置电子邮件账户

**第3步**：填写电子邮件账户信息，在图15-8所示的页面中，“您的姓名”文本框中可以为自定义的任意名字，“电子邮件地址”文本框中则需要输入服务器系统内的账户名外加发件域，“密码”文本框中要输入该账户在服务器内的登录密码。在填写完毕之后，单击“下一步”按钮。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%A1%AB%E5%86%99%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E8%B4%A6%E6%88%B7%E4%BF%A1%E6%81%AF.png)

图15-8 填写电子邮件账户信息

**第4步**：进行电子邮件服务登录验证。由于当前没有可用的SSL加密服务，因此在Dovecot服务程序的主配置文件中写入了一条参数，让客户可以使用明文登录到电子邮件服务。Outlook软件默认会通过SSL加密协议尝试登录电子邮件服务，所以在进行图15-9所示的“搜索liuchuan@linuxprobe.com服务器设置”大约30～60秒后，系统会出现登录失败的报错信息。此时只需再次单击“下一步”按钮，即可让Outlook软件通过非加密的方式验证登录，如图15-10所示。最后验证成功的界面如图15-11所示，点击“完成”按钮，一切搞定！

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%BF%9B%E8%A1%8C%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E9%AA%8C%E8%AF%81%E7%99%BB%E5%BD%95.png)

图15-9 进行电子邮件服务验证登录

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E4%BD%BF%E7%94%A8%E9%9D%9E%E5%8A%A0%E5%AF%86%E7%9A%84%E6%96%B9%E5%BC%8F%E8%BF%9B%E8%A1%8C%E7%94%B5%E5%AD%90%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E9%AA%8C%E8%AF%81%E7%99%BB%E5%BD%95.png)

图15-10 使用非加密的方式进行电子邮件服务验证登录

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E4%BD%BF%E7%94%A8%E9%9D%9E%E5%8A%A0%E5%AF%86%E6%96%B9%E5%BC%8F%E9%AA%8C%E8%AF%81%E6%88%90%E5%8A%9F.png)

图15-11 使用非加密方式配置账户成功

**第5步**：向其他信箱发送邮件。在成功登录Outlook软件后即可尝试编写并发送新邮件了。只需在软件界面的空白处单击鼠标右键，在弹出的菜单中点击“新建电子邮件”选项（见图15-12），然后在邮件界面中填写收件人的信箱地址以及完整的邮件内容后单击“发送”按钮，如图15-13所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%90%91%E5%85%B6%E4%BB%96%E4%BF%A1%E7%AE%B1%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6.png)

图15-12 向其他信箱发送邮件

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%A1%AB%E5%86%99%E6%94%B6%E4%BB%B6%E4%BA%BA%E4%BF%A1%E7%AE%B1%E5%9C%B0%E5%9D%80%E5%B9%B6%E7%BC%96%E5%86%99%E5%AE%8C%E6%95%B4%E7%9A%84%E9%82%AE%E4%BB%B6%E5%86%85%E5%AE%B9.png)

图15-13 填写收件人信箱地址并编写完整的邮件内容

当使用Outlook软件成功发送邮件后，便可以在电子邮件服务器上查看到新邮件提醒了，在RHEL 8系统中查看邮件的命令是mailx，需要自行安装（输出信息省略）。想查看邮件的完整内容，只需输入收件人姓名前面的编号即可。

```
[root@linuxprobe ~]# dnf install mailx
[root@linuxprobe ~]# mailx
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 1 message 1 new
>N  1 liuchuan              Tue Mar 30 01:35  97/3257  "Hello~"
& 1
Message  1:
From liuchuan@linuxprobe.com  Tue Mar 30 01:35:29 2021
Return-Path: <liuchuan@linuxprobe.com>
X-Original-To: root@linuxprobe.com
Delivered-To: root@linuxprobe.com
From: "liuchuan" <liuchuan@linuxprobe.com>
To: <root@linuxprobe.com>
Subject: Hello~
Date: Mon, 29 Mar 2021 19:49:30 +0800
Content-Type: multipart/alternative;
	boundary="----=_NextPart_000_0001_01D724D4.A28BB310"
X-Mailer: Microsoft Outlook 14.0
Thread-Index: AdckkVaUrscA9j2EQ3evqG++j6aSSA==
Content-Language: zh-cn
Status: R

Content-Type: text/plain;
	charset="gb2312"

当您收到这封邮件时，证明我的邮局系统实验已经成功！

& quit
Held 1 message in /var/spool/mail/root
[root@linuxprobe ~]#
```

##### **15.3 设置用户别名邮箱**

用户别名功能是一项简单实用的邮件账户伪装技术，可以用来设置多个虚拟信箱的账户以接受发送的邮件，从而保证自身的邮件地址不被泄露，还可以用来接收自己的多个信箱中的邮件。刚才我们已经顺利地向root账户送了邮件，下面再向bin账户发送一封邮件，如图15-14所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%90%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84bin%E8%B4%A6%E6%88%B7%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6.png)

图15-14 向服务器上的bin账户发送邮件

在邮件发送后登录到服务器，然后尝试以bin账户的身份登录。由于bin账户在Linux系统中是系统账户，默认的[Shell](https://www.linuxcool.com/)终端是/sbin/nologin，因此在以bin账户登录时，系统会提示当前账户不可用。但是，在电子邮件服务器上使用mail命令后，却看到这封原本要发送给bin账户的邮件已经被存放到了root账户的信箱中。

```
[root@linuxprobe ~]# su - bin
This account is currently not available.
[root@linuxprobe ~]# mailx
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 2 messages 1 new
    1 liuchuan              Tue Mar 30 01:35  98/3268  "Hello~"
>N  2 liuchuan              Tue Mar 30 03:53  97/3251  "你好，用户bin。"
& 2
Message  2:
From liuchuan@linuxprobe.com  Tue Mar 30 03:53:37 2021
Return-Path: <liuchuan@linuxprobe.com>
X-Original-To: bin@linuxprobe.com
Delivered-To: bin@linuxprobe.com
From: "liuchuan" <liuchuan@linuxprobe.com>
To: <bin@linuxprobe.com>
Subject: 你好，用户bin。
Date: Mon, 29 Mar 2021 22:07:39 +0800
Content-Type: multipart/alternative;
	boundary="----=_NextPart_000_000E_01D724E7.EEF35A10"
X-Mailer: Microsoft Outlook 14.0
Thread-Index: AdckpJ6n2QIfRYAZTB20gA9VTep2dg==
Content-Language: zh-cn
Status: R

Content-Type: text/plain;
	charset="gb2312"

这是一封发给用户bin的邮件。

& quit
Held 2 messages in /var/spool/mail/root
[root@linuxprobe ~]#
```

太奇怪了！明明发送给bin账户的邮件怎么会被root账户收到了呢？其实，这就是使用用户别名技术来实现的。在aliases邮件别名服务的配置文件中可以看到，里面定义了大量的用户别名，这些用户别名大多数是Linux系统本地的系统账户，而在冒号（:）间隔符后面的root账户则是用来接收这些账户邮件的人。用户别名可以是Linux系统内的本地用户，也可以是完全虚构的用户名字。

### **Tips**

下述命令会显示大量的内容，考虑到篇幅限制，这里已经做了部分删减，其实际的输出名单将是这里的两倍多。

```
[root@linuxprobe ~]# cat /etc/aliases
#
#  Aliases in this file will NOT be expanded in the header from
#  Mail, but WILL be visible over networks or from /bin/mail.
#
#	>>>>>>>>>>	The program "newaliases" must be run after
#	>> NOTE >>	this file is updated for any changes to
#	>>>>>>>>>>	show through to sendmail.
#

# Basic system aliases -- these MUST be present.
mailer-daemon:	postmaster
postmaster:	root

# General redirections for pseudo accounts.
bin:		root
daemon:		root
adm:		root
lp:		root
sync:		root
shutdown:	root
halt:		root
mail:		root
news:		root
uucp:		root
operator:	root
………………省略部分输出信息………………
```

现在大家能猜出是怎么一回事了吧。原来aliases邮件别名服务的配置文件是专门用来定义用户别名与邮件接收人的映射。除了使用本地系统中系统账户的名称外，我们还可以自行定义一些别名来接收邮件。例如，创建一个名为dream的账户，而真正接收该账户邮件的应该是root账户。

```
[root@linuxprobe ~]# cat /etc/aliases
#
#  Aliases in this file will NOT be expanded in the header from
#  Mail, but WILL be visible over networks or from /bin/mail.
#
#       >>>>>>>>>>      The program "newaliases" must be run after
#       >> NOTE >>      this file is updated for any changes to
#       >>>>>>>>>>      show through to sendmail.
#

# Basic system aliases -- these MUST be present.
mailer-daemon:  postmaster
postmaster:     root

# General redirections for pseudo accounts.
dream:          root
bin:            root
daemon:         root
adm:            root
lp:             root
sync:           root
shutdown:       root
halt:           root
mail:           root
news:           root
uucp:           root
operator:       root
………………省略部分输出信息………………
```

保存并退出aliases邮件别名服务的配置文件后，需要再执行一下newaliases命令，其目的是让新的用户别名配置文件立即生效。然后再次尝试发送邮件，如图15-15所示：

```
[root@linuxprobe ~]# newaliases
```

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%90%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84dream%E7%94%A8%E6%88%B7%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6.png)

图15-15 向服务器上的dream用户发送邮件

这时，使用root账户在服务器上执行mail命令后，就能看到这封原本要发送给dream账户的邮件了。最后，[刘遄](https://www.linuxprobe.com/)老师再啰嗦一句，用户别名技术不仅应用广泛，而且配置也很简单。所以更要提醒大家的是，今后千万不要看到有些网站上提供了很多客服信箱就轻易相信别人，没准发往这些客服信箱的邮件会被同一个人收到。

```
[root@linuxprobe ~]# mailx
Heirloom Mail version 12.5 7/5/10.  Type ? for help.
"/var/spool/mail/root": 3 messages 1 new
    1 liuchuan              Tue Mar 30 01:35  98/3268  "Hello~"
    2 liuchuan              Tue Mar 30 03:53  98/3262  "你好，用户bin。"
>N  3 liuchuan              Tue Mar 30 04:12  98/3317  "这是一封发送给dream用户的邮件"
& 3
Message  3:
From liuchuan@linuxprobe.com  Tue Mar 30 04:12:19 2021
Return-Path: <liuchuan@linuxprobe.com>
X-Original-To: dream@linuxprobe.com
Delivered-To: dream@linuxprobe.com
From: "liuchuan" <liuchuan@linuxprobe.com>
To: <dream@linuxprobe.com>
Subject: 这是一封发送给dream用户的邮件
Date: Mon, 29 Mar 2021 22:26:21 +0800
Content-Type: multipart/alternative;
	boundary="----=_NextPart_000_0009_01D724EA.8B9A4750"
X-Mailer: Microsoft Outlook 14.0
Thread-Index: Adckpw3r2QT7QwGITceHTJdfioQeQQ==
Content-Language: zh-cn
Status: R

Content-Type: text/plain;
	charset="gb2312"

顺利的话会被root用户接收到。

& quit
Held 3 messages in /var/spool/mail/root
```

##### **15.4 Linux邮件客户端**

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/Thunderbird-150x150.png)

对于我们大多数人而言，如今更多的是使用浏览器或智能手机的方式来收发电子邮件。但是，为了更快的加载邮件，或是为了更为丰富的编辑功能，有些时候还是要依赖专门的邮件客户端才能完成。Linux系统下的可选邮件客户端有数十种，例如Thunderbird、Evolution、Gear、Elementary Mail、KMail、Mailspring、Sylpheed、Claws Mail等等。

过去经常会有同学抱怨说，要是能在Linux系统下办公就太好了，完全可以把生产环境迁移到开源产品上，那么就趁着这次机会，为读者介绍一款刘遄老师正在使用的邮件客户端吧~

Thunderbird是一款由FireFox火狐浏览器的母公司Mozilla基金会发布的电子邮件客户端，兼具FireFox浏览器的各种优势，实现跨平台支持，拥有各种插件和丰富功能，简单的操作让用户更容易轻松地上手。

RHEL 8系统的光盘镜像中已经包含了Thunderbird客户端的安装包，配置好软件仓库后即可一键安装：

```
[root@linuxprobe ~]# dnf install -y thunderbird
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                           3.1 MB/s | 3.2 kB     00:00    
BaseOS                                              2.1 MB/s | 2.7 kB     00:00    
Dependencies resolved.
====================================================================================
 Package             Arch           Version                 Repository         Size
====================================================================================
Installing:
 thunderbird         x86_64         60.5.0-1.el8            AppStream          79 M

Transaction Summary
====================================================================================
Install  1 Package
………………省略部分输出信息………………
Installed:
  thunderbird-60.5.0-1.el8.x86_64                                                   

Complete!
```

对于这款图形化的客户端程序，我们有两种打开方式。其一是通过在终端中输入“thunderbird”命令后回车，或是通过在左上角的Activities程序菜单中点击客户端图标的方式，如图15-16所示。

```
[root@linuxprobe ~]# thunderbird
```

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/1-40-1024x614.png)图15-16 在程序菜单中点击邮件客户端图标

初次进入到客户端界面时，Thunderbird会要求用户填写邮件账户的名称、地址和密码，如图15-17所示。账号不一定要与系统中的相同，可以理解成是邮件发送人的昵称，密码则是系统中的密码，然后点击“Continue”继续按钮。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/2-32.png)

图15-17 配置客户端的账号、地址和密码

接下来点击“Manual config”手动配置按钮，如图15-18所示，对连接信息进行进一步配置。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/3-13.png)

图15-18 进入到手动配置模式

由于当前没有做SSL邮局加密，因此在如图15-19所示的手动配置模式中，需要将SSL选项更改为“None”，并设置验证为“Normal password”正常账号密码验证模式。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/4-14.png)

图15-19 手动配置连接信息

出于对安全的考虑，如图15-20所示，Thunderbird客户端会提示有警告信息，勾选中“understand the risks”明白此风险选项，然后点击“Done”确认按钮即可。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/5-9.png)

图15-20 安全警告信息

接下来便顺利来到了Thunderbird客户端的使用界面，如图15-21所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/6-7.png)

图15-21 Thunderbird邮件客户端使用界面

邮件客户端的使用方法与平时操作软件大致相同，读者可以安装后多操作、多探索，把办公环境放到Linux系统上是完全可行的。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．电子邮件服务与HTTP、FTP、NFS等程序的服务模式的最大区别是什么？

**答：**当对方主机宕机或对方临时离线时，使用电子邮件服务依然可以发送数据。

2．常见的电子邮件协议有那些？

**答：**SMTP、POP3和IMAP4。

3．电子邮件系统中MUA、MTA、MDA三种服务角色的用途分别是什么？

**答：**MUA用于收发邮件、MTA用于转发邮件、MDA用于保存邮件。

4．使用Postfix与Dovecot部署电子邮件系统前，需要先做什么？

**答：**需要先配置部署DNS域名解析服务，以便提供信箱地址解析功能。

5．能否让Dovecot服务程序限制允许连接的主机范围？

**答：**可以，在Dovecot服务程序的主配置文件中修改login_trusted_networks参数值即可，这样可在不修改防火墙策略的情况下限制来访的主机范围。

6．使用Outlook软件连接电子邮件服务器的地址mail.linuxprobe.com时，提示找不到服务器或连接超时，这可能是什么原因导致的呢？

**答：**很有可能是DNS域名解析问题引起的连接超时，可在服务器与客户端分别执行ping mail.linuxprobe.com 命令，测试是否可以正常解析出IP地址。

7．如何定义用户别名信箱以及让其立即生效？

**答：**可直接修改邮件别名服务的配置文件，并在保存退出后执行newaliases命令即可让新的用户别名立即生效。