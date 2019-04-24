+ rpm -qa | grep vsftp   **查询安装的软件**  
+ rpm -e fielname.rpm  

### 1. 使用vsftpd服务传输文件  
    FTP(File Transfer Protocol),基于C/S模式,默认端口:-21-(数据端口),22(命令端口) 
#### 1.1 主动 被动 2种模式
    vsftpd(very secure ftp daemon) 非常安全的FTP守护进程,  匿名开放 本地用户 
#### 1.2 虚拟用户 3种模式 
   
### 2. 服务端  
`[root@xy ~]# yum install vsftpd`  
`[root@xy ~]# iptables -F`  
`[root@xy ~]# service iptables save`   
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]  
`[root@xy ~]# mv /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf_bak`  
`[root@xy ~]# grep -v "#" /etc/vsftpd/vsftpd.conf_bak > /etc/vsftpd/vsftpd.conf`  
`[root@xy ~]# cat /etc/vsftpd/vsftpd.conf`

    anonymous_enable=YES  
    local_enable=YES  
    write_enable=YES  
    local_umask=022  
    dirmessage_enable=YES  
    xferlog_enable=YES  
    connect_from_port_20=YES  
    xferlog_std_format=YES  
    listen=NO  
    listen_ipv6=YES  
    pam_service_name=vsftpd  
    userlist_enable=YES  
    tcp_wrappers=YES  
#### 2.1 常用参数:  
    listen=[YES|NO]                       是否以独立运行方式监听  
    listen_address=IP                     设置要监听的IP           
    listen_port=21                        设置FTP服务的监听端口  
    download _enable=[YES|NO]             是否允许下载  
    userlist_enable=[YES|NO]              设置用户列表为允许/禁止操作  
    userlist_deny=[YES|NO]       
    max_client=0                          最大客户连接数 0不限制  
    max_per_ip=0                          同一IP最大连接数  
    anonymous_enable=[YES|NO]             是否允许匿名用户访问  
    anon_upload_enable=[YES|NO]           是否允许匿名用户上传文件  
    anon_umask=022                        匿名用户上传文件的umask值  
    anon_root=/var/ftp                    匿名用户FTP根目录  
    anon_mkdir_write_enable=[YES|NO]      是否允许匿名用户创建目录 
    anon_other_write_enable=[YES|NO]      是否允许匿名用户其他写入权限 重命名 删除等  
    anon_max_rate=0                       匿名用户最大传输速率  
    local_enable=[YES|NO]                 是否允许本地用户登陆FTP  
    local_umask=022                       本地用户上传文件的umask值  
    local_root=/var/ftp                   本地用户FTP根目录  
    chroot_local_user=[YES|NO]            是否将用户权限禁锢在FTP目录,以确保安全  
    local_max_rate=0                      本地用户最大传输速率  

#### 2.2 匿名用户模式开发权限参数
    anonymous_enable=YES                 
    anon_upload_enable=YES               
    anon_umask=022                                                  
    anon_mkdir_write_enable=YES            
    anon_other_write_enable=YES 
 
#### 2.3 客户端:
`[root@xy ~]# yum install ftp`    
`[root@xy ~]# ftp 192.168.37.10` 
 
    Connected to 192.168.37.10 (192.168.37.10).  
    220 (vsFTPd 3.0.2)  
    Name (192.168.37.10:root): anonymous  
    331 Please specify the password.  
	Password:  
	230 Login successful.  
	Remote system type is UNIX.  
	Using binary mode to transfer files.  
	ftp> cd pub  
	250 Directory successfully changed.  
	ftp> mkdir files  
	550 Permission denied   **提示没有权限,查看默认访问目录/var/ftp权限**  
	ftp> exit  
	221 Goodbye.  

