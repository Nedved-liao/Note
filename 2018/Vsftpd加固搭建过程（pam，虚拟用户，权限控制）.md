# 部署

 
# 一.卸载系统已经存在的ftp服务器（可选）

 
## 1.1 已经存在的ftp服务器是源码安装

 
```bash
find / | grep vsftp*
rm -f /usr/local/sbin/vsftpd 
rm -f /usr/local/man/man5/vsftpd.conf.5
rm -f /usr/local/man/man8/vsftpd.8
rm -f /etc/xinetd.d/vsftpd
rm -rf /etc/vsftpd  

```
 
## 1.2 已经存在的ftp服务器是YUM安装

 
```bash
yum remove vsftpd -y
rm -rf /etc/vsftpd

```
 
> **"** 可以通过 yum list | grep vsftpd 来判断是否是yum安装的
> 
>  
 
# 二.安装vsftpd及相关依赖包

 
## 2.1 YUM安装

 
### 2.1.1 vsftpd安装程序

 `yum install -y vsftpd` 

 
### 2.1.2 vsftpd虚拟登陆账户必要依赖包

 centos6  
  `yum install -y pam* db4*`   
 centos7  
 yum install -y pam* libdb-utils*

 
## 2.2 源码安装

 
### 2.2.1 官网下载源码包

 Download (HTTP)：[https://security.appspot.com/downloads/vsftpd-3.0.2.tar.gz](https://security.appspot.com/downloads/vsftpd-3.0.2.tar.gz)  
  `wget https://security.appspot.com/downloads/vsftpd-3.0.2.tar.gz` 

 
### 2.2.2 解压，配置环境

 
```bash
tar -xf /tmp/vsftpd-3.0.2.tar.gz
cd /tmp/vsftpd-3.0.2
make && make install
mkdir /etc/vsftpd/
install -v -m 644 vsftpd.conf   /etc/vsftpd/

```
 
## 三.修改配置文件

 
### 3.1 生成虚拟用户，和配置pam验证登录

 
```bash
#生成一个系统用户来映射虚拟用户
groupadd -g 47 vsftpd                         
useradd -c "vsftpd User"  -d /var/vsftpd -g vsftpd -s /sbin/nologin -u 47 vsftpd   

#生成配置虚拟用户信息文件
cat > /etc/vsftpd/vuser.txt << "EOF"


user1
123
user2
123
user3
123
EOF

#添加虚拟用户密码db文件，并删除vuser脚本
cat > /etc/vsftpd/adduser.sh << EOF
#bin/bash
db_load -T -t hash -f /etc/vsftpd/vuser.txt /etc/vsftpd/login.db
chmod 600 /etc/vsftpd/login.db
rm -f /etc/vsftpd/vuser.txt
EOF

#生成pam验证配置文件
cat > /etc/pam.d/vsftpd << "EOF" 
#%PAM-1.0
auth required pam_userdb.so db=/etc/vsftpd/login
account required pam_userdb.so db=/etc/vsftpd/login
EOF

#db_load生成
chmod +x /etc/vsftpd/adduser.sh
sh /etc/vsftpd/adduser.sh


```
 **Note**

 
> vuser.txt文件是文明保存，存在安全问题。  
>   
>  所以每次使用db_load生成db文件后都删除vuser和设置db文件权限为600
> 
>  
 
### 3.2 修改主配置文件

 
```bash

cat > /etc/vsftpd/vsftpd.conf << "EOF"
#禁止匿名登录
anonymous_enable=NO
local_enable=YES
write_enable=YES
dirmessage_enable=YES
anon_umask=022
#启动log
xferlog_enable=YES
xferlog_std_format=YES
#开启虚拟用户
guest_enable=YES
guest_username=vsftpd
#PAM认证文件/etc/pam.d/vsftpd
pam_service_name=vsftpd
#端口设置（默认使用21）
listen=YES
listen_port=21
connect_from_port_20=YES
#用户权限配置目录
user_config_dir=/etc/vsftpd/user.d
#设定支持ASCII模式的上传和下载功能
ascii_upload_enable=YES
ascii_download_enable=YES
#被动模式
pasv_enable=YES
pasv_max_port=1028
pasv_min_port=1024
EOF

```
 
### 3.2.3 生成用户权限配置文件

 
```bash
#生成用户权限配置目录
mkdir /etc/vsftpd/user.d

#生成对应的配置文件
#以下提供三个权限模板

cat > /etc/vsftpd/user.d/user1 << EOF
#有上传/下载/修改权限
anon_world_readable_only=NO
write_enable=YES
anon_mkdir_write_enable=YES
anon_upload_enable=YES
anon_other_write_enable=YES
local_root=/var/vsftpd/user1
EOF

cat > /etc/vsftpd/user.d/user2 << EOF
#只有上传/修改权限
anon_world_readable_only=NO
write_enable=YES
anon_mkdir_write_enable=YES
anon_upload_enable=YES
local_root=/var/vsftpd/user2
EOF

cat > /etc/vsftpd/user.d/user3 << EOF
#只有下载权限
anon_world_readable_only=NO
local_root=/var/vsftpd/user3
EOF


#为每个虚拟用户生成对应存储路径
mkdir -p /var/vsftpd/user{1,2,3}
chown -R 47:47 /var/vsftpd

```
 **Note**

 
> user.d 下面的配置文件名必须和vuser.txt中添加的虚拟用户名称一致  
> 
> 
>  
 
### 3.2.4 配置不可登录名单

  
  2. YUM安装的自动生成在/etc/vsftpd/user_list 
  4. 源码安装的需要自行生成  
      user_list生成（copy自yum生成的user_list）  
```bash
cat > /etc/vsftpd/user_list << EOF
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
EOF


```
 
## 4.1 适用源码安装 生成service 启动文件（YUM可选）

 
```bash
cat  > /etc/init.d/vsftpd << EOF
#!/bin/bash
#
# vsftpd      This shell script takes care of starting and stopping
#             standalone vsftpd.
#
# chkconfig: - 60 50
# description: Vsftpd is a ftp daemon, which is the program
#              that answers incoming ftp service requests.
# processname: vsftpd
# config: /etc/vsftpd/vsftpd.conf
# Source function library.
. /etc/rc.d/init.d/functions
# Source networking configuration.
. /etc/sysconfig/network
# Check that networking is up.
[ ${NETWORKING} = "no" ] && exit 0
[ -x /usr/local/sbin/vsftpd ] || exit 0
RETVAL=0
prog="vsftpd"
start() {
        # Start daemons.
        if [ -d /etc/vsftpd ] ; then
                for i in `ls /etc/vsftpd/*.conf`; do
                        site=`basename $i .conf`
                        echo -n $"Starting $prog for $site: "
                        /usr/local/sbin/vsftpd $i &
                        RETVAL=$?
                        [ $RETVAL -eq 0 ] && {
                           touch /var/lock/subsys/$prog
                           success $"$prog $site"
                        }
                        echo
                done
        else
                RETVAL=1
        fi
        return $RETVAL
}
stop() {
        # Stop daemons.
        echo -n $"Shutting down $prog: "
        killproc $prog
        RETVAL=$?
        echo
        [ $RETVAL -eq 0 ] && rm -f /var/lock/subsys/$prog
        return $RETVAL
}
# See how we were called.
case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart|reload)
        stop
        start
        RETVAL=$?
        ;;
  condrestart)
        if [ -f /var/lock/subsys/$prog ]; then
            stop
            start
            RETVAL=$?
        fi
        ;;
  status)
        status $prog
        RETVAL=$?
        ;;
  *)
        echo $"Usage: $0 {start|stop|restart|condrestart|status}"
        exit 1
esac
exit $RETVAL
EOF

chmod +x /etc/init.d/vsftpd

```
 
### 可能遇到问题

 500 OOPS: cannot change directory:/home/…  
 关闭selinux和设置上下文

 
```bash
setenforce 0
setsebool -P ftp_home_dir=1
setsebool -P allow_ftpd_full_access 1

```
   
  