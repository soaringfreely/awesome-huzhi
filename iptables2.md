## iptables

#### iptables 中的表

iptables 包含 5 张表（tables）:

* `raw` 用于配置数据包，raw 中的数据包不会被系统跟踪。

* `filter` 用于存放所有与防火墙相关操作的默认表。

* `nat` 用于 网络地址转换（例如：端口转发）。

* `mangle` 用于对特定数据包的修改（参考 损坏数据包）。

* `security` 用于强制访问控制网络规则（例如： SELinux）。

notes:

大部分情况仅需要使用 filter 和 nat。其他表用于更复杂的情况——包括多路由和路由判


#### iptables 常见用法

##### 查看规则

```bash
# 通过 -t 指定要查看的表
$ iptables-save -t filter
$ iptables-save -t nat
$ iptables-save

$ iptables -t filter -nvL --line-number
$ iptables -t nat -nvL --line-number

```

##### 添加规则

```bash
# 在位置 7 添加一条规则
$ iptables -t filter -I INPUT 7 -i eth0 -p tcp -m tcp --dport 80 -j ACCEPT
-t 指定要添加表
-I 添加操作
INPUT 指定要添加的链
7 添加的规则在链中的位置
-i eth0 -p tcp -m tcp --dport 80 -j ACCEPT 添加的规则

```

##### 删除规则

```bash
# 删除第一条规则
$ iptables -t filter -D INPUT 1
```

##### 修改规则（修改规则的最佳做法是先删除该条规则，在重新添加）

```bash
# 将第七条规则修改为 drop
$ iptables -t filter -R INPUT 7 -j DROP

# 将第七条规则修改为 accept
$ iptables -t filter -R INPUT 7 -j ACCEPT

```


##### 创建自定义链

```bash
# 在 filter 表中创建自定义的链 DOCKER
$ iptables -t filter -N DOCKER

```

##### 使用自定义链

使用方法：
1. 为自定义链添加规则
2. 在默认链中通过 `-j` 参数引用自定义链
3. 引用后会按照自定义链中的规则处理数据包

```bash
# 在 INPUT 链中引用自定义的 INPUT_direct 链
-A INPUT -j INPUT_direct

# 在 FORWARD 链中引用自定义的 DOCKER 链
-A FORWARD -o br-12f5421c9cfd -j DOCKER
```


##### 在 iptables 规则中插入日志

1. 在需要写入日志的位置添加规则
2. 开启系统日志或者开启iptables的独立日志

```bash
# 第一步添加日志规则，和添加普通规则的方法一样，只是-j参数不同
$ iptables -I INPUT 1 -j LOG --log-prefix "iptables"
$ iptables -I FORWARD 1 -j LOG --log-prefix "iptables"

# 开启系统日志或者开启iptables的独立日志
不同的Linux发行版操作方法不同

```

#### ebtables 常见用法

```bash
# 查看规则
ebtables-save
ebtables --list

# 添加规则
$ ebtables -t broute -A BROUTING    -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:broute:BROUTING" -j ACCEPT
$ ebtables -t nat    -A OUTPUT      -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:nat:OUTPUT"  -j ACCEPT
$ ebtables -t nat    -A PREROUTING  -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:nat:PREROUTING" -j ACCEPT
$ ebtables -t filter -A INPUT       -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:filter:INPUT" -j ACCEPT
$ ebtables -t filter -A FORWARD     -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:filter:FORWARD" -j ACCEPT
$ ebtables -t filter -A OUTPUT      -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:filter:OUTPUT" -j ACCEPT
$ ebtables -t nat    -A POSTROUTING -p ipv4 --ip-proto 1 --log-level 6 --log-ip --log-prefix "TRACE: eb:nat:POSTROUTING" -j ACCEPT

```































#### Docker 设置 iptables

```bash
# 没有安装启动dockerd进程之前，没有任何 iptables 规则
[root@hz1 ~]# iptables -nL --line-numbers -v
Chain INPUT (policy ACCEPT 8079 packets, 10M bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 8643 packets, 1392K bytes)
num   pkts bytes target     prot opt in     out     source               destination         
[root@hz1 ~]# iptables-save
# Generated by iptables-save v1.4.21 on Sat Oct 27 12:43:15 2018
*filter
:INPUT ACCEPT [8227:10467162]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [8819:1436050]
COMMIT
# Completed on Sat Oct 27 12:43:15 2018
[root@hz1 ~]# 

[root@hz1 ~]# ebtables-save
# Generated by ebtables-save v1.0 on 2018年 10月 27日 星期六 12:43:44 CST
[root@hz1 ~]# 
[root@hz1 ~]# ebtables --list
Bridge table: filter

Bridge chain: INPUT, entries: 0, policy: ACCEPT

Bridge chain: FORWARD, entries: 0, policy: ACCEPT

Bridge chain: OUTPUT, entries: 0, policy: ACCEPT
[root@hz1 ~]# 


#########################################3


[root@hz1 ~]# iptables -nL --line-numbers -v
Chain INPUT (policy ACCEPT 777 packets, 209K bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
2        0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
3        0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
4        0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
5        0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
6        0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 875 packets, 246K bytes)
num   pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
num   pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
2        0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
2        0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
num   pkts bytes target     prot opt in     out     source               destination         
1        0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
[root@hz1 ~]# 
[root@hz1 ~]# 
[root@hz1 ~]# iptables-save
# Generated by iptables-save v1.4.21 on Sat Oct 27 12:48:09 2018
*nat
:PREROUTING ACCEPT [4:260]
:INPUT ACCEPT [4:260]
:OUTPUT ACCEPT [18:5915]
:POSTROUTING ACCEPT [18:5915]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
COMMIT
# Completed on Sat Oct 27 12:48:09 2018
# Generated by iptables-save v1.4.21 on Sat Oct 27 12:48:09 2018
*filter
:INPUT ACCEPT [1274:359824]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [1499:416173]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
COMMIT
# Completed on Sat Oct 27 12:48:09 2018
[root@hz1 ~]# 
[root@hz1 ~]# ebtables --list
Bridge table: filter

Bridge chain: INPUT, entries: 0, policy: ACCEPT

Bridge chain: FORWARD, entries: 0, policy: ACCEPT

Bridge chain: OUTPUT, entries: 0, policy: ACCEPT
[root@hz1 ~]# ebtables-save
# Generated by ebtables-save v1.0 on 2018年 10月 27日 星期六 12:49:14 CST
*filter
:INPUT ACCEPT
:FORWARD ACCEPT
:OUTPUT ACCEPT

[root@hz1 ~]# 


```