`[root@xy ~]# ls -ld /var/ftp/pub`   **ls -Zd**  
drwxr-xr-x. 2 root root 6 Mar  7  2014 /var/ftp/pub  
`[root@xy ~]# chown -Rf ftp /var/ftp/pub`      **更改目录所属主身份为ftp**  
`[root@xy ~]# ls -ld /var/ftp/pub`  
drwxr-xr-x. 2 ftp root 6 Mar  7  2014 /var/ftp/pub  
`[root@xy ~]# ftp 192.168.37.10` 
 
	Connected to 192.168.37.10 (192.168.37.10).  
	220 (vsFTPd 3.0.2)  
	Name (192.168.37.10:root): anonymous  
	331 Please specify the password.  
	Password:  
	230 Login successful.  
	Remote system type is UNIX.  
	Using binary mode to transfer files.  
	ftp> cd pub  
	250 Directory successfully changed.  
	ftp> mkdir files  
	550 Create directory operation failed. 提示创建目录失败,原因所在SElinux
  
`[root@xy ~]# getsebool -a | grep ftp`   **getsebool -a** 
 
    ftp_home_dir --> off  
    ftpd_anon_write --> off  
    ftpd_connect_all_unreserved --> off  
    ftpd_connect_db --> off  
    ftpd_full_access --> off       **ftp 全部权限未开启 off 改为 on**
  
`[root@xy ~]# setsebool -P ftpd_full_access=on`      **-P permanent**  
`[root@xy ~]# ftp 192.168.37.10` 
 
	Connected to 192.168.37.10 (192.168.37.10).  
	220 (vsFTPd 3.0.2)  
	Name (192.168.37.10:root): anonymous   默认名称:anonymous  
	331 Please specify the password.  
	Password:  
	230 Login successful.  
	Remote system type is UNIX. 
	Using binary mode to transfer files.  
	ftp> cd pub  
	250 Directory successfully changed.  
	ftp> mkdir files  
	257 "/pub/files" created  
	ftp> rename files database  
	350 Ready for RNTO.  
	250 Rename successful.  
	ftp> rmdir database  
	250 Remove directory operation successful.  
	ftp> exit  
	221 Goodbye.  

#### 2.4 本地用户模式参数:cat /etc/vsftpd/vsftpd.conf  
    anonymous_enable=NO  
    local_enable=YES  
    write_enable=YES  
    local_umask=022  
    userlist_enable=YES  
    userlist_deny=YES  

`[root@xy ~]# ftp 192.168.37.10` 
 
	Connected to 192.168.37.10 (192.168.37.10).  
	220 (vsFTPd 3.0.2)  
	Name (192.168.37.10:root): root  
	530 Permission denied.  
	Login failed.  
	ftp> exit  
	221 Goodbye.  
`[root@xy ~]# cat /etc/vsftpd/user_list`  
`[root@xy ~]# cat /etc/vsftpd/ftpusers`    **名单中含有root,默认禁止root用户**  

#### 2.5 虚拟用户模式  
`[root@xy ~]# cd /etc/vsftpd/`  
`[root@xy vsftpd]# vim vuser.list`   
   
	zhangsan  
	123456 
	lisi  
	123456   
	redhat  
	123456  
`[root@xy vsftpd]# db_load -T -t hash -f vuser.list vuser.db`  **db_load**  
`[root@xy vsftpd]# file vuser.db`  
vuser.db: Berkeley DB (Hash, version 9, native byte-order)  
`[root@xy vsftpd]# chmod 600 vuser.db`  
`[root@xy vsftpd]# rm -f vuser.list`  

`[root@xy vsftpd]# useradd -d /var/ftproot -s /sbin/nologin virtual`   
    -d 设置默认家目录 	-s 设置默认Shell解释器  (命令 -参数 执行语句)    	
`root@xy vsftpd]# ls -ld /var/ftproot/`  
drwx------. 3 virtual virtual 74 Jul 12 08:55 /var/ftproot/  
`[root@xy vsftpd]# chmod -Rf 755 /var/ftproot/`  
`[root@xy vsftpd]# ls -ld /var/ftproot/`  
drwxr-xr-x. 3 virtual virtual 74 Jul 12 08:55 /var/ftproot/  

