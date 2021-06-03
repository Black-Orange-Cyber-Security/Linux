# [第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/basic-learning-09.html)

**章节简述：** 

本章讲解了如何使用nmtui[命令](https://www.linuxcool.com/)配置网卡参数，以及通过nmcli[命令](https://www.linuxcool.com/)查看网络信息并管理网络会话服务，从而让读者能够在不同工作场景中快速地切换网络运行参数；还讲解了如何手工绑定round-robin轮询模式双网卡，实现网络的负载均衡。

本章深入介绍了SSH协议与sshd服务程序的理论知识、[Linux系统](https://www.linuxprobe.com/)的远程管理方法以及在系统中配置服务程序的方法，并采用实验的形式演示了使用基于密码与密钥验证的sshd服务程序进行远程访问，以及使用登录tmux服务程序远程管理[Linux](https://www.linuxprobe.com/)系统的不间断会话等技术。细致讲解日志系统的理论知识，使用journalctl命令基于各种条件进行日志信息的检索，快速定位工作中的故障点。

当读者掌握了本章的内容之后，也就完全具备了对Linux系统进行配置管理的知识。而且后续章节中将陆续引入大量实用服务的配置内容，读者将用到本章学习的知识进行配置，这样一方面能够让读者对生产环境中用到的大多数热门服务程序有一个广泛且深入的认识，另一方面也可以掌握相应的配置方法。

本章目录结构

- 9.1 配置网卡服务
  - [9.1.1 配置网卡参数](https://www.linuxprobe.com/basic-learning-09.html#911)
  - [9.1.2 创建网络会话](https://www.linuxprobe.com/basic-learning-09.html#912)
  - [9.1.3 绑定两块网卡](https://www.linuxprobe.com/basic-learning-09.html#913)
- 9.2 远程控制服务
  - [9.2.1 配置sshd服务](https://www.linuxprobe.com/basic-learning-09.html#921_sshd)
  - [9.2.2 安全密钥验证](https://www.linuxprobe.com/basic-learning-09.html#922)
  - [9.2.3 远程传输命令](https://www.linuxprobe.com/basic-learning-09.html#923)
- 9.3 不间断会话服务
  - [9.3.1 管理远程会话](https://www.linuxprobe.com/basic-learning-09.html#931)
  - [9.3.2 管理多窗格](https://www.linuxprobe.com/basic-learning-09.html#932)
  - [9.3.3 会话共享功能](https://www.linuxprobe.com/basic-learning-09.html#933)
- [9.4 检索日志信息](https://www.linuxprobe.com/basic-learning-09.html#94)

##### **9.1 配置网卡服务**

###### **9.1.1 配置网卡参数**

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

###### **9.1.2 创建网络会话**

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

###### **9.1.3 绑定两块网卡**

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

##### **9.2 远程控制服务**

###### **9.2.1 配置sshd服务**

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

###### **9.2.2 安全密钥验证**

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

###### **9.2.3 远程传输命令**

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

