# iptables 模板
## 1.根据需求调整内核
    1.SYN缓冲
    echo "1" > /proc/sys/net/ipv4/tcp_syncookies
    2.NAT/IP转发
    echo "1" > /proc/sys/net/ipv4/ip_forward
## 2.加载iptables模块
    modprobe ip_tables
    modprobe iptable_nat
    modeprobe nf_conntrack
## 3.清空所有的表链规则    
    iptables -F
    iptables -X
    iptables -Z
    iptables -F -t nat
    iptables -X -t nat 
    iptables -Z -t nat
    iptables -X -t mangle 
## 4.定义默认策略
    iptables -P INPUT DROP
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT 
    iptables -t nat -P PREROUTING ACCEPT
    iptables -t nat -P POSTROUTING ACCEPT
    iptables -t nat -P OUTPUT ACCEPT
## 5.打开回环
    iptables -A INPUT -i lo -j ACCEPT
    iptables -A OUTPUT -o lo -j ACCEPT

## 6.允许状态为ESTABLISHED的数据包进入
    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
## 7.ssh端口
    iptables -A INPUT -p tcp  --dport 22 -j ACCEPT
## 8.启用多端口扩展
    iptables -A INPUT -p tcp -m multiport  --dport 22,80-j ACCEPT
## 9.在INPUT表和FORWARD表中拒绝所有其他不符合上述任何一条规则的数据包，并且发送一条host prohibited的消息给被拒绝的主机。
    iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
    iptables -A FORWARD -j REJECT --reject-with icmp-host-prohibited
## 10.瞬间访问过大的IP自动加到黑名单
    #!/bin/bash
    netstat -an | grep :80 | grep -v 127.0.0.1 | awk '{print $5}' | sort|awk -F: '{print $1}' | uniq -c | sort| awk '$1 >100 {print $1,$2}' > /root/deny.txt

      for i in `awk '{print $2}' /root/deny.txt`
      do
      COUNT=`grep $i /root/deny.txt | awk '{print $1}'`
      DEFINE=1000
      ZERO="0"
      if [ $COUNT -gt $DEFINE ];
        then
        grep $i /root/deny.txt > /dev/null
        if [ $? -eq $ZERO ];
          then
          echo "$COUNT $i"
          iptables -I INPUT -p tcp -s $i -j DROP
        fi
      fi
    done

## 11.防暴力破解软件
  DenyHosts