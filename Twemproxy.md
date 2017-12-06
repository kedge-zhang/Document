# Redis+Twemproxy
[参考链接 Redis+Twemproxy](http://www.cnblogs.com/haoxinyue/p/redis.html)

![架构](http://i.imgur.com/OShlOdg.png)
##1安装
```bash      
yum install gcc gcc-c++ tcl -y &&  cd /usr/local/src/ && wget http://download.redis.io/releases/redis-3.2.8.tar.gz && tar fvx redis-3.2.8.tar.gz -C /usr/local &&  cd /usr/local/redis-3.2.8/ && make MALLOC=libc && make test && make install && cd utils && ./install_server.sh 
```
##2 Error
* `*** [err]: Test replication partial resync: no backlog in tests/integration/replication-psync.tcl; Expected condition '[s -1 sync_partial_err] > 0' to be true ([s -1 sync_partial_err] > 0)`
解决办法  
* `vi tests/integration/replication-psync.tcl ; 把对应报错的那段代码中的 after后面的数字，从100改成 500`

##3 安装  
* 此步慎重，若是automake，autoconf,libtool没有安装则跳过此步 
*  `rm -rf /usr/local/bin/auto* &&  rm -rf /usr/bin/auto* &&  rm -rf /usr/bin/libtool* && rm -rf /usr/bin/aclocal* && rm -rf /usr/local/bin/ifnames`

###3.1 安装autoconf
* `cd /usr/local/src/ && wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz && tar fvx autoconf-2.69.tar.gz && cd autoconf-2.69 && ./configure --prefix=/usr/local/autoconf-2.69 && make && make install && ln -s /usr/local/autoconf-2.69/bin/* /usr/local/bin && ln -s /usr/local/autoconf-2.69/bin/* /usr/bin `

###3.2 安装automake
* `cd /usr/local/src/ && wget http://ftp.gnu.org/gnu/automake/automake-1.15.tar.gz && tar fvx automake-1.15.tar.gz && cd automake-1.15  &&  ./configure --prefix=/usr/local/automake-1.15 && make && make install && ln -s /usr/local/automake-1.15/bin/* /usr/bin/` 

###3.3 安装libtool
* `cd /usr/local/src/ && wget http://ftp.gnu.org/gnu/libtool/libtool-2.4.tar.gz && tar fvx  libtool-2.4.tar.gz && cd libtool-2.4 && ./configure --prefix=/usr/local/libtool-2.4 && make && make install && ln -s /usr/local/libtool-2.4/bin/* /usr/bin/`

###3.4 安装twemproxy
* `cd /usr/local/src/ && git clone git://github.com/twitter/twemproxy.git && cd twemproxy && autoreconf -fvi && yum -y install  libtool autoconf automake && autoreconf -fvi && ./configure --prefix=/usr/local/twemproxy --enable-debug=log && make && make install `
*  `mkdir /usr/local/twemproxy/{etc,logs} && cp /usr/local/src/twemproxy/conf/nutcracker.yml  /usr/local/twemproxy/etc`

`###################nutcracker.yml##################`
* vim nutcracker.yml
```bash
alpha:  
  listen: 0.0.0.0:22121  
  hash: fnv1a_64  
  distribution: ketama  
  auto_eject_hosts: true  
  redis: true  
  server_retry_timeout: 2000  
  server_failure_limit: 1  
  servers:  
   - 127.0.0.1:6379:1  
   - 10.141.160.43:6379:1
```

`####################启动################################`
* ./sbin/nutcracker -d -c /usr/local/twemproxy/etc/nutcracker.yml -o logs/nutcracker.log  
  
##4 性能测试
###4.1 set测试
* 通过twemproxy测试  
```
redis-benchmark -h 10.66.227.147 -p 22121 -c 100 -t set -d 100 -l -q
```
* 2.直接对后端redis测试
```
redis-benchmark -h 10.66.227.147 -p 6379 -c 100 -t set -d 100 -l -q
```

###4.2 get测试
* 通过twemproxy测试
```
redis-benchmark -h 10.66.227.147 -p 22121 -c 100 -t get -d 100 -l -q
```
* 直接对后端redis测试
```
redis-benchmark -h 10.66.227.147 -p 22121 -c 100 -t get -d 100 -l -q
```
###4.3 查看键值分布
* redis-cli info|grep db0