> 建立用于支持虚拟用户的PAM文件, vsftpd.vu PAM(可插拔认证模块)是一种验证机制,通过动态链接库和统一的API把系统提供的服务和认证方式分开,
采用鉴别模块层、用接口层、应设计层三层设计方式 
 
`[root@xy ~]# vim /etc/pam.d/vsftpd.vu`  
auth      required      pam_userdb.so db=/etc/vsftpd/vuser    
account   required      pam_userdb.so db=/etc/vsftpd/vuser    

##### 2.5.1PAM文件参数:  
    anonymous_enable=NO  
    local_enable=YES  
    guest_enable=YES  
    guest_username=virtual  
    pam_service_name=vsftpd.vu   
    allow_writeable_chroot=YES  
为了使不同用户有不同权限:  
`[root@xy ~]# mkdir /etc/vsftpd/vusers_dir/`  
`[root@xy ~]# cd /etc/vsftpd/vusers_dir`  
`[root@xy vusers_dir]# touch lisi`  
`[root@xy vusers_dir]# vim zhangsan`  
   anon_upload_enable=YES  
   anon_mkdir_write_enable=YES  
   anon_other_write_enable=YES  
`[root@xy ~]# vim /etc/vsftpd/vsftpd.conf`  
user_config_dir=/etc/vsftpd/vuser_dir  
`[root@xy vusers_dir]# systemctl restart vsftpd.service`   
`[root@xy vusers_dir]# systemctl enable vsftpd`  

### 3. 简单文件传输协议(TFTP:Trivial File Transfer Protocol):TFTP采用UDP协议,默认端口号69  
    ?               帮助  
    put             上传文件   
    get             下载文件  
    verbose         显示详细处理信息   
    status          显示当前状态信息  
    binary          使用二进制进行传输  
    ascii           使用ASCII码传输  
    timeout         设置重传超时时间  
    quit            退出  

`[root@xy ~]# vim /etc/xinetd.d/tftp`       **disable = no**  
`[root@xy ~]# systemctl restart xinetd` 
`[root@xy ~]# netstat -a | grep tfetp` 	**查询端口启用状态**  
`[root@xy ~]# systemctl enable xinetd`  
`[root@xy ~]# firewall-cmd --permanent --add-port=69/udp`   
success  
`[root@xy ~]# firewall-cmd --reload`   
success  
`[root@xy ~]# echo "I love AnXiaohong" > /var/lib/tftpboot/readme.txt`  
`[root@xy ~]# tftp 192.168.37.10`  
tftp> get readme.txt  
tftp> quit  
`[root@xy ~]# ls`  
anaconda-ks.cfg  Downloads             Pictures    ShellExample  
Desktop          initial-setup-ks.cfg  Public      Templates  
Documents        Music                 readme.txt  Videos  
`[root@xy ~]# cat readme.txt`   
I love AnXiaohong  

### 4. 使用Samba或NFS实现文件"共享"  
   基于SMB(Server Messages Block)协议,开发出SMBServer服务程序,实现Linux与Windows之间的文件共享变得简单  
`[root@xy ~]# mv /etc/samba/smb.conf /etc/samba/smb.conf.bak`  
`[root@xy ~]# cat /etc/samba/smb.conf.bak | grep -v "#" | grep -v ";" | grep -v "^$" > /etc/samba/smb.conf`  **-v 反选 ^$ 空白行**  
`[root@xy ~]# cat /etc/samba/smb.conf`  
 
    [global]
    	workgroup = MYGROUP                        工作组名称    
    	server string = Samba Server Version %v    服务器介绍信息,参数%v 为显示SMB版本      
    	log file = /var/log/samba/log.%m           定义日志文件存放位置与名称,%m来访主机名    
    	max log size = 50            
    	security = user   安全验证方式
           
