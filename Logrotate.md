# Logrotate

## 1.简介
logrotate是一个日志文件管理工具。用来把旧文件轮转、压缩、删除，并且创建新的日志文件。我们可以根据日志文件的大小、天数等来转储，便于对日志文件管理，一般都是通过cron计划任务来完成的。

## 2.安装
```
yum install logrotate
```    
安装完成后，自动在/etc/cron.daily/下生成个logrotate脚本文件。

## 3.日志切割
vim nginx 内容如下：
```
/usr/local/nginx/logs/*.log {
#olddir /home/wwwlogs
daily   #指定转储周期为每天
missingok
rotate 7
dateext
compress    #通过gzip 压缩转储以后的日志
delaycompress   #和compress一起使用时，转储的日志文件到下一次转储时才压缩
notifempty
create 640 nginx nginx
sharedscripts
postrotate
[ -f /usr/local/nginx/logs/nginx.pid ] && kill -USR1 `cat /usr/local/nginx/logs/nginx.pid`   #不是中止Nginx的进程，而是传递给它信号重新生成日志，如果nginx没启动不做操作
endscript
}
```


## 4.配置选项说明
```
logrotate 的默认配置文件是 /etc/logrotate.conf。主要参数：

aily指定转储周期为每天 
weekly指定转储周期为每周 
monthly指定转储周期为每月 
dateext在文件末尾添加当前日期 
compress通过gzip 压缩转储以后的日志 
nocompress不需要压缩时，用这个参数 
copytruncate先把日志内容复制到旧日志文件后才清除日志文件内容，可以保证日志记录的连续性
nocopytruncate备份日志文件但是不截断 
create mode owner group转储文件，使用指定的文件模式创建新的日志文件 
nocreate不建立新的日志文件 
delaycompress和 compress 一起使用时，转储的日志文件到下一次转储时才压缩 
nodelaycompress覆盖 delaycompress 选项，转储同时压缩。 
errors address专储时的错误信息发送到指定的Email 地址 
ifempty即使是空文件也转储，这个是 logrotate 的缺省选项。 
notifempty如果是空文件的话，不转储 
mail address把转储的日志文件发送到指定的E-mail 地址 
nomail转储时不发送日志文件 
olddir directory转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统 
noolddir转储后的日志文件和当前日志文件放在同一个目录下 
rotate count指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份 
tabootext [+] list让logrotate 不转储指定扩展名的文件，缺省的扩展名是：.rpm-orig, .rpmsave, v, 和 ~ 
size size当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB (sizem). 
prerotate/endscript在转储以前需要执行的命令可以放入这个对，这两个关键字必须单独成行
postrotate/endscript在转储以后需要执行的命令可以放入这个对，这两个关键字必须单独成行
```

## 5.实例测试
```
1.计划任务
service crond status
crontab -e
59 23 * * *  /usr/sbin/logrotate -f /etc/logrotate.d/nginx

2.测试
/usr/sbin/logrotate -f /etc/logrotate.d/nginx

```