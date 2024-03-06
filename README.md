# LDNMP 一套完整的网站部署方案

感谢 https://github.com/kejilion/docker/tree/main KEJILION的Yutube分享，大大提高了网站的运行效率，摆脱宝塔的限制，在AWS免费VPS中，宝塔基本上就把内存耗尽了。因此必须采用下述部署方案。

L- Linux 主流的有Ubuntu、Debian、CentOS

D- Docker, 使用容器技术部署

N- Nginz 流行的网站代理和反代理软件

M- MySQL 数据酷

P- Php  包含最新版本和Php7.4

同时包含 Redis 作为缓存数据酷

### 整体项目目录结构：

/home/web

    - html  
      - web1  (建议用域名作为目录)
      - web2
      ...
    - certs
    - conf.d (nginx配置目录，每个王章一个单独的配置文件，并且以网站域名作为文件名）
    - mysql
    - redis

### 第一步， 系统升级

apt update && apt upgrade 

apt install wget socat unzip htop btop

### 第二步， 安装docker 和 docker-compose 

ubuntu:  apt install docker.io 

centos:  yum install docker.io

curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

修稿权限

chmod +x /usr/local/bin/docker-compose

### 第三步： 开放端口 

iptables -P INPUT ACCEPT

iptables -P FORWARD ACCEPT

iptables -P OUTPUT ACCEPT

iptables -F

### 第四步： 申请整数

#### 4.1  通过[wwww.cloudclare.cin](https://www.cloudflare.com/zh-cn/) 来申请，Cloudflare是全球最大的域名解析服务商。

#### 4.2  通过命令行申请

证书申请

curl https://get.acme.sh | sh

~/.acme.sh/acme.sh --register-account -m xxxx@gmail.com --issue -d web1.kjlion.link  -d web2.kjlion.link  -d web3.kjlion.link -d web4.kjlion.link -d web5.kjlion.link -d web6.kjlion.link --standalone --key-file /home/web/certs/key.pem --cert-file /home/web/certs/cert.pem  --force

### 第五步， 编辑docker-compose.yml文件

参考模板： [docker-compose.yml](https://github.com/Duka-Talk/LDNMP/blob/main/docker-compose.yml)

### 第六步， 编辑nginx配置文件

参考模板： [nginx.conf](https://github.com/Duka-Talk/LDNMP/blob/main/nginx.conf)

### 第七步， 下载网站源码

#### WordPress

cd /home/web/html/ && mkdir web1 && cd web1 &&  wget https://cn.wordpress.org/wordpress-6.2.2-zh_CN.zip && unzip wordpress-6.2.2-zh_CN.zip && rm wordpress-6.2.2-zh_CN.zip

#### 可道云网盘

cd /home/web/html/ && mkdir web2 && cd web2 &&  wget https://static.kodcloud.com/update/download/kodbox.1.42.zip && unzip kodbox.1.42.zip && rm kodbox.1.42.zip

#### discuz论坛/迪库兹论坛

cd /home/web/html/ && mkdir web3 && cd web3 && wget https://github.com/kejilion/Website_source_code/raw/main/Discuz_X3.5_SC_UTF8_20230520.zip && unzip Discuz_X3.5_SC_UTF8_20230520.zip && rm Discuz_X3.5_SC_UTF8_20230520.zip


#### 苹果cms

cd /home/web/html && mkdir web4 && cd web4 && wget https://github.com/magicblack/maccms_down/raw/master/maccms10.zip && unzip maccms10.zip && rm maccms10.zip

cd /home/web/html/web4/maccms10-master/template/ && wget https://github.com/kejilion/Website_source_code/raw/main/DYXS2.zip && unzip DYXS2.zip && rm /home/web/html/web4/maccms10-master/template/DYXS2.zip 

cp /home/web/html/web4/maccms10-master/template/DYXS2/asset/admin/Dyxs2.php /home/web/html/web4/maccms10-master/application/admin/controller 

cp /home/web/html/web4/maccms10-master/template/DYXS2/asset/admin/dycms.html /home/web/html/web4/maccms10-master/application/admin/view/system

mv /home/web/html/web4/maccms10-master/admin.php /home/web/html/web4/maccms10-master/vip.php && wget -O /home/web/html/web4/maccms10-master/application/extra/maccms.php https://raw.githubusercontent.com/kejilion/Website_source_code/main/maccms.php

### 第八步， 运行docker-compose部署

#### 运行
cd /home/web && docker-compose up -d

#### 赋予权限
docker exec -it nginx chmod -R 777 /var/www/html

docker exec -it php chmod -R 777 /var/www/html

docker exec -it php74 chmod -R 777 /var/www/html

### 第九步，安装Php和Php74扩展

#### 安装PHP扩展，调整上传文件大小限制，内存限制

docker exec php apt update && docker exec php apt install -y libmariadb-dev-compat libmariadb-dev libzip-dev libmagickwand-dev imagemagick

docker exec php docker-php-ext-install mysqli

docker exec php pecl install redis && docker exec php sh -c 'echo "extension=redis.so" > /usr/local/etc/php/conf.d/docker-php-ext-redis.ini'

#### php7.4安装PHP扩展，调整上传文件大小限制，内存限制

docker exec php74 apt update && docker exec php74 apt install -y libmariadb-dev-compat libmariadb-dev libzip-dev libmagickwand-dev imagemagick

docker exec php74 docker-php-ext-install mysqli

docker exec php74 pecl install imagick && docker exec php74 sh -c 'echo "extension=imagick.so" > /usr/local/etc/php/conf.d/imagick.ini'

#### 重启php 

docker restart php

docker restart php74

#### 查看php扩展安装情况

docker exec -it php php -m

docker exec -it php74 php -m

### 第十步， 数据库操作

#### 创建多个新数据库

docker exec -it mysql mysql -u root -p

CREATE DATABASE web1;

CREATE DATABASE web2;

CREATE DATABASE web3;

CREATE DATABASE web4;

#### 查看数据库列表

SHOW DATABASES;

#### 数据库赋予权限

GRANT ALL PRIVILEGES ON web1.* TO 'web_admin'@'%';

GRANT ALL PRIVILEGES ON web2.* TO 'web_admin'@'%';

GRANT ALL PRIVILEGES ON web3.* TO 'web_admin'@'%';

GRANT ALL PRIVILEGES ON web4.* TO 'web_admin'@'%';

#### 查看权限赋予情况

SHOW GRANTS FOR 'web_admin'@'%';

#### 删除数据库

DROP DATABASE web3;

REVOKE ALL PRIVILEGES ON web3.* FROM 'web_admin'@'%';

#### WP 安装完成之后，需要修稿WP的配置

echo "define('FS_METHOD', 'direct'); define('WP_REDIS_HOST', 'redis'); define('WP_REDIS_PORT', '6379');" >> /home/web/html/web1/wordpress/wp-config.php

也可以用VIM编辑，手动修改。