#### 4.1 4种安全验证方式:
+ share:来访主机无须验证口令  
+ user:需要验证来访主机提供的口令才可以访问  
+ server:使用独立远程主机验证来访主机提供的口令  
+ domain:使用域控制器进行身份验证  
`passdb backend = tdbsam`  

#### 4.2 用户后台类型3种:    
+ smbpasswd:使用smbpasswd命令设置Samba服务程序密码  
+ tdbsam:创建数据库文件并使用pdbedit命令建立Samba服务程序用户  
+ ldapsam:基于LDAP服务进行账户验证  
`load printers = yes`  
`cups options = raw`              打印机选项  
-----------------------------------------------------------------------------------------
    [homes]  
		comment = Home Directories  
		browseable = no  
		writable = yes  
	[printers]
		comment = All Printers  
		path = /var/spool/samba  
		browseable = no  
		guest ok = no  
		writable = no  
		printable = yes  
	[database]            添加进 /etc/samba/smb.conf  
        comment = Do not arbitrarily modify the database file  
        path = /home/database  
        public = no  
        writable = yes  
#### 4.3 pdbedit 命令管理SMB服务程序的账户信息数据库  
  -a 建立Samba账户  -x 删除Samba账户   -L 列出Samba账户   -Lv 列出账户详尽信息  
`[root@xy ~]# id xy`  
uid=1000(xy) gid=1000(xy) groups=1000(xy)    
`[root@xy ~]# pdbedit -a -u xy`         
new password:  
retype new password:  
Unix username:        xy  
NT username:            
Account Flags:          
User SID:             S-1-5-21-2962875295-3166855154-885293666-1000  
Primary Group SID:    S-1-5-21-2962875295-3166855154-885293666-513  
Full Name:            xy  
Home Directory:       \\xy\xy  
HomeDir Drive:        
Logon Script:         
Profile Path:         \\xy\xy\profile  
Domain:               XY  
Account desc:          
Workstations:           
Munged dial:            
Logon time:           0  
Logoff time:          Wed, 06 Feb 2036 23:06:39 CST  
Kickoff time:         Wed, 06 Feb 2036 23:06:39 CST  
Password last set:    Thu, 12 Jul 2018 11:09:43 CST  
Password can change:  Thu, 12 Jul 2018 11:09:43 CST  
Password must change: never  
Last bad password   : 0 
Bad password count  : 0  
Logon hours         : FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF  

`[root@xy ~]# mkdir /home/database`  
`[root@xy ~]# chown -Rf xy:xy /home/database/`  
`[root@xy ~]# ls -ld /home/database/`  
drwxr-xr-x. 2 xy xy 6 Jul 12 11:10 /home/database/  
`[root@xy ~]# semanage fcontext -a -t samba_share_t /home/database`     
`[root@xy ~]# restorecon -Rv /home/database/`  
restorecon reset /home/database context unconfined_u:object_r:home_root_t:s0->unconfined_u:object_r:samba_share_t:s0  
`[root@xy ~]# getsebool -a | grep samba`  
   samba_create_home_dirs --> off  
   samba_domain_controller --> off  
   samba_enable_home_dirs --> off                  
`[root@xy ~]# setsebool -P samba_enable_home_dirs on`    
`[root@xy ~]# systemctl restart smb`  
`[root@xy ~]# systemctl enable smb`  
ln -s '/usr/lib/systemd/system/smb.service' '/etc/systemd/system/multi-user.target.wants/smb.service'  
`[root@xy ~]# iptables -F`  
`[root@xy ~]# service iptables save`  
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]  

#### 4.4 Linux 访问文件共享服务(Samba客户端)  
+ 客户端安装 cifs-utils,配置认证文件  
`[root@xy ~]# vim auth.smb`  
   username=xy  
   password=redhat  
   domain=MYGROUP  
`[root@xy ~]# chmod 600 auth.smb`  
在Linux客户端创建用于挂载Samba服务共享资源目录,写入/etc/fstab

