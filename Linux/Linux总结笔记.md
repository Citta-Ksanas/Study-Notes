
### 基本常见命令

<font color="red">做以下所有操作之前记得先来一遍su提权命令,已经是root权限请忽略</font>

##### 基本操作命令： 

###### 提权命令

```
su -
```

###### 重启命令

```
reboot
```

###### 关机命令

```
poweroff
```

##### 安装图形化界面:

###### 登录用户

localhost login后面跟的是你设置的用户 没有设置用户就写root

###### 密码

* Password	后面跟的是密码（默认输入密码不可见）

* <font color="red">很多人安装Linux，安装完成一堆命令就懵了，在登录的情况下，输入以下命令安装图形化界面：</font>

```
yum groupinstall -y "GNOME Desktop" "Graphical Administration Tools"
```

```
ln -sf lib/systemd/system/runlevel5.target /etc/systemd/system/default.target
```

```
reboot
```

##### 主机命令：

###### 查看主机名

```
hostname
```

###### 设置主机名

* **永久修改**

```
hostnamectl set-hostname 主机名
```

##### 网络服务：

###### **查看IP信息**

```
ip addr
```

* <font color="red">127.0.0.1是本机地址，要看的是ens33网卡的信息</font>

###### **网络连通性测试**

```
ping [选项] 目标主机(即IP地址)
```

###### **设置网络信息**

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

* 配置信息保持如下：

* <font color="red">BOOTPROTO= dhcp 有两种选择：dhcp自动分配        static静态（如果要用SSH的话，建议静态，不然IP地址重启就会改变）</font>

* <font color="red">ONBOOT=yes  网卡自启动</font>

* 如果是static静态的话，请在下面加上以下信息：

* ```
  IPADDR=192.168.174.128
  NETMASK=255.255.255.0
  GATEWAY=192.168.174.2
  DNS1=114.114.114.114
  ```

  * 重启网卡服务


```
systemctl restart network
```

* 解析上面的配置文件信息：

  * BOOTPROTO= static   //设置IP地址固定
* ONBOOT=yes  //网卡自启动
  * IPADDR=192.168.174.128  //给虚拟机固定IP 192.168.174.128
* NETMASK=255.255.255.0  //子网掩码 固定255.255.255.0
  * GATEWAY=192.168.174.2  //网关地址 192.168.174.2
* DNS1 =114.114.114.114  //谷歌的域名解析服务 114.114.114.114
  

其中NETMASK和DNS1是固定的；

IPADDR和GATEWAY要看具体信息和网关决定！

IPADDR是你自己给定的虚拟机IP地址；

  GATEWAY是网关地址；

  在虚拟机中—>菜单栏—>编辑–>虚拟网络编辑器–>选择VMnet8 NAT模式（后面能看到分配的IP）–>在下面选择NAT设置，填写网关IP–>应用即可！（网关IP就是上一步VMnet8 NAT自动分配的IP改最后一组数据即可）比如：VMnet8 NAT模式看到的IP是：192.168.174.0   网关地址就是：192.168.174.2	自己设置虚拟机里Linux的IP也是改最后一位 比如我的是192.168.174.128

* 改完之后在本机运行CMD命令窗口，输入ping 你虚拟机的地址 返回正确即可！

#####  文本编辑器：

* vim编辑器，习惯上也叫vi
* 怎么退出？按一下Esc 此时光标会在最低下，输入:wq即可！（如遇错误，请提权~）
* <font color="red">如果使用vim报错的话执行一次 yum install -y vim就好了</font>

###### 插入命令

| 命令 |           说明           |
| :--: | :----------------------: |
|  i   |       在光标前插入       |
|  ｘ  | 把光标对应位置的字符删除 |

###### **定位命令**

|        命令        |       说明       |
| :----------------: | :--------------: |
|   ：ｓｅｔ　ｎｕ   |     显示行号     |
| ：ｓｅｔ　ｎｏｎｕ |   取消显示行号   |
|        ｇｇ        |  到文本的第一行  |
|         Ｇ         | 到文本的最后一行 |
|        ：ｎ        |  到文本的第ｎ行  |

###### 替换命令

