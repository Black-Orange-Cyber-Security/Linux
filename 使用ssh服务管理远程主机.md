## 使用ssh服务管理远程主机

- 配置网卡服务
  - [9.1.1 配置网卡参数](https://www.linuxprobe.com/basic-learning-09.html#911)
  
    ```shell
    [root@localhost msfconsole]# nmtui
    ```
  
    ```shell
    [root@localhost msfconsole]# nmtui -h
    用法：
      nmtui [OPTION…]
      nmtui
      nmtui edit [连接]
      nmtui connect [连接]
      nmtui hostname [新主机名]
    ```
  
    ![屏幕截图 2021-05-21 101656](C:\Users\guofengli\Desktop\计算机网络编程\Linux就该这么学（RHEL8）\屏幕截图 2021-05-21 101656.png)
  
    ![](C:\Users\guofengli\Desktop\计算机网络编程\Linux就该这么学（RHEL8）\屏幕截图 2021-05-21 101802.png)
  
    ![](C:\Users\guofengli\Desktop\计算机网络编程\Linux就该这么学（RHEL8）\图片\03.png)
  
  ![](C:\Users\guofengli\Desktop\计算机网络编程\Linux就该这么学（RHEL8）\图片\04.png)
  
  ![](C:\Users\guofengli\Desktop\计算机网络编程\Linux就该这么学（RHEL8）\图片\05.png)
  
  ```shell
  [root@localhost msfconsole]# vim /etc/sysconfig/network-scripts//ifcfg-ens160 
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
  NAME=ens160
  UUID=dbe5d4a2-4b2b-465b-84a0-f840bc373311
  DEVICE=ens160
  ONBOOT=yes
  IPADDR=192.168.10.10
  PREFIX=24
  
  ```
  
  ```shell
  [root@localhost msfconsole]# nmcli -h
  Usage: nmcli [OPTIONS] OBJECT { COMMAND | help }
  
  OPTIONS
    -a, --ask                                ask for missing parameters
    -c, --colors auto|yes|no                 whether to use colors in output
    -e, --escape yes|no                      escape columns separators in values
    -f, --fields <field,...>|all|common      specify fields to output
    -g, --get-values <field,...>|all|common  shortcut for -m tabular -t -f
    -h, --help                               print this help
    -m, --mode tabular|multiline             output mode
    -o, --overview                           overview mode
    -p, --pretty                             pretty output
    -s, --show-secrets                       allow displaying passwords
    -t, --terse                              terse output
    -v, --version                            show program version
    -w, --wait <seconds>                     set timeout waiting for finishing operations
  
  OBJECT
    g[eneral]       NetworkManager's general status and operations
    n[etworking]    overall networking control
    r[adio]         NetworkManager radio switches
    c[onnection]    NetworkManager's connections
    d[evice]        devices managed by NetworkManager
    a[gent]         NetworkManager secret agent or polkit agent
    m[onitor]       monitor NetworkManager changes
  
  
  [root@localhost msfconsole]# nmcli connection -h
  Usage: nmcli connection { COMMAND | help }
  
  COMMAND := { show | up | down | add | modify | clone | edit | delete | monitor | reload | load | import | export }
  
    show [--active] [--order <order spec>]
    show [--active] [id | uuid | path | apath] <ID> ...
  
    up [[id | uuid | path] <ID>] [ifname <ifname>] [ap <BSSID>] [passwd-file <file with passwords>]
  
    down [id | uuid | path | apath] <ID> ...
  
    add COMMON_OPTIONS TYPE_SPECIFIC_OPTIONS SLAVE_OPTIONS IP_OPTIONS [-- ([+|-]<setting>.<property> <value>)+]
  
    modify [--temporary] [id | uuid | path] <ID> ([+|-]<setting>.<property> <value>)+
  
    clone [--temporary] [id | uuid | path ] <ID> <new name>
  
    edit [id | uuid | path] <ID>
    edit [type <new_con_type>] [con-name <new_con_name>]
  
    delete [id | uuid | path] <ID>
  
    monitor [id | uuid | path] <ID> ...
  
    reload
  
    load <filename> [ <filename>... ]
  
    import [--temporary] type <type> file <file to import>
  
    export [id | uuid | path] <ID> [<output file>]
  
  [root@localhost msfconsole]# 
  ```
  
  手动重启相应的服务
  
  ```shell
  [root@localhost msfconsole]# nmcli connection reload ens160
  [root@localhost msfconsole]# nmcli connection up ens160
  连接已成功激活（D-Bus 活动路径：/org/freedesktop/NetworkManager/ActiveConnection/4）
  [root@localhost msfconsole]# ping 192.168.10.10
  PING 192.168.10.10 (192.168.10.10) 56(84) bytes of data.
  64 bytes from 192.168.10.10: icmp_seq=1 ttl=64 time=0.208 ms
  64 bytes from 192.168.10.10: icmp_seq=2 ttl=64 time=0.064 ms
  64 bytes from 192.168.10.10: icmp_seq=3 ttl=64 time=0.108 ms
  64 bytes from 192.168.10.10: icmp_seq=4 ttl=64 time=0.108 ms
  ^C
  --- 192.168.10.10 ping statistics ---
  4 packets transmitted, 4 received, 0% packet loss, time 3074ms
  rtt min/avg/max/mdev = 0.064/0.122/0.208/0.052 ms
  [root@localhost msfconsole]# 
  ```
  
  - [9.1.2 创建网络会话](https://www.linuxprobe.com/basic-learning-09.html#912)
  
    RHEL和[CentOS](https://www.linuxprobe.com/)系统默认使用NetworkManager来提供网络服务，这是一种动态管理网络配置的守护进程，能够让网络设备保持连接状态。可以使用nmcli命令来管理NetworkManager服务程序。nmcli是一款基于命令行的网络配置工具，功能丰富，参数众多。它可以轻松地查看网络信息或网络状态：
  
    ```shell
    [root@localhost msfconsole]# nmcli connection show
    NAME    UUID                                  TYPE      DEVICE 
    ens160  dbe5d4a2-4b2b-465b-84a0-f840bc373311  ethernet  ens160 
    virbr0  0052d090-8f33-49cf-a170-d3828f3dde9e  bridge    virbr0 
    [root@localhost msfconsole]# nmcli connection show ens160
    connection.id:                          ens160
    connection.uuid:                        dbe5d4a2-4b2b-465b-84a0-f840bc373311
    connection.stable-id:                   --
    connection.type:                        802-3-ethernet
    connection.interface-name:              ens160
    connection.autoconnect:                 是
    connection.autoconnect-priority:        0
    connection.autoconnect-retries:         -1 (default)
    connection.multi-connect:               0（default）
    connection.auth-retries:                -1
    connection.timestamp:                   1621593127
    connection.read-only:                   否
    connection.permissions:                 --
    connection.zone:                        --
    connection.master:                      --
    connection.slave-type:                  --
    connection.autoconnect-slaves:          -1（default）
    connection.secondaries:                 --
    connection.gateway-ping-timeout:        0
    connection.metered:                     未知
    connection.lldp:                        default
    connection.mdns:                        -1（default）
    connection.llmnr:                       -1（default）
    connection.wait-device-timeout:         -1
    802-3-ethernet.port:                    --
    802-3-ethernet.speed:                   0
    802-3-ethernet.duplex:                  --
    802-3-ethernet.auto-negotiate:          否
    802-3-ethernet.mac-address:             --
    802-3-ethernet.cloned-mac-address:      --
    802-3-ethernet.generate-mac-address-mask:--
    802-3-ethernet.mac-address-blacklist:   --
    802-3-ethernet.mtu:                     自动
    802-3-ethernet.s390-subchannels:        --
    802-3-ethernet.s390-nettype:            --
    802-3-ethernet.s390-options:            --
    802-3-ethernet.wake-on-lan:             default
    802-3-ethernet.wake-on-lan-password:    --
    ipv4.method:                            manual
    ipv4.dns:                               --
    ipv4.dns-search:                        --
    ipv4.dns-options:                       --
    ipv4.dns-priority:                      0
    ipv4.addresses:                         192.168.10.10/24
    ipv4.gateway:                           --
    ipv4.routes:                            --
    ipv4.route-metric:                      -1
    ipv4.route-table:                       0 (unspec)
    ipv4.routing-rules:                     --
    ipv4.ignore-auto-routes:                否
    ipv4.ignore-auto-dns:                   否
    ipv4.dhcp-client-id:                    --
    ipv4.dhcp-iaid:                         --
    ipv4.dhcp-timeout:                      0 (default)
    ipv4.dhcp-send-hostname:                是
    ipv4.dhcp-hostname:                     --
    ipv4.dhcp-fqdn:                         --
    ipv4.dhcp-hostname-flags:               0x0（none）
    ipv4.never-default:                     否
    ipv4.may-fail:                          是
    ipv4.dad-timeout:                       -1 (default)
    ipv4.dhcp-vendor-class-identifier:      --
    ipv4.dhcp-reject-servers:               --
    ipv6.method:                            auto
    ipv6.dns:                               --
    ipv6.dns-search:                        --
    ipv6.dns-options:                       --
    ipv6.dns-priority:                      0
    ipv6.addresses:                         --
    ipv6.gateway:                           --
    ipv6.routes:                            --
    ipv6.route-metric:                      -1
    ipv6.route-table:                       0 (unspec)
    ipv6.routing-rules:                     --
    ipv6.ignore-auto-routes:                否
    ipv6.ignore-auto-dns:                   否
    ipv6.never-default:                     否
    ipv6.may-fail:                          是
    ipv6.ip6-privacy:                       -1（unknown）
    ipv6.addr-gen-mode:                     eui64
    ipv6.ra-timeout:                        0 (default)
    ipv6.dhcp-duid:                         --
    ipv6.dhcp-iaid:                         --
    ipv6.dhcp-timeout:                      0 (default)
    ipv6.dhcp-send-hostname:                是
    ipv6.dhcp-hostname:                     --
    ipv6.dhcp-hostname-flags:               0x0（none）
    ipv6.token:                             --
    proxy.method:                           none
    proxy.browser-only:                     否
    proxy.pac-url:                          --
    proxy.pac-script:                       --
    GENERAL.NAME:                           ens160
    GENERAL.UUID:                           dbe5d4a2-4b2b-465b-84a0-f840bc373311
    GENERAL.DEVICES:                        ens160
    GENERAL.IP-IFACE:                       ens160
    GENERAL.STATE:                          已激活
    GENERAL.DEFAULT:                        否
    GENERAL.DEFAULT6:                       否
    GENERAL.SPEC-OBJECT:                    --
    GENERAL.VPN:                            否
    GENERAL.DBUS-PATH:                      /org/freedesktop/NetworkManager/ActiveC>
    GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/Setting>
    GENERAL.ZONE:                           --
    GENERAL.MASTER-PATH:                    --
    IP4.ADDRESS[1]:                         192.168.10.10/24
    IP4.GATEWAY:                            --
    IP4.ROUTE[1]:                           dst = 192.168.10.0/24, nh = 0.0.0.0, mt>
    IP6.ADDRESS[1]:                         fe80::20c:29ff:fe13:d5ea/64
    IP6.GATEWAY:                            --
    IP6.ROUTE[1]:                           dst = fe80::/64, nh = ::, mt = 100
    IP6.ROUTE[2]:                           dst = ff00::/8, nh = ::, mt = 256, tabl>
    lines 90-112/112 (END)
    ```
  
    另外，RHEL 8系统支持网络会话功能，允许用户在多个配置文件中快速切换（非常类似于firewalld防火墙服务中的区域技术）。如果在公司网络中使用笔记本电脑时需要手动指定网络的IP地址，而回到家中则是使用DHCP自动分配IP地址。这就需要麻烦地频繁修改IP地址，但是使用了网络会话功能后一切就简单多了——只需在不同的使用环境中激活相应的网络会话，就可以实现网络配置信息的自动切换了。
  
    使用nmcli命令并按照“connection add con-name type ifname”的格式来创建网络会话。假设将公司网络中的网络会话称之为company，将家庭网络中的网络会话称之为house，现在依次创建各自的网络会话。
  
    使用con-name参数指定公司所使用的网络会话名称company，然后依次用ifname参数指定本机的网卡名称（千万要以实际环境为准，不要照抄书上的ens160），用autoconnect no参数设置该网络会话默认不被自动激活，以及用ip4及gw4参数手动指定网络的IP地址：
  
    ```
    
    ```
  
    
  
  - [9.1.3 绑定两块网卡](https://www.linuxprobe.com/basic-learning-09.html#913)
  
- 9.2 远程控制服务
  - [9.2.1 配置sshd服务](https://www.linuxprobe.com/basic-learning-09.html#921_sshd)
  - [9.2.2 安全密钥验证](https://www.linuxprobe.com/basic-learning-09.html#922)
  - [9.2.3 远程传输命令](https://www.linuxprobe.com/basic-learning-09.html#923)
  
- 9.3 不间断会话服务
  - [9.3.1 管理远程会话](https://www.linuxprobe.com/basic-learning-09.html#931)
  - [9.3.2 管理多窗格](https://www.linuxprobe.com/basic-learning-09.html#932)
  - [9.3.3 会话共享功能](https://www.linuxprobe.com/basic-learning-09.html#933)
  
  z
  
  9.4 检索日志信息](https://www.linuxprobe.com/basic-learning-09.html#94)