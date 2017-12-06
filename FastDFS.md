# FastDFS
##1 介绍
* FastDFS是一个开源的轻量级分布式文件系统，它对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。  
* FastDFS为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标，使用FastDFS很容易搭建一套高性能的文件服务器集群提供文件上传、下载等服务。
![结构图](http://i.imgur.com/D247hxV.jpg) 
* FastDFS服务端有两个角色：跟踪器（tracker）和存储节点（storage）。跟踪器主要做调度工作，在访问上起负载均衡的作用。 
* 为了支持大容量，存储节点（服务器）采用了分卷（或分组）的组织方式。存储系统由一个或多个卷组成，卷与卷之间的文件是相互独立的，所有卷 的文件容量累加就是整个存储系统中的文件容量。一个卷可以由一台或多台存储服务器组成，一个卷下的存储服务器中的文件都是相同的，卷中的多台存储服务器起 到了冗余备份和负载均衡的作用。   
* 在卷中增加服务器时，同步已有的文件由系统自动完成，同步完成后，系统自动将新增服务器切换到线上提供服务。 
* 当存储空间不足或即将耗尽时，可以动态添加卷。只需要增加一台或多台服务器，并将它们配置为一个新的卷，这样就扩大了存储系统的容量。 

##2 交互过程
###2.1 上传交互过
`![英文图](http://i.imgur.com/QiO8ArU.jpg)`
![上传](http://i.imgur.com/gcd3knF.png)  
* client询问tracker上传到的storage，不需要附加参数；  
* tracker返回一台可用的storage；  
* client直接和storage通讯完成文件上传。  

###2.2 下载交互过程
`![英文图](http://i.imgur.com/GbUPOxH.jpg)`
![下载](http://i.imgur.com/nrJjW6x.png)  
* client询问tracker下载文件的storage，参数为文件标识（卷名和文件名);  
* tracker返回一台可用的storage；  
* client直接和storage通讯完成文件下载。 

`client为使用FastDFS服务的调用方，client也应该是一台服务器，它对tracker和storage的调用均为服务器间的调用。`

##3 集群搭建  
###3.1 需要的安装包
```
wget https://github.com/happyfish100/fastdfs/archive/V5.10.tar.gz
```
```
git clone https://github.com/happyfish100/fastdfs-nginx-module.git
```
```
git clone https://github.com/happyfish100/libfastcommon.git
```
```
wget http://tengine.taobao.org/download/tengine-2.2.0.tar.gz
```
###3.2 tracker配置
```
yum install  gcc gcc-c++ make openssl-devel kernel-devel zlib-devel -y  
```
```
mkdir -p /data/fastdfs/{tracker,storage} //用于存储
```
###3.3 安装libfastcommon
安装libfastcommon，这个是FastDFS必须要安装的东西

```
git clone https://github.com/happyfish100/libfastcommon.git  
```  
```
cd libfastcommon && ./make.sh && ./make.sh install
```  

装完后会看到libfastcommon.so安装到了/usr/lib64/libfastcommon.so，但是FastDFS主程序设置的lib目录是/usr/local/lib，所以需要创建软链接.

```
ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so && ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so && ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so && ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```  

###3.4 安装安装FastDFS
```
tar fvx V5.10  && cd fastdfs-5.10 && ./make.sh && ./make.sh install && cd .. && mv fastdfs-5.10 /usr/local/fastdfs 
```

client.conf    客户端上传配置文件  
storage.conf   文件存储服务器配置文件  
tracker.conf   负责均衡调度服务器配置文件  
http.conf      http服务器配置文件  

###3.5 配置文件
####3.5.1 配置tracker
编辑tracker.conf  
```
cd /etc/fdfs && cp tracker.conf.sample tracker.conf   
```  
vim tracker.conf      
```
base_path=/data/fastdfs/tracker------日志存储目录
```  
```
store_group=group2----------group链接中的组名如：fastdfs/M00/00/00/oYYBAFWd9RaAd4nRAADrmdE3r9M406.png
```
```
log_level=debug--------------日志打印为debug信息会比较全
```
```
use_storage_id = true--------使用的storage_ids.conf配置文件
```
```
id_type_in_filename = id-----使用storage表示为id,原值为ip
```
```
http.server_port=80----------http端口80用来访问storage
```  
####3.5.2 配置storage\_ids.conf
把配置文件storage\_ids.conf复制到/etc/fdfs/----fastdfs的配置文件目录(一组不用修改)
```
cd /etc/fdfs/ && cp storage_ids.conf.sample storage_ids.conf
```   
vim /etc/fdfs/storage\_ids.conf  
//修改为如下  
`# <id>  <group_name>  <ip_or_hostname>`  
100001   fastdfs  172.31.0.19  
100002   fastdfs  172.31.0.20
####3.5.3 启动顺序
启动tracker----一般先启动tracker再启动storage，如果先启动storage的话会卡住，查看storage的日志就知道了  
fdfs_trackerd /etc/fdfs/tracker.conf----启动后会在/data/fastdfs/tracker/下创建logs目录，里面有tracker.log  

启动后查看下日志 看看有啥问题没有,tracker最后    
cp /root/fastdfs-5.10/conf/http.conf mime.types /etc/fdfs/ 后期用tracker上传文件测试用  
####3.5.4 配置 client.conf  
```
cd /etc/fdfs/ && cp client.conf.sample  client.conf
```   
vim client.conf  
base_path=/data/fastdfs  
tracker\_server=172.31.0.19:22122  
tracker\_server=172.31.0.20:22122  
log\_level=debug

####3.5.5 配置storage  
```
cd /etc/fdfs/ && cp storage.conf.sample storage.conf
```  
vim storage.conf  
group\_name=group1  
base\_path=/data/fastdfs  
store\_path0=/data/fastdfs/storage/  
tracker\_server=172.31.0.9:22122  
tracker\_server=172.31.0.10:22122  
log\_level=debug  
http.server\_port=80  

####3.5.6测试
执行命令上传图片，如果没有通查看日志，或者看防火墙端口,命令如下：    
fdfs\_upload\_file client.conf /dir/aaaaaaaaaaaa.png(随意找一个文件)  

会返回类似这样的  
fastdfs/M00/00/00/oYYBAFWd9RaAd4nRAADrmdE3r9M406.png    
然后上第一个storage上/data/fastdfs/data/00/00发现文件被上传 

##4 storage配置nginx ##
`tar zxvf tengine-2.2.0.tar.gz`
编译安装
```
./configure --prefix=/usr/local/nginx --add-module=/root/fastdfs-nginx-module/src
make &&  make install 
```
修改nginx配置文件
`vim /usr/local/nginx/conf/nginx.conf`

server段加上  

location /group1/M00 {  
        root /data/fastdfs/storage/;  
        ngx_fastdfs_module;  
}  


复制mod_fastdfs.con配置文件到/etc/fdfs下
`cp /root/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs/`  
编辑 mod_fastdfs.con  
`vim /etc/fdfs/mod_fastdfs.conf`  
修改以下几项

base\_path=/data/fastdfs  
tracker\_server=172.31.0.19:22122  
tracker\_server=172.31.0.20:22122  
group\_name=group1  
store\_path0=/data/fastdfs/storage/  
url_have\_group_name = true---意思是在url中包含名  
cp /root/fastdfs-5.10/conf/http.conf /etc/fdfs/  
cp /root/fastdfs-5.10/conf/mime.types /etc/fdfs/  
