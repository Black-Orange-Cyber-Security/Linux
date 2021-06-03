# [第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/basic-learning-10.html)

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