|       命令       |                    说明                     |
| :--------------: | :-----------------------------------------: |
|   :s /old/new    | 将当前行中查找到的第一个字符old 串替换为new |
|  :s /old/new/g   |  将当前行中查找到的所有字符串old替换为new   |
| :#,# s/old/new/g |   在行号#,#范围内替换所有的字符串old为new   |
|  :% s/old/new/g  |  在整个文件范围内替换所有的字符串old为new   |
|   :%s/old/new    |  查找文件中所有行第一次出现的old替换为new   |

###### **其他命令**

| 命令 |           说明           |
| :--: | :----------------------: |
|  :W  |  [文件路径]保存当前文件  |
|  :q  | 如果未对文件做改动则退出 |
| :q!  |     放弃存储强制退出     |
|  :w  |           保存           |
| :wq! |      保存并强制退出      |

##### 解压缩命令：

###### 解压文件

```
tar -zxvf xxx.tar	
```

* xxx.tar指的是你要解压的文件

###### 压缩文件

```

```

### 一、安装以及配置SSH服务

* <font color="red">做以下所有操作之前记得先来一遍su提权命令,还要保证有网络（不然就是安装寂寞）；</font>
* <font color="red">注意执行顺序，测试服务的时候，服务必须处于开启状态！！！（最怕无脑CV，属于刚开启就关闭的那种）；</font>

##### 查看本机SSH服务是否可用：

```
rpm -qa | grep ssh
```

* 一般都处于可用状态，SSH服务默认安装（<font color="red">什么都不出现就是没有安装的意思！！！</font>）

##### 安装SSH服务：

```
yum install -y openssh-service
```

##### 释放监听端口，监听地址：

* 编辑SSH配置文件进行释放操作

```
vi /etc/ssh/sshd_config
```

* 修改以下配置信息（去掉1、3、4行前面的#号即可）

```
#Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

* 开启远程登录

```
PermitRootLogin yes
```

* 使用用户名密码作为连接验证：

```
PasswordAuthentication yes
```

##### 开启SSH服务：

```
service sshd start
```

##### 关闭SSH服务：

```
service sshd stop
```

##### 检查SSH服务是否开启：

```
ps -e | grep shhd
```

##### 配置SSH服务自启动：

```
systemctl enable sshd.service
```

##### 连接SSH遇到的问题：

* 修改了网卡信息，重启了网卡服务，ip addr 命令 IP地址出不来的情况？

```
vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

* reboot<font color="red"> 重启 </font>一下，还是不行的话检查网关和IP设定！！！

### 二、安装以及配置http服务

```
su -
```

##### 查看本机http服务是否可用：

```
rpm -qa | grep  httpd
```

##### 安装Apache：

```
yum install -y httpd
```

##### 开启http服务:

```
service httpd start
```

##### 停止http服务：

```
service httpd stop
```

##### 允许防火墙通过80端口：

```
firewall-cmd   --zone=public   --add-port=80/tcp   --permanent
```

* 出现success表明添加成功

  --zone 作用域

  --add-port=80/tcp  添加端口，格式为：端口/通讯协议

  --permanent   永久生效，没有此参数重启后失效

##### 重启防火墙：

```
firewall-cmd --reload
```

##### 测试http服务：

本地计算机启动浏览器，输入虚拟机IP:80		出现页面即成功

（也可以直接输入虚拟机IP，浏览器默认访问的就是80端口）

#####  安装http遇到的问题：

* 出现无法访问的情况
* 检查一下防火墙是否放行了80端口，是否重启了防火墙

### 三、防火墙的各项配置

##### 开启防火墙：

```
systemctl start firewalld.service
```

##### 关闭防火墙：

```
systemctl stop firewalld.service
```

##### 重启防火墙：

```
systemctl restart firewalld.service
```

##### 禁止防火墙开机启动：

```
systemctl disable firewalld.service
```

### 四、安装以及配置ftp服务

##### 查看本机ftp服务是否可用：

```
rpm -qa | grep  vsftpd
```

##### 安装ftp:

```
yum install -y vsftpd
```

##### 启动ftp服务：

```
systemctl start vsftpd
```

##### 检查ftp服务是否启动：

```
ps -ef|grep vsftpd
```

##### 把ftp服务添加进开机启动项：

```
systemctl  enable vsftpd
```

#####  添加ftp服务：

```
firewall-cmd --permanent --zone=public --add-service=ftp
```

##### 重启防火墙：

