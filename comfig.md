## 流程简述

网上找了个图:

![img](https://raw.githubusercontent.com/JakeYue/picgo/master/uPic/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3p6cmpnenM=,size_16,color_FFFFFF,t_70.png2021%2009%2023%2013%2003%2057)

1．安装CentOS 8操作系统

- 自行安装

2．安装DHCP、HTTP、TFTP、XINETD服务(docker)

- docker容器构建的

3．配置上述服务(data_conf里面的配置文件修改)

- 上述服务的配置文件

4．定制启动菜单(自行修改)

5．布署WinPE系统

#### 根目录下文件路径

```shell
/
--IPXE
--sharefile
--tftpboot
--windows
--data_conf
```

### 配置文件目录

```shell
conf_data
├── dhcp
│   ├── dhcpd.conf
│   └── #dhcpd.leases
├── nginx
│   └── nginx.conf
├── samba
│   └── smb.conf
└── tftp
    └── tftp
```

```shell
docker 服务配置文件路径
#dhcp
/etc/dhcp/dhcpd.conf:/etc/dhcp/dhcp.conf
#nginx
/etc/nginx/nginx.conf:/etc/nginx/nginx.conf
/sharefile:/sharefile
/IPXE:/IPXE
#tftp
/tftpboot:/var/tftpboot
#samba
/etc/samba/smb.conf
/windows:/windows
```

### 宿主机安装Docker && Docker-compose

```shell
卸载旧版本
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
                  
                  
设置稳定源
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
安装Docker
sudo yum install docker-ce docker-ce-cli containerd.io

查询docker版本
yum list docker-ce --showduplicates | sort -r
-----------------------------------------------------------------------------
安装特定版本(可选不必操作,主要安装制定版本的docker)
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
-----------------------------------------------------------------------------

启动
sudo systemctl start docker

停止
sudo systemctl stop docker

重启
sudo systemctl restart docker

-----------------------------------------------------------------------------
安装docker-compose
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
权限设置 
sudo chmod +x /usr/local/bin/docker-compose
查看版本
docker-compose --version
```

#### docker载入本地景象

```shell
docker load —input xxx.tar
## xxx.tar 是本地保存的镜像,镜像一般保存为tar格式

#或者使用
docker load < xxx.tar
```

**==或者使用docker-compose.yml 文件构建容器==**(2选一)

- 提示:构建前请创建上述目录并把配置文件放入到目录中

Linux网桥查看工具命令

```shell
rpm -ivh http://mirror.centos.org/centos/7/os/x86_64/Packages/bridge-utils-1.5-9.el7.x86_64.rpm
#docker-compose 下载
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
#nmcli 命令也可以查看网桥
nmcli conn show -a
```

### 客户端网络引导启动所需文件、位置、和软件

客户端网络引导启动所需文件、位置、和软件:

- DHCP分配IP地址 (dhcp-server(提供:HDCP 服务))

```shell
宿主机网段-- DHCP分配子网: 192.168.3.0/24
可以去dhcpd.conf 修改自己需要的网断(data_conf/dhcp/dhcpd.conf)
```

- tftp提供ipxe引导文件

```shell
tftpboot
├── ipxe.efi
└── undionly.kpxe
path:/tftpboot/
```

- HTTP (nginx(提供http服务) && (samba(提供Windows共享文件服务))

```shell
# 根目录下 -文件目录结构
IPXE
├── amd64
│   └── media
│       ├── Boot
│       │   ├── BCD
│       │   ├── boot.sdi
│       ├── #bootmgr
│       ├── #bootmgr.efi
│       └── sources
│           └── boot.wim
├── boot.ipxe
├── installer.bat
├── ipxe.efi
├── undionly.kpxe
├── wimboot
├── winpeshl.ini
└── x86
    └── media
        ├── Boot
        │   ├── BCD
        │   ├── boot.sdi
        ├── #bootmgr
        ├── #bootmgr.efi
        └── sources
            └── boot.wim

#samba(提供Windows共享文件服务,安装文件)
path: /windows

```

#### docker端口

```shell
#端口
web 8080:80
dhcp host
tftp host
samba 137 138 139 445
```

### 需要的配置文件(这些可以不看,想了解可以查看下百度)

#### DHCP

> ```shell
>#dhcpd.conf
> ########IPXE
> option ipxe.no-pxedhcp 1;
> option space ipxe;
> option ipxe-encap-opts code 175 = encapsulate ipxe;
> option ipxe.priority code 1 = signed integer 8;
> option ipxe.keep-san code 8 = unsigned integer 8;
> option ipxe.skip-san-boot code 9 = unsigned integer 8;
> option ipxe.syslogs code 85 = string;
> option ipxe.cert code 91 = string;
> option ipxe.privkey code 92 = string;
> option ipxe.crosscert code 93 = string;
> option ipxe.no-pxedhcp code 176 = unsigned integer 8;
> option ipxe.bus-id code 177 = string;
> option ipxe.san-filename code 188 = string;
> option ipxe.bios-drive code 189 = unsigned integer 8;
> option ipxe.username code 190 = string;
> option ipxe.password code 191 = string;
> option ipxe.reverse-username code 192 = string;
> option ipxe.reverse-password code 193 = string;
> option ipxe.version code 235 = string;
> option iscsi-initiator-iqn code 203 = string;
> # Feature indicators
> option ipxe.pxeext code 16 = unsigned integer 8;
> option ipxe.iscsi code 17 = unsigned integer 8;
> option ipxe.aoe code 18 = unsigned integer 8;
> option ipxe.http code 19 = unsigned integer 8;
> option ipxe.https code 20 = unsigned integer 8;
> option ipxe.tftp code 21 = unsigned integer 8;
> option ipxe.ftp code 22 = unsigned integer 8;
> option ipxe.dns code 23 = unsigned integer 8;
> option ipxe.bzimage code 24 = unsigned integer 8;
> option ipxe.multiboot code 25 = unsigned integer 8;
> option ipxe.slam code 26 = unsigned integer 8;
> option ipxe.srp code 27 = unsigned integer 8;
> option ipxe.nbi code 32 = unsigned integer 8;
> option ipxe.pxe code 33 = unsigned integer 8;
> option ipxe.elf code 34 = unsigned integer 8;
> option ipxe.comboot code 35 = unsigned integer 8;
> option ipxe.efi code 36 = unsigned integer 8;
> option ipxe.fcoe code 37 = unsigned integer 8;
> option ipxe.vlan code 38 = unsigned integer 8;
> option ipxe.menu code 39 = unsigned integer 8;
> option ipxe.sdi code 40 = unsigned integer 8;
> option ipxe.nfs code 41 = unsigned integer 8;
> ######
> option client-architecture code 93 = unsigned integer 16;
> if exists user-class and option user-class = "iPXE" {
> filename "http://my.web.server/real_boot_script.php";#可更改为自己的http地址
> } elsif option client-architecture = 00:00 {
>    filename "undionly.kpxe";
> } else {
>    filename "ipxe.efi";
> }
>    next-server tftp-serverip;
> 
> ###########DHCP
> ddns-update-style none; #动态更新方式 没有
> authoritative; #授权
> #option domain-name "example.org"; #设置域名
> #option domain-name-servers ns1.example.org, ns2.example.org; #设置域名服务器（设置DNS）
> log-facility local7;
> 
> default-lease-time 600;#默认租期时间
> max-lease-time 7200;#最大租期时间
> 
> #subnet 10.152.187.0 netmask 255.255.255.0 {
> #} #没有子网的声明,帮助dhcp了解网络拓扑
> 
> # 基础子网声明,这个声明运行BUUTP(引导协议)客户端获取地址不建议这样）
> #subnet 10.254.239.0 netmask 255.255.255.224 {
> # range 10.254.239.10 10.254.239.20; #范围
> #option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org; #设置路由
> #}
> # 子网声明
> subnet 192.168.3.0 netmask 255.255.255.224 {
> range 192.168.3.50 192.168.3.150; 
> option domain-name-servers 192.168.3.1; #设置DNS
> #option domain-name "internal.example.org"; #设置域名
> option routers 192.168.3.1;  #设置路由地址
> option broadcast-address 192.168.3.255;#设置广播地址
> default-lease-time 600;#默认租期时间
> max-lease-time 7200;#最大租期时间
> }
> #主机绑定,需要特殊配置
> #host passacaglia {
> # hardware ethernet 0:0:c0:5d:bd:95; #硬件以太网
> #filename "vmunix.passacaglia";  #文件名
> #server-name "toccata.fugue.com"; #服务器名
> #}
> #固定IP
> #host fantasia {
> # hardware ethernet 08:00:07:26:c0:a5;
> #fixed-address fantasia.fugue.com; #固定地址
> #}
> 
> #您可以声明一个客户端类，然后以这个为依据进行地址分配。下面的示例显示了所有客户机的情况在某个类上获得10.17.224/24子网上的所有地址其他客户端获得10.0.29/24子网上的地址
> #class "foo" {
> # match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
> #}
> 
> #shared-network 224-29 { 		#共享网络
> # subnet 10.17.224.0 netmask 255.255.255.0 {
> #  option routers rtr-224.example.org;
> #}
> #subnet 10.0.29.0 netmask 255.255.255.0 {
> # option routers rtr-29.example.org;
> #}
> #pool {
> # allow members of "foo";
> #range 10.17.224.10 10.17.224.250;
> #}
>  #pool {
> # deny members of "foo";
> #range 10.0.29.10 10.0.29.230;
> #}
>  #}
> 
> 
> ```
> 

#### Http(nginx)

> ```shell
> #定义Nginx运行的用户和用户组
> #user  nobody;
> 
> #nginx进程数，建议设置为等于CPU总核心数。
> worker_processes  2;
> 
> #全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
> #error_log  logs/error.log;
> #error_log  logs/error.log  notice;
> #error_log  logs/error.log  info;
> 
> #进程文件
> #pid        logs/nginx.pid;
> 
> #工作模式与连接数上限
> events {
>     #单个进程最大连接数（最大连接数=连接数*进程数）
>     worker_connections  1024;
> }
> 
> #设定http服务器
> http {
>     #文件扩展名与文件类型映射表
>     include       mime.types;
>     #默认文件类型
>     default_type  application/octet-stream;
> 
>     #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>     #                  '$status $body_bytes_sent "$http_referer" '
>     #                  '"$http_user_agent" "$http_x_forwarded_for"';
> 
>     #access_log  logs/access.log  main;
> 
>     #开启高效文件传输模式，sendfile指令指定nginx是否调用sendfile函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络I/O处理速度，降低系统的负载。注意：如果图片显示不正常把这个改 成off。
>     sendfile        on;
> 
>     #防止网络阻塞
>     tcp_nopush     on;
>     tcp_nodelay    on;
> 
> 
>     #长连接超时时间，单位是秒
>     #keepalive_timeout  0;
>     keepalive_timeout  65;
> 
>     #开启gzip压缩输出
>     gzip  on;
> 
>     #虚拟主机的配置
>     server {
>         #监听端口
>         listen       80;
> 
>         #域名可以有多个，用空格隔开
>         server_name  install.org;
> 
>         #默认编码
>         charset utf-8;
> 
>         #定义本虚拟主机的访问日志
>         #access_log  logs/host.access.log  main;
> 
>         location / {
>             root   /sharefile;
> 	    autoindex on; # 索引
>    	    autoindex_exact_size on; # 显示文件大小
>     	    autoindex_localtime on; # 显示文件时间
>         }
> 
>         location /ipxe {
>             alias   /IPXE;
> 	    autoindex on; # 索引
>         }
> 
>         #error_page  404              /404.html;
> 
>         # redirect server error pages to the static page /50x.html
>         #
>         error_page   500 502 503 504  /50x.html;
>         location = /50x.html {
>             root   html;
>         }
> 
>         # proxy the PHP scripts to Apache listening on 127.0.0.1:80
>         #
>         #location ~ \.php$ {
>         #    proxy_pass   http://127.0.0.1;
>         #}
> 
>         # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
>         #
>         #location ~ \.php$ {
>         #    root           html;
>         #    fastcgi_pass   127.0.0.1:9000;
>         #    fastcgi_index  index.php;
>         #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
>         #    include        fastcgi_params;
>         #}
> 
>         # deny access to .htaccess files, if Apache's document root
>         # concurs with nginx's one
>         #
>         #location ~ /\.ht {
>         #    deny  all;
>         #}
>     }
> 
> 
>     # another virtual host using mix of IP-, name-, and port-based configuration
>     #
> 
> 
>     # HTTPS server
>     #
>     #server {
>     #    listen       443 ssl;
>     #    server_name  localhost;
> 
>     #    ssl_certificate      cert.pem;
>     #    ssl_certificate_key  cert.key;
> 
>     #    ssl_session_cache    shared:SSL:1m;
>     #    ssl_session_timeout  5m;
> 
>     #    ssl_ciphers  HIGH:!aNULL:!MD5;
>     #    ssl_prefer_server_ciphers  on;
> 
>     #    location / {
>     #        root   html;
>     #        index  index.html index.htm;
>     #    }
>     #}
>     #
> }
> ```

#### samba

> ```shell
> # See smb.conf.example for a more detailed config file or
> # read the smb.conf manpage.
> # Run 'testparm' to verify the config is correct after
> # you modified it.
> 
> [global]
> 	workgroup = WORKGROUP
> 	security = user
> 	map to guest = Bad User
> 	passdb backend = tdbsam
> 	load printers = yes
> 	cups options = raw
> 	interfaces = 192.168.3.5 lo # 访问samba地址
> 	max connections = 0
> 	socket options = TCP_NODELAY SO_RCVBUF=8192 SO_SNDBUF=8192
> [homes]
> 	comment = Home Directories
> 	valid users = %S, %D%w%S
> 	browseable = No
> 	read only = No
> 	inherit acls = Yes
> [sharewin]
> 	comment = windows IOS
> 	path = /windows
> 	browseable = yes
> 	writable = yes
> 	available = yes
> 	guest ok = yes
> ```

#### tftp

> ```shell
> service tftp
> {
>        disable = no
>        socket_type             = dgram
>        protocol                = udp
>        wait                    = yes
>        user                    = root
>        server                  = /usr/sbin/in.tftpd
>        server_args             = -u root -s /tftpboot -c #指定tftp服务器的目录，-c为指定为可以创建文件
>        per_source              = 11
>        cps                     = 100 2
>        flags                   = IPv4
> }
> ```
>
> 

### iPXE相关

创建默认的pxe启动配置菜单

```shell
UI menu.c32
label 1
menu label ^1) install windows 
KERNEL memdisk
INITRD winpwe_amd64.iso
APPEND iso raw

```