### 5. NFS基于TCP/IP协议，用于Linux系统之间上文件资源共享 C/S
#### 5.1 服务端 192.168.37.10  
`[root@xy ~]# iptables -F`  
`[root@xy ~]# service iptables save`  
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]   
创建用于共享的目录,chmod 777 /nfsfile 保证其有足够权限  
`[root@xy ~]# vim /etc/exports`   **nfs 配置文件**    
/nfsfile 192.168.37.10/24.*(rw,sync,root_squash) 

    参数:  
    ro 	只读  rw 只写  *不共存* 
    root_squash 			NFS客户端以root管理员访问,映射为NFS服务器的匿名用户  
    no_root_squash 			NFS客户端以root管理员访问,映射为NFS服务器的root管理员  
    all_squash 				无论NFS客户端以什么访问,均映射为NFS服务器的匿名用户  
    sync 					同时将数据写入内存与硬盘中,保证不丢失数据  
    async 					先将数据保存至内存,然后再写入硬盘,效率高,可能丢失数据  
`[root@xy ~]# systemctl restart rpcbind`   
`[root@xy ~]# systemctl enable rpcbind`  
`[root@xy ~]# systemctl start nfs-server`  
`[root@xy ~]# systemctl enable nfs-server`  
ln -s '/usr/lib/systemd/system/nfs-server.service' '/etc/systemd/system/  
nfs.target.wants/nfs-server.service'  

#### 5.1 客户端配置 `showmount`命令 192.168.37.20  
    -e 显示NFS服务器的共享列表 -a 显示本机挂载文件资源的情况 -v 显示版本
`[root@xy ~]# mkdir /nfsfile`  
`[root@xy ~]# mount -t nfs 192.168.37.10:/nfsfile /nfsfile` 
`[root@xy ~]# cat /nfsfile/readme`   
192.168.10.10:/nfsfile /nfsfile nfs defaults 0 0  

### 6. auto自动挂载服务  
`[root@xy ~]# yum install autofs`  
`[root@xy ~]# vim /etc/auto.master`     **autofs 的配置文件**  
eg. /media /etc/iso.misc   **挂载目录 子配置文件; 挂在目录是设备挂载位置的上一级目录**  
`[root@xy ~]# vim /etc/iso.misc`
iso      -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom   
挂载目录  挂载文件类型及权限                :设备名称  

### 7. 使用BIND提供域名解析服务  
+ DNS(Domain Name System):管理和解析域名 与 IP 的对应关系,域名--IP(正向解析),IP--域名(反向解析)  
+ DNS服务器:主服-masster、从服-slave、缓存服三种,DNS域名解析服务采用分布式数据结构,执行查询请求有 
+ 递归查询(必须向用户返回结果) 迭代查询(一台接一台,直到返回结果)  
   - 配置Bind(Berkely Internet Name Domain)服务:  
   - 主配置文件`/etc/named.conf`----定义bind服务程序的运行  
   - 区域配置文件`/etc/named.rfc1912.zones`----保存域与IP所在的具体位置  
   - 数据配置文件目录`/var/named`----保存域名与IP真实对应关系数据  

`[root@xy ~]# vim /etc/named.conf`
     
    listen-on port 53 { any; };  
    allow-query     { any; };  
`[root@xy ~]# vim /etc/named.rfc1912.zones` 
 
#### 7.1 正向解析 eg: 
    zone "localhost.localdomain" IN {  
       type master;  
       file "named.localhost";  
       allow-update { none; };  
    };  
      
#### 7.2 反向解析 eg:  
    zone "1.0.0.127.in-addr.arpa" IN {   // 写到主机位就可以  
       type master;  
       file "named.loopback";  
       allow-update { none; };  
    };  

