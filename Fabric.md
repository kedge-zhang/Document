# Fabric

## 1.Python  环境升级
```
python2.6（默认） 或 Python2.7
1. 下载Python 安装包
wget https://www.python.org/ftp/python/2.7.6/Python-2.7.6.tgz  
2. 解压
tar -xvf Python-2.7.6.tgz -C /usr/src
3. 安装
cd  /usr/src/Python-2.7.6
./configure --prefix=/usr/local/python2.7.6
make && make
4. 创建链接来使系统默认python变为python2.7
ln -s /usr/local/python2.7.6/bin/python2.7 /usr/bin
5. 查看Python版本
python –V
6.修改yum配置（否则yum无法正常运行）
vi /usr/bin/yum

将第一行的#!/usr/bin/python修改为系统原有的python版本地址#!/usr/bin/python2.6
至此CentOS6.3系统Python已成功升级至2.7.6版本。
```
## 2.Fabric部署
```
yum -y install make gcc gcc-c++ gcc++ python-devel python-pip python-setuptools
***********************
1.pip安装
pip install fabric
***********************
2.源码安装    Fabric1.7依赖的paramiko需要1.10.0及以上，故先下载最新的paramiko-1.11.0.tar.gz
tar zxvf paramiko-1.11.0.tar.gz -C  /usr/src
cd  /usr/src/paramiko-1.17.2
python setup.py install

tar zxvf Fabric-1.7.0.tar.gz  -C  /usr/src
cd /usr/src/Fabric-1.7.0
python setup.py install
```
## 3.Fabric的核心 API 主要有 7 类

Fabric的核心 API 主要有 7 类：带颜色输出类（color output)、上下文管理类（context managers）、装饰器类（decorators）、网络类（network）、操作类（operations）、任务类（task）、工具类（utils）

## 4.常用命令
```
local： 执行本地命令，如 locla('uname -s').
lcd: 本地切换目录，如 lcd('/home').
cd:切换远程目录，如 cd('data/logs')
run:执行远程命令，如 run('free -m')
sudo:以sudo方式执行远程命令，如 sudo('/etc/init.d/https start')
put:上传本地文件到远程主机，如 put('/home/user.info','/data/user.info')
get:从远程主机下载文件到本地，如 get( '/home/user.info','/data/user.info')
prompt:获得用户输入信息，如 prompt('please input user password: ')
confirm:获得提示信息确认， 如confrim('Test faild,Continue[Y/N]')
@runs_once: 函数修饰符。表示词修饰符的函数只会执行一次，不受多台主机影响
@task：函数修饰符
```
## 5.例子
### 部署nginx
```
#!/usr/bin/python26
#coding:utf-8
from fabric.colors import *
from fabric.api import *

env.user = "root"
env.hosts = "172.16.0.146"
env.password = "redhat"
#env.hosts = ["172.16.0.*","172.16.0.#]
#env.passwords = {
#    "root@172.16.0.*:22": "redhat",
#    "root@172.16.0.*:22": "redhat"
#}

nginx_package="/home/nginx/tengine-2.2.0.tar.gz"
pcre_package="/home/nginx/pcre-8.38.tar.gz"
openssl_package="/home/nginx/openssl-1.0.2j.tar.gz"
start_init="/home/nginx/nginx"
rpackage="/usr/local/src"
@task
def install_gcc():
    print yellow("########Install gcc ########")
    run("yum install gcc gcc-c++ zlib zlib-devel -y")
@task
def put_shell():
    put("/home/nginx/nginx.sh","/etc/init.d/nginx")
@task
def put_task():
    print yellow("######Put package####")
    result = put(nginx_package, rpackage)
    result = put(pcre_package, rpackage)
    result = put(openssl_package, rpackage)
@task
def install_pcre():
    print yellow("######Install pcre######")
    with cd("/usr/local/src/"):
         run("tar -zxvf pcre-8.38.tar.gz -C /usr/src")
         with cd ("/usr/src/pcre-8.38"):
              run("./configure --prefix=/usr/local/pcre8.38&& make && make install")
@task
def install_opensll():
    print yellow("######Install opensll######")
    with cd("/usr/local/src/"):
         run("tar -zxvf openssl-1.0.2j.tar.gz -C /usr/src")
         with cd ("/usr/src/openssl-1.0.2j"):
              run("./config --prefix=/usr/local/ssl && make && make install")
@task
def install_nginx():
    print yellow("######install nginx######")
    with cd("/usr/local/src/"):
         run("tar fvx tengine-2.2.0.tar.gz -C /usr/src/")
         with cd("/usr/src/tengine-2.2.0"):
              run("groupadd www && useradd -g www www -s /sbin/nologin")
              run("./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/etc/nginx/nginx.conf --error-log-path=/usr/local/nginx/logs/error.log --http-log-path=/usr/local/nginx/logs/access.log --pid-path=/var/run/nginx/nginx.pid --lock-path=/var/lock/nginx.lock --user=www --group=www --with-openssl=/usr/src/openssl-1.0.2j --with-http_ssl_module --with-http_flv_module --with-http_stub_status_module --with-http_gzip_static_module --http-client-body-temp-path=/var/tmp/nginx/client/ --http-proxy-temp-path=/var/tmp/nginx/proxy/ --http-fastcgi-temp-path=/var/tmp/nginx/fcgi/ --http-uwsgi-temp-path=/var/tmp/nginx/uwsgi --http-scgi-temp-path=/var/tmp/nginx/scgi --without-http_upstream_ip_hash_module --with-pcre=/usr/src/pcre-8.38/")
              run("make && make install")
              run ("mkdir -p /var/tmp/nginx/client/")
#              run("chmod 755 /etc/init.d/nginx && /etc/init.d/nginx start")
              run("chmod 755 /etc/init.d/nginx && /usr/local/nginx/sbin/nginx -c /etc/nginx/nginx.conf")
def go():
     install_gcc()
     put_shell()
     put_task()
     install_pcre()
     install_opensll()
     install_nginx
#fab -f nginx.py -P go
```