```
firewall-cmd --reload
```

##### 添加用户：

```
useradd -g root -d /home/Citta-Ksana -s /sbin/nologin Citta-Ksana
```

新建Citta-Ksana用户，添加到root组；

不允许用户登录，只允许ftp登录；

ftp默认登陆后的目录是/home/Citta-Ksana;

##### 设置用户密码：

```
passwd Citta-Ksana
```

##### 设置权限：

```
chown -R Citta-Ksana:root /home/Citta-Ksana
setsebool -P ftpd_full_access on
```

##### 修改vsftp配置文件，禁止匿名登录：

```
vi /etc/vsftpd/vsftpd.conf
```

* 把：anonymous_enable=YES 改为： anonymous_enable=NO
* 输入--->  :wq!  保存退出；

##### <font color="red">修改认证文件，防止出现530登录错误：</font>

* 也就是用本机CMD测试的时候会提示530，用网页测试的时候，会要求重复登录！

```
vi /etc/pam.d/vsftpd
```

* 把指定行注释掉即可：

```
auth   required   pam_shells.so   改成  #auth   required   pam_shells.so
```

##### 重启ftp服务，使配置文件生效：

```
systemctl restart vsftpd.service
```

##### 测试ftp服务：

本地计算机启动浏览器，输入ftp://虚拟机IP	出现登录页面即成功；

也可以使用本机CMD输入ftp 虚拟机IP 	会要求你登录，出现ftp>即可；

##### 安装ftp遇到的问题：

* 按照上面步骤配置了却还是在网页中显示不出来

```
reboot
```

* 解决方案：万能的重启大法，试一下！（配置了开机启动就不需要手动开启了哈）

### 五、安装以及配置MySQL5.7

* 说明：我们安装MySQL位置在 /usr/local 下

```
cd /usr/local
```

##### 安装wget命令 ：

```
yum install -y wget
```

##### 下载MySQL 安装包：

```
wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
```

##### 安装mysql 安装源：

```
yum -y localinstall mysql57-community-release-el7-11.noarch.rpm 
```

##### 在线安装MySQL:

```
yum -y install mysql-community-server
```

* 等待时间会比较久（喝杯奶茶慢慢等~ 恋爱可以慢慢谈，奶茶必须马上喝）；

##### 启动mysql 服务：

```
systemctl start mysqld
```

##### 设置开机启动：

```
systemctl enable mysqld
systemctl daemon-reload
```

##### 修改root登录密码：

```
vi /var/log/mysqld.log
```

* mysql安装完成之后，会在/var/log/mysqld.log文件中给root生成了一个临时的默认密码；
* root@localhost：后面跟的就是mysql默认生成的数据库密码（暂时记住）；

##### 修改root 密码：

```
mysql -u root -p
```

* 输入刚才看到的默认密码，密码默认不可见；

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'LeGendsCy&1024';
```

* 修改数据库密码为LeGendsCy&1024，用户为root；

```mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'LeGendsCy&1024' WITH GRANT OPTION;
```

* 设置允许远程登录；

##### 退出mysql：

```
exit
```

##### 防火墙开放3306端口：

```
firewall-cmd   --zone=public   --add-port=3306/tcp   --permanent
```

##### 重启防火墙：

```
firewall-cmd --reload
```

##### 配置mysql默认编码为utf-8：

* 为什么设置utf-8，却写成utf8呢？
* 因为MySQL官方当时出的utf-8编码不全，有丢失；后面为了补足，就干脆出了个utf8编码（原来utf-8的修正补全版本）所以才会说MySQL的utf8才是真正意义上的utf-8编码；

```
vi /etc/my.cnf
```

* 在[mysqld]下面的行添加以下语句：

```mysql
character_set_server=utf8
init_connect='SET NAMES utf8'
```

##### 重启MySQL：

```mysql
systemctl restart mysqld
```

##### root 用户登录查看编码：

```mysql
mysql -u root -p
```
* 输入MYSQL的密码；
```mysql
show variables like '%character%';
```

* 此时会看到编码都成了utf8；

```mysql
exit
```

* 大功告成，MYSQL配置完成；

##### 安装MySQL遇到的问题：

* 设置密码的时候提示：

```mysql
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```

出错原因：密码太简单了，当然上面的密码不存在这个情况哈！

