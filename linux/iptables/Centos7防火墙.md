# 一、禁用自带的firewalld服务

CentOS7默认的防火墙不是iptables,而是firewalle

1、关闭firewall

```
#停止firewall
systemctl stop firewalld.service

#禁止firewall开机启动
systemctl disable firewalld.service
或
systemctl mask firewalld

#查看默认防火墙状态(not running:关闭，running:开启)
root># firewall-cmd --state 
not running
```

# 二、安装iptable iptable-service

```
#先检查是否安装了iptables
service iptables status
或
systemctl status iptables.service

#如果没安装，使用yum安装
yum install -y iptables
yum install -y iptables-services

#升级iptables
yum update iptables 

#重启iptables服务并设置为开机启动
systemctl restart iptables.service
systemctl enable iptables.service
```

# 三、通用规则设置
```
#查看iptables现有规则
iptables -L -n
#或者
iptables -S
#先允许所有,不然有可能会杯具
iptables -P INPUT ACCEPT
#清空所有默认规则
iptables -F
#清空所有自定义规则
iptables -X
#所有计数器归 0
iptables -Z
#允许来自于lo接口的数据包(本地访问)
iptables -A INPUT -i lo -j ACCEPT
#开放21端口(FTP)
iptables -A INPUT -p tcp --dport 21 -j ACCEPT
#开放22和33389端口(SSH)
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 33389 -j ACCEPT
#开放80端口(HTTP)
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#开放443端口(HTTPS)
iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#开放3306端口(mysql)
iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
#允许ping
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
#允许接受本机请求之后的返回数据 RELATED,是为FTP设置的
iptables -A INPUT -m state --state  RELATED,ESTABLISHED -j ACCEPT
#过滤所有非以上规则的请求
iptables -P INPUT DROP
#所有出站一律绿灯
iptables -P OUTPUT ACCEPT
#所有转发一律丢弃
iptables -P FORWARD DROP
```

# 四、其他规则设定

```
#使用multiport可以添加多个不连接的端口
iptables -A INPUT -p tcp -m multiport –dport 21:25,135:139 -j DROP
iptables -A INPUT  -p tcp -m multiport --dports 22,80,443 -j ACCEPT

iptables -A OUTPUT -p tcp -m multiport --sports 22,80,443 -j ACCEPT

#允许192.168.1.123访问本机的3306端口
iptables -I INPUT -s 192.168.1.123 -ptcp --dport 3306 -j ACCEPT

#单个IP的命令是
iptables -I INPUT -s 211.1.100.99 -j DROP

#封IP段的命令是
iptables -I INPUT -s 211.1.0.0/16 -j DROP
iptables -I INPUT -s 211.2.0.0/16 -j DROP
iptables -I INPUT -s 211.3.0.0/16 -j DROP

#封整个大段的命令是
iptables -I INPUT -s 211.0.0.0/8 -j DROP

#禁止指定的端口80：
iptables -A INPUT -p tcp --dport 80 -j DROP

#拒绝所有的端口：
iptables -A INPUT -j DROP

#屏蔽HTTP服务Flood攻击
有时会有用户在某个服务，例如 HTTP 80 上发起大量连接请求，此时我们可以启用如下规则：
iptables -A INPUT -p tcp --dport 80 -m limit --limit 100/minute --limit-burst 200 -j ACCEPT
上述命令会将连接限制到每分钟 100 个，上限设定为 200。
```

# 五、解封
```
解封：
iptables -L INPUT

iptables -L --line-numbers 
然后
iptables -D INPUT 序号
```

# 六、保存规则设定
```
#保存iptables规则并重启
service iptables save
systemctl restart iptables.service
```

# 七、防火墙完整设置脚本
```
#!/bin/sh
iptables -P INPUT ACCEPT
iptables -F
iptables -X
iptables -Z
iptables -A INPUT -i lo -j ACCEPT
#iptables -A INPUT -p tcp --dport 22 -j ACCEPT
#iptables -A INPUT -p tcp --dport 21 -j ACCEPT
#iptables -A INPUT -p tcp --dport 80 -j ACCEPT
#iptables -A INPUT -p tcp --dport 443 -j ACCEPT
#iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
iptables -A INPUT -p tcp --dport 33389 -j ACCEPT
iptables -I INPUT -s 192.168.52.0/24 -p tcp --dport 3306 -j ACCEPT
iptables -A INPUT  -s 192.168.52.0/24 -p tcp -m multiport --dports 8080:8090 -j ACCEPT
iptables -A INPUT  -p tcp -m multiport --dports 21:22,80,443 -j ACCEPT
iptables -A INPUT -p icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
service iptables save
systemctl restart iptables.service
```

参考资料：

https://www.cnblogs.com/kreo/p/4368811.html    CentOS7安装iptables防火墙

https://www.cnblogs.com/qtxdy/p/7724652.html   linux iptables 关闭端口和网段 

https://www.cnblogs.com/linuxprobe/p/5643684.html 20条IPTables防火墙规则用法！  
