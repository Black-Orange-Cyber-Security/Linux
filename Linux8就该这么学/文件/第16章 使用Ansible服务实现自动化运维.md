# [第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/basic-learning-16.html)

**章节简述：**

作为近年最火的开源运维自动化工具，正确使用Ansible服务能够帮助运维人员肉眼可见的提高工作效率，并减少人为失误。上千款功能丰富的模块不仅实用，而且有详尽的帮助信息可供查阅，因此即便是小白用户也可以轻松上手。

在本章节中，将学习部署Ansible服务、了解相关术语及配置Inventory主机清单。深入学习如ping、yum、firewalld、service、template、setup、lvol、lvg、copy、file、debug等十余个常用模块，满足工作中一切日常操作。完整实践从系统中加载、从外部环境中获取及自行创建角色的方法，帮助读者在学习中建立对生产环境流程控制的能力。

不限于此，[刘遄](https://www.linuxprobe.com/)老师还将以创建LVM逻辑卷设备、依据主机改写文件、管理文件属性等为目地的实践内容，精心编写Playbook剧本文件让繁琐的事情变容易，让重复的工作批量自动完成。最后，以Ansible Vault对变量及剧本文件加密实验结束全部课程，环环相扣，过瘾！

本章目录结构

- [16.1 Ansible介绍与安装](https://www.linuxprobe.com/basic-learning-16.html#161_Ansible)
- [16.2 设置主机清单](https://www.linuxprobe.com/basic-learning-16.html#162)
- [16.3 运行临时命令](https://www.linuxprobe.com/basic-learning-16.html#163)
- [16.4 剧本文件实战](https://www.linuxprobe.com/basic-learning-16.html#164)
- 16.5 创建及使用角色
  - [16.5.1 加载系统内置角色](https://www.linuxprobe.com/basic-learning-16.html#1651)
  - [16.5.2 从外部获取角色](https://www.linuxprobe.com/basic-learning-16.html#1652)
  - [16.5.3 创建新的角色](https://www.linuxprobe.com/basic-learning-16.html#1653)
- [16.6 创建和使用逻辑卷](https://www.linuxprobe.com/basic-learning-16.html#166)
- [16.7 判断主机组名](https://www.linuxprobe.com/basic-learning-16.html#167)
- [16.8 管理文件属性](https://www.linuxprobe.com/basic-learning-16.html#168)
- [16.9 管理密码库文件](https://www.linuxprobe.com/basic-learning-16.html#169)

##### **16.1 Ansible介绍与安装**

Ansible是一款开源的资源管理工具，是目前运维自动化工具中最简单、容易上手的优秀软件。用户可以通过它自动化的部署应用程序来实现IT基础架构，例如对服务器进行初始化配置、安全基线配置、更新和打补丁都是简单容易的。虽然相比于Chef、Puppet、Saltstack等CS架构的自动化工具来讲，Ansible的执行性能并不是最高的，但是由于其基于的是SSH远程会话协议，无需客户端程序，只要知道受管主机的账号密码，就能直接用SSH协议进行远程控制，因此使用起来优势很明显。

Ansible工具最初是在2012年3月9日由程序员Michael DeHaan发行的第一个版本，他本人对于配置管理和架构设计方面有着丰富的经验，此前在[红帽](https://www.linuxprobe.com/)公司供职时，就研发了Cobble自动化系统安装工具，这期间内也是被各种自动化软件折磨了好几年，最终才决定自己打造一款结合众多软件优点于一身的自动化工具，Ansible便由此诞生。由于这款工具实在太好用了，以至于在Github上的Star和Fork数量甚至超过了SaltStack的两倍还多，足以看出受欢迎的程度，2015年正式被[红帽](https://www.linuxprobe.com/)公司收购，其未来发展潜力更是不可估量。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/Ansible-300x169.png)

使用自动化运维工具，可以肉眼可见的提高运维工程师的工作效率，并减少人为错误。Ansible服务本身并没有批量部署的功能，它仅仅是一个框架，真正具有批量部署能力的是其所运行的模块。随服务的安装自带有上千个模块，通过调用指定的模块，就能实现特定的功能，所以只要掌握了常用模块的作用，即使是小白也能轻松上手。Ansible内置的模块已经非常丰富，几乎可以满足一切需求，管理模式也非常简单，一条[命令](https://www.linuxcool.com/)影响上千台主机，但如果需要更高级的功能，也可以运用Python语言对其再进行二次开发，比其他软件要容易很多。

当前，Ansible已经被AWS、Google Cloud Platform、Microsoft Azure、Cisco、HP、VMware、Twitter等大科技公司接纳并投入使用。红帽公司更是支持自家产品，从2020年8月1日起，[RHCE](https://www.linuxprobe.com/)考试内容由配置多款服务，转变成了Ansible专项考题内容，也就是原先[RHCA](https://www.linuxprobe.com/)培训中DO407的课程内容，现在要想顺利拿到[RHCE](https://www.linuxprobe.com/)红帽认证运维工程师的认证，这节课真要好好学习下了~

有些专用术语需要提前先跟读者们讲解下，建立对术语的统一理解，便于后续实验直奔主题~收集整理好的术语如表16-1所示：

表16-1                   Ansible服务专用术语对照表

| 术语          | 中文叫法 | 含义                                                         |
| ------------- | -------- | ------------------------------------------------------------ |
| Control node  | 控制节点 | 指的是安装了Ansible服务的主机，也被称为Ansible控制端，主要是用来发布运行任务、调用功能模块，对其他主机进行批量控制。 |
| Managed nodes | 受控节点 | 指的是被Ansible服务所管理的主机，也被称为受控主机或客户端，是模块[命令](https://www.linuxcool.com/)的被执行对象。 |
| Inventory     | 主机清单 | 指的是受控节点的列表，可以是IP地址、主机名称或者域名。       |
| Modules       | 模块     | 指的是上文提到的特定功能代码，默认自带有上千款功能模块，在Ansible Galaxy有超多可供选择。 |
| Task          | 任务     | 指的是Ansible客户端上面要被执行的操作。                      |
| Playbook      | 剧本     | 指的是通过YAML语言编写的可重复执行的任务列表，把常做的操作写入到剧本文件中，下次可以直接重复执行一遍。 |
| Roles         | 角色     | 从Ansible 1.2版本开始引入的新特性，用于结构化的组织Playbook，通过调用角色实现一连串的功能。 |



被Ansible服务所管理的主机是不需要安装客户端的，因为SSH协议是Linux系统的标配，因此直接上手控制就行。服务器上面也不用每次都重复开启服务程序，而是可以用ansible命令直接调用模块进行控制。

Ansible服务程序默认不在RHEL 8系统的镜像文件中，而是需要从“Extra Packages for Enterprise Linux”扩展软件包仓库获取，简称为EPEL安装源。EPEL软件包仓库是由红帽公司提供的，用于创建、维护和管理企业版Linux的一个高质量软件扩展仓库，通用于RHEL、CentOS、Oracle Linux等多种“红帽系”企业版系统，目的是对于默认系统仓库软件包进行扩展。

**第1步：**将虚拟机的网络适配器→网络连接选项调整为桥接模式，并设置系统的网卡成DHCP自动获取模式，如图16-1及16-2所示。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BE%E7%BD%AE%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%BD%91%E7%BB%9C%E8%BF%9E%E6%8E%A5%E4%B8%BA%E6%A1%A5%E6%8E%A5%E6%A8%A1%E5%BC%8F.png)

图16-1 设置虚拟机网络连接为桥接模式

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%8D%A1%E4%B8%BADHCP%E6%A8%A1%E5%BC%8F.png)

图16-2 设置网卡参数为DHCP模式

大多数情况下只要把虚拟机设置成桥接模式，且Linux系统的网卡信息保持跟物理机相同，再重启网卡服务，那么就可以连接外部网络了，不放心的话再通过ping命令进行测试：

```
[root@linuxprobe ~]# nmcli connection up ens160 
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
[root@linuxprobe ~]# ping -c 4 www.linuxprobe.com
PING www.linuxprobe.com.w.kunlunno.com (124.95.157.160) 56(84) bytes of data.
64 bytes from www.linuxprobe.com (124.95.157.160): icmp_seq=1 ttl=53 time=17.1 ms
64 bytes from www.linuxprobe.com (124.95.157.160): icmp_seq=2 ttl=53 time=15.6 ms
64 bytes from www.linuxprobe.com (124.95.157.160): icmp_seq=3 ttl=53 time=16.8 ms
64 bytes from www.linuxprobe.com (124.95.157.160): icmp_seq=4 ttl=53 time=17.5 ms

--- www.linuxprobe.com.w.kunlunno.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 10ms
rtt min/avg/max/mdev = 15.598/16.732/17.452/0.708 ms
```

**第2步：**在原有软件仓库配置的下方，继续追加EPEL扩展软件包安装源的信息：

```
[root@linuxprobe ~]# vim /etc/yum.repos.d/rhel.repo
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

[EPEL]
name=EPEL
baseurl=https://dl.fedoraproject.org/pub/epel/8/Everything/x86_64/
enabled=1
gpgcheck=0
```

**第3步：**安装！~

```
[root@linuxprobe ~]# dnf install -y ansible
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:01:31 ago on Sun 04 Apr 2021 02:23:32 AM CST.
Dependencies resolved.
===========================================================================================
 Package                          Arch             Version            Repository        Size
===========================================================================================
Installing:
 ansible                          noarch           2.9.18-2.el8       EPEL               17 M
Installing dependencies:
 python3-babel                    noarch           2.5.1-3.el8        AppStream         4.8 M
 python3-jinja2                   noarch           2.10-9.el8         AppStream         537 k
 python3-jmespath                 noarch           0.9.0-11.el8       AppStream          45 k
 python3-markupsafe               x86_64           0.23-19.el8        AppStream          39 k
 python3-pyasn1                   noarch           0.3.7-6.el8        AppStream         126 k
 libsodium                        x86_64           1.0.18-2.el8       EPEL              162 k
 python3-bcrypt                   x86_64           3.1.6-2.el8.1      EPEL               44 k
 python3-pynacl                   x86_64           1.3.0-5.el8        EPEL              100 k
 sshpass                          x86_64           1.06-9.el8         EPEL               27 k
Installing weak dependencies:
 python3-paramiko                 noarch           2.4.3-1.el8        EPEL              289 k

Transaction Summary
===========================================================================================
Install  11 Packages

………………省略部分输出信息…………

Installed:
  ansible-2.9.18-2.el8.noarch            python3-paramiko-2.4.3-1.el8.noarch
  python3-babel-2.5.1-3.el8.noarch       python3-jinja2-2.10-9.el8.noarch
  python3-jmespath-0.9.0-11.el8.noarch   python3-markupsafe-0.23-19.el8.x86_64
  python3-pyasn1-0.3.7-6.el8.noarch      libsodium-1.0.18-2.el8.x86_64                  
  python3-bcrypt-3.1.6-2.el8.1.x86_64    python3-pynacl-1.3.0-5.el8.x86_64
  sshpass-1.06-9.el8.x86_64                         

Complete!
```

安装完毕后，Ansible服务便默认已经启动，使用“--version”参数可以看到服务的版本及配置信息：

```
[root@linuxprobe ~]# ansible --version
ansible 2.9.18
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/root/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3.6/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.6.8 (default, Jan 11 2019, 02:17:16) [GCC 8.2.1 20180905 (Red Hat 8.2.1-3)]
```

##### **16.2 设置主机清单**

初次使用Ansible服务的同学可能会遇到明明参数已经修改了，但是却不生效的情况。这是因为Ansible服务的主配置文件存在优先级的顺序关系，默认存放在/etc/ansible目录中的主配置文件实际优先级最低，如果在当前目录或用户家目录中也同样存放有一份配置文件，那么则是以自己新建立的为准，具体优先级顺序如表16-2所示。

表16-2                   Ansible服务主配置文件优先级顺序

| 优先级 | 文件位置                 |
| ------ | ------------------------ |
| 高     | ./ansible.cfg            |
| 中     | ~/.ansible.cfg           |
| 低     | /etc/ansible/ansible.cfg |



既然Ansible服务是用于实现主机批量自动化控制的管理工具，要受管的主机一定不是1、2台，而是数十台或上千台，那么在生产环境中主机清单（Inventory）可就帮上大忙了。用户可以预先把要管理的主机IP地址写入到/etc/ansible/hosts文件中，这样后续再执行ansible命令执行动作就自动包含这些主机了，无需每次都重复输入受管主机的地址。例如要管理5台主机，对应的IP地址如表16-3所示。

表16-3                   受管主机信息

| 操作系统 | IP地址        | 功能用途  |
| -------- | ------------- | --------- |
| RHEL 8   | 192.168.10.20 | dev       |
| RHEL 8   | 192.168.10.21 | test      |
| RHEL 8   | 192.168.10.22 | prod      |
| RHEL 8   | 192.168.10.23 | prod      |
| RHEL 8   | 192.168.10.24 | balancers |



首先要进行说明，受管主机系统默认使用RHEL 8是为了避免读者在准备实验机阶段产生歧义，而给出的建议值，也可以用其他Linux系统。主机清单文件/etc/ansible/hosts中默认存在大量的注释信息，建议全部删除，替换成实验信息：

```
[root@linuxprobe ~]# vim /etc/ansible/hosts
192.168.10.20
192.168.10.21
192.168.10.22
192.168.10.23
192.168.10.24
```

而为了增加实验难度，通吃生产环境中常见需求，我们又为这五台主机分别规划了功能用途，有开发机（dev）、测试机（test）、产品机（prod）和负载均衡机（balancers）。有了对主机的分组标注，后期在管理方面就方便很多了，继续进行修改为：

```
[root@linuxprobe ~]# vim /etc/ansible/hosts
[dev]
192.168.10.20
[test]
192.168.10.21
[prod]
192.168.10.22
192.168.10.23
[balancers]
192.168.10.24
```

主机清单文件修改后的是会立即生效的，一般常用“ansible-inventory --graph”命令以结构化的方式显示出受管节点主机信息，对于有层级的分组来讲是非常利于阅读的：

```
[root@linuxprobe ~]# ansible-inventory --graph
@all:
  |--@balancers:
  |  |--192.168.10.24
  |--@dev:
  |  |--192.168.10.20
  |--@prod:
  |  |--192.168.10.22
  |  |--192.168.10.23
  |--@test:
  |  |--192.168.10.21
  |--@ungrouped:
```

等等！~先不要着急开始后面的实验，此前讲过Ansible服务是基于SSH协议进行的自动化控制，这是开展后面实验的前提条件。读者们回忆下第9章节在学习时，肯定还记得sshd服务在初次连接时会要求用户接受一次对方主机的指纹信息，并且输入受管节点的主机账号和密码吧~例如正常的第一次SSH远程连接过程是这样的：

```
[root@linuxprobe ~]# ssh 192.168.10.10
The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.
ECDSA key fingerprint is SHA256:QRW1wrqdwN0PI2bsUvBlW5XOIpBjE+ujCB8yiCqjMQQ.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.
root@192.168.10.10's password: 此处应输入管理员密码后回车确认
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Mon Mar 29 06:30:15 2021
[root@linuxprobe ~]# 
```

同学们都知道自动化运维的好处之一就是能提高工作效率，如果每次执行操作都要输入受管主机的密码，也是比较麻烦的事情。好在Ansible服务已经对此有了解决办法，那就是如表16-4所示的变量。

表16-4                   Ansible常用变量汇总

| 参数               | 作用          |
| ------------------ | ------------- |
| ansible_ssh_host   | 受管主机名    |
| ansible_ssh_port   | 端口号        |
| ansible_ssh_user   | 默认账号      |
| ansible_ssh_pass   | 默认密码      |
| ansible_shell_type | Shell终端类型 |



毕竟键盘也是买来的，少敲就是省钱~用户只需要将对应的变量及信息填写到主机清单文件中，在执行任务时便会自动的对账号密码进行匹配，而不用再每次重复输入它们。继续修改主机清单文件：

```
[root@linuxprobe ~]# vim /etc/ansible/hosts
[dev]
192.168.10.20
[test]
192.168.10.21
[prod]
192.168.10.22
192.168.10.23
[balancers]
192.168.10.24
[all:vars]
ansible_user=root
ansible_password=redhat
```

还剩最后一步，将Ansible主配置文件中的第71行设置成默认不需要SSH协议的指纹验证，以及第107行设置成默认执行Playbook时所使用的管理员名称为root：

```
[root@linuxprobe ~]# vim /etc/ansible/ansible.cfg
69
70 # uncomment this to disable SSH key host checking
71 host_key_checking = False
72
………………省略部分输出信息………………
104
105 # default user to use for playbooks if user is not specified
106 # (/usr/bin/ansible will use current user as default)
107 remote_user = root
108
```

不需要重启服务，以上操作完全搞定后就可以开始后面的实验了。由于刚刚是将Ansible服务器设置成了桥接及DHCP模式，请同学们自行将网络适配器修改回“仅主机模式”及192.168.10.10/24的IP地址吧，如图16-3所示。完成后重启网卡再自行在主机之间互相ping一下哦，保证主机之间网络能够互通是后续实验的基石。

```
[root@linuxprobe ~]# ifconfig
ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.10.10  netmask 255.255.255.0  broadcast 192.168.10.255
        inet6 fe80::d0bb:17c8:880d:e719  prefixlen 64  scopeid 0x20
        ether 00:0c:29:7d:27:bf  txqueuelen 1000  (Ethernet)
        RX packets 32  bytes 5134 (5.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 43  bytes 4845 (4.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
………………省略部分输出信息………………
```

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BE%E7%BD%AE%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%BD%91%E5%8D%A1%E6%A8%A1%E5%BC%8F.png)

图16-3 将虚拟机网卡改回仅主机模式

##### **16.3 运行临时命令**

Ansible服务强大之处在于只需要一条命令，便可以操控千台万台的主机节点，ansible命令便是最得力的工具之一。而用户所使用的Ansible服务实际只是一个框架，能够完成工作的是模块化功能代码，常用的模块大致有二十余个，如表16-5所示，本书将会在后面的实验中逐一详解。

偶遇到书中没有提及的，自己感兴趣的模块亦可用“ansible-doc 模块名称”的命令格式自行查询，或是用“ansibe-doc -l”命令列出所有的模块信息以供选择。

表16-5                   Ansible服务常用模块名称及作用

| 模块名称       | 模块作用                                 |
| -------------- | ---------------------------------------- |
| ping           | 检查受管节点主机网络是否能够联通。       |
| yum            | 安装、更新及卸载软件包。                 |
| yum_repository | 管理主机的软件仓库配置文件。             |
| template       | 复制模板文件到受管节点主机。             |
| copy           | 新建、修改及复制文件。                   |
| user           | 创建、修改及删除用户。                   |
| group          | 创建、修改及删除用户组。                 |
| service        | 启动、关闭及查看服务状态。               |
| get_url        | 从网络中下载文件。                       |
| file           | 设置文件权限及创建快捷方式。             |
| cron           | 添加、修改及删除计划任务。               |
| command        | 直接执行用户指定的命令。                 |
| shell          | 直接执行用户指定的命令（支持特殊字符）。 |
| debug          | 输出调试或报错信息。                     |
| mount          | 挂载硬盘设备文件。                       |
| filesystem     | 格式化硬盘设备文件。                     |
| lineinfile     | 通过正则表达式修改文件内容。             |
| setup          | 收集受管节点主机上的系统及变量信息。     |
| firewalld      | 添加、修改及删除防火墙策略。             |
| lvg            | 管理主机的物理卷及卷组设备。             |
| lvol           | 管理主机的逻辑卷设备。                   |



ansible是用于执行临时任务的命令，也就是执行后即结束，不同于Playbook剧本文件的可重复性。使用ansible命令时必须指明受管主机节点的信息，如果已经设置过主机清单文件（/etc/ansible/hosts）则可以写all参数来代指全体受管节点，或用dev、test等主机组名称来代指某一组的主机节点。

常用的语法格式为“ansible 受管主机节点 -m 模块名称 [-a 模块参数]”，常见的参数如表16-6所示。“-a”是要传递给模块的参数，只有极简单功能的模块才不需要额外参数，所以大多情况下“-m”与“-a”参数都会同时出现。

表16-6                   ansible命令常用参数

| 参数      | 作用                    |
| --------- | ----------------------- |
| -k        | 手动输入SSH协议密码     |
| -i        | 指定主机清单文件        |
| -m        | 指定要使用的模块名      |
| -M        | 指定要使用的模块路径    |
| -S        | 使用su命令              |
| -T        | 设置SSH协议连接超时时间 |
| -a        | 设置传递给模块的参数    |
| --version | 查看版本信息            |
| -h        | 帮助信息                |



如果想实现某个功能，但是却不知道用什么模块合适。又或者是知道了模块名称，不清楚模块具体的作用时，建议用ansible-doc命令进行查找。例如列举出当前Ansible服务所支持的所有模块信息：

```
[root@linuxprobe ~]# ansible-doc -l 
a10_server                                           Manage A10 Networks AX/SoftAX/Thunder/v...
a10_server_axapi3                                    Manage A10 Networks AX/SoftAX/Thunder/v...           
a10_service_group                                    Manage A10 Networks AX/SoftAX/Thunder/v...
a10_virtual_server                                   Manage A10 Networks AX/SoftAX/Thunder/v...
aci_aaa_user                                         Manage AAA users (aaa:User)                                              
aci_aaa_user_certificate                             Manage AAA user certificates (aaa:User...                        
aci_access_port_block_to_access_port                 Manage port blocks of Fabric interface ...
aci_access_port_to_interface_policy_leaf_profile     Manage Fabric interface policy leaf pro...
aci_access_sub_port_block_to_access_port             Manage sub port blocks of Fabric interf...
aci_aep                                              Manage attachable Access Entity Profile...
aci_aep_to_domain                                    Bind AEPs to Physical or Virtual Domain...   
aci_bd_subnet                                        Manage Subnets (fv:Subnet)                 
………………省略部分输出信息………………
```

一般情况下，真的很难通过名称来判别一个模块的作用，只能是参考后面的介绍信息或平时多学多练的积累才行。例如接下来随机查看一个模块的详细信息，ansible-doc会在屏幕显示出它的作用、可用参数及实例等等信息：

```
[root@linuxprobe ~]# ansible-doc a10_server
> A10_SERVER    (/usr/lib/python3.6/site-packages/ansible/modules/network/a10/a10_server.py)

     Manage SLB (Server Load Balancer) server objects on A10 Networks devices via aXAPIv2.

  * This module is maintained by The Ansible Community
………………省略部分输出信息………………
```

刚刚在16.2小节，咱们已经成功的将受管主机IP地址填写到了主机清单文件中，接下来小试牛刀，检查下这些主机的网络连通性吧。ping模块用于进行简单的网络测试，类似于常用的ping命令。可以用ansible命令直接针对所有主机调用ping模块，不需要增加额外的参数，返回值若为SUCCESS则代表主机当前在线。

```
[root@linuxprobe ~]# ansible all -m ping
192.168.10.20 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.10.21 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.10.22 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.10.23 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}192.168.10.24 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false,
    "ping": "pong"
}
```

### **Tips**

由于五台受控主机的输出信息大致相同，因此为了提升读者阅读体验，本章节后续的输出结果默认仅保留192.168.10.20主机的输出值，其余相同的输出信息将会被省略。

是不是感觉很方便呢，一下子就能知道所有主机的在线情况。除了使用“-m”参数直接指定模块名称，还可以用“-a”将参数传递给模块，让功能更加高级，更好的贴合于当前生产需求。例如yum_repository模块的作用是管理主机的软件仓库，能够添加、修改及删除软件仓库的配置信息，参数相对比较复杂。遇到这种情况建议先用ansible-doc命令对其进行了解，尤其最下面的EXAMPLES结构段会有对该模块的实例，有着非常好的参考价值。

```
[root@linuxprobe ~]# ansible-doc yum_repository
> YUM_REPOSITORY    (/usr/lib/python3.6/site-packages/ansible/modules/packaging>

        Add or remove YUM repositories in RPM-based Linux
        distributions. If you wish to update an existing repository
        definition use [ini_file] instead.

  * This module is maintained by The Ansible Core Team

……………………省略部分输出信息………………

EXAMPLES:

- name: Add repository
  yum_repository:
    name: epel
    description: EPEL YUM repo
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/

- name: Add multiple repositories into the same file (1/2)
  yum_repository:
    name: epel
    description: EPEL YUM repo
    file: external_repos
    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/
    gpgcheck: no

- name: Add multiple repositories into the same file (2/2)
  yum_repository:
    name: rpmforge
    description: RPMforge YUM repo
    file: external_repos
    baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
```

还好~参数并不是很多，而且跟此前学过在/etc/yum.repos.d/目录中的配置文件基本相似，现在想为主机清单中所有服务器新增一个如表16-7所示的软件仓库，该怎么操作呢？

表16-7                   新增软件仓库信息

| 仓库名称    | EX294_BASE                                     |
| ----------- | ---------------------------------------------- |
| 仓库描述    | EX294 base software                            |
| 仓库地址    | file:///media/cdrom/BaseOS                     |
| GPG签名     | 启用                                           |
| GPG密钥文件 | file:///media/cdrom/RPM-GPG-KEY-redhat-release |


对照EXAMPLE实例段，逐一将需求值和参数对应填写，标准格式是用-a参数后接整体参数，用单引号圈起，而各个参数字段的值则用双引号圈起，是最严谨的写法。执行后出现CHANGED字样信息代表修改已经成功：

```
[root@linuxprobe ~]# ansible all -m yum_repository -a 'name="EX294_BASE" description="EX294 base software" baseurl="file:///media/cdrom/BaseOS" gpgcheck=yes enabled=1 gpgkey="file:///media/cdrom/RPM-GPG-KEY-redhat-release"'

192.168.10.20 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": true,
    "repo": "EX294_BASE",
    "state": "present"
}
```

命令执行成功后，可以到主机清单中任意服务器上查看到新建成功的软件仓库配置文件，实验虽然参数很多，但是并不难对吧~

```
[root@linuxprobe ~]# cat /etc/yum.repos.d/EX294_BASE.repo 
[EX294_BASE]
baseurl = file:///media/cdrom/BaseOS
enabled = 1
gpgcheck = 1
gpgkey = file:///media/cdrom/RPM-GPG-KEY-redhat-release
name = EX294 base software
```

##### **16.4 剧本文件实战**

作为一名技术狂兼戏剧迷，研究生一年级时曾经阅读过莎士比亚的《李尔王》和《暴风雨》两个剧本原作，深深的沉迷于作者对于剧情的巧妙设计中。执行单个的命令或调用某一个模块根本无法满足正常工作所需要的复杂度，Ansible服务中允许用户根据需求，在类似于Shell脚本的模式下编写出一套自动化运维的脚本，然后由程序自动的、重复的执行，大大的提高了工作效率。

Ansible服务的Playbook剧本文件采用YAML语言编写，有着强制性的格式规范，通过空格将不同信息分组，因此有时会因一两个空格错位而导致报错，需要万分小心。YAML文件开头需要先写三个减号---，多个分组信息需要间隔一致才能执行，上下也要对齐，后缀名一般为.yml。在执行后会在屏幕上输出运行界面，内容会依据不同工作而变化，但绿色均代表成功，黄色代表执行成功并进行了修改，而红色则代表执行失败。

Playbook剧本文件的结构由四部分组成——target、variable、task、handler。target部分用于定义要执行剧本的主机节点范围、variable部分用于定义剧本执行时要用的变量、task部分用于定义将在远程主机上执行的任务列表、handler部分用于定义执行完成后需要调用的后续任务。

YAML语言编写的Ansible剧本文件会按照从上至下的顺序自动运行，形式类似于第4章节学习过的Shell脚本，但格式有严格的要求。例如创建出一个名为packages.yml的playbook，目的是让dev、test和prod组的主机节点可以自动安装数据库软件，并且将dev组主机的软件更新至最新。

安装和更新软件需要使用yum模块，先查看下帮助信息中的实例吧：

```
[root@linuxprobe ~]# ansible-doc yum
> YUM    (/usr/lib/python3.6/site-packages/ansible/modules/packaging/os/yum.py)

        Installs, upgrade, downgrades, removes, and lists packages and
        groups with the `yum' package manager. This module only works
        on Python 2. If you require Python 3 support see the [dnf]
        module.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

………………省略部分输出信息………………

EXAMPLES:

- name: install the latest version of Apache
  yum:
    name: httpd
    state: latest
```

在配置Ansible剧本文件时，ansible-doc命令所提供的帮助信息真是好用。知道了yum模块的使用方法和格式了，接下来就可以开始编写了。初次编写Playbook剧本文件时，请务必看准格式，模块及格式时间上下也要对齐，否则会出现参数一摸一样，但是却不能执行的情况。

其中name字段代表此项Play（动作）的名字，可自行命名，没有限制，用于在执行过程中提示用户执行到了那一步，以及帮助管理员日后阅读时回忆起这段代码的作用。hosts字段代表要在哪些主机节点上执行该剧本，多个主机组之间用逗号间隔，若需要对全部主机进行操作则用all参数。tasks字段用于定义要执行的任务，每个任务都要有一个独立的name字段进行命名，并且每个任务的name字段和模块名称都要严格上下对齐，参数要单独缩进。正确的写法应该是：

```
[root@linuxprobe ~]# vim packages.yml
---
- name: 安装软件包
  hosts: dev,test,prod
  tasks:
          - name: one
            yum:
                    name: mariadb
                    state: latest
[root@linuxprobe ~]#
```

而错误的代码是这样的，感受下YAML语言对格式有多严格吧：

```
[root@linuxprobe ~]# vim packages.yml
---
- name: 安装软件包
  hosts: dev,test,prod
  tasks:
          - name: one
            yum:
            name: mariadb
            state: latest
```

在编写Ansible剧本文件时，RHEL 8系统自带的Vim编辑器会自动进行缩进，真是帮了不少忙呢~确认无误后就可以用ansible-playbook命令来运行这个剧本文件了

```
[root@linuxprobe ~]# ansible-playbook packages.yml 

PLAY [安装软件包] *******************************************************************

TASK [Gathering Facts] **************************************************************
ok: [192.168.10.20]
ok: [192.168.10.21]
ok: [192.168.10.22]
ok: [192.168.10.23]

TASK [one] **************************************************************************
changed: [192.168.10.20]
changed: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23]

PLAY RECAP **************************************************************************
192.168.10.20  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.21  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.22  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.23  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
```

执行成功后主要观察最下方的输出信息，ok和changed代表执行及修改成功。如遇到unreachable或failed大约0的情况，建议手动检查下剧本是否在所有主机中都正确运行了，以及有无安装失败的情况。正确执行过packages.yml文件后，随机切换到dev、test、prod组中任意一台节点主机上，再次安装mariadb软件包则会提示已经存在，刚刚的操作一切顺利~

```
[root@linuxprobe ~]# dnf install mariadb
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 1:05:53 ago on Thu 15 Apr 2021 08:29:11 AM CST.
Package mariadb-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

##### **16.5 创建及使用角色**

日常编写Playbook的工作会让剧本内容越来越长，不利于阅读和维护，而且还无法让其他Playbook灵活的调用其中的功能代码。角色功能（roles）则是Ansible服务自1.2版本开始引入的新特性，用于层次性、结构化的组织剧本。角色功能通过分别把变量、文件、任务、模块及处理器配置放在各个独立的目录中，然后对其进行便捷加载的一种机制。简单来说，角色功能是把常用的一些功能“类模块化”，然后用的时候加载即可。

Ansible服务的角色功能也有些类似于编程中的封装技术，将具体的功能封装起来，用户不仅可以方便的调用它，甚至不用完全理解其中原理，也可以使用。就像普通消费者不需要深入理解汽车刹车是如何实现的那样，制动总泵、分泵、真空助力器、刹车盘、刹车鼓、刹车片或ABS泵都是藏于底层结构中，用户只需要用脚轻踩刹车踏板就能制动车子，这便是技术封装的好处。

角色的好处就在于其组织成了一个简洁的、可重复调用的抽象对象，用户可以把注意力放到Playbook的大局上，统筹各个关键性任务，只有在需要时才去深入了解细节。角色的获取有三种方法，分别是加载系统内置的、从外部环境获取的以及自行创建的，接下来将与读者共同逐一完成这些实验。

###### **16.5.1 加载系统内置角色**

使用RHEL系统内置角色，是一种不需要联网就能实现的功能。用户只需要配置好软件仓库的配置文件，然后安装包含系统角色的软件包rhel-system-roles，随后便可以在系统中找到它们了，以及能够使用Playbook剧本文件进行调用。

```
[root@linuxprobe ~]# dnf install -y rhel-system-roles
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 1:06:26 ago on Tue 13 Apr 2021 07:22:03 AM CST.
Dependencies resolved.
================================================================================
 Package                  Arch          Version          Repository        Size
================================================================================
Installing:
 rhel-system-roles        noarch        1.0-5.el8        AppStream        127 k

Transaction Summary
================================================================================
Install  1 Package

………………省略部分输出信息………………  

Installed:
  rhel-system-roles-1.0-5.el8.noarch                                            

Complete!
```

安装完毕后，使用ansible-galaxy list命令查看RHEL 8系统中有哪些自带的角色可用：

```
[root@linuxprobe ~]# ansible-galaxy list
# /usr/share/ansible/roles
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.network, (unknown version)
- linux-system-roles.postfix, (unknown version)
- linux-system-roles.selinux, (unknown version)
- linux-system-roles.timesync, (unknown version)
- rhel-system-roles.kdump, (unknown version)
- rhel-system-roles.network, (unknown version)
- rhel-system-roles.postfix, (unknown version)
- rhel-system-roles.selinux, (unknown version)
- rhel-system-roles.timesync, (unknown version)
# /etc/ansible/roles
[WARNING]: - the configured path /root/.ansible/roles does not exist.
```

千万不要低估这些由系统镜像自带的角色，日常的工作中能排上大用场，它们的主要功能可参考表格16-5所示。

表16-5                   ansible系统角色描述

| 角色名称                   | 作用                  |
| -------------------------- | --------------------- |
| rhel-system-roles.kdump    | 配置kdump崩溃恢复服务 |
| rhel-system-roles.network  | 配置网络接口          |
| rhel-system-roles.selinux  | 配置SELinux策略及模式 |
| rhel-system-roles.timesync | 配置网络时间协议      |
| rhel-system-roles.postfix  | 配置邮件传输服务      |
| rhel-system-roles.firewall | 配置防火墙服务        |
| rhel-system-roles.tuned    | 配置系统调优选项      |



以rhel-system-roles.timesync角色为例，它用于设置系统的时间和NTP服务，让主机能够同步准确的时间信息。Playbook剧本模板文件存放在/usr/share/doc/rhel-system-roles/目录中，可以复制过来修改使用：

```
[root@linuxprobe ~]# cp /usr/share/doc/rhel-system-roles/timesync/example-timesync-playbook.yml timesync.yml
```

NTP服务器是用于计算机时间同步的一种协议，提供高精度的时间校准服务，帮助计算机校对系统时钟。目前常用的是由国际快速授时服务提供的pool.ntp.org，也是稳定性比较好的。在复制来的剧本模板文件中，删除掉多余的代码，将NTP服务器的地址填写到timesync_ntp_servers变量的hostname字段中即可，变量参数含义可参考表格6-6所示，稍后timesync角色就会自动为用户配置参数信息了。

表16-5                  timesync_ntp_servers变量参数含义

| 参数     | 作用            |
| -------- | --------------- |
| hostname | NTP服务器主机名 |
| iburst   | 启用快速同步    |



```
[root@linuxprobe ~]# vim timesync.yml 
---
- hosts: all
  vars:
    timesync_ntp_servers:
      - hostname: pool.ntp.org
        iburst: yes
  roles:
    - rhel-system-roles.timesync
```

###### **16.5.2 从外部获取角色**

Ansible Galaxy是Ansible的官方社群服务，用于共享角色和功能代码，用户可以自由的在网站上共享和下载Ansible角色，是管理和使用角色的不二之选。如图16-4所示的Ansible Galaxy官网中左侧有三个功能选项，分别是首页（HOME）、搜索（Search）以及社区（Community），点击Search按钮进入到搜索界面中，用nginx网站服务为例进行搜索，即可找到官网源发布的角色信息，如图16-5所示。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E9%A6%96%E9%A1%B5-1024x387.png)

图 16-4 Ansible Galaxy 官网首页

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%90%9C%E7%B4%A2%E7%95%8C%E9%9D%A2-1024x447.png)

图16-5 搜索界面中找到nginx角色信息

### **Tips**

Ansible Galaxy 官网首页：[https://galaxy.ansible.com](https://galaxy.ansible.com/)

当点击nginx角色进入到详情页面后，会有这个项目的软件版本、评分、下载次数等简介信息，在Installation字段能看到安装方式，如图16-6所示。在保持虚拟机能够连接外网的前提下，可以按网页提示的命令进行安装。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/Nginx-1024x377.png)

图16-6 nginx角色详情页

这时如果需要使用这个角色，可以在服务器联网的状态下直接按照“ansible-galaxy install 角色名称”的命令格式进行自动获取：

```
[root@linuxprobe ~]# ansible-galaxy install nginxinc.nginx
- downloading role 'nginx', owned by nginxinc
- downloading role from https://github.com/nginxinc/ansible-role-nginx/archive/0.19.1.tar.gz
- extracting nginxinc.nginx to /etc/ansible/roles/nginxinc.nginx
- nginxinc.nginx (0.19.1) was installed successfully
```

执行完毕后再次查看系统中已有角色，便找到nginx角色信息了：

```
[root@linuxprobe ~]# ansible-galaxy list
# /etc/ansible/roles
- nginxinc.nginx, 0.19.1
# /usr/share/ansible/roles
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.network, (unknown version)
- linux-system-roles.postfix, (unknown version)
- linux-system-roles.selinux, (unknown version)
- linux-system-roles.timesync, (unknown version)
- rhel-system-roles.kdump, (unknown version)
- rhel-system-roles.network, (unknown version)
- rhel-system-roles.postfix, (unknown version)
- rhel-system-roles.selinux, (unknown version)
- rhel-system-roles.timesync, (unknown version)
```

但还存在一些特殊情况，其一是国内访问Ansible官网可能存在不稳定的情况，访问不了或者速度慢。其二是某位作者是将创作品上传到了自己的网站，或者除Ansible Galaxy官网以外的其他平台。那么这两种情况就不能再用“ansible-galaxy install 角色名称”的命令直接加载了，而是需要手动先编写一个YAML语言格式的文件，指明网址链接和角色名称，再用“-r”参数进行加载。

例如刘遄老师在随书配套网站（Www.LinuxProbe.Com）上传了一个叫做nginx_core的角色软件包，是用于对nginx网站进行保护的插件功能，编写yml配置文件如下：

```
[root@linuxprobe ~]# cat nginx.yml
---
- src: https://www.linuxprobe.com/Software/nginxinc-nginx_core-0.3.0.tar.gz
  name: nginx-core
```

随后使用ansible-galaxy命令的“-r”参数加载这个文件，即可查看到新角色信息了

```
[root@linuxprobe ~]# ansible-galaxy install -r nginx.yml
- downloading role from https://www.linuxprobe.com/nginxinc-nginx_core-0.3.0.tar.gz
- extracting nginx to /etc/ansible/roles/nginx
- nginx was installed successfully
[root@linuxprobe ~]# ansible-galaxy list
# /etc/ansible/roles
- nginx-core, (unknown version)
- nginxinc.nginx, 0.19.1
# /usr/share/ansible/roles
- linux-system-roles.kdump, (unknown version)
- linux-system-roles.network, (unknown version)
- linux-system-roles.postfix, (unknown version)
- linux-system-roles.selinux, (unknown version)
- linux-system-roles.timesync, (unknown version)
- rhel-system-roles.kdump, (unknown version)
- rhel-system-roles.network, (unknown version)
- rhel-system-roles.postfix, (unknown version)
- rhel-system-roles.selinux, (unknown version)
- rhel-system-roles.timesync, (unknown version)
```

###### **16.5.3 创建新的角色**

除了能够使用系统镜像自带和Ansible Galaxy中获取的外部角色，也可以自行创建符合工作需求的新角色信息，定制化的编写工作能够更好的贴合生产环境实际情况，但难度也会稍高一些。接下来将会创建一个名称为apache的新角色信息，它能够帮助我们自动的安装、运行httpd网站服务，设置防火墙允许规则及根据每个主机生成独立的index.html首页文件，让用户调用后就能享受到一条龙的部署网站服务。

根据主配置文件第68行所定义的角色保存路径，如果用户新建的角色信息不在规定目录内，使用ansible-galaxy list命令则是无法找到的。因此需要手动填写下新角色目录的路径，或是进入到/etc/ansible/roles目录内再进行创建，为了避免后期角色信息过于分散导致不好管理，还是决定在默认目录下进行创建，不再修改。

```
[root@linuxprobe roles]# vim /etc/ansible/ansible.cfg
 66 
 67 # additional paths to search for roles in, colon separated
 68 #roles_path    = /etc/ansible/roles
 69 
```

创建一个新的角色信息使用“init”参数，且建立成功后便会在当前目录下生成出一个新的目录：

```
[root@linuxprobe ~]# cd /etc/ansible/roles
[root@linuxprobe roles]# ansible-galaxy init apache
- Role apache was created successfully
[root@linuxprobe roles]# ls
apache nginx nginxinc.nginx
```

此时的apache即是角色名称，也是用于存在角色信息的目录名称，切换进入看到目录结构：

```
[root@linuxprobe roles]# cd apache
[root@linuxprobe apache]# ls
defaults  files  handlers  meta  README.md  tasks  templates  tests  vars
```

创建新的角色最关键的便是对目录结构的理解，通俗来说就是要把正确的信息放入到正确的目录中，这样调用角色时才能有正确的效果。角色信息对应的目录结构及含义如表16-8所示。

表16-8                  Ansible角色目录结构及含义

| 目录      | 含义                                           |
| --------- | ---------------------------------------------- |
| defaults  | 包含角色变量的默认值（优先级低）。             |
| files     | 包含角色执行tasks任务时做引用的静态文件。      |
| handlers  | 包含角色的处理程序定义。                       |
| meta      | 包含角色的作者、许可证、频台和依赖关系等信息。 |
| tasks     | 包含角色所执行的任务。                         |
| templates | 包含角色任务所使用的Jinja2模板。               |
| tests     | 包含用于测试角色的剧本文件。                   |
| vars      | 包含角色变量的默认值（优先级高）。             |



第1步：打开编写用于定义角色任务的tasks/main.yml文件。该文件中不需要定义要执行的主机组列表，因为后面会单独编写Playbook剧本进行调用，此时应先对apache角色能做的事情有个明确的思路，调用角色后yml文件会按照从上至下的顺序自动执行。

> 1：安装httpd网站服务。
>
> 2：运行httpd网站服务，并加入到开机启动项中。
>
> 3：配置防火墙放行http网站协议。
>
> 4：根据每台主机的变量值，生成不同的主页文件。

先写出第一个任务，使用yum模块安装httpd网站服务程序，注意格式：

```
[root@linuxprobe apache]# vim tasks/main.yml
---
- name: one
  yum:
          name: httpd
          state: latest
```

第2步：使用service模块启动httpd网站服务程序，并加入到启动项中，保证能够一直为用户提供服务。初次使用模块前先用ansible-doc命令查看下帮助和实例信息吧，但由于书籍出版的限制，信息会做删减，仅保留有用的段落内容。

```
[root@linuxprobe apache]# ansible-doc service
> SERVICE    (/usr/lib/python3.6/site-packages/ansible/modules/system/service.py)

        Controls services on remote hosts. Supported init systems
        include BSD init, OpenRC, SysV, Solaris SMF, systemd, upstart.
        For Windows targets, use the [win_service] module instead.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

………………省略部分输出信息………………

EXAMPLES:

- name: Start service httpd, if not started
  service:
    name: httpd
    state: started

- name: Enable service httpd, and not touch the state
  service:
    name: httpd
    enabled: yes
```

真幸运，默认的EXAMPLES示例用的就是httpd网站服务，通过输出信息可得知启动服务为“state: started”参数，而加入到开机启动项则是“enabled: yes”参数。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml
---
- name: one
  yum:
          name: httpd
          state: latest
- name: two
  service:
          name: httpd
          state: started
          enabled: yes
```

第3步：配置防火墙放行允许策略，让其他主机可以正常访问。配置防火墙使用firewalld模块，同样也先看下帮助示例吧：

```
[root@linuxprobe defaults]# ansible-doc firewalld
> FIREWALLD    (/usr/lib/python3.6/site-packages/ansible/modules/system/firewalld.py)

        This module allows for addition or deletion of services and
        ports (either TCP or UDP) in either running or permanent
        firewalld rules.

  * This module is maintained by The Ansible Community
OPTIONS (= is mandatory):
EXAMPLES:

- firewalld:
    service: https
    permanent: yes
    state: enabled

- firewalld:
    port: 8081/tcp
    permanent: yes
    state: disabled
    immediate: yes
```

依据输出信息可得知，firewalld模块设置防火墙策略中，指定协议名称为“service: http”参数，放行该协议为“state: enabled”参数，设置为永久生效为“permanent: yes”参数，当前也能立即生效为“immediate: yes”参数。参数虽然多了一些，但是基本与在第8章节学习的一致，并不需要担心。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml
---
- name: one
  yum:
          name: httpd
          state: latest
- name: two
  service:
          name: httpd
          state: started
          enabled: yes
- name: three
  firewalld:
          service: http
          permanent: yes
          state: enabled
          immediate: yes
```

第4步：让每个主机显示的首页内容均不同。常规的模块都是这样查一个、写一个就能搞定，为了增加难度再提出个新需求，能否让每个主机上运行的httpd网站服务都有不同的内容，例如显示当前服务器的主机名及IP地址呢？这样的话就要用到template模块及Jinja2技术了。

template模块的使用方法依然用ansible-doc命令进行查询，示例部分有很大帮助：

```
[root@linuxprobe apache]# ansible-doc template
> TEMPLATE    (/usr/lib/python3.6/site-packages/ansible/modules/files/template.>

        Templates are processed by the L(Jinja2 templating
        language,http://jinja.pocoo.org/docs/). Documentation on the
        template formatting can be found in the L(Template Designer
        Documentation,http://jinja.pocoo.org/docs/templates/).
        Additional variables listed below can be used in templates.
        `ansible_managed' (configurable via the `defaults' section of
        `ansible.cfg') contains a string which can be used to describe
        the template name, host, modification time of the template
        file and the owner uid. `template_host' contains the node name
        of the template's machine. `template_uid' is the numeric user
        id of the owner. `template_path' is the path of the template.
        `template_fullpath' is the absolute path of the template.
        `template_destpath' is the path of the template on the remote
        system (added in 2.8). `template_run_date' is the date that
        the template was rendered.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

………………省略部分输出信息………………

EXAMPLES:

- name: Template a file to /etc/files.conf
  template:
    src: /mytemplates/foo.j2
    dest: /etc/file.conf
    owner: bin
    group: wheel
    mode: '0644'
```

从template模块的输出信息中可得知，这是一个用于复制文件模板的模块，能够把文件从Ansible服务器复制到受管主机上，“src”参数用于定义本地文件的路径，“dest”参数用于定义复制到受管主机的文件路径，而owner、group、mode参数可选择性的设置文件归属及权限信息。

正常来说复制文件的操作是直接进行的，受管主机节点上会获取到一个与Ansible服务器上一摸一样的文件，但有些时候想让每台客户端根据自身系统情况产生不同的文件信息，这样就需要使用到Jinja2技术了，文件后缀是.j2结尾。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml
---
- name: one
  yum:
          name: httpd
          state: latest
- name: two
  service:
          name: httpd
          state: started
          enabled: yes
- name: three
  firewalld:
          service: http
          permanent: yes
          state: enabled
          immediate: yes
- name: four
  template:
          src: index.html.j2
          dest: /var/www/html/index.html
```

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/jinja.png)

Jinja2是Python语言中一个被广泛使用的模板引擎，最初的设计思想来源是Django的模块引擎，基于此发展了其语法和一些列强大的功能，能够让受管主机根据自身变量而产生出不同的文件内容。话句话说，正常情况下的复制操作会让新旧文件一摸一样，但Jinja2技术不会在原始文件中直接写入文件内容，而是一系列的变量名称，在使用template模块进行复制的过程中，由Ansible服务负责在受管主机上收集这些变量名称所对应的值，最后再逐一填写到目标文件中，能够让每台主机的文件都根据自身系统情况来独立生成。

例如想要让每个网站的输出信息值为“Welcome to 主机名 on 主机地址”，也就是用每个主机自己独有的名称和IP地址替换文本中的内容，这样就有趣太多了。这种实验的难点主要是查询到对应的变量名称，主机名及地址所对应的值保存在哪里？可以用setup模块进行查询。

```
[root@linuxprobe apache]# ansible-doc setup
> SETUP    (/usr/lib/python3.6/site-packages/ansible/modules/system/setup.py)

        This module is automatically called by playbooks to gather
        useful variables about remote hosts that can be used in
        playbooks. It can also be executed directly by
        `/usr/bin/ansible' to check what variables are available to a
        host. Ansible provides many `facts' about the system,
        automatically. This module is also supported for Windows
        targets.
```

setup模块的作用是自动收集受管主机上的变量信息，用-a参数追加filter指令可以对收集来的进行进行二次过滤，语法格式为ansible all -m setup -a 'filter="*关键词*"'，其中*号是第三章节学习的通配符，进行关键词查询。例如想搜索各个主机的名称，即用通配符搜索所有包含fqdn关键词的变量值信息。

Fully Qualified Domain Name即完全限定主机名，简称FQDN，用于在逻辑上准确表示出主机的位置，也常常被作为主机名的完全表达形式，比/etc/hostname文件中所定义的主机名更加严谨和准确。通过输出信息可得知，ansible_fqdn变量保存有主机名称，随后再进行下一步操作：

```
[root@linuxprobe ~]# ansible all -m setup -a 'filter="*fqdn*"'
192.168.10.20 | SUCCESS => {
    "ansible_facts": {
        "ansible_fqdn": "linuxprobe.com",
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
………………省略部分输出信息………………
```

用于定制主机地址的变量可以用“ip”作为关键词进行检索，能够看到在ansible_all_ipv4_addresses变量中的值是我们想要的信息。如果想输出IPV6级别的地址则可用ansible_all_ipv6_addresses变量。

```
[root@linuxprobe ~]# ansible all -m setup -a 'filter="*ip*"'
192.168.10.20 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "192.168.10.20",
            "192.168.122.1"
        ],
        "ansible_all_ipv6_addresses": [
            "fe80::d0bb:17c8:880d:e719"
        ],
        "ansible_default_ipv4": {},
        "ansible_default_ipv6": {},
        "ansible_fips": false,
        "discovered_interpreter_python": "/usr/libexec/platform-python"
    },
    "changed": false
}
………………省略部分输出信息………………
```

在确认了主机名与IP地址所对应的具体变量名称后，在角色所对应的templates目录内新建一个与上面template模块参数相同的文件名称（index.html.j2）。Jinja2在调用变量值时，格式为在变量名称的两侧格加两个大括号，编写完成即：

```
[root@linuxprobe apache]# vim templates/index.html.j2
Welcome to {{ ansible_fqdn }} on {{ ansible_all_ipv4_addresses }}
```

进行到了这里，任务基本就算完成了。最后要做的就是编写一个用于调用apache角色的yml文件，以及执行它。

```
[root@linuxprobe apache]# cd ~
[root@linuxprobe ~]# vim roles.yml
---
- name: 调用自建角色
  hosts: all
  roles:
          - apache
[root@linuxprobe ~]# ansible-playbook roles.yml 
PLAY [调用自建角色] **************************************************************************

TASK [Gathering Facts] **********************************************************************
ok: [192.168.10.20]
ok: [192.168.10.21]
ok: [192.168.10.22]
ok: [192.168.10.23]
ok: [192.168.10.24]

TASK [apache : one] *************************************************************************
changed: [192.168.10.20]
changed: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23]
changed: [192.168.10.24]

TASK [apache : two] *************************************************************************
changed: [192.168.10.20]
changed: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23]
changed: [192.168.10.24]

TASK [apache : three] ***********************************************************************
changed: [192.168.10.20]
changed: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23]
changed: [192.168.10.24]

TASK [apache : four] ***********************************************************************
changed: [192.168.10.20]
changed: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23] 
changed: [192.168.10.24]

PLAY RECAP **********************************************************************************
192.168.10.20   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.21   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.22   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.23   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
192.168.10.24   : ok=4   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
```

执行完毕后，在浏览器中随机输入几个主机的IP地址，即可访问到包含有主机FQDN完全限定主机名和IP地址的网页了，自此实验完美实现，如图16-7、图16-8及图16-9所示。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BF%E9%97%AE%E7%BD%91%E7%AB%99-1024x170.png)

图16-7 随机访问一台主机节点的网站首页

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/2-34-1024x160.png)

图16-8 随机访问一台主机节点的网站首页![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/3-15-1024x166.png)

图16-9 随机访问一台主机节点的网站首页

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **16.6 创建和使用逻辑卷**

创建一个能批量、自动管理LVM逻辑卷设备的Playbook，能够大大提高对硬盘设备的管理效率，不仅如此，更能避免由于手动创建所带来的错误。例如想要在每个受管主机上都创建出一个名称叫做data的逻辑卷设备，大小为150M，归属于research卷组。如果创建成功，则进一步用ext4文件系统进行格式化操作，若创建失败，则给用户输出一条报错提醒，以便排查原因。

在这种背景下，使用Ansible剧本方式要比使用Shell脚本优势大很多，主要有两点原因。其一，Ansible模块化的功能让操作更标准，只要在执行过程中无报错，那么便会依据远程主机的系统版本及配置自动做出判断和操作，不用担心受到系统变化而导致命令失效的问题。其二，Ansible服务在执行剧本文件时会进行判断，如果该文件或该设备已经被创建过，或是某个动作（Play）已经被执行过，则绝对不会再重复的执行，而使用Shell脚本有可能导致设备被重复格式化，导致数据丢失。

首先在两台prod组的主机上分别添加一块硬盘设备，大小为20G，类型为SCSI，其余选项默认，如图16-10、16-11与16-12所示。

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/1-42.png)

图16-10 添加一块新硬盘
![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/2-33.png)

图16-11 设置硬盘类型

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/3-14.png)

图16-12 新硬盘添加完毕

通过回忆第7章节所学习过的LVM逻辑卷的知识，我们应该让Playbook剧本文件依次创建PV物理卷、VG卷组及LV逻辑卷。先需要使用lvg模块让设备支持逻辑卷技术，并创建出一个名为research的卷组，帮助信息如下：

```
[root@linuxprobe ~]# ansible-doc lvg
> LVG    (/usr/lib/python3.6/site-packages/ansible/modules/system/lvg.py)

        This module creates, removes or resizes volume groups.

  * This module is maintained by The Ansible Community

………………省略部分输出信息………………

EXAMPLES:

- name: Create a volume group on top of /dev/sda1 with physical extent size = 3>
  lvg:
    vg: vg.services
    pvs: /dev/sda1
    pesize: 32

- name: Create a volume group on top of /dev/sdb with physical extent size = 12>
  lvg:
    vg: vg.services
    pvs: /dev/sdb
    pesize: 128K
```

通过输出信息可得知，创建PV和VG卷组的lvg模块总共有三个必备参数。其中“vg”参数用于定义卷组的名称，“pvs”参数用于指定硬盘设备名称，而“pesize”参数用于确定最终卷组的容量大小，可以用PE个数亦可用容量值进行指定。这样的话先创建出一个由/dev/sdb设备组成的名称为research，大小为150M的卷组设备吧。

```
[root@linuxprobe ~]# vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: all
  tasks:
          - name: one
            lvg:
                    vg: research
                    pvs: /dev/sdb
                    pesize: 150M
```

由于刚刚仅在prod组的两台主机上添加了新硬盘设备文件，因此稍后执行时其余三台主机节点会提示未创建成功，属于正常情况。接下来便是用lvol模块来创建出LVM逻辑卷设备了，先查看下模块帮助信息吧：

```
[root@linuxprobe ~]# ansible-doc lvol
> LVOL    (/usr/lib/python3.6/site-packages/ansible/modules/system/lvol.py)

        This module creates, removes or resizes logical volumes.

  * This module is maintained by The Ansible Community

………………省略部分输出信息………………

EXAMPLES:

- name: Create a logical volume of 512m
  lvol:
    vg: firefly
    lv: test
    size: 512

- name: Create a logical volume of 512m with disks /dev/sda and /dev/sdb
  lvol:
    vg: firefly
    lv: test
    size: 512
    pvs: /dev/sda,/dev/sdb
```

通过输出信息可得知，lvol确定是用于创建LVM逻辑卷设备的模块，其中“vg”参数用于指定卷组名称，“lv”参数用于指定逻辑卷名称，而“size”参数则用于指定最终逻辑卷设备的容量大小，不加单位默认为M。填写好参数，创建出一个大小为150M，归属于research卷组，名称为data的逻辑卷设备:

```
[root@linuxprobe ~]# vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: all
  tasks:
          - name: one
            lvg:
                    vg: research
                    pvs: /dev/sdb
                    pesize: 150M
          - name: two
            lvol:
                    vg: research
                    lv: data
                    size: 150M
```

这样还不够好，如果能再将创建出的/dev/research/data逻辑卷设备自动用ext4文件系统进行格式化操作，则又能帮助运维管理员减少了一些工作量。对于设备的文件系统格式化操作使用filesystem模块完成，帮助信息如下：

```
[root@linuxprobe ~]# ansible-doc filesystem
> FILESYSTEM    (/usr/lib/python3.6/site-packages/ansible/modules/system/filesy>

        This module creates a filesystem.

  * This module is maintained by The Ansible Community

………………省略部分输出信息………………

EXAMPLES:

- name: Create a ext2 filesystem on /dev/sdb1
  filesystem:
    fstype: ext2
    dev: /dev/sdb1
```

filesystem模块的参数真是简练，参数“fstype”用于指定文件系统的格式化类型，“dev”参数用于指定要格式化的设备文件路径。继续编写：

```
[root@linuxprobe ~]# vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: all
  tasks:
          - name: one
            lvg:
                    vg: research
                    pvs: /dev/sdb
                    pesize: 150M
          - name: two
            lvol:
                    vg: research
                    lv: data
                    size: 150M
          - name: three
            filesystem:
                    fstype: ext4
                    dev: /dev/research/data 
```

这样按照顺序执行下来，LVM逻辑卷设备就能够自动创建好了。等等！还有个问题没有解决！现在只有prod组主机上面添加了新的硬盘设备文件，其余主机是无法按照既定模块顺利完成的，这时要用类似于第4章节学习过的if条件句的方式进行一次判断——如果失败.....则怎么样....。

首先将上述的三个模块命令用block操作符作为一个整体，相当于是对这三个模块执行结果作为一个整体判断。然后使用rescue操作符进行救援，只有block块中的模块执行失败了才会调用rescue中的救援模块。其中debug模块的msg参数的作用是如果block中的模块执行失败则输出一条信息到屏幕，给予用户一定的提醒作用。完成编写后是这个样子的：

```
[root@linuxprobe ~]# vim lv.yml
---
- name: 创建和使用逻辑卷
  hosts: all
  tasks:
          - block:
                  - name: one
                    lvg:
                            vg: research
                            pvs: /dev/sdb
                            pesize: 150M
                  - name: two
                    lvol:
                            vg: research
                            lv: data
                            size: 150M
                  - name: three
                    filesystem:
                            fstype: ext4
                            dev: /dev/research/data
            rescue:
                    - debug:
                            msg: "Could not create logical volume of that size"
```

YAML语言对于格式有着硬性的要求，既然rescue是对block内的模块进行救援的功能代码，因此两个操作符必须严格对齐，错开一个空格都会导致Playbook执行失败。确认无误后，执行lv.yml剧本文件检阅下效果吧：

```
[root@linuxprobe ~]# ansible-playbook lv.yml 

PLAY [创建和使用逻辑卷] *********************************************************

TASK [Gathering Facts] *********************************************************
ok: [192.168.10.20]
ok: [192.168.10.21]
ok: [192.168.10.22]
ok: [192.168.10.23]
ok: [192.168.10.24]

TASK [one] *********************************************************************
fatal: [192.168.10.20]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}
fatal: [192.168.10.21]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}
changed: [192.168.10.22]
changed: [192.168.10.23]
fatal: [192.168.10.24]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}

TASK [two] *********************************************************************
changed: [192.168.10.22]
changed: [192.168.10.23]

TASK [three] *********************************************************************
changed: [192.168.10.22]
changed: [192.168.10.23]

TASK [debug] *******************************************************************
ok: [192.168.10.20] => {
    "msg": "Could not create logical volume of that size"
}
ok: [192.168.10.21] => {
    "msg": "Could not create logical volume of that size"
}
ok: [192.168.10.24] => {
    "msg": "Could not create logical volume of that size"
}

PLAY RECAP *********************************************************************
192.168.10.20  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0   
192.168.10.21  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0   
192.168.10.22  : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
192.168.10.23  : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
192.168.10.24  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0 
```

在Playbook运行完毕后的执行记录（PLAY RECAP）中可以很清晰的看出只有192.168.10.22及192.168.10.23两台prod组中的主机执行成功了，其余三台主机均触发了rescue救援功能。登录到任意一台prod组的主机节点上，找到新建的逻辑卷设备信息：

```
[root@linuxprobe ~]# lvdisplay 
  --- Logical volume ---
  LV Path                /dev/research/data
  LV Name                data
  VG Name                research
  LV UUID                EOUliC-tbkk-kOJR-8NaH-O9XQ-ijrK-TgEYGj
  LV Write Access        read/write
  LV Creation host, time linuxprobe.com, 2021-04-23 11:00:21 +0800
  LV Status              available
  # open                 0
  LV Size                5.00 GiB
  Current LE             1
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:2
………………省略部分输出信息………………
```

##### **16.7 判断主机组名**

在上面Playbook剧本实验中，我们可以让不同的主机根据自身不同的变量信息而生成出独特的网站首页文件，但却无法对某个主机组进行针对性操作。其实在每个客户端中都会有一个叫做“inventory_hostname”的变量，用于定义着每个节点主机所对应的Ansible服务主机组名称，也就是在/etc/ansible/hosts文件中所对应的分组信息，例如dev、test、prod、balancers。

“inventory_hostname”是Ansible服务中的魔法变量，意味着无法用setup模块直接进行查询，诸如“ansible all -m setup -a 'filter="*关键词*"'”的命令将对它失效。魔法变量需要在执行Playbook剧本文件时的[Gathering Facts]阶段进行搜集，直接查询是看不到的，而只能在剧本文件中进行调用。

获得了存储主机组名称的变量名称，接下便开始实践，需求如下：

> 若主机节点在dev分组中，则修改/etc/issue文件内容为“Development”；
>
> 若主机节点在test分组中，则修改/etc/issue文件内容为“Test”；
>
> 若主机节点在prod分组中，则修改/etc/issue文件内容为“Production”。

第1步：Ansible服务常用模块名称，依据上文所提及的表格16-5新建、修改及复制文件需要用的copy模块，此时便派上了用场。先查询下copy模块的帮助信息：

```
[root@linuxprobe ~]# ansible-doc copy
> COPY    (/usr/lib/python3.6/site-packages/ansible/modules/files/copy.py)

        The `copy' module copies a file from the local or remote
        machine to a location on the remote machine. Use the [fetch]
        module to copy files from remote locations to the local box.
        If you need variable interpolation in copied files, use the
        [template] module. Using a variable in the `content' field
        will result in unpredictable output. For Windows targets, use
        the [win_copy] module instead.

  * This module is maintained by The Ansible Core Team
  * note: This module has a corresponding action plugin.

………………省略部分输出信息………………

EXAMPLES:

- name: Copy file with owner and permissions
  copy:
    src: /srv/myfiles/foo.conf
    dest: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'

- name: Copy using inline content
  copy:
    content: '# This file was moved to /etc/other.conf'
    dest: /etc/mine.conf
```

在输出信息中列举了两种管理文件内容的示例，第一是对于文件的复制行为，第二是通过“content”参数定义内容，“dest”参数指定新建文件的名称，显然第二种更加符合当前的实验场景。编写剧本文件如下：

```
[root@linuxprobe ~]# vim issue.yml
---
- name: 修改文件内容
  hosts: all
  tasks:
          - name: one
            copy:
                    content: 'Development'
                    dest: /etc/issue
          - name: two
            copy:
                    content: 'Test'
                    dest: /etc/issue
          - name: three
            copy:
                    content: 'Production'
                    dest: /etc/issue
```

但按照这种顺序执行下去，每一台主机节点的/etc/issue文件都会被重复修改三次，最终定格在“Production”字样，显然缺少了一些东西。我们应该依据“inventory_hostname”变量中的值进行判断，若主机为dev组则执行第一个Play，若主机为test组则执行第二个Play，若主机为prod组则执行第三个Play，因此要进行三次判断。

when是进行判断的语法，需要在每个Play下方进行判断，只有满足条件才会执行：

```
[root@linuxprobe ~]# vim issue.yml
---
- name: 修改文件内容
  hosts: all
  tasks:
          - name: one
            copy:
                    content: 'Development'
                    dest: /etc/issue
            when: "inventory_hostname in groups.dev"
          - name: two
            copy:
                    content: 'Test'
                    dest: /etc/issue
            when: "inventory_hostname in groups.test"
          - name: three
            copy:
                    content: 'Production'
                    dest: /etc/issue
            when: "inventory_hostname in groups.prod"
```

执行Playbook剧本文件，过程中可清晰的看出由于when语法的作用，未在指定主机组中的节点将被skipping（跳过）：

```
[root@linuxprobe ~]# ansible-playbook issue.yml 

PLAY [修改文件内容] ************************************************************************

TASK [Gathering Facts] ********************************************************************
ok: [192.168.10.20]
ok: [192.168.10.21]
ok: [192.168.10.22]
ok: [192.168.10.23]
ok: [192.168.10.24]

TASK [one] ********************************************************************************
changed: [192.168.10.20]
skipping: [192.168.10.21]
skipping: [192.168.10.22]
skipping: [192.168.10.23] 
skipping: [192.168.10.24]

TASK [two] ********************************************************************************
skipping: [192.168.10.20]
changed: [192.168.10.21]
skipping: [192.168.10.23]
skipping: [192.168.10.24]
skipping: [192.168.10.25]

TASK [three] ******************************************************************************
skipping: [192.168.10.20]
skipping: [192.168.10.21]
changed: [192.168.10.22]
changed: [192.168.10.23]
skipping: [192.168.10.24]

PLAY RECAP ********************************************************************************
192.168.10.20   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   
192.168.10.21   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   
192.168.10.22   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   
192.168.10.23   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0 
192.168.10.24   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0 
```

登录到dev组的192.168.10.20主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue 
Development
```

登录到test组的192.168.10.21主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue 
Test
```

登录到prod组的192.168.10.22/23主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue 
Production
```

##### **16.8 管理文件属性**

学习Playbook剧本语法的目的是满足日常的工作需求，能够把重复的事情写入到脚本中，然后再批量的执行下去，提高运维工作的效率。其中创建文件、管理权限及做快捷方式一定是几乎每天都被使用到的高频技能。尤其是在第5章节学习文件一般权限、特殊权限、隐藏权限时，往往还会因命令的格式问题导致出错，这么多命令该怎么记呢？

Ansible服务为了心疼运维人员，将对文件管理常用的功能都合并到了file模块中，不用再为了寻找模块而“东奔西跑”了，先来看下帮助信息吧：

```
[root@linuxprobe ~]# ansible-doc file
> FILE    (/usr/lib/python3.6/site-packages/ansible/modules/files/file.py)

        Set attributes of files, symlinks or directories.
        Alternatively, remove files, symlinks or directories. Many
        other modules support the same options as the `file' module -
        including [copy], [template], and [assemble]. For Windows
        targets, use the [win_file] module instead.

  * This module is maintained by The Ansible Core Team

………………省略部分输出信息………………

EXAMPLES:

- name: Change file ownership, group and permissions
  file:
    path: /etc/foo.conf
    owner: foo
    group: foo
    mode: '0644'

- name: Create a symbolic link
  file:
    src: /file/to/link/to
    dest: /path/to/symlink
    owner: foo
    group: foo
    state: link

- name: Create a directory if it does not exist
  file:
    path: /etc/some_directory
    state: directory
    mode: '0755'

- name: Remove file (delete file)
  file:
    path: /etc/foo.txt
    state: absent
```

通过上面的输出示例，读者们已经能够了解到file模块的基本参数了，“path”参数定义文件的路径、“owner”参数定义文件所有者、“group”参数定义文件所有组、“mode”参数定义文件权限、“src”参数定义源文件的路径、“dest”参数定义目标文件的路径、“state”参数则定义了文件类型。基本上把第5章节学习过的管理文件权限的功能都包含在内了，那么就来挑战下面的实验吧：

> 请创建出一个名为/linuxprobe的新目录，所有者及所有组均为root管理员身份；
>
> 设置所有者和所有组拥有对文件的完全控制权，而其他人则只有阅读和执行权限；
>
> 给予SGID特殊权限；
>
> 仅在dev主机组主机节点实施。

第二条要求是算术题，将权限描述转换为数字法，即可读为4、可写为2、可执行为1，请读者先自行默默算一下答案。此前在编写Playbook剧本文件时“hosts”参数一直对应的是all，即全体主机节点，而这次也改为仅对dev主机组成员生效，请小心谨慎。编写模块代码如下：

```
[root@linuxprobe ~]# vim chmod.yml
---
- name: 管理文件属性
  hosts: dev
  tasks:
          - name: one
            file:
                    path: /linuxprobe
                    state: directory 
                    owner: root
                    group: root
                    mode: '2775'
```

一不小心把题目出简单了，没能完全展示出file模块的强大之处。再临时加一个需求吧——请将新创建的目录做一个快捷方式到/linuxcool目录。这样用户在访问两个文件时都能有相同的内容了，使用file模块做快捷方式时，不需要再单独创建目标文件，Ansible服务会帮咱们完成的：

```
[root@linuxprobe ~]# vim chmod.yml
---
- name: 管理文件属性
  hosts: dev
  tasks:
          - name: one
            file:
                    path: /linuxprobe
                    state: directory 
                    owner: root
                    group: root
                    mode: '2775'
          - name: two
            file:
                    src: /linuxprobe
                    dest: /linuxcool
                    state: link
```

Playbook剧本文件执行过程：

```
[root@linuxprobe ~]# ansible-playbook chmod.yml 

PLAY [管理文件属性] ***************************************************************

TASK [Gathering Facts] ***********************************************************
ok: [192.168.10.20]
ok: [192.168.10.21]
ok: [192.168.10.22]
ok: [192.168.10.23]
ok: [192.168.10.24]

TASK [one] ***********************************************************************
changed: [192.168.10.20]
skipping: [192.168.10.21]
skipping: [192.168.10.22]
skipping: [192.168.10.23]
skipping: [192.168.10.24]

TASK [two] ***********************************************************************
changed: [192.168.10.20]
skipping: [192.168.10.21]
skipping: [192.168.10.22]
skipping: [192.168.10.23]
skipping: [192.168.10.24]

PLAY RECAP ***********************************************************************
192.168.10.20   : ok=3  changed=2  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   
192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0
192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0
192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0
192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0
```

进入到dev组的主机中，查看到/linuxprobe目录及/linuxcool快捷方式均已经被顺利创建，实验顺利完成：

```
[root@linuxprobe ~]# ls -ld /linuxprobe
drwxrwsr-x. 2 root root 6 Apr 20 09:52 /linuxprobe
[root@linuxprobe ~]# ls -ld /linuxcool
lrwxrwxrwx. 1 root root 11 Apr 20 09:52 /linuxcool -> /linuxprobe
```

##### **16.9 管理密码库文件**

Ansible服务自1.5版本发布后，“Vault”作为一项新功能进入到了运维人员的视野，它不仅实现对密码、剧本等敏感信息进行加密，甚至可以加密变量名称和变量值，保护了数据不会被别人轻易的阅读到。使用ansible-vault命令可以实现对内容的新建（create）、加密（encrypt）、解密（decrypt ）、修改口令（rekey）及查看（view）等等功能。

第1步：例如创建出一个名为locker.yml的配置文件，其中保存有两个变量值：

```
[root@linuxprobe ~]# vim locker.yml
---
pw_developer: Imadev
pw_manager: Imamgr
```

第2步：使用ansible-vault命令对文件进行加密，需要每次输入密码比较麻烦，因此还应新建出一个用于保存密码值的文本文件，让ansible-vault进行自动调用。为了保证数据的安全性，新建密码文件后将权限设置为600，仅管理员可读可写：

```
[root@linuxprobe ~]# vim /root/secret.txt
whenyouwishuponastar
[root@linuxprobe ~]# chmod 600 /root/secret.txt
```

在Ansible服务的主配置文件中进行调用，在第140行的“vault_password_file”参数后指定密码值保存的文件路径：

```
[root@linuxprobe ~]# vim /etc/ansible/ansible.cfg
137 
138 # If set, configures the path to the Vault password file as an alternative to
139 # specifying --vault-password-file on the command line.
140 vault_password_file = /root/secret.txt
141 
```

第3步：设置好了密码文件路径，Ansible服务便会自动进行加载，用户不再需要每次加密或解密时都重复输入密码值了。例如加密刚刚创建的locker.yml文件时，只需要使用“encrypt”参数即可：

```
[root@linuxprobe ~]# ansible-vault encrypt locker.yml
Encryption successful
```

加密过后的文件将使用AES 256格式进行加密，也就是意味着2^256就是256位AES密钥空间的组合数2^256>2^(10*25)>10^(3*25)=10^75>>>3×10^23，以天河4号超级计算机为例，每秒进行20亿亿次破解计算，也无法在我们有生之年搞定这串密码值。查看到加密后的内容为：

```
[root@linuxprobe ~]# cat locker.yml 
$ANSIBLE_VAULT;1.1;AES256
38653234313839336138383931663837333533396161343730353530313038313631653439366335
3432346333346239386334663836643432353434373733310a306662303565633762313232663763
38366334316239376262656230643531656665376166663635656436363338626464333430343162
6664643035316133650a333331393538616130656136653630303239663561663237373733373638
62383234303061623865633466336636363961623039343236356336356361613736333739623961
6334303865663838623363333339396637363061626363383266
```

那如果不想用原始密码了呢，也可以手动对文件进行“rekey”修改密码操作，同时应结合“--ask-vault-pass”参数进行修改，否则Ansible服务会因接收不到用户输入的旧密码值，而拒绝新的密码变更请求：

```
[root@linuxprobe ~]# ansible-vault rekey --ask-vault-pass locker.yml 
Vault password: 输入旧的密码
New Vault password: 输入新的密码
Confirm New Vault password: 再输入新的密码
Rekey successful
```

第4步：那如果想查看和修改加密文件中的内容呢。对于已经加密过的文件，需要使用ansible-vault命令的“edit”参数进行修改，随后用“view”参数即可查看到修改后的内容。编辑操作默认使用Vim编辑器作为修改工具，请修改完毕后记得wq保存退出：

```
[root@linuxprobe ~]# ansible-vault edit locker.yml
---
pw_developer: Imadev
pw_manager: Imamgr
pw_production: Imaprod
```

最后，再用“view”参数进行查看，便是最新的内容了：

```
[root@linuxprobe ~]# ansible-vault view locker.yml
Vault password: 输入密码后敲击回车确认
--- 
pw_developer: Imadev 
pw_manager: Imamgr 
pw_production: Imaprod
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．已经搭建好了软件仓库，但却不能用dnf命令安装Ansible服务，大概率是什么原因呢？

**答：**RHEL8系统中默认的BaseOS和AppStream软件仓库中不包括Ansible服务软件包，需要额外配置EPEL安装源。

2．当/etc/ansible/ansible.cfg与~/.ansible.cfg两个主配置文件都同时存在时，以那个为准？

**答：**个人家目录中的优先级更高。

3．使用什么模块能启动服务，又使用什么服务能挂载硬盘设备文件？

**答：**使用service模块可以启动服务，使用mount模块可以挂载设备文件。

4．想了解一个模块的作用，可以使用什么命令查询帮助信息？

**答：**可以使用ansible-doc命令查询模块的帮助信息。

5．Ansible角色有几种获取方法？

**答：三种，分别是加载系统内置的、从外部获取的、创建新的角色。**

6．在执行Playbook剧本文件时，出现黄色标注changed字样，是什么意思？

**答：**代表Play执行成功，并进行了修改。

7．使用ansible-vault命令加密的内容，默认是何种加密方式？

**答：**AES 256 