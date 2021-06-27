# [第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/basic-learning-19.html)



**章节概述：**

刚入职的运维新手经常会被要求去做一些安装操作系统的工作。如果按照第1章讲解的用光盘镜像来安装操作系统，其效率会相当低下。本章将介绍能够用来实现无人值守安装服务的PXE+Kickstart服务程序，并带领大家动手安装部署PXE + TFTP + FTP + DHCP + Kickstart等服务程序，从而搭建出一套可批量安装[Linux系统](https://www.linuxprobe.com/)的无人值守安装系统。在学完本章内容之后，运维新手就可以避免枯燥乏味的重复性工作，大大提供系统安装的效率。

本章目录结构

- [19.1 无人值守系统](https://www.linuxprobe.com/basic-learning-19.html#191)
- 19.2 部署相关服务程序
  - [19.2.1 配置DHCP服务程序](https://www.linuxprobe.com/basic-learning-19.html#1921_DHCP)
  - [19.2.2 配置TFTP服务程序](https://www.linuxprobe.com/basic-learning-19.html#1922_TFTP)
  - [19.2.3 配置SYSLinux服务程序](https://www.linuxprobe.com/basic-learning-19.html#1923_SYSLinux)
  - [19.2.4 配置VSFtpd服务程序](https://www.linuxprobe.com/basic-learning-19.html#1924_VSFtpd)
  - [19.2.5 创建KickStart应答文件](https://www.linuxprobe.com/basic-learning-19.html#1925_KickStart)
- [19.3 自动部署客户机](https://www.linuxprobe.com/basic-learning-19.html#193)

##### **19.1 无人值守系统**

本书在第1章讲解了使用光盘镜像来安装[Linux](https://www.linuxprobe.com/)系统的方法，坦白讲，该方法适用于只安装少量Linux系统的情况。如果生产环境中有数百台服务器都需要安装系统，这种方式就不合时宜了。这时，我们就需要使用PXE + TFTP +FTP + DHCP + Kickstart服务搭建出一个无人值守安装系统。这种无人值守安装系统可以自动地为数十台、上百台的服务器安装系统，这一方面将运维人员从重复性的工作中解救出来，也大大提升了系统安装的效率。

无人值守安装系统的工作流程如图19-1所示。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2015/10/%E6%97%A0%E4%BA%BA%E5%80%BC%E5%AE%88%E5%AE%89%E8%A3%85%E6%B5%81%E7%A8%8B-1.png)

图19-1 无人值守安装系统的工作流程

PXE（Preboot eXecute Environment，预启动执行环境）是由Intel公司开发的技术，能够让计算机通过网络来启动操作系统（前提是计算机上安装的网卡支持PXE技术），主要用于在无人值守安装系统中引导客户端主机安装Linux操作系统。Kickstart是一种无人值守的安装方式，其工作原理是预先把原本需要运维人员手工填写的参数保存成一个ks.cfg文件，当安装过程中需要填写参数时则自动匹配Kickstart生成的文件。所以只要Kickstart文件包含了安装过程中需要人工填写的所有参数，那么从理论上来讲完全不需要运维人员的干预，就可以自动完成安装工作。TFTP、FTP以及DHCP服务程序的配置与部署已经在第11章和第14章进行了详细讲解，这里不再赘述。

由于当前的客户端主机并没有完整的操作系统，也就不能完成FTP协议的验证了，所以需要使用TFTP协议帮助客户端获取引导及驱动文件。vsftpd服务程序用于将完整的系统安装镜像通过网络传输给客户端。当然，只要能将系统安装镜像成功传输给客户端即可，因此也可以使用httpd来替代vsftpd服务程序。

##### **19.2 部署相关服务程序**

作为总结性的章节，我们接下来会部署多款以前学习过的服务，其作用如表19-1所示。对应的配置过程会比前面单独学习时节奏更快，如有需要读者们可返回到前面的章节再查看下详细讲解，但[刘遄](https://www.linuxprobe.com/)老师相信是没有这个必要啦~

表19-1                    接下来实验中即将用到的服务及作用

| 服务名称    | 主要作用                       |
| ----------- | ------------------------------ |
| dhcpd       | 分配网卡信息及指引获取驱动文件 |
| tftp-server | 提供驱动及引导文件的传输       |
| SYSLinux    | 提供驱动及引导文件             |
| VSFtpd      | 提供完整系统镜像的传输         |
| KickStart   | 提供安装过程中选项的问答设置   |



###### **19.2.1 配置DHCP服务程序**

DHCP服务程序用于为客户端主机分配可用的IP地址，而且这是服务器与客户端主机进行文件传输的基础，因此要先行配置DHCP服务程序。首先按照表19-2为无人值守系统设置IP地址，然后按照图19-2和图19-3在虚拟机的虚拟网络编辑器中关闭自身的DHCP服务，避免与自己配置的服务冲突。

表19-2                     无人值守系统与客户端的设置

| 主机名称     | 操作系统       | IP地址        |
| ------------ | -------------- | ------------- |
| 无人值守系统 | RHEL 8         | 192.168.10.10 |
| 客户端       | 未安装操作系统 | -             |



![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E6%89%93%E5%BC%80%E8%99%9A%E6%8B%9F%E6%9C%BA%E7%9A%84%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C%E7%BC%96%E8%BE%91%E5%99%A8-1024x153.png)

图19-2 打开虚拟机的虚拟网络编辑器

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E5%85%B3%E9%97%AD%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%87%AA%E5%B8%A6%E7%9A%84DHCP%E6%9C%8D%E5%8A%A1.png)

图19-3 关闭虚拟机自带的DHCP服务

由于PXE+KickStart无人值守安装系统需要涉及的服务不限于下面所提及的，还需要允许如sips、slp、mountd等多项相关的服务，因此本实验会临时关闭firewalld防火墙，以让数据能够正常的传送：

```
[root@linuxprobe pub]# iptables -F
[root@linuxprobe pub]# systemctl stop firewalld
```

当挂载好光盘镜像并把仓库文件配置妥当后，就可以安装DHCP服务程序软件包了：

```
[root@linuxprobe ~]# dnf install -y dhcp-server
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:22 ago on Fri 30 Apr 2021 01:03:26 AM CST.
Dependencies resolved.
================================================================================
 Package            Arch          Version                   Repository     Size
================================================================================
Installing:
 dhcp-server        x86_64        12:4.3.6-30.el8           BaseOS        529 k

Transaction Summary
================================================================================
Install  1 Package
………………省略部分输出信息………………
Installed:
  dhcp-server-12:4.3.6-30.el8.x86_64                                                                                                        

Complete!
```

第14章已经详细讲解了DHCP服务程序的配置以及部署方法，相信各位读者对相关的配置参数还有一些印象。但是，我们在这里使用的配置文件与第14章中的配置文件有两个主要区别：允许了BOOTP引导程序协议，旨在让局域网内暂时没有操作系统的主机也能获取静态IP地址；在配置文件的最下面加载了引导驱动文件pxelinux.0（这个文件会在下面的步骤中创建），其目的是让客户端主机获取到IP地址后主动获取引导驱动文件，自行进入下一步的安装过程。

```
[root@linuxprobe ~]# vim /etc/dhcp/dhcpd.conf
allow booting;
allow bootp;
ddns-update-style none;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
        option subnet-mask                 255.255.255.0;
        option domain-name-servers         192.168.10.10;
        range dynamic-bootp 192.168.10.100 192.168.10.200;
        default-lease-time                 21600;
        max-lease-time                     43200;
        next-server                        192.168.10.10;
        filename                           "pxelinux.0";
}
```

### **Tips**

当前pxelinux.0文件不存在，不用担心，后面会找到它的~

在确认DHCP服务程序的参数都填写正确后，重新启动该服务程序，并将其添加到开机启动项中。这样在设备下一次重启之后，在无须人工干预的情况下，自动为客户端主机安装系统。

```
[root@linuxprobe ~]# systemctl restart dhcpd
[root@linuxprobe ~]# systemctl enable  dhcpd
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
```

与以往的[红帽](https://www.linuxprobe.com/)企业版不同，RHEL 8系统中存在一些“讨厌”的服务，它们的参数错误导致服务启动失败，有时却不会在屏幕显示出给用户的提示信息。建议在启动dhcpd后查看下服务状态，避免后续实验中客户端分配不到网卡信息，输出状态为“**active (running)** ”则代表服务已经正常运行：

```
[root@linuxprobe ~]# systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
   Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2021-04-30 01:10:51 CST; 3min 15s ago
     Docs: man:dhcpd(8)
           man:dhcpd.conf(5)
 Main PID: 30964 (dhcpd)
   Status: "Dispatching packets..."
    Tasks: 1 (limit: 12390)
   Memory: 8.8M
   CGroup: /system.slice/dhcpd.service
           └─30964 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid
………………省略部分输出信息………………
```

###### **19.2.2 配置TFTP服务程序**

我们曾经在第11章中学习过vsftpd服务与TFTP服务。vsftpd是一款功能丰富的文件传输服务程序，允许用户以匿名开放模式、本地用户模式、虚拟用户模式来进行访问认证。但是，当前的客户端主机还没有安装操作系统，该如何进行登录认证呢？而TFTP作为一种基于UDP协议的简单文件传输协议，不需要进行用户认证即可获取到所需的文件资源。因此接下来配置TFTP服务程序，为客户端主机提供引导及驱动文件。当客户端主机有了基本的驱动程序之后，再通过vsftpd服务程序将完整的光盘镜像文件传输过去。

```
[root@linuxprobe ~]# dnf install -y tftp-server xinetd
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:34 ago on Fri 30 Apr 2021 01:38:09 AM CST.
Dependencies resolved.
================================================================================
 Package           Arch      Version             Repository          Size
================================================================================
Installing:
 tftp-server       x86_64    5.2-24.el8          AppStream           50 k
 xinetd            x86_64    2:2.3.15-23.el8     AppStream          135 k

Transaction Summary
================================================================================
Install  2 Packages
………………省略部分输出信息………………

Installed:
  tftp-server-5.2-24.el8.x86_64                xinetd-2:2.3.15-23.el8.x86_64               

Complete!
```

TFTP是一种非常精简的文件传输服务程序，它的运行和关闭是由xinetd网络守护进程服务来管理的。xinetd服务程序会同时监听系统的多个端口，然后根据用户请求的端口号调取相应的服务程序来响应用户的请求。需要开启TFTP服务程序，只需在xinetd服务程序的配置文件中把disable参数改成no就可以了，如果配置文件不存则复制下面内容：

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

保存配置文件并退出，然后重启xinetd服务程序，并将其加入到开机启动项中。

```
[root@linuxprobe ~]# systemctl restart xinetd
[root@linuxprobe ~]# systemctl enable  xinetd
```

###### **19.2.3 配置SYSLinux服务程序**

SYSLinux是一个用于提供引导加载的服务程序。与其说SYSLinux是一个服务程序，不如说更需要里面的引导文件，在安装好SYSLinux服务程序软件包后，/usr/share/syslinux目录中会出现很多引导文件。

```
[root@linuxprobe ~]# dnf install -y syslinux
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:07:57 ago on Fri 30 Apr 2021 01:47:18 AM CST.
Dependencies resolved.
================================================================================
 Package                     Arch       Version         Repository     Size
================================================================================
Installing:
 syslinux                    x86_64     6.04-1.el8      BaseOS        576 k
Installing dependencies:
 syslinux-nonlinux           noarch     6.04-1.el8      BaseOS        554 k

Transaction Summary
================================================================================
Install  2 Packages
………………省略部分输出信息………………

Installed:
  syslinux-6.04-1.el8.x86_64              syslinux-nonlinux-6.04-1.el8.noarch             

Complete!
```

我们首先需要把SYSLinux提供的引导文件复制到TFTP服务程序的默认目录中，也就是前文提到的文件pxelinux.0，这样客户端主机就能够顺利地获取到引导文件了。另外在RHEL 8系统光盘镜像中也有一些需要调取的引导文件。确认光盘镜像已经被挂载到/media/cdrom目录后，使用复制[命令](https://www.linuxcool.com/)将光盘镜像中自带的一些引导文件也复制到TFTP服务程序的默认目录中。

```
[root@linuxprobe ~]# cd /var/lib/tftpboot
[root@linuxprobe tftpboot]# cp /usr/share/syslinux/pxelinux.0 .
[root@linuxprobe tftpboot]# cp /media/cdrom/images/pxeboot/* .
[root@linuxprobe tftpboot]# cp /media/cdrom/isolinux/* .
cp: overwrite './initrd.img'? y
cp: overwrite './TRANS.TBL'? y
cp: overwrite './vmlinuz'? y
```

cp[命令](https://www.linuxcool.com/)后面接的句号（.）代表当前工作目录，也就是复制到当前所在/var/lib/tftpboot的作用。在复制过程中若出现多个目录保存有相同的文件情况时，则可手动敲击y进行覆盖即可。

然后在TFTP服务程序的目录中新建pxelinux.cfg目录，虽然该目录的名字带有后缀，但依然也是目录，而非文件！将系统光盘中的开机选项菜单复制到该目录中，并命名为default。这个default文件就是开机时的选项菜单，如图19-4所示。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2020/05/RHEL-8%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E7%95%8C%E9%9D%A2.png)

图19-4 Linux系统的引导菜单界面

```
[root@linuxprobe tftpboot]# mkdir pxelinux.cfg
[root@linuxprobe tftpboot]# cp /media/cdrom/isolinux/isolinux.cfg pxelinux.cfg/default
```

默认的开机菜单中有三个选项，要么是安装系统，要么是对安装介质进行检验，那么是Troubleshooting排错模式。既然已经确定采用无人值守的方式安装系统，还需要为每台主机手动选择相应的选项，未免与我们的主旨（无人值守安装）相悖。

现在我们编辑这个default文件，把第1行的default参数修改为linux，这样系统在开机时就会默认执行那个名称为linux的选项了。对应的linux选项大约在64行，将默认的光盘镜像安装方式修改成FTP文件传输方式，并指定好光盘镜像的获取网址以及Kickstart应答文件的获取路径：

```
[root@linuxprobe tftpboot]# vim pxelinux.cfg/default
  1 default linux
  2 timeout 600
  3 
  4 display boot.msg
  5 
  6 # Clear the screen when exiting the menu, instead of leaving the menu displayed.
  7 # For vesamenu, this means the graphical background is still displayed without
  8 # the menu itself for as long as the screen remains in graphics mode.
  9 menu clear
 10 menu background splash.png
 11 menu title Red Hat Enterprise Linux 8.0.0
 12 menu vshift 8
 13 menu rows 18
 14 menu margin 8
 15 #menu hidden
 16 menu helpmsgrow 15
 17 menu tabmsgrow 13
 18 
 19 # Border Area
 20 menu color border * #00000000 #00000000 none
 21 
 22 # Selected item
 23 menu color sel 0 #ffffffff #00000000 none
 24 
 25 # Title bar
 26 menu color title 0 #ff7ba3d0 #00000000 none
 27 
 28 # Press [Tab] message
 29 menu color tabmsg 0 #ff3a6496 #00000000 none
 30 
 31 # Unselected menu item
 32 menu color unsel 0 #84b8ffff #00000000 none
 33 
 34 # Selected hotkey
 35 menu color hotsel 0 #84b8ffff #00000000 none
 36 
 37 # Unselected hotkey
 38 menu color hotkey 0 #ffffffff #00000000 none
 39 
 40 # Help text
 41 menu color help 0 #ffffffff #00000000 none
 42 
 43 # A scrollbar of some type? Not sure.
 44 menu color scrollbar 0 #ffffffff #ff355594 none
 45 
 46 # Timeout msg
 47 menu color timeout 0 #ffffffff #00000000 none
 48 menu color timeout_msg 0 #ffffffff #00000000 none
 49 
 50 # Command prompt text
 51 menu color cmdmark 0 #84b8ffff #00000000 none
 52 menu color cmdline 0 #ffffffff #00000000 none
 53 
 54 # Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.
 55 
 56 menu tabmsg Press Tab for full configuration options on menu items.
 57 
 58 menu separator # insert an empty line
 59 menu separator # insert an empty line
 60 
 61 label linux
 62   menu label ^Install Red Hat Enterprise Linux 8.0.0
 63   kernel vmlinuz
 64   append initrd=initrd.img inst.stage2=ftp://192.168.10.10 ks=ftp://192.168.10.10/pub/ks.cfg quiet
 65 ………………省略部分输出信息………………
```

建议在安装源的后面加入quiet参数，意为使用静默安装方式，不需要用户再进行确认。文件修改完毕后保存即可，开机选项菜单是被调用的文件，因此不需要单独重启任何的服务。

###### **19.2.4 配置VSFtpd服务程序**

在这套无人值守安装系统的服务中，光盘镜像是通过FTP协议传输的，因此势必要用到vsftpd服务程序。当然，也可以使用httpd服务程序来提供Web网站访问的方式，只要能确保将光盘镜像顺利传输给客户端主机即可。如果打算使用Web网站服务来提供光盘镜像，一定记得将上面配置文件中的光盘镜像获取网址和Kickstart应答文件获取网址修改一下。

```
[root@linuxprobe tftpboot]# dnf install -y vsftpd
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:28:28 ago on Fri 30 Apr 2021 01:47:18 AM CST.
Dependencies resolved.
===============================================================================
 Package       Arch       Version           Repository        Size
===============================================================================
Installing:
 vsftpd        x86_64     3.0.3-28.el8      AppStream         180 k

Transaction Summary
===============================================================================
Install  1 Package
………………省略部分输出信息………………

Installed:
  vsftpd-3.0.3-28.el8.x86_64                                                                     

Complete!
```

RHEL 8系统版本的vsftpd服务默认不允许匿名公开访问模式，因此需要手动进行开启：

```
[root@linuxprobe ~ ]# vim /etc/vsftpd/vsftpd.conf
# Example config file /etc/vsftpd/vsftpd.conf
#
# The default compiled in settings are fairly paranoid. This sample file
# loosens things up a bit, to make the ftp daemon more usable.
# Please see vsftpd.conf.5 for all compiled in defaults.
#
# READ THIS: This example file is NOT an exhaustive list of vsftpd options.
# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's
# capabilities.
#
# Allow anonymous FTP? (Beware - allowed by default if you comment this out).
anonymous_enable=YES
………………省略部分输出信息………………
```

[刘遄](https://www.linuxprobe.com/)老师再啰嗦一句，在配置文件修改正确之后，一定将相应的服务程序添加到开机启动项中，这样无论是在生产环境中还是在[红帽](https://www.linuxprobe.com/)认证考试中，都可以在设备重启之后依然能提供相应的服务。希望各位读者一定养成这个好习惯。

```
[root@linuxprobe ~]# systemctl restart vsftpd
[root@linuxprobe ~]# systemctl enable  vsftpd
Created symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

在确认系统光盘镜像已经正常挂载到/media/cdrom目录后，把目录中的光盘镜像文件全部复制到vsftpd服务程序的工作目录中。

```
[root@linuxprobe ~]# cp -r /media/cdrom/* /var/ftp
```

这个过程大约需要3～5分钟。在此期间，咱们也别闲着，在SELinux中放行FTP传输连接的策略：

```
[root@linuxprobe ~]# setsebool -P ftpd_connect_all_unreserved=on
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

###### **19.2.5 创建KickStart应答文件**

毕竟，我们使用PXE + Kickstart部署的是一套“无人值守安装系统服务”，而不是“无人值守传输系统光盘镜像服务”，因此还需要让客户端主机能够一边获取光盘镜像，还能够一边自动帮用户填写好安装过程中出现的选项。简单来说，如果生产环境中有100台服务器，它们需要安装相同的系统环境，那么在安装过程中单击的按钮和填写的信息也应该都是相同的。那么，为什么不创建一个类似于备忘录的需求清单呢？这样，在无人值守安装系统时，会从这个需求清单中找到相应的选项值，从而免去了手动输入之苦，更重要的是，也彻底解放了人的干预，彻底实现无人值守自动安装系统，而不是单纯地传输系统光盘镜像。

有了上文做铺垫，相信大家现在应该可以猜到Kickstart其实并不是一个服务程序，而是一个应答文件了。是的！Kickstart应答文件中包含了系统安装过程中需要使用的选项和参数信息，系统可以自动调取这个应答文件的内容，从而彻底实现了无人值守安装系统。那么，既然这个文件如此重要，该去哪里找呢？其实在root管理员的家目录中有一个名为anaconda-ks.cfg的文件，它就是应答文件。下面将这个文件复制到vsftpd服务程序的工作目录中（在开机选项菜单的配置文件中已经定义了该文件的获取路径，也就是vsftpd服务程序数据目录中的pub子目录中）。使用chmod命令设置该文件的权限，确保所有人都有可读的权限，以保证客户端主机顺利获取到应答文件及里面的内容：

```
[root@linuxprobe ~]# cp ~/anaconda-ks.cfg /var/ftp/pub/ks.cfg
[root@linuxprobe ~]# chmod +r /var/ftp/pub/ks.cfg
```

Kickstart应答文件并没有想象中的那么复杂，它总共只有44行左右的参数和注释内容，大家完全可以通过参数的名称及介绍来快速了解每个参数的作用。刘遄老师在这里挑选几个比较有代表性的参数进行讲解，其他参数建议大家自行修改测试。

其中第1至10行，代表安装硬盘名称为sda及使用LVM逻辑卷管理期技术。这便要求我们在后续新建客户端虚拟机时，硬盘一定要选择SCSI或SATA，如图19-5所示，否则会变成/dev/hd或/dev/nvme开头的名称，进而会因找不到硬盘设备而终止安装进程。

而第8行的软件仓库，应该为由FTP服务器提供的网络路径。第10行的安装源，也需要由CDROM光盘改为网络安装源：

```
  1 #version=RHEL8
  2 ignoredisk --only-use=sda
  3 autopart --type=lvm
  4 # Partition clearing information
  5 clearpart --none --initlabel
  6 # Use graphical install
  7 graphical
  8 repo --name="AppStream" --baseurl=ftp://192.168.10.10/AppStream
  9 # Use CDROM installation media
 10 url --url=ftp://192.168.10.10/BaseOS
```

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E5%A4%87%E7%B1%BB%E5%9E%8B.png)

图19-5 选择SCSI或SATA硬盘类型

其中第11至20行中，keyboard参数为硬盘类型，一般都不需要修改。但第17行的网卡信息万一要注意，一定要让网卡默认处于DHCP的模式，否则几十、上百台主机同时被创建出来，IP地址会相互冲突导致后续无法管理。

```
 11 # Keyboard layouts
 12 keyboard --vckeymap=us --xlayouts='us'
 13 # System language
 14 lang en_US.UTF-8
 15 
 16 # Network information
 17 network  --bootproto=dhcp --device=ens160 --onboot=on --ipv6=auto --activate
 18 network  --hostname=linuxprobe.com
 19 # Root password
 20 rootpw --iscrypted $6$EzIFyouUyBvWRIXv$y3bW3JZ2vD4c8bwVyKt7J90gyjULALTMLrnZZmvVujA75EpCCn50rlYm64MHAInbMAXAgn2Bmlgou/pYjUZzL1
```

其中第21行至30行中timezone参数定义了系统默认市区为上海，如果同学们服务器时间存在不准确的情况，则如下修改即可。第29行创建了一个普通用户，密码值可复制/etc/shadow文件中的加密密文，它会由系统自动创建出来。

```
 21 # X Window System configuration information
 22 xconfig  --startxonboot
 23 # Run the Setup Agent on first boot
 24 firstboot --enable
 25 # System services
 26 services --disabled="chronyd"
 27 # System timezone
 28 timezone Asia/Shanghai --isUtc --nontp
 29 user --name=linuxprobe --password=$6$a5fEjghDXGPvEoQc$HQqzvBlGVyhsJjgKFDTpiCEavS.inAwNTLZm/I5R5ALLKaMdtxZoKgb4/EaDyiPSSNNHGqrEkRnfJWap56m./. --iscrypted --gecos="linuxprobe"
 30 
```

最后的第31行至44行代表了要安装的软件来源，“graphical-server-environment”即带有图形化界面的服务器环境，对照安装界面中的“Server With GUI”选项。

```
 31 %packages
 32 @^graphical-server-environment
 33 
 34 %end
 35 
 36 %addon com_redhat_kdump --disable --reserve-mb='auto'
 37 
 38 %end
 39 
 40 %anaconda
 41 pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
 42 pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
 43 pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
 44 %end
```

所以实际算下来修改的并不多，默认参数就已经非常合适了。最后预览一下ks.cfg文件的全貌，读者如果在生产环境中需要使用，可复制以下内容就能直接使用：

```
[root@linuxprobe ~ ]# cat /var/ftp/pub/ks.cfg
#version=RHEL8
ignoredisk --only-use=sda
autopart --type=lvm
# Partition clearing information
clearpart --none --initlabel
# Use graphical install
graphical
repo --name="AppStream" --baseurl=ftp://192.168.10.10/AppStream
# Use CDROM installation media
url --url=ftp://192.168.10.10/BaseOS
# Keyboard layouts
keyboard --vckeymap=us --xlayouts='us'
# System language
lang en_US.UTF-8

# Network information
network  --bootproto=dhcp --device=ens160 --onboot=on --ipv6=auto --activate
network  --hostname=linuxprobe.com
# Root password
rootpw --iscrypted $6$EzIFyouUyBvWRIXv$y3bW3JZ2vD4c8bwVyKt7J90gyjULALTMLrnZZmvVujA75EpCCn50rlYm64MHAInbMAXAgn2Bmlgou/pYjUZzL1
# X Window System configuration information
xconfig  --startxonboot
# Run the Setup Agent on first boot
firstboot --enable
# System services
services --disabled="chronyd"
# System timezone
timezone Asia/Shanghai --isUtc --nontp
user --name=linuxprobe --password=$6$a5fEjghDXGPvEoQc$HQqzvBlGVyhsJjgKFDTpiCEavS.inAwNTLZm/I5R5ALLKaMdtxZoKgb4/EaDyiPSSNNHGqrEkRnfJWap56m./. --iscrypted --gecos="linuxprobe"

%packages
@^graphical-server-environment

%end

%addon com_redhat_kdump --disable --reserve-mb='auto'

%end

%anaconda
pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty
pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok
pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty
%end
                                                                                          
```

KickStart应答文件会由FTP服务进行传输，然后安装向导进行调用，因此也不需要重启任何的服务。

##### **19.3 自动部署客户机**

在按照上文讲解的方法成功部署各个相关的服务程序后，就可以使用PXE + Kickstart无人值守安装系统了。在采用下面的步骤建立虚拟主机时，一定要把客户端的网卡模式设定成与服务端一致的“仅主机模式”，否则两台设备无法进行通信，也就更别提自动安装系统了。其余硬件配置选项并没有强制性要求，大家可参考这里的配置选项来设定。

**第1步**：打开“新建虚拟机向导”程序，选择“自定义（高级） ”配置类型，然后单击“下一步”按钮，如图19-6所示。随后的虚拟机硬件兼容性选择默认的“Workstation 16.x”，步骤省略。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E5%88%9B%E5%BB%BA%E8%99%9A%E6%8B%9F%E6%9C%BA.png)

图19-6  选择虚拟机的配置类型

**第2步**：将虚拟机操作系统的安装来源设置为“稍后安装操作系统”。这样做的目的是让虚拟机真正从网络中获取系统安装镜像，同时也可避免VMware Workstation虚拟机软件按照内设的方法自行安装系统。单击“下一步”按钮，如图19-7所示。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E7%A8%8D%E5%90%8E%E5%AE%89%E8%A3%85%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F.png)

图19-7 设置虚拟机操作系统的安装来源

**第3步**：将“客户机操作系统”设置为“Red Hat Enterprise Linux 8 64位”，然后单击“下一步”按钮，如图19-8所示。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E7%BD%AE%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F%E7%9A%84%E7%89%88%E6%9C%AC.png)

图19-8 选择客户端主机的操作系统

**第4步**：对虚拟机进行命名并设置安装位置。大家可自行定义虚拟机的名称，而安装位置则尽量选择磁盘空间较大的分区。然后单击“下一步”按钮，如图19-9所示。随后的设置虚拟机处理器个数及核心数、内存容量值，读者请自行根据实际情况进行选择，步骤省略。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E7%BD%AE%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%90%8D%E7%A7%B0.png)

图19-9 命名虚拟机并设置虚拟机的安装位置

**第5步**：设置虚拟机主机的网络连接类型为“仅主机模式网络”，一定要确保服务器与客户端同处于相同网络模式，如图19-10所示，否则客户端无法获得从服务器传送过来的系统镜像及应答文件。随后的SCSI控制器类型选择默认“LSI Logic”，步骤省略。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E7%BD%AE%E7%BD%91%E7%BB%9C%E8%BF%9E%E6%8E%A5%E6%A8%A1%E5%BC%8F.png)

图19-10 设置客户端的网络模式

**第6步**：设置硬盘类型并指定容量。设置虚拟硬盘类型为SCSI或SATA，如图19-11所示。随后在硬盘创建确认界面，选择“创建新虚拟磁盘”选项，步骤省略。

这里将“最大磁盘大小”设置为20GB，指的是虚拟机系统能够使用的最大上限，而不是会被立即占满，因此设置得稍微大一些也没有关系。然后单击“下一步”按钮，如图19-12所示。随后的确认硬盘文件名称界面，选择默认值即可，步骤省略。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E7%BD%AE%E7%A1%AC%E7%9B%98%E7%B1%BB%E5%9E%8B.png)

图19-11 设置虚拟硬盘类型为SCSI或SATA

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E7%BD%AE%E8%99%9A%E6%8B%9F%E7%A1%AC%E7%9B%98%E5%AE%B9%E9%87%8F.png)

图19-12 将磁盘容量指定为20GB

**第7步**：结束“新建虚拟机向导程序”后，先不要着急打开虚拟机系统。大家还需要单击图19-13中的“自定义硬件”按钮，在弹出的如图19-14所示的界面中，把“网络适配器”设备同样也设置为“仅主机模式”（这个步骤非常重要），移除其它不需要的硬件，然后单击“确定”按钮。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%87%AA%E5%AE%9A%E4%B9%89%E7%A1%AC%E4%BB%B6.png)

图19-13 单击虚拟机的“自定义硬件”按钮

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E7%A1%AC%E4%BB%B6%E9%85%8D%E7%BD%AE%E4%B8%80%E8%A7%88.png)

图19-14 设置虚拟机网络适配器设备为仅主机模式

现在，我们就同时准备好了PXE + Kickstart无人值守安装系统与虚拟主机。在生产环境中，大家只需要将配置妥当的服务器上架，接通服务器和客户端主机之间的网线，然后启动客户端主机即可。接下来就会按照图19-15、19-16和图19-17那样，开始传输光盘镜像文件并进行自动安装了——期间完全无须人工干预，直到安装完毕时才需要运维人员进行简单的初始化工作。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/1-22-1024x704.png)

