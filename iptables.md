---
title: iptables命令
date: 2017-7-31 16:17:41
tags: Linux
---

    iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作

表名包括：

- raw：高级功能，如：网址过滤。
- mangle：数据包修改（QOS），用于实现服务质量。
- net：地址转换，用于网关路由器。
- filter：包过滤，用于防火墙规则。

规则链名包括：

- INPUT链：处理输入数据包。
- OUTPUT链：处理输出数据包。
- PORWARD链：处理转发数据包。
- PREROUTING链：用于目标地址转换（DNAT）。
- POSTOUTING链：用于源地址转换（SNAT）。

<!-- more -->

动作包括：

- accept：接收数据包。
- DROP：丢弃数据包。
- REDIRECT：重定向、映射、透明代理。
- SNAT：源地址转换。
- DNAT：目标地址转换。
- MASQUERADE：IP伪装（NAT），用于ADSL。
- LOG：日志记录。

常用实例

清除已有iptables规则

    iptables -F 
    iptables -X 
    iptables -Z

开放指定的端口

    iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT #允许本地回环接口(即运行本机访问本机) 
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT #允许已建立的或相关连的通行 
    iptables -A OUTPUT -j ACCEPT #允许所有本机向外的访问 
    iptables -A INPUT -p tcp --dport 22 -j ACCEPT #允许访问22端口 
    iptables -A INPUT -p tcp --dport 80 -j ACCEPT #允许访问80端口 
    iptables -A INPUT -p tcp --dport 21 -j ACCEPT #允许ftp服务的21端口 
    iptables -A INPUT -p tcp --dport 20 -j ACCEPT #允许FTP服务的20端口 
    iptables -A INPUT -j reject #禁止其他未允许的规则访问 
    iptables -A FORWARD -j REJECT #禁止其他未允许的规则访问

屏蔽IP

    iptables -I INPUT -s 192.168.6.7 -j DROP #屏蔽单个IP的命令 
    iptables -I INPUT -s 192.0.0.0/8 -j DROP #封整个段即从192.0.0.1到192.255.255.254的命令 
    iptables -I INPUT -s 192.168.0.0/16 -j DROP #封IP段即从192.168.0.1到1192.168.255.254的命令 
    iptables -I INPUT -s 192.168.0.0/24 -j DROP #封IP段即从192.168.0.1到192.168.0.254的命令

查看已添加的iptables规则
    
    iptables -L -n -v

删除已添加的iptables规则

	iptables -L -n --line-numbers //将所有iptables以序号标记显示，执行
	iptables -D INPUT 8           //删除INPUT里序号为8的规则

##iptables常用命令

	service iptables start #启动
    service iptables restart #重启
    service iptables save #保存
    service iptables stop #停止
    service iptables status #查询状态
    
    iptables -L -v #建议查看表时，带上-v参数，会显示in out 方向等信息，显示的信息更完整、详细。
    iptables -A INPUT -i eth0 -s 192.168.0.0/16 -j ACCEPT  #针对INPUT链增加一条规则，接收从eth0口进入且源地址为192.168.0.0/16网段发往本机的数据
    iptables -D #删除规则，命令后可接序号
    iptables -F #所有规则都清除掉
    iptables-save >/home/pi/iptables.bak #备份规则到指定目录的指定文件
    iptables-restore </home/pi/iptables.bak #恢复指定文件的规则
    service iptables save #配置保存到/etc/sysconfig/iptables

需要注意: 直接在/etc/sysconfig/iptables 文件中编辑的话 需要添加到这两句上面，我就是犯了这么个错误，iptables规则一直不生效。

	-A INPUT -j REJECT --reject-with icmp-host-prohibited
	-A FORWARD -j REJECT --reject-with icmp-host-prohibited

