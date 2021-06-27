# [第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/basic-learning-08.html)

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
[root@linuxprobe ~]# dnf install cockpit
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                       3.1 MB/s | 3.2 kB     00:00    
BaseOS                                          2.7 MB/s | 2.7 kB     00:00    
Package cockpit-185-2.el8.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

Cockpit服务程序在RHEL 8.0版本中没有自动运行，将它开启并加入到开机启动项中：

```
[root@linuxprobe ~]# systemctl start cockpit
[root@linuxprobe ~]# systemctl enable cockpit.socket
Created symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/systemd/system/cockpit.socket.
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