图19-15 自动传输光盘镜像文件并安装系统

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/2-18-1024x705.png)

图19-16 根据应答文件自动填写安装信息

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/3-7-1024x704.png)

图19-17 自动安装系统，无须人工干预

由此可见，当生产环境工作中有数百台服务器需要批量安装系统时，使用无人值守安装系统的便捷性是不言而喻的。但为了避免法律风险，红帽公司对于许可界面还不允许用应答文件自动完成，需要人工点击“I accept the license agreement（我接受这份许可协议）”后方可继续安装，如图19-18和图19-19所示，我们通过网络安装的客户端就搞定啦~

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/4-3-1024x704.png)

图19-18 手动点击接受许可协议的按钮
![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/5-3-1024x704.png)

图19-19 顺利进入到新系统中

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．部署无人值守安装系统，需要用到哪些服务程序？

**答：**需要用到SYSLinux引导服务、DHCP服务、vsftpd文件传输服务（或httpd网站服务）、TFTP服务以及KickStart应答文件。

2．在Vmware Workstation虚拟机软件中，DHCP服务总是分配错误IP地址的原因可能是什么？

**答：**虚拟机的虚拟网络编辑器中自带的DHCP服务可能没有关闭，由此产生了错误分配IP地址的情况。

3．如何启用TFTP服务？

**答：**需要在xinetd服务程序的配置文件中把disable参数改成no。

4．成功安装SYSLinux服务程序后，可以在哪个目录中找到引导文件？

**答：**在安装好SYSLinux服务程序软件包后，在/usr/share/syslinux目录中会出现很多引导文件。

5．在开机选项菜单文件中，把default参数设置成linux的作用是什么？

**答：**目的是让系统自动开始安装过程，而不需要运维人员再去选择是安装系统还是校验镜像文件。

6．安装vsftpd文件传输服务或httpd网站服务的作用是什么？

**答：**把光盘镜像文件完整、顺利地传送到客户端主机。

7．Kickstart应答文件的作用是什么？

**答：**Kickstart应答文件中包含了系统安装过程中需要使用的选项和参数信息，客户端主机在安装系统的过程中可以自动调取这个应答文件的内容，从而彻底实现无人值守安装系统。