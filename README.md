# arch-nginx-php-fpm-mariadb
how to install LEMP in Arch Linux for Local Development

Update your system
> $sudo pacman -Syu or sudo pacman -Syyu

First Install Nginx Web Server
$sudo pacman -S nginx
$sudo systemctl start nginx
$sudo systemctl enable nginx
$sudo systemctl status nginx

Next Install MariaDB
$sudo pacman -S mariadb

Once installed, proceed & initialize the MariaDB data directory and create system tables
$sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

$sudo systemctl start mariadb
$sudo systemctl enable mariadb
$sudo systemctl status mariadb

Run the below to begin Mariadb
$sudo mysql_secure_installation

Once you in follow this Steps:

Enter current password for root (Enter for none)

Set root password: Set n

Remove anonymous users: y

Disallow root login remotely: y

Remove test database and access to it: y

Reload privilage tables now : y

Next you need to remove the native password if you want no password login in Mariadb, follow this code:
$sudo mysql

ALTER USER root@localhost IDENTIFIED VIA mysql_native_password USING PASSWORD("");

Now it's time to install PHP server-side scripting language.
$sudo pacman -S php php-fpm
$sudo systemctl start php-fpm
$sudo systemctl enable php-fpm
$sudo systemctl status php-fpm

To list all available PHP module
$sudo pacman -S php[TAB]
$sudo pacman -Ss | grep php

Then configure php.ini file to include necessary extension needed by
$sudo nano /etc/php/php.ini

Locate with [CTRL + W] keys(if you use nano editor) and uncomment(remove ; at the line beginning) extension you want.
ex:
extension=mysqli.so

Next step is to enable PHP-FPM FastCGI on localhost Nginx Directive
$sudo mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak
$sudo nano /etc/nginx/nginx.conf

Add the whole following context on nginx.conf
You will need change the root directory if you have different one.
Mine was in the /srv/http

#user html;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       80;
        server_name  localhost;
            root   /srv/http;
        charset koi8-r;
        location / {
        index  index.php index.html index.htm;
                                autoindex on;
                                autoindex_exact_size off;
                                autoindex_localtime on;
        }

        error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /srv/http;
        }

        location ~ \.php$ {
          #fastcgi_pass 127.0.0.1:9000; (depending on your php-fpm socket configuration)
          fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
          fastcgi_index index.php;
          include fastcgi.conf;
        }

        location ~ /\.ht {
            deny  all;
        }
    }            
}

if you have a Codeigniter Framework. to run it you will need add this code inside in server{}

location /ci-folder/ {
  try_files $uri /ci-folder/index.php$is_args$args;
}

Full Example of Nginx.conf:

#user html;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    #tcp_nopush     on;
    #keepalive_timeout  0;
    keepalive_timeout  65;
    gzip  on;

    server {
        listen       80;
        server_name  localhost;
            root   /srv/http;
        charset koi8-r;
        location / {
        index  index.php index.html index.htm;
                                autoindex on;
                                autoindex_exact_size off;
                                autoindex_localtime on;
        }

        error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   /srv/http;
        }
        
        location /ci-folder/ {
          try_files $uri /ci-folder/index.php$is_args$args;
        }

        location ~ \.php$ {
          #fastcgi_pass 127.0.0.1:9000; (depending on your php-fpm socket configuration)
          fastcgi_pass unix:/run/php-fpm/php-fpm.sock;
          fastcgi_index index.php;
          include fastcgi.conf;
        }

        location ~ /\.ht {
            deny  all;
        }
    }         
}

After All Configuration had been. all you need is to restart Nginx and PHP-FPM

$ sudo systemctl restart php-fpm
$ sudo systemctl restart nginx
