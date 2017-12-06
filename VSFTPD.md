# FTP
## 1.工作流程
  1.TCP协议，使用两个连接：命令通道（Port 21）和数据流通道(Port 20)；  
  2.主动模式
          
    1.客户端Port大于1024；FTP服务器21端口(接受来自客户端主动请求)；
    2.三次握手，告知FTP主动方式连接，客户端先随机启动一个端口，并通过命令通道告知FTP 这个信息。等待FTP来连接；
    3.FTP由通道了解客户需求后，主动由20端口连接，这个过程也需要三次握手建立连接。
  
  3.被动模式
    
    1.通过三次握手建立命令通道；
    2.客户端通过命令发出PASV被动请求，等待FTP回应；
    3.FTP启动一个端口监听，端口可以使随机的，也可以定义某一范围端口；然后FTP通过命令通道告知客户端已经启动的端口并等待连接；
    4.客户端随机先去大语1024的端口进行连接。

## 2.vsftpd.conf 说明
### 2.1 与环境相关设置

    #1.ftp-data
    connect_form_port_20=YES
    
    #2.命令通道端口
    listen_port=21
   
    #3.开启消息通知，让VSFTP寻找该.message文件来显示信息, dirmessage_enable设置为YES，下一条才生效
    dirmessage_enable=YES
    message_file=.message
   
    #4.stand alone 方式启动，默认NO;设置为YES时，后续两个设置才生效；
    #  设置在此模式下设同一时间可以有多少客户端可以连接vsftp，同一个IP同一时间可允许多少连接;
    listen=NO
    max_clients=0
    max_per_ip=0
    
    #5.开启被动模式
    pasv_enable=YES
    
    #6.使用本地时间
    use_localtime=YES
    
    #7.允许用户上传文件
    write_enable=YES
    
    #8.数据连接的主动模式下，发出的信号在60秒内得不到客户端响应，强制断开连接
    connet_timeout=60
   
    #9.PASV模式下，服务器启用pasv port并等待Client超过60秒无回应，则强制断开连接
    accept_timeout=60
    
    #10.无论是主动还是被动模式，连接建立成功后。300秒内无法顺利完成数据传送，客户端的连接被FTP强制剔除
    data_connection_timeout=300
    
    #11.用户300秒内没有命令操作，则断开连接
    idle_session_timeout=300
    
    #12.pasv模式下端口设置，0表示不限制
    pasv_min_port=0
    pasv_max_port=0
    
    #13.Clients 连接vsftp时，在ftp客户端上显示的说明文字，一般不设置，也可以用banner_file 设置值来取代
    ftpd_banner=一些文字说明
    
    #14.这个可以指定纯文本文件作为用户登陆vsftp时所显示的欢迎文字
    banner_file=/path/file
       
### 2.2与实体用户相关设置
    
    #1.这个值设置为YES时，任何实体账号均被假设成为guest（默认不开放）
    guest_enable=NO
    #指定访客身份，依赖上一行设置必须为YES才生效
    guest_username=ftp
   
    #2.此值设置为YES时，/etc/passwd内账号才能以实体用户方式登陆
    local_enable=YES

    #3.实体用户传输速度设置，0表示不限制
    local_max_rate=0
   
    #4.是否将用户限制在自己的用户目录内(chroot),YES则默认就会被chroot
    chroot_local_user=YES
   
    #5.是否启用chroot写入功能，与下面的 chroot_list_file有关，若不开启，下面列表不生效
    chroot_list_enables=NO
    chroot_list_file=/etc/vsftpd/chroot_list
    
    #6.是否借助vsftpd的阻挡机制来处理某些不受欢迎的账号，与下面参数设置有关
    userlist_enable=YES
     
    #6.1. 当#6生效设置YES才生效，当用户账号被列入某个文件时，该文件内用户无法登录vsftp
    userlist_deny=YES
    
    #6.2. 当#6.1为YES时，此文件内账号都无法登录vsftp
    userlist_file=/etc/vsftpd/user_list

### 2.3安全方面
    #1.1.如果设置为YES，客户端优先使用ASCII格式下载
    ascii_download_enable=YES
    
    #1.2.与 1.1 类似，这个是针对上传，默认为NO
    ascii_upload_enable=YES
   
    #2.这个比较危险，若是设置为YES，表示每个建立的连接都会拥有一个process在负责，可以提高vsftpd效率，不过容易耗尽系统资源，且不安全。
    one_process_model=NO
   
    #3.习惯支持TCP wrappers
    tcp_wrappers=YES
   
    #4.设置为YES,上传文件和下载文件都会被记录
    xferlog_enable=YES
    
    #5.如果上一条设置为YES，这个才生效
    xferlog_file=/var/log/xferlog

    #6.是否设置为wu-ftp相同的日志格式，默认为NO。如果需要使用wu-ftp日志文件的分析软件才设置为YES
    xferlog_std_fromat=NO

    #7.定义自己的日志文件格式
    dual_log_enable=YES
    vsftpd_log_file=/var/log/vsftpd.log

    #8.我们用nobody作为此服务执行权限，因为nobody权限相当低，即使被入侵，也只能取得nobody权限
    nopriv_user=nobody
   
    #9.PAM模块名称,放在/etc/pam.d/vsftpd中即可
    pam_service_name=vsftpd
   
