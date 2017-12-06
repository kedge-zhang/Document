# firewall

## 1.简言

firewall daemon 动态管理防火墙，不需要重启整个防火墙便可应用更改。因而也就没有必要重载所有内核防火墙模块了。不过，要使用 firewall daemon 就要求防火墙的所有变更都要通过该守护进程来实现，以确保守护进程中的状态和内核里的防火墙是一致的。另外，firewall daemon 无法解析由 iptables 和 ebtables 命令行工具添加的防火墙规则。

## 2.基础命令
```
# 是否安装
rpm -qa | grep firewalld
# 命令安装
yum install firewalld -y
# 开启服务
systemctl start firewalld.service
# 关闭服务
systemctl stop firewalld.service
# 开机自启
systemctl enable firewalld.service
# 禁止自启
systemctl disable firewalld.service
# 查看状态
systemctl status firewalld
```
## 3.区域管理
```
#查看状态
firewall-cmd --state

#查看默认区域    
firewall-cmd  --get-default-zone

#查看现在的区域设定
firewall-cmd  --list-all

#查看定义的所有区域。
firewall-cmd --list-all-zones

#查看指定区域（如external），所允许的服务。
firewall-cmd --list-service --zone=external

#变更默认区域（如external）
firewall-cmd --set-default-zone=external
```
## 4.服务管理
### 4.1 查看服务及位置
```bash
#确认所定义的服务一览。
firewall-cmd --get-services
#所定义的服务保存目录。
ls /usr/lib/firewalld/services
```
### 4.2 添加/删除服务
```
1.添加http服务（设定即时有效）
# firewall-cmd --add-service=http
success
# firewall-cmd --list-service
http ssh

2.删除hhtp服务
# firewall-cmd --add-service=http --permanent
success
# firewall-cmd --reload
success
# firewall-cmd --list-service
http ssh

3.永久添加http服务（为使设定生效，需重启firewalld）。
# firewall-cmd --add-service=http --permanent
success
# firewall-cmd --reload
success
# firewall-cmd --list-service
http ssh
```

### 4.3 添加/删除端口
```
1.添加888端口。
# firewall-cmd --add-port=888/tcp
success
# firewall-cmd --list-port
888/tcp
2.删除888端口。
# firewall-cmd --remove-port=888/tcp
Success
# firewall-cmd --list-port
3.永远追加888端口（为使设定生效，需重firewall-cmd ）。
# firewall-cmd --add-port=888/tcp --permanent
success
# firewall-cmd --reload
success
# firewall-cmd --list-port
888/tcp
```