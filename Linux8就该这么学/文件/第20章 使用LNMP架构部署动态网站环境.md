# [第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/basic-learning-20.html)

**章节概述：**

LNMP动态网站部署架构是一套由[Linux](https://www.linuxprobe.com/) + Nginx + MySQL + PHP组成的动态网站系统解决方案，具有免费、高效、扩展性强且资源消耗低等优良特性，目前正在被广泛使用。本章首先对比了使用源码包安装服务程序与使用RPM软件包安装服务程序的区别，然后讲解了如何手工编译源码包并安装各个服务程序，以及使用最受欢迎的WordPress博客系统验证架构环境。

本章是本书的最后一章内容，[刘遄](https://www.linuxprobe.com/)老师不仅希望各位读者在学完本书之后，能够顺利找到满意的高薪工作，也希望您能利用书中所学知识搭建自己的博客或论坛系统，并以此为平台，将自己工作中积攒的Linux经验以及技巧分享给更多人，为美好的开源世界贡献自己的力量。**See you later!**

本章目录结构

- [20.1 源码包程序](https://www.linuxprobe.com/basic-learning-20.html#201)
- 20.2 LNMP动态网站架构
  - [20.2.1 配置Nginx服务](https://www.linuxprobe.com/basic-learning-20.html#2021_Nginx)
  - [20.2.2 配置Mysql服务](https://www.linuxprobe.com/basic-learning-20.html#2022_Mysql)
  - [20.2.3 配置php服务](https://www.linuxprobe.com/basic-learning-20.html#2023_php)
- [20.3 搭建Discuz论坛](https://www.linuxprobe.com/basic-learning-20.html#203_Discuz)
- [20.4 选购服务器主机](https://www.linuxprobe.com/basic-learning-20.html#204)

##### **20.1 源码包程序**

本书第1章中曾经讲到，在RPM[红帽](https://www.linuxprobe.com/)软件包管理器技术出现之前，[Linux系统](https://www.linuxprobe.com/)运维人员只能通过源码包的方式来安装各种服务程序，这是一件非常繁琐且极易消耗时间与耐心的事情；而且在安装、升级、卸载程序时还要考虑到与其他程序或函数库的相互依赖关系，这就要求运维人员不仅要掌握更多的Linux系统理论知识以及高超的实操技能，还需要有极好的耐心才能安装好一个源码软件包。考虑到本书的读者都是刚入门或准备入门的运维新人，因为本书在前面的章节中一直都是采用软件仓库的方式来安装服务程序。但是，现在依然有很多软件程序只有源码包的形式，如果我们只会使用dnf[命令](https://www.linuxcool.com/)来安装程序，则面对这些只有源码包的软件程序时，将充满无力感，要么需要等到第三方组织将这些软件程序编写成RPM软件包之后再行使用，要么就只能寻找相关软件程序的替代品了（而且替代软件还必须具备RPM软件包的形式）。由此可见，如果运维人员只会使用软件仓库来安装服务程序，将会形成知识短板，对日后的运维工作带来不利。

本着不能让自己的读者在运维工作中吃亏的想法，[刘遄](https://www.linuxprobe.com/)老师接下来会详细讲解如何使用源码包的方式来安装服务程序。

其实，使用源码包来安装服务程序具有两个优势。

> 源码包的可移植性非常好，几乎可以在任何Linux系统中安装使用，而RPM软件包是针对特定系统和架构编写的指令集，必须严格地符合执行环境才能顺利安装（即只会去“生硬地”安装服务程序）。
>
> 使用源码包安装服务程序时会有一个编译过程，因此能够更好地适应安装主机的系统环境，运行效率和优化程度都会强于使用RPM软件包安装的服务程序。也就是说，可以将采用源码包安装服务程序的方式看作是针对系统的“量体裁衣”。

一般来讲，在安装软件时，如果能通过软件仓库来安装，就用dnf[命令](https://www.linuxcool.com/)搞定它；反之则去寻找合适的RPM软件包来安装；如果是在没有资源可用，那就只能使用源码包来安装了。使用源码包安装服务程序的过程看似复杂，其实在归纳汇总后只需要4～5个步骤即可完成安装。接下来会对每一个步骤进行详解。

需要提前说明的是，在使用源码包安装程序时，会输出大量的过程信息，这些信息的意义并不大，因此本章会省略这部分输出信息而不作特殊备注，请大家在具体操作时以实际为准。

**第1步**：下载及解压源码包文件。为了方便在网络中传输，源码包文件通常会在归档后使用gzip或bzip2等格式进行压缩，因此一般会具有.tar.gz与.tar.bz2的后缀。要想使用源码包安装服务程序，必须先把里面的内容解压出来，然后再切换到源码包文件的目录中：

> [root@linuxprobe ~]# tar xzvf FileName**.tar.gz**
>
> [root@linuxprobe ~]# cd FileDirectory

**第2步**：编译源码包代码。在正式使用源码包安装服务程序之前，还需要使用编译[脚本](https://www.linuxcool.com/)针对当前系统进行一系列的评估工作，包括对源码包文件、软件之间及函数库之间的依赖关系、编译器、汇编器及连接器进行检查。我们还可以根据需要来追加--prefix参数，以指定稍后源码包程序的安装路径，从而对服务程序的安装过程更加可控。当编译工作结束后，如果系统环境符合安装要求，一般会自动在当前目录下生成一个Makefile安装文件。

> [root@linuxprobe ~]# ./configure --prefix=/usr/local/program

**第3步**：生成二进制安装程序。刚刚生成的Makefile文件中会保存有关系统环境、软件依赖关系和安装规则等内容，接下来便可以使用make命令来根据Makefile文件内容提供的合适规则编译生成出真正可供用户安装服务程序的二进制可执行文件了。

> [root@linuxprobe ~]# make

**第4步**：运行二进制的服务程序安装包。由于不需要再检查系统环境，也不需要再编译代码，因此运行二进制的服务程序安装包应该是速度最快的步骤。如果在源码包编译阶段使用了--prefix参数，那么此时服务程序就会被安装到那个目录，如果没有自行使用参数定义目录的话，一般会被默认安装到/usr/local/bin目录中。

> [root@linuxprobe ~]# make install

**第5步**：清理源码包临时文件。由于在安装服务程序的过程中进行了代码编译的工作，因此在安装后目录中会遗留下很多临时垃圾文件，本着尽量不要浪费磁盘存储空间的原则，可以使用make clean命令对临时文件进行彻底的清理工作。

> [root@linuxprobe ~]# make clean

估计有读者会有疑问，为什么通常是安装一个服务程序，源码包的编译工作（configure）与生成二进制文件的工作（make）会使用这么长的时间，而采用RPM软件包安装就特别有效率呢？其实原因很简单，在RHCA认证的RH401考试中，会要求考生写一个RPM软件包。刘遄老师会在本书的进阶篇中讲到，其实RPM软件包就是把软件的源码包和一个针对特定系统、架构、环境编写的安装规定打包成一起的指令集，因此为了让用户都能使用这个软件包来安装程序，通常一个软件程序会发布多种格式的RPM软件包（例如i386、x86_64等架构）来让用户选择。而源码包的软件作者肯定希望自己的软件能够被安装到更多的系统上面，能够被更多的用户所了解、使用，因此便会在编译阶段（configure）来检查用户当前系统的情况，然后制定出一份可行的安装方案，所以会占用很多的系统资源，需要更长的等待时间。

##### **20.2 LNMP动态网站架构**

LNMP动态网站部署架构是一套由Linux + Nginx + MySQL + PHP组成的动态网站系统解决方案。LNMP中的字母L是Linux系统的意思，不仅可以是RHEL、CentOS、Fedora，还可以是Debian、Ubuntu等系统。本书的配套站点[https://www.linuxprobe.com](https://www.linuxprobe.com/)就是基于LNMP部署出来的，目前的运行一直很稳定，访问速度也很快。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2015/11/lnmp%E6%9E%B6%E6%9E%84%E5%9B%BE%E7%89%87.jpg)

在使用源码包安装服务程序之前，首先要让安装主机具备编译程序源码的环境，他需要具备C语言、C++语言、Perl语言的编译器，以及各种常见的编译支持函数库程序。因此请先配置妥当软件仓库，然后把下面列出的这些软件包都统统安装上：

```
[root@linuxprobe ~]# dnf -y install apr* autoconf automake numactl bison bzip2-devel cpp curl-devel fontconfig-devel freetype-devel gcc gcc-c++ gd-devel gettext-devel kernel-headers keyutils-libs-devel krb5-devel libcom_err-devel  libpng-devel  libjpeg* libsepol-devel libselinux-devel libstdc++-devel libtool* libxml2-devel libXpm* libxml* libXaw-devel libXmu-devel libtiff* make openssl-devel patch pcre-devel perl php-common php-gd telnet zlib-devel libtirpc-devel gtk* ntpstat na* bison* lrzsz cmake ncurses-devel libzip-devel libxslt-devel gdbm-devel readline-devel gmp-devel

Updating Subscription Management repositories.
Unable to read consumer identity
AppStream                                       3.1 MB/s | 3.2 kB     00:00    
BaseOS                                          2.0 MB/s | 2.7 kB     00:00    
………………省略部分输出信息………………
  Running scriptlet: mariadb-connector-c-3.0.7-1.el8.x86_64                 1/1 
  Preparing        :                                                        1/1 
  Installing       : xorg-x11-proto-devel-2018.4-1.el8.noarch             1/261 
  Installing       : perl-version-6:0.99.24-1.el8.x86_64                  2/261 
  Installing       : zlib-devel-1.2.11-10.el8.x86_64                      3/261 
  Installing       : perl-Time-HiRes-1.9758-1.el8.x86_64                  4/261 
  Installing       : libpng-devel-2:1.6.34-5.el8.x86_64                   5/261 
  Installing       : perl-CPAN-Meta-Requirements-2.140-396.el8.noarch     6/261 
  Installing       : perl-ExtUtils-ParseXS-1:3.35-2.el8.noarch            7/261 
  Installing       : perl-ExtUtils-Manifest-1.70-395.el8.noarch           8/261 
  Installing       : cmake-filesystem-3.11.4-3.el8.x86_64                 9/261 
  Installing       : perl-Test-Harness-1:3.42-1.el8.noarch               10/261 
  Installing       : perl-Module-CoreList-1:5.20181130-1.el8.noarch      11/261 
  Installing       : perl-Module-Metadata-1.000033-395.el8.noarch        12/261 
  Installing       : perl-SelfLoader-1.23-416.el8.noarch                 13/261 
  Installing       : perl-Perl-OSType-1.010-396.el8.noarch               14/261 
  Installing       : perl-Module-Load-1:0.32-395.el8.noarch              15/261 
  Installing       : perl-JSON-PP-1:2.97.001-3.el8.noarch                16/261 
  Installing       : perl-Filter-2:1.58-2.el8.x86_64                     17/261 
  Installing       : perl-Compress-Raw-Zlib-2.081-1.el8.x86_64           18/261 
  Installing       : perl-encoding-4:2.22-3.el8.x86_64                   19/261 
  Installing       : perl-Text-Balanced-2.03-395.el8.noarch              20/261 
………………省略部分输出信息………………
Complete!
```

如果条件允许，建议适当增加虚拟机的内存上限，让稍后的编译过程快一些。并且由于接下来还需要从外部网络中获取Nginx、Mysql、PHP及WordPress等一系列的安装包，因此需要配置虚拟机联网。

将已经调整为桥接模式的网卡，通过nmtui或nm-connection-editor命令修改为DHCP自动获取网卡信息模式，大多数情况下就可以接入互联网了，如图20-1所示。如若不可访问互联网，则优先考虑是否外部环境有特殊的限制，则将虚拟机内网卡配置成跟物理机一致即可。

```
[root@linuxprobe ~]# ping -c 4 www.linuxprobe.com
PING www.linuxprobe.com.w.kunlunno.com (202.97.231.16) 56(84) bytes of data.
64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=1 ttl=55 time=27.5 ms
64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=2 ttl=55 time=27.10 ms
64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=3 ttl=55 time=27.4 ms
64 bytes from www.linuxprobe.com (202.97.231.16): icmp_seq=4 ttl=55 time=28.9 ms

--- www.linuxprobe.com.w.kunlunno.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 8ms
rtt min/avg/max/mdev = 27.354/27.913/28.864/0.593 ms
```

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%8D%A1%E6%A8%A1%E5%BC%8F.png)

图20-1 设置网卡为DHCP自动获取模式

刘遄老师已经把安装LNMP动态网站部署架构所需的4个软件源码包和1个用于检查效果的论坛网站系统软件包上传到与本书配套的站点服务器上。大家可以在Windows系统中下载后通过ssh服务传送到打算部署LNMP动态网站架构的Linux服务器中，也可以直接在Linux服务器中使用wget命令下载这些源码包文件。为了更好的找到它们，统一放到/lnmp目录下保存：

```
[root@linuxprobe ~]# mdir /lnmp
[root@linuxprobe ~]# cd /lnmp
[root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/rpcsvc-proto-1.4.tar.gz
[root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/nginx-1.16.0.tar.gz
[root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/mysql-8.0.18.tar.xz
[root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/php-7.3.5.tar.gz
[root@linuxprobe lnmp]# wget https://www.linuxprobe.com/Software/wordpress.tar.gz
[root@linuxprobe lnmp]# ls
rpcsvc-proto-1.4.tar.gz       nginx-1.16.0.tar.gz   mysql-8.0.18.tar.xz
php-7.3.5.tar.gz              wordpress.tar.gz
```

将相关软件上传至书籍配套网站的目地是保证读者100%可找到它们，顺利的进行实验。但由于受到服务器或网络的限制，下载速度若不能达到读您的预期，亦可从互联网中下载同版本软件。

百度网盘打包下载链接：https://pan.baidu.com/s/1GRXh4E92OEi19-caz3QJ_w  提取码：**ghi0**

小试牛刀，rpcsvc-proto是一款用于包含rcpsvc协议文件的支持软件包名称，在后续Nginx与Mysql服务程序的部署过程中都需要被调用到。而要想通过源码包安装服务程序，就一定要严格遵守上面总结的安装步骤—下载及解压源码包文件、编译源码包代码、生成二进制安装程序、运行二进制的服务程序安装包。接下来在解压、编译各个软件包源码程序时，都会生成大量的输出信息，下文中将其省略，请读者以实际操作为准。

```
[root@linuxprobe lnmp]# tar xzvf rpcsvc-proto-1.4.tar.gz 
[root@linuxprobe lnmp]# cd rpcsvc-proto-1.4/
[root@linuxprobe rpcsvc-proto-1.4]# ./configure
[root@linuxprobe rpcsvc-proto-1.4]# make 
[root@linuxprobe rpcsvc-proto-1.4]# make install
[root@linuxprobe rpcsvc-proto-1.4]# cd ..
[root@linuxprobe lnmp]#
```

本章节内涉及软件较多，频繁切换工作目录在所难免，一方面我们会在每次操作后尽可能的返回到/lnmp目录下待命，一方面也请读者们仔细看清所在目录路径，避免找不到文件而影响学习心情~

###### **20.2.1 配置Nginx服务**

Nginx是一款相当优秀的用于部署动态网站的轻量级服务程序，它最初是为俄罗斯门户站点而开发的，因其稳定性、功能丰富、占用内存少且并发能力强而备受用户的信赖。目前国内诸如新浪、网易、腾讯等门户站点均已使用了此服务。

Nginx服务程序的稳定性源自于采用了分阶段的资源分配技术，降低了CPU与内存的占用率，所以使用Nginx程序部署的动态网站环境不仅十分稳定、高效，而且消耗的系统资源也很少。此外，Nginx具备的模块数量与Apache具备的模块数量几乎相同，而且现在已经完全支持proxy、rewrite、mod_fcgi、ssl、vhosts等常用模块。更重要的是，Nginx还支持热部署技术，7×24不间断提供服务，还可以在不暂停服务的情况下直接对Nginx服务程序进行升级。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2015/11/Nginx%E4%B8%8EApache.png)

坦白来讲，虽然Nginx程序的代码质量非常高，代码很规范，技术成熟，模块扩展也很容易，但依然存在不少问题，比如是由俄罗斯人开发的，所以在资料文档方面还并不完善，中文资料的质量更是鱼龙混杂。但是Nginx服务程序在近年来增长势头迅猛，相信会在轻量级Web服务器市场具有不错的未来。

第1步：创建用于管理网站服务的系统账户。这是在Linux系统创建之初就刻入的基因片段，为了能够让操作系统更加安全，让不同的系统用户管理对应的服务程序。这样即便有骇客通过网站服务侵入了服务器，也无法提权到更高权限，或是对系统进行更大的破坏，甚至都无法登陆ssh协议，因为仅仅拿到的是一个系统账号而已。不同于以往，此次新建的账户应使用“-M”参数不创建对应的家目录，以及用“-s”指定登录Shell解释器为/sbin/nologin，让任何人都不能通过这个账号登陆到主机。

```
[root@linuxprobe lnmp]# useradd nginx -M -s /sbin/nologin
[root@linuxprobe lnmp]# id nginx
uid=1001(nginx) gid=1001(nginx) groups=1001(nginx)
```

第2步：编译安装Nginx网站服务程序。为了能够让网站服务支持更多的功能，我们会在编译过程中添加额外的参数，较为重要的是用“prefix”参数指定服务将被安装到哪个目录，方便后面找到和调用它。其次是考虑到HTTPS协议越来越被广泛使用，所以用“with-http_ssl_module”参数来开启Nginx服务的SSL加密模块，便于日后开启HTTPS协议功能：

```
[root@linuxprobe lnmp]# tar zxvf nginx-1.16.0.tar.gz
[root@linuxprobe lnmp]# cd nginx-1.16.0/
[root@linuxprobe nginx-1.16.0]# ./configure --prefix=/usr/local/nginx --with-http_ssl_module 
[root@linuxprobe nginx-1.16.0]# make 
[root@linuxprobe nginx-1.16.0]# make install
[root@linuxprobe nginx-1.16.0]# cd ..
```

相对来说，configure编译脚本文件比make生成二进制文件命令要快，而make install安装程序则一般是最快的，相当于双击运行二进制安装包的操作。在编译、生成、安装三阶段中，屏幕上会输出各式各样的信息，主要包含软件包的概要情况、当前系统的软件依赖关系、及是否有条件进行安装操作。但只要进程没有被强制终止，或是输出明显报错信息，全是正常情况。

第3步：安装完毕后进行最终配置阶段。既然在编译环境中使用“prefix”参数指定了安装路径，那么Nginx服务程序配置文件一定会乖乖的在/usr/local/nginx目录中等着您的。

我们总共要进行三处修改，首先是把第2行的注释符（#）删除，然后在后面写上负责运行网站服务程序的账户名称和用户组名称，设定由nginx用户及nginx用户组负责管理网站服务，让网站服务能够顺利的被系统所读取：

```
[root@linuxprobe lnmp]# vim /usr/local/nginx/conf/nginx.conf 
  1 
  2 user  nginx nginx;
```

其次是修改第45行的首页文件名称，添加上index.php的名字，也就是让用户浏览网站时第一眼所看到的文件，也叫首页文件。

```
 43         location / {
 44             root   html;
 45             index  index.php index.html index.htm;
 46         }
```

最后再删除第65至71行前面的注释符（#）来启用虚拟主机功能，将第69行后面对应的网站根目录修改为“/usr/local/nginx/html”，fastcgi_script_name参数用于指代脚本名称，也就是用户请求的URL。正确填写方能使Nginx服务解析用户请求，否则访问的页面都会提示404未找到的错误。

```
 63         # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
 64         #
 65         location ~ \.php$ {
 66             root           html;
 67             fastcgi_pass   127.0.0.1:9000;
 68             fastcgi_index  index.php;
 69             fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html$fastcgi_script_name;
 70             include        fastcgi_params;
 71         }
```

第4步：通过编译源码方式安装的服务默认不能够被systemctl命令所管理，而要用Nginx服务本身的管理工具进行操作，对应的命令目录是/usr/local/nginx/sbin。但使用绝对路径的形式输入命令未免会显得麻烦，建议将/usr/local/nginx/sbin路径加入到PATH变量中，让Bash解释器在后续执行命令时自动的搜索到它。source命令加载配置文件让参数立即生效，下次就只需要输入“nginx”命令即可启动网站服务了，很方便呢~

```
[root@linuxprobe lnmp]# vim ~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
	. ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/usr/local/nginx/sbin

export PATH
[root@linuxprobe lnmp]# source ~/.bash_profile
[root@linuxprobe lnmp]# nginx 
```

操作完毕！重启服务程序~并在浏览器中输入本机的IP地址，即可浏览到Nginx网站服务程序的默认界面，如图20-2所示。相较于Apache服务程序的红色默认页面，Nginx服务程序的默认页面显得更加简洁。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/%E8%AE%BF%E9%97%AE%E7%BD%91%E7%AB%99%E6%88%90%E5%8A%9F-1.png)

图20-2 Nginx服务程序的默认页面

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

###### **20.2.2 配置Mysql服务**

本书在第18章讲解过MySQL和MariaDB数据库管理系统之间的因缘和特性，也狠狠地夸奖了MariaDB数据库，但是MySQL数据库当前依然是生产环境中最常使用的关系型数据库管理系统之一，坐拥极大的市场份额，并且已经通过十几年不断的发展向业界证明了自身的稳定性和安全性。另外，虽然第18章已经讲解了基本的数据库管理知识，但是为了进一步帮助大家夯实基础，本章依然在这里整合了MySQL数据库内容，使大家在温故的同时知新。
![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/mysql-300x155.png)

在使用软件仓库安装服务程序时，系统会自动根据RPM软件包中的指令集完整软件配置等工作。但是一旦选择使用源码包的方式来安装，这一切就需要自己来完成了。针对MySQL数据库来讲，我们需要在系统中创建一个名为mysql的用户，专门用于负责运行MySQL数据库。请记得要把这类账户的Bash终端设置成nologin解释器，避免黑客通过该用户登录到服务器中，从而提高系统安全性。

```
[root@linuxprobe lnmp]# useradd mysql -M -s /sbin/nologin
```

第1步：解压MySQL安装软件包。将解压出的程序目录改名并移动到/usr/local目录下，对其进行初始化操作后便可使用。但请注意.tar.xz结尾的压缩包软件，不应用“z”参数进行解压。

```
[root@linuxprobe lnmp]# tar xvf mysql-8.0.18.tar.xz
[root@linuxprobe lnmp]# mv mysql-8.0.18-linux-glibc2.12-x86_64 mysql
[root@linuxprobe lnmp]# mv mysql /usr/local
```

第2步：生产环境中管理MySQL数据库时，有两个比较常用的目录。其一是/usr/local/mysql目录，这是用于保存MySQL数据库程序文件的路径。还有一个是/usr/local/mysql/data目录，它用于存储数据库的具体内容，每个数据库的内容会被单独存放到一个目录内。对于存放实际数据库文件的data目录，用户需要先手动创建出来：

```
[root@linuxprobe lnmp]# cd /usr/local/mysql
[root@linuxprobe mysql]# mkdir data
```

第3步：初始化MySQL服务程序，对目录进行授权，保证数据能够被mysql系统用户所读取。在初始化阶段，应使用mysqld命令确认管理MySQL数据库服务的用户名称、数据保存目录及编码信息。信息确认无误后会初始化完毕，在最后一段会给予用户初始化临时密码，一定要保存好，例如本书所分配的密码为qfroRs,Ei4Ls。

```
[root@linuxprobe mysql]# chown -R mysql:mysql /usr/local/mysql
[root@linuxprobe mysql]# cd bin
[root@linuxprobe bin]# ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data
2021-05-06T07:07:06.243270Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.18) initializing of server in progress as process 7606
2021-05-06T07:07:08.116268Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qfroRs,Ei4Ls
```

第4步：与Nginx服务相似，MySQL数据库的二进制可执行命令也是单独存放在自身程序目录中的，每一次都切换到/usr/local/mysql/bin目录中再执行着实有些麻烦，要能也加入到PATH变量中可就方便太多了。说干就干：

```
[root@linuxprobe bin]# vim ~/.bash_profile
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:$HOME/bin:/usr/local/nginx/sbin:/usr/local/mysql/bin

export PATH
[root@linuxprobe bin]# source ~/.bash_profile
```

这样设置后，即便返回到了源码目录，也可以继续执行MySQL数据库的管理命令。不过先别着急！既然是手动安装服务，那么让文件“归位”的重任就只得亲力亲为了——将启动脚本mysql.server放入到/etc/init.d目录中，让服务器每次重启后都能自动启动数据库，并给予可执行权限。

libtinfo.so.5文件是MySQL数据库在8.0版本后重要的函数库文件，需要将libtinfo.so.6.1文件复制或者作为链接文件才能正常启动：

```
[root@linuxprobe bin]# cd /usr/local/mysql
[root@linuxprobe mysql]# cp -a support-files/mysql.server /etc/init.d/
[root@linuxprobe mysql]# chmod a+x /etc/init.d/mysql.server
[root@linuxprobe mysql]# ln -s /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5 
```

第5步：执行MySQL数据库服务启动文件，并进行初始化工作。8.0版本以后为了安全着想，不再允许用户使用临时密码管理数据库内容，也不能远程控制，用户必须修改初始化密码后才能使用。数据库作为系统重要的组成服务，密码不建议少于20位，例如想修改为“PObejCBeDzTRCncXwgBy”。

```
[root@linuxprobe mysql]# /etc/init.d/mysql.server start 
Starting MySQL.Logging to '/usr/local/mysql/data/linuxprobe.com.err'.
. SUCCESS! 
[root@linuxprobe mysql]# mysql -u root -p
Enter password: 输入初始化时给的原始密码
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.18

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> alter user 'root'@'localhost' identified by 'PObejCBeDzTRCncXwgBy'; 
Query OK, 0 rows affected (0.01 sec)

mysql> 
```

但这样还是不行，需要继续切换至mysql数据库中，修改user表单的密码值。这也是从MySQL数据库8.0版本后才开始的新安全要求，看过《Linux就该这么学》RHEL7版本的老读者应该记得5.6版本时就没有这么麻烦。

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
| …………省略部分输出信息…………  |
+---------------------------+
33 rows in set (0.00 sec)

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PObejCBeDzTRCncXwgBy';
Query OK, 0 rows affected (0.01 sec)
```

由于稍后将在20.4小节安装部署WordPress网站系统，因此现在需要提前把数据库新建出来：

```
mysql> create database linuxcool;
Query OK, 1 row affected (0.00 sec)

mysql> exit
Bye
```

###### **20.2.3 配置php服务**

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/PHP.png)PHP（Hypertxt Preprocessor，超文本预处理器）是一种通用的开源脚本语言，发明于1995年，它吸取了C语言、Java语言及Perl语言的很多优点，具有开源、免费、快捷、跨平台性强、效率高等优良特性，是目前Web开发领域最常用的语言之一。使用源码包的方式编译安装PHP语言环境其实并不复杂，难点在于解决PHP的程序包和其他软件的依赖关系。

第1步：解压php安装包软件并编译安装。编译中需要使用“prefix”参数指定安装路径，使用“--with-mysqli”等命令开启对数据库的支持模块，为后面在线安装网站夯实基础。

```
[root@linuxprobe mysql]# cd /lnmp
[root@linuxprobe lnmp]# tar xvf php-7.3.5.tar.gz
[root@linuxprobe lnmp]# cd php-7.3.5/
[root@linuxprobe php-7.3.5]# ./configure --prefix=/usr/local/php --enable-fpm --with-mysqli --with-curl --with-pdo_mysql --with-pdo_sqlite --enable-mysqlnd --enable-mbstring --with-gd
```

生成二进制文件和安装依然为下述命令，时间大约为10~20分钟，耐心等待即可：

```
[root@linuxprobe php-7.3.5]# make
[root@linuxprobe php-7.3.5]# make install
```

第2步：将生成出的php服务配置文件复制到安装目录中（/usr/local/php/），让其生效。主配置文件有了，接下来还需要php-fpm的配置文件，好在也已经提供，只需要复制模板即可：

```
[root@linuxprobe php-7.3.5]# cp php.ini-development /usr/local/php/lib/php.ini
[root@linuxprobe php-7.3.5]# cd /usr/local/php/etc/
[root@linuxprobe etc]# mv php-fpm.conf.default php-fpm.conf
```

复制一个模板文件到php-fpm.d的目录中，用于后续控制网站的连接性能：

```
[root@linuxprobe etc]# mv php-fpm.d/www.conf.default php-fpm.d/www.conf
```

第3步：把php服务加入到启动项中，使其重启后依然生效：

```
[root@linuxprobe etc]# cd /lnmp/php-7.3.5
[root@linuxprobe php-7.3.5]# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
[root@linuxprobe php-7.3.5]# chmod 755 /etc/init.d/php-fpm
```

第4步：由于php服务程序的配置参数直接会影响到Web服务服务的运行环境，如果默认开启了一些不必要且高危的功能（如允许用户在网页中执行Linux命令），则会降低网站被入侵的难度，入侵人员甚至可以拿到整台Web服务器的管理权限。因此我们需要编辑php.ini配置文件，在310行的disable_functions参数后面追加上要禁止的功能。下面的禁用功能名单是刘遄老师依据网站运行的经验而定制的，不见得适合每个生产环境，建议大家在此基础上根据自身工作需求酌情删减：

```
[root@linuxprobe php-7.3.5]# vim /usr/local/php/lib/php.ini
 307 ; This directive allows you to disable certain functions for security reasons.
 308 ; It receives a comma-delimited list of function names.
 309 ; http://php.net/disable-functions
 310 disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
```

第5步：LNMP架构源码编译工作就此结束~享受胜利成果吧：

```
[root@linuxprobe php-7.3.5]# /etc/init.d/php-fpm start
Starting php-fpm done
```

##### **20.3 搭建Discuz论坛**

为了检验LNMP动态网站架构环境是否配置妥当，可以在上面部署WordPress博客系统，然后查看效果。如果能够在LNMP动态网站环境中成功安装使用WordPress网站系统，也就意味着这套架构是可用的。WordPress是一个以PHP和MySQL为平台的开源博客软件，具有丰富的插件和模板系统，截止于2021年5月，全球排名前1000万的网站中已有超过41%使用了WordPress，是当前最受欢迎的网站内容管理系统。

把Nginx服务程序根目录的内容清空后，将WordPress解压后的网站文件复制进去：

```
[root@linuxprobe php-7.3.5]# cd ..
[root@linuxprobe lnmp]# rm -f /usr/local/nginx/html/*
[root@linuxprobe lnmp]# tar xzvf wordpress.tar.gz 
[root@linuxprobe lnmp]# mv wordpress/* /usr/local/nginx/html/
```

为了能够让网站文件被Nginx服务程序顺利读取，应设置目录所有身份及可读写的权限：

```
[root@linuxprobe lnmp]# chown -Rf nginx:nginx /usr/local/nginx/html 
[root@linuxprobe lnmp]# chmod -Rf 777 /usr/local/nginx/html
```

随后输入本机IP地址访问网站首页，如图20-3所示，WordPress网站系统的安装界面提醒了用户稍后需要的安装信息。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/1-1-1024x624.png)

图20-3 WordPress网站系统安装首页面

在安装界面应依次输入刚刚建立的数据库名称、账号及重置过的密码值，WordPress要求用户自行创建好数据库，因此请确保网页中填写的数据库名称与刚刚创建的一致，如图20-4所示。点击“提交”按钮进行确认后，便可进行最终的安装阶段，如图20-5所示。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/2-1-1024x624.png)

图20-4 填写安装信息

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/3-1-1024x624.png)

图20-5 确认安装WordPress网站系统

顺利安装完毕后，WordPress网站系统会向用户寻求站点标题、用户名及密码等信息，如图20-6所示。这些信息均可自行填写，密码稍微复杂一些为好，检查无误后即可点击“安装WordPress”按钮进行确认。成功的界面如图20-7所示。

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/4-1024x624.png)

图20-6 填网站标题及管理员名称

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/5-1024x624.png)

图20-7 安装完成界面

WordPress的登录界面将在用户点击登录后自动显现，如图20-8所示以填写账号及密码吗，然后点击“登录”按钮：

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/6-1024x624.png)

图20-8 填写网站账号和密码

顺利进入到管理后台中，如图20-9所示。读者一定会很好奇，WordPress作为最热门的网站内容管理系统，都能做出什么样的网站呢~感兴趣的同学可以进一步访问我们的随书配套站点，三个风格迥异的站点，但都是基于WordPress搭建的呢~

> 《Linux就该这么学》最新版电子书：[https://www.linuxprobe.com](https://www.linuxprobe.com/)
>
> Linux命令大全：[https://www.linuxcool.com](https://www.linuxcool.com/)
>
> Linux系统大全：[https://www.linuxdown.com](https://www.linuxdown.com/)

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/7-1024x624.png)图20-9 WordPress管理后台

看到这个界面，刘遄老师心中真是五味杂陈，从2015年写书创业至今，它见证了我们梦想成真。建议对未来有期许的同学也动手搭建出一个属于自己的网站，不仅可以让好内容帮助到更多人，也或许能够帮助您实现梦想呢~

##### **20.4 选购服务器主机**

我们日常访问的网站是由域名、网站源程序和主机共同组成的，其中，主机则是用于存放网页源代码并能够把网页内容展示给用户的服务器。在本书即将结束之际，再啰嗦几句有关服务器主机的知识以及选购技巧，这些技巧都是在近几年做网站时总结出来的，希望能对大家有所帮助。

**虚拟主机**：在一台服务器中划分一定的磁盘空间供用户放置网站信息、存放数据等；仅提供基础的网站访问、数据存放与传输功能；能够极大地降低用户费用，也几乎不需要用户来维护网站以外的服务；适合小型网站。

**VPS（Virtual Private Server，虚拟专用服务器）**：在一台服务器中利用OpenVZ、Xen或KVM等虚拟化技术模拟出多台“主机”（即VPS），每个主机都有独立的IP地址、操作系统；不同VPS之间的磁盘空间、内存、CPU、进程与系统配置完全隔离，用户可自由使用分配到的主机中的所有资源，为此需要具备一定的维护系统的能力；适合小型网站。

**ECS（Elastic Compute Service，云服务器）**：是一种整合了计算、存储、网络，能够做到弹性伸缩的计算服务；使用起来与VPS几乎一样，差别是云服务器是建立在一组集群服务器中，每个服务器都会保存一个主机的镜像（备份），从而大大提升了安全性和稳定性；另外还具备灵活性与扩展性；用户只需按使用量付费即可；适合大中小型网站。

**独立服务器**：这台服务器仅提供给用户一个人使用，其使用方式分为租用方式与托管方式。租用方式是用户将服务器的硬件配置要求告知IDC服务商，按照月、季、年为单位来租用它们的硬件设备。这些硬件设备由IDC服务商的机房负责维护，用户一般需要自行安装相应的软件并部署网站服务，这减轻了用户在硬件设备上的投入，适合大中型网站。托管方式则是用户需要自行购置服务器硬件设备，并将其交给IDC服务供应商进行管理（需要缴纳管理服务费）。用户对服务器硬件配置有完全的控制权，自主性强，但需要自行维护、修理服务器硬件设备，适合大中型网站。

另外需要提醒读者的是，在选择服务器主机供应商时请一定要注意查看口碑，并在综合分析后再决定购买。某些供应商会有限制功能、强制添加广告、隐藏扣费或强制扣费等恶劣行为，请各位读者一定擦亮眼睛，不要上当!

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．使用源码包安装服务程序的最大优点和缺点是什么？

**答：**使用源码包安装服务程序的最大优势是，服务程序的可移植性好，而且能更好地提升服务程序的运行效率；缺点是源码包程序的安装、管理、卸载和维护都比较麻烦。

2．使用源码包的方式来安装软件服务的大致步骤是什么？

**答：**基本分为4个步骤，分别为下载及解压源码包文件、编译源码包代码、生成二进制安装程序、运行二进制的服务程序安装包。

3．LNMP动态网站部署架构通常包含了哪些服务程序？

**答：**LNMP动态网站部署架构通常包含Linux系统、Nginx网站服务、MySQL数据库管理系统，以及PHP脚本语言。

4．在MySQL数据库服务程序中，/usr/local/mysql与/usr/local/mysql/data目录的作用是什么？

**答：**/usr/local/mysql用于保存MySQL数据库服务程序的目录，/usr/local/mysql/var则用于保存真实数据库文件的目录。

5．较之于Apache服务程序，Nginx最显著的优势是什么？

**答：**Nginx服务程序比较稳定，原因是采用了的资源分配技术，降低了CPU与内存的占用率，所以使用Nginx程序部署的动态网站环境不仅十分稳定、高效，而且消耗的系统资源也很少。

6．如何禁止php服务程序中不安全的功能？

**答：**编辑php服务程序的配置文件（/usr/local/php/etc/php.ini），把要禁用的功能追加到disable_functions参数之后即可。

7． 对于处于创业阶段的小站长群体来说，适合购买哪种服务器类型呢？

**答：**刘遄老师建议他们选择云服务器类型，不但费用便宜（每个月费用不超过100元人民币），而且性能也十分强劲。