`[root@xy named]# ls -al named.localhost`   
-rw-r-----. 1 root named 152 Jun 21  2007 named.localhost  
`[root@xy named]# cp -a named.localhost xy.com.zone`  
`[root@xy named]# vim xy.com.zone`  **正向解析 数据配置文件 (named.localhost)**

    $TTL 1D  
    @       IN SOA  xy.com. root.xy.com. (  
                                        0       ; serial  
                                        1D      ; refresh  
                                        1H      ; retry  
                                        1W      ; expire  
                                        3H )    ; minimum  
            NS      ns.xy.com.  
    ns      IN A    192.168.37.10  
            IN MX 10        mail.xy.com.  
    mail    IN A    192.168.37.10  
    www     IN A    192.168.37.10  
    bbs     IN A    192.168.37.20  

`[root@xy ~]# systemctl restart network`  **虚拟机环境下把本机DNS配置为IP**  
`[root@xy ~]# nslookup`     **测试**  
> www.xy.com  
Server:		192.168.37.10  
Address:	192.168.37.10#53  

Name:	www.xy.com  
Address: 192.168.37.10  
> bbs.xy.com  
Server:		192.168.37.10  
Address:	192.168.37.10#53  

Name:	bbs.xy.com  
Address: 192.168.37.20  

`[root@xy named]# ls -l named.loopback`  
-rw-r-----. 1 root named 168 Dec 15  2009 named.loopback 
 
`[root@xy named]# cp -a named.loopback 192.168.37.arpa`  
`[root@xy named]# vim 192.168.37.arpa`  **反向解析 数据配置文件 (named.loopback)**
  
    $TTL 1D  
    @       IN SOA  xy.com. root.xy.com. (  
                                         0       ; serial  
                                         1D      ; refresh  
                                         1H      ; retry  
                                         1W      ; expire  
                                         3H )    ; minimum  
            NS      ns.xy.com.  
            ns      A       192.168.37.10  
    10      PTR     ns.xy.com.  
    10      PTR     mail.xy.com.  
    10      PTR     www.xy.com.  
    20      PTR     bbs.xy.com.
  
`[root@xy named]# nslookup`   
> 192.168.37.10  
Server:		192.168.37.10  
Address:	192.168.37.10#53  

10.37.168.192.in-addr.arpa	name = www.xy.com.  
10.37.168.192.in-addr.arpa	name = mail.xy.com.  
10.37.168.192.in-addr.arpa	name = ns.xy.com.  
> 192.168.37.20  
Server:		192.168.37.10  
Address:	192.168.37.10#53  

20.37.168.192.in-addr.arpa	name = bbs.xy.com.  
> exit  
 
### 8. 部署从服务器  
#### 8.1 无加密的传输  
1. 主服务器的配置  
`[root@xy ~]# vim /etc/named.rfc1912.zones`   
**正向解析 eg** 
 
        zone "xy.com" IN {  
              type master;  
        	  file "xy.com.zone";  
              allow-update { 192.168.37.20; };  
		};  
**反向解析 eg**
  
        zone "37.168.192.in-addr.arpa" IN {      写到主机位就可以  
              type master;  
        	  file "192.168.37.arpa";  
        	  allow-update { 192.168.37.20; };  
    	};  	

2. 从服务器的配置  
`[root@xy ~]# vim /etc/named.rfc1912.zones`  
**正向解析 eg** 
 
        zone "xy.com" IN {  
        	 type slave;  
        	 master { 192.168.37.10; };  
        	 file "slave/xy.com.zone";  
		};  
**反向解析 eg**  

		zone "37.168.192.in-addr.arpa" IN {       **写到主机位就可以**  
        	 type slave;  
        	 master { 192.168.37.10;};  
        	 file "slaves/192.168.37.arpa";  
     	};  

#### 8.2 有加密的安全传输
1. 配置主服务器
`[root@xy ~]# dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slave` **主机下生成**  
Kmaster-slave.+157+39112  
`[root@xy ~]# ls -al Kmaster-slave.+157+39112.`  
-rw-------. 1 root root  56 Jul 13 13:10 Kmaster-slave.+157+39112.key  
-rw-------. 1 root root 165 Jul 13 13:10 Kmaster-slave.+157+39112.private  
`[root@xy ~]# cat Kmaster-slave.+157+39112.private`   
Private-key-format: v1.3  
Algorithm: 157 (HMAC_MD5)  
Key: pOuRN4eeVTzrZs/aaVpbkQ==      **key**  
Bits: AAA=  
Created: 20180713051051  
Publish: 20180713051051  
Activate: 20180713051051  

