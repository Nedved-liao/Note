```
[root@DB3 ~]#yum install httpd php php-mysql -y
```
 
## 解包并指定释放路径

 phpMyAdmin-4.7.5-all-languages.tar.gz

 
```
[root@DB3 ~]#tar -xvf phpMyAdmin-4.7.5-all-languages.tar.gz -C /var/www/html/

```
 
## 重命名解压得到目录以方便访问

 
```
[root@DB3 html]# mv phpMyAdmin-4.7.5-all-languages phpadmin
```
 
## 安全起见更改权限

 
```
[root@DB3 html]# chown -R apache:apache phpadmin
```
 
## 准备phpmyadmin配置文件

 
```
[root@DB3 phpadmin]# cp config.sample.inc.php config.inc.php 
```
 
## 修改配置文件

 修改config.inc.php的第17行和31行

 
```
[root@DB3 phpadmin]# vim config.inc.php
```
 
```
 17 $cfg['blowfish_secret'] = 'heya'; /* YOU MUST FILL IN THIS FOR COOKIE AUTH! */
 31 $cfg['Servers'][$i]['host'] = 'localhost';
```
 17行的需要在$cfg[‘blowfish_secret’] = ’ ’ 在两单引号间随便添加字符，作用是COOKIE缓存验证。   
 31行则是改动[‘host’] = ‘localhost’; localhost改为你数据库的IP地址，内德的数据是在本机上所以就时localhost啦。

 
## 起服务，并在客户端测试

 
```
[root@DB3 phpadmin]#systemctl restart httpd
```
 客户端测试，就随便找一台能够连上刚设置的机子测试就可以了   
 在浏览器中输入http://机子的IP/phpadmin/

   
  