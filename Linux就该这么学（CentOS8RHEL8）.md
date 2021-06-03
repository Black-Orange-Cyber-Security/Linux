# Linux就该这么学（CentOS8|RHEL8）

------

[TOC]



# [第5章 用户身份与文件权限](https://www.linuxprobe.com/basic-learning-05.html)

**章节简述：**

[Linux](https://www.linuxprobe.com/)是一个多用户、多任务的操作系统，具有很好的稳定性与安全性，在幕后保障[Linux系统](https://www.linuxprobe.com/)安全的则是一系列复杂的配置工作。本章将详细讲解文件的所有者、所属组以及其他人可对文件进行的读（r）、写（w）、执行（x）等操作，以及如何在Linux系统中添加、删除、修改用户账户信息。

我们还可以使用SUID、SGID与SBIT特殊权限更加灵活地设置系统权限，来弥补对文件设置一般操作权限时所带来的不足。隐藏权限能够给系统增加一层隐形的防护层，让黑客最多只能查看关键日志信息，而不能篡改或删除。而文件访问控制列表（Access Control List，ACL）可以进一步让单一用户、用户组对单一文件或目录进行特殊的权限设置，让文件具有能满足工作需求的最小权限。

最后还会讲解如何使用su[命令](https://www.linuxcool.com/)与sudo服务让普通用户具备管理员的权限，不仅能够满足日常工作需求，还可以确保系统的安全性。

本章目录结构

- [5.1 用户身份与能力](https://www.linuxprobe.com/basic-learning-05.html#51)
- [5.2 文件权限与归属](https://www.linuxprobe.com/basic-learning-05.html#52)
- [5.3 文件的特殊权限](https://www.linuxprobe.com/basic-learning-05.html#53)
- [5.4 文件的隐藏属性](https://www.linuxprobe.com/basic-learning-05.html#54)
- [5.5 文件访问控制列表](https://www.linuxprobe.com/basic-learning-05.html#55)
- [5.6 su命令与sudo服务](https://www.linuxprobe.com/basic-learning-05.html#56_susudo)

##### **5.1 用户身份与能力**

受到上世纪70年代计算机发展的影响，Linux系统的设计初衷之一就是为了满足多个用户同时工作的需求，因此必须具备很好的安全性，尤其是不能因为一两个服务出错而影响到整台服务器。在第一章学习安装Linux操作系统时，特别要求设置root管理员的密码，这个root管理员就是存在于所有类UNIX系统中的超级用户。它拥有最高的系统所有权，能够管理系统的各项功能，如添加/删除用户、启动/关闭服务进程、开启/禁用硬件设备等。虽然以root管理员的身份工作时不会受到系统的限制，但俗语讲“能力越大，责任就越大”，因此一旦使用这个高能的root管理员权限执行了错误的[命令](https://www.linuxcool.com/)可能会直接毁掉整个系统。使用与否，确实需要好好权衡一下。

在学习时是否要使用root管理员权限来控制整个系统呢？面对这个问题，网络上有很多文章建议以普通用户的身份来操作—这是一个更安全也更“无责任”的回答。今天，就要冒天下之大不韪给出自己的心得—强烈推荐大家在学习时使用root管理员权限！

这种为root管理员正名的决绝态度在网络中应该还是很少见的，我之所以力荐root管理员权限，原因很简单。因为在Linux的学习过程中如果使用普通用户身份进行操作，则在配置服务之后出现错误时很难判断是系统自身的问题还是因为权限不足而导致的；这无疑会给大家的学习过程徒增坎坷。更何况我们的实验环境是使用VMware虚拟机软件搭建的，能够将安装好的系统设置为一次快照，这即便系统彻底崩溃了，您也可以在5秒钟的时间内快速还原出一台全新的系统，而不用担心数据丢失。

总之，[刘遄](https://www.linuxprobe.com/)老师在培训时都推荐每位学生使用root管理员权限来学习Linux系统，等到工作时再根据生产环境决定使用哪个用户权限；这些仅与选择相关，而非技术性问题。

另外，很多图书或培训机构的老师会讲到，Linux系统中的管理员就是root。这其实是错误的，Linux系统的管理员之所以是root，并不是因为它的名字叫root，而是因为该用户的身份号码即UID（User IDentification）的数值为0。在Linux系统中，UID就相当于身份证号码一样具有唯一性，因此可通过用户的UID值来判断用户身份。在RHEL 8系统中，用户身份有下面这些。

> 管理员UID为0：系统的管理员用户。
>
> 系统用户UID为1～999： Linux系统为了避免因某个服务程序出现漏洞而被黑客提权至整台服务器，默认服务程序会有独立的系统用户负责运行，进而有效控制被破坏范围。
>
> 普通用户UID从1000开始：是由管理员创建的用于日常工作的用户。

需要注意的是，UID是不能冲突的，而且管理员创建的普通用户的UID默认是从1000开始的（即使前面有闲置的号码）。

为了方便管理属于同一组的用户，Linux系统中还引入了用户组的概念。通过使用用户组号码（GID，Group IDentification），可以把多个用户加入到同一个组中，从而方便为组中的用户统一规划权限或指定任务。假设有一个公司中有多个部门，每个部门中又有很多员工，如果只想让员工访问本部门内的资源，则可以针对部门而非具体的员工来设置权限。例如，通过对技术部门设置权限，使得只有技术部门的员工可以访问公司的数据库信息等。

另外，在Linux系统中创建每个用户时，将自动创建一个与其同名的基本用户组，而且这个基本用户组只有该用户一个人。如果该用户以后被归纳入其他用户组，则这个其他用户组称之为扩展用户组。一个用户只有一个基本用户组，但是可以有多个扩展用户组，从而满足日常的工作需要。

### **Tips**

基本组就像是原生家庭，这是随创建账号（出生）就自动完成的；而扩展组则像工作单位，为了完成工作，就要加入到各个不同的群体中，这是需要手动添加的。

**1.  id命令**

id命令用于显示用户详细信息，语法格式为：“id 用户名”。

这是一个要在创建用户前仔细学习的命令，它能够简单轻便的查看到用户的基本信息，例如用户ID、基本组与扩展组GID，便于我们判别某个用户是否已经存在，以及查看到相关信息。

查看一个名称为linuxprobe的用户信息：

```shell
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
```

**2.  useradd命令**

useradd命令用于创建新的用户账户，语法格式为：“useradd [参数] 用户名”。

可以使用useradd命令创建用户账户。使用该命令创建用户账户时，默认的用户家目录会被存放在/home目录中，默认的[Shell](https://www.linuxcool.com/)解释器为/bin/bash，而且默认会创建一个与该用户同名的基本用户组。这些默认设置可以根据表5-1中的useradd命令参数自行修改。

表5-1                    useradd命令中的参数以及作用

| 参数 | 作用                                     |
| ---- | ---------------------------------------- |
| -d   | 指定用户的家目录（默认为/home/username） |
| -e   | 账户的到期时间，格式为YYYY-MM-DD.        |
| -u   | 指定该用户的默认UID                      |
| -g   | 指定一个初始的用户基本组（必须已存在）   |
| -G   | 指定一个或多个扩展用户组                 |
| -N   | 不创建与用户同名的基本用户组             |
| -s   | 指定该用户的默认Shell解释器              |



创建一个名称为linuxcool的用户，并使用id命令确认信息：

```shell
[root@linuxprobe ~]# useradd linuxcool
[root@linuxprobe ~]# id linuxcool
uid=1001(linuxcool) gid=1001(linuxcool) groups=1001(linuxcool)
```

下面我们提高难度再创建一个普通用户并指定家目录的路径、用户的UID以及Shell解释器。在下面的命令中，请注意/sbin/nologin，它是终端解释器中的一员，与Bash解释器有着天壤之别。一旦用户的解释器被设置为nologin，则代表该用户不能登录到系统中：

```shell
[root@linuxprobe ~]# useradd -d /home/linux -u 8888 -s /sbin/nologin linuxdown
[root@linuxprobe ~]# id linuxdown
uid=8888(linuxdown) gid=8888(linuxdown) groups=8888(linuxdown)
```

**3.  groupadd命令**

groupadd命令用于创建新的用户组，语法格式为：“groupadd [参数] 群组名”。

为了能够更加高效地指派系统中各个用户的权限，在工作中常常会把几个用户加入到同一个组里面，这样便可以针对一类用户统一安排权限。例如工作中成立一个部门组，有新的同事加入时就把他的账号添加到部门组中，权限自动就跟其他人一模一样了，省去了一系列繁琐的操作。

创建用户组的步骤非常简单，例如使用如下命令创建一个用户组ronny：

```shell
[root@linuxprobe ~]# groupadd ronny
```

**4.  usermod命令**

usermod命令用于修改用户的属性，英文全称为：“user modify”，语法格式为：“ usermod [参数] 用户名”。

前文曾反复强调，Linux系统中的一切都是文件，因此在系统中创建用户也就是修改配置文件的过程。用户的信息保存在/etc/passwd文件中，可以直接用文本编辑器来修改其中的用户参数项目，也可以用usermod命令修改已经创建的用户信息，诸如用户的UID、基本/扩展用户组、默认终端等。usermod命令的参数以及作用如表5-2所示。

表5-2                      usermod命令中的参数以及作用

| 参数  | 作用                                                         |
| ----- | ------------------------------------------------------------ |
| -c    | 填写用户账户的备注信息                                       |
| -d -m | 参数-m与参数-d连用，可重新指定用户的家目录并自动把旧的数据转移过去 |
| -e    | 账户的到期时间，格式为YYYY-MM-DD                             |
| -g    | 变更所属用户组                                               |
| -G    | 变更扩展用户组                                               |
| -L    | 锁定用户禁止其登录系统                                       |
| -U    | 解锁用户，允许其登录系统                                     |
| -s    | 变更默认终端                                                 |
| -u    | 修改用户的UID                                                |



大家不要被这么多参数吓坏了，再来看一下账户linuxprobe的默认信息：

```shell
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
```

然后将用户linuxprobe加入到root用户组中，这样扩展组列表中则会出现root用户组的字样，而基本组不会受到影响：

```shell
[root@linuxprobe ~]# usermod -G root linuxprobe
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe),0(root)
```

再来试试用-u参数修改linuxprobe用户的UID号码值：

```shell
[root@linuxprobe ~]# usermod -u 8888 linuxprobe
[root@linuxprobe ~]# id linuxprobe
uid=8888(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe),0(root)
```

除此之外，同学们最关心的肯定是如果把用户的解释器终端由默认的/bin/bash修改为/sbin/nologin后会有什么样的效果呢？那来试试吧：

```shell
[root@linuxprobe ~]# usermod -s /sbin/nologin linuxprobe
[root@linuxprobe ~]# su - linuxprobe
This account is currently not available.
```

效果很直观吧，将用户的终端设置成/sbin/nologin后马上就不能被登录了（切换身份也不行），但这个用户依然可以被某个服务所调用，管理某个具体的服务。这样的好处是当骇客通过这个服务入侵成功，破坏的范围也仅仅局限于这个特定的服务，不能使用这个用户身份登录到整台服务器上，尽可能的把损失最小化。

**5.  passwd命令**

passwd命令用于修改用户的密码、过期时间等信息，英文全称为：“password”，语法格式为：“ passwd [参数] 用户名”。

普通用户只能使用passwd命令修改自己的系统密码，而root管理员则有权限修改其他所有人的密码。更酷的是，root管理员在Linux系统中修改自己或他人的密码时不需要验证旧密码，这一点特别方便。既然root管理员能够修改其他用户的密码，就表示完全拥有该用户的管理权限。passwd命令中可用的参数以及作用如表5-3所示。

表5-3                      passwd命令中的参数以及作用

| 参数    | 作用                                                         |
| ------- | ------------------------------------------------------------ |
| -l      | 锁定用户，禁止其登录                                         |
| -u      | 解除锁定，允许用户登录                                       |
| --stdin | 允许通过标准输入修改用户密码，如echo "NewPassWord" \| passwd --stdin Username |
| -d      | 使该用户可用空密码登录系统                                   |
| -e      | 强制用户在下次登录时修改密码                                 |
| -S      | 显示用户的密码是否被锁定，以及密码所采用的加密算法名称       |



修改自己的密码只需要输入命令后敲击回车即可：

```
[root@linuxprobe ~]# passwd
Changing password for user root.
New password: 此处输入密码值
Retype new password: 再次输入进行确认
passwd: all authentication tokens updated successfully.
```

而修改其他人的密码，首先需要检查当前是否为root管理员权限，然后命令后指定要修改那位用户的名称：

```
[root@linuxprobe ~]# passwd linuxprobe
Changing password for user linuxprobe.
New password:此处输入密码值
Retype new password: 再次输入进行确认
passwd: all authentication tokens updated successfully.
```

假设您有位同事正在度假，而且假期很长，那么可以使用passwd命令禁止该用户登录系统，等假期结束回归工作岗位时，再使用该命令允许用户登录系统，而不是将其删除。这样既保证了这段时间内系统的安全，也避免了频繁添加、删除用户带来的麻烦：

```
[root@linuxprobe ~]# passwd -l linuxprobe
Locking password for user linuxprobe.
passwd: Success
[root@linuxprobe ~]# passwd -S linuxprobe
linuxprobe LK 1969-12-31 0 99999 7 -1 (Password locked.)
```

需要解锁时记得也要用管理员的身份，否则普通用户如果有锁定权限的话，系统肯定会乱成一锅粥：

```
[root@linuxprobe ~]# passwd -u linuxprobe
Unlocking password for user linuxprobe.
passwd: Success
[root@linuxprobe ~]# passwd -S linuxprobe
linuxprobe PS 1969-12-31 0 99999 7 -1 (Password set, SHA512 crypt.)
```

**6.  userdel命令**

userdel命令用于删除已有的用户账户，英文全称为：“user delete”，语法格式为：“ userdel [参数] 用户名”。

如果确认某位用户后续不再会登录到系统中，则可以通过userdel命令删除该用户的所有信息。在执行删除操作时，该用户的家目录默认会保留下来，此时可以使用-r参数将其删除。userdel命令的参数以及作用如表5-4所示。

表5-4                       userdel命令中的参数以及作用

| 参数 | 作用                     |
| ---- | ------------------------ |
| -f   | 强制删除用户             |
| -r   | 同时删除用户及用户家目录 |



正常情况下删除一个用户时会建议保留他的家目录数据，避免有什么重要的数据被误删除，所以不用加参数，写清要删除的用户名称就行：

```
[root@linuxprobe ~]# userdel linuxprobe
[root@linuxprobe ~]# id linuxprobe
id: linuxprobe: no such user
```

此时该用户虽然被删除，但家目录数据会继续被存放在/home目录中，确认未来无需使用时再手动删除掉它：

```
[root@linuxprobe ~]# cd /home
[root@linuxprobe home]# ls
linuxprobe linuxcool linuxdown
[root@linuxprobe home]# rm -rf linuxprobe
[root@linuxprobe home]# ls 
linuxcool linuxdown
```

##### **5.2 文件权限与归属**

在Linux系统中，每个文件都有归属的所有者和所有组，并且规定了文件的所有者、所有组以及其他人对文件所拥有的读（r）、写（w）、执行（x）等权限。对于一般文件来说，权限比较容易理解：“可读”表示能够读取文件的实际内容；“可写”表示能够编辑、新增、修改、删除文件的实际内容；“可执行”则表示能够运行一个[脚本](https://www.linuxcool.com/)程序。但是，对于目录文件来说，理解其权限设置来就不那么容易了。很多资深Linux用户其实也没有真正搞明白。对目录文件来说，“可读”表示能够读取目录内的文件列表；“可写”表示能够在目录内新增、删除、重命名文件；而“可执行”则表示能够进入该目录。可以参考表格5-5，帮助同学们理解当文件和目录被设置rwx权限后，能够被执行的命令的区别：

表5-5                       读写执行权限对于文件与目录可执行命令的区别

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AF%BB%E5%86%99%E6%89%A7%E8%A1%8C%E6%9D%83%E9%99%90%E5%AF%B9%E4%BA%8E%E6%96%87%E4%BB%B6%E4%B8%8E%E7%9B%AE%E5%BD%95%E7%9A%84%E4%BD%9C%E7%94%A8-1024x168.png)

文件的读、写、执行权限英文全称分别是read、write、execute，可以简写为r、w、x，亦可分别用数字4、2、1来表示，文件所有者，所属组及其他用户权限之间无关联，如表5-6所示。

表5-6                       文件权限的字符与数字表示

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%96%87%E4%BB%B6%E6%9D%83%E9%99%90%E7%9A%84%E5%AD%97%E7%AC%A6%E4%B8%8E%E6%95%B0%E5%AD%97%E8%A1%A8%E7%A4%BA-1024x201.png)

文件权限的数字法表示基于字符（rwx）的权限计算而来，其目的是简化权限的表示方式。例如，若某个文件的权限为**7**则代表可读、可写、可执行（4+2+1）；若权限为6则代表可读、可写（4+2）。我们来看这样一个例子。现在有这样一个文件，其所有者拥有可读、可写、可执行的权限，其文件所属组拥有可读、可写的权限；而且其他人只有可读的权限。那么，这个文件的权限就是rwxrw-r--，数字法表示即为764。不过大家千万别再将这三个数字相加，计算出7+6+4=17的结果，这是小学的数学加减法，不是Linux系统的权限数字表示法，三者之间没有互通关系。

以rw-r-x-w-权限为例进行讲解，要想转换成数字法，首先要进行各个位之上的数字替代，如图5-1所示。

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%9D%83%E9%99%90%E8%BD%AC%E6%8D%A2-300x136.jpg)

图5-1 字符与数字权限转换示意图

减号是占位符，代表这里没有权限，数字法用0表示，也就是说rw-转换后是420，r-x转换后是401，-w-转换后是020，三组数字之间每组数字进行相加后得出652便是转换后的数字法权限。

而数字法转回到字符权限相比来说就有些难度了，以652权限为例进行讲解。首先数字6是由4+2得到的，不可能是4+1+1，因为每个权限只能占一位，所以数字5则是4+1得到的，2便是本身，没有权限即是空值0，可以按照表5-6所示的格式填写进去后得到420401020这样一串数字。有了这些信息就好办了，如图5-2所示，把他们都转换回来。

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%95%B0%E5%AD%97%E8%BD%AC%E6%8D%A2-300x136.jpg)

图5-2 数字与字符权限转换示意图

心中牢记——文件的所有者、所有组和其他人权限之间无关联，不要写成rrwwx----的样子，一定要把rwx权限位对应到正确的位置，写成rw-r-x-w-。

Linux系统的文件权限相当复杂，但是用途很广泛，建议大家把它彻底搞清楚之后再学习下一节的内容。现在来练习一下。请各位读者分别计算数字表示法764、652、153、731所对应的字符表示法，然后再把rwxrw-r--、rw--w--wx、rw-r--r--转换成数字表示法。

下面我们利用上文讲解的知识，一起分析图5-3中所示的文件信息。

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2015/02/%E6%96%87%E4%BB%B6%E6%9D%83%E9%99%90.png)

图5-3 通过ls命令查看到的文件属性信息

在图5-3中，包含了文件的类型、访问权限、所有者（属主）、所属组（属组）、占用的磁盘大小、最后修改时间和文件名称等信息。通过分析可知，该文件的类型为普通文件，所有者权限为可读、可写（rw-），所属组权限为可读（r--），除此以外的其他人也只有可读权限（r--），文件的磁盘占用大小是34298字节，最近一次的修改时间为4月2日的凌晨23分，文件的名称为install.log。

其中排在权限前面的减号（-）是文件类型，新手经常会把它跟无权限混淆。尽管在Linux系统中一切都是文件，但是不同的文件由于作用不同，因此类型也不尽相同，有一点点像Windows系统的后缀名。常见的文件类型包括有：普通文件（-）、目录文件（d）、链接文件（l）、管道文件（p）、块设备文件（b）以及字符设备文件（c）。

普通文件的范围特别广泛，比如纯文本信息、服务配置信息、日志信息以及Shell[脚本](https://www.linuxcool.com/)等等都包含在内，所以几乎在每个目录下都能看到普通文件（-）和目录文件（d）的身影。块设备文件（b）和字符设备文件（c）一般是指硬件设备，比如鼠标、键盘、光驱、硬盘等等都是设备文件，主要集中在/dev/目录中最为常见，不过其实很少会对鼠标键盘进行硬件级别的管理吧~

##### **5.3 文件的特殊权限**

在复杂多变的生产环境中，单纯设置文件的rwx权限无法满足我们对安全和灵活性的需求，因此便有了SUID、SGID与SBIT的特殊权限位。这是一种对文件权限进行设置的特殊功能，可以与一般权限同时使用，以弥补一般权限不能实现的功能。

下面具体解释这3个特殊权限位的功能以及用法。

**1.  SUID**

SUID是一种对二进制程序进行设置的特殊权限，能够让二进制程序的执行者临时拥有属主的权限（仅对拥有执行权限的二进制程序有效）。例如，所有用户都可以执行passwd命令来修改自己的用户密码，而用户密码保存在/etc/shadow文件中。仔细查看这个文件就会发现它的默认权限是000，也就是说除了root管理员以外，所有用户都没有查看或编辑该文件的权限。但是，在使用passwd命令时如果加上SUID特殊权限位，就可让普通用户临时获得程序所有者的身份，把变更的密码信息写入到shadow文件中。这很像在古装剧中见到的手持尚方宝剑的钦差大臣，他手持的尚方宝剑代表的是皇上的权威，因此可以惩戒贪官，但这并不意味着他永久成为了皇上。因此这只是一种有条件的、临时的特殊权限授权方法。

查看passwd命令属性时发现所有者的权限由rwx变成了rws，其中x改变成s就意味着该文件被赋予了SUID权限。另外有读者会好奇，那么如果原本的权限是rw-呢？如果原先权限位上没有x执行权限，那么被赋予特殊权限后将变成大写的S。

```
[root@linuxprobe ~]# ls -l /etc/shadow
----------. 1 root root 1312 Jul 21 05:08 /etc/shadow
[root@linuxprobe ~]# ls -l /bin/passwd 
-rwsr-xr-x. 1 root root 34512 Aug 13 2018 /bin/passwd
```

### **Tips**

醒目的红色提醒告诫着用户一定要小心这个权限，因为一旦某个命令文件被设置上了SUID权限，那么就意味着凡是执行的人都可以临时获取到更高的权限，千万不要设置到vim、cat、rm等命令上面！！！

**2.  SGID**

SGID特殊权限有两种应用场景，当对二进制程序进行设置时，能够让执行者临时获取到文件所有组的权限；而对目录进行设置时，则是让目录内新创建的文件自动继承该目录原有用户组的名称。

第一种功能是参考SUID而设计的，不同点在于执行程序的用户获取的不再是文件所有者的临时权限，而是获取到文件所属组的权限。举例来说，在早期的Linux系统中，/dev/kmem是一个字符设备文件，用于存储内核程序要访问的数据，权限为：

> **cr--r-----**  1 root system 2, 1 Feb 11 2017  kmem

大家看出问题了吗？除了root管理员或属于system组成员外，所有用户都没有读取该文件的权限。由于在平时我们需要查看系统的进程状态，为了能够获取到进程的状态信息，可在用于查看系统进程状态的ps命令文件上增加SGID特殊权限位。查看ps命令文件的属性信息：

> -r-xr-**s**r-x  1 bin system 59346 Feb 11 2017  ps

这样一来，由于ps命令被增加了SGID特殊权限位，所以当用户执行该命令时，也就临时获取到了system用户组的权限，从而顺利地读取到了设备文件。

前文提到，每个文件都有其归属的所有者和所属组，当创建或传送一个文件后，这个文件就会自动归属于执行这个操作的用户（即该用户是文件的所有者）。如果现在需要在一个部门内设置共享目录，让部门内的所有人员都能够读取目录中的内容，那么就可以创建部门共享目录后，在该目录上设置SGID特殊权限位。这样，部门内的任何人员在里面创建的任何文件都会归属于该目录的所属组，而不再是自己的基本用户组。此时，用到的就是SGID的第二个功能，即在某个目录中创建的文件自动继承该目录的用户组（只可以对目录进行设置）。

```
[root@linuxprobe ~]# cd /tmp
[root@linuxprobe tmp]# mkdir testdir
[root@linuxprobe tmp]# ls -ald testdir
drwxr-xr-x. 2 root root 6 Oct 27 23:44 testdir
[root@linuxprobe tmp]# chmod -R 777 testdir
[root@linuxprobe tmp]# chmod -R g+s testdir
[root@linuxprobe tmp]# ls -ald testdir
drwxrwsrwx. 2 root root 6 Oct 27 23:44 testdir
```

在使用上述命令设置好目录的777权限（确保普通用户可以向其中写入文件），并为该目录设置了SGID特殊权限位后，就可以切换至一个普通用户，然后尝试在该目录中创建文件，并查看新创建的文件是否会继承新创建的文件所在的目录的所属组名称：

```
[root@linuxprobe tmp]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ cd /tmp/testdir
[linuxprobe@linuxprobe testdir]$ echo "linuxprobe.com" > test
[linuxprobe@linuxprobe testdir]$ ls -al test 
-rw-rw-r--. 1 linuxprobe root 15 Oct 27 23:47 test
```

除了上面提到的SGID的这两个功能，再介绍两个与本小节内容相关的命令：chmod和chown。

chmod命令用于设置文件的一般权限及特殊权限，英文全称为：“change mode”，语法格式为：“chmod [参数] 文件名”。

这是一个与日常设置文件权限强相关的命令，例如要把一个文件的权限设置成其所有者可读可写可执行、所属组可读可写、其他人没有任何权限，则相应的字符法表示为rwxrw----，其对应的数字法表示为760。

```
[root@linuxprobe ~]# ls -l anaconda-ks.cfg 
-rw-------. 1 root root 1407 Jul 21 05:09 anaconda-ks.cfg
[root@linuxprobe ~]# chmod 760 anaconda-ks.cfg 
[root@linuxprobe ~]# ls -l anaconda-ks.cfg 
-rwxrw----. 1 root root 1407 Jul 21 05:09 anaconda-ks.cfg
```

chown命令用于设置文件的所有者和所有组，英文全称为：“change own”，语法格式为：“chown 所有者:所有组 文件名”。

chmod和chown命令是用于修改文件属性和权限的最常用命令，它们还有一个特别的共性，就是针对目录进行操作时需要加上大写参数-R来表示递归操作，即对目录内所有的文件进行整体操作。

使用“所有者:所有组”的格式轻松把刚刚那个文件的所属信息修改一下，变更后的效果如下：

```
[root@linuxprobe ~]# chown linuxprobe:linuxprobe anaconda-ks.cfg 
[root@linuxprobe ~]# ls -l anaconda-ks.cfg 
-rwxrw----. 1 linuxprobe linuxprobe 1407 Jul 21 05:09 anaconda-ks.cfg
```

**3.  SBIT**

现在，大学里的很多老师都要求学生将作业上传到服务器的特定共享目录中，但总是有几个“破坏分子”喜欢删除其他同学的作业，这时就要设置SBIT（Sticky Bit）特殊权限位了（也可以称之为特殊权限位之粘滞位）。SBIT特殊权限位可确保用户只能删除自己的文件，而不能删除其他用户的文件。换句话说，当对某个目录设置了SBIT粘滞位权限后，那么该目录中的文件就只能被其所有者执行删除操作了。

最初不知道是哪位非资深技术人员将Sticky Bit直译成了“粘滞位”，刘遄老师建议将其称为“保护位”，这既好记，又能立刻让人了解它的作用。RHEL 8系统中的/tmp作为一个共享文件的目录，默认已经设置了SBIT特殊权限位，因此除非是该目录的所有者，否则无法删除这里面的文件。

与前面所讲的SUID和SGID权限显示方法不同，当目录被设置SBIT特殊权限位后，文件的其他人权限部分的x执行权限就会被替换成t或者T，原本有x执行权限则会写成t，原本没有x执行权限则会被写成T。

/tmp目录上的SBIT权限默认已经存在，体现为“其他用户”权限字段的权限变为rwt：

```
[root@linuxprobe ~]# ls -ald /tmp
drwxrwxrwt. 17 root root 4096 Oct 28 00:29 /tmp
```

其实，文件能否被删除并不取决于自身的权限，而是看其所在目录是否有写入权限（其原理会在下个章节讲到）。为了避免现在很多读者不放心，所以下面的命令还是赋予了这个test文件最大的777权限（rwxrwxrwx）：

```
[root@linuxprobe ~]# cd /tmp
[root@linuxprobe tmp]# echo "Welcome to linuxprobe.com" > test
[root@linuxprobe tmp]# chmod 777 test
[root@linuxprobe tmp]# ls -al test 
-rwxrwxrwx. 1 root root 26 Oct 29 14:29 test
```

随后，切换到一个普通用户身份下，尝试删除这个其他人创建的文件就会发现，即便读、写、执行权限全开，但是由于SBIT特殊权限位的缘故，依然无法删除该文件：

```
[root@linuxprobe tmp]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ cd /tmp
[linuxprobe@linuxprobe tmp]$ rm -f test
rm: cannot remove 'test': Operation not permitted
```

工作中对特殊权限善加使用，能够实现很多巧妙的功能，chmod命令设置特殊权限时的参数如表5-7所示。

表5-7                     SUID、SGID、SBIT特殊权限的设置参数

| 参数 | 作用         |
| ---- | ------------ |
| u+s  | 设置SUID权限 |
| u-s  | 取消SUID权限 |
| g+s  | 设置SGID权限 |
| g-s  | 取消SGID权限 |
| o+t  | 设置SBIT权限 |
| o-t  | 取消SBIT权限 |



切换回root管理员的身份下，在家目录中创建一个名为linux的新目录，随后设置上SBIT权限吧：

```
[linuxprobe@linuxprobe tmp]$ exit
Logout
[root@linuxprobe tmp]# cd ~
[root@linuxprobe ~]# mkdir linux
[root@linuxprobe ~]# chmod -R o+t linux/
[root@linuxprobe ~]# ls -ld linux/
drwxr-xr-t. 2 root root 6 Feb 11 19:34 linux/
```

上面的o+t参数是在一般权限已经设置完毕的前提下，又新增了一项特殊权限，那如果我们想一般权限和特殊权限一起设置，有什么高效率的方法呢？

其实SUID、SGID与SBIT也有对应的数字法表示，分别即是4、2、1。也就是说777还不是最大权限，满权限应该是7777，第一个数字代表的是特殊权限位。既然知道了数字表示法是由“特殊权限+一般权限”构成的，那就以上面linux目录的权限为例为同学们梳理下计算方法吧。

在“rwxr-xr-t”权限中，最后一位是t，代表该文件的一般权限为“rwxr-xr-x”，并带有SBIT特殊权限。对于读（r）、写（w）、执行（x）权限的数字计算方法大家应该很熟悉了——即755，而SBIT特殊权限位是1，合并后结果为1755。

再增加点难度，如果权限是“rwsrwSr--”呢？首先不要慌，看到大写S代表原先没有执行权限，因此一般权限为“rwxrw-r--”，数字法计算后结果是764。带有的SUID和SGID特殊权限数字法表示是4和2，心算得出结果是6，合并后结果为6764。这个示例确实难度高一些，读者们可以参考学习图5-4的计算过程，搞明白了再往下看。

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%89%B9%E6%AE%8A%E6%9D%83%E9%99%90%E6%95%B0%E5%AD%97%E6%B3%95%E8%AE%A1%E7%AE%97-300x244.jpg)

图5-4 权限的字符表示法转数字表示法

将数字法转换成回字符法难度略微高一些，以5537为例为大家讲解。首先特殊权限的5是由4+1组成的，意味着是有SUID和SBIT。SUID和SGID的写法是原先有执行权限则是小写s，如果没有执行权限则是大写S，而SBIT的写法则是原先有执行权限是小写t，没有执行权限是大写T。一般权限的537进行字符转换后应为“r-x-wxrwx”，然后在此基础上增加SUID和SGID特殊权限，合并后结果是“r-s-wxrwt”。可以参考图5-5所示的计算过程来学习理解。

![第5章 用户身份与文件权限第5章 用户身份与文件权限](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%89%B9%E6%AE%8A%E6%9D%83%E9%99%90%E5%AD%97%E7%AC%A6%E6%B3%95%E8%AE%A1%E7%AE%97-1-300x223.jpg)

图5-5 权限的数字表示法转字符标识法

### Tips

Linux系统中文件的权限位真像北京的房价，寸土寸金，一个占位竟能有这么多含义，工作中一定要小心谨慎。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **5.4 文件的隐藏属性**

Linux系统中的文件除了具备一般权限和特殊权限之外，还有一种隐藏权限，即被隐藏起来的权限，默认情况下不能直接被用户发觉。有用户曾经在生产环境和RHCE考试题目中碰到过明明权限充足但却无法删除某个文件的情况，或者仅能在日志文件中追加内容而不能修改或删除内容，这在一定程度上阻止了黑客篡改系统日志的图谋，因此这种“奇怪”的文件也保障了Linux系统的安全性。

既然叫隐藏权限了，那肯定不能用常规的ls命令就让我们看到它的真面目，专用的设置命令是chattr，专用的查看命令是lsattr。

**1.  chattr命令**

chattr命令用于设置文件的隐藏权限，英文全称为：“change attributes”，语法格式为：“chattr [参数] 文件名称”。

如果想要把某个隐藏功能添加到文件上，则需要在命令后面追加“+参数”，如果想要把某个隐藏功能移出文件，则需要追加“-参数”。chattr命令中可供选择的隐藏权限参数非常丰富，具体如表5-8所示。

表5-8                 chattr命令中的参数及其作用

| 参数 | 作用                                                         |
| ---- | ------------------------------------------------------------ |
| i    | 无法对文件进行修改；若对目录设置了该参数，则仅能修改其中的子文件内容而不能新建或删除文件 |
| a    | 仅允许补充（追加）内容，无法覆盖/删除内容（Append Only）     |
| S    | 文件内容在变更后立即同步到硬盘（sync）                       |
| s    | 彻底从硬盘中删除，不可恢复（用0填充原文件所在硬盘区域）      |
| A    | 不再修改这个文件或目录的最后访问时间（atime）                |
| b    | 不再修改文件或目录的存取时间                                 |
| D    | 检查压缩文件中的错误                                         |
| d    | 使用dump命令备份时忽略本文件/目录                            |
| c    | 默认将文件或目录进行压缩                                     |
| u    | 当删除该文件后依然保留其在硬盘中的数据，方便日后恢复         |
| t    | 让文件系统支持尾部合并（tail-merging）                       |
| x    | 可以直接访问压缩文件中的内容                                 |



为了让读者能够更好地见识隐藏权限的效果，我们先来创建一个普通文件，然后立即尝试删除（这个操作肯定会成功）：

```
[root@linuxprobe ~]# echo "for Test" > linuxprobe
[root@linuxprobe ~]# rm linuxprobe
rm: remove regular file ‘linuxprobe’? y
```

实践是检验真理的唯一标准。如果您没有亲眼见证过隐藏权限强大功能的美妙，就一定不会相信原来Linux系统会如此安全。接下来再次新建一个普通文件，并为其设置不允许删除与覆盖（+a参数）权限，然后再尝试将这个文件删除：

```
[root@linuxprobe ~]# echo "for Test" > linuxprobe
[root@linuxprobe ~]# chattr +a linuxprobe
[root@linuxprobe ~]# rm linuxprobe
rm: remove regular file ‘linuxprobe’? y
rm: cannot remove ‘linuxprobe’: Operation not permitted
```

可见，上述操作失败了。

**2.  lsattr命令**

lsattr命令用于查看文件的隐藏权限，英文全称为：“list attributes”，语法格式为：“lsattr [参数] 文件名称”。

在Linux系统中，文件的隐藏权限必须使用lsattr命令来查看，平时使用的ls之类的命令则看不出端倪：

```
[root@linuxprobe ~]# ls -al linuxprobe
-rw-r--r--. 1 root root 9 Feb 12 11:42 linuxprobe
```

一旦使用lsattr命令后，文件上被赋予的隐藏权限马上就会原形毕露：

```
[root@linuxprobe ~]# lsattr linuxprobe
-----a---------- linuxprobe
```

此时按照显示的隐藏权限的类型（字母），使用chattr命令将其去掉：

```
[root@linuxprobe ~]# chattr -a linuxprobe
[root@linuxprobe ~]# lsattr linuxprobe 
---------------- linuxprobe
[root@linuxprobe ~]# rm linuxprobe 
rm: remove regular file ‘linuxprobe’? y
```

一般我们会将-a参数设置到日志文件（/var/log/messages）上面，这样在不影响系统正常写入日志的前提下，还可以防止骇客清理自己的作案证据。如果希望彻底的保护起某个文件，不允许任何人修改和删除它的话，不妨加上-i参数试试，效果特别好。

在热映美剧《越狱》的第一季中，主人公迈克尔·斯科菲尔德（Scofield）把装有越狱计划的硬盘开窗扔进了湖中，结果在第二季被警探打捞出来恢复了数据，又有了第二、第三、第四、第五季……他和哥哥的逃亡故事。所以想彻底删除某个文件时，可以使用-s参数来保证其被删除后不可恢复，文件的硬盘数据会被用零块重新填充，那就更保险了。

##### **5.5 文件访问控制列表**

不知道大家是否发现，前文讲解的一般权限、特殊权限、隐藏权限其实有一个共性——权限是针对某一类用户设置的，能够对很多人同时生效。如果希望对某个指定的用户进行单独的权限控制，就需要用到文件的访问控制列表（FACL，File Access Control Lists）了。通俗来讲，基于普通文件或目录设置ACL访问控制其实就是针对指定的用户或用户组设置文件或目录的操作权限，更加精准的派发权限。另外，如果针对某个目录设置了ACL，则目录中的文件会继承其权限；若针对文件设置了ACL，则文件不再继承其所在目录的权限。

为了更直观地看到ACL对文件权限控制的强大效果，我们先切换到普通用户，然后尝试进入root管理员的家目录中。在没有针对普通用户对root管理员的家目录设置ACL之前，其执行结果如下所示：

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ cd /root
-bash: cd: /root: Permission denied
[linuxprobe@linuxprobe root]$ exit
```

**1.  setfacl命令**

setfacl命令用于管理文件的ACL权限规则，英文全称为：“set files ACL”，语法格式为：“ setfacl [参数] 文件名称”。

ACL权限提供的是在所有者、所属组、其他人的读/写/执行权限之外的特殊权限控制，使用setfacl命令可以针对单一用户或用户组、单一文件或目录来进行读/写/执行权限的控制。其中，针对目录文件需要使用-R递归参数；针对普通文件则使用-m参数；如果想要删除某个文件的ACL，则可以使用-b参数。setfacl命令的常用参数如表5-9所示：

表5-9                       setfacl命令中的参数以及作用

| 参数 | 作用             |
| ---- | ---------------- |
| -m   | 修改权限         |
| -M   | 从文件中读取权限 |
| -x   | 删除某个权限     |
| -b   | 删除全部权限     |
| -R   | 递归子目录       |



例如我们原本是无法进入到/root目录中的，现在给这位普通用户单独设置下权限吧：

```
[root@linuxprobe ~]# setfacl -Rm u:linuxprobe:rwx /root
```

随后再切换到这位普通用户的身份下，就能正常进入了呢：

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ cd /root
[linuxprobe@linuxprobe root]$ ls
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  Templates
Desktop          Downloads  Music                 Public    Videos
[linuxprobe@linuxprobe root]$ exit
```

是不是觉得效果很酷呢？但是现在有这样一个小问题—怎么去查看文件上有那些ACL呢？常用的ls命令是看不到ACL表信息的，但是却可以看到文件的权限最后一个点（**.**）变成了加号（**+**）,这就意味着该文件已经设置了ACL了。现在大家是不是感觉学得越多，越不敢说自己精通Linux系统了吧？就这么一个不起眼的点（.），竟然还表示这么一种重要的权限。

```
[root@linuxprobe ~]# ls -ld /root
dr-xrwx---+ 14 root root 4096 May 4 2020 /root
```

**2.  getfacl命令**

getfacl命令用于查看文件的ACL权限规则，英文全称为：“get files ACL”，语法格式为：“ getfacl [参数] 文件名称”。

getfacl命令用于显示文件上设置的ACL信息，格式为“getfacl 文件名称”。Linux系统中的命令就是这么又可爱又好记。想要设置ACL，用的是setfacl命令；要想查看ACL，则用的是getfacl命令。下面使用getfacl命令显示在root管理员家目录上设置的所有ACL信息。

```
[root@linuxprobe ~]# getfacl /root
ggetfacl: Removing leading '/' from absolute path names
# file: root
# owner: root
# group: root
user::r-x
user:linuxprobe:rwx
group::r-x
mask::rwx
other::---
```

ACL权限还可以针对某个用户组来设置，例如允许某个组的用户都可以读写/etc/fstab个文件：

```
[root@linuxprobe ~]# setfacl -m g:linuxprobe:rw /etc/fstab
[root@linuxprobe ~]# getfacl /etc/fstab 
getfacl: Removing leading '/' from absolute path names
# file: etc/fstab
# owner: root
# group: root
user::rw-
group::r--
group:linuxprobe:rw-
mask::rw-
other::r--
```

设置错了想删除？没问题！清空所有ACL权限用-b参数，指定删除某一条权限就用-x参数：

```
[root@linuxprobe ~]# setfacl -x g:linuxprobe /etc/fstab
[root@linuxprobe ~]# getfacl /etc/fstab 
getfacl: Removing leading '/' from absolute path names
# file: etc/fstab
# owner: root
# group: root
user::rw-
group::r--
mask::r--
other::r--
```

对于ACL权限的设置都是立即且永久生效的，不需要再编辑什么配置文件，这点特别方便~但是也有个安全隐患，如果我们不小心设置错了权限怎么办？就会覆盖掉文件原始的权限信息，并且永远都找不回来了。

**操作前备份一下，总是好的习惯。**

例如将/home目录上的ACL权限备份一份，要用-R递归参数，这样不仅能够把目录本身的权限备份了，里面的文件权限也都能自动备份，另再外加上第3章节学习过的输出重定向操作就可以实现：

```
[root@linuxprobe ~]# getfacl -R /home > backup.acl
getfacl: Removing leading '/' from absolute path names
[root@linuxprobe ~]# ls
anaconda-ks.cfg  Desktop    Downloads             Music     Public     Videos
backup.acl       Documents  initial-setup-ks.cfg  Pictures  Templates
```

ACL权限的恢复也很简单，使用的是“--restore”参数，由于备份时已经指定了是对/home目录进行的操作，所以不需要写对应的目录名称，它能够自动找到要恢复的对象：

```
[root@linuxprobe /]# setfacl --restore backup.acl
```

##### **5.6 su命令与sudo服务**

各位读者在实验环境中很少遇到安全问题，并且为了避免因权限因素导致配置服务失败，从而建议使用root管理员来学习本书，但是在生产环境中还是要对安全多一份敬畏之心，不要用root管理员去做所有事情。因为一旦执行了错误的命令，可能会直接导致系统崩溃，这样一来，不但客户指责、领导批评，没准奖金也会鸡飞蛋打。但转头一想，尽管Linux系统为了安全性考虑，使得许多系统命令和服务只能被root管理员来使用，但是这也让普通用户受到了更多的权限束缚，从而导致无法顺利完成特定的工作任务。

su命令可以解决切换用户身份的需求，使得当前用户在不退出登录的情况下，顺畅地切换到其他用户，比如从root管理员切换至普通用户：

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ id
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

细心的读者一定会发现，上面的su命令与用户名之间有一个减号（-），这意味着完全切换到新的用户，即把环境变量信息也变更为新用户的相应信息，而不是保留原始的信息。强烈建议在切换用户身份时添加这个减号（-）。

另外，当从root管理员切换到普通用户时是不需要密码验证的，而从普通用户切换成root管理员就需要进行密码验证了；这也是一个必要的安全检查：

```
[linuxprobe@linuxprobe ~]$ su - root
Password: 此处输入管理员密码
[root@linuxprobe ~]# 
```

尽管像上面这样使用su命令后，普通用户可以完全切换到root管理员身份来完成相应工作，但这将暴露root管理员的密码，从而增大了系统密码被黑客获取的几率；这并不是最安全的方案。

刘遄老师接下来将介绍如何使用sudo命令把特定命令的执行权限赋予给指定用户，这样既可保证普通用户能够完成特定的工作，也可以避免泄露root管理员密码。我们要做的就是合理配置sudo服务，以便兼顾系统的安全性和用户的便捷性。

### **Tips**

**授权原则：**在保证普通用户完成相应工作的前提下，尽可能少地赋予额外的权限。

sudo命令用于给普通用户提供额外的权限，语法格式为：“ sudo [参数] 用户名”。

使用sudo命令可以给普通用户提供额外的权限来完成原本只有root管理员才能完成的任务。可以限制用户执行指定的命令、记录用户执行过的每一条命令、集中的管理用户与权限（/etc/sudoers）以及验证密码后一段时间内免验证的方便措施。常见的sudo命令可用参数如表5-10所示：

表5-10                     sudo命令中的可用参数以及作用

| 参数             | 作用                                                   |
| ---------------- | ------------------------------------------------------ |
| -h               | 列出帮助信息                                           |
| -l               | 列出当前用户可执行的命令                               |
| -u 用户名或UID值 | 以指定的用户身份执行命令                               |
| -k               | 清空密码的有效时间，下次执行sudo时需要再次进行密码验证 |
| -b               | 在后台执行指定的命令                                   |
| -p               | 更改询问密码的提示语                                   |



当然，如果担心直接修改配置文件会出现问题，则可以使用sudo命令提供的visudo命令来配置用户权限。

visudo命令用于编辑配置用户sudo权限文件，语法格式为：“ visudo [参数]”。

这是一条会自动调用vi编辑器来配置/etc/sudoers权限文件的命令，能够解决多个用户同时修改权限而导致的冲突问题，不仅如此，visudo命令还可以对配置文件内的参数进行语法检查，在发现参数错误时进行报错提醒，比用户直接修改文件更友好、安全、方便。

```
>>> /etc/sudoers: syntax error near line 1 <<<
What now? 
Options are:
(e)dit sudoers file again
e(x)it without saving changes to sudoers file
(Q)uit and save changes to sudoers file (DANGER!)
```

使用visudo命令配置权限文件时，其操作方法与Vim编辑器中用到的方法完全一致，因此在编写完成后记得在末行模式下保存并退出。配置权限文件时，按照下面的格式将第101行（大约）填写上指定的信息：

> **谁可以使用 允许使用的主机=(以谁的身份) 可执行命令的列表**
>
> **谁可以使用：**稍后要为那位用户进行命令授权。
>
> **允许使用的主机：**可以填写ALL代表不限制来源主机，亦可填写如192.168.10.0/24的网段限制来源地址，只有从允许网段登录时才能使用sudo命令。
>
> **以谁的身份：**可以填写ALL代表系统最高权限，也可以是另外一位用户的名字。
>
> **可执行命令的列表：**可以填写ALL代表不限制命令的列表，亦可填写如/usr/bin/cat的文件名称来限制命令列表，多个命令文件之间用逗号（,）间隔。

在Linux系统中配置服务文件时，虽然没有硬性规定，但从经验来讲新增的参数位置不建议太靠上，避免服务一些必要的功能没加载完成时，我们填写的新参数不被执行成功。一般会在配置文件中找一下相似的参数，然后在相邻位置进行新的修改，或者选择文件的中下部位置。

```
[root@linuxprobe ~]# visudo
 99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) ALL
```

在填写完毕后记得要先保存再退出，然后切换至指定的普通用户身份，此时就可以用sudo -l命令查看到所有可执行的命令了（下面的命令中，验证的是该普通用户的密码，而不是root管理员的密码，请读者不要搞混了）：

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ sudo -l
We trust you have received the usual lecture from the local System
Administrator. It usually boils down to these three things:

    #1) Respect the privacy of others.
    #2) Think before you type.
    #3) With great power comes great responsibility.

[sudo] password for linuxprobe: 此处输入linuxprobe用户的密码
Matching Defaults entries for linuxprobe on localhost:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin

User linuxprobe may run the following commands on localhost:
    (ALL) ALL
```

接下来是见证奇迹的时刻！作为一名普通用户，是肯定不能看到root管理员的家目录（/root）中的文件信息的，但是，只需要在想执行的命令前面加上sudo命令就行了：

```
[linuxprobe@linuxprobe ~]$ ls /root
ls: cannot open directory '/root': Permission denied
[linuxprobe@linuxprobe ~]$ sudo ls /root
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  Templates
Desktop		 Downloads  Music		  Public    Videos
```

效果立竿见影！但是考虑到生产环境中不允许某个普通用户拥有整个系统中所有命令的最高执行权（这也不符合前文提到的权限赋予原则，即尽可能少地赋予权限），因此ALL参数就有些不合适了。因此只能赋予普通用户具体的命令以满足工作需求，这也受到了必要的权限约束。如果需要让某个用户只能使用root管理员的身份执行指定的命令，切记一定要给出该命令的绝对路径，否则系统会识别不出来，这时可以先使用whereis命令找出命令所对应的保存路径：

```
[linuxprobe@linuxprobe ~]$ exit 
logout 
[root@linuxprobe ~]# whereis cat 
cat: /usr/bin/cat /usr/share/man/man1/cat.1.gz /usr/share/man/man1p/cat.1p.gz
[root@linuxprobe ~]# whereis reboot
reboot: /usr/sbin/reboot /usr/share/man/man2/reboot.2.gz /usr/share/man/man8/reboot.8.gz
```

然后使用visudo命令继续编辑权限文件，将原先101行所新增的参数作如下修改，多个命令之间用逗号（,）间隔。

```
[root@linuxprobe ~]# visudo
 99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) /usr/bin/cat,/usr/sbin/reboot
```

在编辑好后依然是先保存再退出。再次切换到指定的普通用户，然后尝试正常查看某个系统文件的内容，此时系统提示没有权限（Permission denied）。再使用sudo命令就能顺利地查看文件内容了：

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ cat /etc/shadow
cat: /etc/shadow: Permission denied
[linuxprobe@linuxprobe ~]$ sudo cat /etc/shadow
[sudo] password for linuxprobe: 此处输入linuxprobe用户的密码
root:$6$tTbuw5DkOPYqq.vI$RMk9FCGHoJOq2qAPRURTQm.Qok2nN3yFn/i4f/falVGgGND9XoiYFbrxDn16WWiziaSJ0/cR06U66ipEoGLPJ.::0:99999:7:::
bin:*:17784:0:99999:7:::
daemon:*:17784:0:99999:7:::
adm:*:17784:0:99999:7:::
lp:*:17784:0:99999:7:::
sync:*:17784:0:99999:7:::
shutdown:*:17784:0:99999:7:::
halt:*:17784:0:99999:7:::
………………省略部分输出信息………………
[linuxprobe@linuxprobe ~]$ exit 
logout
```

大家千万不要以为到这里就结束了，刘遄老师还有更压箱底的宝贝。不知大家是否发觉在每次执行sudo命令后都会要求验证一下密码。虽然这个密码就是当前登录用户的密码，但是每次执行sudo命令都要输入一次密码其实也挺麻烦的，这时可以添加NOPASSWD参数，使得用户下次再执行sudo命令时就不用密码验证啦~

```
[root@linuxprobe ~]# visudo
 99 ## Allow root to run any commands anywhere
100 root ALL=(ALL) ALL
101 linuxprobe ALL=(ALL) NOPASSWD:/usr/bin/cat,/usr/sbin/reboot
```

这样，当切换到普通用户后再执行命令时，就不用再频繁地验证密码了，我们在日常工作中也就痛快至极了。

```
[root@linuxprobe ~]# su - linuxprobe
[linuxprobe@linuxprobe ~]$ reboot
User root is logged in on tty2.
Please retry operation after closing inhibitors and logging out other users.
Alternatively, ignore inhibitors and users with 'systemctl reboot -i'.
[linuxprobe@linuxprobe ~]$ sudo reboot
```

同学们请细心留意上面的用户身份变换，visudo命令只有root管理员才可以执行，普通用户使用会提示权限不足。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．在RHEL 8系统中，root管理员是谁？

**答：**是UID为0的用户，是权限最大、限制最小的管理员。

2．如何使用Linux系统的命令行来添加或删除用户？

**答：**添加和删除用户的命令分别是useradd与userdel。

3．若某个文件的所有者具有文件的读/写/执行权限，其余人仅有读权限，那么用数字法表示应该是什么?

**答：**所有者权限为rwx，所属组和其他人的权限为r--，因此数字法表示应该是744。

4．某文件的字符权限为rwxrw-r--，那么对应的数字法应该为？

**答：**数字法权限应该为764。

5．某链接文件的权限用数字法表示为755，那么相应的字符法表示是什么呢？

**答：**在Linux系统中，不同文件具有不同的类型，因此这里应写成lrwxr-xr-x。

6．如果希望用户执行某命令时临时拥有该命令所有者的权限，应该设置什么特殊权限？

**答：**特殊权限中的SUID。

7．若对文件设置了隐藏权限+i，则意味着什么？

**答：**无法对文件进行修改；若对目录设置了该参数，则仅能修改其中的子文件内容而不能新建或删除文件。

8．使用访问控制列表（ACL）来限制linuxprobe用户组，使得该组中的所有成员不得在/tmp目录中写入内容。

**答：**想要设置用户组的ACL，则需要把u改成g，即setfacl -Rm g:linuxprobe:r-x /tmp。

9．当普通用户使用sudo命令时是否需要验证密码？

**答：**系统在默认情况下需要验证当前登录用户的密码，若不想要验证，可添加NOPASSWD参数。 

# [第6章 存储结构与管理硬盘](https://www.linuxprobe.com/basic-learning-06.html)

**章节简述：**

[Linux系统](https://www.linuxprobe.com/)中颇具特色的文件存储结构常常搞得新手头晕脑胀，本章将从[Linux](https://www.linuxprobe.com/)系统中的文件存储结构开始讲起，了解FHS文件系统层次化标准、udev硬件命名规则以及硬盘设备原理。

为了让读者更好地理解文件系统的作用，[刘遄](https://www.linuxprobe.com/)老师将在本章中详细地分析Linux系统中最常见的Ext3、Ext4与XFS文件系统的不同之处，并带领各位读者着重练习硬盘设备分区、格式化以及挂载等常用的硬盘管理操作，以便熟练掌握文件系统的使用方法。

在打下坚实的理论基础与完成一些相关的实践练习后，我们将进一步完整地部署SWAP交换分区、配置quota磁盘配额服务、VDO虚拟数据优化技术以及掌握ln[命令](https://www.linuxcool.com/)带来的软硬链接。相信各位读者在学习完本章后，会对Linux系统以及Windows系统中的磁盘存储以及文件系统有深入的理解。

本章目录结构

- [6.1 一切从“/”开始](https://www.linuxprobe.com/basic-learning-06.html#61)
- [6.2 物理设备的命名规则](https://www.linuxprobe.com/basic-learning-06.html#62)
- [6.3 文件系统与数据资料](https://www.linuxprobe.com/basic-learning-06.html#63)
- [6.4 挂载硬件设备](https://www.linuxprobe.com/basic-learning-06.html#64)
- [6.5 添加硬盘设备](https://www.linuxprobe.com/basic-learning-06.html#65)
- [6.6 添加交换分区](https://www.linuxprobe.com/basic-learning-06.html#66)
- [6.7 磁盘容量配额](https://www.linuxprobe.com/basic-learning-06.html#67)
- [6.8 VDO虚拟数据优化](https://www.linuxprobe.com/basic-learning-06.html#68_VDO)
- [6.9 软硬方式链接](https://www.linuxprobe.com/basic-learning-06.html#69)

##### **6.1 一切从“/”开始**

在Linux系统中，目录、字符设备、套接字、硬盘、光驱、打印机等都会被抽象成了文件形式，即[刘遄](https://www.linuxprobe.com/)老师所一直强调的“Linux系统中一切都是文件”。既然平时我们打交道的都是文件，那么又应该如何找到它们呢？在Windows操作系统中，想要找到一个文件，要依次进入该文件所在的磁盘分区（也叫盘符），然后再进入该分区下的具体目录，最终找到这个文件。但是在Linux系统中并不存在C/D/E/F等盘符，Linux系统中的一切文件都是从“根（/）”目录开始的，并按照文件系统层次化标准（FHS：Filesystem Hierarchy Standard）采用倒树状结构来存放文件，以及定义了常见目录的用途。

另外，Linux系统中的文件和目录名称是严格区分大小写的。例如，root、rOOt、Root、rooT均代表不同的目录，并且文件名称中不得包含斜杠（/）。Linux系统中的文件存储结构如图6-1所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2015/02/Linux%E5%AD%98%E5%82%A8%E6%9E%B6%E6%9E%84.png)

图6-1 Linux系统中的文件存储结构

前文提到的FHS是根据以往无数Linux系统用户和开发者的经验而总结出来的，是用户在Linux系统中存储文件时需要遵守的规则，用于指导用户应该把文件保存到什么位置，以及告诉用户应该在何处找到所需的文件。但是，FHS对于用户来讲只能算是一种道德上的约束，有些用户就是懒得遵守，依然会把文件到处乱放，有些甚至从来没有听说过它。这里并不是号召各位读者去谴责他们，而是建议大家要灵活运用所学的知识，千万不要认准这个FHS协定只讲死道理，不然吃亏的可就是自己了。在Linux系统中，最常见的目录以及所对应的存放内容如表6-1所示。

表6-1                 Linux系统中常见的目录名称以及相应内容

| 目录名称    | 应放置文件的内容                                             |
| ----------- | ------------------------------------------------------------ |
| /boot       | 开机所需文件—内核、开机菜单以及所需配置文件等                |
| /dev        | 以文件形式存放任何设备与接口                                 |
| /etc        | 配置文件                                                     |
| /home       | 用户主目录                                                   |
| /bin        | 存放单用户模式下还可以操作的[命令](https://www.linuxcool.com/) |
| /lib        | 开机时用到的函数库，以及/bin与/sbin下面的命令要调用的函数    |
| /sbin       | 开机过程中需要的命令                                         |
| /media      | 用于挂载设备文件的目录                                       |
| /opt        | 放置第三方的软件                                             |
| /root       | 系统管理员的家目录                                           |
| /srv        | 一些网络服务的数据文件目录                                   |
| /tmp        | 任何人均可使用的“共享”临时目录                               |
| /proc       | 虚拟文件系统，例如系统内核、进程、外部设备及网络状态等       |
| /usr/local  | 用户自行安装的软件                                           |
| /usr/sbin   | Linux系统开机时不会使用到的软件/命令/[脚本](https://www.linuxcool.com/) |
| /usr/share  | 帮助与说明文件，也可放置共享文件                             |
| /var        | 主要存放经常变化的文件，如日志                               |
| /lost+found | 当文件系统发生错误时，将一些丢失的文件片段存放在这里         |



在Linux系统中另外还有一个重要的概念—路径。路径指的是如何定位到某个文件，分为绝对路径与相对路径。绝对路径指的是从根目录（/）开始写起的文件或目录名称，而相对路径则指的是相对于当前路径的写法。我们来看下面这个例子，以帮助大家理解。假如有位外国游客来到北京潘家园旅游，当前内急但是找不到洗手间，特意向您问路，那么咱们有两种正确的指路方法。

> **绝对路径（absolute path）:**首先坐飞机来到中国，到了北京出首都机场坐机场快轨到三元桥，然后换乘10号线到潘家园站，出站后坐34路公交车到农光里，下车后路口左转。
>
> **相对路径（relative path）:**前面路口左转。

这两种方法都正确。如果您说的是绝对路径，那么任何一位外国游客都可以按照这个提示找到潘家园的洗手间，但是太繁琐了。如果说的是相对路径，虽然表达很简练，但是这位外国游客只能从当前位置（不见得是潘家园）出发找到洗手间，因此并不能保证在前面的路口左转后可以找到洗手间，由此可见，相对路径不具备普适性。

如果各位读者现在还是不能理解相对路径和绝对路径的区别，也不要着急，以后通过实践练习肯定可以彻底搞明白。当前建议大家先记住FHS中规范的目录作用，这将在以后派上用场。

##### **6.2 物理设备的命名规则**

在Linux系统中一切都是文件，硬件设备也不例外。既然是文件，就必须有文件名称。系统内核中的udev设备管理器会自动把硬件名称规范起来，目的是让用户通过设备文件的名字可以猜出设备大致的属性以及分区信息等；这对于陌生的设备来说特别方便。另外，udev设备管理器的服务会一直以守护进程的形式运行并侦听内核发出的信号来管理/dev目录下的设备文件。Linux系统中常见的硬件设备的文件名称如表6-2所示。

表6-2                       常见的硬件设备及其文件名称

| 硬件设备      | 文件名称           |
| ------------- | ------------------ |
| IDE设备       | /dev/hd[a-d]       |
| SCSI/SATA/U盘 | /dev/sd[a-z]       |
| virtio设备    | /dev/vd[a-z]       |
| 软驱          | /dev/fd[0-1]       |
| 打印机        | /dev/lp[0-15]      |
| 光驱          | /dev/cdrom         |
| 鼠标          | /dev/mouse         |
| 磁带机        | /dev/st0或/dev/ht0 |



由于现在的IDE设备已经很少见了，所以一般的硬盘设备都会是以“/dev/sd”开头的。而一台主机上可以有多块硬盘，因此系统采用a～z来代表24块不同的硬盘（默认从a开始分配），而且硬盘的分区编号也很有讲究：

> 主分区或扩展分区的编号从1开始，到4结束；
>
> 逻辑分区从编号5开始。

国内很多Linux培训讲师以及很多知名Linux图书在讲到设备和分区名称时，总会讲错两个知识点。第一个知识点是设备名称的理解错误。很多培训讲师和Linux技术图书中会提到，比如/dev/sda表示主板上第一个插槽上的存储设备，学员或读者在实践操作的时候会发现果然如此，因此也就对这条理论知识更加深信不疑。但真相不是这样的，/dev目录中sda设备之所以是a，并不是由插槽决定的，而是由系统内核的识别顺序来决定的，而恰巧很多主板的插槽顺序就是系统内核的识别顺序，因此才会被命名为/dev/sda。大家以后在使用iSCSI网络存储设备时就会发现，明明主板上第二个插槽是空着的，但系统却能识别到/dev/sdb这个设备就是这个道理。

第二个知识点是对分区名称的理解错误。很多Linux培训讲师会告诉学员，分区的编号代表分区的个数。比如sda3表示这是设备上的第三个分区，而学员在做实验的时候确实也会得出这样的结果，但是这个理论知识是错误的，因为分区的数字编码不一定是强制顺延下来的，也有可能是手工指定的。因此sda3只能表示是编号为3的分区，而不能判断sda设备上已经存在了3个分区。

在填了这两个“坑”之后，再来分析一下/dev/sda5这个设备文件名称包含哪些信息，如图6-2所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2015/02/%E7%A1%AC%E7%9B%98%E5%91%BD%E5%90%8D%E8%A7%84%E5%88%99.png)

图6-2 设备文件名称

首先，/dev/目录中保存的应当是硬件设备文件；其次，sd表示是存储设备；然后，a表示系统中同类接口中第一个被识别到的设备，最后，5表示这个设备是一个逻辑分区。一言以蔽之，“/dev/sda5”表示的就是“这是系统中第一块被识别到的硬件设备中分区编号为5的逻辑分区的设备文件”。考虑到很多读者完全没有Linux基础，不太容易理解前面所说的主分区、扩展分区和逻辑分区的概念，因此接下来简单科普一下硬盘相关的知识。

正是因为计算机有了硬盘设备，我们才能够在玩游戏的过程中或游戏通关之后随时存档，而不用每次重头开始。硬盘设备是由大量的扇区组成的，每个扇区的容量为512字节。其中第一个扇区最重要，它里面保存着主引导记录与分区表信息。就第一个扇区来讲，主引导记录需要占用446字节，分区表为64字节，结束符占用2字节；其中分区表中每记录一个分区信息就需要16字节，这样一来最多只有4个分区信息可以写到第一个扇区中，这4个分区就是4个主分区。第一个扇区中的数据信息如图6-3所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E7%AC%AC%E4%B8%80%E4%B8%AA%E6%89%87%E5%8C%BA%E4%B8%AD%E7%9A%84%E6%95%B0%E6%8D%AE%E4%BF%A1%E6%81%AF-1.jpg)

图6-3 第一个扇区中的数据信息

现在，问题来了——每块硬盘最多只能创建出4个分区？明显不够用也不合情理。

于是为了解决分区个数不够的问题，可以将第一个扇区的分区表中16字节（原本要写入主分区信息）的空间（称之为扩展分区）拿出来指向另外一个分区。也就是说，扩展分区其实并不是一个真正的分区，而更像是一个占用16字节分区表空间的指针—一个指向另外一个分区的指针。这样一来，用户一般会选择使用3个主分区加1个扩展分区的方法，然后在扩展分区中创建出数个逻辑分区，从而来满足多分区（大于4个）的需求。当然，就目前来讲大家只要明白为什么主分区不能超过4个就足够了。主分区、扩展分区、逻辑分区可以像图6-4那样来规划。

### **Tips**

所谓扩展分区，严格地讲它不是一个实际意义的分区，它仅仅是一个指向其它分区的指针，这种指针结构将形成一个单向链表。因此扩展分区自身不能够存储数据，用户需要在其指向的对应分区上进行操作，称之为逻辑分区。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2015/02/%E9%80%BB%E8%BE%91%E5%88%86%E5%8C%BA.png)

图6-4 硬盘分区的规划

**读者们来试着解读下/dev/hdc8代表着什么？**（**答案模式**）

**答案：**这是第三块IDE设备（比较少见了）中的编号为8的逻辑分区。

对了！如果参加[红帽](https://www.linuxprobe.com/)RHCE考试或者购买了一台云主机，还会看到类似于/dev/vda、/dev/vdb的设备。这种以vd开头的叫做virtio设备，简单来说就是一种虚拟化设备。像KVM、Xen这种虚拟机监控器（Hypervisor）默认就都是这种设备，等大家工作了可能会见到更多。

##### **6.3 文件系统与数据资料**

同学们可以拿出一张A4纸，横过来，然后在上面随便的写上几行字，慢慢会发现字会写的越来越歪，最后整行的字都有了向上或向下的斜度。为了能字写的更工整、阅读更舒服一些，文具店里就会有各种不同的本本啦——单线本、双线本、田字格本、五线谱等等。离开了格式之后的内容，完全不受我们主观控制。而用户在硬件存储设备中执行的文件建立、写入、读取、修改、转存与控制等操作都是依靠文件系统来完成的。文件系统的作用是合理规划硬盘，以保证用户正常的使用需求。

Linux系统支持数十种的文件系统，而最常见的文件系统如下所示。

**Ext2**：最早可追溯到1993年，是Linux系统第一个商业级文件系统，基本沿袭的是Unix文件系统的设计标准。但由于不包含读写日志功能，数据丢失可能性很大，能不用就不要用，或者顶多建议用于SD存储卡或U盘。

**Ext3**：是一款日志文件系统，它会把整个硬盘的每个写入动作的细节都预先记录下来，然后再实际操作，以便在发生异常宕机后能回溯追踪到被中断的部分。Ext3能够在系统异常宕机时避免文件系统资料丢失，并能自动修复数据的不一致与错误。然而，当硬盘容量较大时，所需的修复时间也会很长，而且也不能百分之百地保证资料不会丢失。

**Ext4**：Ext3的改进版本，作为RHEL 6系统中的默认文件管理系统，它支持的存储容量高达1EB（1EB=1,073,741,824GB），且能够有无限多的子目录。另外，Ext4文件系统能够批量分配block块，从而极大地提高了读写效率。现在很多主流服务器也会使用。

**XFS**：是一种高性能的日志文件系统，而且是RHEL 7/8中默认的文件管理系统，它的优势在发生意外宕机后尤其明显，即可以快速地恢复可能被破坏的文件，而且强大的日志功能只用花费极低的计算和存储性能。并且它最大可支持的存储容量为18EB，这几乎满足了所有需求。

RHEL 7/8系统中一个比较大的变化就是使用了XFS作为文件系统，这不同于RHEL 6使用的Ext4。从红帽公司官方发布的说明来看，这确实是一个不小的进步，但是刘遄老师在实测中发现并不完全属实。因为单纯就测试一款文件系统的“读取”性能来说，到底要读取多少个文件，每个文件的大小是多少，读取文件时的CPU、内存等系统资源的占用率如何，以及不同的硬件配置是否会有不同的影响，因此在充分考虑到这些不确定因素后，实在不敢直接照抄红帽官方的介绍。我个人认为XFS虽然在性能方面比Ext4有所提升，但绝不是压倒性的，因此XFS文件系统最卓越的亮点应该当属可支持高达18EB的存储容量吧。

18EB等于18874368TB，假设每块硬盘容量是100TB，那么大概要一万九千块硬盘才能把数据都装下。总之当用了XFS之后，文件的存储上限就不再取决于技术层面，而是钱包了。过去常常跟同学们开玩笑提问，如果有18EB的数据在上海机房，想以最快的方法传送到北京，我们有什么好办法呢？答案是京沪高铁。

在拿到了一块新的硬盘存储设备后，先需要分区，然后再格式化文件系统，最后才能挂载并正常使用。硬盘的分区操作取决于您的需求和硬盘大小；也可以选择不进行分区，但是必须对硬盘进行格式化处理。

### **Tips**

就像拿到了一张未裁切的完整纸张那样，首先要进行裁切以方便使用（**分区**），接下来在裁切后的纸张上画格以便能书写工整（格式化），最后是正式的使用（**挂载**）。

接下来向大家简单地科普一下硬盘在格式化后发生的事情。再次强调，不用刻意去记住，只要能看懂就行了。

日常在硬盘中需要保存的数据实在太多了，因此Linux系统中有一个名为super block的“硬盘地图”。Linux并不是把文件内容直接写入到这个“硬盘地图”里面，而是在里面记录着整个文件系统的信息。因为如果把所有的文件内容都写入到这里面，它的体积将变得非常大，而且文件内容的查询与写入速度也会变得很慢。Linux只是把每个文件的权限与属性记录在inode中，而且每个文件占用一个独立的inode表格，该表格的大小默认为128字节，里面记录着如下信息：

> 该文件的访问权限（read、write、execute）；
>
> 该文件的所有者与所属组（owner、group）；
>
> 该文件的大小（size）；
>
> 该文件的创建或内容修改时间（ctime）；
>
> 该文件的最后一次访问时间（atime）；
>
> 该文件的修改时间（mtime）；
>
> 文件的特殊权限（SUID、SGID、SBIT）；
>
> 该文件的真实数据地址（point）。

而文件的实际内容则保存在block块中（大小一般是1KB、2KB或4KB），一个inode的默认大小仅为128字节，记录一个block则消耗4字节。当文件的inode被写满后，Linux系统会自动分配出一个block块，专门用于像inode那样记录其他block块的信息，这样把各个block块的内容串到一起，就能够让用户读到完整的文件内容了。对于存储文件内容的block块，有下面两种常见情况（以4KB的block大小为例进行说明）。

> 情况1：文件很小（1KB），但依然会占用一个block，因此会潜在地浪费3KB。
>
> 情况2：文件很大（5KB），那么会占用两个block（5KB-4KB后剩下的1KB也要占用一个block）。

初次听到这种说法，是不是觉得Linux系统好浪费啊？为什么最后一个block块容量总不能被完全使用。其实每个系统都是一样滴，只是大家此前没有留意过罢了，如图6-5所示，同学们可以随手查看一个电脑中已有的文件，看看文件的实际大小与占用空间是否一致。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E6%96%87%E4%BB%B6%E7%9A%84%E5%8D%A0%E7%94%A8%E5%A4%A7%E5%B0%8F.png)

图6-5 文件的实际大小与占用空间

计算机系统在发展过程中产生了众多的文件系统，为了使用户在读取或写入文件时不用关心底层的硬盘结构，Linux内核中的软件层为用户程序提供了一个VFS（Virtual File System，虚拟文件系统）接口，这样用户实际上在操作文件时就是统一对这个虚拟文件系统进行操作了。图6-6所示为VFS的架构示意图。从中可见，实际文件系统在VFS下隐藏了自己的特性和细节，这样用户在日常使用时会觉得“文件系统都是一样的”，也就可以随意使用各种命令在任何文件系统中进行各种操作了（比如使用cp命令来复制文件）。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/VFS%E7%9A%84%E6%9E%B6%E6%9E%84%E7%A4%BA%E6%84%8F%E5%9B%BE-1.jpg)

图6-6 VFS的架构示意图

VFS虚拟文件系统也有点像是一个翻译官，我们不需要知道对方的情况，只要告诉VFS想进行的操作是什么，它就会自动判断对方能够听得懂什么指令，然后翻译并交代下去，让用户不用操心这些“小事情”，专注于自己的操作。

### **Tips**

在医学生圈里有句话说的很好，当你开始关注身体某个器官的时候，大概率它最近不舒服了。由于VFS真的是太好用了，而且几乎不会出现任何的问题，所以如果不讲理论的话，很多同学可能几年后都不知道自己用了它呢~

##### **6.4 挂载硬件设备**

在用惯了Windows系统后总觉得一切都是理所当然的，平时把U盘插入到电脑后也从来没有考虑过Windows系统做了哪些事情，才使得我们可以访问这个U盘的。接下来会逐一学习在Linux系统中挂载和卸载存储设备的方法，以便大家更好地了解Linux系统添加硬件设备的工作原理和流程。前面讲到，在拿到一块全新的硬盘存储设备后要先分区，然后格式化，最后才能挂载并正常使用。“分区”和“格式化”大家以前经常听到，但“挂载”又是什么呢？

刘遄老师在这里给您一个最简单、最贴切的解释—当用户需要使用硬盘设备或分区中的数据时，需要先将其与一个已存在的目录文件进行关联，而这个关联动作就是“挂载”。下文将向读者逐步讲解如何使用硬盘设备，但是鉴于与挂载相关的理论知识比较复杂，而且很重要，因此决定再拿出一个小节单独讲解，这次希望大家不仅要看懂，而且还要记住。

mount命令用于挂载文件系统，格式为“mount 文件系统 挂载目录”。mount命令中可用的参数及作用如表6-3所示。挂载是在使用硬件设备前所执行的最后一步操作。只需使用mount命令把硬盘设备或分区与一个目录文件进行关联，然后就能在这个目录中看到硬件设备中的数据了。对于比较新的Linux系统来讲，一般不需要使用-t参数来指定文件系统的类型，Linux系统会自动进行判断。而mount 中的-a参数则厉害了，它会在执行后自动检查/etc/fstab文件中有无疏漏被挂载的设备文件，如果有，则进行自动挂载操作。

表6-3                       mount命令中的参数以及作用

| 参数 | 作用                                 |
| ---- | ------------------------------------ |
| -a   | 挂载所有在/etc/fstab中定义的文件系统 |
| -t   | 指定文件系统的类型                   |



例如，要把设备/dev/sdb2挂载到/backup目录，只需要在mount命令中填写设备与挂载目录参数就行，系统会自动去判断要挂载文件的类型，命令如下：

```
[root@linuxprobe ~]# mount /dev/sdb2 /backup
```

如果在工作中要挂载的是一块网络存储设备，名字可能会变来变去，再写sdb就不太合适了。这时推荐用UUID（通用唯一识别码）进行挂载操作，这是一串用于标识每块独立硬盘的字符串，具有唯一性及稳定性，特别适合挂载网络设备时使用。

blkid命令用于显示设备的属性信息，英文全称为：“block id”，语法格式为：“blkid [设备名]”。

```
[root@linuxprobe ~]# blkid
/dev/sdb1: UUID="2db66eb4-d9c1-4522-8fab-ac074cd3ea0b" TYPE="xfs" PARTUUID="eb23857a-01"
/dev/sdb2: UUID="478fRb-1pOc-oPXv-fJOS-tTvH-KyBz-VaKwZG" TYPE="ext4" PARTUUID="eb23857a-02"
```

有了设备的UUID值之后，以后就可以用它挂载网络设备了：

```
[root@linuxprobe ~]# mount UUID=478fRb-1pOc-oPXv-fJOS-tTvH-KyBz-VaKwZG /backup
```

虽然按照上面的方法执行mount命令后就能立即使用文件系统了，但系统在重启后挂载就会失效，也就是需要每次开机后都手动挂载一下。这肯定不是大家想要的效果，如果想让硬件设备和目录永久地进行自动关联，就必须把挂载信息按照指定的填写格式“设备文件 挂载目录 格式类型 权限选项 是否备份 是否自检”（各字段的意义见表6-4）写入到/etc/fstab文件中。这个文件中包含着挂载所需的诸多信息项目，一旦配置好之后就能一劳永逸了。

表6-4            用于挂载信息的指定填写格式中，各字段所表示的意义

| 字段     | 意义                                                         |
| -------- | ------------------------------------------------------------ |
| 设备文件 | 一般为设备的路径+设备名称，也可以写唯一识别码（UUID，Universally Unique Identifier） |
| 挂载目录 | 指定要挂载到的目录，需在挂载前创建好                         |
| 格式类型 | 指定文件系统的格式，比如Ext3、Ext4、XFS、SWAP、iso9660（此为光盘设备）等 |
| 权限选项 | 若设置为defaults，则默认权限为：rw, suid, dev, exec, auto, nouser, async |
| 是否备份 | 若为1则开机后使用dump进行磁盘备份，为0则不备份               |
| 是否自检 | 若为1则开机后自动进行磁盘自检，为0则不自检                   |



如果想将文件系统为ext4的硬件设备/dev/sdb2在开机后自动挂载到/backup目录上，并保持默认权限且无需开机自检，就需要在/etc/fstab文件中写入下面的信息，这样在系统重启后也会成功挂载。

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                     /        xfs     defaults    0 0
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot    xfs     defaults    0 0
/dev/mapper/rhel-swap                     swap     swap    defaults    0 0
/dev/sdb2                                 /backup  ext4    defaults    0 0
```

由于后面需要使用系统镜像制作Yum/DNF软件仓库，所以提前把光盘设备挂载到/media/cdrom目录中吧，光盘设备的文件系统格式是iso9660：

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                     /              xfs        defaults       0 0
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot          xfs        defaults       0 0
/dev/mapper/rhel-swap                     swap           swap       defaults       0 0
/dev/sdb2                                 /backup        ext4       defaults       0 0
/dev/cdrom                                /media/cdrom   iso9660    defaults       0 0
```

写入到/etc/fstab文件中的设备信息并不会立即生效，需要使用mount -a参数进行自动挂载：

```
[root@linuxprobe ~]# mount -a
```

df命令用于已挂载的磁盘空间使用情况，英文全称为：“disk free”，语法格式为：“df -h”。

如果想查看一下当前系统中设备的挂载情况，非常推荐大家试试df命令，它不仅能够列出系统中正在被使用的设备有哪些，还可以用-h参数便捷的对存储容量进行“进位”操作，例如遇到像10240K的时候会自动进位写成10M，便于我们的阅读。

```
[root@linuxprobe~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               969M     0  969M   0% /dev
tmpfs                  984M     0  984M   0% /dev/shm
tmpfs                  984M   18M  966M   2% /run
tmpfs                  984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.9G   14G  23% /
/dev/sda1             1014M  152M  863M  15% /boot
/dev/sdb2              480M   20M  460M   4% /backup
tmpfs                  197M   16K  197M   1% /run/user/42
tmpfs                  197M  3.5M  194M   2% /run/user/0
/dev/sr0               6.7G  6.7G     0 100% /media/cdrom
```

让我想想还有没有其他可能的情况呢~

对了！说到网络存储设备，建议您加上_netdev参数。加上后系统会等待联网成功后再尝试挂载这块网络存储设备，避免了开机时间过长或失败的情况，建议第17章节学iSCSI技术时可以用上。

```
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /              xfs       defaults            0 0
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b   /boot          xfs       defaults            0 0
/dev/mapper/rhel-swap                       swap           swap      defaults            0 0
/dev/sdb2                                   /backup        ext4      defaults,_netdev    0 0
/dev/cdrom                                  /media/cdrom   iso9660   defaults            0 0
```

挂载文件系统的目的是为了使用硬件资源，而卸载文件系统就意味不再使用硬件的设备资源；相对应地，挂载操作就是把硬件设备与目录两者进行关联的动作，因此卸载操作只需要说明想要取消关联的设备文件或挂载目录的其中一项即可，一般不需要加其他额外的参数。

umount命令用于卸载设备或文件系统，英文全称为：“un mount”，语法格式为：“umount [设备文件/挂载目录]”。

```
[root@linuxprobe ~]# umount /dev/sdb2
```

如果当前就处在设备所挂载的目录时，系统会提示该设备繁忙，这种情况只需要退出到其他目录再尝试一次就行了，轻松搞定。

```
[root@linuxprobe ~]# cd /media/cdrom/[root@linuxprobe cdrom]# umount /dev/cdromumount: /media/cdrom: target is busy.[root@linuxprobe  cdrom]# cd ~[root@linuxprobe ~]# umount /dev/cdrom[root@linuxprobe ~]#
```

### **Tips**

挂载操作就像两人结为夫妻，双方同时到场，信息一旦被登记到民政局的系统中，再想重婚（重复挂载某设备）可就不行喽~

最后再教给同学们一个小技巧。如果系统中硬盘特别多、分区特别多，有时候真的很乱，都不知道它们都有没有被使用，又做了什么。那就用lsblk命令以树状图的形式列举一下啦。

lsblk命令用于已挂载的磁盘空间使用情况，英文全称为：“list block id”，输入后回车执行即可。

```
[root@linuxprobe ~]# lsblk NAME          MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTsda             8:0    0   20G  0 disk ├─sda1          8:1    0    1G  0 part /boot└─sda2          8:2    0   19G  0 part   ├─rhel-root 253:0    0   17G  0 lvm  /  └─rhel-swap 253:1    0    2G  0 lvm  [SWAP]sr0            11:0    1  6.6G  0 rom  /media/cdrom
```

##### **6.5 添加硬盘设备**

根据前文讲解的与管理硬件设备相关的理论知识，先来理清一下添加硬盘设备的操作思路：首先需要在虚拟机中模拟添加入一块新的硬盘存储设备，然后再进行分区、格式化、挂载等操作，最后通过检查系统的挂载状态并真实地使用硬盘来验证硬盘设备是否成功添加。

鉴于我们不需要为了做这个实验而特意买一块真实的硬盘，而是通过虚拟机软件进行硬件模拟，因此这再次体现出了使用虚拟机软件的好处。具体的操作步骤如下。

第1步：首先把虚拟机系统关机，稍等几分钟会自动返回到虚拟机管理主界面，然后单击“编辑虚拟机设置”选项，在弹出的界面中单击“添加”按钮，新增一块硬件设备，如图6-7所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/1-23-1024x709.png)

图6-7 在虚拟机系统中添加硬件设备

第2步：选择想要添加的硬件类型为“硬盘”，然后单击“下一步”按钮就可以了，这确实没有什么需要进一步解释的，如图6-8所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/2-20.png)

图6-8 选择添加硬件类型

第3步：选择虚拟硬盘的类型为SATA，并单击“下一步”按钮，这样虚拟机中的设备名称过一会儿后应该为/dev/sdb，如图6-9所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E7%A1%AC%E7%9B%98%E7%B1%BB%E5%9E%8B.png)

图6-9 选择硬盘设备类型

第4步：选中“创建新虚拟磁盘”单选按钮，而不是其他选项，再次单击“下一步”按钮，如图6-10所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/4-6.png)

图6-10 选择“创建新虚拟磁盘”选项

第5步：将“最大磁盘大小”设置为默认的20GB。这个数值是限制这台虚拟机所使用的最大硬盘空间，而不是立即将其填满，因此默认20GB就很合适了。单击“下一步”按钮，如图6-11所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/5-5.png)

图6-11 设置硬盘的最大使用空间

第6步：设置磁盘文件的文件名和保存位置（这里采用默认设置即可，无需修改），直接单击“完成”按钮，如图6-12所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E7%A3%81%E7%9B%98%E5%90%8D%E7%A7%B0.png)

图6-12 设置磁盘文件的文件名和保存位置

第7步：将新硬盘添加好后就可以看到设备信息了。这里不需要做任何修改，直接单击“确认”按钮后就可启虚拟机了，如图6-13所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E7%A1%AC%E4%BB%B6%E4%B8%80%E8%A7%88.png)

图6-13 查看虚拟机硬件设置信息

在虚拟机中模拟添加了硬盘设备后就应该能看到抽象成的硬盘设备文件了。按照前文讲解的udev服务命名规则，第二个被识别的SATA设备应该会被保存为/dev/sdb，这个就是硬盘设备文件了。但在开始使用该硬盘之前还需要进行分区操作，例如从中取出一个2GB的分区设备以供后面的操作使用。

fdisk命令用新建、修改及删除磁盘的分区表信息，英文全称为：“format disk”，语法格式为：“fdisk 磁盘名称”。

在Linux系统中，管理硬盘设备最常用的方法就当属fdisk命令了。fdisk命令用于管理磁盘分区，格式为“fdisk [磁盘名称]”，它提供了集添加、删除、转换分区等功能于一身的“一站式分区服务”。不过与前面讲解的直接写到命令后面的参数不同，这条命令的参数（见表6-5）是交互式的，一问一答的形式，因此在管理硬盘设备时特别方便，可以根据需求动态调整。

表6-5                       fdisk命令中的参数以及作用

| 参数 | 作用                   |
| ---- | ---------------------- |
| m    | 查看全部可用的参数     |
| n    | 添加新的分区           |
| d    | 删除某个分区信息       |
| l    | 列出所有可用的分区类型 |
| t    | 改变某个分区的类型     |
| p    | 查看分区表信息         |
| w    | 保存并退出             |
| q    | 不保存直接退出         |



第1步：首先使用fdisk命令来尝试管理/dev/sdb硬盘设备。在看到提示信息后输入参数p来查看硬盘设备内已有的分区信息，其中包括了硬盘的容量大小、扇区个数等信息：

```
[root@linuxprobe ~]# fdisk /dev/sdbWelcome to fdisk (util-linux 2.32.1).Changes will remain in memory only, until you decide to write them.Be careful before using the write command.Device does not contain a recognized partition table.Created a new DOS disklabel with disk identifier 0x88b2c2b0.Command (m for help): pDisk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectorsUnits: sectors of 1 * 512 = 512 bytesSector size (logical/physical): 512 bytes / 512 bytesI/O size (minimum/optimal): 512 bytes / 512 bytesDisklabel type: dosDisk identifier: 0x88b2c2b0
```

第2步：输入参数n尝试添加新的分区。系统会要求您是选择继续输入参数p来创建主分区，还是输入参数e来创建扩展分区。这里输入参数p来创建一个主分区：

```
Command (m for help): nPartition type   p   primary (0 primary, 0 extended, 4 free)   e   extended (container for logical partitions)Select (default p): p
```

第3步：在确认创建一个主分区后，系统要求您先输入主分区的编号。在前文得知，主分区的编号范围是1～4，因此这里输入默认的1就可以了。接下来系统会提示定义起始的扇区位置，这不需要改动，敲击回车键保留默认设置即可，系统会自动计算出最靠前的空闲扇区的位置。最后，系统会要求定义分区的结束扇区位置，这其实就是要去定义整个分区的大小是多少。我们不用去计算扇区的个数，只需要输入+2G即可创建出一个容量为2GB的硬盘分区。

```
Partition number (1-4, default 1): 1First sector (2048-41943039, default 2048): 此处敲击回车即可Last sector, +sectors or +size{K,M,G,T,P} (2048-41943039, default 41943039): +2GCreated a new partition 1 of type 'Linux' and of size 2 GiB.
```

第4步：再次使用参数p来查看硬盘设备中的分区信息。果然就能看到一个名称为/dev/sdb1、起始扇区位置为2048、结束扇区位置为4196351的主分区了。这时候千万不要直接关闭窗口，而应该敲击参数w后回车，这样分区信息才是真正的写入成功啦。

```
Command (m for help): pDisk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectorsUnits: sectors of 1 * 512 = 512 bytesSector size (logical/physical): 512 bytes / 512 bytesI/O size (minimum/optimal): 512 bytes / 512 bytesDisklabel type: dosDisk identifier: 0x88b2c2b0Device     Boot Start     End Sectors Size Id Type/dev/sdb1        2048 4196351 4194304   2G 83 LinuxCommand (m for help): wThe partition table has been altered.Calling ioctl() to re-read partition table.Syncing disks.
```

分区信息中第六个字段的Id值代表标识该分区作用的编码，帮助用户快速了解该分区的作用，一般没必要修改。使用l参数查看都有哪些的磁盘编码，然后下一个章节做SWAP时再修改吧：

```
Command (m for help): l   0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris         1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT- 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT- 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden or  c6  DRDOS/sec (FAT- 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx          5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data     6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / . 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility    8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt          9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access      a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O         b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor       c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         eb  BeOS fs         f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ee  GPT            10  OPUS            55  EZ-Drive        a7  NeXTSTEP        ef  EFI (FAT-12/16/11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f0  Linux/PA-RISC b12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f1  SpeedStor      14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f4  SpeedStor      16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      f2  DOS secondary  17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fb  VMware VMFS    18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fc  VMware VMKCORE 1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fd  Linux raid auto1c  Hidden W95 FAT3 75  PC/IX           bc  Acronis FAT32 L fe  LANstep        1e  Hidden W95 FAT1 80  Old Minix       be  Solaris boot    ff  BBT   
```

第5步：在上述步骤执行完毕之后，Linux系统会自动把这个硬盘主分区抽象成/dev/sdb1设备文件。可以使用file命令查看该文件的属性，但是在讲课和工作中发现，有些时候系统并没有自动把分区信息同步给Linux内核，而且这种情况似乎还比较常见（但不能算作是严重的bug）。我们可以输入partprobe命令手动将分区信息同步到内核，而且一般推荐连续两次执行该命令，效果会更好。如果使用这个命令都无法解决问题，那么就重启计算机吧，这个杀手锏百试百灵，一定会有用的。

```
[root@linuxprobe ]# file /dev/sdb1/dev/sdb1: cannot open `/dev/sdb1' (No such file or directory)[root@linuxprobe ]# partprobe[root@linuxprobe ]# partprobe[root@linuxprobe ]# file /dev/sdb1/dev/sdb1: block special
```

如果硬件存储设备没有进行格式化，则Linux系统无法得知怎么在其上写入数据。因此，在对存储设备进行分区后还需要进行格式化操作。在Linux系统中用于格式化操作的命令是mkfs。这条命令很有意思，因为在Shell终端中输入mkfs名后再敲击两下用于补齐命令的Tab键，会有如下所示的效果：

```
[root@linuxprobe ~]# mkfs
mkfs         mkfs.ext2    mkfs.ext4    mkfs.minix   mkfs.vfat    
mkfs.cramfs  mkfs.ext3    mkfs.fat     mkfs.msdos   mkfs.xfs     
```

对！这个mkfs命令很贴心地把常用的文件系统名称用后缀的方式保存成了多个命令文件，用起来也非常简单—mkfs.文件类型名称。例如要格式分区为XFS的文件系统，则命令应为mkfs.xfs /dev/sdb1。

```
[root@linuxprobe ~]# mkfs.xfs /dev/sdb1
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

终于完成了存储设备的分区和格式化操作，接下来就是要来挂载并使用存储设备了。与之相关的步骤也非常简单：首先是创建一个用于挂载设备的挂载点目录；然后使用mount命令将存储设备与挂载点进行关联；最后使用df -h命令来查看挂载状态和硬盘使用量信息。

```
[root@linuxprobe ~]# mkdir /newFS
[root@linuxprobe ~]# mount /dev/sdb1 /newFS
[root@linuxprobe ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               969M     0  969M   0% /dev
tmpfs                  984M     0  984M   0% /dev/shm
tmpfs                  984M  9.6M  974M   1% /run
tmpfs                  984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.9G   14G  23% /
/dev/sr0               6.7G  6.7G     0 100% /media/cdrom
/dev/sda1             1014M  152M  863M  15% /boot
tmpfs                  197M   16K  197M   1% /run/user/42
tmpfs                  197M  3.5M  194M   2% /run/user/0
/dev/sdb1              2.0G   47M  2.0G   3% /newFS
```

du命令用查看分区或目录所占用的磁盘容量大小，英文全称为：“Disk Usage”，语法格式为：“du -sh 目录名称”。

既然存储设备已经顺利挂载，接下来就可以尝试通过挂载点目录向存储设备中写入文件了。在写入文件之前，先介绍一个用于查看文件数据占用量的du命令，其格式为“du [选项] [文件]”。简单来说，该命令就是用来查看一个或多个文件占用了多大的硬盘空间。

在使用Window系统时，我们总会遇到C盘容量不足，清理垃圾后又很快被占满的情况。在Linux系统中可以使用du -sh /*命令来查看在Linux系统根目录下所有一级目录分别占用的空间大小，一秒钟找到是那个小坏蛋目录占用的空间最多：

```
[root@linuxprobe ~]# du -sh /*0	/bin113M	/boot0	/dev29M	/etc12K	/home0	/lib0	/lib646.7G	/media0	/mnt0	/newFS0	/opt0	/proc8.6M	/root9.6M	/run0	/sbin0	/srv0	/sys12K	/tmp3.5G	/usr155M	/var
```

先从某些目录中复制过来一批文件，然后查看这些文件总共占用了多大的容量：

```
[root@linuxprobe ~]# cp -rf /etc/* /newFS[root@linuxprobe ~]# ls /newFSadjtime                     hostname                  profile.daliases                     hosts                     protocolsalsa                        hosts.allow               pulsealternatives                hosts.deny                qemu-gaanacrontab                  hp                        qemu-kvmasound.conf                 idmapd.conf               radvd.conf………………省略部分输入信息………………[root@linuxprobe ~]# du -sh /newFS39M /newFS/
```

细心的读者一定还记得，前面在讲解mount命令时提到，使用mount命令挂载的设备文件会在系统下一次重启的时候失效。如果想让这个设备文件的挂载永久有效，则需要把挂载的信息写入到配置文件中：

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                     /                      xfs      defaults        0 0UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot                  xfs      defaults        0 0/dev/mapper/rhel-swap                     swap                   swap     defaults        0 0/dev/cdrom                                /media/cdrom           iso9660  defaults        0 0 /dev/sdb1                                 /newFS                 xfs      defaults        0 0 
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **6.6 添加交换分区**

SWAP交换分区是一种通过在硬盘中预先划分一定的空间，然后将把内存中暂时不常用的数据临时存放到硬盘中，以便腾出物理内存空间让更活跃的程序服务来使用的技术，其设计目的是为了解决真实物理内存不足的问题。通俗来讲就是让硬盘帮内存分担压力。但由于交换分区毕竟是通过硬盘设备读写数据的，速度肯定要比物理内存慢，所以只有当真实的物理内存耗尽后才会调用交换分区的资源。

交换分区的创建过程与前文讲到的挂载并使用存储设备的过程非常相似。在对/dev/sdb存储设备进行分区操作前，有必要先说一下交换分区的划分建议：在生产环境中，交换分区的大小一般为真实物理内存的1.5～2倍，为了让大家更明显地感受交换分区空间的变化，这里取出一个大小为5GB的主分区作为交换分区资源：

```
[root@linuxprobe ~]# fdisk /dev/sdbWelcome to fdisk (util-linux 2.32.1).Changes will remain in memory only, until you decide to write them.Be careful before using the write command.Command (m for help): nPartition type   p   primary (1 primary, 0 extended, 3 free)   e   extended (container for logical partitions)Select (default p): pPartition number (2-4, default 2): 敲击回车即可First sector (4196352-41943039, default 4196352): 敲击回车即可Last sector, +sectors or +size{K,M,G,T,P} (4196352-41943039, default 41943039): +5GCreated a new partition 2 of type 'Linux' and of size 5 GiB.
```

上面操作结束后，我们就得到了一个容量为5G的新分区，试试修改下硬盘的标识码吧~改成82(Linux swap)方便以后知道它的作用：

```
Command (m for help): tPartition number (1,2, default 2): 2Hex code (type L to list all codes): 82Changed type of partition 'Linux' to 'Linux swap / Solaris'.Command (m for help): p Disk /dev/sdb: 20 GiB, 21474836480 bytes, 41943040 sectorsUnits: sectors of 1 * 512 = 512 bytesSector size (logical/physical): 512 bytes / 512 bytesI/O size (minimum/optimal): 512 bytes / 512 bytesDisklabel type: dosDisk identifier: 0x88b2c2b0Device     Boot   Start      End  Sectors Size Id Type/dev/sdb1          2048  4196351  4194304   2G 83 Linux/dev/sdb2       4196352 14682111 10485760   5G 82 Linux swap / Solaris
```

搞定，敲击w参数退出分区表编辑工具：

```
Command (m for help): wThe partition table has been altered.Calling ioctl() to re-read partition table.Syncing disks.
```

mkswap命令用于对新设备做交换分区格式化，英文全称为：“make swap”，语法格式为：“mkswap 设备名称”。

```
[root@linuxprobe ~]# mkswap /dev/sdb2Setting up swapspace version 1, size = 5 GiB (5368705024 bytes)no label, UUID=45a4047c-49bf-4c88-9b99-f6ac93908485
```

swapon命令用于激活新的交换分区设备，英文全称为：“swap on”，语法格式为：“swapon设备名称”。

使用swapon命令把准备好的SWAP硬盘设备正式挂载到系统中，读者可以使用free -m命令查看交换分区的大小变化（由2047MB增加到7167MB）：

```
[root@linuxprobe ~]# free -m              total        used        free      shared  buff/cache   availableMem:           1966        1391         105          12         469         384Swap:          2047           9        2038[root@linuxprobe ~]# swapon /dev/sdb2[root@linuxprobe ~]# free -m              total        used        free      shared  buff/cache   availableMem:           1966        1395         101          12         469         380Swap:          7167           9        7158
```

为了能够让新的交换分区设备在重启后依然生效，需要按照下面的格式将相关信息写入到配置文件中，并记得保存：

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                        /               xfs        defaults    1 1UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b    /boot           xfs        defaults    1 2/dev/mapper/rhel-swap                        swap            swap       defaults    0 0/dev/cdrom                                   /media/cdrom    iso9660    defaults    0 0 /dev/sdb1                                    /newFS          xfs        defaults    0 0 /dev/sdb2                                    swap            swap       defaults    0 0 
```

##### **6.7 磁盘容量配额**

本书在前面曾经讲到，Linux系统的设计初衷就是让许多人一起使用并执行各自的任务，从而成为多用户、多任务的操作系统。但是，硬件资源是固定且有限的，如果某些用户不断地在Linux系统上创建文件或者存放电影，硬盘空间总有一天会被占满。针对这种情况，root管理员就需要使用磁盘容量配额服务来限制某位用户或某个用户组针对特定文件夹可以使用的最大硬盘空间或最大文件个数，一旦达到这个最大值就不再允许继续使用。可以使用quota技术进行磁盘容量配额管理，从而限制用户的硬盘可用容量或所能创建的最大文件个数。quota技术还有软限制和硬限制的功能。

> 软限制：当达到软限制时会提示用户，但仍允许用户在限定的额度内继续使用。
>
> 硬限制：当达到硬限制时会提示用户，且强制终止用户的操作。

RHEL 8系统中已经安装了quota磁盘容量配额服务程序包，但存储设备却默认没有开启对quota技术的支持，此时需要手动编辑配置文件再重启一次，让系统中的启动目录（/boot）能够支持quota磁盘配额技术。

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                        /             xfs        defaults         1 1UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b    /boot         xfs        defaults,uquota  1 2/dev/mapper/rhel-swap                        swap          swap       defaults         0 0/dev/cdrom                                   /media/cdrom  iso9660    defaults         0 0 /dev/sdb1                                    /newFS        xfs        defaults         0 0 /dev/sdb2                                    swap          swap       defaults         0 0 [root@linuxprobe ~]# reboot
```

另外，对于学习过早期的Linux系统，或者具有RHEL 5/6系统使用经验的读者来说，这里需要特别注意。早期的Linux系统要想让硬盘设备支持quota磁盘容量配额服务，使用的是usrquota参数，而RHEL 7/8系统使用的则是uquota参数。在重启系统后使用mount命令查看，即可发现/boot目录已经支持quota磁盘配额技术了：

```
[root@linuxprobe ~]# mount | grep boot/dev/sda1 on /boot type xfs (rw,relatime,seclabel,attr2,inode64,usrquota)
```

接下来创建一个用于检查quota磁盘容量配额效果的用户tom，并针对/boot目录增加其他人的写权限，保证用户能够正常写入数据：

```
[root@linuxprobe ~]# useradd tom[root@linuxprobe ~]# chmod -R o+w /boot
```

xfs_quota命令用于管理设备的磁盘容量配额，语法格式为：“xfs_quota [参数] 配额 文件系统”。

这是一个专门针对XFS文件系统来管理quota磁盘容量配额服务而设计的命令，其中，-c参数用于以参数的形式设置要执行的命令；-x参数是专家模式，让运维人员能够对quota服务进行更多复杂的配置。接下来使用xfs_quota命令来设置用户tom对/boot目录的quota磁盘容量配额。具体的限额控制包括：硬盘使用量的软限制和硬限制分别为3MB和6MB；创建文件数量的软限制和硬限制分别为3个和6个。

```
[root@linuxprobe ~]# xfs_quota -x -c 'limit bsoft=3m bhard=6m isoft=3 ihard=6 tom' /boot[root@linuxprobe ~]# xfs_quota -x -c report /bootUser quota on /boot (/dev/sda1)                               Blocks                     User ID          Used       Soft       Hard    Warn/Grace     ---------- -------------------------------------------------- root           114964          0          0     00 [--------]tom                 0       3072       6144     00 [--------]
```

上面所使用的参数分为两组，分别是isoft/ihard与bsoft/bhard，我们来深入的讲解一下。在6.3小节中曾经讲过，在Linux系统中每个文件都会被一个独立的inode信息块所保存属性信息，一个文件对应一个inode信息块，所有isoft和ihard就是通过限制了系统最大使用的inode个数来限制了文件格式。bsoft和bhard则是代表文件所占用的block块大小，也就是文件最多所占用的总统计。

soft是软限制，超过了也只是写到日志中，不对用户行为进行限制。而hard是硬限制，一旦超过就会马上进行禁止，再也不能创建或新占任何的硬盘容量。

当配置好上述的各种软硬限制后，尝试切换到这个普通用户，然后分别尝试创建一个体积为5MB和8MB的文件。可以发现，在创建8MB的文件时受到了系统限制：

```
[root@linuxprobe ~]# su - tom[tom@linuxprobe ~]$ cd /boot[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=5M count=11+0 records in1+0 records out5242880 bytes (5.2 MB, 5.0 MiB) copied, 0.00298178 s, 1.8 GB/s[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=8M count=1dd: error writing '/boot/tom': Disk quota exceeded1+0 records in0+0 records out4194304 bytes (4.2 MB, 4.0 MiB) copied, 0.00398607 s, 1.1 GB/s
```

棒棒的！

edquota命令用于管理系统的磁盘配额，英文全称为：“edit quota”，语法格式为：“edquota [参数] 用户名”。

在为用户设置了quota磁盘容量配额限制后，可以使用edquota命令按需修改限额的数值。其中，-u参数表示要针对哪个用户进行设置；-g参数表示要针对哪个用户组进行设置，如表6-6所示。

表6-6                       edquota命令中可用的参数以及作用

| 参数 | 作用                        |
| ---- | --------------------------- |
| -u   | 对某个用户进行设置          |
| -g   | 对某个用户组进行设置        |
| -p   | 复制原有的规则到新的用户/组 |
| -t   | 限制宽限期限                |



edquota命令会调用Vi或Vim编辑器来让root管理员修改要限制的具体细节，记得用wq保存退出呦。动手把用户tom的硬盘使用量的硬限额从5MB提升到8MB吧：

```
[tom@linuxprobe ~]$ exit[root@linuxprobe ~]# edquota -u tomDisk quotas for user tom (uid 1001):  Filesystem                   blocks       soft       hard     inodes     soft     hard  /dev/sda1                      4096       3072       8192          1        3        6[root@linuxprobe ~]# su - tom[tom@linuxprobe ~]$ cd /boot[tom@linuxprobe boot]$ dd if=/dev/zero of=/boot/tom bs=8M count=11+0 records in1+0 records out8388608 bytes (8.4 MB, 8.0 MiB) copied, 0.0185476 s, 452 MB/s
```

##### **6.8 VDO虚拟数据优化**

Virtual Data Optimize是一种通过压缩或删除存储设备上的数据来优化存储空间的技术，简称VDO，中文名叫虚拟数据优化。是由红帽公司收购了Permabit公司后获取的新技术，在2019年至2020年前后，多次在RHEL7.5/7.6/7.7上进行测试，最终随RHEL 8系统正式公布。VDO技术的关键就是对硬盘内原有的数据进行删重操作，理论上只用原来的一半空间就够了，有点类似于大家平时用的网盘服务，第一次正常上传特别慢，第二次上传的文件几乎可以达到“秒传”的效果，无需再多占用一份空间及漫长等待。除了删重操作，还可以对日志和数据库进行自动压缩，进一步减少存储浪费的情况，针对各种类型文件的压缩效果如表6-7所示。

表6-7                      对各种类型文件压缩效果汇总表

| 文件名  | 描述                            | 类型              | 原始大小（KB） | 实际占用空间（KB） |
| ------- | ------------------------------- | ----------------- | -------------- | ------------------ |
| dickens | 狄更斯文集                      | 英文原文          | 9953           | 9948               |
| mozilla | Mozilla的1.0可执行文件          | 执行程序          | 50020          | 33228              |
| mr      | 医用resonanse图像               | 图片              | 9736           | 9272               |
| nci     | 结构化的化学数据库              | 数据库            | 32767          | 10168              |
| ooffice | Open Office.org 1.01 DLL        | 可执行程序        | 6008           | 5640               |
| osdb    | 基准测试用的MySQL格式示例数据库 | 数据库            | 9849           | 9824               |
| reymont | 瓦迪斯瓦夫·雷蒙特的书           | PDF               | 6471           | 6312               |
| samba   | samba源代码                     | src源码           | 21100          | 11768              |
| sao     | 星空数据                        | 天文格式的bin文件 | 7081           | 7036               |
| webster | 辞海                            | HTML              | 40487          | 40144              |
| xml     | XML文件                         | HTML              | 5220           | 2180               |
| x-ray   | 透视医学图片                    | 医院数据          | 8275           | 8260               |



VDO可以作为本地文件系统、iSCSI或Ceph存储下的附加存储层使用，支持本地和远程存储。参考红帽公司在介绍页面的说明，专家建议做虚拟机或容器时，采用逻辑与物理10：1的比率进行配置，即使用1TB物理存储对应10TB的逻辑存储；而做对象存储时（例如Ceph）则采用3：1的比率进行配置，即使用1TB物理存储对应3TB的逻辑存储。

简而言之，能省空间！

有两种特殊情况我们提前讲一讲。其一，如果公司服务器上已有DM crypt之类的技术是可以与VDO兼容的，但记得要先做加密卷再使用VDO。因为加密会使重复的数据变得有所不同，因此删重操作无法实现，始终记得把加密层放到VDO之下，如图6-14所示。

其二，VDO技术不可叠加使用，1T物理存储提升成10T逻辑存储没问题，再用10T翻成100T就不行了。左脚踩右脚，真的没法飞起来。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/VDO.jpg)

图6-14 VDO技术拓扑图

通过6.5小节的学习，同学们肯定已经把对硬盘进行分区、格式化、挂载操作的方法拿捏死死的了。我们再把虚拟机关闭，添加一块容量为20G的新SATA硬盘进来，开机后就能看到这块名称为/dev/sdc的新硬盘了：

```
[root@linuxprobe ~]# ls -l /dev/sdcbrw-rw----. 1 root disk 8, 32 Jan 6 22:26 /dev/sdc
```

RHEL/CentOS 8系统中默认已经启用VDO技术了，既然是红帽公司自己的技术，兼容性自然没得说。如果您所在的系统没有安装的话不要着急，用dnf命令即可完成：

```
[root@linuxprobe ~]# dnf install kmod-kvdo vdoUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:01:56 ago on Wed 06 Jan 2021 10:37:19 PM CST.Package kmod-kvdo-6.2.0.293-50.el8.x86_64 is already installed.Package vdo-6.2.0.293-10.el8.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

第1步：创建一个全新的VDO卷

管理设备用的就是vdo命令本身，name参数代表新的设备卷的名称；device参数代表由那块磁盘进行制作；vdoLogicalSize参数代表制作后的逻辑卷大小，遵循红帽推荐的原则，20G硬盘翻成200G逻辑卷：

```
[root@linuxprobe ~]# vdo create --name=storage --device=/dev/sdc --vdoLogicalSize=200GCreating VDO storageStarting VDO storageStarting compression on VDO storageVDO instance 0 volume is ready at /dev/mapper/storage
```

### **Tips**

Linux命令行严格区别大小写，vdoLogicalSize参数中的L与S字母需要大写。

第2步：创建成功后，使用status参数查看新建卷的概述信息。

```
[root@linuxprobe ~]# vdo status --name=storageVDO status:  Date: '2021-01-06 22:51:33+08:00'  Node: linuxprobe.comKernel module:  Loaded: true  Name: kvdo  Version information:    kvdo version: 6.2.0.293Configuration:  File: /etc/vdoconf.yml  Last modified: '2021-01-06 22:49:33'VDOs:  storage:    Acknowledgement threads: 1    Activate: enabled    Bio rotation interval: 64    Bio submission threads: 4    Block map cache size: 128M    Block map period: 16380    Block size: 4096    CPU-work threads: 2    Compression: enabled    Configured write policy: auto    Deduplication: enabled………………省略部分输出信息………………
```

输出信息中包含了VDO卷创建的时间、主机名、版本、是否删重（Deduplication）及是否压缩（Compression）等关键指标。

第3步：对新建卷做格式化操作并挂载使用。

新建的VDO卷设备会被乖乖的存放在/dev/mapper目录下，以设备名称命名的文件，对它操作就行。另外挂载前可以用udevadm settle命令来对设备进行一次刷新操作，避免刚刚的配置没有生效：

```
[root@linuxprobe ~]# mkfs.xfs /dev/mapper/storage meta-data=/dev/mapper/storage    isize=512    agcount=4, agsize=13107200 blks         =                       sectsz=4096  attr=2, projid32bit=1         =                       crc=1        finobt=1, sparse=1, rmapbt=0         =                       reflink=1data     =                       bsize=4096   blocks=52428800, imaxpct=25         =                       sunit=0      swidth=0 blksnaming   =version 2              bsize=4096   ascii-ci=0, ftype=1log      =internal log           bsize=4096   blocks=25600, version=2         =                       sectsz=4096  sunit=1 blks, lazy-count=1realtime =none                   extsz=4096   blocks=0, rtextents=0[root@linuxprobe ~]# udevadm settle[root@linuxprobe ~]# mkdir /storage[root@linuxprobe ~]# mount /dev/mapper/storage /storage
```

如果想看下设备的实际使用情况，用vdostats命令即可，human-readable参数作用是存储容量自动进位，以人们更易读的单位输出（显示20G而不是20971520K）：

```
[root@linuxprobe ~]# vdostats --human-readableDevice                    Size      Used Available Use% Space saving%/dev/mapper/storage      20.0G      4.0G     16.0G  20%           99%
```

这里显示的Size是实际物理存储空间大小，20.0G是硬盘的大小，如果想看逻辑存储空间可以用df命令进行查看：

```
[root@linuxprobe ~]# df -hFilesystem             Size  Used Avail Use% Mounted ondevtmpfs               969M     0  969M   0% /devtmpfs                  984M     0  984M   0% /dev/shmtmpfs                  984M  9.6M  974M   1% /runtmpfs                  984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root   17G  3.9G   14G  23% //dev/sr0               6.7G  6.7G     0 100% /media/cdrom/dev/sda1             1014M  152M  863M  15% /boottmpfs                  197M   16K  197M   1% /run/user/42tmpfs                  197M  3.5M  194M   2% /run/user/0/dev/sdb1              2.0G   47M  2.0G   3% /newFS/dev/mapper/storage    200G  2.4G  198G   2% /storage
```

第4步：随便复制来一个大文件，看看占用了多少容量，以及空间节省率Space saving是多少呢？

```
[root@linuxprobe ~]# ls -lh /media/cdrom/images/install.img -r--r--r--. 1 root root 448M Apr 4 2019 /media/cdrom/images/install.img[root@linuxprobe ~]# cp /media/cdrom/images/install.img /storage/[root@linuxprobe ~]# ls -lh /storage/install.img -r--r--r--. 1 root root 448M Jan  6 23:06 /storage/install.img[root@linuxprobe ~]# vdostats --human-readableDevice                    Size      Used Available Use% Space saving%/dev/mapper/storage      20.0G      4.4G     15.6G  22%           18%
```

嗯？效果不明显，再复制来一份，看看这次占用了多少空间：

```
[root@linuxprobe ~]# cp /media/cdrom/images/install.img /storage/rhel.img[root@linuxprobe ~]# vdostats --human-readableDevice                    Size      Used Available Use% Space saving%/dev/mapper/storage      20.0G      4.5G     15.5G  22%           55%
```

是不是感觉很棒，原先448M的文件这次只占用了不到100M的容量，空间节省率也从18%提升到了55%。当然这还仅仅是两次操作而已，好处就已经如此明显了。

第5步：将设备设置成永久挂载生效，一直提供服务。

VDO设备卷创建后就会一直存在了，但需要手动的编辑/etc/fstab文件后才能在下一次重启后自动挂载生效，为我们所用。对于这种逻辑存储设备，其实不太建议用/dev/mapper/storage作为设备名进行挂载，不如试试前面所说的UUID唯一标识符吧？

```
[root@linuxprobe ~]# blkid /dev/mapper/storage /dev/mapper/storage: UUID="cd4e9f12-e16a-415c-ae76-8de069076713" TYPE="xfs"
```

高高兴兴的打开/etc/fstab文件，把对应的字段填写完整即可。建议再加上_netdev参数，代表等系统及网络都启动后再挂载VDO设备卷，保证万无一失。

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                        /             xfs        defaults           1 1UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b    /boot         xfs        defaults,uquota    1 2/dev/mapper/rhel-swap                        swap          swap       defaults           0 0/dev/cdrom                                   /media/cdrom  iso9660    defaults           0 0 /dev/sdb1                                    /newFS        xfs        defaults           0 0 /dev/sdb2                                    swap          swap       defaults           0 0 UUID=cd4e9f12-e16a-415c-ae76-8de069076713    /storage      xfs        defaults,_netdev   0 0 
```

##### **6.9 软硬方式链接**

当引领大家学习完本章所有的硬盘管理知识之后，终于可以放心大胆地讲解Linux系统中的“快捷方式”了。在Windows系统中，快捷方式就是指向原始文件的一个链接文件，可以让用户从不同的位置来访问原始的文件；原文件一旦被删除或剪切到其他地方后，会导致链接文件失效。但是，这个看似简单的东西在Linux系统中可不太一样。

在Linux系统中存在软链接和硬链接两种不同类型。

**软链接（symbolic link）：**也叫符号链接，仅仅包含所链接文件的名称和路径，像个记录地址的标签。当原始文件被删除或移动后，新的链接文件也会随之失效，不能被访问，可以对文件、目录做软链接，跨文件系统也不是问题，从这一点来看与Windows系统的“快捷方式”具有一样的性质。用户访问起来的效果如图6-15所示。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E8%BD%AF%E9%93%BE%E6%8E%A5.jpg)

图6-15 软链接原理示意图

**硬链接（hard link）：**可以将它理解为一个“指向原始文件block的指针”，系统会创建出一个与原来一摸一样的inode信息块。所以，硬链接文件与原始文件其实是一摸一样的，只是名字不同。每添加一个硬链接，该文件的inode个数就会增加1；而且只有当该文件的inode个数为0时，才算彻底将它删除。换言之，由于硬链接实际上是指向原文件block的指针，因此即便原始文件被删除，依然可以通过硬链接文件来访问。需要注意的是，由于技术的局限性，不能跨分区对目录文件进行硬链接。

![第6章 存储结构与管理硬盘第6章 存储结构与管理硬盘](https://www.linuxprobe.com/wp-content/uploads/2020/12/%E7%A1%AC%E9%93%BE%E6%8E%A5.jpg)

图6-16 硬链接原理示意图

### **Tips**

翻开书的目录页，看到标题和对应的页码就应该能够理解了，链接文件就是指向实际内容所在位置的一个标签，通过这个标签，我们就可以找到对应的数据了。

ln命令用于创建文件的软硬链接，英文全称为：“link”，语法格式为：“ln [参数] 原始文件名 链接文件名”。

ln命令用于创建链接文件，格式为“ln [选项] 目标”，其可用的参数以及作用如表6-8所示。在使用ln命令时，是否添加-s参数，将创建出性质不同的两种“快捷方式”。因此如果没有扎实的理论知识和实践经验做铺垫，尽管能够成功完成实验，但永远不会明白为什么会成功。

表6-8                       ln命令中可用的参数以及作用

| 参数 | 作用                                               |
| ---- | -------------------------------------------------- |
| -s   | 创建“符号链接”（如果不带-s参数，则默认创建硬链接） |
| -f   | 强制创建文件或目录的链接                           |
| -i   | 覆盖前先询问                                       |
| -v   | 显示创建链接的过程                                 |



为了更好地理解软链接、硬链接的不同性质，我们先创建出一个文件，做个软链接：

```
[root@linuxprobe ~]# echo "Welcome to linuxprobe.com" > old.txt[root@linuxprobe ~]# ln -s old.txt new.txt[root@linuxprobe ~]# cat old.txt Welcome to linuxprobe.com[root@linuxprobe ~]# cat new.txt Welcome to linuxprobe.com[root@linuxprobe ~]# ls -l old.txt -rw-r--r-- 1 root root 26 Jan 11 00:08 old.txt
```

原始文件叫old，新的软链接文件叫new。删掉原始文件后，软链接立刻就无法读取了：

```
[root@linuxprobe ~]# rm -f old.txt [root@linuxprobe ~]# cat new.txt cat: readit.txt: No such file or directory
```

接下来还是针对原始文件创建一个硬链接，即相当于针对原始文件的硬盘存储位置创建了一个指针，这样一来，新创建的这个硬链接就不再依赖于原始文件的名称等信息，也不会因为原始文件的删除而导致无法读取。同时可以看到创建硬链接后，原始文件的硬盘链接数量增加到了2。

```
[root@linuxprobe ~]# echo "Welcome to linuxprobe.com" > old.txt[root@linuxprobe ~]# ln old.txt new.txt[root@linuxprobe ~]# cat old.txt Welcome to linuxprobe.com[root@linuxprobe ~]# cat new.txt Welcome to linuxprobe.com[root@linuxprobe ~]# ls -l old.txt -rw-r--r-- 2 root root 26 Jan 11 00:13 old.txt
```

非常有意思的现象，创建的硬链接文件竟然会让文件属性第二列的数字变成了2，这个数字就是文件的inode信息块数量。相信同学们已经非常肯定的知道，即便删除了原始文件，新的文件也会一如既往的可以读取，因为只有当文件inode数量被“清零”时，才真正代表这个文件被删除了。

```
[root@linuxprobe ~]# rm -f old.txt [root@linuxprobe ~]# cat new.txt Welcome to linuxprobe.com
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．/home目录与/root目录内存放的文件有何相同点以及不同点？

**答：**这两个目录都是用来存放用户家目录数据的，但是，/root目录存放的是root管理员的家目录数据。

2．假如一个设备的文件名称为/dev/sdb，可以确认它是主板第二个插槽上的设备吗？

**答：**不一定，因为设备的文件名称是由系统的识别顺序来决定的。

3．如果硬盘中需要5个分区，至少需要几个逻辑分区？

**答：**可以选用创建3个主分区+1个扩展分区的方法，然后把扩展分区再分成2个逻辑分区，即有了5个分区。

4．/dev/sda5是主分区还是逻辑分区？

**答：**逻辑分区。

5．哪个服务决定了设备在/dev目录中的名称？

**答：**udev设备管理器服务。

6．用一句话来描述挂载操作。

**答：**当用户需要使用硬盘设备或分区中的数据时，需要先将其与一个已存在的目录文件进行关联，而这个关联动作就是“挂载”。

7．在配置quota磁盘容量配额服务时，软限制数值必须小于硬限制数值么？

**答：**不一定，软限制数值可以小于等于硬限制数值。

8．VDO虚拟数据优化技术能够提升硬盘的物理存储空间？

**答：**不可以，VDO指通过压缩或删重操作，提高硬盘的逻辑空间大小。

9．若原始文件被改名，那么之前创建的硬链接还能访问到这个原始文件么？

**答：**可以。

# [第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/basic-learning-07.html)

**章节简述：**

在学习了第6章讲解的硬盘设备分区、格式化、挂载等知识后，本章将深入讲解各个常用RAID（Redundant Array of Independent Disks，独立冗余磁盘阵列）技术方案的特性，并通过实际部署RAID 10、RAID 5+备份盘等方案来更直观地查看RAID的强大效果，以便进一步满足生产环境对硬盘设备的IO读写速度和数据冗余备份机制的需求。同时，考虑到用户可能会动态调整存储资源，本章还将介绍LVM（Logical Volume Manager，逻辑卷管理器）的部署、扩容、缩小、快照以及卸载删除的相关知识。相信读者在学完本章内容后，便可以在企业级生产环境中灵活运用RAID和LVM来满足对存储资源的高级管理需求。

本章目录结构

- 7.1 RAID磁盘冗余阵列
  - [7.1.1 部署磁盘阵列](https://www.linuxprobe.com/basic-learning-07.html#711)
  - [7.1.2 损坏磁盘阵列及修复](https://www.linuxprobe.com/basic-learning-07.html#712)
  - [7.1.3 磁盘阵列+备份盘](https://www.linuxprobe.com/basic-learning-07.html#713)
  - [7.1.4 删除磁盘阵列](https://www.linuxprobe.com/basic-learning-07.html#714)
- 7.2 LVM逻辑卷管理器
  - [7.2.1 部署逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#721)
  - [7.2.2 扩容逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#722)
  - [7.2.3 缩小逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#723)
  - [7.2.4 逻辑卷快照](https://www.linuxprobe.com/basic-learning-07.html#724)
  - [7.2.5 删除逻辑卷](https://www.linuxprobe.com/basic-learning-07.html#725)

##### **7.1 RAID磁盘冗余阵列**

近年来， CPU的处理性能保持着高速增长，即便是被称为“牙膏厂”的英特尔公司也在2017年发布了i9-7980XE处理器芯片，率先让家用电脑达到了18核心36线程。2020年末，AMD公司又推出了线程撕裂者系统处理器3990X，让家用电脑也可以驾驭的了64核心128线程的处理器小怪兽了。但与此同时，硬盘设备的性能提升却不是很大，逐渐成为当代计算机整体性能的瓶颈。而且，由于硬盘设备需要进行持续、频繁、大量的IO操作，相较于其他设备，其损坏几率也大幅增加，导致重要数据丢失的几率也随之增加。

硬盘设备是计算机中较容易故障的元器件之一，加之由于其需要存储数据的特殊性质，不能像CPU、内存、电源甚至主板故障后更换新的就好，所以生产环境中一定要未雨绸缪，提前做好数据的冗余及异地备份等工作。

1988年，美国加利福尼亚大学伯克利分校首次提出并定义了**R**edundant **A**rray of **I**ndependent **D**isks技术的概念，中文名是磁盘冗余阵列，简称RAID。RAID技术通过把多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个区段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果。

任何事物都有它的两面性。RAID技术确实具有非常好的数据冗余备份功能，但是它也相应地提高了成本支出。就像原本我们只有一个电话本，但是为了避免遗失，把联系人号码信息写成了两份，自然要为此多买一个电话本，这也就相应地提升了成本支出。RAID技术的设计初衷是减少因为采购硬盘设备带来的费用支出，但是与数据本身的价值相比较，现代企业更看重的则是RAID技术所具备的冗余备份机制以及带来的硬盘吞吐量的提升。也就是说，RAID不仅降低了硬盘设备损坏后丢失数据的几率，还提升了硬盘设备的读写速度，所以它在绝大多数运营商或大中型企业中得以广泛部署和应用。

出于成本和技术方面的考虑，需要针对不同的需求在数据可靠性及读写性能上作出权衡，制定出满足各自需求的不同方案。目前已有的RAID磁盘阵列的方案至少有十几种，而[刘遄](https://www.linuxprobe.com/)老师接下来会详细讲解RAID 0、RAID 1、RAID 5与RAID 10这4种最常见的方案，这四种方案的对比如表7-1所示，其中n代表硬盘总数。

表7-3                          RAID 0、1、5、10方案技术对比

| RAID级别 | 最少硬盘 | 可用容量 | 读写性能 | 安全性 | 特点                                                         |
| -------- | -------- | -------- | -------- | ------ | ------------------------------------------------------------ |
| 0        | 2        | n        | n        | 低     | 追求最大容量和速度，任何一块盘损坏，数据全部异常。           |
| 1        | 2        | n/2      | n        | 高     | 追求最大安全性，只要阵列组中有一块硬盘可用，数据不受影响。   |
| 5        | 3        | n-1      | n-1      | 中     | 在控制成本的前提下，追求硬盘的最大容量、速度及安全性，允许有一块硬盘异常，数据不受影响。 |
| 10       | 4        | n/2      | n/2      | 高     | 综合RAID1和RAID0的优点，追求硬盘的速度和安全性，允许有一半硬盘异常（不可同组），数据不受影响 |



**1. RAID 0**

RAID 0技术把多块物理硬盘设备（至少两块）通过硬件或软件的方式串联在一起，组成一个大的卷组，并将数据依次写入到各个物理硬盘中。这样一来，在最理想的状态下，硬盘设备的读写性能会提升数倍，但是若任意一块硬盘发生故障将导致整个系统的数据都受到破坏。通俗来说，RAID 0技术能够有效地提升硬盘数据的吞吐速度，但是不具备数据备份和错误修复能力。如图7-1所示，数据被分别写入到不同的硬盘设备中，即硬盘A和硬盘B设备会分别保存数据资料，最终实现提升读取、写入速度的效果。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-0-1.jpg)

图7-1 RAID 0技术示意图

**2. RAID 1**

尽管RAID 0技术提升了硬盘设备的读写速度，但是它是将数据依次写入到各个物理硬盘中，也就是说，它的数据是分开存放的，其中任何一块硬盘发生故障都会损坏整个系统的数据。因此，如果生产环境对硬盘设备的读写速度没有要求，而是希望增加数据的安全性时，就需要用到RAID 1技术了。

在图7-2所示的RAID 1技术示意图中可以看到，它是把两块以上的硬盘设备进行绑定，在写入数据时，是将数据同时写入到多块硬盘设备上（可以将其视为数据的镜像或备份）。当其中某一块硬盘发生故障后，一般会立即自动以热交换的方式来恢复数据的正常使用。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-1-1.jpg)

图7-2 RAID 1技术示意图

考虑到写入操作时CPU切换硬盘的开销，速度会比RAID 0有微弱的降低，但在读取数据的时候，操作系统可以分别从两块硬盘中读取信息，理论读取速度的峰值可以是硬盘数量的倍数。另外平时只要保证有一块硬盘稳定运行，数据就不会出现损坏的情况，可靠性较高。

另外RAID 1技术虽然十分注重数据的安全性，但是因为是在多块硬盘设备中写入了相同的数据，因此硬盘设备的利用率得以下降，从理论上来说，图7-2所示的硬盘空间的真实可用率只有50%，由三块硬盘设备组成的RAID 1磁盘阵列的可用率只有33%左右，以此类推。而且，由于需要把数据同时写入到两块以上的硬盘设备，这无疑也在一定程度上增大了系统计算功能的负载。

那么，有没有一种RAID方案既考虑到了硬盘设备的读写速度和数据安全性，还兼顾了成本问题呢？实际上，单从数据安全和成本问题上来讲，就不可能在保持原有硬盘设备的利用率且还不增加新设备的情况下，能大幅提升数据的安全性。[刘遄](https://www.linuxprobe.com/)老师也没有必要忽悠各位读者，下面将要讲解的RAID 5技术虽然在理论上兼顾了三者（读写速度、数据安全性、成本），但实际上更像是对这三者的“相互妥协”。

**3. RAID 5**

如图7-3所示，RAID5技术是把硬盘设备的数据奇偶校验信息保存到其他硬盘设备中。RAID 5磁盘阵列组中数据的奇偶校验信息并不是单独保存到某一块硬盘设备中，而是存储到除自身以外的其他每一块硬盘设备上，这样的好处是其中任何一设备损坏后不至于出现致命缺陷；图7-3中parity部分存放的就是数据的奇偶校验信息，换句话说，就是RAID 5技术实际上没有备份硬盘中的真实数据信息，而是当硬盘设备出现问题后通过奇偶校验信息来尝试重建损坏的数据。RAID这样的技术特性“妥协”地兼顾了硬盘设备的读写速度、数据安全性与存储成本问题。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-5-1.jpg)

图7-3 RAID5技术示意图

RAID 5最少由三块硬盘组成，使用的是Disk Striping硬盘切割技术。比RAID 1级别好处就在于保存的是奇偶校验信息而不是一模一样的文件内容，所以当重复写入某个文件时，RAID 5级别的磁盘阵列组只需要对应一个奇偶校验信息就可以，效率更高，存储成本也会随之降低。

**4.  RAID 10**

鉴于RAID 5技术是因为硬盘设备的成本问题对读写速度和数据的安全性能而有了一定的妥协，但是大部分企业更在乎的是数据本身的价值而非硬盘价格，因此生产环境中主要使用RAID 10技术。

顾名思义，RAID 10技术是RAID 1+RAID 0技术的一个“组合体”。如图7-4所示，RAID 10技术需要至少4块硬盘来组建，其中先分别两两制作成RAID 1磁盘阵列，以保证数据的安全性；然后再对两个RAID 1磁盘阵列实施RAID 0技术，进一步提高硬盘设备的读写速度。这样从理论上来讲，只要坏的不是同一组中的所有硬盘，那么最多可以损坏50%的硬盘设备而不丢失数据。由于RAID 10技术继承了RAID 0的高读写速度和RAID 1的数据安全性，在不考虑成本的情况下RAID 10的性能都超过了RAID 5，因此当前成为广泛使用的一种存储技术。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/RAID-10-2.jpg)

图7-4 RAID 10技术示意图

### **Tips**

由于RAID 10是由RAID 1和RAID 0组成的，因此正确叫法是“RAID 一零”，而不是“RAID 十”。

细看上图7-4可以分析出，RAID 10是先对信息进行分割，然后再两两一组做的镜像。也就是将RAID 1作为最低级别的组合，再使用RAID 0技术组合到一起，将它们视为“一整块”硬盘。而RAID 01则是相反的，它回先将硬盘分为两组，使用RAID 0作为最低级别的组合，再将两组硬盘通过RAID 1技术组合到一起。

但区别非常明显，RAID 10级别中任何一块硬盘损坏都不会影响到数据安全性，其余硬盘均会正常运作。但RAID 01只要有任何一块盘损坏，最低级别的RAID 0硬盘组马上会停止运作，可能造成严重隐患。所以RAID 10远比RAID 01常见，很多主板甚至不支持RAID 01。

###### **7.1.1 部署磁盘阵列**

在具备了第六章中管理硬盘设备的基础知识后，再来部署RAID和LVM就变得十分轻松了。首先，需要在虚拟机中添加4块硬盘设备来制作一个RAID 10磁盘阵列，如图7-5所示。添加硬盘的步骤省略大家自己操作就行，记得要用SCSI或SATA接口类型，大小默认20GB就可以。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E5%9B%9B%E5%9D%97%E7%A1%AC%E7%9B%98%E8%AE%BE%E5%A4%87-1.png)

图7-5 添加四块硬盘设备

这几块硬盘设备是模拟出来的，不需要特意去买几块真实的物理硬盘插到电脑上。需要注意的是，一定要记得在关闭系统之后，再在虚拟机中添加硬盘设备，否则可能会因为计算机架构的不同而导致虚拟机系统无法识别新添加的硬盘设备。

当前，生产环境中用到的服务器一般都配备RAID阵列卡，尽管服务器的价格越来越便宜，但是我们没有必要为了做一个实验而去单独购买一台服务器，而是可以学会用mdadm[命令](https://www.linuxcool.com/)在[Linux系统](https://www.linuxprobe.com/)中创建和管理软件RAID磁盘阵列，而且它涉及的理论知识的操作过程与生产环境中的完全一致。mdadm[命令](https://www.linuxcool.com/)的常用参数以及作用如表7-2所示。

mdadm命令用于创建、调整、监控和管理RAID设备，英文全称为：“multiple devices admin”，语法格式为：“mdadm 参数 硬盘名称”。

表7-2                      mdadm命令的常用参数和作用

| 参数 | 作用             |
| ---- | ---------------- |
| -a   | 检测设备名称     |
| -n   | 指定设备数量     |
| -l   | 指定RAID级别     |
| -C   | 创建             |
| -v   | 显示过程         |
| -f   | 模拟设备损坏     |
| -r   | 移除设备         |
| -Q   | 查看摘要信息     |
| -D   | 查看详细信息     |
| -S   | 停止RAID磁盘阵列 |



接下来，使用mdadm命令创建RAID 10，名称为“/dev/md0”。

第6章中讲到，udev是[Linux](https://www.linuxprobe.com/)系统内核中用来给硬件命名的服务，其命名规则也非常简单。我们可以通过命名规则猜测到第二个SCSI存储设备的名称会是/dev/sdb，然后依此类推。使用硬盘设备来部署RAID磁盘阵列很像是将几位同学组成一个班级，但总不能将班级命名为/dev/sdbcde吧。尽管这样可以一眼看出它是由哪些元素组成的，但是并不利于记忆和阅读。更何况如果是使用10、50、100个硬盘来部署RAID磁盘阵列呢？

此时，就需要使用mdadm中的参数了。其中，-C参数代表创建一个RAID阵列卡；-v参数显示创建的过程，同时在后面追加一个设备名称/dev/md0，这样/dev/md0就是创建后的RAID磁盘阵列的名称；-n 4参数代表使用4块硬盘来部署这个RAID磁盘阵列；而-l 10参数则代表RAID 10方案；最后再加上4块硬盘设备的名称就搞定了。

```
[root@linuxprobe ~]# mdadm -Cv /dev/md0 -n 4 -l 10 /dev/sdb /dev/sdc /dev/sdd /dev/sde
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 20954112K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

初始化过程可以用-D参数进行查看，大约需要1分钟左右，也可以用-Q参数查看简要信息：

```
[root@linuxprobe ~]# mdadm -Q /dev/md0
/dev/md0: 39.97GiB raid10 4 devices, 0 spares. Use mdadm --detail for more detail.
```

同学们会好奇，为什么四块20G大小的硬盘组成的磁盘阵列组，可用空间只有39.97G呢？

这还是不得不要提到RAID 10级别的原理，通过两两一组硬盘组成的RAID 1保证了数据的可靠性，每一份数据都会被保存两次，50%的使用率，50%的冗余率，因此80G的容量显示也就是只有一半了。

等两三分钟后，把制作好的RAID磁盘阵列格式化为ext4格式：

```
[root@linuxprobe ~]# mkfs.ext4 /dev/md0
mke2fs 1.44.3 (10-July-2018)
Creating filesystem with 10477056 4k blocks and 2621440 inodes
Filesystem UUID: d1c68318-a919-4211-b4dc-c4437bcfe9da
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done   
```

随后，创建挂载点后把硬盘设备进行挂载操作。

```
[root@linuxprobe ~]# mkdir /RAID
[root@linuxprobe ~]# mount /dev/md0 /RAID
[root@linuxprobe ~]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               969M     0  969M   0% /dev
tmpfs                  984M     0  984M   0% /dev/shm
tmpfs                  984M  9.6M  975M   1% /run
tmpfs                  984M     0  984M   0% /sys/fs/cgroup
/dev/mapper/rhel-root   17G  3.9G   14G  23% /
/dev/sr0               6.7G  6.7G     0 100% /media/cdrom
/dev/sda1             1014M  152M  863M  15% /boot
tmpfs                  197M   16K  197M   1% /run/user/42
tmpfs                  197M  3.5M  194M   2% /run/user/0
/dev/md0                40G   49M   38G   1% /RAID
```

再来查看/dev/md0磁盘阵列组设备的详细信息，确认下RAID级别（Raid Level）、大小（Array Size）和总硬盘数（Total Devices）都是否正确：

```
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jan 13 08:24:58 2021
        Raid Level : raid10
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 04:49:57 2021
             State : clean 
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host linuxprobe.com)
              UUID : 289f501b:3f5f70f9:79189d77:f51ca11a
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde
```

如果想让创建好的RAID磁盘阵列组能够一直为我们服务，不会因每次的重启操作而取消，那么一定要记得将信息添加到/etc/fstab文件中，这样每次重启后还都是有效的。

```
[root@linuxprobe ~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
[root@linuxprobe ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Jul 21 05:03:40 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rhel-root                       /                 xfs         defaults      0 0
UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot             xfs         defaults      0 0
/dev/mapper/rhel-swap                       swap              swap        defaults      0 0
/dev/cdrom                                  /media/cdrom      iso9660     defaults      0 0 
/dev/md0                                    /RAID             ext4        defaults      0 0
```

###### **7.1.2 损坏磁盘阵列及修复**

咱们在生产环境中部署RAID 10磁盘阵列组目的就是为了提高存储设备的IO读写速度及数据的安全性，但因为这次是在本机电脑上模拟出来的硬盘设备所以对于读写速度的改善可能并不直观，因此决定给同学们讲解下RAID磁盘阵列组损坏后的处理方法，这样以后步入了运维岗位后不会因为突发事件而手忙脚乱。首先确认有一块物理硬盘设备出现损坏不能再继续正常使用后，应该使用mdadm命令来予以移除之后查看下RAID磁盘阵列组的状态已经被改变。

在确认有一块物理硬盘设备出现损坏而不能继续正常使用后，应该使用mdadm命令将其移除，然后查看RAID磁盘阵列的状态，可以发现状态已经改变。

```
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Jan 14 05:12:20 2021
        Raid Level : raid10
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Thu Jan 14 05:33:06 2021
             State : clean, degraded 
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 81ee0668:7627c733:0b170c41:cd12f376
            Events : 19

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       0       8       16        -      faulty   /dev/sdb
```

刚刚使用的-f参数是让硬盘模拟损坏，为了能够彻底的将故障盘移除，还要再来一步操作：

```
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
```

在RAID 10级别的磁盘阵列中，当RAID 1磁盘阵列中存在一个故障盘时并不影响RAID 10磁盘阵列的使用。当购买了新的硬盘设备后再使用mdadm命令来予以替换即可，在此期间可以在/RAID目录中正常地创建或删除文件。由于我们是在虚拟机中模拟硬盘，所以先重启系统，然后再把新的硬盘添加到RAID磁盘阵列中。

更换硬盘后再次使用-a参数进行添加操作，默认会自动开始数据的同步工作，使用-D参数即可看到整个过程和百分比进度：

```
[root@linuxprobe ~]# mdadm /dev/md0 -a /dev/sdbmdadm: added /dev/sdb[root@linuxprobe ~]# mdadm -D /dev/md0/dev/md0:           Version : 1.2     Creation Time : Thu Jan 14 05:12:20 2021        Raid Level : raid10        Array Size : 41908224 (39.97 GiB 42.91 GB)     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)      Raid Devices : 4     Total Devices : 4       Persistence : Superblock is persistent       Update Time : Thu Jan 14 05:37:32 2021             State : clean, degraded, recovering     Active Devices : 3   Working Devices : 4    Failed Devices : 0     Spare Devices : 1            Layout : near=2        Chunk Size : 512KConsistency Policy : resync    Rebuild Status : 77% complete              Name : localhost.localdomain:0  (local to host localhost.localdomain)              UUID : 81ee0668:7627c733:0b170c41:cd12f376            Events : 34    Number   Major   Minor   RaidDevice State       4       8       16        0      spare rebuilding    /dev/sdb       1       8       32        1      active sync set-B   /dev/sdc       2       8       48        2      active sync set-A   /dev/sdd       3       8       64        3      active sync set-B   /dev/sde
```

这时候会有小可爱学生举手提问了：“老师，我们公司机房的阵列卡上带了三十几块硬盘呢，就算看到了/dev/sdb硬盘故障了，我也不知道该替换那一块啊，怕错拔了好设备”。其实这到不用担心，因为一旦硬盘故障了，服务器上指示灯也会转成红灯（或者闪烁黄灯），如图7-6所示，对照着处理即可。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%A1%AC%E7%9B%98%E6%95%85%E9%9A%9C.png)

###### **7.1.3 磁盘阵列+备份盘**

RAID 10磁盘阵列中最多允许50%的硬盘设备发生故障，但是存在这样一种极端情况，即同一RAID 1磁盘阵列中的硬盘设备若全部损坏，也会导致数据丢失。换句话说，在RAID 10磁盘阵列中，如果RAID 1中的某一块硬盘出现了故障，而我们正在前往修复的路上，恰巧该RAID1磁盘阵列中的另一块硬盘设备也出现故障，那么数据就被彻底丢失了。刘遄老师可真不是乌鸦嘴，这种RAID 1磁盘阵列中的硬盘设备同时损坏的情况还真被我的学生遇到过。

在这样的情况下，该怎么办呢？其实，完全可以使用RAID备份盘技术来预防这类事故。该技术的核心理念就是准备一块足够大的硬盘，这块硬盘平时处于闲置状态，一旦RAID磁盘阵列中有硬盘出现故障后则会马上自动顶替上去。这样很棒吧！

为了避免多个实验之间相互发生冲突，我们需要保证每个实验的相对独立性，为此需要大家自行将虚拟机还原到初始状态。另外，由于刚才已经演示了RAID 10磁盘阵列的部署方法，现在来看一下RAID 5的部署效果吧。部署RAID 5磁盘阵列时，至少需要用到3块硬盘，还需要再加一块备份硬盘（也叫热备盘），所以总计需要在虚拟机中模拟4块硬盘设备，如图7-7所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E5%9B%9B%E5%9D%97%E7%A1%AC%E7%9B%98%E8%AE%BE%E5%A4%87-1.png)

图7-7 重置虚拟机后，再添加四块硬盘设备

现在创建一个RAID 5磁盘阵列+备份盘。在下面的命令中，参数-n 3代表创建这个RAID 5磁盘阵列所需的硬盘数，参数-l 5代表RAID的级别，而参数-x 1则代表有一块备份盘。当查看/dev/md0（即RAID 5磁盘阵列的名称）磁盘阵列的时候就能看到有一块备份盘在等待中了。

```
[root@linuxprobe ~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sdd /dev/sdemdadm: layout defaults to left-symmetricmdadm: layout defaults to left-symmetricmdadm: chunk size defaults to 512Kmdadm: size set to 20954112Kmdadm: Defaulting to version 1.2 metadatamdadm: array /dev/md0 started.[root@linuxprobe ~]# mdadm -D /dev/md0/dev/md0:           Version : 1.2     Creation Time : Thu Jan 14 06:12:32 2021        Raid Level : raid5        Array Size : 41908224 (39.97 GiB 42.91 GB)     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)      Raid Devices : 3     Total Devices : 4       Persistence : Superblock is persistent       Update Time : Thu Jan 14 06:14:16 2021             State : clean     Active Devices : 3   Working Devices : 4    Failed Devices : 0     Spare Devices : 1            Layout : left-symmetric        Chunk Size : 512KConsistency Policy : resync              Name : localhost.localdomain:0  (local to host localhost.localdomain)              UUID : cf0c34b6:3b08edfb:85dfa14f:e2bffc1e            Events : 18    Number   Major   Minor   RaidDevice State       0       8       16        0      active sync   /dev/sdb       1       8       32        1      active sync   /dev/sdc       4       8       48        2      active sync   /dev/sdd       3       8       64        -      spare   /dev/sde
```

现在将部署好的RAID 5磁盘阵列格式化为ext4文件格式，然后挂载到目录上，之后就能够使用了：

```
[root@linuxprobe ~]# mkfs.ext4 /dev/md0mke2fs 1.44.3 (10-July-2018)Creating filesystem with 10477056 4k blocks and 2621440 inodesFilesystem UUID: ff016386-1126-4799-8a5b-d716242276ecSuperblock backups stored on blocks: 	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 	4096000, 7962624Allocating group tables: done                            Writing inode tables: done                            Creating journal (65536 blocks): doneWriting superblocks and filesystem accounting information: done   [root@linuxprobe ~]# mkdir /RAID[root@linuxprobe ~]# echo "/dev/md0 /RAID ext4 defaults 0 0" >> /etc/fstab
```

由三块硬盘组成的RAID 5级别磁盘阵列，它对应的可用空间是n-1，也就是40G。热备盘的空间是不算入内的，平时完全就是在“睡觉”中，只有意外出现时才会开始工作。

```
[root@linuxprobe ~]# mount -a[root@linuxprobe ~]# df -hFilesystem             Size  Used Avail Use% Mounted ondevtmpfs               969M     0  969M   0% /devtmpfs                  984M     0  984M   0% /dev/shmtmpfs                  984M  9.6M  974M   1% /runtmpfs                  984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root   17G  3.9G   14G  23% //dev/sr0               6.7G  6.7G     0 100% /media/cdrom/dev/sda1             1014M  152M  863M  15% /boottmpfs                  197M   16K  197M   1% /run/user/42tmpfs                  197M  3.5M  194M   2% /run/user/0/dev/md0                40G   49M   38G   1% /RAID
```

最后是见证奇迹的时刻！我们再次把硬盘设备/dev/sdb移出磁盘阵列，然后迅速查看/dev/md0磁盘阵列的状态，就会发现备份盘已经被自动顶替上去并开始了数据同步。RAID中的这种备份盘技术非常实用，可以在保证RAID磁盘阵列数据安全性的基础上进一步提高数据可靠性，所以，如果公司不差钱的话还是再买上一块备份盘以防万一。

```
[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdbmdadm: set /dev/sdb faulty in /dev/md0[root@linuxprobe ~]# mdadm -D /dev/md0/dev/md0:           Version : 1.2     Creation Time : Thu Jan 14 06:12:32 2021        Raid Level : raid5        Array Size : 41908224 (39.97 GiB 42.91 GB)     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)      Raid Devices : 3     Total Devices : 4       Persistence : Superblock is persistent       Update Time : Thu Jan 14 06:24:38 2021             State : clean     Active Devices : 3   Working Devices : 3    Failed Devices : 1     Spare Devices : 0            Layout : left-symmetric        Chunk Size : 512KConsistency Policy : resync              Name : localhost.localdomain:0  (local to host localhost.localdomain)              UUID : cf0c34b6:3b08edfb:85dfa14f:e2bffc1e            Events : 37    Number   Major   Minor   RaidDevice State       3       8       64        0      active sync   /dev/sde       1       8       32        1      active sync   /dev/sdc       4       8       48        2      active sync   /dev/sdd       0       8       16        -      faulty   /dev/sdb
```

是不是感觉很有意思呢，另外考虑到不想让篇幅过长，所以我们刚刚一直没有复制粘贴/RAID目录中文件的信息，有兴趣的同学可以自己动手试一下，里面的文件内容非常安全，不会出现丢失的情况。如果后面像再添加进去一块热备盘，使用-a参数就可以啦。

###### **7.1.4 删除磁盘阵列**

生产环境中，RAID磁盘阵列组部署后一般就不会轻易被停用了，但万一赶上了，还是要知道怎么样删除的。上面这种RAID 5+热备盘损坏的情况是比较复杂的，就以这种情形来进行讲解是再好不过的了。

首先需要将所有的磁盘都设置成停用状态：

```
[root@linuxprobe ~]# umount /RAID[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdcmdadm: set /dev/sdc faulty in /dev/md0[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sddmdadm: set /dev/sdd faulty in /dev/md0[root@linuxprobe ~]# mdadm /dev/md0 -f /dev/sdemdadm: set /dev/sde faulty in /dev/md0
```

然后再逐一的移除出去：

```
[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdbmdadm: hot removed /dev/sdb from /dev/md0[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdcmdadm: hot removed /dev/sdc from /dev/md0[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sddmdadm: hot removed /dev/sdd from /dev/md0[root@linuxprobe ~]# mdadm /dev/md0 -r /dev/sdemdadm: hot removed /dev/sde from /dev/md0
```

如果着急的同学也可以用“mdadm /dev/md0 -f /dev/sdb -r /dev/sdb”一条命令搞定。但由于这个命令在早期版本的服务器中不能一起使用，因此还是保守起见一步步的操作吧。

将所有的硬盘都移除后，再来看下磁盘阵列组的状态：

```
[root@linuxprobe ~]# mdadm -D /dev/md0/dev/md0:           Version : 1.2     Creation Time : Fri Jan 15 08:53:41 2021        Raid Level : raid5        Array Size : 41908224 (39.97 GiB 42.91 GB)     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)      Raid Devices : 3     Total Devices : 0       Persistence : Superblock is persistent       Update Time : Fri Jan 15 09:00:57 2021             State : clean, FAILED     Active Devices : 0    Failed Devices : 0     Spare Devices : 0            Layout : left-symmetric        Chunk Size : 512KConsistency Policy : resync    Number   Major   Minor   RaidDevice State       -       0        0        0      removed       -       0        0        1      removed       -       0        0        2      removed
```

很棒，继续再停用整个RAID磁盘组，咱们的工作就彻底的完成了：

```
[root@linuxprobe ~]# mdadm --stop /dev/md0mdadm: stopped /dev/md0[root@linuxprobe ~]# ls /dev/md0ls: cannot access '/dev/md0': No such file or directory
```

有一些老版本的服务器，使用完--stop参数后依然会保留设备文件，很明显是没有处理干净，这时再执行一下：“mdadm --remove /dev/md0”命令即可，同学们可以记一下，以备不时之需。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **7.2 LVM逻辑卷管理器**

前面学习的硬盘设备管理技术虽然能够有效地提高硬盘设备的读写速度以及数据的安全性，但是在硬盘分好区或者部署为RAID磁盘阵列之后，再想修改硬盘分区大小就不容易了。换句话说，当用户想要随着实际需求的变化调整硬盘分区的大小时，会受到硬盘“灵活性”的限制。这时就需要用到另外一项非常普及的硬盘设备资源管理技术了— Logical Volume Manager（逻辑卷管理器，简称LVM）。LVM允许用户对硬盘资源进行动态调整。

逻辑卷管理器是Linux系统用于对硬盘分区进行管理的一种机制，理论性较强，其创建初衷是为了解决硬盘设备在创建分区后不易修改分区大小的缺陷。尽管对传统的硬盘分区进行强制扩容或缩容从理论上来讲是可行的，但是却可能造成数据的丢失。而LVM技术是在硬盘分区和文件系统之间添加了一个逻辑层，它提供了一个抽象的卷组，可以把多块硬盘进行卷组合并。这样一来，用户不必关心物理硬盘设备的底层架构和布局，就可以实现对硬盘分区的动态调整。LVM的技术架构如图7-8所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/LVM%E9%80%BB%E8%BE%91%E5%8D%B7%E7%AE%A1%E7%90%86%E5%99%A8-1.jpg)

图7-8 逻辑卷管理器的技术结构

为了帮助大家理解，来举一个吃货的例子吧。比如小明家里想吃馒头但是面粉不够了，于是妈妈从隔壁老王家、老李家、老张家分别借来一些面粉，准备蒸馒头吃。首先需要把这些面粉（物理卷[PV，Physical Volume]）揉成一个大面团（卷组[VG，Volume Group]），然后再把这个大面团分割成一个个小馒头（逻辑卷[LV，Logical Volume]），而且每个小馒头的重量必须是每勺面粉（基本单元[PE，Physical Extent]）的倍数。

在日常的使用中，如果VG卷组的剩余容量不足了，可以随时将新的PV物理卷加入到里面去，不断的扩容。怕同学们还是不好理解，又准备了一张逻辑卷管理器的使用流程示意图，如图7-9所示。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/LVM%E9%80%BB%E8%BE%91%E5%8D%B7%E7%AE%A1%E7%90%86%E6%9C%9F%E4%BD%BF%E7%94%A8%E6%B5%81%E7%A8%8B%E7%A4%BA%E6%84%8F%E5%9B%BE-1.jpg)

图7-9 逻辑卷管理器使用流程图

物理卷处于LVM中的最底层，可以将其理解为物理硬盘、硬盘分区或者RAID磁盘阵列。卷组建立在物理卷之上，一个卷组能够包含多个物理卷，而且在卷组创建之后也可以继续向其中添加新的物理卷。逻辑卷是用卷组中空闲的资源建立的，并且逻辑卷在建立后可以动态地扩展或缩小空间。这就是LVM的核心理念。

###### **7.2.1 部署逻辑卷**

一般而言，在生产环境中无法在最初时就精确地评估每个硬盘分区在日后的使用情况，因此会导致原先分配的硬盘分区不够用。比如，伴随着业务量的增加，用于存放交易记录的数据库目录的体积也随之增加；因为分析并记录用户的行为从而导致日志目录的体积不断变大，这些都会导致原有的硬盘分区在使用上捉襟见肘。而且，还存在对较大的硬盘分区进行精简缩容的情况。

我们可以通过部署LVM来解决上述问题。部署时需要逐个配置物理卷、卷组和逻辑卷，常用的部署命令如表7-3所示。

表7-3                          常用的LVM部署命令

| 功能/命令 | 物理卷管理 | 卷组管理  | 逻辑卷管理 |
| --------- | ---------- | --------- | ---------- |
| 扫描      | pvscan     | vgscan    | lvscan     |
| 建立      | pvcreate   | vgcreate  | lvcreate   |
| 显示      | pvdisplay  | vgdisplay | lvdisplay  |
| 删除      | pvremove   | vgremove  | lvremove   |
| 扩展      |            | vgextend  | lvextend   |
| 缩小      |            | vgreduce  | lvreduce   |



为了避免实验之间相互发生冲突，请大家自行将虚拟机还原到初始状态，并重新添加两块新硬盘设备，如图7-10所示，然后开机。

![第7章 使用RAID与LVM磁盘阵列技术第7章 使用RAID与LVM磁盘阵列技术](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%B7%BB%E5%8A%A0%E4%B8%A4%E5%9D%97%E6%96%B0%E7%A1%AC%E7%9B%98-1.png)

图7-10 在虚拟机中添加两块新的硬盘设备

在虚拟机中添加两块新硬盘设备的目的，是为了更好地演示LVM理念中用户无需关心底层物理硬盘设备的特性。我们先对这两块新硬盘进行创建物理卷的操作，可以将该操作简单理解成让硬盘设备支持LVM技术，或者理解成是把硬盘设备加入到LVM技术可用的硬件资源池中，然后对这两块硬盘进行卷组合并，卷组的名称允许由用户来自定义。接下来，根据需求把合并后的卷组切割出一个约为150MB的逻辑卷设备，最后把这个逻辑卷设备格式化成EXT4文件系统后挂载使用。在下文中，将对每一个步骤再作一些简单的描述。

**第1步**：让新添加的两块硬盘设备支持LVM技术。

```
[root@linuxprobe ~]# pvcreate /dev/sdb /dev/sdc  Physical volume "/dev/sdb" successfully created.  Physical volume "/dev/sdc" successfully created.
```

**第2步**：把两块硬盘设备加入到storage卷组中，然后查看卷组的状态。

```
[root@linuxprobe ~]# vgcreate storage /dev/sdb /dev/sdc Volume group "storage" successfully created[root@linuxprobe ~]# vgdisplay  --- Volume group ---  VG Name               storage  System ID               Format                lvm2  Metadata Areas        2  Metadata Sequence No  1  VG Access             read/write  VG Status             resizable  MAX LV                0  Cur LV                0  Open LV               0  Max PV                0  Cur PV                2  Act PV                2  VG Size               39.99 GiB  PE Size               4.00 MiB  Total PE              10238  Alloc PE / Size       0 / 0     Free  PE / Size       10238 / 39.99 GiB  VG UUID               HPwsm4-lOvI-8O0Q-TG54-BkyI-ONYE-owlGLd………………省略部分输出信息………………
```

**第3步**：再切割出一个约为150MB的逻辑卷设备。

这里需要注意切割单位的问题。在对逻辑卷进行切割时有两种计量单位。第一种是以容量为单位，所使用的参数为-L。例如，使用-L 150M生成一个大小为150MB的逻辑卷。另外一种是以基本单元的个数为单位，所使用的参数为-l。每个基本单元的大小默认为4MB。例如，使用-l 37可以生成一个大小为37×4MB=148MB的逻辑卷。

```
[root@linuxprobe ~]# lvcreate -n vo -l 37 storage Logical volume "vo" created.[root@linuxprobe ~]# lvdisplay   --- Logical volume ---  LV Path                /dev/storage/vo  LV Name                vo  VG Name                storage  LV UUID                AsDGJj-G6Uo-HG4q-auD6-lmyn-aLY0-o36HEj  LV Write Access        read/write  LV Creation host, time localhost.localdomain, 2021-01-15 00:47:35 +0800  LV Status              available  # open                 0  LV Size                148.00 MiB  Current LE             37  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     8192  Block device           253:2………………省略部分输出信息………………
```

**第4步**：把生成好的逻辑卷进行格式化，然后挂载使用。

Linux系统会把LVM中的逻辑卷设备存放在/dev设备目录中，实际上就是个快捷方式，同时会以卷组的名称来建立一个目录，其中保存了逻辑卷的设备映射文件，即/dev/卷组名称/逻辑卷名称。

```
[root@linuxprobe ~]# mkfs.ext4 /dev/storage/vo mke2fs 1.44.3 (10-July-2018)Creating filesystem with 151552 1k blocks and 38000 inodesFilesystem UUID: 429cbc28-4463-4a1b-b601-02a7cf81a1b2Superblock backups stored on blocks: 	8193, 24577, 40961, 57345, 73729Allocating group tables: done                            Writing inode tables: done                            Creating journal (4096 blocks): doneWriting superblocks and filesystem accounting information: done [root@linuxprobe ~]# mkdir /linuxprobe[root@linuxprobe ~]# mount /dev/storage/vo /linuxprobe
```

对了，如果用了LVM逻辑卷管理器的话，不建议用XFS文件系统。因为XFS文件系统自身就可以使用xfs_growfs命令进行磁盘扩容，虽然不比LVM灵活，但起码也是够用的。在实测阶段我们发现，有一些服务器上XFS与LVM兼容性并不好。

**第5步**：查看挂载状态，并写入到配置文件，使其永久生效。

```
[root@linuxprobe ~]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sr0                6.7G  6.7G     0 100% /media/cdrom/dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0/dev/mapper/storage-vo  140M  1.6M  128M   2% /linuxprobe[root@linuxprobe ~]# echo "/dev/storage/vo /linuxprobe ext4 defaults 0 0" >> /etc/fstab[root@linuxprobe ~]# cat /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                       /                       xfs      defaults        0 0UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot                   xfs      defaults        0 0/dev/mapper/rhel-swap                       swap                    swap     defaults        0 0/dev/cdrom                                  /media/cdrom            iso9660  defaults        0 0 /dev/storage/vo                             /linuxprobe             ext4     defaults        0 0
```

### **Tips**

细心的同学又发现了个小问题，刚刚明明写的是148M，怎么查看到只有140M了呢？这是因为硬件厂商制造标准是1GB=1,000MB、1MB＝1,000KB、1KB＝1,000byte，而计算机系统的算法是1GB=1,024MB、1MB＝1,024KB、1KB＝1,024byte，会有3%左右的“缩水”是正常情况。

###### **7.2.2 扩容逻辑卷**

在前面的实验中，卷组是由两块硬盘设备共同组成的。用户在使用存储设备时感知不到设备底层的架构和布局，更不用关心底层是由多少块硬盘组成的，只要卷组中有足够的资源，就可以一直为逻辑卷扩容。扩展前请一定要记得卸载设备和挂载点的关联。

```
[root@linuxprobe ~]# umount /linuxprobe
```

**第1步**：把上一个实验中的逻辑卷vo扩展至290M。

```
[root@linuxprobe ~]# lvextend -L 290M /dev/storage/voRounding size to boundary between physical extents: 292.00 MiB.Size of logical volume storage/vo changed from 148 MiB (37 extents) to 292 MiB (73 extents).Logical volume storage/vo successfully resized.
```

**第2步**：检查硬盘的完整性，确认目录结构、内容和文件内容没有丢失，一般情况下是要没有报错，均为正常情况。

```
[root@linuxprobe ~]# e2fsck -f /dev/storage/voe2fsck 1.44.3 (10-July-2018)Pass 1: Checking inodes, blocks, and sizesPass 2: Checking directory structurePass 3: Checking directory connectivityPass 4: Checking reference countsPass 5: Checking group summary information/dev/storage/vo: 11/38000 files (0.0% non-contiguous), 10453/151552 blocks
```

**第3步**：重置设备在系统中的容量，刚刚是对LV逻辑卷设备进行了扩容操作，但系统内核还没有同步到这部分新修改的信息，需要手动进行同步。

```
[root@linuxprobe ~]# resize2fs /dev/storage/voresize2fs 1.44.3 (10-July-2018)Resizing the filesystem on /dev/storage/vo to 299008 (1k) blocks.The filesystem on /dev/storage/vo is now 299008 (1k) blocks long.
```

**第4步**：重新挂载硬盘设备并查看挂载状态。

```
[root@linuxprobe ~]# mount -a[root@linuxprobe ~]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sr0                6.7G  6.7G     0 100% /media/cdrom/dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0/dev/mapper/storage-vo  279M  2.1M  259M   1% /linuxprobe
```

###### **7.2.3 缩小逻辑卷**

相较于扩容逻辑卷，在对逻辑卷进行缩容操作时，其丢失数据的风险更大。所以在生产环境中执行相应操作时，一定要提前备份好数据。另外Linux系统规定，在对LVM逻辑卷进行缩容操作之前，要先检查文件系统的完整性（当然这也是为了保证数据安全）。在执行缩容操作前记得先把文件系统卸载掉。

```
[root@linuxprobe ~]# umount /linuxprobe
```

**第1步**：检查文件系统的完整性。

```
[root@linuxprobe ~]# e2fsck -f /dev/storage/voe2fsck 1.44.3 (10-July-2018)Pass 1: Checking inodes, blocks, and sizesPass 2: Checking directory structurePass 3: Checking directory connectivityPass 4: Checking reference countsPass 5: Checking group summary information/dev/storage/vo: 11/74000 files (0.0% non-contiguous), 15507/299008 blocks
```

**第2步**：通知系统内核将逻辑卷vo的容量减小到120M。

```
[root@linuxprobe ~]# resize2fs /dev/storage/vo 120Mresize2fs 1.44.3 (10-July-2018)Resizing the filesystem on /dev/storage/vo to 122880 (1k) blocks.The filesystem on /dev/storage/vo is now 122880 (1k) blocks long.
```

**第3步**：将LV逻辑卷的容量修改为120M。

```
[root@linuxprobe ~]# lvreduce -L 120M /dev/storage/vo  WARNING: Reducing active logical volume to 120.00 MiB.  THIS MAY DESTROY YOUR DATA (filesystem etc.)Do you really want to reduce storage/vo? [y/n]: y  Size of logical volume storage/vo changed from 292 MiB (73 extents) to 120 MiB (30 extents).  Logical volume storage/vo successfully resized.
```

咦？步骤跟扩容是反过来了，缩小操作是先通知系统内核设备要改变成120M，然后再正式进行缩小操作，这是为什么呢？举个例子大家就明白了。小强是初中生，开学后看到班里有位同学纹身了感觉很酷，自己也想纹个身又怕家里责骂，于是他回家了就说：“爸妈，我纹身了”。如果他父母的反应是很平和，那么他就可以放心大胆的去纹身了~如果父母的态度非常强烈的不同意，他马上就可以哈哈一笑说逗着玩呢，也不会挨打。

所以缩小操作也是同样的道理，先通知系统内核自己想缩小逻辑卷，如果resize2fs命令执行后没有报错，系统允许了，再正式操作。

**第4步**：重新挂载文件系统并查看系统状态。

```
[root@linuxprobe ~]# mount -a[root@linuxprobe ~]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sr0                6.7G  6.7G     0 100% /media/cdrom/dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0/dev/mapper/storage-vo  113M  1.6M  103M   2% /linuxprobe
```

###### **7.2.4 逻辑卷快照**

LVM还具备有“快照卷”功能，该功能类似于虚拟机软件的还原时间点功能。例如，对某一个逻辑卷设备做一次快照，如果日后发现数据被改错了，就可以利用之前做好的快照卷进行覆盖还原。LVM的快照卷功能有两个特点：

> 快照卷的容量必须等同于逻辑卷的容量；
>
> 快照卷仅一次有效，一旦执行还原操作后则会被立即自动删除。

在正式操作前，先看看VG卷组中的容量是否够用：

```
[root@linuxprobe ~]# vgdisplay  --- Volume group ---  VG Name               storage  System ID               Format                lvm2  Metadata Areas        2  Metadata Sequence No  4  VG Access             read/write  VG Status             resizable  MAX LV                0  Cur LV                1  Open LV               1  Max PV                0  Cur PV                2  Act PV                2  VG Size               39.99 GiB  PE Size               4.00 MiB  Total PE              10238  Alloc PE / Size       30 / 120.00 MiB  Free  PE / Size       10208 / <39.88 GiB  VG UUID               k3ZnaP-wGPr-TQJ5-PCtA-0RgO-jvsi-9elZ5M………………省略部分输出信息………………
```

通过卷组的输出信息可以清晰看到，卷组中已经使用了120MB的容量，空闲容量还有39.88GB。接下来用重定向往逻辑卷设备所挂载的目录中写入一个文件。

```
[root@linuxprobe ~]# echo "Welcome to Linuxprobe.com" > /linuxprobe/readme.txt[root@linuxprobe ~]# ls -l /linuxprobetotal 14drwx------. 2 root root 12288 Jan 15 01:11 lost+found-rw-r--r--. 1 root root    26 Jan 15 07:01 readme.txt
```

**第1步**：使用-s参数生成一个快照卷，使用-L参数指定切割的大小，需要与要做快照的设备容量保持一致。另外，还需要在命令后面写上是针对哪个逻辑卷执行的快照操作，稍后数据也会还原到这个对应的设备上。

```
[root@linuxprobe ~]# lvcreate -L 120M -s -n SNAP /dev/storage/vo Logical volume "SNAP" created[root@linuxprobe ~]# lvdisplay  --- Logical volume ---  LV Path                /dev/storage/SNAP  LV Name                SNAP  VG Name                storage  LV UUID                qd7l6w-3Iv1-6E3X-RGkC-t5xl-170r-rDZSEf  LV Write Access        read/write  LV Creation host, time localhost.localdomain, 2021-01-15 07:02:44 +0800  LV snapshot status     active destination for vo  LV Status              available  # open                 0  LV Size                120.00 MiB  Current LE             30  COW-table size         120.00 MiB  COW-table LE           30  Allocated to snapshot  0.01%  Snapshot chunk size    4.00 KiB  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     8192  Block device           253:5………………省略部分输出信息………………
```

**第2步**：在逻辑卷所挂载的目录中创建一个100MB的垃圾文件，然后再查看快照卷的状态。可以发现存储空间占的用量上升了。

```
[root@linuxprobe ~]# dd if=/dev/zero of=/linuxprobe/files count=1 bs=100M1+0 records in1+0 records out104857600 bytes (105 MB, 100 MiB) copied, 0.312057 s, 336 MB/s[root@linuxprobe ~]# lvdisplay  --- Logical volume ---  LV Path                /dev/storage/SNAP  LV Name                SNAP  VG Name                storage  LV UUID                qd7l6w-3Iv1-6E3X-RGkC-t5xl-170r-rDZSEf  LV Write Access        read/write  LV Creation host, time localhost.localdomain, 2021-01-15 07:02:44 +0800  LV snapshot status     active destination for vo  LV Status              available  # open                 0  LV Size                120.00 MiB  Current LE             30  COW-table size         120.00 MiB  COW-table LE           30  Allocated to snapshot  83.71%  Snapshot chunk size    4.00 KiB  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     8192  Block device           253:5………………省略部分输出信息………………
```

**第3步**：为了校验SNAP快照卷的效果，需要对逻辑卷进行快照还原操作。在此之前记得先卸载掉逻辑卷设备与目录的挂载。

lvconvert命令用于管理逻辑卷的快照，语法格式为：“lvconvert [参数] 快照卷名称”。

使用lvconvert能够将逻辑卷的快照进行自动恢复，在早期5版本中要写全格式：“--mergesnapshot”，而从6版本开始至8版本，已经允许用户只输入“--merge”参数进行操作了，系统会自动分辨设备的类型。

```
[root@linuxprobe ~]# umount /linuxprobe[root@linuxprobe ~]# lvconvert --merge /dev/storage/SNAP  Merging of volume storage/SNAP started.  storage/vo: Merged: 36.41%  storage/vo: Merged: 100.00%
```

**第4步**：快照卷会被自动删除掉，并且刚刚在逻辑卷设备被执行快照操作后再创建出来的100MB的垃圾文件也被清除了。

```
[root@linuxprobe ~]# mount -a[root@linuxprobe ~]# cd /linuxprobe/[root@linuxprobe linuxprobe]# lslost+found readme.txt[root@linuxprobe linuxprobe]# cat readme.txt Welcome to Linuxprobe.com
```

###### **7.2.5 删除逻辑卷**

当生产环境中想要重新部署LVM或者不再需要使用时，则需要执行LVM的删除操作。为此，需要提前备份好重要的数据信息，然后依次删除逻辑卷、卷组、物理卷设备，这个顺序不可颠倒。

**第1步**：取消逻辑卷与目录的挂载关联，删除配置文件中永久生效的设备参数。

```
[root@linuxprobe ~]# umount /linuxprobe[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2020## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                       /                       xfs      defaults        0 0UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot                   xfs      defaults        0 0/dev/mapper/rhel-swap                       swap                    swap     defaults        0 0/dev/cdrom                                  /media/cdrom            iso9660  defaults        0 0 /dev/storage/vo                             /linuxprobe             ext4     defaults        0 0
```

**第2步**：删除逻辑卷设备，需要输入y来确认操作。

```
[root@linuxprobe ~]# lvremove /dev/storage/vo Do you really want to remove active logical volume storage/vo? [y/n]: y  Logical volume "vo" successfully removed
```

**第3步**：删除卷组，此处只写卷组名称即可，不需要设备的绝对路径。

```
[root@linuxprobe ~]# vgremove storage  Volume group "storage" successfully removed
```

**第4步**：删除物理卷设备。

```
[root@linuxprobe ~]# pvremove /dev/sdb /dev/sdc  Labels on physical volume "/dev/sdb" successfully wiped.  Labels on physical volume "/dev/sdc" successfully wiped.
```

在上述操作执行完毕之后，再执行lvdisplay、vgdisplay、pvdisplay命令来查看LVM的信息时就不会再看到信息了（前提是上述步骤的操作是正确的），干净利落！

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1． RAID技术主要是为了解决什么问题呢？

**答：**RAID技术可以解决存储设备的读写速度问题及数据的冗余备份问题。

2． RAID 0和RAID 5哪个更安全？

**答：**RAID 0没有数据冗余功能，因此RAID 5更安全。

3．假设使用4块硬盘来部署RAID 10方案，外加一块备份盘，最多可以允许几块硬盘同时损坏呢？

**答：**最多允许5块硬盘设备中的3块设备同时损坏。

4．位于LVM最底层的是物理卷还是卷组？

**答：**最底层的是物理卷，然后在通过物理卷组成卷组。

5． LVM对逻辑卷的扩容和缩容操作有何异同点呢？

**答：**扩容和缩容操作都需要先取消逻辑卷与目录的挂载关联；扩容操作是先扩容后检查文件系统完整性，而缩容操作为了保证数据的安全，需要先检查文件系统完整性再缩容。

6． LVM的快照卷能使用几次？

**答：**只可使用一次，而且使用后即自动删除。

7． LVM的删除顺序是怎么样的？

**答：**依次移除逻辑卷、卷组和物理卷。

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

### **Tips**

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
[root@Client A ~]# ssh 192.168.10.10The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.root@192.168.10.10's password: 此处输入服务器密码Activate the web console with: systemctl enable --now cockpit.socketLast login: Wed Jan 20 16:30:28 2021 from 192.168.10.1
```

然后，再使用IP地址在192.168.20.0/24网段内的主机访问服务器的22端口（虽网段不同，但已确认可以相互通信），效果如下，就会提示连接请求被拒绝了（Connection failed）：

```
[root@Client B ~]# ssh 192.168.10.10Connecting to 192.168.10.10:22...Could not connect to '192.168.10.10' (port 22): Connection failed.
```

**7．向INPUT规则链中添加拒绝所有人访问本机12345端口的策略规则**：

```
[root@linuxprobe ~]# iptables -I INPUT -p tcp --dport 12345 -j REJECT[root@linuxprobe ~]# iptables -I INPUT -p udp --dport 12345 -j REJECT[root@linuxprobe ~]# iptables -LChain INPUT (policy ACCEPT)target prot opt source destination  REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable………………省略部分输出信息………………
```

**8．向INPUT规则链中添加拒绝192.168.10.5主机访问本机80端口（Web服务）的策略规则**：

```
[root@linuxprobe ~]# iptables -I INPUT -p tcp -s 192.168.10.5 --dport 80 -j REJECT[root@linuxprobe ~]# iptables -LChain INPUT (policy ACCEPT)target prot opt source destination  REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable………………省略部分输出信息………………
```

**9．向INPUT规则链中添加拒绝所有主机访问本机1000**～**1024****端口的策略规则**：

刚刚添加防火墙策略时用的是-I参数，它默认会把规则添加到最上面位置，优先级是最高的。如果工作中需要添加一条最后“兜底”的规则 ，那就用-A参数吧，效果差别很大呦：

```
[root@linuxprobe ~]# iptables -A INPUT -p tcp --dport 1000:1024 -j REJECT[root@linuxprobe ~]# iptables -A INPUT -p udp --dport 1000:1024 -j REJECT[root@linuxprobe ~]# iptables -LChain INPUT (policy ACCEPT)target prot opt source destination  REJECT tcp -- 192.168.10.5 anywhere tcp dpt:http reject-with icmp-port-unreachable REJECT udp -- anywhere anywhere udp dpt:italk reject-with icmp-port-unreachable REJECT tcp -- anywhere anywhere tcp dpt:italk reject-with icmp-port-unreachable ACCEPT tcp -- 192.168.10.0/24 anywhere tcp dpt:ssh REJECT tcp -- anywhere anywhere tcp dpt:ssh reject-with icmp-port-unreachable REJECT tcp -- anywhere anywhere tcp dpts:cadlock2:1024 reject-with icmp-port-unreachable REJECT udp -- anywhere anywhere udp dpts:cadlock2:1024 reject-with icmp-port-unreachable………………省略部分输出信息………………
```

有关iptables命令的知识讲解到此就结束了，大家是不是意犹未尽？考虑到Linux防火墙的发展趋势，大家只要能把上面的实例吸收消化，就可以完全搞定日常的iptables配置工作了。但是请特别注意，使用iptables命令配置的防火墙规则默认会在系统下一次重启时失效，如果想让配置的防火墙策略永久生效，还要执行保存命令：

```
[root@linuxprobe ~]# iptables-save # Generated by xtables-save v1.8.2 on Wed Jan 20 16:56:27 2021………………省略部分输出信息………………
```

对了，如果公司服务器是5/6/7版本的话，对应的保存命令应该是：

```
[root@linuxprobe ~]# service iptables saveiptables: Saving firewall rules to /etc/sysconfig/iptables: [ OK ]
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

### **Tips**

Runtime：当前立即生效，重启后失效。

Permanent：当前不生效，重启后生效。

接下来的实验都很简单，但是提醒大家一定要仔细查看书中使用的是Runtime模式还是Permanent模式。如果不关注这个细节，就算是正确配置了防火墙策略，也可能无法达到预期的效果。

**1．查看firewalld服务当前所使用的区域：**

这是一步非常重要的操作，在配置防火墙策略前，必须查看当前生效的是那个区域，否则配置的防火墙策略将不会立即生效。

```
[root@linuxprobe ~]# firewall-cmd --get-default-zonepublic
```

**2．查询指定网卡在firewalld服务中绑定的区域：**

生产环境中，服务器大多不止有一块网卡，假如作为网关的服务器有两块网卡，一块对公网，一块对内网，那么这两块网卡对审查流量时的策略肯定也是不一致的。于是可以针对不同的来源方向的网卡绑定上不同的区域，实现对防火墙策略的灵活管控。

```
[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=ens160public
```

**3．把网卡默认区域修改为external，并在系统重启后生效：**

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=external --change-interface=ens160The interface is under control of NetworkManager, setting zone to 'external'.success[root@linuxprobe ~]# firewall-cmd --permanent --get-zone-of-interface=ens160external
```

**4．把firewalld服务的默认区域设置为public：**

默认区域也叫全局配置，指的是对所有网卡都生效的配置，优先级较低。我们可以看到当前默认区域为public，而ens160网卡的区域为external，这种情况便是以网卡的区域名称为准。

通俗来说默认区域就是一种通用的政策，例如食堂为所有人准备了一次性餐具，而如果是环保主义者，自己携带了筷子勺子，那么以您实际情况为准，如果没有带的话才用餐厅统一提供的。

```
[root@linuxprobe ~]# firewall-cmd --set-default-zone=publicWarning: ZONE_ALREADY_SET: publicsuccess[root@linuxprobe ~]# firewall-cmd --get-default-zone public[root@linuxprobe ~]# firewall-cmd --get-zone-of-interface=ens160externa
```

**5．启动和关闭firewalld防火墙服务的应急状况模式：**

如果想一秒钟阻断一切网络连接有什么好办法呢？大家潜意识一定马上会说：“拔掉网线！”，确实是个物理级别的高招~但如果人在北京，服务器却在异地呢？这时就用panic紧急模式吧。使用“--panic-on”参数会立即切断一切网络连接，而“--panic-off”则是恢复网络连接。切记紧急模式会切断一切网络连接，包括ping都是不通的，因此在远程管理服务器时，在按下回车键前一定要三思而后行。

```
[root@linuxprobe ~]# firewall-cmd --panic-onsuccess[root@linuxprobe ~]# firewall-cmd --panic-offsuccess
```

**6．查询ssh和https协议的流量是否允许放行：**

虽然工作中可以不使用“--zone”参数指定区域名称，firewall-cmd命令会自动依据默认区域进行查询，减少用户输入量。但如果默认区域与网卡所绑定的不一致时，就会发生冲突的情况，因此规范写法是一定要加的。

```
[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=sshyes[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=httpsno
```

**7．把https协议的流量设置为永久允许放行，并立即生效：**

默认情况下做的修改都属于Runtime模式，即当前生效而重启后失效，在工作和考试中尽量避免使用。而使用“--permanent”参数时则当前不会立即看到效果，而是重启或重新加载后方可生效。于是，当添加了允许放行https协议的流量后，查询当前模式策略依然是不允许：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=httpssuccess[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=httpsno
```

不想重启服务器的话，我们就用“--reload”参数吧：

```
[root@linuxprobe ~]# firewall-cmd --reloadsuccess[root@linuxprobe ~]# firewall-cmd --zone=public --query-service=httpsyes
```

**8．把http协议的流量设置为永久拒绝，并立即生效：**

由于默认来讲http协议流量就没有被允许，所以会有：“Warning: NOT_ENABLED: http”这样的提示信息，对实际操作没有影响。

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --remove-service=httpWarning: NOT_ENABLED: httpsuccess[root@linuxprobe ~]# firewall-cmd --reload success
```

**9．把访问8080和8081端口的流量策略设置为允许，但仅限当前生效：**

```
[root@linuxprobe ~]# firewall-cmd --zone=public --add-port=8080-8081/tcpsuccess[root@linuxprobe ~]# firewall-cmd --zone=public --list-ports8080-8081/tcp
```

**10．把原本访问本机888端口的流量转发到22端口，要且求当前和长期均有效：**

下一章节咱们会讲到的ssh远程控制协议是基于22/tcp端口号传输控制指令的，如果想让用户通过其他端口号也能访问到ssh服务，就可以试试端口号转发技术了。通过这项技术，新的端口号收到了用户请求后会自动转发到原本服务的端口上，使得用户能够通过新的端口号访问到原本的服务。

来举个例子帮助同学们理解，假如小强是电子厂的工人，喜欢上了三号流水线的工人小花，但不好意思表白，于是写了一封情书交给了门卫张大爷，再由张大爷转交给小花。信息的传输从小强到小花，变成了小强到张大爷再到小花，信息也能顺利送达。

使用firewall-cmd命令实现端口号转发的格式有点长，为同学们总结好了：

> firewall-cmd --permanent --zone=**<区域>** --add-forward-port=port=<源端口号>:proto=**<协议>**:toport=**<目标端口号>**:toaddr=**<目标IP地址>**

命令中的目标IP地址一般写服务器本机：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-forward-port=port=888:proto=tcp:toport=22:toaddr=192.168.10.10success[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

在客户端使用ssh命令尝试访问192.168.10.10主机的888端口，顺利成功：

```
[root@client A ~]# ssh -p 888 192.168.10.10The authenticity of host '[192.168.10.10]:888 ([192.168.10.10]:888)' can't be established.ECDSA key fingerprint is b8:25:88:89:5c:05:b6:dd:ef:76:63:ff:1a:54:02:1a.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '[192.168.10.10]:888' (ECDSA) to the list of known hosts.root@192.168.10.10's password:此处输入远程root管理员的密码Last login: Sun Jul 19 21:43:48 2021 from 192.168.10.10
```

**11．富规则的设置：**

富规则也叫复规则。表示更细致、更详细的防火墙策略配置，它可以针对系统服务、端口号、源地址和目标地址等诸多信息进行更有针对性的策略配置。它的优先级在所有的防火墙策略中也是最高的。比如，我们可以在firewalld服务中配置一条富规则，使其拒绝192.168.10.0/24网段的所有用户访问本机的ssh服务（22端口）：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-rich-rule="rule family="ipv4" source address="192.168.10.0/24" service name="ssh" reject"success[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

在客户端使用ssh命令尝试访问192.168.10.10主机的ssh服务（22端口）：

```
[root@client A ~]# ssh 192.168.10.10Connecting to 192.168.10.10:22...Could not connect to '192.168.10.10' (port 22): Connection failed.
```

### **Tips**

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

### **Tips**

系统镜像下载后就是iso格式结尾的文件，选中即可，无需解压。

然后，把光盘设备中的系统镜像挂载到/media/cdrom目录。

```
[root@linuxprobe ~]# mkdir -p /media/cdrom[root@linuxprobe ~]# mount /dev/cdrom /media/cdrommount: /media/cdrom: WARNING: device write-protected, mounted read-only.
```

为了能够一直为用户提供服务，更加严谨的做法是写入到/etc/fstab文件中，保证万无一失：

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Tue Jul 21 05:03:40 2021## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                       /                  xfs       defaults        0 0UUID=2db66eb4-d9c1-4522-8fab-ac074cd3ea0b   /boot              xfs       defaults        0 0/dev/mapper/rhel-swap                       swap               swap      defaults        0 0/dev/cdrom                                  /media/cdrom       iso9660   defaults        0 0 
```

最后，使用Vim文本编辑器创建软件仓库的配置文件。与过往版本的系统不同，RHEL8需要配置两个软件仓库，即[BaseOS]与[AppStream]缺一不可。下述命令中用到的具体参数的含义，可参考4.1.4小节。

```
[root@linuxprobe ~]# vim /etc/yum.repos.d/rhel8.repo[BaseOS]name=BaseOS。baseurl=file:///media/cdrom/BaseOSenabled=1gpgcheck=0[AppStream]name=AppStreambaseurl=file:///media/cdrom/AppStreamenabled=1gpgcheck=0
```

正确的配置软件仓库文件后，就可以开始用yum或dnf命令安装软件了。这两个命令在实际操作中除了名字不同外，执行方法完全一致，可随时用yum替代dnf命令。安装firewalld图形化界面工具：

```
[root@linuxprobe ~]# dnf install firewall-configUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                       3.1 MB/s | 3.2 kB     00:00    BaseOS                                          2.7 MB/s | 2.7 kB     00:00    Dependencies resolved.================================================================================ Package                Arch          Version            Repository        Size================================================================================Installing: firewall-config        noarch        0.6.3-7.el8        AppStream        157 kTransaction Summary================================================================================Install  1 PackageTotal size: 157 kInstalled size: 1.1 MIs this ok [y/N]: yDownloading Packages:Running transaction checkTransaction check succeeded.Running transaction testTransaction test succeeded.Running transaction  Preparing        :                                                        1/1   Installing       : firewall-config-0.6.3-7.el8.noarch                     1/1   Running scriptlet: firewall-config-0.6.3-7.el8.noarch                     1/1   Verifying        : firewall-config-0.6.3-7.el8.noarch                     1/1 Installed products updated.Installed:  firewall-config-0.6.3-7.el8.noarch                                            Complete!
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
[root@linuxprobe ~]# vim /etc/hosts.deny## hosts.deny This file contains access rules which are used to# deny connections to network services that either use# the tcp_wrappers library or that have been# started through a tcp_wrappers-enabled xinetd.## The rules in this file can also be set up in# /etc/hosts.allow with a 'deny' option instead.## See 'man 5 hosts_options' and 'man 5 hosts_access'# for information on rule syntax.# See 'man tcpd' for information on tcp_wrapperssshd:*[root@linuxprobe ~]# ssh 192.168.10.10ssh_exchange_identification: read: Connection reset by peer
```

接下来，在允许策略规则文件中添加一条规则，使其放行源自192.168.10.0/24网段，访问本机sshd服务的所有流量。可以看到，服务器立刻就放行了访问sshd服务的流量，效果非常直观：

```
[root@linuxprobe ~]# vim /etc/hosts.allow## hosts.allow This file contains access rules which are used to# allow or deny connections to network services that# either use the tcp_wrappers library or that have been# started through a tcp_wrappers-enabled xinetd.## See 'man 5 hosts_options' and 'man 5 hosts_access'# for information on rule syntax.# See 'man tcpd' for information on tcp_wrapperssshd:192.168.10.[root@linuxprobe ~]# ssh 192.168.10.10The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.ECDSA key fingerprint is 70:3b:5d:37:96:7b:2e:a5:28:0d:7e:dc:47:6a:fe:5c.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.root@192.168.10.10's password: Last login: Wed May 4 07:56:29 2021[root@linuxprobe ~]# 
```

##### **8.5 Cockpit驾驶舱管理工具**

首先，Cockpit是一个英文单词，即飞机驾驶舱的意思，它用名字传达出了功能丰富的特性。其次，Cockpit是一个基于网页的图形化工具，天然具备很好的跨平台性，被 广泛使用于服务器、容器、虚拟机等等多种管理场景，即便是新手都可以直接上手操作。最后要说的是红帽公司对Cockpit十分的看重，从最开始就默认安装到了RHEL 8系统中，衍生的CentOS和Fedora也都是标配它。

![第8章 Iptables与Firewalld防火墙第8章 Iptables与Firewalld防火墙](https://www.linuxprobe.com/wp-content/uploads/2020/05/Cockpit.jpg)

默认情况下就已经被安装过了，执行下dnf命令确认下也好：

```
[root@linuxprobe ~]# dnf install cockpitUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                       3.1 MB/s | 3.2 kB     00:00    BaseOS                                          2.7 MB/s | 2.7 kB     00:00    Package cockpit-185-2.el8.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

Cockpit服务程序在RHEL 8.0版本中没有自动运行，将它开启并加入到开机启动项中：

```
[root@linuxprobe ~]# systemctl start cockpit[root@linuxprobe ~]# systemctl enable cockpit.socketCreated symlink /etc/systemd/system/sockets.target.wants/cockpit.socket → /usr/lib/systemd/system/cockpit.socket.
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

### **Tips**

再多提一句，我们的这本《Linux就该这么学》不仅学习门槛低、简单易懂，而且还有一个潜在的优势——书中所有的服务器主机IP地址均为192.168.10.10，而客户端主机均为192.168.10.20及192.168.10.30。这样的好处就是，在后面部署Linux服务的时候，不用每次都要考虑IP地址变化的问题，从而可以心无旁骛地关注配置细节。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/6-8.png)

图9-6 填写IP地址和子网掩码

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/7-6.png)

图9-7 单击OK按钮保存配置

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/8-4.png)

图9-8 单击Back按钮结束配置工作

至此，在Linux系统中配置网络的步骤就结束了。

[刘遄](https://www.linuxprobe.com/)老师在培训时经常会发现，很多学员在安装RHEL 8系统时默认没有激活网卡。如果各位读者有同样的情况也不用担心，只需使用Vim编辑器将网卡配置文件中的ONBOOT参数修改成yes，这样在系统重启后网卡就被激活了。

```
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

```
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

```
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

```
[root@linuxprobe ~]# nmcli connection add con-name company ifname ens160 autoconnect no type ethernet ip4 192.168.10.10/24 gw4 192.168.10.1
Connection 'company' (6ac8f3ad-0846-42f4-819a-e1ae84f4da86) successfully added.
```

使用con-name参数指定家庭所使用的网络会话名称house。因为要从外部DHCP服务器自动获得IP地址，所以这里不需要进行手动指定。

```
[root@linuxprobe ~]# nmcli connection add con-name house type ethernet ifname ens160
Connection 'house' (d848242a-4bdf-4446-9079-6e12ab5d1f15) successfully added.
```

在成功创建网络会话后，可以使用nmcli命令查看创建的所有网络会话：

```
[root@linuxprobe ~]# nmcli connection show
NAME     UUID                                  TYPE      DEVICE 
ens160   97486c86-6d1e-4e99-9aa2-68d3172098b2  ethernet  ens160 
virbr0   e5fca1ee-7020-4c21-a65b-259d0f993b44  bridge    virbr0 
company  6ac8f3ad-0846-42f4-819a-e1ae84f4da86  ethernet  --     
house    d848242a-4bdf-4446-9079-6e12ab5d1f15  ethernet  --  
```

使用nmcli命令配置过的网络会话是永久生效的，这样当我们上班后，顺手启动company网络会话，网卡信息就自动配置好了：

```
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

```
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

```
[root@linuxprobe ~]# nmcli connection delete houseConnection 'house' (d848242a-4bdf-4446-9079-6e12ab5d1f15) successfully deleted.[root@linuxprobe ~]# nmcli connection delete company Connection 'company' (6ac8f3ad-0846-42f4-819a-e1ae84f4da86) successfully deleted.
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

```
[root@linuxprobe ~]# nmcli connection add type bond con-name bond0 ifname bond0 bond.options "mode=balance-rr"Connection 'bond0' (b37b720d-c5fa-43f8-8578-820d19811f32) successfully added.
```

这里使用的是balance-rr网卡绑定模式，rr是round-robin的缩写，全称也叫轮循模式。round-robin的特点是会根据设备顺序依次传输数据包，提供负载均衡的效果，让带宽变得更大，一旦某个网卡故障马上切换到另外一台设备上，保证网络传输不被中断。而active-backup网卡绑定模式也比较常用，特点是平时只有一块网卡正常工作，另一个设备待命，一旦出现损坏时自动顶替上去，冗余能力比较强，因此也被叫做主备模式。

比如有一台用于提供NFS或者samba服务的文件服务器，它所能提供的最大网络传输速度为100Mbit/s，但是访问该服务器的用户数量特别多，那么它的访问压力一定很大。在生产环境中，网络的可靠性是极为重要的，而且网络的传输速度也必须得以保证。针对这样的情况，比较好的选择就是balance-rr网卡绑定模式了。因为balance-rr能够让两块网卡同时一起工作，当其中一块网卡出现故障后能自动备援，且无需交换机设备支援，从而提供了可靠的网络传输保障。

**2．向bond0添加从属网卡**

刚刚创建成功的bond0设备当前仅仅是个名称，里面并没有真正能为用户传输数据的网卡设备，接下来要把ens160和ens192网卡添加进去。其中“con-name”参数后面接的是从属网卡名称，可以随时设置，而“ifname”参数后面接的是两块网卡名称，一定要以同学们实际为准，不要直接复制：

```
[root@linuxprobe ~]# nmcli connection add type ethernet slave-type bond con-name bond0-port1 ifname ens160 master bond0Connection 'bond0-port1' (8a2f77ee-cc92-4c11-9292-d577ccf8753d) successfully added.[root@linuxprobe ~]# nmcli connection add type ethernet slave-type bond con-name bond0-port2 ifname ens192 master bond0Connection 'bond0-port2' (b1ca9c47-3051-480a-9623-fbe4bf731a89) successfully added.
```

**3．配置bond0设备的网卡信息**

配置网卡参数的方法有很多，为了让咱们这个实验的配置过程更加具有一致性，所以决定还是用nmcli命令依次配置网卡的IP地址及子网掩码、网关、DNS、搜索域和手动配置等参数。如果同学们不习惯这个命令，也可以用编辑网卡配置文件或nmtui命令的方式完成下面操作：

```
[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.addresses 192.168.10.10/24[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.gateway 192.168.10.1[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.dns 192.168.10.1[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.dns-search linuxprobe.com[root@linuxprobe ~]# nmcli connection modify bond0 ipv4.method manual
```

**4．启动它！**

接下来就是最激动人心的时刻了，启动它吧！再顺便看下设备详细的列表：

```
[root@linuxprobe ~]# nmcli connection up bond0Connection successfully activated (master waiting for slaves) (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/22)[root@linuxprobe ~]# nmcli device statusDEVICE      TYPE      STATE      CONNECTION  bond0       bond      connected  bond0       ens160      ethernet  connected  ens160      virbr0      bridge    connected  virbr0      ens192      ethernet  connected  bond0-port2 lo          loopback  unmanaged  --          virbr0-nic  tun       unmanaged  --
```

当接下来用户访问192.168.10.10这个主机IP地址的时候，实际上是由两块网卡设备在共同提供服务。 可以在本地主机执行ping 192.168.10.10命令检查网络的连通性。为了检验网卡绑定技术的自动备援功能，我们突然在虚拟机硬件配置中随机移除一块网卡设备，如图9-13所示。可以非常清晰地看到网卡切换的过程（一般只有1个数据丢包），然后另外一块网卡会继续为用户提供服务。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E7%A7%BB%E9%99%A4%E6%8E%89%E4%B8%80%E5%9D%97%E7%BD%91%E5%8D%A1-1.png)

图9-13 随机移除掉任意一块网卡

```
[root@linuxprobe ~]# ping 192.168.10.10PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.109 ms64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.102 ms64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.066 msping: sendmsg: Network is unreachable64 bytes from 192.168.10.10: icmp_seq=5 ttl=64 time=0.065 ms64 bytes from 192.168.10.10: icmp_seq=6 ttl=64 time=0.048 ms64 bytes from 192.168.10.10: icmp_seq=7 ttl=64 time=0.042 ms64 bytes from 192.168.10.10: icmp_seq=8 ttl=64 time=0.079 ms^C--- 192.168.10.10 ping statistics ---8 packets transmitted, 7 received, 12% packet loss, time 7006msrtt min/avg/max/mdev = 0.042/0.073/0.109/0.023 ms
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

```
[root@Client ~]# ssh 192.168.10.10The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.root@192.168.10.10's password: 此处输入服务器管理员密码Activate the web console with: systemctl enable --now cockpit.socketLast login: Fri Jul 24 06:26:58 2020[root@Server ~]# [root@Server ~]# exitlogoutConnection to 192.168.10.10 closed.
```

如果禁止以root管理员的身份远程登录到服务器，则可以大大降低被黑客暴力破解密码的几率。下面进行相应配置。首先使用Vim文本编辑器打开服务器上的sshd服务主配置文件，然后把第46行#PermitRootLogin yes参数前的井号（#）去掉，并把参数值yes改成no，这样就不再允许root管理员远程登录了。记得最后保存文件并退出。

```
[root@Server ~]# vim /etc/ssh/sshd_config  ………………省略部分输出信息……………… 43 # Authentication: 44  45 #LoginGraceTime 2m 46 PermitRootLogin no 47 #StrictModes yes 48 #MaxAuthTries 6 49 #MaxSessions 10 ………………省略部分输出信息………………
```

再次提醒的是，一般的服务程序并不会在配置文件修改之后立即获得最新的参数。如果想让新配置文件生效，则需要手动重启相应的服务程序。最好也将这个服务程序加入到开机启动项中，这样系统在下一次启动时，该服务程序便会自动运行，继续为用户提供服务。

```
[root@Server ~]# systemctl restart sshd[root@Server ~]# systemctl enable sshd
```

这样一来，当root管理员再来尝试访问sshd服务程序时，系统会提示不可访问的错误信息。虽然sshd服务程序的参数相对比较简单，但这就是在Linux系统中配置服务程序的正确方法。大家要做的是举一反三、活学活用，这样即便以后遇到了陌生的服务，也一样可以搞定了。

```
[root@Client ~]# ssh 192.168.10.10root@192.168.10.10's password:此处输入服务器管理员密码Permission denied, please try again.
```

为了避免后续实验不能用root管理员账号登录了，请同学们再动手把上面的参数修改回来吧：

```
[root@Server ~]# vim /etc/ssh/sshd_config ………………省略部分输出信息………………  43 # Authentication: 44  45 #LoginGraceTime 2m 46 PermitRootLogin yes 47 #StrictModes yes 48 #MaxAuthTries 6 49 #MaxSessions 10 ………………省略部分输出信息………………[root@Server ~]# systemctl restart sshd [root@Server ~]# systemctl enable sshd
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

```
[root@Client ~]# ssh-keygenGenerating public/private rsa key pair.Enter file in which to save the key (/root/.ssh/id_rsa): 按回车键或设置密钥的存储路径Enter passphrase (empty for no passphrase): 直接按回车键或设置密钥的密码Enter same passphrase again: 再次按回车键或设置密钥的密码Your identification has been saved in /root/.ssh/id_rsa.Your public key has been saved in /root/.ssh/id_rsa.pub.The key fingerprint is:SHA256:kHa7B8V0nk63evABRrfZhxUpLM5Hx0I6gb7isNG9Hkg root@linuxprobe.comThe key's randomart image is:+---[RSA 2048]----+|          o.=.o.+||       . + =oB X ||      + o =oO O o||     . o + *.+ ..||      .ES . + o  ||     o.o.=   + . ||      =.o.o . o  ||     . . o.  .   ||        ..       |+----[SHA256]-----+
```

**第2步**：把客户端主机中生成的公钥文件传送至远程服务器：

```
[root@Client ~]# ssh-copy-id 192.168.10.10/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keysroot@192.168.10.10's password: 此处输入服务器管理员密码Number of key(s) added: 1Now try logging into the machine, with:   "ssh '192.168.10.10'"and check to make sure that only the key(s) you wanted were added.
```

**第3步**：对服务器进行设置，使其只允许密钥验证，拒绝传统的口令验证方式。记得在修改配置文件后保存并重启sshd服务程序。

```
[root@Server ~]# vim /etc/ssh/sshd_config  ………………省略部分输出信息……………… 70 # To disable tunneled clear text passwords, change to no here! 71 #PasswordAuthentication yes 72 #PermitEmptyPasswords no 73 PasswordAuthentication no 74  ………………省略部分输出信息………………[root@Server ~]# systemctl restart sshd
```

**第4步**：在客户端尝试登录到服务器，此时无须输入密码也可成功登录，特别方便：

```
[root@Client ~]# ssh 192.168.10.10Activate the web console with: systemctl enable --now cockpit.socketLast failed login: Thu Jan 28 13:44:09 CST 2021 from 192.168.10.20 on ssh:nottyThere were 2 failed login attempts since the last successful login.Last login: Thu Jan 28 13:22:34 2021 from 192.168.10.20
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

```
[root@Client ~]# echo "Welcome to LinuxProbe.Com" > readme.txt[root@Client ~]# scp /root/readme.txt 192.168.10.10:/homereadme.txt                                    100%   26    13.6KB/s   00:00
```

此外，还可以使用scp命令把远程服务器上的文件下载到本地主机，这样就无须先登录远程主机再进行文件传送了，也就省去了很多周折。例如，可以把远程主机的系统版本信息文件下载过来。

其命令格式为“scp [参数] 远程用户@远程IP地址:远程文件 本地目录”。

```
[root@Client ~]# scp 192.168.10.10:/etc/redhat-release /root[root@Client ~]# scp 192.168.10.10:/etc/redhat-release /rootredhat-release                                100%   45    23.6KB/s   00:00    [root@Client ~]# cat redhat-release Red Hat Enterprise Linux release 8.0 (Ootpa)
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

```
[root@linuxprobe ~]# dnf install tmuxUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                       3.1 MB/s | 3.2 kB     00:00    BaseOS                                          2.7 MB/s | 2.7 kB     00:00    Dependencies resolved.================================================================================ Package         Arch              Version              Repository         Size================================================================================Installing: tmux            x86_64            2.7-1.el8            BaseOS            317 kTransaction Summary================================================================================Install  1 PackageTotal size: 317 kInstalled size: 770 kIs this ok [y/N]: yDownloading Packages:Running transaction checkTransaction check succeeded.Running transaction testTransaction test succeeded.Running transaction  Preparing        :                                                        1/1   Installing       : tmux-2.7-1.el8.x86_64                                  1/1   Running scriptlet: tmux-2.7-1.el8.x86_64                                  1/1   Verifying        : tmux-2.7-1.el8.x86_64                                  1/1 Installed products updated.Installed:  tmux-2.7-1.el8.x86_64                                                         Complete!
```

### **Tips**

简捷起见，刘遄老师将对后面章节中出现的软件安装信息进行过滤—把重复性高及无意义的非必要信息省略。

###### **9.3.1 管理远程会话**

Tmux服务能做的事情非常多，例如创建不间断会话、恢复离线工作、界面切割窗格、会话共享功能等等，下面可以直接敲击tmux命令进入到会话窗口中，如图9-18所示。不难发现，进入到会话的终端底部出现了一个绿色的状态栏，分别显示的是会话编号、名称、主机名及系统时间。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/tmux.png)

图9-18 Tumx服务程序会话界面

退出会话的命令是exit，敲击后即可返回到正常的终端界面，如图9-19所示：

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E9%80%80%E5%87%BATmux.png)

图9-19 从会话窗口退回到终端界面

会话窗口的编号是从0开始自动排序，即0、1、2、3……，数量少还没关系，数量多的时候区分起来很麻烦。接下来创建一个指定名称为backup的会话窗口吧，请各位读者留心观察，当在命令行中敲下这条命令的一瞬间，屏幕会快速闪动一下，这时就已经进入Tmux服务会话中了，在里面运行的任何操作都会被后台记录下来。

```
[root@linuxprobe ~]# tmux new -s backup
```

突然要去忙其他事情了，但是会话窗口中执行的进程还不能被中断，此时便可以用detach参数将会话隐藏到后台。虽然看起来与刚才没有不同，但实际上可以查看到当前的会话正在工作中：

```
[root@linuxprobe ~]# tmux detach[detached (from session backup)]
```

如果觉得每次都要输入detach参数很麻烦，可以直接如图9-20所示关闭中断窗口（这与进行远程连接时突然断网具有相同的效果），Tmux服务程序会自动帮我们进行保存。

![第9章 使用ssh服务管理远程主机第9章 使用ssh服务管理远程主机](https://www.linuxprobe.com/wp-content/uploads/2021/01/%E5%85%B3%E9%97%AD%E4%BC%9A%E8%AF%9D-1024x613.png)

图9-20 强行关闭会话窗口

这样执行过后服务和进程都会一直在后台默默运行，不会因为窗口被关闭而造成数据丢失。不放心的话可以查看下后台有哪些会话：

```
[root@linuxprobe ~]# tmux lsbackup: 1 windows (created Thu Jan 28 15:57:40 2021) [80x23]
```

由于刚才关闭了会话窗口，这样的操作在传统的远程控制中一定会导致正在运行的命令也突然终止，但在不间断会话服务中则不会这样。我们只需查看一下刚刚离线的会话名称，然后尝试恢复回来就可以继续工作了，回归到刚刚backup中的方法很简单，再直接attach加会话编号或会话名称就可以的。刚刚离开时正在进行的一切工作状态都会被原原本本的被呈现出来，丝毫不影响：

```
[root@linuxprobe ~]# tmux attach -t backup
```

不需要使用这个Tmux会话了，不用费事先attach接入，再exit命令退出。其实可以直接用kill进行关闭的：

```
[root@linuxprobe ~]# tmux attach -t backup[exited][root@linuxprobe ~]# tmux lsno server running on /tmp/tmux-0/default
```

在日常的生产环境中，其实并不是必须先创建会话，然后再开始工作。直接使用tmux命令执行要运行的命令，这样在命令中的一切操作也都会被记录下来，当命令执行结束后后台会话也会自动结束。

```
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

```
[root@client A ~]# ssh 192.168.10.10The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.root@192.168.10.10's password: 此处输入服务器管理员密码Activate the web console with: systemctl enable --now cockpit.socketLast login: Fri Jul 24 06:26:58 2020[root@client A ~]# tmux new -s share
```

然后，使用ssh服务将客户端B也远程连接到服务器，并执行获取远程会话的命令。接下来，两台主机就能看到相同的内容了。

```
[root@client B ~]# ssh 192.168.10.10The authenticity of host '192.168.10.10 (192.168.10.10)' can't be established.ECDSA key fingerprint is SHA256:5d52kZi1la/FJK4v4jibLBZhLqzGqbJAskZiME6ZXpQ.Are you sure you want to continue connecting (yes/no)? yesWarning: Permanently added '192.168.10.10' (ECDSA) to the list of known hosts.root@192.168.10.10's password: 此处输入服务器管理员密码Activate the web console with: systemctl enable --now cockpit.socketLast login: Fri Jul 24 06:26:58 2020[root@client B ~]# tmux attach-session -t share
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

```
[root@linuxprobe ~]# journalctl -n 5 -- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 13:39:51 CS>Jan 31 13:33:54 localhost.localdomain systemd[1]: Started Fingerprint Authentic>Jan 31 13:33:55 localhost.localdomain gnome-keyring-daemon[2533]: couldn't init>Jan 31 13:33:55 localhost.localdomain gdm-password][4983]: gkr-pam: unlocked lo>Jan 31 13:33:56 localhost.localdomain NetworkManager[1203]:   [1612071236>Jan 31 13:39:51 localhost.localdomain cupsd[1230]: REQUEST localhost - - "POST >lines 1-6/6 (END)
```

还可以使用-f参数进行实时刷新日志最新内容，与第二章节学习过的tail -f /var/log/message命令效果相同：

```
[root@linuxprobe ~]# journalctl -f -- Logs begin at Fri 2020-07-24 05:59:38 CST. --Jan 31 13:33:54 localhost.localdomain dbus-daemon[1058]: [system] Activating via systemd: service name='net.reactivated.Fprint' unit='fprintd.service' requested by ':1.172' (uid=0 pid=2600 comm="/usr/bin/gnome-shell " label="unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023")Jan 31 13:33:54 localhost.localdomain systemd[1]: Starting Fingerprint Authentication Daemon...Jan 31 13:33:54 localhost.localdomain dbus-daemon[1058]: [system] Successfully activated service 'net.reactivated.Fprint'Jan 31 13:33:54 localhost.localdomain systemd[1]: Started Fingerprint Authentication Daemon.………………省略部分输出信息………………
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

```
[root@linuxprobe ~]# journalctl -p crit-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:06:07 CST. --Jul 24 05:59:38 localhost.localdomain kernel: Detected CPU family 6 model 158 stepping 13Jul 24 05:59:38 localhost.localdomain kernel: Warning: Intel Processor - this hardware has not undergone testing by Red Hat and might not be certified. Please consult https://hardware.redhat.com for certified hardware.………………省略部分输出信息………………
```

越用越顺手了，不仅能够根据日志类型进行检索，还可以用“--since”参数按照今日（today），近N小时（hour），指定时间范围的格式进行检索，找出最近的日志数据：

**1. 仅查询今日的日志信息**

```
[root@linuxprobe ~]# journalctl --since today-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --Jan 31 12:48:25 localhost.localdomain systemd[1]: Starting update of the root trust anchor >Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: sd_journal_get_cursor() fail>Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: journal reloaded... [v8.3>Jan 31 12:48:25 localhost.localdomain systemd[1]: Started update of the root trust anchor for>Jan 31 12:48:25 localhost.localdomain sssd[kcm][2764]: Shutting down………………省略部分输出信息………………
```

**2. 仅查询最近一小时的日志信息**

```
[root@linuxprobe ~]# journalctl --since "-1 hour"-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --Jan 31 14:25:36 localhost.localdomain systemd[1]: Starting dnf makecache...Jan 31 14:25:36 localhost.localdomain dnf[5516]: Updating Subscription Management repositories.Jan 31 14:25:36 localhost.localdomain dnf[5516]: Unable to read consumer identityJan 31 14:25:36 localhost.localdomain dnf[5516]: This system is not registered to Red Hat>Jan 31 14:25:36 localhost.localdomain dnf[5516]: Metadata cache refreshed recently.Jan 31 14:25:36 localhost.localdomain systemd[1]: Started dnf makecache.………………省略部分输出信息………………
```

**3. 仅查询从上午8点整到10点整的日志信息**

```
[root@linuxprobe ~]# journalctl --since "12:00" --until "14:00"-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --Jan 31 12:48:25 localhost.localdomain systemd[1]: Starting update of the root trust anchor>Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: sd_journal_get_cursor()>Jan 31 12:48:25 localhost.localdomain rsyslogd[1392]: imjournal: journal reloaded... [v8.37>Jan 31 12:48:25 localhost.localdomain systemd[1]: Started update of the root trust anchor>Jan 31 12:48:25 localhost.localdomain sssd[kcm][2764]: Shutting downJan 31 12:48:30 localhost.localdomain systemd[1]: Starting SSSD Kerberos Cache Manager...Jan 31 12:48:30 localhost.localdomain systemd[1]: Started SSSD Kerberos Cache Manager.Jan 31 12:48:30 localhost.localdomain sssd[kcm][3981]: Starting up………………省略部分输出信息………………
```

**4. 仅查询从2020年7月1日至2020年8月1日的日志信息**

```
[root@linuxprobe ~]# journalctl --since "2020-07-01" --until "2020-08-01"-- Logs begin at Fri 2020-07-24 05:59:38 CST, end at Sun 2021-01-31 15:10:01 CST. --Jul 24 05:59:38 localhost.localdomain kernel: Linux version 4.18.0-80.el8.x86_64 (mockbuild>Jul 24 05:59:38 localhost.localdomain kernel: Command line: BOOT_IMAGE=(hd0,msdos1)/vmlinuz>Jul 24 05:59:38 localhost.localdomain kernel: Disabled fast string operationsJul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x001: 'x87>Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x002: 'SSE>Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Supporting XSAVE feature 0x004: 'AVX>Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: xstate_offset[2]:  576, xstate_sizes>Jul 24 05:59:38 localhost.localdomain kernel: x86/fpu: Enabled xstate features 0x7, context>Jul 24 05:59:38 localhost.localdomain kernel: BIOS-provided physical RAM map:………………省略部分输出信息………………
```

**5. 查询指定服务的日志信息**

默认情况下所有的日志信息都是综合的、混在一起的，如果想看具体某项服务的日志信息，可以使用“_SYSTEMD_UNIT”参数进行查询，服务名称的后面还有.service，这个是标准服务名称的写法。：

```
[root@linuxprobe ~]# journalctl _SYSTEMD_UNIT=sshd.service-- Logs begin at Mon 2020-09-14 15:35:27 CST, end at Sun 2021-01-31 17:26:15 CST. --Nov 09 13:50:03 iZuf61gqesu0zmrcsma8x4Z sshd[1218]: Server listening on 0.0.0.0 port 22.Nov 09 13:58:45 iZuf61gqesu0zmrcsma8x4Z sshd[1218]: Received signal 15; terminating.-- Reboot --Nov 09 13:59:29 iZuf61gqesu0zmrcsma8x4Z sshd[1127]: Server listening on 0.0.0.0 port 22.Nov 09 14:12:12 iZuf61gqesu0zmrcsma8x4Z sshd[1262]: Accepted password for root from 111.196Nov 09 14:12:12 iZuf61gqesu0zmrcsma8x4Z sshd[1262]: pam_unix(sshd:session): session openedNov 09 14:14:31 iZuf61gqesu0zmrcsma8x4Z sshd[1127]: Received signal 15; terminating.………………省略部分输出信息………………
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



 [第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/basic-learning-10.html)

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
[root@linuxprobe ~]# systemctl restart httpd[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E6%9D%83%E9%99%90%E4%B8%8D%E8%B6%B3-1024x207.png)

图10-6 Web页面提示权限不足

##### **10.3 SELinux****安全子系统**

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2015/05/selinux.png)

SELinux（Security-Enhanced Linux）是美国国家安全局在Linux开源社区的帮助下开发的一个强制访问控制（MAC，Mandatory Access Control）的安全子系统。Linux系统使用SELinux技术的目的是为了让各个服务进程都受到约束，使其仅获取到本应获取的资源。

例如，您在自己的电脑上下载了一个美图软件，正全神贯注地使用它给照片进行美颜的时候，它却在后台默默监听着浏览器中输入的密码信息，而这显然不应该是它应做的事情（哪怕是访问电脑中的图片资源，都情有可原）。SELinux安全子系统就是为了杜绝此类情况而设计的，它能够从多方面监控违法行为：对服务程序的功能进行限制——SELinux域限制可以确保服务程序做不了出格的事情；对文件资源的访问限制——SELinux安全上下文确保文件资源只能被其所属的服务程序进行访问。

### **Tips**

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
[root@linuxprobe ~]# vim /etc/selinux/config# This file controls the state of SELinux on the system.# SELINUX= can take one of these three values:#     enforcing - SELinux security policy is enforced.#     permissive - SELinux prints warnings instead of enforcing.#     disabled - No SELinux policy is loaded.SELINUX=enforcing# SELINUXTYPE= can take one of these three values:#     targeted - Targeted processes are protected,#     minimum - Modification of targeted policy. Only selected processes are protected. #     mls - Multi Level Security protection.SELINUXTYPE=targeted
```

SELinux服务的主配置文件中，定义的是SELinux的默认运行状态，可以将其理解为系统重启后的状态，因此它不会在更改后立即生效。可以使用getenforce命令获得当前SELinux服务的运行模式：

```
[root@linuxprobe ~]# getenforce Enforcing
```

为了确认图10-6所示的结果确实是因为SELinux而导致的，可以用setenforce [0|1]命令修改SELinux当前的运行模式（0为禁用，1为启用）。注意，这种修改只是临时的，在系统重启后就会失效：

```
[root@linuxprobe ~]# setenforce 0[root@linuxprobe ~]# getenforcePermissive
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
[root@linuxprobe ~]# setenforce 1[root@linuxprobe ~]# ls -Zd /var/www/htmldrwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html[root@linuxprobe ~]# ls -Zd /home/wwwrootdrwxrwxrwx. root root unconfined_u:object_r:home_root_t:s0 /home/wwwroot
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
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/*
```

注意，执行上述设置之后，还无法立即访问网站，还需要使用restorecon命令将设置好的SELinux安全上下文立即生效。在使用restorecon命令时，可以加上-Rv参数对指定的目录进行递归操作，以及显示SELinux安全上下文的修改过程。最后，再次刷新页面，就可以正常看到网页内容了，结果如图10-8所示。

```
[root@linuxprobe ~]# restorecon -Rv /home/wwwroot/Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%86%8D%E6%AC%A1%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-1024x132.png)

图10-8 正常看到网页内容

真可谓是一波三折！原本认为只要把httpd服务程序配置妥当就可以大功告成，结果却反复受到了SELinux安全上下文的限制。所以，建议大家在配置httpd服务程序时，一定要细心、耐心。一旦成功配妥httpd服务程序之后，就会发现SELinux服务并没有那么难。

### **Tips**

由于在RHCSA、RHCE或RHCA考试中，都需要先重启您的机器然后再执行判分脚本。因此，建议读者在日常工作中要养成将所需服务添加到开机启动项中的习惯，比如这里就需要添加systemctl enable httpd命令。

##### **10.4 个人用户主页功能**

如果想在系统中为每位用户建立一个独立的网站，通常的方法是基于虚拟网站主机功能来部署多个网站。但这个工作会让管理员苦不堪言（尤其是用户数量很庞大时），而且在用户自行管理网站时，还会碰到各种权限限制，需要为此做很多额外的工作。其实，httpd服务程序提供的个人用户主页功能完全可以胜任这个工作。该功能可以让系统内所有的用户在自己的家目录中管理个人的网站，而且访问起来也非常容易。

**第1步**：在httpd服务程序中，默认没有开启个人用户主页功能。为此，我们需要编辑下面的配置文件，然后在第17行的UserDir disabled参数前面加上井号（#），表示让httpd服务程序开启个人用户主页功能；同时再把第24行的UserDir public_html参数前面的井号（#）去掉（UserDir参数表示网站数据在用户家目录中的保存目录名称，即public_html目录）。最后，在修改完毕后记得保存。

```
[root@linuxprobe ~]# vim /etc/httpd/conf.d/userdir.conf   1 #  2 # UserDir: The name of the directory that is appended onto a user's home  3 # directory if a ~user request is received.  4 #  5 # The path to the end user account 'public_html' directory must be  6 # accessible to the webserver userid.  This usually means that ~userid  7 # must have permissions of 711, ~userid/public_html must have permissions  8 # of 755, and documents contained therein must be world-readable.  9 # Otherwise, the client will only receive a "403 Forbidden" message. 10 # 11 <IfModule mod_userdir.c> 12     # 13     # UserDir is disabled by default since it can confirm the presence 14     # of a username on the system (depending on home directory 15     # permissions). 16     # 17     # UserDir disabled 18  19     # 20     # To enable requests to /~user/ to serve the user's public_html 21     # directory, remove the "UserDir disabled" line above, and uncomment 22     # the following line instead: 23     #  24       UserDir public_html 25 </IfModule> 26  27 # 28 # Control access to UserDir directories.  The following is an example 29 # for a site where these directories are restricted to read-only. 30 # 31 <Directory "/home/*/public_html"> 32     AllowOverride FileInfo AuthConfig Limit Indexes 33     Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec 34     Require method GET POST OPTIONS 35 </Directory>
```

**第2步**：在用户家目录中建立用于保存网站数据的目录及首页面文件。另外，还需要把家目录的权限修改为755，保证其他人也有权限读取里面的内容。

```
[root@linuxprobe home]# su - linuxprobe[linuxprobe@linuxprobe ~]$ mkdir public_html[linuxprobe@linuxprobe ~]$ echo "This is linuxprobe's website" > public_html/index.html[linuxprobe@linuxprobe ~]$ chmod -R 755 /home/linuxprobe
```

**第3步**：重新启动httpd服务程序，在浏览器的地址栏中输入网址，其格式为“网址/~用户名”（其中的波浪号是必需的，而且网址、波浪号、用户名之间没有空格），从理论上来讲就可以看到用户的个人网站了。不出所料的是，系统显示报错页面，如图10-9所示。这一定还是SELinux惹的祸。

```
[linuxprobe@linuxprobe ~]$ exitlogout[root@linuxprobe ~]# systemctl restart httpd
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/20210208134750-1024x184.png)

图10-9 禁止访问用户的个人网站

**第4步**：思考这次报错的原因是什么。httpd服务程序在提供个人用户主页功能时，该用户的网站数据目录本身就应该是存放到与这位用户对应的家目录中的，所以应该不需要修改家目录的SELinux安全上下文。但是，前文还讲到了SELinux域的概念。SELinux域确保服务程序不能执行违规的操作，只能本本分分地为用户提供服务。httpd服务中突然开启的这项个人用户主页功能到底有没有被SELinux域默认允许呢？

接下来使用getsebool命令查询并过滤出所有与HTTP协议相关的安全策略。其中，off为禁止状态，on为允许状态。

```
[root@linuxprobe ~]# getsebool -a | grep httphttpd_anon_write --> offhttpd_builtin_scripting --> onhttpd_can_check_spam --> offhttpd_can_connect_ftp --> offhttpd_can_connect_ldap --> offhttpd_can_connect_mythtv --> offhttpd_can_connect_zabbix --> offhttpd_can_network_connect --> offhttpd_can_network_connect_cobbler --> offhttpd_can_network_connect_db --> offhttpd_can_network_memcache --> offhttpd_can_network_relay --> offhttpd_can_sendmail --> offhttpd_dbus_avahi --> offhttpd_dbus_sssd --> offhttpd_dontaudit_search_dirs --> offhttpd_enable_cgi --> onhttpd_enable_ftp_server --> offhttpd_enable_homedirs --> offhttpd_execmem --> offhttpd_graceful_shutdown --> offhttpd_manage_ipa --> offhttpd_mod_auth_ntlm_winbind --> offhttpd_mod_auth_pam --> offhttpd_read_user_content --> offhttpd_run_ipa --> offhttpd_run_preupgrade --> offhttpd_run_stickshift --> offhttpd_serve_cobbler_files --> offhttpd_setrlimit --> offhttpd_ssi_exec --> offhttpd_sys_script_anon_write --> offhttpd_tmp_exec --> offhttpd_tty_comm --> offhttpd_unified --> offhttpd_use_cifs --> offhttpd_use_fusefs --> offhttpd_use_gpg --> offhttpd_use_nfs --> offhttpd_use_openstack --> offhttpd_use_sasl --> offhttpd_verify_dns --> offmysql_connect_http --> offnamed_tcp_bind_http_port --> offprosody_bind_http_port --> off
```

面对如此多的SELinux域安全策略规则，实在没有必要逐个理解它们，我们只要能通过名字大致猜测出相关的策略用途就足够了。比如，想要开启httpd服务的个人用户主页功能，那么用到的SELinux域安全策略应该是httpd_enable_homedirs吧？大致确定后就可以用setsebool命令来修改SELinux策略中各条规则的布尔值了。大家一定要记得在setsebool命令后面加上-P参数，让修改后的SELinux策略规则永久生效且立即生效。随后刷新网页，其效果如图10-10所示。

```
[root@linuxprobe ~]# setsebool -P httpd_enable_homedirs=on[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-1024x137.png)

图10-10 正常看到个人用户主页面中的内容

有时，网站的拥有者并不希望直接将网页内容显示出来，只想让通过身份验证的用户访客看到里面的内容。

**第1步**：先使用htpasswd命令生成密码数据库。-c参数表示第一次生成；后面再分别添加密码数据库的存放文件，以及验证要用到的用户名称（该用户不必是系统中已有的本地账户）。

```
[root@linuxprobe ~]# htpasswd -c /etc/httpd/passwd linuxprobeNew password:此处输入用于网页验证的密码Re-type new password:再输入一遍进行确认Adding password for user linuxprobe
```

**第2步**：继续编辑个人用户主页功能的配置文件。把第31～37行的参数信息修改成下列内容，其中井号（#）开头的内容为添加的注释信息，可将其忽略。随后保存并退出配置文件，重启httpd服务程序即可生效。

```
[root@linuxprobe ~]# vim /etc/httpd/conf.d/userdir.conf………………省略部分输出信息……………… 27 # 28 # Control access to UserDir directories.  The following is an example 29 # for a site where these directories are restricted to read-only. 30 # 31 <Directory "/home/*/public_html"> 32     AllowOverride all         #刚刚生成出的密码验证文件保存路径 33     authuserfile "/etc/httpd/passwd"         #当用户访问网站时的提示信息 34     authname "My privately website"        #验证方式为口令模式 35     authtype basic        #访问网站时需要验证的用户名称 36     require user linuxprobe 37 </Directory>[root@linuxprobe ~]# systemctl restart httpd
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
[root@linuxprobe ~]# nmcli connection up ens160 Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E7%BD%91%E7%BB%9C%E9%80%9A%E4%BF%A1-1024x612.png)

图10-15 分别检查3个IP地址的连通性

**第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/10[root@linuxprobe ~]# mkdir -p /home/wwwroot/20[root@linuxprobe ~]# mkdir -p /home/wwwroot/30[root@linuxprobe ~]# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html[root@linuxprobe ~]# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html[root@linuxprobe ~]# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
```

**第2步**：在httpd服务的配置文件中大约132行处开始，分别追加写入三个基于IP地址的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf………………省略部分输出信息………………132 <VirtualHost 192.168.10.10>133     DocumentRoot /home/wwwroot/10134     ServerName www.linuxprobe.com135     <Directory /home/wwwroot/10>136     AllowOverride None137     Require all granted138     </Directory>139 </VirtualHost>  140 <VirtualHost 192.168.10.20>141     DocumentRoot /home/wwwroot/20142     ServerName www.linuxcool.com143     <Directory /home/wwwroot/20>144     AllowOverride None145     Require all granted146     </Directory>147 </VirtualHost>  148 <VirtualHost 192.168.10.30>149     DocumentRoot /home/wwwroot/30150     ServerName www.linuxdown.com151     <Directory /home/wwwroot/30>152     AllowOverride None153     Require all granted154     </Directory>155 </VirtualHost>………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart httpd
```

**第3步**：此时访问网站，则会看到httpd服务程序的默认首页面，提示权限不足。大家现在应该立刻就反应过来—这是SELinux在捣鬼。由于当前的/home/wwwroot目录及里面的网站数据目录的SELinux安全上下文与网站服务不吻合，因此httpd服务程序无法获取到这些网站数据目录。我们需要手动把新的网站数据目录的SELinux安全上下文设置正确（见前文的实验），并使用restorecon命令让新设置的SELinux安全上下文立即生效，这样就可以立即看到网站的访问效果了，如图10-16所示。

```
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30/*[root@linuxprobe ~]# restorecon -Rv /home/wwwrootRelabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/10 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/10/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/20 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/20/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/30 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/30/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E4%B8%8D%E5%90%8C%E7%9A%84IP%E5%9C%B0%E5%9D%80%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x352.png)

图10-16 基于不同的IP地址访问虚拟主机网站

###### **10.5.2 基于主机域名**

当服务器无法为每个网站都分配一个独立IP地址的时候，可以尝试让Apache自动识别用户请求的域名，从而根据不同的域名请求来传输不同的内容。在这种情况下的配置更加简单，只需要保证位于生产环境中的服务器上有一个可用的IP地址（这里以192.168.10.10为例）就可以了。由于当前还没有介绍如何配置DNS解析服务，因此需要手工定义IP地址与域名之间的对应关系。/etc/hosts是Linux系统中用于强制把某个主机域名解析到指定IP地址的配置文件。简单来说，只要这个文件配置正确，即使网卡参数中没有DNS信息也依然能够将域名解析为某个IP地址。

**第1步**：手工定义IP地址与域名之间对应关系的配置文件，保存并退出后会立即生效。可以通过分别ping这些域名来验证域名是否已经成功解析为IP地址。

```
[root@linuxprobe ~]# vim /etc/hosts127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4::1         localhost localhost.localdomain localhost6 localhost6.localdomain6192.168.10.10   www.linuxprobe.com www.linuxcool.com www.linuxdown.com[root@linuxprobe ~]# ping -c 4 www.linuxprobe.comPING www.linuxprobe.com (192.168.10.10) 56(84) bytes of data.64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=1 ttl=64 time=0.070 ms64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=2 ttl=64 time=0.077 ms64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=3 ttl=64 time=0.061 ms64 bytes from www.linuxprobe.com (192.168.10.10): icmp_seq=4 ttl=64 time=0.069 ms--- www.linuxprobe.com ping statistics ---4 packets transmitted, 4 received, 0% packet loss, time 2999msrtt min/avg/max/mdev = 0.061/0.069/0.077/0.008 ms[root@linuxprobe ~]# 
```

**第2步**：分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxprobe[root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxcool[root@linuxprobe ~]# mkdir -p /home/wwwroot/linuxdown[root@linuxprobe ~]# echo "www.linuxprobe.com" > /home/wwwroot/linuxprobe/index.html[root@linuxprobe ~]# echo "www.linuxcool.com" > /home/wwwroot/linuxcool/index.html[root@linuxprobe ~]# echo "www.linuxdown.com" > /home/wwwroot/linuxdown/index.html
```

**第3步**：在httpd服务的配置文件中大约132行处开始，分别追加写入三个基于主机名的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf………………省略部分输出信息………………132 <VirtualHost 192.168.10.10>133     Documentroot /home/wwwroot/linuxprobe134     ServerName www.linuxprobe.com135     <Directory /home/wwwroot/linuxprobe>136     AllowOverride None137     Require all granted138     </Directory>139 </VirtualHost> 140 <VirtualHost 192.168.10.10>141     Documentroot /home/wwwroot/linuxcool142     ServerName www.linuxcool.com143     <Directory /home/wwwroot/linuxcool>144     AllowOverride None145     Require all granted146     </Directory>147 </VirtualHost> 148 <VirtualHost 192.168.10.10>149     Documentroot /home/wwwroot/linuxdown150     ServerName www.linuxdown.com151     <Directory /home/wwwroot/linuxdown>152     AllowOverride None153     Require all granted154     </Directory>155 </VirtualHost>………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart httpd
```

**第4步**：因为当前的网站数据目录还是在/home/wwwroot目录中，因此还是必须要正确设置网站数据目录文件的SELinux安全上下文，使其与网站服务功能相吻合。最后记得用restorecon命令让新配置的SELinux安全上下文立即生效，这样就可以立即访问到虚拟主机网站了，效果如图10-17所示。

```
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxprobe/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxcool/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/linuxdown/*[root@linuxprobe ~]# restorecon -Rv /home/wwwrootRelabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxprobe from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxprobe/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxcool from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxcool/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxdown from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/linuxdown/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0[root@linuxprobe ~]# firefox 
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E4%B8%BB%E6%9C%BA%E5%9F%9F%E5%90%8D%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x383.png)

图10-17 基于主机域名访问虚拟主机网站

###### **10.5.3 基于端口号**

基于端口号的虚拟主机功能可以让用户通过指定的端口号来访问服务器上的网站资源。在使用Apache配置虚拟网站主机功能时，基于端口号的配置方式是最复杂的。因此我们不仅要考虑httpd服务程序的配置因素，还需要考虑到SELinux服务对新开设端口的监控。一般来说，使用80、443、8080等端口号来提供网站访问服务是比较合理的，如果使用其他端口号则会受到SELinux服务的限制。

在接下来的实验中，我们不但要考虑到目录上应用的SELinux安全上下文的限制，还需要考虑SELinux域对httpd服务程序的管控。

**第1步**：分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便稍后能更直观地检查效果。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/6111[root@linuxprobe ~]# mkdir -p /home/wwwroot/6222[root@linuxprobe ~]# mkdir -p /home/wwwroot/6333[root@linuxprobe ~]# echo "port:6111" > /home/wwwroot/6111/index.html[root@linuxprobe ~]# echo "port:6222" > /home/wwwroot/6222/index.html[root@linuxprobe ~]# echo "port:6333" > /home/wwwroot/6333/index.html
```

**第2步**：在httpd服务配置文件的第46行至48行分别添加用于监听6111、6222和6333端口的参数。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf ………………省略部分输出信息………………  37 # Listen: Allows you to bind Apache to specific IP addresses and/or 38 # ports, instead of the default. See also the  39 # directive. 40 # 41 # Change this to Listen on specific IP addresses as shown below to  42 # prevent Apache from glomming onto all bound IP addresses. 43 # 44 #Listen 12.34.56.78:80 45 Listen 80 46 Listen 6111 47 Listen 6222 48 Listen 6333………………省略部分输出信息……………… 
```

**第3步**：在httpd服务的配置文件中大约134行处开始，分别追加写入三个基于端口号的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf………………省略部分输出信息……………… 134 <VirtualHost 192.168.10.10:6111>135     DocumentRoot /home/wwwroot/6111136     ServerName www.linuxprobe.com137     <Directory /home/wwwroot/6111>138     AllowOverride None139     Require all granted140     </Directory> 141 </VirtualHost>142 <VirtualHost 192.168.10.10:6222>143     DocumentRoot /home/wwwroot/6222144     ServerName www.linuxcool.com145     <Directory /home/wwwroot/6222>146     AllowOverride None147     Require all granted148     </Directory>149 </VirtualHost>150 <VirtualHost 192.168.10.10:6333>151     DocumentRoot /home/wwwroot/6333152     ServerName www.linuxdown.com153     <Directory /home/wwwroot/6333>154     AllowOverride None155     Require all granted156     </Directory>157 </VirtualHost>………………省略部分输出信息………………
```

**第4步**：因为我们把网站数据目录存放在/home/wwwroot目录中，因此还是必须要正确设置网站数据目录文件的SELinux安全上下文，使其与网站服务功能相吻合。最后记得用restorecon命令让新配置的SELinux安全上下文立即生效。

```
[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222/*[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333[root@linuxprobe ~]# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6333/*[root@linuxprobe ~]# restorecon -Rv /home/wwwroot/Relabeled /home/wwwroot from unconfined_u:object_r:user_home_dir_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6111 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6111/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6222 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6222/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6333 from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0Relabeled /home/wwwroot/6333/index.html from unconfined_u:object_r:user_home_t:s0 to unconfined_u:object_r:httpd_sys_content_t:s0[root@linuxprobe ~]# systemctl restart httpdJob for httpd.service failed because the control process exited with error code.See "systemctl status httpd.service" and "journalctl -xe" for details.
```

见鬼了！在妥当配置httpd服务程序和SELinux安全上下文并重启httpd服务后，竟然出现报错信息。这是因为SELinux服务检测到6111、6222和6333端口原本不属于Apache服务应该需要的资源，但现在却以httpd服务程序的名义监听使用了，所以SELinux会拒绝使用Apache服务使用这三个端口。我们可以使用semanage命令查询并过滤出所有与HTTP协议相关且SELinux服务允许的端口列表。

```
[root@linuxprobe ~]# semanage port -l | grep httphttp_cache_port_t            tcp      8080, 8118, 8123, 10001-10010http_cache_port_t            udp      3130http_port_t                  tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000pegasus_http_port_t          tcp      5988pegasus_https_port_t         tcp      5989
```

**第5步**：SELinux允许的与HTTP协议相关的端口号中默认没有包含6111、6222和6333，因此需要将这三个端口号手动添加进去。该操作会立即生效，而且在系统重启过后依然有效。设置好后再重启httpd服务程序，然后就可以看到网页内容了，结果如图10-18所示。

```
[root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6111[root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6222[root@linuxprobe ~]# semanage port -a -t http_port_t -p tcp 6333[root@linuxprobe ~]# semanage port -l | grep httphttp_cache_port_t            tcp      8080, 8118, 8123, 10001-10010http_cache_port_t            udp      3130http_port_t                  tcp      6333, 6222, 6111, 80, 81, 443, 488, 8008, 8009, 8443, 9000pegasus_http_port_t          tcp      5988pegasus_https_port_t         tcp      5989[root@linuxprobe ~]# systemctl restart httpd[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E5%9F%BA%E4%BA%8E%E7%AB%AF%E5%8F%A3%E5%8F%B7%E8%AE%BF%E9%97%AE%E8%99%9A%E6%8B%9F%E4%B8%BB%E6%9C%BA%E7%BD%91%E7%AB%99-1024x369.png)

图10-18 基于端口号访问虚拟主机网站

##### **10.6 Apache的访问控制**

Apache可以基于源主机名、源IP地址或源主机上的浏览器特征等信息对网站上的资源进行访问控制。它通过Allow指令允许某个主机访问服务器上的网站资源，通过Deny指令实现禁止访问。在允许或禁止访问网站资源时，还会用到Order指令，这个指令用来定义Allow或Deny指令起作用的顺序，其匹配原则是按照顺序进行匹配，若匹配成功则执行后面的默认指令。比如“Order Allow, Deny”表示先将源主机与允许规则进行匹配，若匹配成功则允许访问请求，反之则拒绝访问请求。

**第1步**：先在服务器上的网站数据目录中新建一个子目录，并在这个子目录中创建一个包含Successful单词的首页文件。

```
[root@linuxprobe ~]# mkdir /var/www/html/server[root@linuxprobe ~]# echo "Successful" > /var/www/html/server/index.html
```

**第2步**：打开httpd服务的配置文件，在第161行后面添加下述规则来限制源主机的访问。这段规则的含义是允许使用Firefox浏览器的主机访问服务器上的首页文件，除此之外的所有请求都将被拒绝。使用Firefox浏览器的访问效果如图10-19所示，而其它浏览器访问效果如图10-20所示。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf………………省略部分输出信息………………161 <Directory "/var/www/html/server">162     SetEnvIf User-Agent "Firefox" ff=1163     Order allow,deny164     Allow from env=ff165 </Directory>………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart httpd[root@linuxprobe ~]# firefox
```

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E6%88%90%E5%8A%9F-2-1024x130.png)

图10-19 火狐浏览器成功访问

![第10章 使用Apache服务部署静态网站第10章 使用Apache服务部署静态网站](https://www.linuxprobe.com/wp-content/uploads/2020/05/%E8%AE%BF%E9%97%AE%E5%A4%B1%E8%B4%A5-1024x115.png)

图10-20 其它浏览器访问失败

除了匹配源主机的浏览器特征之外，还可以通过匹配源主机的IP地址进行访问控制。例如，我们只允许IP地址为192.168.10.20的主机访问网站资源，那么就可以在httpd服务配置文件的第161行后面添加下述规则。这样在重启httpd服务程序后再用本机（即服务器，其IP地址为192.168.10.10）来访问网站的首页面时就会提示访问被拒绝了，如图10-21所示。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf………………省略部分输出信息………………161 <Directory "/var/www/html/server">162     Order allow,deny 163     Allow from 192.168.10.20164 </Directory>………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart httpd[root@linuxprobe ~]# firefox
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
[root@linuxprobe ~]# ls -ld /var/ftp/pubdrwxr-xr-x. 2 root root 6 Aug 13 2021 /var/ftp/pub[root@linuxprobe ~]# chown -R ftp /var/ftp/pub[root@linuxprobe ~]# ls -ld /var/ftp/pubdrwxr-xr-x. 2 ftp root 6 Aug 13 2021 /var/ftp/pub[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): anonymous331 Please specify the password.Password:此处敲击回车即可230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp> cd pub250 Directory successfully changed.ftp> mkdir files550 Create directory operation failed.
```

系统再次报错！尽管在使用ftp命令登入FTP服务器后，再创建目录时系统依然提示操作失败，但是报错信息却发生了变化。在没有写入权限时，系统提示“权限拒绝”（Permission denied）所以[刘遄](https://www.linuxprobe.com/)老师怀疑是权限的问题。但现在系统提示“创建目录的操作失败”（Create directory operation failed），想必各位读者也应该意识到是SELinux服务在“捣乱”了吧。

下面使用getsebool命令查看与FTP相关的SELinux域策略都有哪些：

```
[root@linuxprobe ~]# getsebool -a | grep ftpftpd_anon_write --> offftpd_connect_all_unreserved --> offftpd_connect_db --> offftpd_full_access --> offftpd_use_cifs --> offftpd_use_fusefs --> offftpd_use_nfs --> offftpd_use_passive_mode --> offhttpd_can_connect_ftp --> offhttpd_enable_ftp_server --> offtftp_anon_write --> offtftp_home_dir --> off
```

我们可以根据经验（需要长期培养，别无它法）和策略的名称判断出是ftpd_full_access--> off策略规则导致了操作失败。接下来修改该策略规则，并且在设置时使用-P参数让修改过的策略永久生效，确保在服务器重启后依然能够顺利写入文件。

```
[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

等SELinux域策略修改完毕，现在便能够顺利执行文件创建、修改及删除等操作了：

```
[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): anonymous331 Please specify the password.Password:此处敲击回车即可230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp> cd pub250 Directory successfully changed.ftp> mkdir files257 "/pub/files" createdftp> rename files database350 Ready for RNTO.250 Rename successful.ftp> rmdir database250 Remove directory operation successful.ftp> exit221 Goodbye.
```

在上面的操作中，由于权限的不足所以将/var/ftp/pub目录的所有者设置成了ftp用户本身。而除了这种方法外，也可以通过设置权限的方法让其它用户获取到写入权限，例如777这样的权限。但是由于vsftpd服务自身带有安全保护机制，不要对/var/ftp直接修改权限，有可能导致服务被“安全锁定”而不能登录，一定要记得是里面的pub目录哦：

```
[root@linuxprobe ~]# chmod -R 777 /var/ftp[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): anonymous331 Please specify the password.Password:此处敲击回车即可500 OOPS: vsftpd: refusing to run with writable root inside chroot()Login failed.421 Service not available, remote server has closed connection
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
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf  1 anonymous_enable=NO  2 local_enable=YES  3 write_enable=YES  4 local_umask=022  5 dirmessage_enable=YES  6 xferlog_enable=YES  7 connect_from_port_20=YES  8 xferlog_std_format=YES  9 listen=NO 10 listen_ipv6=YES 11 pam_service_name=vsftpd 12 userlist_enable=YES
```

在vsftpd服务程序的主配置文件中正确填写参数，然后保存并退出。还需要重启vsftpd服务程序，让新的配置参数生效。在执行完上一个实验后还原了虚拟机的读者，还需要将配置好的服务添加到开机启动项中，以便在系统重启自后依然可以正常使用vsftpd服务。

```
[root@linuxprobe ~]# systemctl restart vsftpd[root@linuxprobe ~]# systemctl enable vsftpdCreated symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

按理来讲，现在已经完全可以本地用户的身份登录FTP服务器了。但是在使用root管理员登录后，系统提示如下的错误信息：

```
[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): root530 Permission denied.Login failed.ftp> 
```

可见，在我们输入root管理员的密码之前，就已经被系统拒绝访问了。这是因为vsftpd服务程序所在的目录中默认存放着两个名为“用户名单”的文件（ftpusers和user_list）。不知道大家是否已看过一部日本电影“死亡笔记”，里面就提到有一个黑色封皮的小本子，只要将别人的名字写进去，这人就会挂掉。vsftpd服务程序目录中的这两个文件也有类似的功能—只要里面写有某位用户的名字，就不再允许这位用户登录到FTP服务器上。

```
[root@linuxprobe ~]# cat /etc/vsftpd/user_list # vsftpd userlist# If userlist_deny=NO, only allow users in this file# If userlist_deny=YES (default), never allow users in this file, and# do not even prompt for a password.# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers# for users that are denied.rootbindaemonadmlpsyncshutdownhaltmailnewsuucpoperatorgamesnobody[root@linuxprobe ~]# cat /etc/vsftpd/ftpusers # Users that are not allowed to login via ftprootbindaemonadmlpsyncshutdownhaltmailnewsuucpoperatorgamesnobody
```

果然如此！vsftpd服务程序为了保证服务器的安全性而默认禁止了root管理员和大多数系统用户的登录行为，这样可以有效地避免骇客通过FTP服务对root管理员密码进行暴力破解。如果您确认在生产环境中使用root管理员不会对系统安全产生影响，只需按照上面的提示删除掉root用户名即可，也可以选择ftpusers和user_list文件中没有的一个普通用户尝试登录FTP服务器：

```
[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): root331 Please specify the password.Password:此处输入该用户的密码230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp>
```

在继续后面实验之前，不知道同学们有没有思考一个问题——为什么同样是禁止用户登录的功能，却要做两个一摸一样的文件呢？

这个小玄机其实在user_list文件上面，如果把上面主配置文件中userlist_deny参数值改成NO，那么user_list列表就变成了强制白名单，功能完全是反过来的，只允许列表内的用户访问，拒绝其他人。

另外在采用本地用户模式登录FTP服务器后，默认访问的是该用户的家目录，而且该目录的默认所有者、所属组都是该用户自己，因此不存在写入权限不足的情况。但是当前的操作仍然被拒绝，是因为我们刚才将虚拟机系统还原到最初的状态了。为此，需要再次开启SELinux域中对FTP服务的允许策略：

```
[root@linuxprobe ~]# getsebool -a | grep ftpftpd_anon_write --> offftpd_connect_all_unreserved --> offftpd_connect_db --> offftpd_full_access --> offftpd_use_cifs --> offftpd_use_fusefs --> offftpd_use_nfs --> offftpd_use_passive_mode --> offhttpd_can_connect_ftp --> offhttpd_enable_ftp_server --> offtftp_anon_write --> offtftp_home_dir --> off[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

刘遄老师再啰嗦几句。在实验课程和生产环境中设置SELinux域策略时，一定记得添加-P参数，否则服务器在重启后就会按照原有的策略进行控制，从而导致配置过的服务无法使用。

在配置妥当后再使用本地用户尝试登录下FTP服务器，分别执行文件的创建、重命名及删除等命令。操作均成功！

```
[root@linuxprobe vsftpd]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): root331 Please specify the password.Password:此处输入该用户的密码230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp> mkdir files257 "/root/files" createdftp> rename files database350 Ready for RNTO.250 Rename successful.ftp> rmdir database250 Remove directory operation successful.ftp> exit221 Goodbye.
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
[root@linuxprobe ~]# cd /etc/vsftpd/[root@linuxprobe vsftpd]# vim vuser.listzhangsanredhatlisiredhat
```

但是，明文信息既不安全，也不符合让vsftpd服务程序直接加载的格式，因此需要使用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。

```
[root@linuxprobe vsftpd]# db_load -T -t hash -f vuser.list vuser.db[root@linuxprobe vsftpd]# chmod 600 vuser.db[root@linuxprobe vsftpd]# rm -f vuser.list
```

**第2步**：创建vsftpd服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户。FTP服务用于存储文件的根目录指的是，当虚拟用户登录后所访问的默认位置。

由于Linux系统中的每一个文件都有所有者、所属组属性，例如使用虚拟账户“张三”新建了一个文件，但是系统中找不到账户“张三”，就会导致这个文件的权限出现错误。为此，需要再创建一个可以映射到虚拟用户的系统本地用户。简单来说，就是让虚拟用户默认登录到与之有映射关系的这个系统本地用户的家目录中，虚拟用户创建的文件的属性也都归属于这个系统本地用户，从而避免Linux系统无法处理虚拟用户所创建文件的属性权限。

为了方便管理FTP服务器上的数据，可以把这个系统本地用户的家目录设置为/var目录（该目录用来存放经常发生改变的数据）。并且为了安全起见，我们将这个系统本地用户设置为不允许登录FTP服务器，这不会影响虚拟用户登录，而且还能够避免骇客通过这个系统本地用户进行登录。

```
[root@linuxprobe ~]# useradd -d /var/ftproot -s /sbin/nologin virtual[root@linuxprobe ~]# ls -ld /var/ftproot/drwx------. 3 virtual virtual 74 Jul 14 17:50 /var/ftproot/[root@linuxprobe ~]# chmod -Rf 755 /var/ftproot/
```

**第3步**：建立用于支持虚拟用户的PAM文件。

PAM可插拔认证模块是一种认证机制，通过一些动态链接库和统一的API把系统提供的服务与认证方式分开，使得系统管理员可以根据需求灵活调整服务程序的不同认证方式。要想把PAM功能和作用完全讲透，至少要一个章节的篇幅才行（对该主题感兴趣的读者敬请关注本书的进阶篇，里面会详细讲解PAM）。

通俗来讲，PAM是一组安全机制的模块，系统管理员可以用来轻易地调整服务程序的认证方式，而不必对应用程序进行任何修改。PAM采取了分层设计的思想，包含应用程序层、应用接口层、鉴别模块层，其结构如图11-2所示。

![第11章 使用Vsftpd服务传输文件第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/wp-content/uploads/2021/03/PAM%E7%9A%84%E5%88%86%E5%B1%82%E8%AE%BE%E8%AE%A1%E7%BB%93%E6%9E%84.jpg)

图11-2 PAM的分层设计结构

新建一个用于虚拟用户认证的PAM文件vsftpd.vu，其中PAM文件内的“db=”参数为使用db_load命令生成的账户密码数据库文件的路径，但不用写数据库文件的后缀：

```
[root@linuxprobe ~]# vim /etc/pam.d/vsftpd.vuauth       required     pam_userdb.so db=/etc/vsftpd/vuseraccount    required     pam_userdb.so db=/etc/vsftpd/vuser
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
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf    1 anonymous_enable=NO  2 local_enable=YES  3 write_enable=YES  4 guest_enable=YES  5 guest_username=virtual  6 allow_writeable_chroot=YES  7 local_umask=022  8 dirmessage_enable=YES  9 xferlog_enable=YES 10 connect_from_port_20=YES 11 xferlog_std_format=YES 12 listen=NO 13 listen_ipv6=YES 14 pam_service_name=vsftpd.vu 15 userlist_enable=YES
```

**第5步**：为虚拟用户设置不同的权限。虽然账户zhangsan和lisi都是用于vsftpd服务程序认证的虚拟账户，但是我们依然想对这两人进行区别对待。比如，允许张三上传、创建、修改、查看、删除文件，只允许李四查看文件。这可以通过vsftpd服务程序来实现。只需新建一个目录，在里面分别创建两个以zhangsan和lisi命名的文件，其中在名为zhangsan的文件中写入允许的相关权限（使用匿名用户的参数）：

```
[root@linuxprobe ~]# mkdir /etc/vsftpd/vusers_dir/[root@linuxprobe ~]# cd /etc/vsftpd/vusers_dir/[root@linuxprobe vusers_dir]# touch lisi[root@linuxprobe vusers_dir]# vim zhangsananon_upload_enable=YESanon_mkdir_write_enable=YESanon_other_write_enable=YES
```

然后再次修改vsftpd主配置文件，通过添加user_config_dir参数来定义这两个虚拟用户不同权限的配置文件所存放的路径。为了让修改后的参数立即生效，需要重启vsftpd服务程序并将该服务添加到开机启动项中：

```
[root@linuxprobe ~]# vim /etc/vsftpd/vsftpd.conf  1 anonymous_enable=NO  2 local_enable=YES  3 write_enable=YES  4 guest_enable=YES  5 guest_username=virtual  6 allow_writeable_chroot=YES  7 local_umask=022  8 dirmessage_enable=YES  9 xferlog_enable=YES 10 connect_from_port_20=YES 11 xferlog_std_format=YES 12 listen=NO 13 listen_ipv6=YES 14 pam_service_name=vsftpd.vu 15 userlist_enable=YES 16 user_config_dir=/etc/vsftpd/vusers_dir[root@linuxprobe ~]# systemctl restart vsftpd[root@linuxprobe ~]# systemctl enable vsftpdCreated symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
```

**第6步**：设置SELinux域允许策略，然后使用虚拟用户模式登录FTP服务器。相信大家可以猜到，SELinux会继续来捣乱。所以，先按照前面实验中的步骤开启SELinux域的允许策略，以免再次出现操作失败的情况：

```
[root@linuxprobe ~]# getsebool -a | grep ftpftpd_anon_write --> offftpd_connect_all_unreserved --> offftpd_connect_db --> offftpd_full_access --> offftpd_use_cifs --> offftpd_use_fusefs --> offftpd_use_nfs --> offftpd_use_passive_mode --> offhttpd_can_connect_ftp --> offhttpd_enable_ftp_server --> offtftp_anon_write --> offtftp_home_dir --> off[root@linuxprobe ~]# setsebool -P ftpd_full_access=on
```

此时，不但可以使用虚拟用户模式成功登录到FTP服务器，还可以分别使用账户zhangsan和lisi来检验他们的权限。李四用户只能登录，没有其余权限：

```
[root@linuxprobe ~]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): lisi331 Please specify the password.Password:此处输入虚拟用户的密码230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp> mkdir files550 Permission denied.ftp> exit221 Goodbye.
```

而张三用户不仅可以登录，还可以创建、改名和删除文件，满权限状态。当然，读者在生产环境中一定要根据真实需求来灵活配置参数，不要照搬这里的实验操作。

```
[root@linuxprobe vusers_dir]# ftp 192.168.10.10Connected to 192.168.10.10 (192.168.10.10).220 (vsFTPd 3.0.3)Name (192.168.10.10:root): zhangsan331 Please specify the password.Password:此处输入虚拟用户的密码230 Login successful.Remote system type is UNIX.Using binary mode to transfer files.ftp> mkdir files257 "/files" createdftp> rename files database350 Ready for RNTO.250 Rename successful.ftp> rmdir database250 Remove directory operation successful.
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
[root@linuxprobe ~]# dnf install tftp-server tftp xinetdUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                               3.1 MB/s | 3.2 kB     00:00    BaseOS                                                  2.7 MB/s | 2.7 kB     00:00    Dependencies resolved.======================================================================================== Package              Arch            Version                  Repository          Size========================================================================================Installing: tftp                 x86_64          5.2-24.el8               AppStream           42 k tftp-server          x86_64          5.2-24.el8               AppStream           50 k xinetd               x86_64          2:2.3.15-23.el8          AppStream          135 kTransaction Summary========================================================================================Install  3 PackagesTotal size: 227 kInstalled size: 397 kIs this ok [y/N]: yDownloading Packages:Running transaction checkTransaction check succeeded.Running transaction testTransaction test succeeded.Running transaction  Preparing        :                                                                1/1   Installing       : xinetd-2:2.3.15-23.el8.x86_64                                  1/3   Running scriptlet: xinetd-2:2.3.15-23.el8.x86_64                                  1/3   Installing       : tftp-server-5.2-24.el8.x86_64                                  2/3   Running scriptlet: tftp-server-5.2-24.el8.x86_64                                  2/3   Installing       : tftp-5.2-24.el8.x86_64                                         3/3   Running scriptlet: tftp-5.2-24.el8.x86_64                                         3/3   Verifying        : tftp-5.2-24.el8.x86_64                                         1/3   Verifying        : tftp-server-5.2-24.el8.x86_64                                  2/3   Verifying        : xinetd-2:2.3.15-23.el8.x86_64                                  3/3 Installed products updated.Installed:  tftp-5.2-24.el8.x86_64  tftp-server-5.2-24.el8.x86_64  xinetd-2:2.3.15-23.el8.x86_64 Complete!
```

在Linux系统中，TFTP服务是使用xinetd服务程序来管理的。xinetd服务可以用来管理多种轻量级的网络服务，而且具有强大的日志功能，专门用于管理那些比较小的应用程序的开关工作，有点类似于有独立控制的插线板那样，如图11-3所示，想开启那个服务就编辑对应的xinetd配置文件的开关参数。

![第11章 使用Vsftpd服务传输文件第11章 使用Vsftpd服务传输文件](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%9C%AA%E6%A0%87%E9%A2%98-1-1-1.jpg)

图11-3 一个带有独立开关的插线板

简单来说，在安装TFTP软件包后，还需要在xinetd服务程序中将其开启。在RHEL 8版本系统中tftp所对应的配置文件默认不存在，需要用户根据示例文件（/usr/share/doc/xinetd/sample.conf）自行创建。读者们直接复制下面内容到文件中，可直接使用：

```
[root@linuxprobe ~]# vim /etc/xinetd.d/tftpservice tftp{        socket_type             = dgram        protocol                = udp        wait                    = yes        user                    = root        server                  = /usr/sbin/in.tftpd        server_args             = -s /var/lib/tftpboot        disable                 = no        per_source              = 11        cps                     = 100 2        flags                   = IPv4}
```

然后，重启xinetd服务并将它添加到系统的开机启动项中，以确保TFTP服务在系统重启后依然处于运行状态。考虑到有些系统的防火墙默认没有允许UDP协议的69端口，因此需要手动将该端口号加入到防火墙的允许策略中：

```
[root@linuxprobe ~]# systemctl restart xinetd[root@linuxprobe ~]# systemctl enable xinetd[root@linuxprobe ~]# firewall-cmd --zone=public --permanent --add-port=69/udpsuccess[root@linuxprobe ~]# firewall-cmd --reload success
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
[root@linuxprobe ~]# echo "i love linux" > /var/lib/tftpboot/readme.txt[root@linuxprobe ~]# tftp 192.168.10.10tftp> get readme.txttftp> quit[root@linuxprobe ~]# lsanaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  readme.txt  VideosDesktop          Downloads  Music                 Public    Templates[root@linuxprobe ~]# cat readme.txt i love linux
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
[root@linuxprobe ~]# iptables -F[root@linuxprobe ~]# iptables-save [root@linuxprobe ~]# firewall-cmd --zone=public --permanent --add-service=sambasuccess[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

**第6步：**在服务器本地检查samba服务是否启动可以用“systemctl status smb”进行查看，而如果想进一步看samba服务都共享出去了哪些共享目录，则可以用smbclient命令来查看共享详情，-U参数指定了用户名称，建议一会用哪位用户进行挂载，就用哪位用户身份进行查看；-L参数列举共享清单。

```
[root@linuxprobe ~]# smbclient -U linuxprobe -L 192.168.10.10Enter SAMBA\linuxprobe's password: 此处输入该账户在Samba服务数据库中的密码	Sharename       Type      Comment	---------       ----      -------	database        Disk      Do not arbitrarily modify the database file	IPC$            IPC       IPC Service (Samba 4.9.1)Reconnecting with SMB1 for workgroup listing.	Server               Comment	---------            -------	Workgroup            Master	---------            -------
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
[root@linuxprobe ~]# dnf install cifs-utilsUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                                    3.1 MB/s | 3.2 kB     00:00    BaseOS                                                       2.7 MB/s | 2.7 kB     00:00    Dependencies resolved.============================================================================================= Package                Arch               Version                  Repository          Size=============================================================================================Installing: cifs-utils             x86_64             6.8-2.el8                BaseOS              93 kTransaction Summary=============================================================================================Install  1 Package         ………………省略部分输出信息………………    Installed:  cifs-utils-6.8-2.el8.x86_64                                                     Complete!
```

安装好软件包后，在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，可以与服务器上的共享名称同名，这样便于日后查找。mount命令的-t参数指定协议类型，-o参数指定用户命和密码，最后追加上服务器IP地址和共享名称和本地挂载目录即可。服务器IP地址后面的共享名称指的是配置文件中[database]的值，而不是服务器本地挂载的目录名称，虽然这两个值可能一样，但读者们应该认出它们的区别。

```
[root@linuxprobe ~]# mkdir /database[root@linuxprobe ~]# mount -t cifs -o username=linuxprobe,password=redhat //192.168.10.10/database /database[root@linuxprobe ~]# df -hFilesystem                Size  Used Avail Use% Mounted ondevtmpfs                  969M     0  969M   0% /devtmpfs                     984M     0  984M   0% /dev/shmtmpfs                     984M  9.6M  974M   1% /runtmpfs                     984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root      17G  3.9G   14G  23% //dev/sr0                  6.7G  6.7G     0 100% /media/cdrom/dev/sda1                1014M  152M  863M  15% /boottmpfs                     197M   16K  197M   1% /run/user/42tmpfs                     197M  3.4M  194M   2% /run/user/0//192.168.10.10/database   17G  3.9G   14G  23% /database
```

如果说每次重启电脑后都需要再手动的mount挂载一下远程共享目录，是不是觉得很麻烦呢？其实我们可以按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中，然后让/etc/fstab文件和系统自动的加载它。为了保证不被其他人随意看到，最后把这个认证文件的权限修改为仅root管理员才能够读写：

```
[root@linuxprobe ~]# vim auth.smbusername=linuxprobepassword=redhatdomain=MYGROUP[root@linuxprobe ~]# chmod 600 auth.smb
```

挂载信息写入到/etc/fstab文件中，以确保共享挂载信息在服务器重启后依然生效：

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Thu Feb 25 10:42:11 2021## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                      /             xfs     defaults                    0 0UUID=37d0bdc6-d70d-4cc0-b356-51195ad90369  /boot         xfs     defaults                    1 0/dev/mapper/rhel-swap                      swap          swap    defaults                    0 0/dev/cdrom                                 /media/cdrom  iso9660 defaults                    0 0 //192.168.10.10/database                   /database     cifs    credentials=/root/auth.smb  0 0[root@linuxprobe ~]# mount -a
```

Linux客户端成功地挂载了Samba服务的共享资源。进入到挂载目录/database后就可以看到Windows系统访问Samba服务程序时留下来的文件了（即文件Memo.txt）。当然，也可以对该文件进行读写操作并保存。

```
[root@linuxprobe ~]# cat /database/Memo.txti can edit it .
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
[root@linuxprobe ~]# dnf install nfs-utilsUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:00:12 ago on Sat 06 Mar 2021 04:48:38 AM CST.Package nfs-utils-1:2.3.3-14.el8.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

**第1步**：为了检验NFS服务配置的效果，我们需要使用两台Linux主机（一台充当NFS服务器，一台充当NFS客户端），并按照表12-6来设置它们所使用的IP地址。

表12-6               两台Linux主机所使用的操作系统以及IP地址

| 主机名称  | 操作系统 | IP地址        |
| --------- | -------- | ------------- |
| NFS服务器 | RHEL 8   | 192.168.10.10 |
| NFS客户端 | RHEL 8   | 192.168.10.20 |



另外，不要忘记配置好防火墙，以免默认的防火墙策略禁止正常的NFS共享服务。

```
[root@linuxprobe ~]# iptables -F[root@linuxprobe ~]# iptables-save[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=nfssuccess[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=rpc-bindsuccess[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=mountdsuccess[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

**第2步**：在NFS服务器上建立用于NFS文件共享的目录，并设置足够的权限确保其他人也有写入权限。

```
[root@linuxprobe ~]# mkdir /nfsfile[root@linuxprobe ~]# chmod -R 777 /nfsfile[root@linuxprobe ~]# echo "welcome to linuxprobe.com" > /nfsfile/readme
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
[root@linuxprobe ~]# vim /etc/exports/nfsfile 192.168.10.*(rw,sync,root_squash)
```

在NFS服务的配置文件中巧用通配符能够实现很多便捷功能，就比如匹配IP地址就有三种方法——第一种是直接写*号，代表任何主机都可以访问；第二种则是实验中采用的192.168.10.*通配格式，代表来自192.168.10.0/24网段的主机；第三种则是直接写对方的IP地址，如192.168.10.20，代表仅允许某个主机进行访问。

**第4步**：启动和启用NFS服务程序。由于在使用NFS服务进行文件共享之前，需要使用RPC（Remote Procedure Call，远程过程调用）服务将NFS服务器的IP地址和端口号等信息发送给客户端。因此，在启动NFS服务之前，还需要顺带重启并启用rpcbind服务程序，并将这两个服务一并加入开机启动项中。

```
[root@linuxprobe ~]# systemctl restart rpcbind[root@linuxprobe ~]# systemctl enable rpcbind[root@linuxprobe ~]# systemctl start nfs-server[root@linuxprobe ~]# systemctl enable nfs-serverCreated symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
```

NFS客户端的配置步骤也十分简单。先使用showmount命令查询NFS服务器的远程共享信息，必要的参数见表12-8，其输出格式为“共享的目录名称 允许使用客户端地址”。

表12-8                 showmount命令中可用的参数以及作用

| 参数 | 作用                                      |
| ---- | ----------------------------------------- |
| -e   | 显示NFS服务器的共享列表                   |
| -a   | 显示本机挂载的文件资源的情况NFS资源的情况 |
| -v   | 显示版本号                                |



```
[root@linuxprobe ~]# showmount -e 192.168.10.10Export list for 192.168.10.10:/nfsfile 192.168.10.*
```

然后在NFS客户端创建一个挂载目录。使用mount命令并结合-t参数，指定要挂载的文件系统的类型，并在命令后面写上服务器的IP地址、服务器上的共享目录以及要挂载到本地系统（即客户端）的目录。

```
[root@linuxprobe ~]# mkdir /nfsfile[root@linuxprobe ~]# mount -t nfs 192.168.10.10:/nfsfile /nfsfile[root@linuxprobe ~]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sr0                6.7G  6.7G     0 100% /media/cdrom/dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile
```

挂载成功后就应该能够顺利地看到在执行前面的操作时写入的文件内容了。如果希望NFS文件共享服务能一直有效，则需要将其写入到fstab文件中：

```
[root@linuxprobe ~]# cat /nfsfile/readmewelcome to linuxprobe.com[root@linuxprobe ~]# vim /etc/fstab ## /etc/fstab# Created by anaconda on Thu Feb 25 10:42:11 2021## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                     /                       xfs     defaults        0 0UUID=37d0bdc6-d70d-4cc0-b356-51195ad90369 /boot                   xfs     defaults        0 0/dev/mapper/rhel-swap                     swap                    swap    defaults        0 0/dev/cdrom                                /media/cdrom            iso9660 defaults        0 0 192.168.10.10:/nfsfile                    /nfsfile                nfs     defaults        0 0   
```

##### **12.3 AutoFs自动挂载服务**

无论是Samba服务还是NFS服务，都要把挂载信息写入到/etc/fstab中，这样远程共享资源就会自动随服务器开机而进行挂载。虽然这很方便，但是如果挂载的远程资源太多，则会给网络带宽和服务器的硬件资源带来很大负载。如果在资源挂载后长期不使用，也会造成服务器硬件资源的浪费。可能会有读者说，“可以在每次使用之前执行mount命令进行手动挂载”。这是一个不错的选择，但是每次都需要先挂载再使用，您不觉得麻烦吗？

autofs自动挂载服务可以帮我们解决这一问题。与mount命令不同，autofs服务程序是一种Linux系统守护进程，当检测到用户试图访问一个尚未挂载的文件系统时，将自动挂载该文件系统。换句话说，将挂载信息填入/etc/fstab文件后，系统在每次开机时都自动将其挂载，而autofs服务程序则是在用户需要使用该文件系统时才去动态挂载，从而节约了网络资源和服务器的硬件资源。

需要自行安装下autofs服务程序：

```
[root@linuxprobe ~]# dnf install autofsUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:28:58 ago on Sat 06 Mar 2021 04:57:01 AM CST.Dependencies resolved.======================================================================================== Package           Arch              Version                    Repository         Size========================================================================================Installing: autofs            x86_64            1:5.1.4-29.el8             BaseOS            755 kTransaction Summary========================================================================================Install  1 Package………………省略部分输出信息………………Installed:  autofs-1:5.1.4-29.el8.x86_64                                                          Complete!
```

处于生产环境中的Linux服务器，一般会同时管理许多设备的挂载操作。如果把这些设备挂载信息都写入到autofs服务的主配置文件中，无疑会让主配置文件臃肿不堪，不利于服务执行效率，也不利于日后修改里面的配置内容，因此在autofs服务程序的主配置文件中需要按照“挂载目录 子配置文件”的格式进行填写。挂载目录是设备挂载位置的上一级目录。例如，光盘设备一般挂载到/media/cdrom目录中，那么挂载目录写成/media即可。对应的子配置文件则是对这个挂载目录内的挂载设备信息作进一步的说明。子配置文件需要用户自行定义，文件名字没有严格要求，但后缀建议以.misc结束。具体的配置参数如第7行的加粗字所示。

```
[root@linuxprobe ~]# vim /etc/auto.master## Sample auto.master file# This is a 'master' automounter map and it has the following format:# mount-point [map-type[,format]:]map [options]# For details of the format look at auto.master(5).#/media  /etc/iso.misc/misc   /etc/auto.misc## NOTE: mounts done from a hosts map will be mounted with the#       "nosuid" and "nodev" options unless the "suid" and "dev"#       options are explicitly given.#/net    -hosts## Include /etc/auto.master.d/*.autofs# The included files must conform to the format of this file.#+dir:/etc/auto.master.d## If you have fedfs set up and the related binaries, either# built as part of autofs or installed from another package,# uncomment this line to use the fedfs program map to access# your fedfs mounts.#/nfs4  /usr/sbin/fedfs-map-nfs4 nobind## Include central master map if it can be found using# nsswitch sources.## Note that if there are entries for /net or /misc (as# above) in the included master map any keys that are the# same will not be seen as the first read key seen takes# precedence.#+auto.master
```

在子配置文件中，应按照“挂载目录 挂载文件类型及权限 :设备名称”的格式进行填写。例如，要把光盘设备挂载到/media/iso目录中，可将挂载目录写为iso，而-fstype为文件系统格式参数，iso9660为光盘设备格式，ro、nosuid及nodev为光盘设备具体的权限参数，/dev/cdrom则是定义要挂载的设备名称。配置完成后再顺手将autofs服务程序启动并加入到系统启动项中：

```
[root@linuxprobe ~]# vim /etc/iso.misciso   -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom[root@linuxprobe ~]# systemctl start autofs [root@linuxprobe ~]# systemctl enable autofs Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service → /usr/lib/systemd/system/autofs.service.
```

接下来将发生一件非常有趣的事情。先查看当前的光盘设备挂载情况，确认光盘设备没有被挂载上，而且/media目录中根本就没有iso子目录：

```
[root@linuxprobe ~]# umount /dev/cdrom[root@linuxprobe ~]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile[root@linuxprobe ~]# cd /media[root@linuxprobe media]# ls[root@linuxprobe media]#
```

但是，我们却可以使用cd命令切换到这个iso子目录中，而且光盘设备会被立即自动挂载上，也就能顺利查看光盘内的内容了。

```
[root@linuxprobe media]# cd iso[root@linuxprobe iso]# lsAppStream  EULA              images      RPM-GPG-KEY-redhat-betaBaseOS     extra_files.json  isolinux    RPM-GPG-KEY-redhat-releaseEFI        GPL               media.repo  TRANS.TBL[root@linuxprobe iso]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0192.168.10.10:/nfsfile   17G  3.9G   14G  23% /nfsfile/dev/sr0                6.7G  6.7G     0 100% /media/iso
```

### **Tips**

咦？怎么光盘设备的名称变成了/dev/sr0呢？实际上和/dev/cdrom是连接方式，只是名称不同而已。

```
[root@linuxprobe ~]# ls -l /dev/cdromlrwxrwxrwx. 1 root root 3 Feb 26 00:09 /dev/cdrom -> sr0
```

是不是很有方便，趁着刚学完的知识还没忘，再对NFS服务动手试试吧。

首先应该把NFS共享目录先卸载掉，然后在autofs服务程序的主配置文件中会有一条“/misc /etc/auto.misc”参数，这个auto.misc相当于自动挂载的参考文件，而它默认就已经存在，所以这步我们不需要进行任何操作：

```
[root@linuxprobe ~]# umount /nfsfile[root@linuxprobe ~]# vim /etc/auto.master## Sample auto.master file# This is a 'master' automounter map and it has the following format:# mount-point [map-type[,format]:]map [options]# For details of the format look at auto.master(5).#/media   /etc/iso.misc/misc    /etc/auto.misc………………省略部分输出信息………………
```

接下来找到这个对应的auto.misc文件，填写本地挂载的路径和NFS服务器的挂载信息，如下所示：

```
[root@linuxprobe ~]# vim /etc/auto.misc## This is an automounter map and it has the following format# key [ -mount-options-separated-by-comma ] location# Details may be found in the autofs(5) manpagenfsfile         192.168.10.10:/nfsfilecd              -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom# the following entries are samples to pique your imagination#linux          -ro,soft,intr           ftp.example.org:/pub/linux#boot           -fstype=ext2            :/dev/hda1#floppy         -fstype=auto            :/dev/fd0#floppy         -fstype=ext2            :/dev/fd0#e2floppy       -fstype=ext2            :/dev/fd0#jaz            -fstype=ext2            :/dev/sdc1#removable      -fstype=ext2            :/dev/hdd
```

这样填写信息后重启autofs服务程序，进入到/misc/nfsfile目录时，共享信息便会自动挂载：

```
[root@linuxprobe ~]# systemctl restart autofs[root@linuxprobe ~]# cd /misc/nfsfile[root@linuxprobe nfsfile]# df -hFilesystem              Size  Used Avail Use% Mounted ondevtmpfs                969M     0  969M   0% /devtmpfs                   984M     0  984M   0% /dev/shmtmpfs                   984M  9.6M  974M   1% /runtmpfs                   984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root    17G  3.9G   14G  23% //dev/sda1              1014M  152M  863M  15% /boottmpfs                   197M   16K  197M   1% /run/user/42tmpfs                   197M  3.4M  194M   2% /run/user/0192.168.10.10:/nfsfile   17G  3.9G   14G  23% /misc/nfsfile/dev/sr0                6.7G  6.7G     0 100% /media/iso
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
[root@linuxprobe ~]# nslookup> 192.168.10.1010.10.168.192.in-addr.arpa	name = www.linuxprobe.com.> 192.168.10.2020.10.168.192.in-addr.arpa	name = bbs.linuxprobe.com.
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
[root@linuxprobe ~]# vim /etc/named.rfc1912.zoneszone "linuxprobe.com" IN {        type master;        file "linuxprobe.com.zone";        allow-update { 192.168.10.20;};};zone "10.168.192.in-addr.arpa" IN {        type master;        file "192.168.10.arpa";        allow-update { 192.168.10.20;};};[root@linuxprobe ~]# systemctl restart named
```

**第2步**：在主服务器上配置防火墙放行规则，让DNS协议流量可以被顺利传递：

```
[root@linuxprobe ~]# iptables -F [root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=dnssuccess[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

**第3步**：在从服务器上安装bind-chroot软件包（输出信息省略），修改配置文件让从服务器也能够对外提供DNS服务，并且测试与主服务器的网络连通性：

```
[root@linuxprobe ~]# dnf install bind-chroot[root@linuxprobe ~]# vim /etc/named.conf  1 //  2 // named.conf  3 //  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS  5 // server as a caching only nameserver (as a localhost DNS resolver only).  6 //  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.  8 //  9  10 options { 11         listen-on port 53 { any; }; 12         listen-on-v6 port 53 { ::1; }; 13         directory       "/var/named"; 14         dump-file       "/var/named/data/cache_dump.db"; 15         statistics-file "/var/named/data/named_stats.txt"; 16         memstatistics-file "/var/named/data/named_mem_stats.txt"; 17         secroots-file   "/var/named/data/named.secroots"; 18         recursing-file  "/var/named/data/named.recursing"; 19         allow-query     { any; };………………省略部分输出信息………………[root@linuxprobe ~]# ping -c 4 192.168.10.10 PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=2.44 ms64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=3.31 ms64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.503 ms64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.359 ms--- 192.168.10.10 ping statistics ---4 packets transmitted, 4 received, 0% packet loss, time 15msrtt min/avg/max/mdev = 0.359/1.654/3.311/1.262 ms
```

**第4步**：在从服务器中填写主服务器的IP地址与要抓取的区域信息，然后重启服务。注意此时的服务类型应该是slave（从），而不再是master（主）。masters参数后面应该为主服务器的IP地址，而且file参数后面定义的是同步数据配置文件后要保存到的位置，稍后可以在该目录内看到同步的文件。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zoneszone "linuxprobe.com" IN {        type slave;        masters { 192.168.10.10; };        file "slaves/linuxprobe.com.zone";};zone "10.168.192.in-addr.arpa" IN {        type slave;        masters { 192.168.10.10; };        file "slaves/192.168.10.arpa";};[root@linuxprobe ~]# systemctl restart named
```

### **Tips**

这里的masters参数比正常的主服务类型master多了个字母s，代表主服务器可以有多个的意思，请小心，不要漏掉呦~

**第5步**：检验解析结果。当从服务器的DNS服务程序在重启后，一般就已经自动从主服务器上同步了数据配置文件，而且该文件默认会放置在区域配置文件中所定义的目录位置中。随后修改从服务器的网络参数，把DNS地址参数修改成192.168.10.20，这样即可使用从服务器自身提供的DNS域名解析服务。最后就可以使用nslookup命令顺利看到解析结果了。

```
[root@linuxprobe ~]# cd /var/named/slaves[root@linuxprobe slaves]# ls 192.168.10.arpa linuxprobe.com.zone[root@linuxprobe slaves]# nslookup> www.linuxprobe.comServer:		192.168.10.20Address:	192.168.10.20#53Name:	www.linuxprobe.comAddress: 192.168.10.10> 192.168.10.1010.10.168.192.in-addr.arpa	name = www.linuxprobe.com.
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
[root@linuxprobe ~]# ls -al /var/named/slaves/total 8drwxrwx---. 2 named named  56 Mar 12 09:53 .drwxrwx--T. 6 root  named 141 Mar 12 09:57 ..-rw-r--r--. 1 named named 436 Mar 12 09:53 192.168.10.arpa-rw-r--r--. 1 named named 282 Mar 12 09:53 linuxprobe.com.zone[root@linuxprobe ~]# rm -rf /var/named/slaves/*
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
[root@linuxprobe ~]# dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slaveKmaster-slave.+157+62533[root@linuxprobe ~]# ls -l Kmaster-slave.+157+62533.*-rw-------. 1 root root  56 Mar 14 09:54 Kmaster-slave.+157+62533.key-rw-------. 1 root root 165 Mar 14 09:54 Kmaster-slave.+157+62533.private[root@linuxprobe ~]# cat Kmaster-slave.+157+62533.private Private-key-format: v1.3Algorithm: 157 (HMAC_MD5)Key: NI6icnb74FxHx2gK+0MVOg==Bits: AAA=Created: 20210314015436Publish: 20210314015436Activate: 20210314015436
```

**第2步**：在主服务器中创建密钥验证文件。进入bind服务程序用于保存配置文件的目录，把刚刚生成的密钥名称、加密算法和私钥加密字符串按照下面格式写入到tansfer.key传输配置文件中。为了安全起见，需要将文件的所属组修改成named，并将文件权限设置得要小一点，然后把该文件做一个硬链接到/etc目录中。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc/[root@linuxprobe etc]# vim transfer.keykey "master-slave" {        algorithm hmac-md5;        secret "NI6icnb74FxHx2gK+0MVOg==";};[root@linuxprobe etc]# chown root:named transfer.key[root@linuxprobe etc]# chmod 640 transfer.key[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第3步**：开启并加载Bind服务的密钥验证功能。首先需要在主服务器的主配置文件中加载密钥验证文件，然后进行设置，使得只允许带有master-slave密钥认证的DNS服务器同步数据配置文件：

```
[root@linuxprobe ~]# vim /etc/named.conf  1 //  2 // named.conf  3 //  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS  5 // server as a caching only nameserver (as a localhost DNS resolver only).  6 //  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.  8 //  9 include "/etc/transfer.key"; 10 options { 11         listen-on port 53 { any; }; 12         listen-on-v6 port 53 { ::1; }; 13         directory       "/var/named"; 14         dump-file       "/var/named/data/cache_dump.db"; 15         statistics-file "/var/named/data/named_stats.txt"; 16         memstatistics-file "/var/named/data/named_mem_stats.txt"; 17         secroots-file   "/var/named/data/named.secroots"; 18         recursing-file  "/var/named/data/named.recursing"; 19         allow-query     { any; }; 20         allow-transfer { key master-slave; };………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart named
```

至此，DNS主服务器的TSIG密钥加密传输功能就已经配置完成。此时清空DNS从服务器同步目录中所有的数据配置文件，然后再次重启bind服务程序，这时就已经不能像刚才那样自动获取到数据配置文件了。

```
[root@linuxprobe ~]# rm -rf /var/named/slaves/*[root@linuxprobe ~]# systemctl restart named[root@linuxprobe ~]# ls  /var/named/slaves/
```

**第4步**：配置从服务器，使其支持密钥验证。配置DNS从服务器和主服务器的方法大致相同，都需要在bind服务程序的配置文件目录中创建密钥认证文件，并设置相应的权限，然后把该文件做一个硬链接到/etc目录中。

```
[root@linuxprobe ~]# cd /var/named/chroot/etc/[root@linuxprobe etc]# vim transfer.keykey "master-slave" {        algorithm hmac-md5;        secret "NI6icnb74FxHx2gK+0MVOg==";};[root@linuxprobe etc]# chown root:named transfer.key[root@linuxprobe etc]# chmod 640 transfer.key[root@linuxprobe etc]# ln transfer.key /etc/transfer.key
```

**第5步**：开启并加载从服务器的密钥验证功能。这一步的操作步骤也同样是在主配置文件中加载密钥认证文件，然后按照指定格式写上主服务器的IP地址和密钥名称。注意，密钥名称等参数位置不要太靠前，大约在第51行比较合适，否则bind服务程序会因为没有加载完预设参数而报错：

```
[root@linuxprobe etc]# vim /etc/named.conf  1 //  2 // named.conf  3 //  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS  5 // server as a caching only nameserver (as a localhost DNS resolver only).  6 //  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.  8 //  9 include "/etc/transfer.key"; 10 options { 11         listen-on port 53 { any; }; 12         listen-on-v6 port 53 { ::1; }; 13         directory       "/var/named"; 14         dump-file       "/var/named/data/cache_dump.db"; 15         statistics-file "/var/named/data/named_stats.txt"; 16         memstatistics-file "/var/named/data/named_mem_stats.txt"; 17         secroots-file   "/var/named/data/named.secroots"; 18         recursing-file  "/var/named/data/named.recursing"; 19         allow-query     { any; }; 20  21         /*  22          - If you are building an AUTHORITATIVE DNS server, do NOT enable recursion. 23          - If you are building a RECURSIVE (caching) DNS server, you need to enable  24            recursion.  25          - If your recursive DNS server has a public IP address, you MUST enable access  26            control to limit queries to your legitimate users. Failing to do so will 27            cause your server to become part of large scale DNS amplification  28            attacks. Implementing BCP38 within your network would greatly 29            reduce such attack surface  30         */ 31         recursion yes; 32  33         dnssec-enable yes; 34         dnssec-validation yes; 35  36         managed-keys-directory "/var/named/dynamic"; 37  38         pid-file "/run/named/named.pid"; 39         session-keyfile "/run/named/session.key"; 40  41         /* https://fedoraproject.org/wiki/Changes/CryptoPolicy */ 42         include "/etc/crypto-policies/back-ends/bind.config"; 43 }; 44  45 logging { 46         channel default_debug { 47                 file "data/named.run"; 48                 severity dynamic; 49         }; 50 }; 51 server 192.168.10.10 52 { 53         keys { master-slave; }; 54 }; 55 zone "." IN { 56         type hint; 57         file "named.ca"; 58 }; 59  60 include "/etc/named.rfc1912.zones"; 61 include "/etc/named.root.key"; 62 
```

**第6步**：DNS从服务器同步域名区域数据。现在，两台服务器的bind服务程序都已经配置妥当，并匹配到了相同的密钥认证文件。接下来在从服务器上重启bind服务程序，可以发现又能顺利地同步到数据配置文件了。

```
[root@linuxprobe ~]# systemctl restart named[root@linuxprobe ~]# ls /var/named/slaves/192.168.10.arpa  linuxprobe.com.zone
```

**第7步**：再次进行解析验证，功能正常，注意看是由192.168.10.20从服务器进行解析的呦：

```
[root@linuxprobe etc]# nslookup www.linuxprobe.comServer:		192.168.10.20Address:	192.168.10.20#53Name:	www.linuxprobe.comAddress: 192.168.10.10[root@linuxprobe etc]# nslookup 192.168.10.1010.10.168.192.in-addr.arpa	name = www.linuxprobe.com.
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
[root@linuxprobe ~]# nmcli connection up ens192Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/5)
```

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E6%9F%A5%E7%9C%8B%E7%BD%91%E5%8D%A1%E7%9A%84%E5%B7%A5%E4%BD%9C%E7%8A%B6%E6%80%81.png)

图13-11 查看网卡的工作状态

**第3步**：在bind服务程序的主配置文件中添加缓存转发参数。在大约第20行处添加一行参数“forwarders { 上级DNS服务器地址; };”，上级DNS服务器地址指的是获取数据配置文件的服务器。考虑到查询速度、稳定性、安全性等因素，[刘遄](https://www.linuxprobe.com/)老师在这里使用的是北京市公共DNS服务器的地址210.73.64.1。如果大家也使用该地址，请先测试是否可以ping通，以免导致DNS域名解析失败。

```
[root@linuxprobe ~]# vim /etc/named.conf  1 //  2 // named.conf  3 //  4 // Provided by Red Hat bind package to configure the ISC BIND named(8) DNS  5 // server as a caching only nameserver (as a localhost DNS resolver only).  6 //  7 // See /usr/share/doc/bind*/sample/ for example named configuration files.  8 //  9  10 options { 11         listen-on port 53 { any; }; 12         listen-on-v6 port 53 { ::1; }; 13         directory       "/var/named"; 14         dump-file       "/var/named/data/cache_dump.db"; 15         statistics-file "/var/named/data/named_stats.txt"; 16         memstatistics-file "/var/named/data/named_mem_stats.txt"; 17         secroots-file   "/var/named/data/named.secroots"; 18         recursing-file  "/var/named/data/named.recursing"; 19         allow-query     { any; }; 20         forwarders { 210.73.64.1; };………………省略部分输出信息………………[root@linuxprobe ~]# systemctl restart named
```

对了，如果您也还原了虚拟机系统到最初始的状态，记得把防火墙的放行规则一并完成：

```
[root@linuxprobe ~]# iptables -F[root@linuxprobe ~]# iptables-save[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=dnssuccess[root@linuxprobe ~]# firewall-cmd --reloadsuccess
```

**第4步**：重启DNS服务，验证成果。把客户端主机的DNS服务器地址参数修改为DNS缓存服务器的IP地址192.168.10.10，如图13-12所示。这样即可让客户端使用本地DNS缓存服务器提供的域名查询解析服务。

![第13章 使用Bind提供域名解析服务第13章 使用Bind提供域名解析服务](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E8%AE%BE%E7%BD%AE%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B8%BB%E6%9C%BA%E7%9A%84DNS%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9C%B0%E5%9D%80%E5%8F%82%E6%95%B0.png)

图13-12 设置客户端主机的DNS服务器地址参数

客户端主机的网络参数设置妥当后重启网络服务，即可使用nslookup命令来验证实验结果（如果解析失败，请读者留意是否是上级DNS服务器选择的问题）。其中，Server参数为域名解析记录提供的服务器地址，因此可见是由本地DNS缓存服务器提供的解析内容。

```
[root@linuxprobe ~]# nmcli connection up ens160 Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)[root@linuxprobe ~]# nslookup> www.linuxprobe.comServer:		192.168.10.10Address:	192.168.10.10#53Non-authoritative answer:www.linuxprobe.com	canonical name = www.linuxprobe.com.w.kunlunno.com.Name:	www.linuxprobe.com.w.kunlunno.comAddress: 139.215.131.226
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
[root@linuxprobe ~]# vim /etc/named.conf………………省略部分输出信息……………… 44  45 logging { 46         channel default_debug { 47                 file "data/named.run"; 48                 severity dynamic; 49         }; 50 }; 51  52 zone "." IN { 53         type hint; 54         file "named.ca"; 55 }; 56  57 include "/etc/named.rfc1912.zones"; 58 include "/etc/named.root.key"; 59 ………………省略部分输出信息………………
```

**第2步**：编辑区域配置文件。把区域配置文件中原有的数据清空，然后按照以下格式写入参数。首先使用acl参数分别定义两个变量名称（china与america），当下面需要匹配IP地址时只需写入变量名称即可，这样不仅容易阅读识别，而且也利于修改维护。这里的难点是理解view参数的作用。它的作用是通过判断用户的IP地址是中国的还是美国的，然后去分别加载不同的数据配置文件（linuxprobe.com.china或linuxprobe.com.america）。这样，当把相应的IP地址分别写入到数据配置文件后，即可实现DNS的分离解析功能。这样一来，当中国的用户访问linuxprobe.com域名时，便会按照linuxprobe.com.china数据配置文件内的IP地址找到对应的服务器。

```
[root@linuxprobe ~]# vim /etc/named.rfc1912.zonesacl "china" { 122.71.115.0/24; };acl "america" { 106.185.25.0/24; };view "china"{        match-clients { "china"; };        zone "linuxprobe.com" {        type master;        file "linuxprobe.com.china";        };};view "america" {        match-clients { "america"; };        zone "linuxprobe.com" {        type master;        file "linuxprobe.com.america";        };};    
```

**第3步**：建立数据配置文件。分别通过模板文件创建出两份不同名称的区域数据文件，其名称应与上面区域配置文件中相对应。

```
[root@linuxprobe ~]# cd /var/named[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.china[root@linuxprobe named]# cp -a named.localhost linuxprobe.com.america[root@linuxprobe named]# vim linuxprobe.com.china
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

# [第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/basic-learning-14.html)

**章节简述：**

本章讲解动态主机配置协议（DHCP，Dynamic Host Configuration Protocol），该协议用于自动管理局域网内主机的IP地址、子网掩码、网关地址及DNS地址等参数，可以有效地提升IP地址的利用率，提高配置效率，并降低管理与维护成本。

本章详细讲解了在[Linux系统](https://www.linuxprobe.com/)中配置部署dhcpd服务程序的方法，剖析了dhcpd服务程序配置文件内每个参数的作用，并通过自动分配IP地址、绑定IP地址与MAC地址等实验，让各位读者更直观地体会DHCP协议的强大之处。

本章目录结构

- [14.1 动态主机地址管理协议](https://www.linuxprobe.com/basic-learning-14.html#141)
- [14.2 部署dhcpd服务程序](https://www.linuxprobe.com/basic-learning-14.html#142_dhcpd)
- [14.3 自动管理IP地址](https://www.linuxprobe.com/basic-learning-14.html#143_IP)
- [14.4 分配固定IP地址](https://www.linuxprobe.com/basic-learning-14.html#144_IP)

##### **14.1 动态主机地址管理协议**

动态主机配置协议（DHCP）是一种基于UDP协议且仅限于在局域网内部使用的网络协议，主要用于大型的局域网环境或者存在较多移动办公设备的局域网环境中，用途是为局域网内部的设备或网络供应商自动分配IP地址等参数，提供网络配置的“全家桶”服务。

简单来说，DHCP协议就是让局域网中的主机自动获得网络参数的服务。在图14-1所示的拓扑图中存在多台主机，如果手动配置每台主机的网络参数会相当麻烦，日后维护起来也让人头大。而且当机房内的主机数量进一步增加时（比如有100台，甚至1000台），这个手动配置以及维护工作的工作量足以让运维人员崩溃。借助于DHCP协议，不仅可以为主机自动分配网络参数，还可以确保主机使用的IP地址是唯一的，更重要的是，还能为特定主机分配固定的IP地址。

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2015/07/DHCP%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

图14-1 DHCP协议的拓扑示意图

DHCP协议的应用十分广泛，无论是服务器机房还是家庭、机场、咖啡馆，都会见到它的身影。比如，本书的某位读者开了一家咖啡厅，在为顾客提供咖啡的同时，还为顾客免费提供无线上网服务。这样一来，顾客就可以一边惬意地喝着咖啡，一边连着无线网络刷朋友圈了。但是，作为咖啡厅老板的您，肯定不希望（也没有时间）为每一位造访的顾客手动设置IP地址、子网掩码、网关地址等信息。另外，考虑到咖啡馆使用的内网网段一般为192.168.10.0/24（C类私有地址），最多能容纳的主机数为200多台。而咖啡厅一天的客流量肯定不止200人。如果采用手动方式为他们分配IP地址，则当他们在离开咖啡厅时并不会自动释放这个IP地址，这就可能出现IP地址不够用的情况。这一方面会造成IP地址的浪费，另外一方面也增加的IP地址的管理成本。而使用DHCP协议，这一切都迎刃而解—老板只需安心服务好顾客，为其提供美味的咖啡；顾客通过运行DHCP协议的服务器自动获得上网所需的IP地址，等离开咖啡厅时IP地址将被DHCP服务器收回，以备其他顾客使用。

既然确定在今后的生产环境中肯定离不开DHCP了，那么也就有必要好好地熟悉一下DHCP涉及的常见术语了。

> **作用域**：一个完整的IP地址段，DHCP协议根据作用域来管理网络的分布、分配IP地址及其他配置参数。
>
> **超级作用域**：用于管理处于同一个物理网络中的多个逻辑子网段，包含了可以统一管理的作用域列表。
>
> **排除范围**：把作用域中的某些IP地址排除，确保这些IP地址不会分配给DHCP客户端。
>
> **地址池**：在定义了DHCP的作用域并应用了排除范围后，剩余的用来动态分配给客户端的IP地址范围。
>
> **租约**：DHCP客户端能够使用动态分配的IP地址的时间。
>
> **预约**：保证网络中的特定设备总是获取到相同的IP地址。

##### **14.2 部署dhcpd服务程序**

dhcpd是[Linux](https://www.linuxprobe.com/)系统中用于提供DHCP协议的服务程序。尽管DHCP协议的功能十分强大，但是dhcpd服务程序的配置步骤却十分简单，这也在很大程度上降低了在Linux中实现动态主机管理服务的门槛。

在确认软件仓库配置妥当之后，安装dhcpd服务程序，其软件包名称为dhcp-server：

```
[root@linuxprobe ~]# dnf install -y dhcp-server
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
AppStream                                               3.1 MB/s | 3.2 kB     00:00    
BaseOS                                                  2.7 MB/s | 2.7 kB     00:00    
Dependencies resolved.
========================================================================================
 Package              Arch            Version                     Repository       Size
========================================================================================
Installing:
 dhcp-server          x86_64          12:4.3.6-30.el8             BaseOS          529 k

Transaction Summary
========================================================================================
Install  1 Package
………………省略部分输出信息………………
Installed:
  dhcp-server-12:4.3.6-30.el8.x86_64                                                    

Complete!
```

查看dhcpd服务程序的配置文件内容。

```
[root@linuxprobe ~]# cat /etc/dhcp/dhcpd.conf
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp-server/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
```

是的，您没有看错！dhcp的服务程序的配置文件中只有3行注释语句，这意味着我们需要自行编写这个文件。如果读者不知道怎么编写，可以看一下配置文件中第2行的参考示例文件，其组成架构如图14-2所示。

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2015/07/dhcp%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png)

图14-2 dhcpd服务程序配置文件的架构

一个标准的配置文件应该包括全局配置参数、子网网段声明、地址配置选项以及地址配置参数。其中，全局配置参数用于定义dhcpd服务程序的整体运行参数；子网网段声明用于配置整个子网段的地址属性。

考虑到dhcpd服务程序配置文件的可用参数比较多，[刘遄](https://www.linuxprobe.com/)老师挑选了最常用的参数（见表14-1），并逐一进行了简单介绍，以便为接下来的实验打好基础。

表14-1            dhcpd服务程序配置文件中使用的常见参数以及作用

| 参数                               | 作用                                                         |
| ---------------------------------- | ------------------------------------------------------------ |
| ddns-update-style 类型             | 定义DNS服务动态更新的类型，类型包括： none（不支持动态更新）、interim（互动更新模式）与ad-hoc（特殊更新模式） |
| allow/ignore client-updates        | 允许/忽略客户端更新DNS记录                                   |
| default-lease-time 21600           | 默认超时时间                                                 |
| max-lease-time 43200               | 最大超时时间                                                 |
| option domain-name-servers 8.8.8.8 | 定义DNS服务器地址                                            |
| option domain-name "domain.org"    | 定义DNS域名                                                  |
| range                              | 定义用于分配的IP地址池                                       |
| option subnet-mask                 | 定义客户端的子网掩码                                         |
| option routers                     | 定义客户端的网关地址                                         |
| broadcast-address 广播地址         | 定义客户端的广播地址                                         |
| ntp-server IP地址                  | 定义客户端的网络时间服务器（NTP）                            |
| nis-servers IP地址                 | 定义客户端的NIS域服务器的地址                                |
| hardware 硬件类型 MAC地址          | 指定网卡接口的类型与MAC地址                                  |
| server-name 主机名                 | 向DHCP客户端通知DHCP服务器的主机名                           |
| fixed-address IP地址               | 将某个固定的IP地址分配给指定主机                             |
| time-offset 偏移差                 | 指定客户端与格林尼治时间的偏移差                             |



##### **14.3 自动管理IP地址**

DHCP协议的设计初衷是为了更高效地集中管理局域网内的IP地址资源。DHCP服务器会自动把IP地址、子网掩码、网关、DNS地址等网络信息分配给有需要的客户端，而且当客户端的租约时间到期后还可以自动回收所分配的IP地址，以便交给新加入的客户端。

为了让实验更有挑战性，来模拟一个真实生产环境的需求：

“机房运营部门：明天会有100名学员自带笔记本电脑来我司培训学习，请保证他们能够使用机房的本地DHCP服务器自动获取IP地址并正常上网”。

机房所用的网络地址及参数信息如表14-2所示。

表14-2                    机房所用的网络地址以及参数信息

| 参数名称      | 值                           |
| ------------- | ---------------------------- |
| 默认租约时间  | 21600秒                      |
| 最大租约时间  | 43200秒                      |
| IP地址范围    | 192.168.10.50~192.168.10.150 |
| 子网掩码      | 255.255.255.0                |
| 网关地址      | 192.168.10.1                 |
| DNS服务器地址 | 192.168.10.1                 |
| 搜索域        | linuxprobe.com               |



在了解了真实需求以及机房网络中的配置参数之后，我们按照表14-3来配置DHCP服务器以及客户端。

表14-3                  DHCP服务器以及客户端的配置信息

| 主机类型   | 操作系统   | IP地址           |
| ---------- | ---------- | ---------------- |
| DHCP服务器 | RHEL 8     | 192.168.10.1     |
| DHCP客户端 | Windows 10 | DHCP自动获取地址 |



前文讲到，作用域一般是个完整的IP地址段，而地址池中的IP地址才是真正供客户端使用的，因此地址池应该小于或等于作用域的IP地址范围。另外，由于VMware Workstation虚拟机软件自带DHCP服务，为了避免与自己配置的dhcpd服务程序产生冲突，应该先按照图14-3和图14-4所示将虚拟机软件自带的DHCP功能关闭。

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E8%99%9A%E6%8B%9F%E7%BD%91%E7%BB%9C%E7%BC%96%E8%BE%91%E5%99%A8-1024x180.png)

图14-3 单击虚拟机软件的“虚拟网络编辑器”菜单

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E5%85%B3%E9%97%ADDHCP.png)

图14-4 关闭虚拟机自带的DHCP功能

可随意开启几台客户端，准备进行验证。但是一定要注意，DHCP客户端与服务器需要处于同一种网络模式—仅主机模式（Hostonly），否则就会产生物理隔离，从而无法获取IP地址。建议开启1～3台客户端虚拟机验证一下效果就好，以免物理主机的CPU和内存的负载太高。

在确认DHCP服务器的IP地址等网络信息配置妥当后就可以配置dhcpd服务程序了。请注意，在配置dhcpd服务程序时，配置文件中的每行参数后面都需要以分号（;）结尾，这是规定。另外，dhcpd服务程序配置文件内的参数都十分重要，因此[刘遄](https://www.linuxprobe.com/)老师在表14-4中罗列出了每一行参数，并对其用途进行了简单介绍。

```
[root@linuxprobe ~]# vim /etc/dhcp/dhcpd.conf
ddns-update-style none;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
        range 192.168.10.50 192.168.10.150;
        option subnet-mask 255.255.255.0;
        option routers 192.168.10.1;
        option domain-name "linuxprobe.com";
        option domain-name-servers 192.168.10.1;
        default-lease-time 21600;
        max-lease-time 43200;
}
```

表14-4              dhcpd服务程序配置文件中使用的参数以及作用

| 参数                                        | 作用                                         |
| ------------------------------------------- | -------------------------------------------- |
| ddns-update-style none;                     | 设置DNS服务不自动进行动态更新                |
| ignore client-updates;                      | 忽略客户端更新DNS记录                        |
| subnet 192.168.10.0 netmask 255.255.255.0 { | 作用域为192.168.10.0/24网段                  |
| range 192.168.10.50 192.168.10.150;         | IP地址池为192.168.10.50-150（约100个IP地址） |
| option subnet-mask 255.255.255.0;           | 定义客户端默认的子网掩码                     |
| option routers 192.168.10.1;                | 定义客户端的网关地址                         |
| option domain-name "linuxprobe.com";        | 定义默认的搜索域                             |
| option domain-name-servers 192.168.10.1;    | 定义客户端的DNS地址                          |
| default-lease-time 21600;                   | 定义默认租约时间（单位：秒）                 |
| max-lease-time 43200;                       | 定义最大预约时间（单位：秒）                 |
| }                                           | 结束符                                       |



在[红帽](https://www.linuxprobe.com/)认证考试以及生产环境中，都需要把配置过的dhcpd服务加入到开机启动项中，以确保当服务器下次开机后dhcpd服务依然能自动启动，并顺利地为客户端分配IP地址等信息。真心建议大家能养成“配置好服务程序，顺手加入开机启动项”的好习惯：

```
[root@linuxprobe ~]# systemctl start dhcpd
[root@linuxprobe ~]# systemctl enable dhcpd
Created symlink /etc/systemd/system/multi-user.target.wants/dhcpd.service → /usr/lib/systemd/system/dhcpd.service.
```

把dhcpd服务程序配置妥当之后就可以开启客户端来检验IP分配效果了。日常工作中Windows 10是主流桌面操作系统，所以只要确保两个主机都处于同一个网络模式内，然后像如图14-5所示的方法设置Windows系统的网卡为DHCP模式，再稍等片刻即可自动获取到网卡信息了，如图14-6所示，特别的方便。

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E8%AE%BE%E7%BD%AE%E7%BD%91%E5%8D%A1%E6%A8%A1%E5%BC%8F.png)

图14-5 设置网卡模式

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E8%87%AA%E5%8A%A8%E8%8E%B7%E5%8F%96%E5%88%B0IP%E5%9C%B0%E5%9D%80.png)

图14-6 自动获取到IP地址

如果是在生产环境配置的dhcpd服务，有可能会因为DHCP协议没有被防火墙放行而实验失败，则执行下面[命令](https://www.linuxcool.com/)即可：

```
[root@linuxprobe ~]# firewall-cmd --zone=public --permanent --add-service=dhcp
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

在正常情况下，DHCP协议的运作会经历四个过程——请求、提供、选择和确认。当客户端顺利获得一个IP地址及相关网卡信息后，就会发送一个ARP请求给服务器，不会重复的领取IP地址，并且在dhcpd服务程序收到这条信息后，也会再把这个IP地址分配给其他主机，从根源上避免了IP地址冲突的情况。

##### **14.4 分配固定IP地址**

在DHCP协议中有个术语是“预约”，它用来确保局域网中特定的设备总是获取到固定的IP地址。换句话说，就是dhcpd服务程序会把某个IP地址私藏下来，只将其用于相匹配的特定设备。有点像高档餐厅的预定服务，虽然客人还没有到场，但是桌子上会放个小牌子写着已预定。

要想把某个IP地址与某台主机进行绑定，就需要用到这台主机的MAC地址。即网卡上面的一串独立的标识符，具备唯一性，因此不会存在冲突的情况。MAC地址在Linux系统中查看到的样子如图14-7所示，在Windows系统中查看到的样子如图14-8所示。
![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/Linux.png)

图14-7 在Linux系统中查看网卡MAC地址

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/Windows.png)

图14-8 在Windows系统中查看网卡MAC地址

在Linux系统或Windows系统中，都可以通过查看网卡的状态来获知主机的MAC地址。在dhcpd服务程序的配置文件中，按照如下格式将IP地址与MAC地址进行绑定。

| host 主机名称 { |               |                 |                  |      |
| --------------- | ------------- | --------------- | ---------------- | ---- |
|                 | hardware      | ethernet        | 该主机的MAC地址; |      |
|                 | fixed-address | 欲指定的IP地址; |                  |      |
| }               |               |                 |                  |      |



如果不方便查看主机的MAC地址，该怎么办呢？比如，要给老板使用的主机绑定IP地址，总不能随便就去查看老板的主机信息吧。针对这种情况，刘遄老师告诉大家一个很好的办法。我们首先启动dhcpd服务程序，为老板的主机分配一个IP地址，这样就会在DHCP服务器本地的日志文件中保存这次的IP地址分配记录。然后查看日志文件，就可以获悉主机的MAC地址了（即下面加粗的内容）。

```
[root@linuxprobe ~]# tail -f /var/log/messages
………………省略部分输出信息………………
Mar 22 00:28:54 linuxprobe cupsd[1206]: REQUEST linuxprobe.com- - "POST / HTTP/1.1" 200 183 Renew-Subscription client-error-not-found
Mar 22 00:29:35 linuxprobe dhcpd[30959]: DHCPREQUEST for 192.168.10.50 from 00:0c:29:dd:f2:22 (DESKTOP-3OGV50E) via ens160
Mar 22 00:29:35 linuxprobe dhcpd[30959]: DHCPACK on 192.168.10.50 to 00:0c:29:dd:f2:22 (DESKTOP-3OGV50E) via ens160
```

之前我在线下讲课时，讲完DHCP服务后总是看到有些学员在挠头。起初我很不理解，毕竟dhcpd服务程序是Linux系统中一个很简单的实验，总共就那么十几行的配置参数还能写错？后来发现了原因—有些学员是以Windows系统为对象做的IP与MAC地址的绑定实验。而在Windows系统中看到的MAC地址，其格式类似于00-0c-29-dd-f2-22，间隔符为减号（-）。但是在Linux系统中，MAC地址的间隔符则变成了冒号（:）。

```
[root@linuxprobe ~]# vim /etc/dhcp/dhcpd.conf 
ddns-update-style none;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
        range 192.168.10.50 192.168.10.150;
        option subnet-mask 255.255.255.0;
        option routers 192.168.10.1;
        option domain-name "linuxprobe.com";
        option domain-name-servers 192.168.10.1;
        default-lease-time 21600;
        max-lease-time 43200;
        host linuxprobe {
                hardware ethernet 00:0c:29:dd:f2:22;
                fixed-address 192.168.10.88;
                }
}
```

确认参数填写正确后就可以保存退出配置文件，然后就可以重启dhcpd服务程序了。

```
[root@linuxprobe ~]# systemctl restart dhcpd
```

需要说明的是，如果您刚刚为这台主机分配了IP地址，则它的IP地址租约时间还没有到期，因此不会立即换成新绑定的IP地址。要想立即查看绑定效果，则需要重启一下客户端的网络服务，如图14-9所示，然后就能看到效果了~如果14-10所示。

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E9%87%8D%E5%90%AF%E7%BD%91%E5%8D%A1%E8%AE%BE%E5%A4%87-1.png)

 

图14-9 重启网卡设备

![第14章 使用DHCP动态管理主机地址第14章 使用DHCP动态管理主机地址](https://www.linuxprobe.com/wp-content/uploads/2021/02/%E6%96%B0%E7%9A%84IP%E5%9C%B0%E5%9D%80.png)

图14-10 查看绑定后的网卡信息

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．简述DHCP协议的主要用途。

**答：**为局域网内部的设备或网络供应商自动分配IP地址等参数。

2．DHCP协议能够为客户端分配什么网卡资源？

**答：**可为客户端分配IP地址、子网掩码、网关地址以及DNS地址等信息。

3．真正供用户使用的IP地址范围是作用域还是地址池？

**答：**地址池，因为作用域内还会包含要排除掉的IP地址。

4．简述DHCP协议中“租约”的作用。

**答：**租约分为默认租约时间和最大租约时间，用于在租约时间到期后自动回收主机的IP地址，以免造成IP地址的浪费。

5．把IP地址与主机的什么信息绑定，就可以保证该主机一直获取到固定的IP地址？

**答：**主机网卡的MAC地址。

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
[root@linuxprobe ~]# dnf install postfixUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:10:38 ago on Mon 29 Mar 2021 06:40:32 AM CST.Dependencies resolved.================================================================================ Package             Arch          Version             Repository    Size================================================================================Installing: postfix             x86_64        2:3.3.1-8.el8       BaseOS        1.5 MTransaction Summary================================================================================Install  1 Package………………省略部分输出信息………………Installed:  postfix-2:3.3.1-8.el8.x86_64                                                                                     Complete!
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
[root@linuxprobe ~]# vim /etc/postfix/main.cf 86  87 # INTERNET HOST AND DOMAIN NAMES 88 #  89 # The myhostname parameter specifies the internet hostname of this 90 # mail system. The default is to use the fully-qualified domain name 91 # from gethostname(). $myhostname is used as a default value for many 92 # other configuration parameters. 93 # 94 #myhostname = host.domain.tld 95 myhostname = mail.linuxprobe.com 96 
```

然后在第102行定义一个名为mydomain的变量，用来保存邮件域的名称。大家也要记住这个变量名称，下面将调用它：

```
 96  97 # The mydomain parameter specifies the local internet domain name. 98 # The default is to use $myhostname minus the first component. 99 # $mydomain is used as a default value for many other configuration100 # parameters.101 #102 mydomain = linuxprobe.com103 
```

在第118行调用前面的mydomain变量，用来定义发出邮件的域。调用变量的好处是避免重复写入信息，以及便于日后统一修改：

```
105 # 106 # The myorigin parameter specifies the domain that locally-posted107 # mail appears to come from. The default is to append $myhostname,108 # which is fine for small sites.  If you run a domain with multiple109 # machines, you should (1) change this to $mydomain and (2) set up110 # a domain-wide alias database that aliases each user to111 # user@that.users.mailhost.112 #113 # For the sake of consistency between sender and recipient addresses,114 # myorigin also specifies the default domain name that is appended115 # to recipient addresses that have no @domain part.116 #117 #myorigin = $myhostname118 myorigin = $mydomain119 
```

第四处修改是在第135行定义网卡监听地址。可以指定要使用服务器的哪些IP地址对外提供电子邮件服务；也可以干脆写成all，代表所有IP地址都能提供电子邮件服务：

```
121 122 # The inet_interfaces parameter specifies the network interface123 # addresses that this mail system receives mail on.  By default,124 # the software claims all active interfaces on the machine. The125 # parameter also controls delivery of mail to user@[ip.address].126 #127 # See also the proxy_interfaces parameter, for network addresses that128 # are forwarded to us via a proxy or network address translator.129 #130 # Note: you need to stop/start Postfix when this parameter changes.131 #132 #inet_interfaces = all133 #inet_interfaces = $myhostname134 #inet_interfaces = $myhostname, localhost135 inet_interfaces = all136 
```

最后一处修改是在第183行定义可接收邮件的主机名或域名列表。这里可以直接调用前面定义好的myhostname和mydomain变量（如果不想调用变量，也可以直接调用变量中的值）：

```
151 152 # The mydestination parameter specifies the list of domains that this153 # machine considers itself the final destination for.154 #155 # These domains are routed to the delivery agent specified with the156 # local_transport parameter setting. By default, that is the UNIX157 # compatible delivery agent that lookups all recipients in /etc/passwd158 # and /etc/aliases or their equivalent.159 #160 # The default is $myhostname + localhost.$mydomain + localhost.  On161 # a mail domain gateway, you should also include $mydomain.162 #163 # Do not specify the names of virtual domains - those domains are164 # specified elsewhere (see VIRTUAL_README).165 #166 # Do not specify the names of domains that this machine is backup MX167 # host for. Specify those names via the relay_domains settings for168 # the SMTP server, or use permit_mx_backup if you are lazy (see169 # STANDARD_CONFIGURATION_README).170 #171 # The local machine is always the final destination for mail addressed172 # to user@[the.net.work.address] of an interface that the mail system173 # receives mail on (see the inet_interfaces parameter).174 #175 # Specify a list of host or domain names, /file/name or type:table176 # patterns, separated by commas and/or whitespace. A /file/name177 # pattern is replaced by its contents; a type:table is matched when178 # a name matches a lookup key (the right-hand side is ignored).179 # Continue long lines by starting the next line with whitespace.180 #181 # See also below, section "REJECTING MAIL FOR UNKNOWN LOCAL USERS".182 #183 mydestination = $myhostname, $mydomain184 #mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain185 #mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain,186 #       mail.$mydomain, www.$mydomain, ftp.$mydomain187 
```

**第3步**：创建电子邮件系统的登录账户。Postfix与vsftpd服务程序一样，都可以调用本地系统的账户和密码，因此在本地系统创建常规账户即可。最后重启配置妥当的postfix服务程序，并将其添加到开机启动项中。大功告成！

```
[root@linuxprobe ~]# useradd liuchuan[root@linuxprobe ~]# echo "linuxprobe" | passwd --stdin liuchuanChanging password for user liuchuan.passwd: all authentication tokens updated successfully.[root@linuxprobe ~]# systemctl restart postfix[root@linuxprobe ~]# systemctl enable  postfixCreated symlink /etc/systemd/system/multi-user.target.wants/postfix.service → /usr/lib/systemd/system/postfix.service.
```

###### **15.2.2 配置Dovecot服务程序**

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/dovecot.png)

Dovecot是一款能够为Linux系统提供IMAP和POP3电子邮件服务的开源服务程序，安全性极高，配置简单，执行速度快，而且占用的服务器硬件资源也较少，因此是一款值得推荐的收件服务程序。

**第1步**：安装Dovecot服务程序软件包。

```
[root@linuxprobe ~]# dnf install -y dovecotUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:49:52 ago on Mon 29 Mar 2021 06:40:32 AM CST.Dependencies resolved.============================================================================================= Package                   Arch      Version                              Repository    Size=============================================================================================Installing: dovecot                   x86_64    1:2.2.36-5.el8                       AppStream     4.6 MInstalling dependencies: clucene-core              x86_64    2.3.3.4-31.20130812.e8e3d20git.el8   AppStream     590 kTransaction Summary===============================================================================================Install  2 Packages………………省略部分输出信息………………Installed:  dovecot-1:2.2.36-5.el8.x86_64                                                                 clucene-core-2.3.3.4-31.20130812.e8e3d20git.el8.x86_64                                                              Complete!
```

**第2步**：配置部署Dovecot服务程序。在Dovecot服务程序的主配置文件中进行如下修改。首先是第24行，把Dovecot服务程序支持的电子邮件协议修改为imap、pop3和lmtp。然后在这一行下面添加一行参数，允许用户使用明文进行密码验证。之所以这样操作，是因为Dovecot服务程序为了保证电子邮件系统的安全而默认强制用户使用加密方式进行登录，而由于当前还没有加密系统，因此需要添加该参数来允许用户的明文登录。

```
[root@linuxprobe ~]# vim /etc/dovecot/dovecot.conf………………省略部分输出信息……………… 22  23 # Protocols we want to be serving. 24 protocols = imap pop3 lmtp 25 disable_plaintext_auth = no 26 ………………省略部分输出信息………………
```

在主配置文件中的第49行，设置允许登录的网段地址，也就是说我们可以在这里限制只有来自于某个网段的用户才能使用电子邮件系统。如果想允许所有人都能使用，则不用修改本参数：

```
 44  45 # Space separated list of trusted network ranges. Connections from these 46 # IPs are allowed to override their IP addresses and ports (for logging and 47 # for authentication checks). disable_plaintext_auth is also ignored for 48 # these networks. Typically you'd specify your IMAP proxy servers here. 49 login_trusted_networks = 192.168.10.0/24 50 
```

**第3步**：配置邮件格式与存储路径。在Dovecot服务程序单独的子配置文件中，定义一个路径，用于指定要将收到的邮件存放到服务器本地的哪个位置。这个路径默认已经定义好了，只需要将该配置文件中第25行前面的井号（#）删除即可。

```
[root@linuxprobe ~]# vim /etc/dovecot/conf.d/10-mail.conf  1 ##  2 ## Mailbox locations and namespaces  3 ##  4   5 # Location for users' mailboxes. The default is empty, which means that Dovecot  6 # tries to find the mailboxes automatically. This won't work if the user  7 # doesn't yet have any mail, so you should explicitly tell Dovecot the full  8 # location.  9 # 10 # If you're using mbox, giving a path to the INBOX file (eg. /var/mail/%u) 11 # isn't enough. You'll also need to tell Dovecot where the other mailboxes are 12 # kept. This is called the "root mail directory", and it must be the first 13 # path given in the mail_location setting. 14 # 15 # There are a few special variables you can use, eg.: 16 # 17 #   %u - username 18 #   %n - user part in user@domain, same as %u if there's no domain 19 #   %d - domain part in user@domain, empty if there's no domain 20 #   %h - home directory 21 # 22 # See doc/wiki/Variables.txt for full list. Some examples: 23 # 24 #   mail_location = maildir:~/Maildir 25     mail_location = mbox:~/mail:INBOX=/var/mail/%u 26 #   mail_location = mbox:/var/mail/%d/%1n/%n:INDEX=/var/indexes/%d/%1n/%n 27 #………………省略部分输出信息………………
```

然后切换到配置Postfix服务程序时创建的boss账户，并在家目录中建立用于保存邮件的目录。记得要重启Dovecot服务并将其添加到开机启动项中。至此，对Dovecot服务程序的配置部署步骤全部结束。

```
[root@linuxprobe ~]# su - liuchuan[liuchuan@linuxprobe ~]$ mkdir -p mail/.imap/INBOX[liuchuan@linuxprobe ~]$ exitlogout[root@linuxprobe ~]# systemctl restart dovecot[root@linuxprobe ~]# systemctl enable  dovecotCreated symlink /etc/systemd/system/multi-user.target.wants/dovecot.service → /usr/lib/systemd/system/dovecot.service.
```

老读者肯定觉得少了点什么吧~对喽，还要记得把上面所提到的邮件协议在防火墙中的策略予以放行，这样客户端就能正常访问了：

```
[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=imapsuccess[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=pop3success[root@linuxprobe ~]# firewall-cmd --permanent --zone=public --add-service=smtpsuccess[root@linuxprobe ~]# firewall-cmd --reloadsuccess
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
[root@linuxprobe ~]# dnf install mailx[root@linuxprobe ~]# mailxHeirloom Mail version 12.5 7/5/10.  Type ? for help."/var/spool/mail/root": 1 message 1 new>N  1 liuchuan              Tue Mar 30 01:35  97/3257  "Hello~"& 1Message  1:From liuchuan@linuxprobe.com  Tue Mar 30 01:35:29 2021Return-Path: <liuchuan@linuxprobe.com>X-Original-To: root@linuxprobe.comDelivered-To: root@linuxprobe.comFrom: "liuchuan" <liuchuan@linuxprobe.com>To: <root@linuxprobe.com>Subject: Hello~Date: Mon, 29 Mar 2021 19:49:30 +0800Content-Type: multipart/alternative;	boundary="----=_NextPart_000_0001_01D724D4.A28BB310"X-Mailer: Microsoft Outlook 14.0Thread-Index: AdckkVaUrscA9j2EQ3evqG++j6aSSA==Content-Language: zh-cnStatus: RContent-Type: text/plain;	charset="gb2312"当您收到这封邮件时，证明我的邮局系统实验已经成功！& quitHeld 1 message in /var/spool/mail/root[root@linuxprobe ~]#
```

##### **15.3 设置用户别名邮箱**

用户别名功能是一项简单实用的邮件账户伪装技术，可以用来设置多个虚拟信箱的账户以接受发送的邮件，从而保证自身的邮件地址不被泄露，还可以用来接收自己的多个信箱中的邮件。刚才我们已经顺利地向root账户送了邮件，下面再向bin账户发送一封邮件，如图15-14所示。

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%90%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84bin%E8%B4%A6%E6%88%B7%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6.png)

图15-14 向服务器上的bin账户发送邮件

在邮件发送后登录到服务器，然后尝试以bin账户的身份登录。由于bin账户在Linux系统中是系统账户，默认的[Shell](https://www.linuxcool.com/)终端是/sbin/nologin，因此在以bin账户登录时，系统会提示当前账户不可用。但是，在电子邮件服务器上使用mail命令后，却看到这封原本要发送给bin账户的邮件已经被存放到了root账户的信箱中。

```
[root@linuxprobe ~]# su - binThis account is currently not available.[root@linuxprobe ~]# mailxHeirloom Mail version 12.5 7/5/10.  Type ? for help."/var/spool/mail/root": 2 messages 1 new    1 liuchuan              Tue Mar 30 01:35  98/3268  "Hello~">N  2 liuchuan              Tue Mar 30 03:53  97/3251  "你好，用户bin。"& 2Message  2:From liuchuan@linuxprobe.com  Tue Mar 30 03:53:37 2021Return-Path: <liuchuan@linuxprobe.com>X-Original-To: bin@linuxprobe.comDelivered-To: bin@linuxprobe.comFrom: "liuchuan" <liuchuan@linuxprobe.com>To: <bin@linuxprobe.com>Subject: 你好，用户bin。Date: Mon, 29 Mar 2021 22:07:39 +0800Content-Type: multipart/alternative;	boundary="----=_NextPart_000_000E_01D724E7.EEF35A10"X-Mailer: Microsoft Outlook 14.0Thread-Index: AdckpJ6n2QIfRYAZTB20gA9VTep2dg==Content-Language: zh-cnStatus: RContent-Type: text/plain;	charset="gb2312"这是一封发给用户bin的邮件。& quitHeld 2 messages in /var/spool/mail/root[root@linuxprobe ~]#
```

太奇怪了！明明发送给bin账户的邮件怎么会被root账户收到了呢？其实，这就是使用用户别名技术来实现的。在aliases邮件别名服务的配置文件中可以看到，里面定义了大量的用户别名，这些用户别名大多数是Linux系统本地的系统账户，而在冒号（:）间隔符后面的root账户则是用来接收这些账户邮件的人。用户别名可以是Linux系统内的本地用户，也可以是完全虚构的用户名字。

### **Tips**

下述命令会显示大量的内容，考虑到篇幅限制，这里已经做了部分删减，其实际的输出名单将是这里的两倍多。

```
[root@linuxprobe ~]# cat /etc/aliases##  Aliases in this file will NOT be expanded in the header from#  Mail, but WILL be visible over networks or from /bin/mail.##	>>>>>>>>>>	The program "newaliases" must be run after#	>> NOTE >>	this file is updated for any changes to#	>>>>>>>>>>	show through to sendmail.## Basic system aliases -- these MUST be present.mailer-daemon:	postmasterpostmaster:	root# General redirections for pseudo accounts.bin:		rootdaemon:		rootadm:		rootlp:		rootsync:		rootshutdown:	roothalt:		rootmail:		rootnews:		rootuucp:		rootoperator:	root………………省略部分输出信息………………
```

现在大家能猜出是怎么一回事了吧。原来aliases邮件别名服务的配置文件是专门用来定义用户别名与邮件接收人的映射。除了使用本地系统中系统账户的名称外，我们还可以自行定义一些别名来接收邮件。例如，创建一个名为dream的账户，而真正接收该账户邮件的应该是root账户。

```
[root@linuxprobe ~]# cat /etc/aliases##  Aliases in this file will NOT be expanded in the header from#  Mail, but WILL be visible over networks or from /bin/mail.##       >>>>>>>>>>      The program "newaliases" must be run after#       >> NOTE >>      this file is updated for any changes to#       >>>>>>>>>>      show through to sendmail.## Basic system aliases -- these MUST be present.mailer-daemon:  postmasterpostmaster:     root# General redirections for pseudo accounts.dream:          rootbin:            rootdaemon:         rootadm:            rootlp:             rootsync:           rootshutdown:       roothalt:           rootmail:           rootnews:           rootuucp:           rootoperator:       root………………省略部分输出信息………………
```

保存并退出aliases邮件别名服务的配置文件后，需要再执行一下newaliases命令，其目的是让新的用户别名配置文件立即生效。然后再次尝试发送邮件，如图15-15所示：

```
[root@linuxprobe ~]# newaliases
```

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/%E5%90%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%8A%E7%9A%84dream%E7%94%A8%E6%88%B7%E5%8F%91%E9%80%81%E9%82%AE%E4%BB%B6.png)

图15-15 向服务器上的dream用户发送邮件

这时，使用root账户在服务器上执行mail命令后，就能看到这封原本要发送给dream账户的邮件了。最后，[刘遄](https://www.linuxprobe.com/)老师再啰嗦一句，用户别名技术不仅应用广泛，而且配置也很简单。所以更要提醒大家的是，今后千万不要看到有些网站上提供了很多客服信箱就轻易相信别人，没准发往这些客服信箱的邮件会被同一个人收到。

```
[root@linuxprobe ~]# mailxHeirloom Mail version 12.5 7/5/10.  Type ? for help."/var/spool/mail/root": 3 messages 1 new    1 liuchuan              Tue Mar 30 01:35  98/3268  "Hello~"    2 liuchuan              Tue Mar 30 03:53  98/3262  "你好，用户bin。">N  3 liuchuan              Tue Mar 30 04:12  98/3317  "这是一封发送给dream用户的邮件"& 3Message  3:From liuchuan@linuxprobe.com  Tue Mar 30 04:12:19 2021Return-Path: <liuchuan@linuxprobe.com>X-Original-To: dream@linuxprobe.comDelivered-To: dream@linuxprobe.comFrom: "liuchuan" <liuchuan@linuxprobe.com>To: <dream@linuxprobe.com>Subject: 这是一封发送给dream用户的邮件Date: Mon, 29 Mar 2021 22:26:21 +0800Content-Type: multipart/alternative;	boundary="----=_NextPart_000_0009_01D724EA.8B9A4750"X-Mailer: Microsoft Outlook 14.0Thread-Index: Adckpw3r2QT7QwGITceHTJdfioQeQQ==Content-Language: zh-cnStatus: RContent-Type: text/plain;	charset="gb2312"顺利的话会被root用户接收到。& quitHeld 3 messages in /var/spool/mail/root
```

##### **15.4 Linux邮件客户端**

![第15章 使用Postfix与Dovecot部署邮件系统第15章 使用Postfix与Dovecot部署邮件系统](https://www.linuxprobe.com/wp-content/uploads/2021/03/Thunderbird-150x150.png)

对于我们大多数人而言，如今更多的是使用浏览器或智能手机的方式来收发电子邮件。但是，为了更快的加载邮件，或是为了更为丰富的编辑功能，有些时候还是要依赖专门的邮件客户端才能完成。Linux系统下的可选邮件客户端有数十种，例如Thunderbird、Evolution、Gear、Elementary Mail、KMail、Mailspring、Sylpheed、Claws Mail等等。

过去经常会有同学抱怨说，要是能在Linux系统下办公就太好了，完全可以把生产环境迁移到开源产品上，那么就趁着这次机会，为读者介绍一款刘遄老师正在使用的邮件客户端吧~

Thunderbird是一款由FireFox火狐浏览器的母公司Mozilla基金会发布的电子邮件客户端，兼具FireFox浏览器的各种优势，实现跨平台支持，拥有各种插件和丰富功能，简单的操作让用户更容易轻松地上手。

RHEL 8系统的光盘镜像中已经包含了Thunderbird客户端的安装包，配置好软件仓库后即可一键安装：

```
[root@linuxprobe ~]# dnf install -y thunderbirdUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.AppStream                                           3.1 MB/s | 3.2 kB     00:00    BaseOS                                              2.1 MB/s | 2.7 kB     00:00    Dependencies resolved.==================================================================================== Package             Arch           Version                 Repository         Size====================================================================================Installing: thunderbird         x86_64         60.5.0-1.el8            AppStream          79 MTransaction Summary====================================================================================Install  1 Package………………省略部分输出信息………………Installed:  thunderbird-60.5.0-1.el8.x86_64                                                   Complete!
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
[root@linuxprobe ~]# vim /etc/ansible/hosts[dev]192.168.10.20[test]192.168.10.21[prod]192.168.10.22192.168.10.23[balancers]192.168.10.24[all:vars]ansible_user=rootansible_password=redhat
```

还剩最后一步，将Ansible主配置文件中的第71行设置成默认不需要SSH协议的指纹验证，以及第107行设置成默认执行Playbook时所使用的管理员名称为root：

```
[root@linuxprobe ~]# vim /etc/ansible/ansible.cfg6970 # uncomment this to disable SSH key host checking71 host_key_checking = False72………………省略部分输出信息………………104105 # default user to use for playbooks if user is not specified106 # (/usr/bin/ansible will use current user as default)107 remote_user = root108
```

不需要重启服务，以上操作完全搞定后就可以开始后面的实验了。由于刚刚是将Ansible服务器设置成了桥接及DHCP模式，请同学们自行将网络适配器修改回“仅主机模式”及192.168.10.10/24的IP地址吧，如图16-3所示。完成后重启网卡再自行在主机之间互相ping一下哦，保证主机之间网络能够互通是后续实验的基石。

```
[root@linuxprobe ~]# ifconfigens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500        inet 192.168.10.10  netmask 255.255.255.0  broadcast 192.168.10.255        inet6 fe80::d0bb:17c8:880d:e719  prefixlen 64  scopeid 0x20        ether 00:0c:29:7d:27:bf  txqueuelen 1000  (Ethernet)        RX packets 32  bytes 5134 (5.0 KiB)        RX errors 0  dropped 0  overruns 0  frame 0        TX packets 43  bytes 4845 (4.7 KiB)        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0………………省略部分输出信息………………
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
[root@linuxprobe ~]# ansible-doc -l a10_server                                           Manage A10 Networks AX/SoftAX/Thunder/v...a10_server_axapi3                                    Manage A10 Networks AX/SoftAX/Thunder/v...           a10_service_group                                    Manage A10 Networks AX/SoftAX/Thunder/v...a10_virtual_server                                   Manage A10 Networks AX/SoftAX/Thunder/v...aci_aaa_user                                         Manage AAA users (aaa:User)                                              aci_aaa_user_certificate                             Manage AAA user certificates (aaa:User...                        aci_access_port_block_to_access_port                 Manage port blocks of Fabric interface ...aci_access_port_to_interface_policy_leaf_profile     Manage Fabric interface policy leaf pro...aci_access_sub_port_block_to_access_port             Manage sub port blocks of Fabric interf...aci_aep                                              Manage attachable Access Entity Profile...aci_aep_to_domain                                    Bind AEPs to Physical or Virtual Domain...   aci_bd_subnet                                        Manage Subnets (fv:Subnet)                 ………………省略部分输出信息………………
```

一般情况下，真的很难通过名称来判别一个模块的作用，只能是参考后面的介绍信息或平时多学多练的积累才行。例如接下来随机查看一个模块的详细信息，ansible-doc会在屏幕显示出它的作用、可用参数及实例等等信息：

```
[root@linuxprobe ~]# ansible-doc a10_server> A10_SERVER    (/usr/lib/python3.6/site-packages/ansible/modules/network/a10/a10_server.py)     Manage SLB (Server Load Balancer) server objects on A10 Networks devices via aXAPIv2.  * This module is maintained by The Ansible Community………………省略部分输出信息………………
```

刚刚在16.2小节，咱们已经成功的将受管主机IP地址填写到了主机清单文件中，接下来小试牛刀，检查下这些主机的网络连通性吧。ping模块用于进行简单的网络测试，类似于常用的ping命令。可以用ansible命令直接针对所有主机调用ping模块，不需要增加额外的参数，返回值若为SUCCESS则代表主机当前在线。

```
[root@linuxprobe ~]# ansible all -m ping192.168.10.20 | SUCCESS => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false,    "ping": "pong"}192.168.10.21 | SUCCESS => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false,    "ping": "pong"}192.168.10.22 | SUCCESS => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false,    "ping": "pong"}192.168.10.23 | SUCCESS => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false,    "ping": "pong"}192.168.10.24 | SUCCESS => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false,    "ping": "pong"}
```

### **Tips**

由于五台受控主机的输出信息大致相同，因此为了提升读者阅读体验，本章节后续的输出结果默认仅保留192.168.10.20主机的输出值，其余相同的输出信息将会被省略。

是不是感觉很方便呢，一下子就能知道所有主机的在线情况。除了使用“-m”参数直接指定模块名称，还可以用“-a”将参数传递给模块，让功能更加高级，更好的贴合于当前生产需求。例如yum_repository模块的作用是管理主机的软件仓库，能够添加、修改及删除软件仓库的配置信息，参数相对比较复杂。遇到这种情况建议先用ansible-doc命令对其进行了解，尤其最下面的EXAMPLES结构段会有对该模块的实例，有着非常好的参考价值。

```
[root@linuxprobe ~]# ansible-doc yum_repository> YUM_REPOSITORY    (/usr/lib/python3.6/site-packages/ansible/modules/packaging>        Add or remove YUM repositories in RPM-based Linux        distributions. If you wish to update an existing repository        definition use [ini_file] instead.  * This module is maintained by The Ansible Core Team……………………省略部分输出信息………………EXAMPLES:- name: Add repository  yum_repository:    name: epel    description: EPEL YUM repo    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/- name: Add multiple repositories into the same file (1/2)  yum_repository:    name: epel    description: EPEL YUM repo    file: external_repos    baseurl: https://download.fedoraproject.org/pub/epel/$releasever/$basearch/    gpgcheck: no- name: Add multiple repositories into the same file (2/2)  yum_repository:    name: rpmforge    description: RPMforge YUM repo    file: external_repos    baseurl: http://apt.sw.be/redhat/el7/en/$basearch/rpmforge
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
[root@linuxprobe ~]# ansible all -m yum_repository -a 'name="EX294_BASE" description="EX294 base software" baseurl="file:///media/cdrom/BaseOS" gpgcheck=yes enabled=1 gpgkey="file:///media/cdrom/RPM-GPG-KEY-redhat-release"'192.168.10.20 | CHANGED => {    "ansible_facts": {        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": true,    "repo": "EX294_BASE",    "state": "present"}
```

命令执行成功后，可以到主机清单中任意服务器上查看到新建成功的软件仓库配置文件，实验虽然参数很多，但是并不难对吧~

```
[root@linuxprobe ~]# cat /etc/yum.repos.d/EX294_BASE.repo [EX294_BASE]baseurl = file:///media/cdrom/BaseOSenabled = 1gpgcheck = 1gpgkey = file:///media/cdrom/RPM-GPG-KEY-redhat-releasename = EX294 base software
```

##### **16.4 剧本文件实战**

作为一名技术狂兼戏剧迷，研究生一年级时曾经阅读过莎士比亚的《李尔王》和《暴风雨》两个剧本原作，深深的沉迷于作者对于剧情的巧妙设计中。执行单个的命令或调用某一个模块根本无法满足正常工作所需要的复杂度，Ansible服务中允许用户根据需求，在类似于Shell脚本的模式下编写出一套自动化运维的脚本，然后由程序自动的、重复的执行，大大的提高了工作效率。

Ansible服务的Playbook剧本文件采用YAML语言编写，有着强制性的格式规范，通过空格将不同信息分组，因此有时会因一两个空格错位而导致报错，需要万分小心。YAML文件开头需要先写三个减号---，多个分组信息需要间隔一致才能执行，上下也要对齐，后缀名一般为.yml。在执行后会在屏幕上输出运行界面，内容会依据不同工作而变化，但绿色均代表成功，黄色代表执行成功并进行了修改，而红色则代表执行失败。

Playbook剧本文件的结构由四部分组成——target、variable、task、handler。target部分用于定义要执行剧本的主机节点范围、variable部分用于定义剧本执行时要用的变量、task部分用于定义将在远程主机上执行的任务列表、handler部分用于定义执行完成后需要调用的后续任务。

YAML语言编写的Ansible剧本文件会按照从上至下的顺序自动运行，形式类似于第4章节学习过的Shell脚本，但格式有严格的要求。例如创建出一个名为packages.yml的playbook，目的是让dev、test和prod组的主机节点可以自动安装数据库软件，并且将dev组主机的软件更新至最新。

安装和更新软件需要使用yum模块，先查看下帮助信息中的实例吧：

```
[root@linuxprobe ~]# ansible-doc yum> YUM    (/usr/lib/python3.6/site-packages/ansible/modules/packaging/os/yum.py)        Installs, upgrade, downgrades, removes, and lists packages and        groups with the `yum' package manager. This module only works        on Python 2. If you require Python 3 support see the [dnf]        module.  * This module is maintained by The Ansible Core Team  * note: This module has a corresponding action plugin.………………省略部分输出信息………………EXAMPLES:- name: install the latest version of Apache  yum:    name: httpd    state: latest
```

在配置Ansible剧本文件时，ansible-doc命令所提供的帮助信息真是好用。知道了yum模块的使用方法和格式了，接下来就可以开始编写了。初次编写Playbook剧本文件时，请务必看准格式，模块及格式时间上下也要对齐，否则会出现参数一摸一样，但是却不能执行的情况。

其中name字段代表此项Play（动作）的名字，可自行命名，没有限制，用于在执行过程中提示用户执行到了那一步，以及帮助管理员日后阅读时回忆起这段代码的作用。hosts字段代表要在哪些主机节点上执行该剧本，多个主机组之间用逗号间隔，若需要对全部主机进行操作则用all参数。tasks字段用于定义要执行的任务，每个任务都要有一个独立的name字段进行命名，并且每个任务的name字段和模块名称都要严格上下对齐，参数要单独缩进。正确的写法应该是：

```
[root@linuxprobe ~]# vim packages.yml---- name: 安装软件包  hosts: dev,test,prod  tasks:          - name: one            yum:                    name: mariadb                    state: latest[root@linuxprobe ~]#
```

而错误的代码是这样的，感受下YAML语言对格式有多严格吧：

```
[root@linuxprobe ~]# vim packages.yml---- name: 安装软件包  hosts: dev,test,prod  tasks:          - name: one            yum:            name: mariadb            state: latest
```

在编写Ansible剧本文件时，RHEL 8系统自带的Vim编辑器会自动进行缩进，真是帮了不少忙呢~确认无误后就可以用ansible-playbook命令来运行这个剧本文件了

```
[root@linuxprobe ~]# ansible-playbook packages.yml PLAY [安装软件包] *******************************************************************TASK [Gathering Facts] **************************************************************ok: [192.168.10.20]ok: [192.168.10.21]ok: [192.168.10.22]ok: [192.168.10.23]TASK [one] **************************************************************************changed: [192.168.10.20]changed: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23]PLAY RECAP **************************************************************************192.168.10.20  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.21  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.22  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.23  : ok=2   changed=1  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   
```

执行成功后主要观察最下方的输出信息，ok和changed代表执行及修改成功。如遇到unreachable或failed大约0的情况，建议手动检查下剧本是否在所有主机中都正确运行了，以及有无安装失败的情况。正确执行过packages.yml文件后，随机切换到dev、test、prod组中任意一台节点主机上，再次安装mariadb软件包则会提示已经存在，刚刚的操作一切顺利~

```
[root@linuxprobe ~]# dnf install mariadbUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 1:05:53 ago on Thu 15 Apr 2021 08:29:11 AM CST.Package mariadb-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

##### **16.5 创建及使用角色**

日常编写Playbook的工作会让剧本内容越来越长，不利于阅读和维护，而且还无法让其他Playbook灵活的调用其中的功能代码。角色功能（roles）则是Ansible服务自1.2版本开始引入的新特性，用于层次性、结构化的组织剧本。角色功能通过分别把变量、文件、任务、模块及处理器配置放在各个独立的目录中，然后对其进行便捷加载的一种机制。简单来说，角色功能是把常用的一些功能“类模块化”，然后用的时候加载即可。

Ansible服务的角色功能也有些类似于编程中的封装技术，将具体的功能封装起来，用户不仅可以方便的调用它，甚至不用完全理解其中原理，也可以使用。就像普通消费者不需要深入理解汽车刹车是如何实现的那样，制动总泵、分泵、真空助力器、刹车盘、刹车鼓、刹车片或ABS泵都是藏于底层结构中，用户只需要用脚轻踩刹车踏板就能制动车子，这便是技术封装的好处。

角色的好处就在于其组织成了一个简洁的、可重复调用的抽象对象，用户可以把注意力放到Playbook的大局上，统筹各个关键性任务，只有在需要时才去深入了解细节。角色的获取有三种方法，分别是加载系统内置的、从外部环境获取的以及自行创建的，接下来将与读者共同逐一完成这些实验。

###### **16.5.1 加载系统内置角色**

使用RHEL系统内置角色，是一种不需要联网就能实现的功能。用户只需要配置好软件仓库的配置文件，然后安装包含系统角色的软件包rhel-system-roles，随后便可以在系统中找到它们了，以及能够使用Playbook剧本文件进行调用。

```
[root@linuxprobe ~]# dnf install -y rhel-system-rolesUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 1:06:26 ago on Tue 13 Apr 2021 07:22:03 AM CST.Dependencies resolved.================================================================================ Package                  Arch          Version          Repository        Size================================================================================Installing: rhel-system-roles        noarch        1.0-5.el8        AppStream        127 kTransaction Summary================================================================================Install  1 Package………………省略部分输出信息………………  Installed:  rhel-system-roles-1.0-5.el8.noarch                                            Complete!
```

安装完毕后，使用ansible-galaxy list命令查看RHEL 8系统中有哪些自带的角色可用：

```
[root@linuxprobe ~]# ansible-galaxy list# /usr/share/ansible/roles- linux-system-roles.kdump, (unknown version)- linux-system-roles.network, (unknown version)- linux-system-roles.postfix, (unknown version)- linux-system-roles.selinux, (unknown version)- linux-system-roles.timesync, (unknown version)- rhel-system-roles.kdump, (unknown version)- rhel-system-roles.network, (unknown version)- rhel-system-roles.postfix, (unknown version)- rhel-system-roles.selinux, (unknown version)- rhel-system-roles.timesync, (unknown version)# /etc/ansible/roles[WARNING]: - the configured path /root/.ansible/roles does not exist.
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
[root@linuxprobe ~]# vim timesync.yml ---- hosts: all  vars:    timesync_ntp_servers:      - hostname: pool.ntp.org        iburst: yes  roles:    - rhel-system-roles.timesync
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
[root@linuxprobe ~]# ansible-galaxy install nginxinc.nginx- downloading role 'nginx', owned by nginxinc- downloading role from https://github.com/nginxinc/ansible-role-nginx/archive/0.19.1.tar.gz- extracting nginxinc.nginx to /etc/ansible/roles/nginxinc.nginx- nginxinc.nginx (0.19.1) was installed successfully
```

执行完毕后再次查看系统中已有角色，便找到nginx角色信息了：

```
[root@linuxprobe ~]# ansible-galaxy list# /etc/ansible/roles- nginxinc.nginx, 0.19.1# /usr/share/ansible/roles- linux-system-roles.kdump, (unknown version)- linux-system-roles.network, (unknown version)- linux-system-roles.postfix, (unknown version)- linux-system-roles.selinux, (unknown version)- linux-system-roles.timesync, (unknown version)- rhel-system-roles.kdump, (unknown version)- rhel-system-roles.network, (unknown version)- rhel-system-roles.postfix, (unknown version)- rhel-system-roles.selinux, (unknown version)- rhel-system-roles.timesync, (unknown version)
```

但还存在一些特殊情况，其一是国内访问Ansible官网可能存在不稳定的情况，访问不了或者速度慢。其二是某位作者是将创作品上传到了自己的网站，或者除Ansible Galaxy官网以外的其他平台。那么这两种情况就不能再用“ansible-galaxy install 角色名称”的命令直接加载了，而是需要手动先编写一个YAML语言格式的文件，指明网址链接和角色名称，再用“-r”参数进行加载。

例如刘遄老师在随书配套网站（Www.LinuxProbe.Com）上传了一个叫做nginx_core的角色软件包，是用于对nginx网站进行保护的插件功能，编写yml配置文件如下：

```
[root@linuxprobe ~]# cat nginx.yml---- src: https://www.linuxprobe.com/Software/nginxinc-nginx_core-0.3.0.tar.gz  name: nginx-core
```

随后使用ansible-galaxy命令的“-r”参数加载这个文件，即可查看到新角色信息了

```
[root@linuxprobe ~]# ansible-galaxy install -r nginx.yml- downloading role from https://www.linuxprobe.com/nginxinc-nginx_core-0.3.0.tar.gz- extracting nginx to /etc/ansible/roles/nginx- nginx was installed successfully[root@linuxprobe ~]# ansible-galaxy list# /etc/ansible/roles- nginx-core, (unknown version)- nginxinc.nginx, 0.19.1# /usr/share/ansible/roles- linux-system-roles.kdump, (unknown version)- linux-system-roles.network, (unknown version)- linux-system-roles.postfix, (unknown version)- linux-system-roles.selinux, (unknown version)- linux-system-roles.timesync, (unknown version)- rhel-system-roles.kdump, (unknown version)- rhel-system-roles.network, (unknown version)- rhel-system-roles.postfix, (unknown version)- rhel-system-roles.selinux, (unknown version)- rhel-system-roles.timesync, (unknown version)
```

###### **16.5.3 创建新的角色**

除了能够使用系统镜像自带和Ansible Galaxy中获取的外部角色，也可以自行创建符合工作需求的新角色信息，定制化的编写工作能够更好的贴合生产环境实际情况，但难度也会稍高一些。接下来将会创建一个名称为apache的新角色信息，它能够帮助我们自动的安装、运行httpd网站服务，设置防火墙允许规则及根据每个主机生成独立的index.html首页文件，让用户调用后就能享受到一条龙的部署网站服务。

根据主配置文件第68行所定义的角色保存路径，如果用户新建的角色信息不在规定目录内，使用ansible-galaxy list命令则是无法找到的。因此需要手动填写下新角色目录的路径，或是进入到/etc/ansible/roles目录内再进行创建，为了避免后期角色信息过于分散导致不好管理，还是决定在默认目录下进行创建，不再修改。

```
[root@linuxprobe roles]# vim /etc/ansible/ansible.cfg 66  67 # additional paths to search for roles in, colon separated 68 #roles_path    = /etc/ansible/roles 69 
```

创建一个新的角色信息使用“init”参数，且建立成功后便会在当前目录下生成出一个新的目录：

```
[root@linuxprobe ~]# cd /etc/ansible/roles[root@linuxprobe roles]# ansible-galaxy init apache- Role apache was created successfully[root@linuxprobe roles]# lsapache nginx nginxinc.nginx
```

此时的apache即是角色名称，也是用于存在角色信息的目录名称，切换进入看到目录结构：

```
[root@linuxprobe roles]# cd apache[root@linuxprobe apache]# lsdefaults  files  handlers  meta  README.md  tasks  templates  tests  vars
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
[root@linuxprobe apache]# vim tasks/main.yml---- name: one  yum:          name: httpd          state: latest
```

第2步：使用service模块启动httpd网站服务程序，并加入到启动项中，保证能够一直为用户提供服务。初次使用模块前先用ansible-doc命令查看下帮助和实例信息吧，但由于书籍出版的限制，信息会做删减，仅保留有用的段落内容。

```
[root@linuxprobe apache]# ansible-doc service> SERVICE    (/usr/lib/python3.6/site-packages/ansible/modules/system/service.py)        Controls services on remote hosts. Supported init systems        include BSD init, OpenRC, SysV, Solaris SMF, systemd, upstart.        For Windows targets, use the [win_service] module instead.  * This module is maintained by The Ansible Core Team  * note: This module has a corresponding action plugin.………………省略部分输出信息………………EXAMPLES:- name: Start service httpd, if not started  service:    name: httpd    state: started- name: Enable service httpd, and not touch the state  service:    name: httpd    enabled: yes
```

真幸运，默认的EXAMPLES示例用的就是httpd网站服务，通过输出信息可得知启动服务为“state: started”参数，而加入到开机启动项则是“enabled: yes”参数。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml---- name: one  yum:          name: httpd          state: latest- name: two  service:          name: httpd          state: started          enabled: yes
```

第3步：配置防火墙放行允许策略，让其他主机可以正常访问。配置防火墙使用firewalld模块，同样也先看下帮助示例吧：

```
[root@linuxprobe defaults]# ansible-doc firewalld> FIREWALLD    (/usr/lib/python3.6/site-packages/ansible/modules/system/firewalld.py)        This module allows for addition or deletion of services and        ports (either TCP or UDP) in either running or permanent        firewalld rules.  * This module is maintained by The Ansible CommunityOPTIONS (= is mandatory):EXAMPLES:- firewalld:    service: https    permanent: yes    state: enabled- firewalld:    port: 8081/tcp    permanent: yes    state: disabled    immediate: yes
```

依据输出信息可得知，firewalld模块设置防火墙策略中，指定协议名称为“service: http”参数，放行该协议为“state: enabled”参数，设置为永久生效为“permanent: yes”参数，当前也能立即生效为“immediate: yes”参数。参数虽然多了一些，但是基本与在第8章节学习的一致，并不需要担心。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml---- name: one  yum:          name: httpd          state: latest- name: two  service:          name: httpd          state: started          enabled: yes- name: three  firewalld:          service: http          permanent: yes          state: enabled          immediate: yes
```

第4步：让每个主机显示的首页内容均不同。常规的模块都是这样查一个、写一个就能搞定，为了增加难度再提出个新需求，能否让每个主机上运行的httpd网站服务都有不同的内容，例如显示当前服务器的主机名及IP地址呢？这样的话就要用到template模块及Jinja2技术了。

template模块的使用方法依然用ansible-doc命令进行查询，示例部分有很大帮助：

```
[root@linuxprobe apache]# ansible-doc template> TEMPLATE    (/usr/lib/python3.6/site-packages/ansible/modules/files/template.>        Templates are processed by the L(Jinja2 templating        language,http://jinja.pocoo.org/docs/). Documentation on the        template formatting can be found in the L(Template Designer        Documentation,http://jinja.pocoo.org/docs/templates/).        Additional variables listed below can be used in templates.        `ansible_managed' (configurable via the `defaults' section of        `ansible.cfg') contains a string which can be used to describe        the template name, host, modification time of the template        file and the owner uid. `template_host' contains the node name        of the template's machine. `template_uid' is the numeric user        id of the owner. `template_path' is the path of the template.        `template_fullpath' is the absolute path of the template.        `template_destpath' is the path of the template on the remote        system (added in 2.8). `template_run_date' is the date that        the template was rendered.  * This module is maintained by The Ansible Core Team  * note: This module has a corresponding action plugin.………………省略部分输出信息………………EXAMPLES:- name: Template a file to /etc/files.conf  template:    src: /mytemplates/foo.j2    dest: /etc/file.conf    owner: bin    group: wheel    mode: '0644'
```

从template模块的输出信息中可得知，这是一个用于复制文件模板的模块，能够把文件从Ansible服务器复制到受管主机上，“src”参数用于定义本地文件的路径，“dest”参数用于定义复制到受管主机的文件路径，而owner、group、mode参数可选择性的设置文件归属及权限信息。

正常来说复制文件的操作是直接进行的，受管主机节点上会获取到一个与Ansible服务器上一摸一样的文件，但有些时候想让每台客户端根据自身系统情况产生不同的文件信息，这样就需要使用到Jinja2技术了，文件后缀是.j2结尾。继续编写：

```
[root@linuxprobe apache]# vim tasks/main.yml---- name: one  yum:          name: httpd          state: latest- name: two  service:          name: httpd          state: started          enabled: yes- name: three  firewalld:          service: http          permanent: yes          state: enabled          immediate: yes- name: four  template:          src: index.html.j2          dest: /var/www/html/index.html
```

![第16章 使用Ansible服务实现自动化运维第16章 使用Ansible服务实现自动化运维](https://www.linuxprobe.com/wp-content/uploads/2021/03/jinja.png)

Jinja2是Python语言中一个被广泛使用的模板引擎，最初的设计思想来源是Django的模块引擎，基于此发展了其语法和一些列强大的功能，能够让受管主机根据自身变量而产生出不同的文件内容。话句话说，正常情况下的复制操作会让新旧文件一摸一样，但Jinja2技术不会在原始文件中直接写入文件内容，而是一系列的变量名称，在使用template模块进行复制的过程中，由Ansible服务负责在受管主机上收集这些变量名称所对应的值，最后再逐一填写到目标文件中，能够让每台主机的文件都根据自身系统情况来独立生成。

例如想要让每个网站的输出信息值为“Welcome to 主机名 on 主机地址”，也就是用每个主机自己独有的名称和IP地址替换文本中的内容，这样就有趣太多了。这种实验的难点主要是查询到对应的变量名称，主机名及地址所对应的值保存在哪里？可以用setup模块进行查询。

```
[root@linuxprobe apache]# ansible-doc setup> SETUP    (/usr/lib/python3.6/site-packages/ansible/modules/system/setup.py)        This module is automatically called by playbooks to gather        useful variables about remote hosts that can be used in        playbooks. It can also be executed directly by        `/usr/bin/ansible' to check what variables are available to a        host. Ansible provides many `facts' about the system,        automatically. This module is also supported for Windows        targets.
```

setup模块的作用是自动收集受管主机上的变量信息，用-a参数追加filter指令可以对收集来的进行进行二次过滤，语法格式为ansible all -m setup -a 'filter="*关键词*"'，其中*号是第三章节学习的通配符，进行关键词查询。例如想搜索各个主机的名称，即用通配符搜索所有包含fqdn关键词的变量值信息。

Fully Qualified Domain Name即完全限定主机名，简称FQDN，用于在逻辑上准确表示出主机的位置，也常常被作为主机名的完全表达形式，比/etc/hostname文件中所定义的主机名更加严谨和准确。通过输出信息可得知，ansible_fqdn变量保存有主机名称，随后再进行下一步操作：

```
[root@linuxprobe ~]# ansible all -m setup -a 'filter="*fqdn*"'192.168.10.20 | SUCCESS => {    "ansible_facts": {        "ansible_fqdn": "linuxprobe.com",        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false}………………省略部分输出信息………………
```

用于定制主机地址的变量可以用“ip”作为关键词进行检索，能够看到在ansible_all_ipv4_addresses变量中的值是我们想要的信息。如果想输出IPV6级别的地址则可用ansible_all_ipv6_addresses变量。

```
[root@linuxprobe ~]# ansible all -m setup -a 'filter="*ip*"'192.168.10.20 | SUCCESS => {    "ansible_facts": {        "ansible_all_ipv4_addresses": [            "192.168.10.20",            "192.168.122.1"        ],        "ansible_all_ipv6_addresses": [            "fe80::d0bb:17c8:880d:e719"        ],        "ansible_default_ipv4": {},        "ansible_default_ipv6": {},        "ansible_fips": false,        "discovered_interpreter_python": "/usr/libexec/platform-python"    },    "changed": false}………………省略部分输出信息………………
```

在确认了主机名与IP地址所对应的具体变量名称后，在角色所对应的templates目录内新建一个与上面template模块参数相同的文件名称（index.html.j2）。Jinja2在调用变量值时，格式为在变量名称的两侧格加两个大括号，编写完成即：

```
[root@linuxprobe apache]# vim templates/index.html.j2Welcome to {{ ansible_fqdn }} on {{ ansible_all_ipv4_addresses }}
```

进行到了这里，任务基本就算完成了。最后要做的就是编写一个用于调用apache角色的yml文件，以及执行它。

```
[root@linuxprobe apache]# cd ~[root@linuxprobe ~]# vim roles.yml---- name: 调用自建角色  hosts: all  roles:          - apache[root@linuxprobe ~]# ansible-playbook roles.yml PLAY [调用自建角色] **************************************************************************TASK [Gathering Facts] **********************************************************************ok: [192.168.10.20]ok: [192.168.10.21]ok: [192.168.10.22]ok: [192.168.10.23]ok: [192.168.10.24]TASK [apache : one] *************************************************************************changed: [192.168.10.20]changed: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23]changed: [192.168.10.24]TASK [apache : two] *************************************************************************changed: [192.168.10.20]changed: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23]changed: [192.168.10.24]TASK [apache : three] ***********************************************************************changed: [192.168.10.20]changed: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23]changed: [192.168.10.24]TASK [apache : four] ***********************************************************************changed: [192.168.10.20]changed: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23] changed: [192.168.10.24]PLAY RECAP **********************************************************************************192.168.10.20   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.21   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.22   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.23   : ok=5   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0   192.168.10.24   : ok=4   changed=4  unreachable=0   failed=0   skipped=0   rescued=0   ignored=0
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
[root@linuxprobe ~]# ansible-doc lvg> LVG    (/usr/lib/python3.6/site-packages/ansible/modules/system/lvg.py)        This module creates, removes or resizes volume groups.  * This module is maintained by The Ansible Community………………省略部分输出信息………………EXAMPLES:- name: Create a volume group on top of /dev/sda1 with physical extent size = 3>  lvg:    vg: vg.services    pvs: /dev/sda1    pesize: 32- name: Create a volume group on top of /dev/sdb with physical extent size = 12>  lvg:    vg: vg.services    pvs: /dev/sdb    pesize: 128K
```

通过输出信息可得知，创建PV和VG卷组的lvg模块总共有三个必备参数。其中“vg”参数用于定义卷组的名称，“pvs”参数用于指定硬盘设备名称，而“pesize”参数用于确定最终卷组的容量大小，可以用PE个数亦可用容量值进行指定。这样的话先创建出一个由/dev/sdb设备组成的名称为research，大小为150M的卷组设备吧。

```
[root@linuxprobe ~]# vim lv.yml---- name: 创建和使用逻辑卷  hosts: all  tasks:          - name: one            lvg:                    vg: research                    pvs: /dev/sdb                    pesize: 150M
```

由于刚刚仅在prod组的两台主机上添加了新硬盘设备文件，因此稍后执行时其余三台主机节点会提示未创建成功，属于正常情况。接下来便是用lvol模块来创建出LVM逻辑卷设备了，先查看下模块帮助信息吧：

```
[root@linuxprobe ~]# ansible-doc lvol> LVOL    (/usr/lib/python3.6/site-packages/ansible/modules/system/lvol.py)        This module creates, removes or resizes logical volumes.  * This module is maintained by The Ansible Community………………省略部分输出信息………………EXAMPLES:- name: Create a logical volume of 512m  lvol:    vg: firefly    lv: test    size: 512- name: Create a logical volume of 512m with disks /dev/sda and /dev/sdb  lvol:    vg: firefly    lv: test    size: 512    pvs: /dev/sda,/dev/sdb
```

通过输出信息可得知，lvol确定是用于创建LVM逻辑卷设备的模块，其中“vg”参数用于指定卷组名称，“lv”参数用于指定逻辑卷名称，而“size”参数则用于指定最终逻辑卷设备的容量大小，不加单位默认为M。填写好参数，创建出一个大小为150M，归属于research卷组，名称为data的逻辑卷设备:

```
[root@linuxprobe ~]# vim lv.yml---- name: 创建和使用逻辑卷  hosts: all  tasks:          - name: one            lvg:                    vg: research                    pvs: /dev/sdb                    pesize: 150M          - name: two            lvol:                    vg: research                    lv: data                    size: 150M
```

这样还不够好，如果能再将创建出的/dev/research/data逻辑卷设备自动用ext4文件系统进行格式化操作，则又能帮助运维管理员减少了一些工作量。对于设备的文件系统格式化操作使用filesystem模块完成，帮助信息如下：

```
[root@linuxprobe ~]# ansible-doc filesystem> FILESYSTEM    (/usr/lib/python3.6/site-packages/ansible/modules/system/filesy>        This module creates a filesystem.  * This module is maintained by The Ansible Community………………省略部分输出信息………………EXAMPLES:- name: Create a ext2 filesystem on /dev/sdb1  filesystem:    fstype: ext2    dev: /dev/sdb1
```

filesystem模块的参数真是简练，参数“fstype”用于指定文件系统的格式化类型，“dev”参数用于指定要格式化的设备文件路径。继续编写：

```
[root@linuxprobe ~]# vim lv.yml---- name: 创建和使用逻辑卷  hosts: all  tasks:          - name: one            lvg:                    vg: research                    pvs: /dev/sdb                    pesize: 150M          - name: two            lvol:                    vg: research                    lv: data                    size: 150M          - name: three            filesystem:                    fstype: ext4                    dev: /dev/research/data 
```

这样按照顺序执行下来，LVM逻辑卷设备就能够自动创建好了。等等！还有个问题没有解决！现在只有prod组主机上面添加了新的硬盘设备文件，其余主机是无法按照既定模块顺利完成的，这时要用类似于第4章节学习过的if条件句的方式进行一次判断——如果失败.....则怎么样....。

首先将上述的三个模块命令用block操作符作为一个整体，相当于是对这三个模块执行结果作为一个整体判断。然后使用rescue操作符进行救援，只有block块中的模块执行失败了才会调用rescue中的救援模块。其中debug模块的msg参数的作用是如果block中的模块执行失败则输出一条信息到屏幕，给予用户一定的提醒作用。完成编写后是这个样子的：

```
[root@linuxprobe ~]# vim lv.yml---- name: 创建和使用逻辑卷  hosts: all  tasks:          - block:                  - name: one                    lvg:                            vg: research                            pvs: /dev/sdb                            pesize: 150M                  - name: two                    lvol:                            vg: research                            lv: data                            size: 150M                  - name: three                    filesystem:                            fstype: ext4                            dev: /dev/research/data            rescue:                    - debug:                            msg: "Could not create logical volume of that size"
```

YAML语言对于格式有着硬性的要求，既然rescue是对block内的模块进行救援的功能代码，因此两个操作符必须严格对齐，错开一个空格都会导致Playbook执行失败。确认无误后，执行lv.yml剧本文件检阅下效果吧：

```
[root@linuxprobe ~]# ansible-playbook lv.yml PLAY [创建和使用逻辑卷] *********************************************************TASK [Gathering Facts] *********************************************************ok: [192.168.10.20]ok: [192.168.10.21]ok: [192.168.10.22]ok: [192.168.10.23]ok: [192.168.10.24]TASK [one] *********************************************************************fatal: [192.168.10.20]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}fatal: [192.168.10.21]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}changed: [192.168.10.22]changed: [192.168.10.23]fatal: [192.168.10.24]: FAILED! => {"changed": false, "msg": "Device /dev/sdb not found."}TASK [two] *********************************************************************changed: [192.168.10.22]changed: [192.168.10.23]TASK [three] *********************************************************************changed: [192.168.10.22]changed: [192.168.10.23]TASK [debug] *******************************************************************ok: [192.168.10.20] => {    "msg": "Could not create logical volume of that size"}ok: [192.168.10.21] => {    "msg": "Could not create logical volume of that size"}ok: [192.168.10.24] => {    "msg": "Could not create logical volume of that size"}PLAY RECAP *********************************************************************192.168.10.20  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0   192.168.10.21  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0   192.168.10.22  : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   192.168.10.23  : ok=4  changed=3  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   192.168.10.24  : ok=2  changed=0  unreachable=0  failed=0  skipped=0  rescued=1  ignored=0 
```

在Playbook运行完毕后的执行记录（PLAY RECAP）中可以很清晰的看出只有192.168.10.22及192.168.10.23两台prod组中的主机执行成功了，其余三台主机均触发了rescue救援功能。登录到任意一台prod组的主机节点上，找到新建的逻辑卷设备信息：

```
[root@linuxprobe ~]# lvdisplay   --- Logical volume ---  LV Path                /dev/research/data  LV Name                data  VG Name                research  LV UUID                EOUliC-tbkk-kOJR-8NaH-O9XQ-ijrK-TgEYGj  LV Write Access        read/write  LV Creation host, time linuxprobe.com, 2021-04-23 11:00:21 +0800  LV Status              available  # open                 0  LV Size                5.00 GiB  Current LE             1  Segments               1  Allocation             inherit  Read ahead sectors     auto  - currently set to     8192  Block device           253:2………………省略部分输出信息………………
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
[root@linuxprobe ~]# ansible-doc copy> COPY    (/usr/lib/python3.6/site-packages/ansible/modules/files/copy.py)        The `copy' module copies a file from the local or remote        machine to a location on the remote machine. Use the [fetch]        module to copy files from remote locations to the local box.        If you need variable interpolation in copied files, use the        [template] module. Using a variable in the `content' field        will result in unpredictable output. For Windows targets, use        the [win_copy] module instead.  * This module is maintained by The Ansible Core Team  * note: This module has a corresponding action plugin.………………省略部分输出信息………………EXAMPLES:- name: Copy file with owner and permissions  copy:    src: /srv/myfiles/foo.conf    dest: /etc/foo.conf    owner: foo    group: foo    mode: '0644'- name: Copy using inline content  copy:    content: '# This file was moved to /etc/other.conf'    dest: /etc/mine.conf
```

在输出信息中列举了两种管理文件内容的示例，第一是对于文件的复制行为，第二是通过“content”参数定义内容，“dest”参数指定新建文件的名称，显然第二种更加符合当前的实验场景。编写剧本文件如下：

```
[root@linuxprobe ~]# vim issue.yml---- name: 修改文件内容  hosts: all  tasks:          - name: one            copy:                    content: 'Development'                    dest: /etc/issue          - name: two            copy:                    content: 'Test'                    dest: /etc/issue          - name: three            copy:                    content: 'Production'                    dest: /etc/issue
```

但按照这种顺序执行下去，每一台主机节点的/etc/issue文件都会被重复修改三次，最终定格在“Production”字样，显然缺少了一些东西。我们应该依据“inventory_hostname”变量中的值进行判断，若主机为dev组则执行第一个Play，若主机为test组则执行第二个Play，若主机为prod组则执行第三个Play，因此要进行三次判断。

when是进行判断的语法，需要在每个Play下方进行判断，只有满足条件才会执行：

```
[root@linuxprobe ~]# vim issue.yml---- name: 修改文件内容  hosts: all  tasks:          - name: one            copy:                    content: 'Development'                    dest: /etc/issue            when: "inventory_hostname in groups.dev"          - name: two            copy:                    content: 'Test'                    dest: /etc/issue            when: "inventory_hostname in groups.test"          - name: three            copy:                    content: 'Production'                    dest: /etc/issue            when: "inventory_hostname in groups.prod"
```

执行Playbook剧本文件，过程中可清晰的看出由于when语法的作用，未在指定主机组中的节点将被skipping（跳过）：

```
[root@linuxprobe ~]# ansible-playbook issue.yml PLAY [修改文件内容] ************************************************************************TASK [Gathering Facts] ********************************************************************ok: [192.168.10.20]ok: [192.168.10.21]ok: [192.168.10.22]ok: [192.168.10.23]ok: [192.168.10.24]TASK [one] ********************************************************************************changed: [192.168.10.20]skipping: [192.168.10.21]skipping: [192.168.10.22]skipping: [192.168.10.23] skipping: [192.168.10.24]TASK [two] ********************************************************************************skipping: [192.168.10.20]changed: [192.168.10.21]skipping: [192.168.10.23]skipping: [192.168.10.24]skipping: [192.168.10.25]TASK [three] ******************************************************************************skipping: [192.168.10.20]skipping: [192.168.10.21]changed: [192.168.10.22]changed: [192.168.10.23]skipping: [192.168.10.24]PLAY RECAP ********************************************************************************192.168.10.20   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   192.168.10.21   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   192.168.10.22   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0   192.168.10.23   : ok=2  changed=1  unreachable=0  failed=0  skipped=2  rescued=0  ignored=0 192.168.10.24   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0 
```

登录到dev组的192.168.10.20主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue Development
```

登录到test组的192.168.10.21主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue Test
```

登录到prod组的192.168.10.22/23主机节点上，查看文件内容：

```
[root@linuxprobe ~]# cat /etc/issue Production
```

##### **16.8 管理文件属性**

学习Playbook剧本语法的目的是满足日常的工作需求，能够把重复的事情写入到脚本中，然后再批量的执行下去，提高运维工作的效率。其中创建文件、管理权限及做快捷方式一定是几乎每天都被使用到的高频技能。尤其是在第5章节学习文件一般权限、特殊权限、隐藏权限时，往往还会因命令的格式问题导致出错，这么多命令该怎么记呢？

Ansible服务为了心疼运维人员，将对文件管理常用的功能都合并到了file模块中，不用再为了寻找模块而“东奔西跑”了，先来看下帮助信息吧：

```
[root@linuxprobe ~]# ansible-doc file> FILE    (/usr/lib/python3.6/site-packages/ansible/modules/files/file.py)        Set attributes of files, symlinks or directories.        Alternatively, remove files, symlinks or directories. Many        other modules support the same options as the `file' module -        including [copy], [template], and [assemble]. For Windows        targets, use the [win_file] module instead.  * This module is maintained by The Ansible Core Team………………省略部分输出信息………………EXAMPLES:- name: Change file ownership, group and permissions  file:    path: /etc/foo.conf    owner: foo    group: foo    mode: '0644'- name: Create a symbolic link  file:    src: /file/to/link/to    dest: /path/to/symlink    owner: foo    group: foo    state: link- name: Create a directory if it does not exist  file:    path: /etc/some_directory    state: directory    mode: '0755'- name: Remove file (delete file)  file:    path: /etc/foo.txt    state: absent
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
[root@linuxprobe ~]# vim chmod.yml---- name: 管理文件属性  hosts: dev  tasks:          - name: one            file:                    path: /linuxprobe                    state: directory                     owner: root                    group: root                    mode: '2775'
```

一不小心把题目出简单了，没能完全展示出file模块的强大之处。再临时加一个需求吧——请将新创建的目录做一个快捷方式到/linuxcool目录。这样用户在访问两个文件时都能有相同的内容了，使用file模块做快捷方式时，不需要再单独创建目标文件，Ansible服务会帮咱们完成的：

```
[root@linuxprobe ~]# vim chmod.yml---- name: 管理文件属性  hosts: dev  tasks:          - name: one            file:                    path: /linuxprobe                    state: directory                     owner: root                    group: root                    mode: '2775'          - name: two            file:                    src: /linuxprobe                    dest: /linuxcool                    state: link
```

Playbook剧本文件执行过程：

```
[root@linuxprobe ~]# ansible-playbook chmod.yml PLAY [管理文件属性] ***************************************************************TASK [Gathering Facts] ***********************************************************ok: [192.168.10.20]ok: [192.168.10.21]ok: [192.168.10.22]ok: [192.168.10.23]ok: [192.168.10.24]TASK [one] ***********************************************************************changed: [192.168.10.20]skipping: [192.168.10.21]skipping: [192.168.10.22]skipping: [192.168.10.23]skipping: [192.168.10.24]TASK [two] ***********************************************************************changed: [192.168.10.20]skipping: [192.168.10.21]skipping: [192.168.10.22]skipping: [192.168.10.23]skipping: [192.168.10.24]PLAY RECAP ***********************************************************************192.168.10.20   : ok=3  changed=2  unreachable=0  failed=0  skipped=0  rescued=0  ignored=0   192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0192.168.10.22   : ok=1  changed=0  unreachable=0  failed=0  skipped=3  rescued=0  ignored=0
```

进入到dev组的主机中，查看到/linuxprobe目录及/linuxcool快捷方式均已经被顺利创建，实验顺利完成：

```
[root@linuxprobe ~]# ls -ld /linuxprobedrwxrwsr-x. 2 root root 6 Apr 20 09:52 /linuxprobe[root@linuxprobe ~]# ls -ld /linuxcoollrwxrwxrwx. 1 root root 11 Apr 20 09:52 /linuxcool -> /linuxprobe
```

##### **16.9 管理密码库文件**

Ansible服务自1.5版本发布后，“Vault”作为一项新功能进入到了运维人员的视野，它不仅实现对密码、剧本等敏感信息进行加密，甚至可以加密变量名称和变量值，保护了数据不会被别人轻易的阅读到。使用ansible-vault命令可以实现对内容的新建（create）、加密（encrypt）、解密（decrypt ）、修改口令（rekey）及查看（view）等等功能。

第1步：例如创建出一个名为locker.yml的配置文件，其中保存有两个变量值：

```
[root@linuxprobe ~]# vim locker.yml---pw_developer: Imadevpw_manager: Imamgr
```

第2步：使用ansible-vault命令对文件进行加密，需要每次输入密码比较麻烦，因此还应新建出一个用于保存密码值的文本文件，让ansible-vault进行自动调用。为了保证数据的安全性，新建密码文件后将权限设置为600，仅管理员可读可写：

```
[root@linuxprobe ~]# vim /root/secret.txtwhenyouwishuponastar[root@linuxprobe ~]# chmod 600 /root/secret.txt
```

在Ansible服务的主配置文件中进行调用，在第140行的“vault_password_file”参数后指定密码值保存的文件路径：

```
[root@linuxprobe ~]# vim /etc/ansible/ansible.cfg137 138 # If set, configures the path to the Vault password file as an alternative to139 # specifying --vault-password-file on the command line.140 vault_password_file = /root/secret.txt141 
```

第3步：设置好了密码文件路径，Ansible服务便会自动进行加载，用户不再需要每次加密或解密时都重复输入密码值了。例如加密刚刚创建的locker.yml文件时，只需要使用“encrypt”参数即可：

```
[root@linuxprobe ~]# ansible-vault encrypt locker.ymlEncryption successful
```

加密过后的文件将使用AES 256格式进行加密，也就是意味着2^256就是256位AES密钥空间的组合数2^256>2^(10*25)>10^(3*25)=10^75>>>3×10^23，以天河4号超级计算机为例，每秒进行20亿亿次破解计算，也无法在我们有生之年搞定这串密码值。查看到加密后的内容为：

```
[root@linuxprobe ~]# cat locker.yml $ANSIBLE_VAULT;1.1;AES256386532343138393361383839316638373335333961613437303535303130383136316534393663353432346333346239386334663836643432353434373733310a306662303565633762313232663763383663343162393762626562306435316566653761666636356564363633386264643334303431626664643035316133650a333331393538616130656136653630303239663561663237373733373638623832343030616238656334663366363639616230393432363563363563616137363337396239616334303865663838623363333339396637363061626363383266
```

那如果不想用原始密码了呢，也可以手动对文件进行“rekey”修改密码操作，同时应结合“--ask-vault-pass”参数进行修改，否则Ansible服务会因接收不到用户输入的旧密码值，而拒绝新的密码变更请求：

```
[root@linuxprobe ~]# ansible-vault rekey --ask-vault-pass locker.yml Vault password: 输入旧的密码New Vault password: 输入新的密码Confirm New Vault password: 再输入新的密码Rekey successful
```

第4步：那如果想查看和修改加密文件中的内容呢。对于已经加密过的文件，需要使用ansible-vault命令的“edit”参数进行修改，随后用“view”参数即可查看到修改后的内容。编辑操作默认使用Vim编辑器作为修改工具，请修改完毕后记得wq保存退出：

```
[root@linuxprobe ~]# ansible-vault edit locker.yml---pw_developer: Imadevpw_manager: Imamgrpw_production: Imaprod
```

最后，再用“view”参数进行查看，便是最新的内容了：

```
[root@linuxprobe ~]# ansible-vault view locker.ymlVault password: 输入密码后敲击回车确认--- pw_developer: Imadev pw_manager: Imamgr pw_production: Imaprod
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

# [第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/basic-learning-17.html)

**章节概述：**

本章开篇介绍了计算机硬件存储设备的不同接口技术的优缺点，并由此切入iSCSI技术主题的讲解。iSCSI技术实现了物理硬盘设备与TCP/IP网络协议的相互结合，使得用户能够通过互联网方便地访问远程机房提供的共享存储资源。本章将带领大家在[Linux系统](https://www.linuxprobe.com/)上部署iSCSI服务端程序，并分别基于[Linux](https://www.linuxprobe.com/)系统和Windows系统来访问远程的存储资源。通过本章以及第6章、第7章的学习，读者将进一步理解和掌握如何在Linux系统中管理硬盘设备和存储资源，为今后走向运营岗位打下坚实的基础。

本章目录结构

- [17.1 iSCSI技术介绍](https://www.linuxprobe.com/basic-learning-17.html#171_iSCSI)
- [17.2 创建RAID磁盘阵列](https://www.linuxprobe.com/basic-learning-17.html#172_RAID)
- [17.3 配置iSCSI服务端](https://www.linuxprobe.com/basic-learning-17.html#173_iSCSI)
- [17.4 配置Linux客户端](https://www.linuxprobe.com/basic-learning-17.html#174Linux)
- [17.5 配置Windows客户端](https://www.linuxprobe.com/basic-learning-17.html#175_Windows)

##### **17.1 iSCSI技术介绍**

硬盘是计算机硬件设备中重要的组成部分之一，硬盘存储设备读写速度的快慢也会对服务器的整体性能造成影响。第6章、第7章讲解的硬盘存储结构、RAID磁盘阵列技术以及LVM技术等都是用于存储设备的技术，尽管这些技术有软件层面和硬件层面之分，但是它们都旨在解决硬盘存储设备的读写速度问题，或者竭力保障存储数据的安全。

为了进一步提升硬盘存储设备的读写速度和性能，人们一直在努力改进物理硬盘设备的接口协议。当前的硬盘接口类型主要有IDE、SCSI和SATA这3种。

> IDE是一种成熟稳定、价格便宜的并行传输接口。
>
> SATA是一种传输速度更快、数据校验更完整的串行传输接口。
>
> SCSI是一种用于计算机和硬盘、光驱等设备之间系统级接口的通用标准，具有系统资源占用率低、转速高、传输速度快等优点。

不论使用什么类型的硬盘接口，硬盘上的数据总是要通过计算机主板上的总线与CPU、内存设备进行数据交换，这种物理环境上的限制给硬盘资源的共享带来了各种不便。后来，IBM公司开始动手研发基于TCP/IP协议和SCSI接口协议的新型存储技术，这也就是我们目前能看到的互联网小型计算机系统接口（iSCSI，Internet Small Computer System Interface）。这是一种将SCSI接口与以太网技术相结合的新型存储技术，可以用来在网络中传输SCSI接口的[命令](https://www.linuxcool.com/)和数据。这样，不仅克服了传统SCSI接口设备的物理局限性，实现了跨区域的存储资源共享，还可以在不停机的状态下扩展存储容量。

为了让各位读者做到知其然，知其所以然，以便在工作中灵活使用这项技术，下面将讲解一下iSCSI技术在生产环境中的优势和劣势。首先，iSCSI存储技术非常便捷，在访问存储资源的形式上发生了很大变化，摆脱了物理环境的限制，同时还可以把存储资源分给多个服务器共同使用，因此是一种非常推荐使用的存储技术。但是，iSCSI存储技术受到了网速的制约。以往硬盘设备直接通过主板上的总线进行数据传输，现在则需要让互联网作为数据传输的载体和通道，因此传输速率和稳定性是iSCSI技术的瓶颈。随着网络技术的持续发展，相信iSCSI技术也会随之得以改善。

既然要通过以太网来传输硬盘设备上的数据，那么数据是通过网卡传入到计算机中的么？这就有必要向大家介绍iSCSI-HBA卡了（见图17-1）。与一般的网卡不同（连接网络总线和内存，供计算机上网使用），iSCSI-HBA卡连接的则是SCSI接口或FC（光纤通道）总线和内存，专门用于在主机之间交换存储数据，其使用的协议也与一般网卡有本质的不同。运行Linux系统的服务器会基于iSCSI协议把硬盘设备[命令](https://www.linuxcool.com/)与数据打包成标准的TCP/IP数据包，然后通过以太网传输到目标存储设备，而当目标存储设备接收到这些数据包后，还需要基于iSCSI协议把TCP/IP数据包解压成硬盘设备命令与数据。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2015/09/ISCSI-HBA%E5%8D%A1.jpg)

图17-1 iSCSI-HBA卡实拍图

总结来说，iSCSI技术具有硬件成本低、操作简单、维护方便以及扩展性强等优势。为我们提供了数据集中化存储的服务，以区块为单位的数据存储空间，在简化了存储空间管理步骤的前提下，还增添了存储空间的弹性。对于用户而言，仿佛计算机上多了一块新的“本地硬盘”，可以使用本地段计算机操作系统进行管理，像是在用自己本地硬盘一样使用远程存储空间。这种高扩展性和低组建、低维护成本的整合存储方式，正是大部分有预算考虑的中小企业和办公室所需求的。

##### **17.2 创建RAID磁盘阵列**

既然要使用iSCSI存储技术为远程用户提供共享存储资源，首先要保障用于存放资源的服务器的稳定性与可用性，否则一旦在使用过程中出现故障，则维护的难度相较于本地硬盘设备要更加复杂、困难。因此推荐各位读者按照本书第7章讲解的知识来部署RAID磁盘阵列组，确保数据的安全性。下面以配置RAID 5磁盘阵列组为例进行讲解。考虑到第7章已经事无巨细地讲解了RAID磁盘阵列技术和配置方法，因此本节不会再重复介绍相关参数的意义以及用途，忘记了的读者可以翻回去看一下。

首先在虚拟机中添加4块新硬盘，用于创建RAID 5磁盘阵列和备份盘，如图17-2所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E6%B7%BB%E5%8A%A0%E6%96%B0%E7%A1%AC%E7%9B%98.png)

图17-2 添加4块用于创建RAID 5级别磁盘阵列的新硬盘

启动虚拟机系统，使用mdadm命令创建RAID磁盘阵列。其中，“-Cv”参数为创建阵列并显示过程，/dev/md0为生成的阵列组名称，“-n 3”参数为创建RAID 5磁盘阵列所需的硬盘个数，“-l 5”参数为RAID磁盘阵列的级别，“-x 1”参数为磁盘阵列的备份盘个数。在命令后面要逐一写上使用的硬盘名称。另外，还允许使用第3章讲解的通配符来指定硬盘设备的名称，有兴趣的读者可以试一下。

```
[root@linuxprobe ~]# mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sdb /dev/sdc /dev/sdd /dev/sde
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 20954112K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

在上述命令成功执行之后，得到一块名称为/dev/md0的新设备，这是一块RAID 5级别的磁盘阵列，并且还有一块备份盘为硬盘数据保驾护航。大家可使用mdadm -D命令来查看设备的详细信息。另外，由于在使用远程设备时极有可能出现设备识别顺序发生变化的情况，因此，如果直接在fstab挂载配置文件中写入/dev/sdb、/dev/sdc等设备名称的话，就有可能在下一次挂载了错误的存储设备。而UUID值是设备的唯一标识符，用于精确地区分本地或远程设备。于是我们可以把这个值记录下来，一会儿准备填写到挂载配置文件中。

```
[root@linuxprobe ~]# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Tue Apr 27 08:06:43 2021
        Raid Level : raid5
        Array Size : 41908224 (39.97 GiB 42.91 GB)
     Used Dev Size : 20954112 (19.98 GiB 21.46 GB)
      Raid Devices : 3
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Tue Apr 27 08:08:28 2021
             State : clean 
    Active Devices : 3
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 1

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : linuxprobe.com:0  (local to host linuxprobe.com)
              UUID : 759282f9:093dbf7c:a6c4a16d:ed70333c
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       4       8       48        2      active sync   /dev/sdd

       3       8       64        -      spare   /dev/sde
```

##### **17.3 配置iSCSI服务端**

iSCSI技术在工作形式上分为服务端（target）与客户端（initiator）。iSCSI服务端即用于存放硬盘存储资源的服务器，它作为前面创建的RAID磁盘阵列的存储端，能够为用户提供可用的存储资源。iSCSI客户端则是用户使用的软件，用于访问远程服务端的存储资源。下面按照表17-1来配置iSCSI服务端和客户端所用的IP地址。

表17-1               iSCSI服务端和客户端的操作系统以及IP地址

| 主机名称    | 操作系统 | IP地址        |
| ----------- | -------- | ------------- |
| iSCSI服务端 | RHEL 8   | 192.168.10.10 |
| iSCSI客户端 | RHEL 8   | 192.168.10.20 |



**第1步**：在RHEL 8 / [CentOS](https://www.linuxprobe.com/) 8系统中iSCSI服务端程序已经默认被安装，用户需要做的是配置好软件仓库后安装iSCSI服务端的交换式配置工具，通过交互式的配置过程来完成对参数的设定，相比于直接修改配置文件则又方便又安全。通过在dnf命令的后面添加-y参数，在安装过程中就不需要再进行手动确认了：

```
[root@linuxprobe ~]# dnf install -y targetcli
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:23:54 ago on Tue 27 Apr 2021 08:12:59 AM CST.
Dependencies resolved.
=============================================================================================
 Package                     Arch           Version                  Repository         Size
=============================================================================================
Installing:
 targetcli                   noarch         2.1.fb49-1.el8           AppStream          73 k
Installing dependencies:
 python3-configshell         noarch         1:1.1.fb25-1.el8         BaseOS             74 k
 python3-kmod                x86_64         0.9-20.el8               BaseOS             90 k
 python3-pyparsing           noarch         2.1.10-7.el8             BaseOS            142 k
 python3-rtslib              noarch         2.1.fb69-3.el8           BaseOS            100 k
 python3-urwid               x86_64         1.3.1-4.el8              BaseOS            783 k
 target-restore              noarch         2.1.fb69-3.el8           BaseOS             23 k

Transaction Summary
=============================================================================================
Install  7 Packages
………………省略部分输出信息………………
Installed products updated.

Installed:
  targetcli-2.1.fb49-1.el8.noarch           python3-configshell-1:1.1.fb25-1.el8.noarch     
  python3-kmod-0.9-20.el8.x86_64            python3-pyparsing-2.1.10-7.el8.noarch           
  python3-rtslib-2.1.fb69-3.el8.noarch      python3-urwid-1.3.1-4.el8.x86_64                
  target-restore-2.1.fb69-3.el8.noarch     

Complete!
```

iSCSI是跨平台的协议，因此用户也可以在Windows系统下搭建iSCSI Target再共享给Linux系统主机。不过根据[刘遄](https://www.linuxprobe.com/)老师过往的经验，类似于DataCorc Software的SANmelody或是FalconStor Software的iSCSI Server for Windows等等软件，Windows系统使用都是要收费的呦。

**第2步**：配置iSCSI服务端共享资源。targetcli是用于管理iSCSI服务端存储资源的专用配置命令，它能够提供类似于fdisk命令的交互式配置功能，将iSCSI共享资源的配置内容抽象成“目录”的形式，我们只需将各类配置信息填入到相应的“目录”中即可。这里的难点主要在于认识每个“参数目录”的作用。当把配置参数正确地填写到“目录”中后，iSCSI服务端也可以提供共享资源服务了。

在执行targetcli命令后就能看到交互式的配置界面了。在该界面中允许使用很多Linux命令，比如利用ls查看目录参数的结构，使用cd切换到不同的目录中。

```
[root@linuxprobe ~]# targetcli
targetcli shell version 2.1.fb49
Copyright 2011-2013 by Datera, Inc and others.
For help on commands, type 'help'.

/> ls
o- / ............................................................................ [...]
  o- backstores ................................................................. [...]
  | o- block ..................................................... [Storage Objects: 0]
  | o- fileio .................................................... [Storage Objects: 0]
  | o- pscsi ......................................................[Storage Objects: 0]
  | o- ramdisk ................................................... [Storage Objects: 0]
  o- iscsi ................................................................[Targets: 0]
  o- loopback ............................................................ [Targets: 0]
/> 
```

/backstores/block是iSCSI服务端配置共享设备的位置。我们需要把刚刚创建的RAID 5磁盘阵列md0文件加入到配置共享设备的“资源池”中，并将该文件重新命名为disk0，这样用户就不会知道是由服务器中的哪块硬盘来提供共享存储资源，而只会看到一个名为disk0的存储设备。

```
/> cd /backstores/block 
/backstores/block> create disk0 /dev/md0
Created block storage object disk0 using /dev/md0.
/backstores/block> cd /
/> ls
o- / ............................................................................ [...]
  o- backstores ................................................................. [...]
  | o- block ..................................................... [Storage Objects: 1]
  | | o- disk0 ............................ [/dev/md0 (40.0GiB) write-thru deactivated]
  | |   o- alua ...................................................... [ALUA Groups: 1]
  | |     o- default_tg_pt_gp ...........................[ALUA state: Active/optimized]
  | o- fileio .................................................... [Storage Objects: 0]
  | o- pscsi ..................................................... [Storage Objects: 0]
  | o- ramdisk ................................................... [Storage Objects: 0]
  o- iscsi ............................................................... [Targets: 0]
  o- loopback ............................................................ [Targets: 0]
/> 
```

**第3步**：创建iSCSI target名称及配置共享资源。iSCSI target名称是由系统自动生成的，这是一串用于描述共享资源的唯一字符串。稍后用户在扫描iSCSI服务端时即可看到这个字符串，因此我们不需要记住它。

```
/iscsi> create
Created target iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5.
Created TPG 1.
Global pref auto_add_default_portal=true
Created default portal listening on all IPs (0.0.0.0), port 3260.
/iscsi> ls
o- iscsi ................................................................... [Targets: 1]
  o- iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5 ............. [TPGs: 1]
    o- tpg1 ...................................................... [no-gen-acls, no-auth]
      o- acls ................................................................. [ACLs: 0]
      o- luns ................................................................. [LUNs: 0]
      o- portals ........................................................... [Portals: 1]
        o- 0.0.0.0:3260 ............................................................ [OK]
```

### **Tips**

请注意iSCSI自动生成的名称中，最后一个.为句号，不是名称中的一部分。

系统在生成这个target名称后，还会在/iscsi参数目录中创建一个与其字符串同名的新“目录”用来存放共享资源。我们需要把前面加入到iSCSI共享资源池中的硬盘设备添加到这个新目录中，这样用户在登录iSCSI服务端后，即可默认使用这硬盘设备提供的共享存储资源了。

```
/iscsi> cd iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5/
/iscsi/iqn.20....745b21d6cad5> cd tpg1/luns 
/iscsi/iqn.20...ad5/tpg1/luns> create /backstores/block/disk0 
Created LUN 0.
```

**第4步**：设置访问控制列表（ACL）。iSCSI协议是通过客户端名称进行验证的，也就是说，用户在访问存储共享资源时不需要输入密码，只要iSCSI客户端的名称与服务端中设置的访问控制列表中某一名称条目一致即可，因此需要在iSCSI服务端的配置文件中写入一串能够验证用户信息的名称。acls参数目录用于存放能够访问iSCSI服务端共享存储资源的客户端名称。推荐在刚刚系统生成的iSCSI target后面追加上类似于:client的参数，这样既能保证客户端的名称具有唯一性，又非常便于管理和阅读：

```
/iscsi/iqn.20...ad5/tpg1/luns> cd ..
/iscsi/iqn.20...21d6cad5/tpg1> cd acls 
/iscsi/iqn.20...ad5/tpg1/acls> create iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5:client
Created Node ACL for iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5:client
Created mapped LUN 0.
```

**第5步**：设置iSCSI服务端的监听IP地址和端口号。位于生产环境中的服务器上可能有多块网卡，那么到底是由哪个网卡或IP地址对外提供共享存储资源呢？配置文件中默认是允许所有网卡提供iSCSI服务，如果您认为这有些许不安全，可以手动删除：

```
/iscsi/iqn.20...ad5/tpg1/acls> cd ../portals//iscsi/iqn.20.../tpg1/portals> lso- portals ................................................................... [Portals: 1]  o- 0.0.0.0:3260 .................................................................... [OK]/iscsi/iqn.20.../tpg1/portals> delete 0.0.0.0 3260Deleted network portal 0.0.0.0:3260
```

接下来设定由系统使用服务器IP地址192.168.10.10的3260端口向外提供iSCSI共享存储资源服务：

```
/iscsi/iqn.20.../tpg1/portals> create 192.168.10.10Using default IP port 3260Created network portal 192.168.10.10:3260.
```

**第6步**：在参数文件配置妥当后，浏览刚刚配置的信息，确保上述提到的“目录”都已经填写正确的内容。在确认信息无误后输入exit命令来退出配置。注意，千万不要习惯性地按Ctrl + C组合键结束进程，这样不会保存配置文件，我们的工作也就白费了。

```
/iscsi/iqn.20.../tpg1/portals> cd //> lso- / ............................................................................... [...]  o- backstores .................................................................... [...]  | o- block ........................................................ [Storage Objects: 1]  | | o- disk0 ................................. [/dev/md0 (40.0GiB) write-thru activated]  | |   o- alua ......................................................... [ALUA Groups: 1]  | |     o- default_tg_pt_gp ............................. [ALUA state: Active/optimized]  | o- fileio ....................................................... [Storage Objects: 0]  | o- pscsi ........................................................ [Storage Objects: 0]  | o- ramdisk ...................................................... [Storage Objects: 0]  o- iscsi .................................................................. [Targets: 1]  | o- iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5 ............ [TPGs: 1]  |   o- tpg1 ..................................................... [no-gen-acls, no-auth]  |     o- acls ................................................................ [ACLs: 1]  |     | o- iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5:client[Mapped LUNs: 1]  |     |   o- mapped_lun0 ....................................... [lun0 block/disk0 (rw)]  |     o- luns ................................................................ [LUNs: 1]  |     | o- lun0 ............................ [block/disk0 (/dev/md0) (default_tg_pt_gp)]  |     o- portals .......................................................... [Portals: 1]  |       o- 192.168.10.10:3260 ..................................................... [OK]  o- loopback ............................................................... [Targets: 0]/> exitGlobal pref auto_save_on_exit=trueConfiguration saved to /etc/target/saveconfig.json
```

清空iptables防火墙中默认策略，设置firewalld防火墙放行iSCSI服务或3260/tcp端口号：

```
[root@linuxprobe ~]# iptables -F[root@linuxprobe ~]# iptables-save[root@linuxprobe ~]# firewall-cmd --permanent --add-port=3260/tcp success[root@linuxprobe ~]# firewall-cmd --reload success
```

##### **17.4 配置Linux客户端**

我们在前面的章节中已经配置了很多Linux服务，基本上可以说，无论是什么服务，客户端的配置步骤都要比服务端的配置步骤简单一些。在RHEL 8系统中，已经默认安装了iSCSI客户端服务程序initiator。如果您的系统没有安装的话，可以使用软件仓库手动安装。

```
[root@linuxprobe ~]# dnf install iscsi-initiator-utilsUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:00:04 ago on Tue 27 Apr 2021 01:34:47 AM CST.Package iscsi-initiator-utils-6.2.0.876-7.gitf3c8e90.el8.x86_64 is already installed.Dependencies resolved.Nothing to do.Complete!
```

前面讲到，iSCSI协议是通过客户端的名称来进行验证，而该名称也是iSCSI客户端的唯一标识，而且必须与服务端配置文件中访问控制列表中的信息一致，否则客户端在尝试访问存储共享设备时，系统会弹出验证失败的保存信息。

下面编辑iSCSI客户端中的initiator名称文件，把服务端的访问控制列表名称填写进来，然后重启客户端iscsid服务程序并将其加入到开机启动项中：

```
[root@linuxprobe ~]# vim /etc/iscsi/initiatorname.iscsi InitiatorName=iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5:client[root@linuxprobe ~]# systemctl restart iscsid[root@linuxprobe ~]# systemctl enable  iscsidCreated symlink /etc/systemd/system/multi-user.target.wants/iscsid.service → /usr/lib/systemd/system/iscsid.service.
```

iSCSI客户端访问并使用共享存储资源的步骤很简单，只需要记住[刘遄](https://www.linuxprobe.com/)老师的一个小口诀“先发现，再登录，最后挂载并使用”。iscsiadm是用于管理、查询、插入、更新或删除iSCSI数据库配置文件的命令行工具，用户需要先使用这个工具扫描发现远程iSCSI服务端，然后查看找到的服务端上有哪些可用的共享存储资源。其中，“-m discovery”参数的目的是扫描并发现可用的存储资源，“-t st”参数为执行扫描操作的类型，“-p 192.168.10.10”参数为iSCSI服务端的IP地址：

```
[root@linuxprobe ~]# iscsiadm -m discovery -t st -p 192.168.10.10192.168.10.10:3260,1 iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5
```

在使用iscsiadm命令发现了远程服务器上可用的存储资源后，接下来准备登录iSCSI服务端。其中，“-m node”参数为将客户端所在主机作为一台节点服务器，“-T”参数为要使用的存储资源（大家可以直接复制前面命令中扫描发现的结果，以免录入错误），“-p 192.168.10.10”参数依然为对方iSCSI服务端的IP地址。最后使用“--login”或“-l”参数进行登录验证。

```
[root@linuxprobe ~]# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5 -p 192.168.10.10 --loginLogging in to [iface: default, target: iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5, portal: 192.168.10.10,3260] (multiple)Login to [iface: default, target: iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5, portal: 192.168.10.10,3260] successful.
```

在iSCSI客户端成功登录之后，会在客户端主机上多出一块名为/dev/sdb的设备文件。第6章曾经讲过，udev服务在命名硬盘名称时，与硬盘插槽是没有关系的。接下来便能够像使用本地主机上的硬盘那样来操作这个设备文件了。

```
[root@linuxprobe ~]# ls -l /dev/sdbbrw-rw----. 1 root disk 8, 16 Apr 27 01:43 /dev/sdb[root@linuxprobe ~]# file /dev/sdb /dev/sdb: block special (8/16)
```

下面进入标准的磁盘操作流程。考虑到大家已经在第6章学习了这部分内容，外加这个设备文件本身只有40GB的容量，因此不必进行分区，而是直接格式化并挂载使用。

```
[root@linuxprobe ~]# mkfs.xfs /dev/sdbmeta-data=/dev/sdb               isize=512    agcount=16, agsize=654720 blks         =                       sectsz=512   attr=2, projid32bit=1         =                       crc=1        finobt=1, sparse=1, rmapbt=0         =                       reflink=1data     =                       bsize=4096   blocks=10475520, imaxpct=25         =                       sunit=128    swidth=256 blksnaming   =version 2              bsize=4096   ascii-ci=0, ftype=1log      =internal log           bsize=4096   blocks=5120, version=2         =                       sectsz=512   sunit=0 blks, lazy-count=1realtime =none                   extsz=4096   blocks=0, rtextents=0[root@linuxprobe ~]# mkdir /iscsi[root@linuxprobe ~]# mount /dev/sdb /iscsi/
```

不放心的话，再用df命令查看下挂载情况：

```
[root@linuxprobe ~]# df -hFilesystem             Size  Used Avail Use% Mounted ondevtmpfs               969M     0  969M   0% /devtmpfs                  984M     0  984M   0% /dev/shmtmpfs                  984M  9.6M  974M   1% /runtmpfs                  984M     0  984M   0% /sys/fs/cgroup/dev/mapper/rhel-root   17G  3.9G   14G  23% //dev/sr0               6.7G  6.7G     0 100% /media/cdrom/dev/sda1             1014M  152M  863M  15% /boottmpfs                  197M   16K  197M   1% /run/user/42tmpfs                  197M  3.5M  194M   2% /run/user/0/dev/sdb                40G  319M   40G   1% /iscsi
```

从此以后，这个设备文件就如同是客户端本机主机上的硬盘那样工作。需要提醒大家的是，由于udev服务是按照系统识别硬盘设备的顺序来命名硬盘设备的，当客户端主机同时使用多个远程存储资源时，如果下一次识别远程设备的顺序发生了变化，则客户端挂载目录中的文件也将随之混乱。为了防止发生这样的问题，我们应该在/etc/fstab配置文件中使用设备的UUID唯一标识符进行挂载，这样，不论远程设备资源的识别顺序再怎么变化，系统也能正确找到设备所对应的目录。

blkid命令用于查看设备的名称、文件系统及UUID。可以使用管道符（详见第3章）进行过滤，只显示与/dev/sdb设备相关的信息：

```
[root@linuxprobe ~]# blkid | grep /dev/sdb/dev/sdb: UUID="0937b4ec-bada-4a09-9f99-9113296ab72d" TYPE="xfs"
```

刘遄老师还要再啰嗦一句，由于/dev/sdb是一块网络存储设备，而iSCSI协议是基于TCP/IP网络传输数据的，因此必须在/etc/fstab配置文件中添加上“_netdev”参数，表示当系统联网后再进行挂载操作，以免系统开机时间过长或开机失败：

```
[root@linuxprobe ~]# vim /etc/fstab## /etc/fstab# Created by anaconda on Thu Feb 25 10:42:11 2021## Accessible filesystems, by reference, are maintained under '/dev/disk/'.# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.## After editing this file, run 'systemctl daemon-reload' to update systemd# units generated from this file.#/dev/mapper/rhel-root                       /             xfs      defaults   0 0UUID=37d0bdc6-d70d-4cc0-b356-51195ad90369   /boot         xfs      defaults   0 0/dev/mapper/rhel-swap                       swap          swap     defaults   0 0/dev/cdrom                                  /media/cdrom  iso9660  defaults   0 0 UUID="0937b4ec-bada-4a09-9f99-9113296ab72d" /iscsi        xfs      defaults   0 0 
```

如果我们不再需要使用iSCSI共享设备资源了，可以用iscsiadm命令的“-u”参数将其设备卸载：

```
[root@linuxprobe ~]# iscsiadm -m node -T iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5 -uLogging out of session [sid: 1, target: iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5, portal: 192.168.10.10,3260]Logout of [sid: 1, target: iqn.2003-01.org.linux-iscsi.linuxprobe.x8664:sn.745b21d6cad5, portal: 192.168.10.10,3260] successful.
```

这种获取iSCSI远程存储的方法依赖的是RHEL 8系统自带的iSCSI Initiator软件程序，由Initiator软件将以太网卡虚拟成了iSCSI卡，进而接受数据报文，实现主机与iSCSI存储设备之间基于TCP/IP协议的传输功能。这种方式仅需主机与网络即可实现，因此成本是最低的，但是iSCSI和TCP/IP协议报文需要消耗客户端自身的CPU计算性能，还是有一定的额外开销，一般是建议在低I/O或低带宽性能要求的环境中使用软件模拟方式。

如果同学们在后续的生产环境中需要进行大量的远程数据存储，建议自行配备iSCSI HBA硬件卡设备，HBA全称即Host Bus Adapter。安装到iSCSI服务器上，从而实现iSCSI服务器与交换机之间、iSCSI服务器与客户端之间的高效数据传输。与Initiator软件模拟相比，iSCSI HBA硬件卡设备不需要消耗CPU计算性能，专用的设备对于iSCSI的支持也会更好。但价格会稍微贵一些，需要读者在性能和成本之间进行权衡。

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **17.5 配置Windows客户端**

使用Windows系统的客户端也可以正常访问iSCSI服务器上的共享存储资源，而且操作原理及步骤与Linux系统的客户端基本相同。在进行下面的实验之前，请先关闭Linux系统客户端，以免这两台客户端主机同时使用iSCSI共享存储资源而产生潜在问题。下面按照表17-2来配置iSCSI服务器和Windows客户端所用的IP地址。

表17-2  iSCSI服务器和客户端的操作系统以及IP地址

| 主机名称          | 操作系统   | IP地址        |
| ----------------- | ---------- | ------------- |
| iSCSI服务端       | RHEL 8     | 192.168.10.10 |
| Windows系统客户端 | Windows 10 | 192.168.10.30 |



**第1步**：运行iSCSI发起程序。在Windows 10操作系统中已经默认安装了iSCSI客户端程序，我们只需在控制面板中找到“系统和安全”标签，然后单击“管理工具”（见图17-3），进入到“管理工具”页面后即可看到“iSCSI发起程序”图标。双击该图标。在第一次运行iSCSI发起程序时，系统会提示“Microsoft iSCSI服务端未运行”，单击“是”按钮即可自动启动并运行iSCSI发起程序，如图17-4所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/7-3-1.png)

图17-3 在控制面板中单击“管理工具”

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/7-4.png)

图17-4 双击“iSCSI发起程序”图标

**第2步**：扫描发现iSCSI服务端上可用的存储资源。不论是Windows系统还是Linux系统，要想使用iSCSI共享存储资源都必须先进行扫描发现操作。运行iSCSI发起程序后在“目标”选项卡的“目标”文本框中写入iSCSI服务端的IP地址，然后单击“快速连接”按钮，如图17-5所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-5.png)

图17-5 填写iSCSI服务端的IP地址

在弹出的“快速连接”提示框中可看到共享的硬盘存储资源，此时显示无法登录到目标属于正常情况，单击“完成”按钮即可，如图17-6所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-6.png)

图17-6 在“快速连接”提示框中看到的共享的硬盘存储资源

回到“目标”选项卡页面，可以看到共享存储资源的名称已经出现，如图17-7所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-7.png)

图17-7 在“目标”选项卡中看到了共享存储资源

**第3步**：准备连接iSCSI服务端的共享存储资源。由于在iSCSI服务端程序上设置了ACL，使得只有客户端名称与ACL策略中的名称保持一致时才能使用远程存储资源，因此首先需要在“配置”选项卡中单击“更改”按钮（见图17-8），随后在修改界面写入iSCSI服务器ACL策略名称（见图17-9），最后重新返回到iSCSI发起程序的“目标”界面（见图17-10）。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-8.png)

图17-8 更改客户端的发起程序名称

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-8-%E5%A2%9E%E5%8A%A0.png)

图17-9 修改iSCSI发起程序的名称

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-9.png)

图17-10 返回到“目标”界面

在确认iSCSI发起程序名称与iSCSI服务器ACL策略一致后，重新点击“连接”按钮，如图17-11所示，并点击确认。大约1~3秒后，状态会更新为“已连接”，如图17-12所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-10.png)

图17-11 尝试连接iSCSI存储目标
![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-11.png)

图17-12 成功连接到远程共享存储资源

**第4步**：访问iSCSI远程共享存储资源。右键单击桌面上的“计算机”图标，打开计算机管理程序，如图17-13所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-12.png)

图17-13 计算机管理程序的界面

开始对磁盘进行初始化操作，如图17-4所示。Windows系统用来初始化磁盘设备的步骤十分简单，各位读者都可以玩得转Linux系统，相信Windows系统就更不在话下了。Windows系统的初始化过程步骤如图17-15至图17-21所示。

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-13.png)

图17-14 对磁盘设备进行初始化操作

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-14.png)

图17-15 开始使用“新建简单卷向导”

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-15.png)

图17-16 对磁盘设备进行分区操作

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-16.png)

图17-17 设置系统中显示的盘符

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-17.png)

图17-18 设置磁盘设备的格式以及卷标

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-18.png)

图17-19  检查磁盘初始化信息是否正确

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-19.png)

图17-20  等待磁盘设备初始化过程结束

![第17章 使用iSCSI服务部署网络存储第17章 使用iSCSI服务部署网络存储](https://www.linuxprobe.com/wp-content/uploads/2021/04/17-20.png)

图17-21 磁盘初始化完毕后弹出设备图标

接下来在正常的使用过程中，不知情的用户可能都察觉不到这是一块远程存储设备呢，整个的传输过程都是完全透明的，像是一块本地硬盘一样的稳定。不过，这只是理论状态，实际上的iSCSI数据传输速率并不能完全达到本地硬盘的性能，或多或少的受到网络带宽的影响，但差别并不明显。更何况iSCSI网络存储模式的还有一个优势是安全性高，这对于数据集中存储来讲显得十分重要，值得一试！~

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．简述iSCSI存储技术在生产环境中的作用。

**答：**iSCSI存储技术通过把硬件存储设备与TCP/IP网络协议相互结合，使得用户可以通过互联网方便的访问远程机房提供的共享存储资源。

2．在Linux系统中，iSCSI服务端和iSCSI客户端所使用的服务程序分别叫什么？

**答：**iSCSI服务端程序为targetd，iSCSI客户端程序为initiator。

3．在使用targetcli命令配置iSCSI服务端配置文件时，acls与portals参数目录中分别存放什么内容？

**答：**acls参数目录用于存放能够访问iSCSI服务端共享存储资源的客户端名称，portals参数目录用于定义由服务器的哪个IP地址对外提供共享存储资源服务。

4．iSCSI协议占用了服务器哪个协议和端口号？

**答：**iSCSI协议占用了服务器TCP协议的3260端口号。

5．用户在填写fstab设备挂载配置文件时，一般会把远程存储资源的UUID（而非设备的名称）填写到配置文件中。这是为什么？

**答：**在Linux系统中，设备名称是由udev服务进行管理的，而udev服务的设备命名规则是由设备类型及系统识别顺序等信息共同组成的。考虑到网络存储设备具有识别顺序不稳定的特点，所以为了避免识别顺序混乱造成的挂载错误问题，故使用UUID进行挂载操作。

6．在使用Windows系统来访问iSCSI共享存储资源时，它有两个步骤与Linux系统一样。请说明是哪两个步骤。

**答：**扫描并发现服务端上可用的iSCSI共享存储资源；验证登录。

# [第18章 使用MariaDB数据库管理系统](https://www.linuxprobe.com/basic-learning-18.html)

**章节简述：**

MySQL数据库项目自从被Oracle公司收购之后，从开源软件转变成为了“闭源”软件，这导致IT行业中的很多企业以及厂商纷纷选择使用了数据库软件的后起之秀—MariaDB数据库管理系统。MariaDB数据库管理系统也因此快速占据了市场。

本章将介绍数据库以及数据库管理系统的理论知识，然后再介绍MariaDB数据库管理系统的内容，最后将通过动手实验的方式，帮助各位读者掌握MariaDB数据库管理系统的一些常规操作。比如，账户的创建与管理、账户权限的授权；新建数据库、新建数据库表单；对数据库执行新建、删除、修改和查询等操作。最后还介绍了数据库的备份与恢复方法，不仅能做到“增删改查”，更能胜任生产环境中的数据库管理工作。

本章目录结构

- [18.1 数据库管理系统](https://www.linuxprobe.com/basic-learning-18.html#181)
- [18.2 初始化mariaDB服务](https://www.linuxprobe.com/basic-learning-18.html#182_mariaDB)
- [18.3 管理用户以及授权](https://www.linuxprobe.com/basic-learning-18.html#183)
- [18.4 创建数据库与表单](https://www.linuxprobe.com/basic-learning-18.html#184)
- [18.5 管理表单及数据](https://www.linuxprobe.com/basic-learning-18.html#185)
- [18.6 数据库的备份及恢复](https://www.linuxprobe.com/basic-learning-18.html#186)

##### **18.1 数据库管理系统**

数据库是指按照某些特定结构来存储数据资料的数据仓库。在当今这个大数据技术迅速崛起的年代，互联网上每天都会生成海量的数据信息，数据库技术也从最初只能存储简单的表格数据的单一集中存储模式，发展到了现如今存储海量数据的大型分布式模式。在信息化社会中，能够充分有效地管理和利用各种数据，挖掘其中的价值，是进行科学研究与决策管理的重要前提。同时，数据库技术也是管理信息系统、办公自动化系统、决策支持系统等各类信息系统的核心组成部分，是进行科学研究和决策管理的重要技术手段。

数据库管理系统是一种能够对数据库中存放的数据进行建立、修改、删除、查找、维护等操作的软件程序。它通过把计算机中具体的物理数据转换成适合用户理解的抽象逻辑数据，有效地降低数据库管理的技术门槛，因此即便是从事[Linux](https://www.linuxprobe.com/)运维工作的工程师也可以对数据库进行基本的管理操作。但是，[刘遄](https://www.linuxprobe.com/)老师有必要提醒各位读者，本书的技术主线依然是[Linux系统](https://www.linuxprobe.com/)的运维，而数据库管理系统只不过是在此主线上的一个内容不断横向扩展、纵向加深的分支，不能指望在一两天之内就可以精通数据库管理技术。如果有读者在学完本章内容之后对数据库管理技术产生了浓厚兴趣，并希望谋得一份相关的工作，那么就需要额外为自己定制一个学习规划了。

![第18章 使用MariaDB数据库管理系统第18章 使用MariaDB数据库管理系统](https://www.linuxprobe.com/wp-content/uploads/2015/10/marialANDmysql.png)

图18-1 MariaDB与Mysql数据库管理系统著名LOGO

这种凤凰涅槃般的浴火重生，是由MySQL项目创始人Michael Widenius带领着团队完成的。根据MariaDB官网的介绍，他有两个可爱的小天使女儿，大女儿叫做My，而二女儿叫做Maria，因此这也是第二款他用亲人名字命名的软件。

```
Why is the Software Called MariaDB?

The 'MySQL' name is trademarked by Oracle, and they have chosen to keep that trademark to themselves. The name MySQL (just like the MyISAM storage engine) comes from Monty's first daughter My. The first part of 'MySQL' is pronounced like the English adjective, even if this doesn't match the correct pronunciation of the Finnish name.

MariaDB continues this tradition by being named after his younger daughter, Maria.

The name Maria was initially given to a storage engine. After MariaDB was started, to avoid confusion, it was renamed to Aria. The new name was decided as a result of a contest.
```

既然是讲解数据库管理技术，就肯定绕不开MySQL。MySQL是一款市场占有率非常高的数据库管理系统，技术成熟、配置步骤相对简单，而且具有良好的可扩展性。但是，由于Oracle公司在2009年收购了MySQL的母公司Sun，因此MySQL数据库项目也随之纳入Oracle麾下，逐步演变为保持着开源软件的身份，但又申请了多项商业专利的软件系统。开源软件是全球黑客、极客、程序员等技术高手在开源社区的大旗下的公共智慧结晶，自己的劳动成果被其他公司商业化自然也伤了一大批开源工作者的心，因此由MySQL项目创始者重新研发了一款名为MariaDB的全新数据库管理系统。

该软件当前由开源社区进行维护，是MySQL的分支产品，而且几乎完全兼容MySQL。MariaDB与MySQL具有高度的兼容性，具有库二进制奇偶校验的直接替换功能，与Mysql API和[命令](https://www.linuxcool.com/)均保持一致。并且MariaDB还自带了一个新的存储引擎Aria，用来替代MyISAM，与MySQL一样好用。

与此同时，由于各大公司之间存在着竞争关系或利益关系，外加MySQL在被收购之后逐渐由开源向闭源软件转变，很多公司抛弃了MySQL。当前，谷歌、维基百科等技术领域决定将MySQL数据库上的业务转移到MariaDB数据库，Linux开源系统的领袖[红帽](https://www.linuxprobe.com/)公司也决定在RHEL 8、[CentOS](https://www.linuxprobe.com/) 8以及最新的Fedora系统中，将MariaDB作为默认的数据库管理系统，而且[红帽](https://www.linuxprobe.com/)公司更是将数据库知识加入到了[RHCE](https://www.linuxprobe.com/)认证的考试内容中。随后，还有数十个常见的Linux系统（如openSUSE、Slackware等）也作出了同样的表态。

但是，坦白来讲，虽然IT行业巨头都决定采用MariaDB数据库管系统，这并不意味着MariaDB较之于MySQL有明显的优势。刘遄老师用了近两周的时间测试了MariaDB与MySQL的区别，并进行了多项性能测试，并没有发现媒体所说的那种明显的优势。可以说，MariaDB和MySQL在性能上基本保持一致，两者的操作命令也十分相似。从务实的角度来讲，在掌握了MariaDB数据库的命令和基本操作之后，在今后的工作中即使遇到MySQL数据库，也可以快速上手。

##### **18.2 初始化mariaDB服务**

相较于MySQL，MariaDB数据库管理系统有了很多新鲜的扩展特性，例如对微秒级别的支持、线程池、子查询优化、进程报告等。在配置妥当软件仓库后，即可安装部署MariaDB数据库主程序及服务端程序了。

```
[root@linuxprobe ~]# dnf install -y mariadb mariadb-server
Updating Subscription Management repositories.
Unable to read consumer identity
This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.
Last metadata expiration check: 0:00:19 ago on Tue 27 Apr 2021 05:04:27 PM CST.
Dependencies resolved.
===========================================================================================
 Package                    Arch   Version              Repository             Size
===========================================================================================
Installing:
 mariadb                    x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream 6.2 M
 mariadb-server             x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream  16 M
Installing dependencies:
 mariadb-common             x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream  62 k
 mariadb-connector-c        x86_64 3.0.7-1.el8                             AppStream 148 k
 mariadb-connector-c-config noarch 3.0.7-1.el8                             AppStream  13 k
 mariadb-errmsg             x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream 232 k
 perl-DBD-MySQL             x86_64 4.046-2.module+el8+2515+0650e81c        AppStream 156 k
Installing weak dependencies:
 mariadb-backup             x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream 6.2 M
 mariadb-gssapi-server      x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream  49 k
 mariadb-server-utils       x86_64 3:10.3.11-1.module+el8+2765+cfa4f87b    AppStream 1.6 M
Enabling module streams:
 mariadb                           10.3                                                   
 perl-DBD-MySQL                    4.046                                                  

Transaction Summary
===========================================================================================
Install  10 Packages
………………省略部分输出信息………………
Installed:
  mariadb-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                                      
  mariadb-server-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                               
  mariadb-backup-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                               
  mariadb-gssapi-server-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                        
  mariadb-server-utils-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                         
  mariadb-common-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                               
  mariadb-connector-c-3.0.7-1.el8.x86_64                                                   
  mariadb-connector-c-config-3.0.7-1.el8.noarch                                            
  mariadb-errmsg-3:10.3.11-1.module+el8+2765+cfa4f87b.x86_64                               
  perl-DBD-MySQL-4.046-2.module+el8+2515+0650e81c.x86_64                                   

Complete!
```

在安装完毕后，记得启动服务程序，并将其加入到开机启动项中：

```
[root@linuxprobe ~]# systemctl start  mariadb 
[root@linuxprobe ~]# systemctl enable mariadb 
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
```

在确认MariaDB数据库软件程序安装完毕并成功启动后请不要立即使用。为了确保数据库的安全性和正常运转，需要先对数据库程序进行初始化操作。这个初始化操作涉及下面5个步骤。

> 1：设置root管理员在数据库中的密码值；
>
> （注意，该密码并非root管理员在系统中的密码，这里的密码值默认应该为空，可直接按回车键）
>
> 2：设置root管理员在数据库中的专有密码；
>
> 3：删除匿名账户，并使用root管理员从远程登录数据库，以确保数据库上运行的业务的安全性；
>
> 4：删除默认的测试数据库，取消测试数据库的一系列访问权限；
>
> 5：刷新授权列表，让初始化的设定立即生效。

对于上述数据库初始化的操作步骤，刘遄老师已经在下面的输出信息旁边进行了简单注释，确保各位读者更直观地了解要输入的内容：

```
[root@linuxprobe ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 输入管理员原始密码，默认为空值，直接回车即可
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y（设置管理员密码）
New password: 输入新的密码
Re-enter new password: 再次输入密码
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y（删除匿名账户）
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y（禁止管理员从远程登录）
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y（删除测试数据库及其访问权限）
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y（刷新授权表，让初始化后的设定立即生效）
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

在很多生产环境中都需要使用站库分离的技术（即网站和数据库不在同一个服务器上），如果需要让root管理员远程访问数据库，可在上面的初始化操作中设置策略，以允许root管理员从远程访问。然后还需要设置防火墙，使其放行对数据库服务程序的访问请求，数据库服务程序默认会占用3306端口，在防火墙策略中服务名称统一叫作mysql：

```
[root@linuxprobe ~]# firewall-cmd --permanent --add-service=mysql
success
[root@linuxprobe ~]# firewall-cmd --reload
success
```

一切准备就绪。现在我们将首次登录MariaDB数据库。管理数据库的命令为mysql，其中，“-u”参数用来指定以root管理员的身份登录，而“-p”参数用来验证该用户在数据库中的密码值。

```
[root@linuxprobe ~]# mysql -u root -p
Enter password: 输入刚才设置的管理员密码后敲击回车
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.3.11-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

初次使用数据库管理工具的读者，可以输入help命令查看MariaDB能做的操作，语句的用法和MySQL一摸一样：

```
MariaDB [(none)]> help

General information about MariaDB can be found at http://mariadb.org

List of all MySQL commands:
Note that all text commands must be first on line and end with ';'
?         (\?) Synonym for `help'.
clear     (\c) Clear the current input statement.
connect   (\r) Reconnect to the server. Optional arguments are db and host.
delimiter (\d) Set statement delimiter.
edit      (\e) Edit command with $EDITOR.
ego       (\G) Send command to mysql server, display result vertically.
exit      (\q) Exit mysql. Same as quit.
go        (\g) Send command to mysql server.
help      (\h) Display this help.
nopager   (\n) Disable pager, print to stdout.
notee     (\t) Don't write into outfile.
pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.
print     (\p) Print current command.
prompt    (\R) Change your mysql prompt.
quit      (\q) Quit mysql.
rehash    (\#) Rebuild completion hash.
source    (\.) Execute an SQL script file. Takes a file name as an argument.
status    (\s) Get status information from the server.
system    (\!) Execute a system shell command.
tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.
use       (\u) Use another database. Takes database name as argument.
charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.
warnings  (\W) Show warnings after every statement.
nowarning (\w) Don't show warnings after every statement.

For server side help, type 'help contents'
```

在登录MariaDB数据库后执行数据库命令时，都需要在命令后面用分号（;）结尾，这也是与Linux命令最显著的区别。大家需要慢慢习惯数据库命令的这种设定。下面执行如下命令查看数据库管理系统中当前都有哪些数据库：

```
MariaDB [(none)]> SHOW databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.000 sec)
```

小试牛刀过后，接下来使用数据库命令将root管理员在数据库管理系统中的密码值修改为linuxprobe。这样退出后再尝试登录，如果还坚持输入原先的密码，则将提示访问失败。

```
MariaDB [(none)]> SET password = PASSWORD('linuxprobe');Query OK, 0 rows affected (0.001 sec)MariaDB [(none)]> exitBye[root@linuxprobe ~]# mysql -u root -pEnter password: 此处输入管理员在数据库中的旧密码ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
```

而输入新密码（linuxprobe）后，便可顺利进入到数据库管理工具中：

```
[root@linuxprobe ~]# mysql -u root -pEnter password: 此处输入管理员在数据库中的新密码Welcome to the MariaDB monitor.  Commands end with ; or \g.Your MariaDB connection id is 20Server version: 10.3.11-MariaDB MariaDB ServerCopyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

##### **18.3 管理用户以及授权**

在生产环境中总不能一直“死啃”root管理员。为了保障数据库系统的安全性，以及让其他用户协同管理数据库，我们可以在MariaDB数据库管理系统中为他们创建多个专用的数据库管理账户，然后再分配合理的权限，以满足他们的工作需求。为此，可使用root管理员登录数据库管理系统，然后按照“CREATE USER 用户名@主机名 IDENTIFIED BY '密码'; ”的格式创建数据库管理账户。再次提醒大家，一定不要忘记每条数据库命令后面的分号（;）。

```
MariaDB [(none)]> CREATE USER luke@localhost IDENTIFIED BY 'linuxprobe';Query OK, 0 rows affected (0.00 sec)
```

创建的账户信息可以使用select命令语句来查询。下面命令查询的是账户luke的主机名称、账户名称以及经过加密的密码值信息：

```
MariaDB [(none)]> use mysql;Database changedMariaDB [mysql]> SELECT HOST,USER,PASSWORD FROM user WHERE USER="luke";+-----------+------+-------------------------------------------+| HOST      | USER | PASSWORD                                  |+-----------+------+-------------------------------------------+| localhost | luke | *55D9962586BE75F4B7D421E6655973DB07D6869F |+-----------+------+-------------------------------------------+1 row in set (0.001 sec)
```

不过，用户luke仅仅是一个普通账户，没有数据库的任何操作权限。不信的话，可以切换到luke账户来查询数据库管理系统中当前都有哪些数据库。可以发现，该账户甚至没法查看完整的数据库列表（刚才使用root账户时可以查看到3个数据库列表）：

```
MariaDB [mysql]> exitBye[root@linuxprobe ~]# mysql -u luke -pEnter password: 输入luke用户的数据库密码Welcome to the MariaDB monitor.  Commands end with ; or \g.Your MariaDB connection id is 21Server version: 10.3.11-MariaDB MariaDB ServerCopyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.MariaDB [(none)]> SHOW databases;+--------------------+| Database           |+--------------------+| information_schema |+--------------------+1 row in set (0.001 sec)
```

数据库管理系统所使用的命令一般都比较复杂。我们以grant命令为例进行说明。grant命令用于为账户进行授权，其常见格式如表18-1所示。在使用grant命令时需要写上要赋予的权限、数据库及表单名称，以及对应的账户及主机信息。其实，只要理解了命令中每个字段的功能含义，也就不觉得命令复杂难懂了。

表18-1                    GRANT命令的常见格式以及解释

| 命令                                           | 作用                                             |
| ---------------------------------------------- | ------------------------------------------------ |
| GRANT 权限 ON 数据库.表单名称 TO 用户名@主机名 | 对某个特定数据库中的特定表单给予授权             |
| GRANT 权限 ON 数据库.* TO 用户名@主机名        | 对某个特定数据库中的所有表单给予授权             |
| GRANT 权限 ON *.* TO 用户名@主机名             | 对所有数据库及所有表单给予授权                   |
| GRANT 权限1,权限2 ON 数据库.* TO 用户名@主机名 | 对某个数据库中的所有表单给予多个授权             |
| GRANT ALL PRIVILEGES ON *.* TO 用户名@主机名   | 对所有数据库及所有表单给予全部授权（需谨慎操作） |



当然，账户的授权工作肯定是需要数据库管理员来执行的。下面以root管理员的身份登录到数据库管理系统中，针对mysql数据库中的user表单向账户luke授予查询、更新、删除以及插入等权限。

刘遄老师特别懂同学们现在心里想的是什么~哈哈，我起初也觉得在每条数据库命令后都要加上;（分号）来结束特别的不方便，时常还会忘记，但敲的命令多了也就自然习惯了。授权操作执行后来查看下luke用户的权限吧：

```
[root@linuxprobe ~]# mysql -u root -pEnter password: 输入管理员的数据库密码MariaDB [(none)]> use mysql;Reading table information for completion of table and column namesYou can turn off this feature to get a quicker startup with -ADatabase changedMariaDB [mysql]> GRANT SELECT,UPDATE,DELETE,INSERT ON mysql.user TO luke@localhost;Query OK, 0 rows affected (0.001 sec)
```

在执行完上述授权操作之后，我们再查看一下账户luke的权限：

```
MariaDB [(none)]>  SHOW GRANTS FOR luke@localhost;+---------------------------------------------------------------------------------------------+| Grants for luke@localhost                                                                   |+---------------------------------------------------------------------------------------------+| GRANT USAGE ON *.* TO 'luke'@'localhost' IDENTIFIED BY PASSWORD '*55D9962586BE75F4B7D421E6655973DB07D6869F' || GRANT SELECT, INSERT, UPDATE, DELETE ON `mysql`.`user` TO 'luke'@'localhost'                |+---------------------------------------------------------------------------------------------+2 rows in set (0.000 sec)
```

上面输出信息中显示账户luke已经拥有了针对mysql数据库中user表单的一系列权限了。这时我们再切换到账户luke，此时就能够看到mysql数据库了，而且还能看到表单user（其余表单会因无权限而被继续隐藏）：

```
[root@linuxprobe ~]# mysql -u luke -pEnter password: 输入luke用户的数据库密码MariaDB [(none)]> SHOW databases;+--------------------+| Database           |+--------------------+| information_schema || mysql              |+--------------------+2 rows in set (0.000 sec)MariaDB [(none)]> use mysql;Database changedMariaDB [mysql]> SHOW tables;+-----------------+| Tables_in_mysql |+-----------------+| user            |+-----------------+1 row in set (0.001 sec)MariaDB [mysql]> exitByes
```

大家不要心急，我们接下来会慢慢学习数据库内容的修改方法。当前，先切换回管理员，移除刚才的授权。

```
[root@linuxprobe ~]# mysql -u root -pEnter password: 输入管理员的数据库密码MariaDB [(none)]> use mysql;Database changedMariaDB [(none)]> REVOKE SELECT,UPDATE,DELETE,INSERT ON mysql.user FROM luke@localhost;Query OK, 0 rows affected (0.00 sec)
```

可以看到，除了移除授权的命令（revoke）与授权命令（grant）不同之外，其余部分都是一致的。这不仅好记而且也容易理解。执行移除授权命令后，再来查看账户luke的信息：

```
MariaDB [(none)]> SHOW GRANTS FOR luke@localhost;+---------------------------------------------------------------------------------------------+| Grants for luke@localhost                                                                   |+---------------------------------------------------------------------------------------------+| GRANT USAGE ON *.* TO 'luke'@'localhost' IDENTIFIED BY PASSWORD '*55D9962586BE75F4B7D421E6655973DB07D6869F' |+---------------------------------------------------------------------------------------------+1 row in set (0.001 sec)
```

不再需要某个用户时，可以直接用DROP命令进行删除：

```
MariaDB [(none)]> DROP user luke@localhost;Query OK, 0 rows affected (0.000 sec)
```

##### **18.4 创建数据库与表单**

在MariaDB数据库管理系统中，一个数据库可以存放多个数据表，数据表单是数据库中最重要最核心的内容。我们可以根据自己的需求自定义数据库表结构，然后在其中合理地存放数据，以便后期轻松地维护和修改。表18-2罗列了后文中将使用到的数据库命令以及对应的作用。

表18-2                    用于创建数据库的命令以及作用

| 命令用法                                                     | 作用                   |
| ------------------------------------------------------------ | ---------------------- |
| CREATE database 数据库名称。                                 | 创建新的数据库         |
| DESCRIBE 表单名称;                                           | 描述表单               |
| UPDATE 表单名称 SET attribute=新值 WHERE attribute > 原始值; | 更新表单中的数据       |
| USE 数据库名称;                                              | 指定使用的数据库       |
| SHOW databases;                                              | 显示当前已有的数据库   |
| SHOW tables;                                                 | 显示当前数据库中的表单 |
| SELECT * FROM 表单名称;                                      | 从表单中选中某个记录值 |
| DELETE FROM 表单名 WHERE attribute=值;                       | 从表单中删除某个记录值 |



建立数据库是管理数据的起点。现在尝试创建一个名为linuxprobe的数据库，然后再查看数据库列表，此时就能看到它了：

```
MariaDB [(none)]>  CREATE DATABASE linuxprobe;Query OK, 1 row affected (0.001 sec)MariaDB [(none)]>  SHOW databases;+--------------------+| Database           |+--------------------+| information_schema || linuxprobe         || mysql              || performance_schema |+--------------------+4 rows in set (0.001 sec)
```

MariaDB与MySQL同属关系型数据库（Relational Database Management System），“关系型”数据库有些类似于表格的概念，一个关系型数据库由一个或多个表格/表单组成，如图18-2所示。

![第18章 使用MariaDB数据库管理系统第18章 使用MariaDB数据库管理系统](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%A6%82%E5%BF%B5.png)

图18-2 数据库存储概念

上图中，表头代表每一列的名称；列代表具有相同数据类型的数据集合；行代表用来描述事物的具体信息；值代行的具体信息，每个值均与该列的其他数据类型相同；键代表用来识别某个特定事物的方法，在当前列中具有唯一性。

比如在新建的linuxprobe数据库中创建表单mybook，然后进行表单的初始化，即定义存储数据内容的结构。我们分别定义3个字段项，其中，长度为15个字符的字符型字段name用来存放图书名称，整型字段price和pages分别存储图书的价格和页数。当执行完下述命令之后，就可以看到表单的结构信息了：

```
MariaDB [(none)]> use linuxprobe;Database changedMariaDB [linuxprobe]> CREATE TABLE mybook (name char(15),price int,pages int);Query OK, 0 rows affected (0.009 sec)MariaDB [linuxprobe]> DESCRIBE mybook;+-------+----------+------+-----+---------+-------+| Field | Type     | Null | Key | Default | Extra |+-------+----------+------+-----+---------+-------+| name  | char(15) | YES  |     | NULL    |       || price | int(11)  | YES  |     | NULL    |       || pages | int(11)  | YES  |     | NULL    |       |+-------+----------+------+-----+---------+-------+3 rows in set (0.002 sec)
```

##### **18.5 管理表单及数据**

接下来向mybook数据表单中插一条图书信息。为此需要使用INSERT命令，并在命令中写清表单名称以及对应的字段项。执行该命令之后即可完成图书写入信息。下面我们使用该命令插入一条图书信息，其中书名为linuxprobe，价格和页数分别是60元和518页。在命令执行后也就意味着图书信息已经成功写入到数据表单中，然后就可以查询表单中的内容了。我们在使用select命令查询表单内容时，需要加上想要查询的字段；如果想查看表单中的所有内容，则可以使用星号（*）通配符来显示：

```
MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe','60', '518');Query OK, 1 row affected (0.001 sec)MariaDB [linuxprobe]> SELECT * from mybook;+------------+-------+-------+| name       | price | pages |+------------+-------+-------+| linuxprobe |    60 |   518 |+------------+-------+-------+1 row in set (0.000 sec)
```

对数据库运维人员来讲，需要做好四门功课—增、删、改、查。这意味着创建数据表单并在其中插入内容仅仅是第一步，还需要掌握数据表单内容的修改方法。例如，我们可以使用update命令将刚才插入的linuxprobe图书信息的价格修改为55元，然后在使用select命令查看该图书的名称和定价信息。注意，因为这里只查看图书的名称和定价，而不涉及页码，所以无须再用星号通配符来显示所有内容。

```
MariaDB [linuxprobe]> UPDATE mybook SET price=55 ;Query OK, 1 row affected (0.002 sec)Rows matched: 1  Changed: 1  Warnings: 0MariaDB [linuxprobe]> SELECT name,price FROM mybook;+------------+-------+| name       | price |+------------+-------+| linuxprobe |    55 |+------------+-------+1 row in set (0.000 sec)
```

想指定某一条进行修改？没问题的，用where命令进行限定即可，例如再插入两条书籍信息：

```
MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxcool','85', '300');Query OK, 1 row affected (0.001 sec)MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxdown','105', '500');Query OK, 1 row affected (0.001 sec)
```

然后仅修改名称为linuxcool的书籍价格为60元，不影响其他书籍信息：

```
MariaDB [linuxprobe]> UPDATE mybook SET price=60 where name='linuxcool';Query OK, 1 row affected (0.001 sec)Rows matched: 1  Changed: 1  Warnings: 0MariaDB [linuxprobe]> select * from mybook;+------------+-------+-------+| name       | price | pages |+------------+-------+-------+| linuxprobe |    55 |   518 || linuxcool  |    60 |   300 || linuxdown  |   105 |   500 |+------------+-------+-------+3 rows in set (0.001 sec)
```

我们还可以使用delete命令删除某个数据表单中的内容。下面我们使用delete命令删除数据表单mybook中的所有内容，然后再查看该表单中的内容，可以发现该表单内容为空了。

```
MariaDB [linuxprobe]> DELETE FROM mybook;Query OK, 3 row affected (0.001 sec)MariaDB [linuxprobe]> SELECT * FROM mybook;Empty set (0.000 sec)
```

一般来讲，数据表单中会存放成千上万条数据信息。比如我们刚刚创建的用于保存图书信息的mybook表单，随着时间的推移，里面的图书信息也会越来越多。在这样的情况下，如果我们只想查看其价格大于某个数值的图书时，又该如何定义查询语句呢？

下面先使用insert插入命令依次插入4条图书信息：

```
MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe1','30','518');Query OK, 1 row affected (0.05 sec)MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe2','50','518');Query OK, 1 row affected (0.05 sec)MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe3','80','518');Query OK, 1 row affected (0.01 sec)MariaDB [linuxprobe]> INSERT INTO mybook(name,price,pages) VALUES('linuxprobe4','100','518');Query OK, 1 row affected (0.00 sec)
```

要想让查询结果更加精准，就需要结合使用select与where命令了。其中，where命令是在数据库中进行匹配查询的条件命令。通过设置查询条件，就可以仅查找出符合该条件的数据。表18-3列出了where命令中常用的查询参数以及作用。

表18-3                    where命令中使用的参数以及作用

| 参数    | 作用             |
| ------- | ---------------- |
| =       | 相等             |
| <>或!=  | 不相等           |
| >       | 大于             |
| <       | 小于             |
| >=      | 大于或等于       |
| <=      | 小于或等于       |
| BETWEEN | 在某个范围内     |
| LIKE    | 搜索一个例子     |
| IN      | 在列中搜索多个值 |



现在进入动手环节。分别在mybook表单中查找出价格大于75元或价格不等于80元的图书，其对应的命令如下所示。在熟悉了这两个查询条件之后，大家可以自行尝试精确查找图书名为linuxprobe2的图书信息。

```
MariaDB [linuxprobe]> SELECT * FROM mybook WHERE price>75;+-------------+-------+-------+| name        | price | pages |+-------------+-------+-------+| linuxprobe3 |    80 |   518 || linuxprobe4 |   100 |   518 |+-------------+-------+-------+2 rows in set (0.001 sec)MariaDB [linuxprobe]> SELECT * FROM mybook WHERE price!=80;+-------------+-------+-------+| name        | price | pages |+-------------+-------+-------+| linuxprobe1 |    30 |   518 || linuxprobe2 |    50 |   518 || linuxprobe4 |   100 |   518 |+-------------+-------+-------+3 rows in set (0.000 sec)
```

匹配的条件越多，获得信息就越精准。在WHERE命令的后面追加AND操作符，可以进行多次匹配。例如找到这本价格为30元，页数为518的书籍名称：

```
MariaDB [linuxprobe]> SELECT * from mybook WHERE price=30 AND pages=518 ;+-------------+-------+-------+| name        | price | pages |+-------------+-------+-------+| linuxprobe1 |    30 |   518 |+-------------+-------+-------+1 row in set (0.000 sec)
```

##### **18.6 数据库的备份及恢复**

前文提到，本书的技术主线是Linux系统的运维方向，不会对数据库管理系统的操作进行深入的讲解，因此大家掌握了上面这些基本的数据库操作命令之后就足够了。下面要讲解的是数据库的备份以及恢复，这些知识比较实用，希望大家能够掌握。

mysqldump命令用于备份数据库数据，格式为“mysqldump [参数] [数据库名称]”。其中参数与mysql命令大致相同，-u参数用于定义登录数据库的账户名称，-p参数代表密码提示符。下面将linuxprobe数据库中的内容导出成一个文件，并保存到root管理员的家目录中：

```
[root@linuxprobe ~]# mysqldump -u root -p linuxprobe > /root/linuxprobeDB.dumpEnter password: 输入管理员的数据库密码
```

然后进入MariaDB数据库管理系统，彻底删除linuxprobe数据库，这样mybook数据表单也将被彻底删除。然后重新建立linuxprobe数据库：

```
[root@linuxprobe ~]# mysql -u root -pEnter password: 输入管理员的数据库密码MariaDB [(none)]> DROP DATABASE linuxprobe;Query OK, 1 row affected (0.04 sec)MariaDB [(none)]> SHOW databases;+--------------------+| Database           |+--------------------+| information_schema || mysql              || performance_schema |+--------------------+3 rows in set (0.02 sec)MariaDB [(none)]> CREATE DATABASE linuxprobe;Query OK, 1 row affected (0.00 sec)
```

接下来是见证数据恢复效果的时刻！使用输入重定向符把刚刚备份的数据库文件导入到mysql命令中，然后执行该命令。接下来登录到MariaDB数据库，就又能看到linuxprobe数据库以及mybook数据表单了。数据库恢复成功！

```
[root@linuxprobe ~]# mysql -u root -p linuxprobe < /root/linuxprobeDB.dump Enter password: 输入管理员的数据库密码[root@linuxprobe ~]# mysql -u root -pEnter password: 输入管理员的数据库密码MariaDB [(none)]> use linuxprobe;Database changedMariaDB [linuxprobe]> SHOW tables;+----------------------+| Tables_in_linuxprobe |+----------------------+| mybook               |+----------------------+1 row in set (0.000 sec)MariaDB [linuxprobe]> describe mybook;+-------+----------+------+-----+---------+-------+| Field | Type     | Null | Key | Default | Extra |+-------+----------+------+-----+---------+-------+| name  | char(15) | YES  |     | NULL    |       || price | int(11)  | YES  |     | NULL    |       || pages | int(11)  | YES  |     | NULL    |       |+-------+----------+------+-----+---------+-------+3 rows in set (0.002 sec)
```

**出现问题?大胆提问!**

> 因读者们硬件不同或操作错误都可能导致实验配置出错，请耐心再仔细看看操作步骤吧，不要气馁~
>
> Linux技术交流学习请加读者群（**推荐**）：https://www.linuxprobe.com/club
>
> *本群特色：确保每一位群友都是《Linux就该这么学》的读者，答疑更有针对性，不定期领取定制礼品。

**本章节的复习作业(答案就在问题的下一行哦，用鼠标选中即可看到的~)**

1．RHEL 7系统为何选择使用MariaDB替代MySQL数据库管理系统？

**答：**因为MariaDB由开源社区进行维护，且不受商业专利限制。

2．初始化MariaDB或MySQL数据库管理系统的命令是什么？

**答：**是mysql_secure_installation命令，建议每次安装MariaDB或MySQL数据库管理系统后都执行这条命令。

3．用来查看已有数据库或数据表单的命令是什么？

**答：**要查看当前已有的数据库列表，需执行SHOW databases;命令；要查看已有的数据表单列表，则需执行SHOW tables;命令。

4．切换至某个指定数据库的命令是什么？

**答：**执行“use 数据库名称”命令即可切换成功。

5．若想针对某个账户进行授权或取消授权操作，应该执行什么命令？

**答：**针对账户进行授权，需执行GRANT命令；取消授权则需执行REVOKE命令。

6．若只想查看mybook表单中的name字段，应该执行什么命令？

**答：**应执行SELECT name FROM mybook命令。

7．若只想查看mybook表单中价格大于75元的图书信息，应该执行什么命令？

**答：**应执行SELECT * FROM mybook WHERE price>75命令。

8． 要想把linuxprobe数据库中的内容导出为一个文件（保存到root管理员的家目录中），应该执行什么命令？

**答：**应执行mysqldump -u root -p linuxprobe > /root/linuxprobeDB.dump命令。

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
[root@linuxprobe ~]# dnf install -y syslinuxUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:07:57 ago on Fri 30 Apr 2021 01:47:18 AM CST.Dependencies resolved.================================================================================ Package                     Arch       Version         Repository     Size================================================================================Installing: syslinux                    x86_64     6.04-1.el8      BaseOS        576 kInstalling dependencies: syslinux-nonlinux           noarch     6.04-1.el8      BaseOS        554 kTransaction Summary================================================================================Install  2 Packages………………省略部分输出信息………………Installed:  syslinux-6.04-1.el8.x86_64              syslinux-nonlinux-6.04-1.el8.noarch             Complete!
```

我们首先需要把SYSLinux提供的引导文件复制到TFTP服务程序的默认目录中，也就是前文提到的文件pxelinux.0，这样客户端主机就能够顺利地获取到引导文件了。另外在RHEL 8系统光盘镜像中也有一些需要调取的引导文件。确认光盘镜像已经被挂载到/media/cdrom目录后，使用复制[命令](https://www.linuxcool.com/)将光盘镜像中自带的一些引导文件也复制到TFTP服务程序的默认目录中。

```
[root@linuxprobe ~]# cd /var/lib/tftpboot[root@linuxprobe tftpboot]# cp /usr/share/syslinux/pxelinux.0 .[root@linuxprobe tftpboot]# cp /media/cdrom/images/pxeboot/* .[root@linuxprobe tftpboot]# cp /media/cdrom/isolinux/* .cp: overwrite './initrd.img'? ycp: overwrite './TRANS.TBL'? ycp: overwrite './vmlinuz'? y
```

cp[命令](https://www.linuxcool.com/)后面接的句号（.）代表当前工作目录，也就是复制到当前所在/var/lib/tftpboot的作用。在复制过程中若出现多个目录保存有相同的文件情况时，则可手动敲击y进行覆盖即可。

然后在TFTP服务程序的目录中新建pxelinux.cfg目录，虽然该目录的名字带有后缀，但依然也是目录，而非文件！将系统光盘中的开机选项菜单复制到该目录中，并命名为default。这个default文件就是开机时的选项菜单，如图19-4所示。

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2020/05/RHEL-8%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E7%95%8C%E9%9D%A2.png)

图19-4 Linux系统的引导菜单界面

```
[root@linuxprobe tftpboot]# mkdir pxelinux.cfg[root@linuxprobe tftpboot]# cp /media/cdrom/isolinux/isolinux.cfg pxelinux.cfg/default
```

默认的开机菜单中有三个选项，要么是安装系统，要么是对安装介质进行检验，那么是Troubleshooting排错模式。既然已经确定采用无人值守的方式安装系统，还需要为每台主机手动选择相应的选项，未免与我们的主旨（无人值守安装）相悖。

现在我们编辑这个default文件，把第1行的default参数修改为linux，这样系统在开机时就会默认执行那个名称为linux的选项了。对应的linux选项大约在64行，将默认的光盘镜像安装方式修改成FTP文件传输方式，并指定好光盘镜像的获取网址以及Kickstart应答文件的获取路径：

```
[root@linuxprobe tftpboot]# vim pxelinux.cfg/default  1 default linux  2 timeout 600  3   4 display boot.msg  5   6 # Clear the screen when exiting the menu, instead of leaving the menu displayed.  7 # For vesamenu, this means the graphical background is still displayed without  8 # the menu itself for as long as the screen remains in graphics mode.  9 menu clear 10 menu background splash.png 11 menu title Red Hat Enterprise Linux 8.0.0 12 menu vshift 8 13 menu rows 18 14 menu margin 8 15 #menu hidden 16 menu helpmsgrow 15 17 menu tabmsgrow 13 18  19 # Border Area 20 menu color border * #00000000 #00000000 none 21  22 # Selected item 23 menu color sel 0 #ffffffff #00000000 none 24  25 # Title bar 26 menu color title 0 #ff7ba3d0 #00000000 none 27  28 # Press [Tab] message 29 menu color tabmsg 0 #ff3a6496 #00000000 none 30  31 # Unselected menu item 32 menu color unsel 0 #84b8ffff #00000000 none 33  34 # Selected hotkey 35 menu color hotsel 0 #84b8ffff #00000000 none 36  37 # Unselected hotkey 38 menu color hotkey 0 #ffffffff #00000000 none 39  40 # Help text 41 menu color help 0 #ffffffff #00000000 none 42  43 # A scrollbar of some type? Not sure. 44 menu color scrollbar 0 #ffffffff #ff355594 none 45  46 # Timeout msg 47 menu color timeout 0 #ffffffff #00000000 none 48 menu color timeout_msg 0 #ffffffff #00000000 none 49  50 # Command prompt text 51 menu color cmdmark 0 #84b8ffff #00000000 none 52 menu color cmdline 0 #ffffffff #00000000 none 53  54 # Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message. 55  56 menu tabmsg Press Tab for full configuration options on menu items. 57  58 menu separator # insert an empty line 59 menu separator # insert an empty line 60  61 label linux 62   menu label ^Install Red Hat Enterprise Linux 8.0.0 63   kernel vmlinuz 64   append initrd=initrd.img inst.stage2=ftp://192.168.10.10 ks=ftp://192.168.10.10/pub/ks.cfg quiet 65 ………………省略部分输出信息………………
```

建议在安装源的后面加入quiet参数，意为使用静默安装方式，不需要用户再进行确认。文件修改完毕后保存即可，开机选项菜单是被调用的文件，因此不需要单独重启任何的服务。

###### **19.2.4 配置VSFtpd服务程序**

在这套无人值守安装系统的服务中，光盘镜像是通过FTP协议传输的，因此势必要用到vsftpd服务程序。当然，也可以使用httpd服务程序来提供Web网站访问的方式，只要能确保将光盘镜像顺利传输给客户端主机即可。如果打算使用Web网站服务来提供光盘镜像，一定记得将上面配置文件中的光盘镜像获取网址和Kickstart应答文件获取网址修改一下。

```
[root@linuxprobe tftpboot]# dnf install -y vsftpdUpdating Subscription Management repositories.Unable to read consumer identityThis system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.Last metadata expiration check: 0:28:28 ago on Fri 30 Apr 2021 01:47:18 AM CST.Dependencies resolved.=============================================================================== Package       Arch       Version           Repository        Size===============================================================================Installing: vsftpd        x86_64     3.0.3-28.el8      AppStream         180 kTransaction Summary===============================================================================Install  1 Package………………省略部分输出信息………………Installed:  vsftpd-3.0.3-28.el8.x86_64                                                                     Complete!
```

RHEL 8系统版本的vsftpd服务默认不允许匿名公开访问模式，因此需要手动进行开启：

```
[root@linuxprobe ~ ]# vim /etc/vsftpd/vsftpd.conf# Example config file /etc/vsftpd/vsftpd.conf## The default compiled in settings are fairly paranoid. This sample file# loosens things up a bit, to make the ftp daemon more usable.# Please see vsftpd.conf.5 for all compiled in defaults.## READ THIS: This example file is NOT an exhaustive list of vsftpd options.# Please read the vsftpd.conf.5 manual page to get a full idea of vsftpd's# capabilities.## Allow anonymous FTP? (Beware - allowed by default if you comment this out).anonymous_enable=YES………………省略部分输出信息………………
```

[刘遄](https://www.linuxprobe.com/)老师再啰嗦一句，在配置文件修改正确之后，一定将相应的服务程序添加到开机启动项中，这样无论是在生产环境中还是在[红帽](https://www.linuxprobe.com/)认证考试中，都可以在设备重启之后依然能提供相应的服务。希望各位读者一定养成这个好习惯。

```
[root@linuxprobe ~]# systemctl restart vsftpd[root@linuxprobe ~]# systemctl enable  vsftpdCreated symlink /etc/systemd/system/multi-user.target.wants/vsftpd.service → /usr/lib/systemd/system/vsftpd.service.
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
[root@linuxprobe ~]# cp ~/anaconda-ks.cfg /var/ftp/pub/ks.cfg[root@linuxprobe ~]# chmod +r /var/ftp/pub/ks.cfg
```

Kickstart应答文件并没有想象中的那么复杂，它总共只有44行左右的参数和注释内容，大家完全可以通过参数的名称及介绍来快速了解每个参数的作用。刘遄老师在这里挑选几个比较有代表性的参数进行讲解，其他参数建议大家自行修改测试。

其中第1至10行，代表安装硬盘名称为sda及使用LVM逻辑卷管理期技术。这便要求我们在后续新建客户端虚拟机时，硬盘一定要选择SCSI或SATA，如图19-5所示，否则会变成/dev/hd或/dev/nvme开头的名称，进而会因找不到硬盘设备而终止安装进程。

而第8行的软件仓库，应该为由FTP服务器提供的网络路径。第10行的安装源，也需要由CDROM光盘改为网络安装源：

```
  1 #version=RHEL8  2 ignoredisk --only-use=sda  3 autopart --type=lvm  4 # Partition clearing information  5 clearpart --none --initlabel  6 # Use graphical install  7 graphical  8 repo --name="AppStream" --baseurl=ftp://192.168.10.10/AppStream  9 # Use CDROM installation media 10 url --url=ftp://192.168.10.10/BaseOS
```

![第19章 使用PXE+Kickstart无人值守安装服务第19章 使用PXE+Kickstart无人值守安装服务](https://www.linuxprobe.com/wp-content/uploads/2021/04/%E8%AE%BE%E5%A4%87%E7%B1%BB%E5%9E%8B.png)

图19-5 选择SCSI或SATA硬盘类型

其中第11至20行中，keyboard参数为硬盘类型，一般都不需要修改。但第17行的网卡信息万一要注意，一定要让网卡默认处于DHCP的模式，否则几十、上百台主机同时被创建出来，IP地址会相互冲突导致后续无法管理。

```
 11 # Keyboard layouts 12 keyboard --vckeymap=us --xlayouts='us' 13 # System language 14 lang en_US.UTF-8 15  16 # Network information 17 network  --bootproto=dhcp --device=ens160 --onboot=on --ipv6=auto --activate 18 network  --hostname=linuxprobe.com 19 # Root password 20 rootpw --iscrypted $6$EzIFyouUyBvWRIXv$y3bW3JZ2vD4c8bwVyKt7J90gyjULALTMLrnZZmvVujA75EpCCn50rlYm64MHAInbMAXAgn2Bmlgou/pYjUZzL1
```

其中第21行至30行中timezone参数定义了系统默认市区为上海，如果同学们服务器时间存在不准确的情况，则如下修改即可。第29行创建了一个普通用户，密码值可复制/etc/shadow文件中的加密密文，它会由系统自动创建出来。

```
 21 # X Window System configuration information 22 xconfig  --startxonboot 23 # Run the Setup Agent on first boot 24 firstboot --enable 25 # System services 26 services --disabled="chronyd" 27 # System timezone 28 timezone Asia/Shanghai --isUtc --nontp 29 user --name=linuxprobe --password=$6$a5fEjghDXGPvEoQc$HQqzvBlGVyhsJjgKFDTpiCEavS.inAwNTLZm/I5R5ALLKaMdtxZoKgb4/EaDyiPSSNNHGqrEkRnfJWap56m./. --iscrypted --gecos="linuxprobe" 30 
```

最后的第31行至44行代表了要安装的软件来源，“graphical-server-environment”即带有图形化界面的服务器环境，对照安装界面中的“Server With GUI”选项。

```
 31 %packages 32 @^graphical-server-environment 33  34 %end 35  36 %addon com_redhat_kdump --disable --reserve-mb='auto' 37  38 %end 39  40 %anaconda 41 pwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notempty 42 pwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyok 43 pwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty 44 %end
```

所以实际算下来修改的并不多，默认参数就已经非常合适了。最后预览一下ks.cfg文件的全貌，读者如果在生产环境中需要使用，可复制以下内容就能直接使用：

```
[root@linuxprobe ~ ]# cat /var/ftp/pub/ks.cfg#version=RHEL8ignoredisk --only-use=sdaautopart --type=lvm# Partition clearing informationclearpart --none --initlabel# Use graphical installgraphicalrepo --name="AppStream" --baseurl=ftp://192.168.10.10/AppStream# Use CDROM installation mediaurl --url=ftp://192.168.10.10/BaseOS# Keyboard layoutskeyboard --vckeymap=us --xlayouts='us'# System languagelang en_US.UTF-8# Network informationnetwork  --bootproto=dhcp --device=ens160 --onboot=on --ipv6=auto --activatenetwork  --hostname=linuxprobe.com# Root passwordrootpw --iscrypted $6$EzIFyouUyBvWRIXv$y3bW3JZ2vD4c8bwVyKt7J90gyjULALTMLrnZZmvVujA75EpCCn50rlYm64MHAInbMAXAgn2Bmlgou/pYjUZzL1# X Window System configuration informationxconfig  --startxonboot# Run the Setup Agent on first bootfirstboot --enable# System servicesservices --disabled="chronyd"# System timezonetimezone Asia/Shanghai --isUtc --nontpuser --name=linuxprobe --password=$6$a5fEjghDXGPvEoQc$HQqzvBlGVyhsJjgKFDTpiCEavS.inAwNTLZm/I5R5ALLKaMdtxZoKgb4/EaDyiPSSNNHGqrEkRnfJWap56m./. --iscrypted --gecos="linuxprobe"%packages@^graphical-server-environment%end%addon com_redhat_kdump --disable --reserve-mb='auto'%end%anacondapwpolicy root --minlen=6 --minquality=1 --notstrict --nochanges --notemptypwpolicy user --minlen=6 --minquality=1 --notstrict --nochanges --emptyokpwpolicy luks --minlen=6 --minquality=1 --notstrict --nochanges --notempty%end                                                                                          
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

### **Tips**

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

### **Tips**

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
 63         # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000 64         # 65         location ~ \.php$ { 66             root           html; 67             fastcgi_pass   127.0.0.1:9000; 68             fastcgi_index  index.php; 69             fastcgi_param  SCRIPT_FILENAME  /usr/local/nginx/html$fastcgi_script_name; 70             include        fastcgi_params; 71         }
```

第4步：通过编译源码方式安装的服务默认不能够被systemctl命令所管理，而要用Nginx服务本身的管理工具进行操作，对应的命令目录是/usr/local/nginx/sbin。但使用绝对路径的形式输入命令未免会显得麻烦，建议将/usr/local/nginx/sbin路径加入到PATH变量中，让Bash解释器在后续执行命令时自动的搜索到它。source命令加载配置文件让参数立即生效，下次就只需要输入“nginx”命令即可启动网站服务了，很方便呢~

```
[root@linuxprobe lnmp]# vim ~/.bash_profile# .bash_profile# Get the aliases and functionsif [ -f ~/.bashrc ]; then	. ~/.bashrcfi# User specific environment and startup programsPATH=$PATH:$HOME/bin:/usr/local/nginx/sbinexport PATH[root@linuxprobe lnmp]# source ~/.bash_profile[root@linuxprobe lnmp]# nginx 
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
[root@linuxprobe lnmp]# tar xvf mysql-8.0.18.tar.xz[root@linuxprobe lnmp]# mv mysql-8.0.18-linux-glibc2.12-x86_64 mysql[root@linuxprobe lnmp]# mv mysql /usr/local
```

第2步：生产环境中管理MySQL数据库时，有两个比较常用的目录。其一是/usr/local/mysql目录，这是用于保存MySQL数据库程序文件的路径。还有一个是/usr/local/mysql/data目录，它用于存储数据库的具体内容，每个数据库的内容会被单独存放到一个目录内。对于存放实际数据库文件的data目录，用户需要先手动创建出来：

```
[root@linuxprobe lnmp]# cd /usr/local/mysql[root@linuxprobe mysql]# mkdir data
```

第3步：初始化MySQL服务程序，对目录进行授权，保证数据能够被mysql系统用户所读取。在初始化阶段，应使用mysqld命令确认管理MySQL数据库服务的用户名称、数据保存目录及编码信息。信息确认无误后会初始化完毕，在最后一段会给予用户初始化临时密码，一定要保存好，例如本书所分配的密码为qfroRs,Ei4Ls。

```
[root@linuxprobe mysql]# chown -R mysql:mysql /usr/local/mysql[root@linuxprobe mysql]# cd bin[root@linuxprobe bin]# ./mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data2021-05-06T07:07:06.243270Z 0 [System] [MY-013169] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.18) initializing of server in progress as process 76062021-05-06T07:07:08.116268Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: qfroRs,Ei4Ls
```

第4步：与Nginx服务相似，MySQL数据库的二进制可执行命令也是单独存放在自身程序目录中的，每一次都切换到/usr/local/mysql/bin目录中再执行着实有些麻烦，要能也加入到PATH变量中可就方便太多了。说干就干：

```
[root@linuxprobe bin]# vim ~/.bash_profile# .bash_profile# Get the aliases and functionsif [ -f ~/.bashrc ]; then        . ~/.bashrcfi# User specific environment and startup programsPATH=$PATH:$HOME/bin:/usr/local/nginx/sbin:/usr/local/mysql/binexport PATH[root@linuxprobe bin]# source ~/.bash_profile
```

这样设置后，即便返回到了源码目录，也可以继续执行MySQL数据库的管理命令。不过先别着急！既然是手动安装服务，那么让文件“归位”的重任就只得亲力亲为了——将启动脚本mysql.server放入到/etc/init.d目录中，让服务器每次重启后都能自动启动数据库，并给予可执行权限。

libtinfo.so.5文件是MySQL数据库在8.0版本后重要的函数库文件，需要将libtinfo.so.6.1文件复制或者作为链接文件才能正常启动：

```
[root@linuxprobe bin]# cd /usr/local/mysql[root@linuxprobe mysql]# cp -a support-files/mysql.server /etc/init.d/[root@linuxprobe mysql]# chmod a+x /etc/init.d/mysql.server[root@linuxprobe mysql]# ln -s /usr/lib64/libtinfo.so.6.1 /usr/lib64/libtinfo.so.5 
```

第5步：执行MySQL数据库服务启动文件，并进行初始化工作。8.0版本以后为了安全着想，不再允许用户使用临时密码管理数据库内容，也不能远程控制，用户必须修改初始化密码后才能使用。数据库作为系统重要的组成服务，密码不建议少于20位，例如想修改为“PObejCBeDzTRCncXwgBy”。

```
[root@linuxprobe mysql]# /etc/init.d/mysql.server start Starting MySQL.Logging to '/usr/local/mysql/data/linuxprobe.com.err'.. SUCCESS! [root@linuxprobe mysql]# mysql -u root -pEnter password: 输入初始化时给的原始密码Welcome to the MySQL monitor.  Commands end with ; or \g.Your MySQL connection id is 8Server version: 8.0.18Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.Oracle is a registered trademark of Oracle Corporation and/or itsaffiliates. Other names may be trademarks of their respectiveowners.Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.mysql> alter user 'root'@'localhost' identified by 'PObejCBeDzTRCncXwgBy'; Query OK, 0 rows affected (0.01 sec)mysql> 
```

但这样还是不行，需要继续切换至mysql数据库中，修改user表单的密码值。这也是从MySQL数据库8.0版本后才开始的新安全要求，看过《Linux就该这么学》RHEL7版本的老读者应该记得5.6版本时就没有这么麻烦。

```
mysql> use mysql;Reading table information for completion of table and column namesYou can turn off this feature to get a quicker startup with -ADatabase changedmysql> show tables;+---------------------------+| Tables_in_mysql           |+---------------------------+| columns_priv              || tables_priv               || time_zone                 || time_zone_leap_second     || time_zone_name            || time_zone_transition      || time_zone_transition_type || user                      || …………省略部分输出信息…………  |+---------------------------+33 rows in set (0.00 sec)mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PObejCBeDzTRCncXwgBy';Query OK, 0 rows affected (0.01 sec)
```

由于稍后将在20.4小节安装部署WordPress网站系统，因此现在需要提前把数据库新建出来：

```
mysql> create database linuxcool;Query OK, 1 row affected (0.00 sec)mysql> exitBye
```

###### **20.2.3 配置php服务**

![第20章 使用LNMP架构部署动态网站环境第20章 使用LNMP架构部署动态网站环境](https://www.linuxprobe.com/wp-content/uploads/2021/05/PHP.png)PHP（Hypertxt Preprocessor，超文本预处理器）是一种通用的开源脚本语言，发明于1995年，它吸取了C语言、Java语言及Perl语言的很多优点，具有开源、免费、快捷、跨平台性强、效率高等优良特性，是目前Web开发领域最常用的语言之一。使用源码包的方式编译安装PHP语言环境其实并不复杂，难点在于解决PHP的程序包和其他软件的依赖关系。

第1步：解压php安装包软件并编译安装。编译中需要使用“prefix”参数指定安装路径，使用“--with-mysqli”等命令开启对数据库的支持模块，为后面在线安装网站夯实基础。

```
[root@linuxprobe mysql]# cd /lnmp[root@linuxprobe lnmp]# tar xvf php-7.3.5.tar.gz[root@linuxprobe lnmp]# cd php-7.3.5/[root@linuxprobe php-7.3.5]# ./configure --prefix=/usr/local/php --enable-fpm --with-mysqli --with-curl --with-pdo_mysql --with-pdo_sqlite --enable-mysqlnd --enable-mbstring --with-gd
```

生成二进制文件和安装依然为下述命令，时间大约为10~20分钟，耐心等待即可：

```
[root@linuxprobe php-7.3.5]# make[root@linuxprobe php-7.3.5]# make install
```

第2步：将生成出的php服务配置文件复制到安装目录中（/usr/local/php/），让其生效。主配置文件有了，接下来还需要php-fpm的配置文件，好在也已经提供，只需要复制模板即可：

```
[root@linuxprobe php-7.3.5]# cp php.ini-development /usr/local/php/lib/php.ini[root@linuxprobe php-7.3.5]# cd /usr/local/php/etc/[root@linuxprobe etc]# mv php-fpm.conf.default php-fpm.conf
```

复制一个模板文件到php-fpm.d的目录中，用于后续控制网站的连接性能：

```
[root@linuxprobe etc]# mv php-fpm.d/www.conf.default php-fpm.d/www.conf
```

第3步：把php服务加入到启动项中，使其重启后依然生效：

```
[root@linuxprobe etc]# cd /lnmp/php-7.3.5[root@linuxprobe php-7.3.5]# cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm[root@linuxprobe php-7.3.5]# chmod 755 /etc/init.d/php-fpm
```

第4步：由于php服务程序的配置参数直接会影响到Web服务服务的运行环境，如果默认开启了一些不必要且高危的功能（如允许用户在网页中执行Linux命令），则会降低网站被入侵的难度，入侵人员甚至可以拿到整台Web服务器的管理权限。因此我们需要编辑php.ini配置文件，在310行的disable_functions参数后面追加上要禁止的功能。下面的禁用功能名单是刘遄老师依据网站运行的经验而定制的，不见得适合每个生产环境，建议大家在此基础上根据自身工作需求酌情删减：

```
[root@linuxprobe php-7.3.5]# vim /usr/local/php/lib/php.ini 307 ; This directive allows you to disable certain functions for security reasons. 308 ; It receives a comma-delimited list of function names. 309 ; http://php.net/disable-functions 310 disable_functions = passthru,exec,system,chroot,chgrp,chown,shell_exec,proc_open,proc_get_status,popen,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server
```

第5步：LNMP架构源码编译工作就此结束~享受胜利成果吧：

```
[root@linuxprobe php-7.3.5]# /etc/init.d/php-fpm startStarting php-fpm done
```

##### **20.3 搭建Discuz论坛**

为了检验LNMP动态网站架构环境是否配置妥当，可以在上面部署WordPress博客系统，然后查看效果。如果能够在LNMP动态网站环境中成功安装使用WordPress网站系统，也就意味着这套架构是可用的。WordPress是一个以PHP和MySQL为平台的开源博客软件，具有丰富的插件和模板系统，截止于2021年5月，全球排名前1000万的网站中已有超过41%使用了WordPress，是当前最受欢迎的网站内容管理系统。

把Nginx服务程序根目录的内容清空后，将WordPress解压后的网站文件复制进去：

```
[root@linuxprobe php-7.3.5]# cd ..[root@linuxprobe lnmp]# rm -f /usr/local/nginx/html/*[root@linuxprobe lnmp]# tar xzvf wordpress.tar.gz [root@linuxprobe lnmp]# mv wordpress/* /usr/local/nginx/html/
```

为了能够让网站文件被Nginx服务程序顺利读取，应设置目录所有身份及可读写的权限：

```
[root@linuxprobe lnmp]# chown -Rf nginx:nginx /usr/local/nginx/html [root@linuxprobe lnmp]# chmod -Rf 777 /usr/local/nginx/html
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