`[root@xy ~]# cd /var/named/chroot/etc/`  
`[root@xy etc]# vim transfer.key`
  
	key "master-slave" {  
        algorithm hmac-md5;  
        secret "pOuRN4eeVTzrZs/aaVpbkQ==";  
	};   
`[root@xy etc]# chown root:named transfer.key`  
`[root@xy etc]# chmod 640 transfer.key`   
`[root@xy etc]# ln transfer.key /etc/transfer.key `  
`[root@xy etc]# vim /etc/named.conf`
  
	include "/etc/transfer.key"   //9 行  
	allow-transfer { key master-slave; };  
2. 配置从服务器  
`[root@xy ~]# cd /var/named/chroot/etc/`  
`[root@xy etc]# vim transfer.key`
  
	key "master-slave" {  
        algorithm hmac-md5;  
        secret "pOuRN4eeVTzrZs/aaVpbkQ==";  
	};   
`[root@xy etc]# chown root:named transfer.key`  
`[root@xy etc]# chmod 640 transfer.key`  
`[root@xy etc]# ln transfer.key /etc/transfer.key`   
`[root@xy etc]# vim /etc/named.conf`
  
	include "/etc/transfer.key"   //9 行  
	server 192.168.37.10      //43行  
	{  
		key { master-slave; };   //45行  

### 9. 使用DHCP动态管理主机地址  
+ DHCP(Dynamic Host Configuration Protocol):基于UDP协议仅仅在局域网内部使用提高IP利用率,提高配置效率,降低管理维护成本  
+ 作用域:完整的IP段,DHCP协议根据作用域分配IP  
+ 超级作用域:	管理处于一个物理网络中多个逻辑子网段  
+ 排除范围: 把作用域的某些IP排除  
+ 地址池: 剩余的IP  
+ 租约: DHCP 客户端能够使用IP的时间  
+ 预约: 保证网络中特定设备总是获取到相同IP  
`[root@xy ~]# /etc/dhcp/dhcpd.conf`      
#### 9.1 dhcpd.conf 配置例子:  
	ddns-update-style interim；       **全局配置参数**  
	subnet 10.5.5.0 netmask 255.255.255.224 {                **子网段声明**  
  		range 10.5.5.26 10.5.5.30;
  		option domain-name-servers ns1.internal.example.org;   **地址配置选项**  
  		option domain-name "internal.example.org";
  		option routers 10.5.5.1;
  		option broadcast-address 10.5.5.31;
  		default-lease-time 600;                      **地址配置参数**  
  		max-lease-time 7200;  
	}  
#### 9.2 dhcpd服务程序配置最常见参数:  
    ddns-update-style [类型]           	DNS 服务更新类型 none interim ad-hoc  
    [allow | ignore] client-update 
    default-lease-time [21600]            默认超时时间  
    max-lease-time [43200]                 
    option domain-name-server [8.8.8.8]  
    option domain-name ["domain.org"]  
    range                                  定义分配的IP地址池  
    option subnet-mask                  定义客户端子网掩码 
    option routers                         定义客户端网关地址  
    broadcast-address [广播地址]          
    ntp-server [IP]                    定义客户端网络时间服务器(NTP)  
    nis-servers [IP]                   定义客户端NIS域服务器地址                  
    Hardware [网卡物理地址]             
    server-name [主机名]			向DHCP客户端通知DHCP服务器的主机名  
    fixed-address [IP]          将某个固定IP地址分配给指定主机
    time-offset [偏移误差]      
       
	ddns-update-style none;  
	ignore client-updates;  
	subnet 192.168.10.1 netmask 255.255.255.0 {  
  		range 192.168.10.50 192.168.10.150;  
  		option subnet-mask 255.255.255.0;  
  		option domain-name-servers 192.168.10.1;  
  		option domain-name "xy.com";  
  		option routers 192.168.10.1;         **网关**  
  		default-lease-time 21600;  
  		max-lease-time 43200;  
	}  

#### 9.3分配固定IP:IP与MAC地址绑定  
	host xy {  
		hardware ethernet 00:0c:29:11:26:05;  
		fixed-address 192.168.37.55;  
	}

### 10. 使用Postfix 与 Dovecot部署邮件系统  
+ 电子邮件协议:
- STMP(Simple Transfer Protocol):发送及中转邮件,占用服务器25/TCP端口  
- POP3(Post Office Protocol 3):将电子邮件存储到本地,占用服务器110/TCP端口  
- IMAP4(Internet Message  Access Protocol 4):用于本地主机上访问邮件,占用服务器143/TCP端口  
- MUA----SMTP---->MTA----SMTP---->MTA----POP3/IMAP4---->MUA  

#### 10.1 配置DNS服务程序bind-chroot  
`[root@xy ~]# iptables -F`  
`[root@xy ~]# service iptables save`  
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]  
`[root@xy ~]# systemctl disable iptables`  
`[root@xy ~]# systemctl disable firewalld.service`   
`[root@xy ~]# vim /etc/hostname`   
`[root@xy ~]# hostname`  
mail.xy.com  
`[root@mail named]# vim /etc/named.conf`
   
    listen-on port 53 { any; };  
    allow-query     { any; };  
