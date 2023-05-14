# Ubuntu安装zabbix

#### 1、安装6.4版本zabbix，以 Ubuntu 22.04、MySql、Nginx为例

```Bash
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-nginx-conf zabbix-sql-scripts zabbix-agent
```

#### 2 、安装MySql

下载[https://dev.mysql.com/downloads/repo/apt/](https://dev.mysql.com/downloads/repo/apt/)并上传

或者[http://mirrors.ustc.edu.cn/mysql-repo/](http://mirrors.ustc.edu.cn/mysql-repo/)

```Bash
sudo dpkg -i mysql-apt-config_0.8.24-1_all.deb
sudo apt update
sudo apt install mysql-server
```

安装输入数据库ROOT密码，其他保持默认即可

#### 3、配置数据库

```SQL
mysql -uroot -p

create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'zabbix数据库密码';
grant all privileges on zabbix.* to zabbix@localhost;
set global log_bin_trust_function_creators = 1;
quit;
```

#### 4、在Zabbix服务器主机上导入初始数据

```Bash
sudo zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

输入zabbix数据库密码

#### 5、导入数据库架构后禁用log_bin_trust_function_creators选项。

```Bash
mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;
```

#### 6、修改zabbix的配置文件

```Bash
sudo vi /etc/zabbix/zabbix_server.conf
```

```XML
DBPassword=zabbix数据库密码
```

#### 7、修改nginx配置，修改注释和端口号

```Bash
sudo vi /etc/zabbix/nginx.conf
sudo vi /etc/nginx/sites-available
```

```XML
 listen 80;
 server_name example.com;
```

#### 8添加中文支持，取消UTF-8的中文注释

```Bash
 sudo vi /etc/locale.gen 
 sudo locale-gen
```

可能需要root账户执行locale-gen。。

#### 9、重启服务设置自启动

```Bash
sudo systemctl restart zabbix-server zabbix-agent nginx php8.1-fpm
sudo systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
```









参考链接：





---

> 作者: WAKE  
> URL: https://weiqinke.com/2023/05/ubuntu-an-zhuang-zabbix/  