```bash
[root@hz1 ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 28:6e:d4:88:c6:39 brd ff:ff:ff:ff:ff:ff
    inet 172.16.24.11/24 brd 172.16.24.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::2a6e:d4ff:fe88:c639/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:dd:49:66:e7 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:ddff:fe49:66e7/64 scope link 
       valid_lft forever preferred_lft forever
7: vethba3c6fb@if6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether f6:31:3d:bb:e2:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::f431:3dff:febb:e2f4/64 scope link 
       valid_lft forever preferred_lft forever
9: vetha75dc99@if8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP 
    link/ether 2e:67:fa:a7:7b:94 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::2c67:faff:fea7:7b94/64 scope link 
       valid_lft forever preferred_lft forever
[root@hz1 ~]# 

[root@hz1 ~]# docker exec c1 ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
6: eth0@if7: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@hz1 ~]# 
[root@hz1 ~]# docker exec c2 ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.3/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
[root@hz1 ~]# 
[root@hz1 ~]# docker exec c1 route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
[root@hz1 ~]# 
[root@hz1 ~]# docker exec c2 route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 eth0
[root@hz1 ~]# 
[root@hz1 ~]# 
[root@hz1 ~]#  route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.16.24.1     0.0.0.0         UG    100    0        0 eth0
172.16.24.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
[root@hz1 ~]# 


```


```bash
[root@hz1 ~]# iptables -t raw -A OUTPUT -p icmp -j TRACE
[root@hz1 ~]# iptables -t raw -A PREROUTING -p icmp -j TRACE
[root@hz1 ~]# 
[root@hz1 ~]# iptables-save
# Generated by iptables-save v1.4.21 on Sat Oct 27 13:59:48 2018
*raw
:PREROUTING ACCEPT [477:120161]
:OUTPUT ACCEPT [602:162780]
-A PREROUTING -p icmp -j TRACE
-A OUTPUT -p icmp -j TRACE
COMMIT
# Completed on Sat Oct 27 13:59:48 2018
# Generated by iptables-save v1.4.21 on Sat Oct 27 13:59:48 2018
*nat
:PREROUTING ACCEPT [17:997]
:INPUT ACCEPT [11:624]
:OUTPUT ACCEPT [143:16252]
:POSTROUTING ACCEPT [143:16252]
:DOCKER - [0:0]
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
-A DOCKER -i docker0 -j RETURN
COMMIT
# Completed on Sat Oct 27 13:59:48 2018
# Generated by iptables-save v1.4.21 on Sat Oct 27 13:59:48 2018
*filter
:INPUT ACCEPT [106090:85919254]
:FORWARD DROP [0:0]
:OUTPUT ACCEPT [123254:23975398]
:DOCKER - [0:0]
:DOCKER-ISOLATION-STAGE-1 - [0:0]
:DOCKER-ISOLATION-STAGE-2 - [0:0]
:DOCKER-USER - [0:0]
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT
-A DOCKER-ISOLATION-STAGE-1 -i docker0 ! -o docker0 -j DOCKER-ISOLATION-STAGE-2
-A DOCKER-ISOLATION-STAGE-1 -j RETURN
-A DOCKER-ISOLATION-STAGE-2 -o docker0 -j DROP
-A DOCKER-ISOLATION-STAGE-2 -j RETURN
-A DOCKER-USER -j RETURN
COMMIT
# Completed on Sat Oct 27 13:59:48 2018
[root@hz1 ~]# 
[root@hz1 ~]# 
[root@hz1 ~]# 
[root@hz1 ~]# 
[root@hz1 ~]# ebtables-save
# Generated by ebtables-save v1.0 on 2018年 10月 27日 星期六 14:00:36 CST
*filter
:INPUT ACCEPT
:FORWARD ACCEPT
:OUTPUT ACCEPT

[root@hz1 ~]# 

```