`[root@mail named]# vim /etc/named.rfc1912.zones` 
 
	zone "xy.com" IN {  
         type master;  
         file "xy.com.zone";  
         allow-update { none; };  
	};  
`[root@mail named]# vim xy.com.zone`
   
	$TTL 1D
	@       IN SOA  xy.com. root.xy.com. (  
                                         0       ; serial  
                                         1D      ; refresh  
                                         1H      ; retry  
                                         1W      ; expire  
                                         3H )    ; minimum  
            NS              ns.xy.com.  
 	ns      IN A            192.168.37  
	@       IN MX 10        mail.xy.com.  
	mail    IN A            192.168.37.10  

#### 10.2 配置基于SMTP协议的Postfix服务程序  
`[root@mail ~]# vim /etc/postfix/main.cf`
  
	myhostname = mail.xy.com      			//76  
	mydomain = xy.com            			//83  
	myorigin = $mydomain           				//99  
	inet_interfaces = all            			//116  
	mydestination = $myhostname , $mydomain    //164  
	(relay_domains = $mydestination)  

`[root@mail named]# useradd boss`  
`[root@mail named]# echo "xy" | passwd --stdin boss`  
Changing password for user boss.  
passwd: all authentication tokens updated successfully.  
`[root@mail ~]# systemctl restart postfix`  
`[root@mail ~]# systemctl enable postfix`  

#### 10.3 配置基于POP3与IMAP4协议的Dovecot服务程序  
`[root@mail ~]# vim /etc/dovecot/dovecot.conf`

	protocols = imap pop3 lmtp         			// 24  
	disable_plaintext_auth = no        			//25  
	login_trusted_networks = 192.168.37.0/24     //48  

`[root@mail ~]# vim /etc/dovecot/conf.d/10-mail.conf`   
mail_location = mbox:~/mail:INBOX=/var/mail/%u  
`[root@mail ~]# su - boss`  
`[boss@mail ~]$ mkdir -p mail/.imap/INBOX`  
`[boss@mail ~]$ exit`  
logout  
`[root@mail ~]# systemctl restart dovecot`  
`[root@mail ~]# systemctl enable dovecot`  
ln -s '/usr/lib/systemd/system/dovecot.service' '/etc/systemd/system/multi-user.target.wants/dovecot.service'  