### 2.4匿名用户
    
    #1.控制是否允许匿名用户登入，YES 为允许匿名登入，NO 为不允许。默认值为YES;设置为YES后下面的后续设置才生效
    anonymous_enable=NO
    
    #2允许匿名登入者有下载可读文件的权限，默认是YES
    anon_world_readable_only=YES
    
    #3是否允许anonymous具有除了写入之外的权限，包括删除与修改服务器上的文件及文件名等权限，默认为NO；如果设置为YES,那么开放给anonymous写入的目录一需要调整权限，让vsftpd的PID拥有者可以写入才行。
    anon_other_write_enable=YES
    
    #4.是否让anonymous具有建立目录的权限，默认为NO;如果设置为YES,那么anony_other_write_enble必须设置为YES
    anon_mkdir_write_enable=YES
   
    #5.是否让anonymous具有上传数据的功能，默认为NO；如果设置为YES，那么anony_other_write_enble必须设置为YES
    anon_upload_enable=YES

    #6.若是启动这项功能，则必须提供一个档案/etc/vsftpd/banner_emails，内容为email address。若是使用匿名登入，则会要求输入email address,若输入的email address在此档案内，则不允许进入。默认值为NO。需于下个设置项配合使用
    deny_email_enable=YES
    
    #7.上一项设置为YES时，可以定义哪些email address 不可以登录vsftpd,一行输入一个email address
    banned_email_file=/etc/vsftpd/banner_emails
    
    #8.当设置为YES时，表示anonymous将会略过密码直接登录，一般默认为NO
    no_anon_password=NO
   
    #9.限制anonymous的传输速度，0为不限制；单位是bytes/秒，若是想让anonymous仅有30K/s,可以设置为30000
    anon_max_rate=0
   
    #限制anonymous的上传权限，如果是077，则anonymous传送过来的文件权限为-rw------
    anon_umask=077

## 3.启动模式
### 3.1模式
   
    stand alone::(centos6默认启动方式)若是FTP提供给整个因特网来进行大量任务下载，例如各大院校的FTP服务器  
    super daemon:(centos7默认启动方式)仅是提供给内部员工使用的FTP服务器

### 3.2启动方式
#### 3.2.1.script  
    /etc/init.d/vsftpd start
   
#### 3.2.2super daemon
    
    #1.vim /etc/vsftpd/vsftpd.conf   
    找到 #listen=YES这一行，大约在109行，改成 listen=NO
    
    #2. yum -y install xinetd  <==假设xinetd不存在
    vim /etc/xinetd.d/vsftpd
    service ftp
    {
           
            socket_type      = stream
            wait             = no
            user             = root
            server           = /usr/bin/vsftpd
            log_on_success  += DURATION USERID
            log_on_failure  += USERID
            nice             = 10
            disable          = no
    }
    
    #3.启动
    /etc/init.d/vsftpd stop
    /etc/init.d/vsftpd restart

## 4.vsftpd.conf 案例
###4.1.针对实体账号的设定(centos7)
    #禁用匿名账户
    anonymous_enable=NO

    #2.与实体账户有关设置，可写入,且 umask 为 002
    local_enable=YES
    write_enable=YES
    local_umask=002
    userlist_enable=YES
    userlist_deny=YES
    userlist_file=/etc/vsftpd/user_list
   
    #3.服务器环境有关设置
    use_localtime=YES
    dirmessage_enable=YES
    xferlog_enable=YE
    xferlog_file=/var/log/xferlog
    xferlog_std_fromat=NO
    dual_log_enable=YES
    vsftpd_log_file=/var/log/vsftpd.log
    connect_from_port_20=YES
    pam_service_name=YES
    listen=YES
    tcpwrappers=YES
    banner_file=/etc/vsftpd/welcome.txt
   
    #4.建立欢迎信息
    echo "欢迎光临本小站,本站提供 FTP 的相关服务"  > /etc/vsftpd/welcome.txt
    echo "主要的服务是针对本机实体用户提供的"  >> /etc/vsftpd/welcome.txt
    echo "若有任何问题，请与kedge.zhang@100tal.com 联系" >> /etc/vsftpd/welcome.txt
    
    #5.建立限制系统登陆的文件
    /etc/vsftpd/ftpusers :就是/etc/pam.d/vsftpd 这个文件设置所影响。
    /etc/vsftpd/user_list ：由vsftpd.conf的userlist_file 所设置。
