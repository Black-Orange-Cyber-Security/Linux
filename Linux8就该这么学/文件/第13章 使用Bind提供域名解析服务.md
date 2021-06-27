# [第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/basic-learning-13.html)

**章节简述：** 

本章讲解了DNS域名解析服务的原理以及作用，介绍了域名查询功能中正向解析与反向解析的作用，并通过实验的方式演示了如何在DNS主服务器上部署正、反解析工作模式，以便让大家深刻体会到DNS域名查询的便利以及强大。

本章还介绍了如何部署DNS从服务器以及DNS缓存服务器来提升用户的域名查询体验，以及如何使用chroot牢笼机制插件来保障bind服务程序的可靠性，并向大家演示如何在主服务器与从服务器之间部署TSIG密钥加密功能，来进一步保障迭代查询中数据的安全性。最后，本章还从实战层面讲解了DNS分离解析技术，让来自不同国家、不同地区的用户都能获得最优的网站访问体验。

相信大家在学完本章内容之后，一定会对bind服务程序有更深入的了解和认识，并能深刻地体会到作为互联网基础设施中重要一环的DNS域名解析服务，在互联网中所承担的重要角色和发挥的重要作用。

本章目录结构

- [13.1 DNS域名解析服务](https://www.linuxprobe.com/basic-learning-13.html#131_DNS)
- 13.2 安装Bind服务程序
  - [13.2.1 正向解析实验](https://www.linuxprobe.com/basic-learning-13.html#1321)
  - [13.2.2 反向解析实验](https://www.linuxprobe.com/basic-learning-13.html#1322)
- [13.3 部署从服务器](https://www.linuxprobe.com/basic-learning-13.html#133)
- [13.4 安全的加密传输](https://www.linuxprobe.com/basic-learning-13.html#134)
- [13.5 部署缓存服务器](https://www.linuxprobe.com/basic-learning-13.html#135)
- [13.6 分离解析技术](https://www.linuxprobe.com/basic-learning-13.html#136)

##### **13.1 DNS域名解析服务**

相较于由数字构成的IP地址，域名更容易被理解和记忆，所以我们通常更习惯通过域名的方式来访问网络中的资源。但是，网络中的计算机之间只能基于IP地址来相互识别对方的身份，而且要想在互联网中传输数据，也必须基于外网的IP地址来完成。

为了降低用户访问网络资源的门槛，DNS（Domain Name System）域名系统技术应运而生。这是一项用于管理和解析域名与IP地址对应关系的技术，简单来说，就是能够接受用户输入的域名或IP地址，然后自动查找与之匹配（或者说具有映射关系）的IP地址或域名，即将域名解析为IP地址（正向解析），或将IP地址解析为域名（反向解析）。这样一来，只需要在浏览器中输入域名就能打开想要访问的网站了。DNS域名解析技术的正向解析也是最常使用的一种工作模式。

鉴于互联网中的域名和IP地址对应关系数据库太过庞大，DNS域名解析服务采用了类似目录树的层次结构来记录域名与IP地址之间的对应关系，从而形成了一个分布式的数据库系统，如图13-1所示。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2015/06/DNS%E7%BB%93%E6%9E%84%E6%A8%A1%E5%9E%8B.jpg)

图13-1 DNS域名解析服务采用的目录树层次结构

域名后缀一般分为国际域名和国内域名。原则上来讲，域名后缀都有严格的定义，但在实际使用时可以不必严格遵守。目前最常见的域名后缀有.com（商业组织）、.org（非营利组织）、.gov（政府部门）、.net（网络服务商）、.edu（教研机构）、.pub（公共大众）、.cn（中国国家顶级域名）等。

当今世界的信息化程度越来越高，大数据、云计算、物联网、人工智能等新技术不断涌现，全球网民的数量据说也超过了53亿，而且每年还在以7%的速度迅速增长。这些因素导致互联网中的域名数量进一步激增，被访问的频率也进一步加大。假设全球网民每人每天只访问一个网站域名，而且只访问一次，也会产生53亿次的查询请求，如此庞大的请求数量肯定无法被某一台服务器全部处理掉。DNS技术作为互联网基础设施中重要的一环，为了为网民提供不间断、稳定且快速的域名查询服务，保证互联网的正常运转，提供了下面三种类型的服务器。

> **主服务器**：在特定区域内具有唯一性，负责维护该区域内的域名与IP地址之间的对应关系。
>
> **从服务器**：从主服务器中获得域名与IP地址的对应关系并进行维护，以防主服务器宕机等情况。
>
> **缓存服务器**：通过向其他域名解析服务器查询获得域名与IP地址的对应关系，并将经常查询的域名信息保存到服务器本地，以此来提高重复查询时的效率。

简单来说，主服务器是用于管理域名和IP地址对应关系的真正服务器，从服务器帮助主服务器“打下手”，分散部署在各个国家、省市或地区，以便让用户就近查询域名，从而减轻主服务器的负载压力。缓存服务器不太常用，一般部署在企业内网的网关位置，用于加速用户的域名查询请求。

DNS域名解析服务采用分布式的数据结构来存放海量的“区域数据”信息，在执行用户发起的域名查询请求时，具有递归查询和迭代查询两种方式。所谓递归查询，是指DNS服务器在收到用户发起的请求时，必须向用户返回一个准确的查询结果。如果DNS服务器本地没有存储与之对应的信息，则该服务器需要询问其他服务器，并将返回的查询结果提交给用户。而迭代查询则是指，DNS服务器在收到用户发起的请求时，并不直接回复查询结果，而是告诉另一台DNS服务器的地址，用户再向这台DNS服务器提交请求，这样依次反复，直到返回查询结果。

由此可见，当用户向就近的一台DNS服务器发起对某个域名的查询请求之后（这里以[www.linuxprobe.com](https://www.linuxprobe.com/)为例），其查询流程大致如图13-2所示。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2015/06/DNS%E6%9F%A5%E8%AF%A2%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

图13-2 向DNS服务器发起域名查询请求的流程

当用户向网络指定的DNS服务器发起一个域名请求时，通常情况下会有本地由此DNS服务器向上级的DNS服务器发送迭代查询请求；如果该DNS服务器没有要查询的信息，则会进一步向上级DNS服务器发送迭代查询请求，直到获得准确的查询结果为止。其中最高级、最权威的根DNS服务器总共有13台，分布在世界各地，其管理单位、具体的地理位置，以及IP地址如表13-1所示。

表13-1                    13台根DNS服务器的具体信息

| 名称 | 管理单位               | 地理位置          | IP地址         |
| ---- | ---------------------- | ----------------- | -------------- |
| A    | INTERNIC.NET           | 美国-弗吉尼亚州   | 198.41.0.4     |
| B    | 美国信息科学研究所     | 美国-加利弗尼亚州 | 128.9.0.107    |
| C    | PSINet公司             | 美国-弗吉尼亚州   | 192.33.4.12    |
| D    | 马里兰大学             | 美国-马里兰州     | 128.8.10.90    |
| E    | 美国航空航天管理局     | 美国加利弗尼亚州  | 192.203.230.10 |
| F    | 因特网软件联盟         | 美国加利弗尼亚州  | 192.5.5.241    |
| G    | 美国国防部网络信息中心 | 美国弗吉尼亚州    | 192.112.36.4   |
| H    | 美国陆军研究所         | 美国-马里兰州     | 128.63.2.53    |
| I    | Autonomica公司         | 瑞典-斯德哥尔摩   | 192.36.148.17  |
| J    | VeriSign公司           | 美国-弗吉尼亚州   | 192.58.128.30  |
| K    | RIPE NCC               | 英国-伦敦         | 193.0.14.129   |
| L    | IANA                   | 美国-弗吉尼亚州   | 199.7.83.42    |
| M    | WIDE Project           | 日本-东京         | 202.12.27.33   |



### **Tips**

我们所提到的13台根域服务器并非真的是指只有13台服务器，没有那台服务器能独立承受住如此大的请求量，这是技术圈习惯的叫法而已。实际上用于根域名的服务器总共有504台，它们从A到M进行了排序，共用13个IP地址以此进行负载均衡，抵抗分布式拒绝服务攻击（DDoS）的影响。

随着互联网接入设备数量增长，原有 IPv4 体系已经不能满足需求，IPv6 协议在全球开始普及。基于 IPv6 的新型地址结构为新增根服务器提供了契机。下一代互联网国家工程中心于2013 年联合日本和美国相关运营机构和专业人士发起“雪人计划”，提出以 IPv6 为基础、面向新兴应用、自主可控的一整套根服务器解决方案和技术体系，在全球完成25台 IPv6(互联网协议第六版) 根服务器架设。

##### **13.2 安装Bind服务程序**

BIND（Berkeley Internet Name Domain，伯克利因特网名称域）服务是全球范围内使用最广泛、最安全可靠且高效的域名解析服务程序。DNS域名解析服务作为互联网基础设施服务，其责任之重可想而知，因此建议大家在生产环境中安装部署bind服务程序时加上chroot（俗称牢笼机制）扩展包，以便有效地限制bind服务程序仅能对自身的配置文件进行操作，以确保整个服务器的安全。

```
[root@linuxprobe ~]# yum install bind-chroot
Loaded plugins: langpacks, product-id, subscription-manager
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                               3.1 MB/s | 3.2 kB     00:00    
BaseOS                                                  2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
========================================================================================
 Package             Arch           Version                     Repository         Size
========================================================================================
Installing:
 bind-chroot         x86_64         32:9.11.4-16.P2.el8         AppStream          99 k
Installing dependencies:
 bind                x86_64         32:9.11.4-16.P2.el8         AppStream         2.1 M

Transaction Summary
========================================================================================
Install  2 Packages
………………省略部分输出信息………………
Installed:
  bind-chroot-32:9.11.4-16.P2.el8.x86_64                      bind-32:9.11.4-16.P2.el8.x86_64                     

Complete!
```

不难看出，作为主程序的bind有2.1M，而“安全插件”的bind-chroot仅有99K。

bind服务程序的配置并不简单，因为要想为用户提供健全的DNS查询服务，要在本地保存相关的域名数据库，而如果把所有域名和IP地址的对应关系都写入到某个配置文件中，估计要有上千万条的参数，这样既不利于程序的执行效率，也不方便日后的修改和维护。因此在bind服务程序中有下面这三个比较关键的文件。

> 主配置文件（/etc/named.conf）：只有59行，而且在去除注释信息和空行之后，实际有效的参数仅有30行左右，这些参数用来定义bind服务程序的运行。
>
> 区域配置文件（/etc/named.rfc1912.zones）：用来保存域名和IP地址对应关系的所在位置。类似于图书的目录，对应着每个域和相应IP地址所在的具体位置，当需要查看或修改时，可根据这个位置找到相关文件。
>
> 数据配置文件目录（/var/named）：该目录用来保存域名和IP地址真实对应关系的数据配置文件。

在[Linux系统](https://www.linuxprobe.com/)中，bind服务程序的名称为named。首先需要在/etc目录中找到该服务程序的主配置文件，然后把第11行和第19行的地址均修改为any，分别表示服务器上的所有IP地址均可提供DNS域名解析服务，以及允许所有人对本服务器发送DNS查询请求。这两个地方一定要修改准确。

```
[root@linuxprobe ~]# vim /etc/named.conf
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
 21         /* 
 22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable 
 24            recursion. 
 25          - If your recursive DNS server has a public IP address, you MUST enable access 
 26            control to limit queries to your legitimate users. Failing to do so will
 27            cause your server to become part of large scale DNS amplification 
 28            attacks. Implementing BCP38 within your network would greatly
 29            reduce such attack surface 
 30         */
 31         recursion yes;
 32 
 33         dnssec-enable yes;
 34         dnssec-validation yes;
 35 
 36         managed-keys-directory "/var/named/dynamic";
 37 
 38         pid-file "/run/named/named.pid";
 39         session-keyfile "/run/named/session.key";
 40 
 41         /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
 42         include "/etc/crypto-policies/back-ends/bind.config";
 43 };
 44 
 45 logging {
 46         channel default_debug {
 47                 file "data/named.run";
 48                 severity dynamic;
 49         };
 50 };
 51 
 52 zone "." IN {
 53         type hint;
 54         file "named.ca";
 55 };
 56 
 57 include "/etc/named.rfc1912.zones";
 58 include "/etc/named.root.key";
 59 
```

如前所述，bind服务程序的区域配置文件（/etc/named.rfc1912.zones）用来保存域名和IP地址对应关系的所在位置。在这个文件中，定义了域名与IP地址解析规则保存的文件位置以及服务类型等内容，而没有包含具体的域名、IP地址对应关系等信息。服务类型有三种，分别为hint（根区域）、master（主区域）、slave（辅助区域），其中常用的master和slave指的就是主服务器和从服务器。将域名解析为IP地址的正向解析参数和将IP地址解析为域名的反向解析参数分别如图13-3和图13-4所示。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2015/06/%E6%AD%A3%E5%90%91%E8%A7%A3%E6%9E%90%E5%8C%BA%E5%9F%9F%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F.png)

图13-3 正向解析参数

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2015/06/%E5%8F%8D%E5%90%91%E8%A7%A3%E6%9E%90%E5%8C%BA%E5%9F%9F%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F-1.png)

图13-4 反向解析参数

下面的实验中会分别修改bind服务程序的主配置文件、区域配置文件与数据配置文件。如果在实验中遇到了bind服务程序启动失败的情况，而您认为这是由于参数写错而导致的，则可以执行named-checkconf[命令](https://www.linuxcool.com/)和named-checkzone[命令](https://www.linuxcool.com/)，分别检查主配置文件与数据配置文件中语法或参数的错误。

###### **13.2.1 正向解析实验**

在DNS域名解析服务中，正向解析是指根据域名（主机名）查找到对应的IP地址。也就是说，当用户输入了一个域名后，bind服务程序会自动进行查找，并将匹配到的IP地址返给用户。如图13-5所示，这也是最常用的DNS工作模式。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%AD%A3%E5%90%91%E8%A7%A3%E6%9E%90.jpg)

图13-5 正向解析技术示意图

**第1步**：编辑区域配置文件。该文件中默认已经有了一些无关紧要的解析参数，旨在让用户有一个参考。我们可以将下面的参数添加到区域配置文件的最下面，当然，也可以将该文件中的原有信息全部清空，而只保留自己的域名解析信息：

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
        type master;
        file "linuxprobe.com.zone";
        allow-update {none;};
};
```

### **Tips**

配置文件中的代码缩进仅是为了提升阅读体验，有无缩进对参数效果均没有任何影响，不排版也没问题~

**第2步**：编辑数据配置文件。可以从/var/named目录中复制一份正向解析的模板文件（named.localhost），然后把域名和IP地址的对应数据填写数据配置文件中并保存。在复制时记得加上-a参数，这可以保留原始文件的所有者、所属组、权限属性等信息，以便让bind服务程序顺利读取文件内容：

```
[root@linuxprobe ~]# cd /var/named/
[root@linuxprobe named]# ls -al named.localhost
-rw-r-----. 1 root named 152 Jun 21 2007 named.localhost
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.zone
```

编辑数据配置文件。在保存并退出后文件后记得重启named服务程序，让新的解析数据生效。考虑到正向解析文件中的参数较多，而且相对都比较重要，[刘遄](https://www.linuxprobe.com/)老师在每个参数后面都作了简要的说明。

```
[root@linuxprobe named]# vim linuxprobe.com.zone
[root@linuxprobe named]# systemctl restart named
[root@linuxprobe named]# systemctl enable named
Created symlink /etc/systemd/system/multi-user.target.wants/named.service → /usr/lib/systemd/system/named.service.
```

| $TTL 1D | #生存周期为1天 |                    |                                |              |                         |
| ------- | -------------- | ------------------ | ------------------------------ | ------------ | ----------------------- |
| @       | IN SOA         | linuxprobe.com.    | root.linuxprobe.com.           | (            |                         |
|         | #授权信息开始: | #DNS区域的地址     | #域名管理员的邮箱(不要用@符号) |              |                         |
|         |                |                    |                                | 0;serial     | #更新序列号             |
|         |                |                    |                                | 1D;refresh   | #更新时间               |
|         |                |                    |                                | 1H;retry     | #重试延时               |
|         |                |                    |                                | 1W;expire    | #失效时间               |
|         |                |                    |                                | 3H );minimum | #无效解析记录的缓存时间 |
|         | NS             | ns.linuxprobe.com. | #域名服务器记录                |              |                         |
| ns      | IN A           | 192.168.10.10      | #地址记录(ns.linuxprobe.com.)  |              |                         |
| www     | IN A           | 192.168.10.10      | #地址记录(www.linuxprobe.com.) |              |                         |



在解析文件中，除了A记录类型代表将域名指向一个IPV4地址，而AAAA则代表将域名指向一个IPV6地址，此外还有8种记录类型，一并为读者整理如下表13-2所示，供日后备查：

表13-2           域名解析记录类型

| 记录类型 | 作用                                             |
| -------- | ------------------------------------------------ |
| A        | 将域名指向一个IPV4地址                           |
| CNAME    | 将域名指向另外一个域名                           |
| AAAA     | 将域名指向一个IPV6地址                           |
| NS       | 将子域名指定其他DNS服务器解析                    |
| MX       | 将域名指向邮件服务器地址                         |
| SRV      | 记录提供特定的服务的服务器                       |
| TXT      | 文本内容一般为512字节，常作为反垃圾邮件的SPF记录 |
| CAA      | CA证书办法机构授权校验                           |
| 显性URL  | 将域名重定向到另外一个地址                       |
| 隐性URL  | 与显性URL类型，但是会隐藏真实目标地址            |



**第3步**：检验解析结果。为了检验解析结果，一定要先把Linux系统网卡中的DNS地址参数修改成本机IP地址，如图13-6所示，这样就可以使用由本机提供的DNS查询服务了。nslookup命令用于检测能否从DNS服务器中查询到域名与IP地址的解析记录，进而更准确地检验DNS服务器是否已经能够为用户提供服务。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E9%85%8D%E7%BD%AE%E7%BD%91%E5%8D%A1DNS%E4%BF%A1%E6%81%AF.png)

图13-6 配置网卡DNS参数信息

```
[root@linuxprobe named]# nmcli connection up ens160 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[root@linuxprobe named]# nslookup
Name:	www.linuxprobe.com
Address: 192.168.10.10
> ns.linuxprobe.com
Server:		192.168.10.10
Address:	192.168.10.10#53

Name:	ns.linuxprobe.com
Address: 192.168.10.10
```

若解析出的结果不是192.168.10.10，很大概率是虚拟机选择了联网模式，由互联网DNS服务器进行了解析。此时应确认服务器信息是否为“Address: 192.168.10.10#53”，即由本地服务器192.168.10.10的53端口号进行解析，若不是，则重启网卡后再试一下。

###### **13.2.2 反向解析实验**

在DNS域名解析服务中，反向解析的作用是将用户提交的IP地址解析为对应的域名信息，它一般用于对某个IP地址上绑定的所有域名进行整体屏蔽，屏蔽由某些域名发送的垃圾邮件。它也可以针对某个IP地址进行反向解析，大致判断出有多少个网站运行在上面。当购买虚拟主机时，可以使用这一功能验证虚拟主机提供商是否有严重的超售问题。如图13-6所示，对IP地址所关联的域名信息进行反推。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%8F%8D%E5%90%91%E8%A7%A3%E6%9E%90.jpg)

图13-6 反向解析技术示意图

**第1步**：编辑区域配置文件。在编辑该文件时，除了不要写错格式之外，还需要记住此处定义的数据配置文件名称，因为一会儿还需要在/var/named目录中建立与其对应的同名文件。反向解析是把IP地址解析成域名格式，因此在定义zone（区域）时应该要把IP地址反写，比如原来是192.168.10.0，反写后应该就是10.168.192，而且只需写出IP地址的网络位即可。把下列参数添加至正向解析参数的后面。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
        type master;
        file "linuxprobe.com.zone";
        allow-update {none;};
};
zone "10.168.192.in-addr.arpa" IN {
        type master;
        file "192.168.10.arpa";
        allow-update {none;};
};
```

**第2步**：编辑数据配置文件。首先从/var/named目录中复制一份反向解析的模板文件（named.loopback），然后把下面的参数填写到文件中。其中，IP地址仅需要写主机位，如图13-7所示。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2015/06/%E5%8F%8D%E5%90%91%E5%8C%BA%E5%9F%9F%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E4%B8%8D%E9%9C%80%E8%A6%81%E5%86%99%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%BB%9C%E9%83%A8%E5%88%86.png)

图13-7 反向解析文件中IP地址参数规范

```
[root@linuxprobe ~]# cd /var/named
[root@linuxprobe named]# cp -a named.loopback 192.168.10.arpa
[root@linuxprobe named]# vim 192.168.10.arpa
[root@linuxprobe named]# systemctl restart named
```

| $TTL 1D |        |                     |                                    |             |
| ------- | ------ | ------------------- | ---------------------------------- | ----------- |
| @       | IN SOA | linuxprobe.com.     | root.linuxprobe.com.               | (           |
|         |        |                     |                                    | 0;serial    |
|         |        |                     |                                    | 1D;refresh  |
|         |        |                     |                                    | 1H;retry    |
|         |        |                     |                                    | 1W;expire   |
|         |        |                     |                                    | 3H);minimum |
|         | NS     | ns.linuxprobe.com.  |                                    |             |
| ns      | A      | 192.168.10.10       |                                    |             |
| 10      | PTR    | ns.linuxprobe.com.  | #PTR为指针记录，仅用于反向解析中。 |             |
| 10      | PTR    | www.linuxprobe.com. |                                    |             |
| 20      | PTR    | bbs.linuxprobe.com. |                                    |             |



**第3步**：检验解析结果。在前面的正向解析实验中，已经把系统网卡中的DNS地址参数修改成了本机IP地址，因此可以直接使用nslookup命令来检验解析结果，仅需输入IP地址即可查询到对应的域名信息。

```
[root@linuxprobe ~]# nslookup
> 192.168.10.10
10.10.168.192.in-addr.arpa	name = www.linuxprobe.com.
> 192.168.10.20
20.10.168.192.in-addr.arpa	name = bbs.linuxprobe.com.
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **13.3 部署从服务器**

作为重要的互联网基础设施服务，保证DNS域名解析服务的正常运转至关重要，只有这样才能提供稳定、快速且不间断的域名查询服务。在DNS域名解析服务中，从服务器可以从主服务器上获取指定的区域数据文件，从而起到备份解析记录与负载均衡的作用，因此通过部署从服务器可以减轻主服务器的负载压力，还可以提升用户的查询效率。

在本实验中，主服务器与从服务器分别使用的操作系统和IP地址如表13-2所示。

表13-2           主服务器与从服务器分别使用的操作系统与IP地址信息

| 主机名称 | 操作系统 | IP地址        |
| -------- | -------- | ------------- |
| 主服务器 | RHEL 8   | 192.168.10.10 |
| 从服务器 | RHEL 8   | 192.168.10.20 |



**第1步**：在主服务器的区域配置文件中允许该从服务器的更新请求，即修改allow-update {允许更新区域信息的主机地址;};参数，然后重启主服务器的DNS服务程序。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
        type master;
        file "linuxprobe.com.zone";
        allow-update { 192.168.10.20;};
};
zone "10.168.192.in-addr.arpa" IN {
        type master;
        file "192.168.10.arpa";
        allow-update { 192.168.10.20;};
};
[root@linuxprobe ~]# systemctl restart named
```

**第2步**：在主服务器上配置防火墙放行规则，让DNS协议流量可以被顺利传递：

```
[root@linuxprobe ~]# iptables -F 
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=dns
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**第3步**：在从服务器上安装bind-chroot软件包（输出信息省略），修改配置文件让从服务器也能够对外提供DNS服务，并且测试与主服务器的网络连通性：

```
[root@linuxprobe ~]# dnf install bind-chroot
[root@linuxprobe ~]# vim /etc/named.conf
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
………………省略部分输出信息………………
[root@linuxprobe ~]# ping -c 4 192.168.10.10 
PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=2.44 ms
64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=3.31 ms
64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.503 ms
64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.359 ms

--- 192.168.10.10 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 15ms
rtt min/avg/max/mdev = 0.359/1.654/3.311/1.262 ms
```

**第4步**：在从服务器中填写主服务器的IP地址与要抓取的区域信息，然后重启服务。注意此时的服务类型应该是slave（从），而不再是master（主）。masters参数后面应该为主服务器的IP地址，而且file参数后面定义的是同步数据配置文件后要保存到的位置，稍后可以在该目录内看到同步的文件。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
        type slave;
        masters { 192.168.10.10; };
        file "slaves/linuxprobe.com.zone";
};
zone "10.168.192.in-addr.arpa" IN {
        type slave;
        masters { 192.168.10.10; };
        file "slaves/192.168.10.arpa";
};
[root@linuxprobe ~]# systemctl restart named
```

### **Tips**

这里的masters参数比正常的主服务类型master多了个字母s，代表主服务器可以有多个的意思，请小心，不要漏掉呦~

**第5步**：检验解析结果。当从服务器的DNS服务程序在重启后，一般就已经自动从主服务器上同步了数据配置文件，而且该文件默认会放置在区域配置文件中所定义的目录位置中。随后修改从服务器的网络参数，把DNS地址参数修改成192.168.10.20，这样即可使用从服务器自身提供的DNS域名解析服务。最后就可以使用nslookup命令顺利看到解析结果了。

```
[root@linuxprobe ~]# cd /var/named/slaves
[root@linuxprobe slaves]# ls 
192.168.10.arpa linuxprobe.com.zone
[root@linuxprobe slaves]# nslookup
> www.linuxprobe.com
Server:		192.168.10.20
Address:	192.168.10.20#53

Name:	www.linuxprobe.com
Address: 192.168.10.10
> 192.168.10.10
10.10.168.192.in-addr.arpa	name = www.linuxprobe.com.
```

如果读者的解析地址跟上述不一致，很可能是从服务器的网卡DNS地址没有指向到本机，顺手修改下就搞定啦！~

##### **13.4 安全的加密传输**

前文反复提及，域名解析服务是互联网基础设施中重要的一环，几乎所有的网络应用都依赖于DNS才能正常运行。如果DNS服务发生故障，那么即便Web网站或电子邮件系统服务等都正常运行，用户也无法找到并使用它们了。

互联网中的绝大多数DNS服务器（超过95%）都是基于BIND域名解析服务搭建的，而bind服务程序为了提供安全的解析服务，已经对TSIG RFC 2845加密机制提供了支持。TSIG主要是利用了密码编码的方式来保护区域信息的传输（Zone Transfer），即TSIG加密机制保证了DNS服务器之间传输域名区域信息的安全性。

接下来的实验依然使用了表13-2中的两台服务器。

书接上回。前面在从服务器上配妥bind服务程序并重启后，即可看到从主服务器中获取到的数据配置文件。

| 主机名称 | 操作系统 | IP地址        |
| -------- | -------- | ------------- |
| 主服务器 | RHEL 8   | 192.168.10.10 |
| 从服务器 | RHEL 8   | 192.168.10.20 |



```
[root@linuxprobe ~]# ls -al /var/named/slaves/
total 8
drwxrwx---. 2 named named  56 Mar 12 09:53 .
drwxrwx--T. 6 root  named 141 Mar 12 09:57 ..
-rw-r--r--. 1 named named 436 Mar 12 09:53 192.168.10.arpa
-rw-r--r--. 1 named named 282 Mar 12 09:53 linuxprobe.com.zone
[root@linuxprobe ~]# rm -rf /var/named/slaves/*
```

**第1步**：在主服务器中生成密钥。dnssec-keygen命令用于生成安全的DNS服务密钥，其格式为“dnssec-keygen [参数]”，常用的参数以及作用如表13-3所示。

表13-3                    dnssec-keygen命令的常用参数

| 参数 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| -a   | 指定加密算法，包括RSAMD5（RSA）、RSASHA1、DSA、NSEC3RSASHA1、NSEC3DSA等 |
| -b   | 密钥长度（HMAC-MD5的密钥长度在1~512位之间）                  |
| -n   | 密钥的类型（HOST表示与主机相关）                             |



使用下述命令生成一个主机名称为master-slave的128位HMAC-MD5算法的密钥文件。在执行该命令后默认会在当前目录中生成公钥和私钥文件，我们需要把私钥文件中Key参数后面的值记录下来，一会儿要将其写入传输配置文件中。

```
[root@linuxprobe ~]# dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slave
Kmaster-slave.+157+62533
[root@linuxprobe ~]# ls -l Kmaster-slave.+157+62533.*
-rw-------. 1 root root  56 Mar 14 09:54 Kmaster-slave.+157+62533.key
-rw-------. 1 root root 165 Mar 14 09:54 Kmaster-slave.+157+62533.private
[root@linuxprobe ~]# cat Kmaster-slave.+157+62533.private 
Private-key-format: v1.3
Algorithm: 157 (HMAC_MD5)
Key: NI6icnb74FxHx2gK+0MVOg==
Bits: AAA=
Created: 20210314015436
Publish: 20210314015436
Activate: 20210314015436
```

**第2步**：在主服务器中创建密钥验证文件。进入bind服务程序用于保存配置文件的目录，把刚刚生成的密钥名称、加密算法和私钥加密字符串按照下面格式写入到tansfer.key传输配置文件中。为了安全起见，需要将文件的所属组修改成named，并将文件权限设置得要小一点，然后把该文件做一个硬链接到/etc目录中。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc/
[root@linuxprobe etc]# vim transfer.key
key "master-slave" {
        algorithm hmac-md5;
        secret "NI6icnb74FxHx2gK+0MVOg==";
};
[root@linuxprobe etc]# chown root:named transfer.key
[root@linuxprobe etc]# chmod 640 transfer.key
[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第3步**：开启并加载Bind服务的密钥验证功能。首先需要在主服务器的主配置文件中加载密钥验证文件，然后进行设置，使得只允许带有master-slave密钥认证的DNS服务器同步数据配置文件：

```
[root@linuxprobe ~]# vim /etc/named.conf
  1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9 include "/etc/transfer.key";
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
 20         allow-transfer { key master-slave; };
………………省略部分输出信息………………
[root@linuxprobe ~]# systemctl restart named
```

至此，DNS主服务器的TSIG密钥加密传输功能就已经配置完成。此时清空DNS从服务器同步目录中所有的数据配置文件，然后再次重启bind服务程序，这时就已经不能像刚才那样自动获取到数据配置文件了。

```
[root@linuxprobe ~]# rm -rf /var/named/slaves/*
[root@linuxprobe ~]# systemctl restart named
[root@linuxprobe ~]# ls  /var/named/slaves/
```

**第4步**：配置从服务器，使其支持密钥验证。配置DNS从服务器和主服务器的方法大致相同，都需要在bind服务程序的配置文件目录中创建密钥认证文件，并设置相应的权限，然后把该文件做一个硬链接到/etc目录中。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc/
[root@linuxprobe etc]# vim transfer.key
key "master-slave" {
        algorithm hmac-md5;
        secret "NI6icnb74FxHx2gK+0MVOg==";
};
[root@linuxprobe etc]# chown root:named transfer.key
[root@linuxprobe etc]# chmod 640 transfer.key
[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第5步**：开启并加载从服务器的密钥验证功能。这一步的操作步骤也同样是在主配置文件中加载密钥认证文件，然后按照指定格式写上主服务器的IP地址和密钥名称。注意，密钥名称等参数位置不要太靠前，大约在第51行比较合适，否则bind服务程序会因为没有加载完预设参数而报错：

```
[root@linuxprobe etc]# vim /etc/named.conf
  1 //
  2 // named.conf
  3 //
  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS
  5 // server as a caching only nameserver (as a localhost DNS resolver only).
  6 //
  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.
  8 //
  9 include "/etc/transfer.key";
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
 21         /* 
 22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion.
 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable 
 24            recursion. 
 25          - If your recursive DNS server has a public IP address, you MUST enable access 
 26            control to limit queries to your legitimate users. Failing to do so will
 27            cause your server to become part of large scale DNS amplification 
 28            attacks. Implementing BCP38 within your network would greatly
 29            reduce such attack surface 
 30         */
 31         recursion yes;
 32 
 33         dnssec-enable yes;
 34         dnssec-validation yes;
 35 
 36         managed-keys-directory "/var/named/dynamic";
 37 
 38         pid-file "/run/named/named.pid";
 39         session-keyfile "/run/named/session.key";
 40 
 41         /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */
 42         include "/etc/crypto-policies/back-ends/bind.config";
 43 };
 44 
 45 logging {
 46         channel default_debug {
 47                 file "data/named.run";
 48                 severity dynamic;
 49         };
 50 };
 51 server 192.168.10.10
 52 {
 53         keys { master-slave; };
 54 };
 55 zone "." IN {
 56         type hint;
 57         file "named.ca";
 58 };
 59 
 60 include "/etc/named.rfc1912.zones";
 61 include "/etc/named.root.key";
 62 
```

**第6步**：DNS从服务器同步域名区域数据。现在，两台服务器的bind服务程序都已经配置妥当，并匹配到了相同的密钥认证文件。接下来在从服务器上重启bind服务程序，可以发现又能顺利地同步到数据配置文件了。

```
[root@linuxprobe ~]# systemctl restart named
[root@linuxprobe ~]# ls /var/named/slaves/
192.168.10.arpa  linuxprobe.com.zone
```

**第7步**：再次进行解析验证，功能正常，注意看是由192.168.10.20从服务器进行解析的呦：

```
[root@linuxprobe etc]# nslookup www.linuxprobe.com
Server:		192.168.10.20
Address:	192.168.10.20#53

Name:	www.linuxprobe.com
Address: 192.168.10.10

[root@linuxprobe etc]# nslookup 192.168.10.10
10.10.168.192.in-addr.arpa	name = www.linuxprobe.com.
```

##### **13.5 部署缓存服务器**

DNS缓存服务器（Caching DNS Server）是一种不负责域名数据维护的DNS服务器。简单来说，缓存服务器就是把用户经常使用到的域名与IP地址的解析记录保存在主机本地，从而提升下次解析的效率。DNS缓存服务器一般用于经常访问某些固定站点而且对这些网站的访问速度有较高要求的企业内网中，但实际的应用并不广泛。而且，缓存服务器是否可以成功解析还与指定的上级DNS服务器的允许策略有关，因此当前仅需了解即可。

**第1步**：配置系统的双网卡参数。前面讲到，缓存服务器一般用于企业内网，旨在降低内网用户查询DNS的时间消耗。因此，为了更加贴近真实的网络环境，实现外网查询功能，我们需要在缓存服务器中再添加一块网卡，并按照表13-4所示的信息来配置出两台Linux虚拟机系统。图13-8展示了缓存服务器实验环境的结构拓扑，客户端不仅限于一台。

表13-4                用于配置Linux虚拟机系统所需的参数信息



| 主机名称   | 操作系统 | IP地址                                                       |
| ---------- | -------- | ------------------------------------------------------------ |
| 缓存服务器 | RHEL 8   | 网卡（外网）：根据物理设备的网络参数进行配置（通过DHCP或手动方式指定IP地址与网关等信息） |
|            |          | 网卡（内网）：192.168.10.10                                  |
| 客户端     | RHEL 8   | 192.168.10.20                                                |



![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E7%BC%93%E5%AD%98%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%AE%9E%E9%AA%8C%E7%8E%AF%E5%A2%83%E6%8B%93%E6%89%91-1.jpg)

图13-8 缓存服务器实验环境拓扑

**第2****步**：还需要在虚拟机软件中将新添加的网卡设置为“桥接模式”，如图13-9所示。然后设置成与物理设备相同的网络参数（此处需要大家按照物理设备真实的网络参数来配置，图13-10所示为以DHCP方式获取IP地址与网关等信息，重启网络服务后的效果如图13-11所示）。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%96%B0%E6%B7%BB%E5%8A%A0%E4%B8%80%E5%9D%97%E6%A1%A5%E6%8E%A5%E7%BD%91%E5%8D%A1.png)

图13-9 新添加一块桥接网卡

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E4%BB%A5DHCP%E6%96%B9%E5%BC%8F%E8%8E%B7%E5%8F%96%E7%BD%91%E7%BB%9C%E5%8F%82%E6%95%B0.png)

图13-10 以DHCP方式获取网络参数

### **Tips**

新添加的网卡设备默认没有配置文件，需要自行输入网卡名称和类型即可，另外记得让新的网卡参数生效下：

```
[root@linuxprobe ~]# nmcli connection up ens192
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
```

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%9F%A5%E7%9C%8B%E7%BD%91%E5%8D%A1%E7%9A%84%E5%B7%A5%E4%BD%9C%E7%8A%B6%E6%80%81.png)

图13-11 查看网卡的工作状态

**第3步**：在bind服务程序的主配置文件中添加缓存转发参数。在大约第20行处添加一行参数“forwarders { 上级DNS服务器地址; };”，上级DNS服务器地址指的是获取数据配置文件的服务器。考虑到查询速度、稳定性、安全性等因素，[刘遄](https://www.linuxprobe.com/)老师在这里使用的是北京市公共DNS服务器的地址210.73.64.1。如果大家也使用该地址，请先测试是否可以ping通，以免导致DNS域名解析失败。

```
[root@linuxprobe ~]# vim /etc/named.conf
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
 20         forwarders { 210.73.64.1; };
………………省略部分输出信息………………
[root@linuxprobe ~]# systemctl restart named
```

对了，如果您也还原了虚拟机系统到最初始的状态，记得把防火墙的放行规则一并完成：

```
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# iptables-save
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=dns
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

**第4步**：重启DNS服务，验证成果。把客户端主机的DNS服务器地址参数修改为DNS缓存服务器的IP地址192.168.10.10，如图13-12所示。这样即可让客户端使用本地DNS缓存服务器提供的域名查询解析服务。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BE%E7%BD%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%BB%E6%9C%BA%E7%9A%84DNS%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9C%B0%E5%9D%80%E5%8F%82%E6%95%B0.png)

图13-12 设置客户端主机的DNS服务器地址参数

客户端主机的网络参数设置妥当后重启网络服务，即可使用nslookup命令来验证实验结果（如果解析失败，请读者留意是否是上级DNS服务器选择的问题）。其中，Server参数为域名解析记录提供的服务器地址，因此可见是由本地DNS缓存服务器提供的解析内容。

```
[root@linuxprobe ~]# nmcli connection up ens160 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[root@linuxprobe ~]# nslookup
> www.linuxprobe.com
Server:		192.168.10.10
Address:	192.168.10.10#53

Non-authoritative answer:
www.linuxprobe.com	canonical name = www.linuxprobe.com.w.kunlunno.com.
Name:	www.linuxprobe.com.w.kunlunno.com
Address: 139.215.131.226
```

最后与读者们分享下实验心得，这个缓存DNS服务的配置参数只有一行参数，因此不存在写错的可能性，但如果出错了大概率有两个可能性。其一，上述210.73.64.1服务器可能停用，可以改为8.8.8.8或114.114.114.114再重新尝试。其二，检查nslookup输出结果中服务器地址是否正确，有可能是本地网卡参数没有生效而导致的。

##### **13.6 分离解析技术**

现在，喜欢看这本《Linux就该这么学》的海外读者越来越多，如果继续把本书配套的网站服务器（https://www.linuxprobe.com）架设在北京市的机房内，则海外读者的访问速度势必会很慢。可如果把服务器架设在美国那边的机房，也将增大国内读者的访问难度。

为了满足海内外读者的需求，于是可以购买多台服务器并分别部署在全球各地，然后再使用DNS服务的分离解析功能，即可让位于不同地理范围内的读者通过访问相同的网址，而从不同的服务器获取到相同的数据。例如，我们可以按照表13-5所示，分别为处于北京的DNS服务器和处于美国的DNS服务器分配不同的IP地址，然后让国内读者在访问时自动匹配到北京的服务器，而让海外读者自动匹配到美国的服务器，如图13-13所示。

表13-5                   不同主机的操作系统与IP地址情况

| 主机名称  | 操作系统   | IP地址                  |
| --------- | ---------- | ----------------------- |
| DNS服务器 | RHEL 8     | 北京网络：122.71.115.10 |
|           |            | 美国网络：106.185.25.10 |
| 北京用户  | Windows 10 | 122.71.115.1            |
| 海外用户  | Windows 10 | 106.185.25.1            |



![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/DNS%E5%88%86%E7%A6%BB%E8%A7%A3%E6%9E%90%E6%8A%80%E6%9C%AF.png)

图13-13 DNS分离解析技术

为了解决海外读者访问https://www.linuxprobe.com时的速度问题，刘遄老师已经在美国机房购买并架设好了相应的网站服务器，接下来需要手动部署DNS服务器并实现分离解析功能，以便让不同地理区域的读者在访问相同的域名时，能解析出不同的IP地址。

> 建议读者将虚拟机还原到初始状态，并重新安装bind服务程序，以免多个实验之间相互产生冲突。

**第1步**：修改bind服务程序的主配置文件，把第11行的监听端口与第19行的允许查询主机修改为any。由于配置的DNS分离解析功能与DNS根服务器配置参数有冲突，所以需要把第52~55行的根域信息删除。

```
[root@linuxprobe ~]# vim /etc/named.conf
………………省略部分输出信息………………
 44 
 45 logging {
 46         channel default_debug {
 47                 file "data/named.run";
 48                 severity dynamic;
 49         };
 50 };
 51 
 52 zone "." IN {
 53         type hint;
 54         file "named.ca";
 55 };
 56 
 57 include "/etc/named.rfc1912.zones";
 58 include "/etc/named.root.key";
 59 
………………省略部分输出信息………………
```

**第2步**：编辑区域配置文件。把区域配置文件中原有的数据清空，然后按照以下格式写入参数。首先使用acl参数分别定义两个变量名称（china与america），当下面需要匹配IP地址时只需写入变量名称即可，这样不仅容易阅读识别，而且也利于修改维护。这里的难点是理解view参数的作用。它的作用是通过判断用户的IP地址是中国的还是美国的，然后去分别加载不同的数据配置文件（linuxprobe.com.china或linuxprobe.com.america）。这样，当把相应的IP地址分别写入到数据配置文件后，即可实现DNS的分离解析功能。这样一来，当中国的用户访问linuxprobe.com域名时，便会按照linuxprobe.com.china数据配置文件内的IP地址找到对应的服务器。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zones
acl "china" { 122.71.115.0/24; };
acl "america" { 106.185.25.0/24; };
view "china"{
        match-clients { "china"; };
        zone "linuxprobe.com" {
        type master;
        file "linuxprobe.com.china";
        };
};
view "america" {
        match-clients { "america"; };
        zone "linuxprobe.com" {
        type master;
        file "linuxprobe.com.america";
        };
};    
```

**第3步**：建立数据配置文件。分别通过模板文件创建出两份不同名称的区域数据文件，其名称应与上面区域配置文件中相对应。

```
[root@linuxprobe ~]# cd /var/named
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.china
[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.america
[root@linuxprobe named]# vim linuxprobe.com.china
```

| $TTL 1D | #生存周期为1天 |                    |                                |             |                         |
| ------- | -------------- | ------------------ | ------------------------------ | ----------- | ----------------------- |
| @       | IN SOA         | linuxprobe.com.    | root.linuxprobe.com.           | (           |                         |
|         | #授权信息开始: | #DNS区域的地址     | #域名管理员的邮箱(不要用@符号) |             |                         |
|         |                |                    |                                | 0;serial    | #更新序列号             |
|         |                |                    |                                | 1D;refresh  | #更新时间               |
|         |                |                    |                                | 1H;retry    | #重试延时               |
|         |                |                    |                                | 1W;expire   | #失效时间               |
|         |                |                    |                                | 3H);minimum | #无效解析记录的缓存时间 |
|         | NS             | ns.linuxprobe.com. | #域名服务器记录                |             |                         |
| ns      | IN A           | 122.71.115.10      | #地址记录(ns.linuxprobe.com.)  |             |                         |
| www     | IN A           | 122.71.115.15      | #地址记录(www.linuxprobe.com.) |             |                         |



```
[root@linuxprobe named]# vim linuxprobe.com.america
```

| $TTL 1D | #生存周期为1天 |                    |                                |             |                         |
| ------- | -------------- | ------------------ | ------------------------------ | ----------- | ----------------------- |
| @       | IN SOA         | linuxprobe.com.    | root.linuxprobe.com.           | (           |                         |
|         | #授权信息开始: | #DNS区域的地址     | #域名管理员的邮箱(不要用@符号) |             |                         |
|         |                |                    |                                | 0;serial    | #更新序列号             |
|         |                |                    |                                | 1D;refresh  | #更新时间               |
|         |                |                    |                                | 1H;retry    | #重试延时               |
|         |                |                    |                                | 1W;expire   | #失效时间               |
|         |                |                    |                                | 3H);minimum | #无效解析记录的缓存时间 |
|         | NS             | ns.linuxprobe.com. | #域名服务器记录                |             |                         |
| ns      | IN A           | 106.185.25.10      | #地址记录(ns.linuxprobe.com.)  |             |                         |
| www     | IN A           | 106.185.25.15      | #地址记录(www.linuxprobe.com.) |             |                         |



其中122.71.115.15和106.185.25.15两台主机并没有在实验环节中配置，需要同学们自行准备，如果不想太过于麻烦，可以直接将www.linuxprobe.com域名解析到122.71.115.10和106.185.25.10服务器上面，这样只需要准备一台服务器就够了。

**第4步**：重新启动named服务程序，验证结果。将客户端主机（Windows系统或Linux系统均可）的IP地址分别设置为122.71.115.1与106.185.25.1，将DNS地址分别设置为服务器主机的两个IP地址。这样，当尝试使用nslookup命令解析域名时就能清晰地看到解析结果，分别如图13-14与图13-15所示。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%A8%A1%E6%8B%9F%E4%B8%AD%E5%9B%BD%E7%94%A8%E6%88%B7%E7%9A%84%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90%E6%93%8D%E4%BD%9C.png)

图13-14  模拟中国用户的域名解析操作

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%A8%A1%E6%8B%9F%E7%BE%8E%E5%9B%BD%E7%94%A8%E6%88%B7%E7%9A%84%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90%E6%93%8D%E4%BD%9C.png)

图13-15 模拟美国用户的域名解析

恭喜！又学完了一个新章节，休息一下继续学习吧~

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．DNS技术提供的三种类型的服务器分别是什么？

**答：**DNS主服务器、DNS从服务器与DNS缓存服务器。

2．DNS服务器之间传输区域数据文件时，使用的是递归查询还是迭代查询？

**答：**DNS服务器之间是迭代查询，用户与DNS服务器之间是递归查询。

3．在Linux系统中使用Bind服务程序部署DNS服务时，为什么推荐安装chroot插件？

**答：**能有效地限制Bind服务程序仅能对自身的配置文件进行操作，以确保整个服务器的安全。

4．在DNS服务中，正向解析和反向解析的作用是什么？

**答：**正向解析是将指定的域名转换为IP地址，而反向解析则是将IP地址转换为域名。正向解析模式更为常用。

5．是否可以限制使用DNS域名解析服务的主机？如何限制？

**答：**是的，修改主配置文件中第17行的allow-query参数即可。

6．部署DNS从服务器的作用是什么？

**答：**部署从服务器可以减轻主服务器的负载压力，还可以提升用户的查询效率。

7．当用户与DNS服务器之间传输数据配置文件时，是否可以使用TSIG加密机制来确保文件内容不被篡改？

**答：**不能，TSIG加密机制保障的是DNS服务器与DNS服务器之间迭代查询的安全。

8．部署DNS缓存服务器的作用是什么？

**答：**DNS缓存服务器把用户经常使用到的域名与IP地址的解析记录保存在主机本地，从而提升下次解析的效率。一般用于经常访问某些固定站点而且对这些网站的访问速度有较高要求的企业内网中，但实际的应用并不广泛。

9． DNS分离解析技术的作用是什么？

**答：**可以让位于不同地理范围内的读者通过访问相同的网址，而从不同的服务器获取到相同的数据，以提升访问效率。