# Ubuntu安装wordpress


Ubuntu安装wordpress

<!--more-->

1. 安装 LNMP（Linux Nginx MySQL PHP）套件

    ```bash
    sudo apt-get update
    sudo apt-get install nginx mysql-server php-fpm php-mysql
    
    ```

2. 配置 MySQL 数据库和用户。

    ```bash
    sudo mysql -u root -p
    CREATE DATABASE wordpress;
    CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'password';
    GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
    FLUSH PRIVILEGES;
    exit
    ```

3. 下载 WordPress 并将其解压缩到 Nginx 网站目录中

    ```bash
    cd /var/www/
    sudo wget <https://wordpress.org/latest.tar.gz>
    sudo tar -zxvf latest.tar.gz
    sudo chown -R www-data:www-data /var/www/wordpress/
    
    ```

4. 配置 Nginx

    ```bash
    sudo vi /etc/nginx/sites-available/wordpress
    
    ```

   然后将以下配置文件粘贴到该文件中，注意将 domain_name 替换为你的域名或 IP 地址：

    ```
    server {
            listen 80;
            listen [::]:80;
            root /var/www/wordpress;
            index index.php index.html index.htm;
            server_name domain_name;
            location / {
                    try_files $uri $uri/ /index.php?$args;
            }
            location ~ \\.php$ {
                    include snippets/fastcgi-php.conf;
                    fastcgi_pass unix:/run/php/php7.4-fpm.sock;
            }
            location = /favicon.ico {
                    log_not_found off;
                    access_log off;
            }
            location = /robots.txt {
                    log_not_found off;
                    access_log off;
                    allow all;
            }
            location ~* \\.(css|gif|ico|jpeg|jpg|js|png)$ {
                    expires max;
                    log_not_found off;
            }
            error_page 404 /404.html;
            error_page 500 502 503 504 /50x.html;
            location = /50x.html {
                    root /usr/share/nginx/html;
            }
    }
    
    ```

   保存并退出。

5. 激活 Nginx Virtual Host

    ```bash
    sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
    sudo service nginx restart
    
    ```

6. 完成

   现在你可以通过在浏览器中输入你的域名或 IP 地址来访问 WordPress 站点，并按照安装向导完成 WordPress 的设置。

一些可能遇到的问题：

- 如果你已经安装了默认的Nginx站点并且监听了80端口，那么将会和新的WordPress站点 监-
  听同一个端口，导致冲突。在这种情况下，你需要关闭或删除默认站点以释放80端口。

  你可以通过执行以下命令来关闭默认站点：

    ```bash
    sudo unlink /etc/nginx/sites-enabled/default
    ```

  如果你想删除默认站点配置文件，则可以执行以下命令：

  {{< admonition >}}
  删除文件是永久性的操作，请谨慎执行。
  {{< /admonition >}}

    ```bash
    sudo rm /etc/nginx/sites-available/default
    ```


---

> 作者: WAKE  
> URL: https://weiqinke.com/2023/05/ubuntu-an-zhuang-wordpress/  

