# image
FROM registry.cn-hangzhou.aliyuncs.com/pixseed/centos7_base

# author
MAINTAINER WisdomFusion <WisdomFusion@gmail.com>

# repo

RUN mv /etc/yum.repos.d/CentOS-Base.repo{,.backup}
RUN wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo 

RUN yum -y install epel-release

# update

RUN yum clean all && yum makecache && yum -y update

# install compile tools and libs

RUN yum -y install gcc gcc-c++ autoconf automake cmake zlib zlib-devel compat-libstdc++-33 glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel unzip zip nmap ncurses ncurses-devel sysstat ntp curl curl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libtiff-devel gd gd-devel libxml2 libxml2-devel libXpm libXpm-devel libmcrypt libmcrypt-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel openldap openldap-devel nss_ldap openldap-clients openldap-servers pam-devel libicu libicu-devel

# install tools

RUN yum -y install man wget net-tools iptraf iotop vim-enhanced bash-completion

# Vim settings

RUN alias vi='vim' \
  && echo "alias vi='vim'" >> ~/.bashrc \
  && echo "set fencs=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936" >> ~/.vimrc \
  && echo "set fileformats=unix,dos" >> ~/.vimrc

# copy packages

COPY packages/* /usr/src/

# install libs

RUN cd /usr/src/ \
  && tar zxvf libmcrypt-2.5.8.tar.gz \
  && cd libmcrypt-2.5.8/ \
  && ./configure && make && make install \
  && /sbin/ldconfig \
  && cd libltdl/ \ 
  && ./configure --enable-ltdl-install \
  && make && make install

RUN cd /usr/src/ \
  && tar zxvf mhash-0.9.9.9.tar.gz
  && cd mhash-0.9.9.9/
  && ./configure && make && make install

RUN ln -s /usr/local/lib/libmcrypt.la /usr/lib/libmcrypt.la \
  && ln -s /usr/local/lib/libmcrypt.so /usr/lib/libmcrypt.so \
  && ln -s /usr/local/lib/libmcrypt.so.4 /usr/lib/libmcrypt.so.4 \
  && ln -s /usr/local/lib/libmcrypt.so.4.4.8 /usr/lib/libmcrypt.so.4.4.8 \
  && ln -s /usr/local/lib/libmhash.a /usr/lib/libmhash.a \
  && ln -s /usr/local/lib/libmhash.la /usr/lib/libmhash.la \
  && ln -s /usr/local/lib/libmhash.so /usr/lib/libmhash.so \
  && ln -s /usr/local/lib/libmhash.so.2 /usr/lib/libmhash.so.2 \
  && ln -s /usr/local/lib/libmhash.so.2.0.1 /usr/lib/libmhash.so.2.0.1 \
  && ln -s /usr/local/bin/libmcrypt-config /usr/bin/libmcrypt-config

RUN cd /usr/src/ \
  && tar zxvf mcrypt-2.6.8.tar.gz \
  && cd mcrypt-2.6.8/ \
  && /sbin/ldconfig \
  && ./configure && make && make install

RUN echo "/usr/local/lib" >> /etc/ld.so.conf
RUN /sbin/ldconfig

# install mysql

RUN /usr/sbin/groupadd mysql
RUN /usr/sbin/useradd -g mysql -s /sbin/nologin mysql

RUN cd /usr/src/ \
  && tar zxvf mysql-5.7.*.tar.gz \
  && cd mysql-5.7.*/ \
  && cmake \
     -DCMAKE_INSTALL_PREFIX=/usr/local/webserver/mysql \
     -DMYSQL_DATADIR=/u01/mysql \
     -DEXTRA_CHARSETS=all \
     -DDEFAULT_CHARSET=utf8 \
     -DDEFAULT_COLLATION=utf8_general_ci \
     -DMYSQL_TCP_PORT=3306 \
     -DMYSQL_USER=mysql \
     -DSYSCONFDIR=/etc \
     -DWITH_MYISAM_STORAGE_ENGINE=1 \
     -DWITH_INNOBASE_STORAGE_ENGINE=1 \
     -DWITH_PARTITION_STORAGE_ENGINE=1 \
     -DWITH_ARCHIVE_STORAGE_ENGINE=1 \
     -DWITH_MEMORY_STORAGE_ENGINE=1 \
     -DWITH_BOOST=boost/boost_1_59_0 \
     -DENABLE_DOWNLOADS=1 \
  && make -j `nproc` \
  && make install

RUN cp /usr/local/webserver/mysql/support-files/mysql.server /etc/init.d/mysqld \
  && chmod +x /etc/init.d/mysqld \
  && chkconfig --add mysqld \
  && chkconfig mysqld on

RUN echo "/usr/local/webserver/mysql/lib/" > /etc/ld.so.conf.d/mysql.conf \
  && /sbin/ldconfig

RUN /etc/init.d/mysqld start

EVN dbrootpwd=123456
RUN /usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'127.0.0.1' identified by \"$dbrootpwd\" with grant option;"
RUN /usr/local/webserver/mysql/bin/mysql -e "grant all privileges on *.* to root@'localhost' identified by \"$dbrootpwd\" with grant option;"

RUN /etc/init.d/mysqld stop

# install php

RUN /usr/sbin/groupadd www \
  && /usr/sbin/useradd -g www -s /sbin/nologin www \
  && mkdir -p /u01/www \
  && chmod +w /u01/www \
  && chown -R www:www /u01/www

WORKDIR /usr/src/ 
RUN tar zxvf php-7.*.tar.gz \
  && cd php-7.*/ \
  && ./configure --prefix=/usr/local/webserver/php7 \
     --with-config-file-path=/usr/local/webserver/php7/etc \
     --with-libdir=lib64 \
     --enable-fpm \
     --with-fpm-user=www \
     --with-fpm-group=www \
     --enable-mysqlnd \
     --with-mysqli=mysqlnd \
     --enable-pdo \
     --with-pdo-mysql=mysqlnd \
     --enable-opcache \
     --enable-pcntl \
     --enable-mbstring \
     --enable-soap \
     --enable-zip \
     --enable-calendar \
     --enable-bcmath \
     --enable-exif \
     --enable-ftp \
     --enable-intl \
     --enable-xml \
     --enable-sockets \
     --with-xmlrpc \
     --with-openssl \
     --with-zlib \
     --enable-mbregex \
     --with-curl \
     --with-gd \
     --with-zlib-dir=/usr/lib \
     --with-png-dir=/usr/lib \
     --with-jpeg-dir=/usr/lib \
     --with-freetype-dir=/usr/include/freetype2/ \
     --with-gettext \
     --with-mhash \
     --with-mcrypt \
     --with-ldap \
     --disable-ipv6 \
  && make -j `nproc`
  && make install

RUN cp php.ini-production /usr/local/webserver/php7/etc/php.ini \
  && cp /usr/local/webserver/php7/etc/php-fpm.conf.default /usr/local/webserver/php7/etc/php-fpm.conf
  && cp /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default /usr/local/webserver/php7/etc/php-fpm.d/www.conf

RUN sed -i '/^;pid/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;error_log/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;emergency_restart_threshold/c\emergency_restart_threshold = 60' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;emergency_restart_interval/c\emergency_restart_interval = 60' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;process/c\process_control_timeout = 5s' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;daemonize/s/^;//' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^;rlimit_files/c\rlimit_files = 65535' /usr/local/webserver/php7/etc/php-fpm.conf
RUN sed -i '/^pm =/c\pm = static' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^pm.max_children/s/[0-9]\+$/256/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^pm.start_servers/s/[0-9]\+$/20/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^pm.min_spare_servers/s/[0-9]\+$/5/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^pm.max_spare_servers/s/[0-9]\+$/35/' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^;pm.process_idle_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^;pm.max_requests/c\pm.max_requests = 51200' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default
RUN sed -i '/^;request_terminate_timeout/s/^;//' /usr/local/webserver/php7/etc/php-fpm.d/www.conf.default

RUN cp sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm \
  && chmod +x /etc/init.d/php-fpm \
  && chkconfig --add php-fpm \
  && chkconfig php-fpm on

# install php extensions

WORKDIR /usr/src/
RUN tar zxvf ImageMagick.tar.gz \
  && cd ImageMagick-*/ \
  && ./configure && make && make install \
  && /sbin/ldconfig \

WORKDIR /usr/src/
RUN tar zxvf imagick*.tgz \
  && cd imagick*/ \
  && /sbin/ldconfig \
  && /usr/local/webserver/php7/bin/phpize \
  && ./configure --with-php-config=/usr/local/webserver/php7/bin/php-config \
  && make
  && make install

WORKDIR /usr/src/
RUN wget https://pecl.php.net/get/redis-3.1.0.tgz \
  && tar zxvf redis*.tgz \
  && cd redis*/ \
  && /usr/local/webserver/php7/bin/phpize \
  && ./configure --with-php-config=/usr/local/webserver/php7/bin/php-config \
  && make \
  && make install

===============
未完




vi /usr/local/webserver/php7/etc/php.ini

; extension_dir = "ext"
; 在该行下添加如下配置：
extension_dir = "/usr/local/webserver/php7/lib/php/extensions/no-debug-non-zts-20151012/"
extension=imagick.so
extension=redis.so


; 添加
zend_extension=opcache.so
; 修改
opcache.enable=1
opcache.memory_consumption=256
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=4000
opcache.revalidate_freq=60
opcache.fast_shutdown=1




cd /usr/src/
tar zxvf pcre-*.tar.gz
cd pcre-*/
./configure && make && make install
cd ../



tar zxvf nginx-*.tar.gz
cd nginx-*/
/sbin/ldconfig
./configure \
--prefix=/usr/local/webserver/nginx \
--user=www \
--group=www \
--with-http_v2_module \
--with-http_sub_module \
--with-http_ssl_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_realip_module \
--with-http_flv_module \
--with-http_stub_status_module \
--with-http_addition_module
make && make install
cd ../

cat > /etc/init.d/nginx <<\EOF
#!/bin/sh
#
# nginx - this script starts and stops the nginx daemon
#
# chkconfig:   - 85 15
# description:  NGINX is an HTTP(S) server, HTTP(S) reverse \
#               proxy and IMAP/POP3 proxy server
# processname: nginx
# nginx:       /usr/local/webserver/nginx/sbin/nginx
# config:      /usr/local/webserver/nginx/conf/nginx.conf
# pidfile:     /usr/local/webserver/nginx/logs/nginx.pid

# Source function library.
. /etc/rc.d/init.d/functions

# Source networking configuration.
. /etc/sysconfig/network

# Check that networking is up.
[ "$NETWORKING" = "no" ] && exit 0

nginx="/usr/local/webserver/nginx/sbin/nginx"
prog=$(basename $nginx)

NGINX_CONF_FILE="/usr/local/webserver/nginx/conf/nginx.conf"

lockfile=/var/lock/subsys/nginx

make_dirs() {
   # make required directories
   user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
   if [ -z "`grep $user /etc/passwd`" ]; then
       useradd -M -s /bin/nologin $user
   fi
   options=`$nginx -V 2>&1 | grep 'configure arguments:'`
   for opt in $options; do
       if [ `echo $opt | grep '.*-temp-path'` ]; then
           value=`echo $opt | cut -d "=" -f 2`
           if [ ! -d "$value" ]; then
               # echo "creating" $value
               mkdir -p $value && chown -R $user $value
           fi
       fi
   done
}

start() {
    [ -x $nginx ] || exit 5
    [ -f $NGINX_CONF_FILE ] || exit 6
    make_dirs
    echo -n $"Starting $prog: "
    daemon $nginx -c $NGINX_CONF_FILE
    retval=$?
    echo
    [ $retval -eq 0 ] && touch $lockfile
    return $retval
}

stop() {
    echo -n $"Stopping $prog: "
    killproc $prog -QUIT
    retval=$?
    echo
    [ $retval -eq 0 ] && rm -f $lockfile
    return $retval
}

restart() {
    configtest || return $?
    stop
    sleep 1
    start
}

reload() {
    configtest || return $?
    echo -n $"Reloading $prog: "
    killproc $nginx -HUP
    RETVAL=$?
    echo
}

force_reload() {
    restart
}

configtest() {
  $nginx -t -c $NGINX_CONF_FILE
}

rh_status() {
    status $prog
}

rh_status_q() {
    rh_status >/dev/null 2>&1
}

case "$1" in
    start)
        rh_status_q && exit 0
        $1
        ;;
    stop)
        rh_status_q || exit 0
        $1
        ;;
    restart|configtest)
        $1
        ;;
    reload)
        rh_status_q || exit 7
        $1
        ;;
    force-reload)
        force_reload
        ;;
    status)
        rh_status
        ;;
    condrestart|try-restart)
        rh_status_q || exit 0
            ;;
    *)
        echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
        exit 2
esac
EOF

chmod +x /etc/init.d/nginx
chkconfig --add nginx
chkconfig nginx on


mkdir -p /u01/logfiles/nginx
chmod +w /u01/logfiles/nginx
chown -R www:www /u01/logfiles/nginx

cp /usr/local/webserver/nginx/conf/nginx.conf{,.original}
cat > /usr/local/webserver/nginx/conf/nginx.conf <<\EOF
user  www www;

#worker_processes 8;
#worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
worker_rlimit_nofile 65535;

error_log  /u01/logfiles/nginx/nginx_error.log  warn;

events
{
  use epoll;
  worker_connections 65535;
}

http
{
  include       mime.types;
  default_type  application/octet-stream;

  server_names_hash_bucket_size 128;
  client_header_buffer_size 32k;
  large_client_header_buffers 4 32k;
  client_max_body_size 8m;

#  limit_conn_zone $binary_remote_addr zone=connlimit:9m;

  server_tokens off;
  sendfile on;
  tcp_nopush on;

  keepalive_timeout 10;

  tcp_nodelay on;

  fastcgi_connect_timeout 300;
  fastcgi_send_timeout 300;
  fastcgi_read_timeout 300;
  fastcgi_buffer_size 64k;
  fastcgi_buffers 8 64k;
  fastcgi_busy_buffers_size 128k;
  fastcgi_temp_file_write_size 128k;

  gzip on;
  gzip_min_length  1k;
  gzip_buffers     4 16k;
  gzip_http_version 1.0;
  gzip_comp_level 2;
  gzip_types       text/plain application/x-javascript text/css application/xml;
  gzip_vary on;

  log_format  access  '$remote_addr - $remote_user [$time_local] "$request" '
              '$status $body_bytes_sent "$http_referer" '
              '"$http_user_agent" $http_x_forwarded_for';

  server
  {
    listen       80;
    index index.php index.html;
    root  /u01/www/;

    #limit_conn  connlimit 15;
    #limit_rate  256k;

    location ~ .*\.php$
    {
      fastcgi_pass 127.0.0.1:9000;
      fastcgi_index index.php;
      include fastcgi.conf;
    }

    location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
    {
      expires      30d;
      access_log   off;
    }
    location ~ .*\.(js|css)$
    {
      expires      1h;
      access_log   off;
    }

    access_log  /u01/logfiles/nginx/access.log  access;
  }

}
EOF


cat > /usr/local/webserver/nginx/cut_nginx_log.sh <<\EOF
#!/bin/bash
# cut_nginx_log.sh
# This script run at 00:00

# The Nginx logs path
log_path="/u01/logfiles/nginx"
new_log_path=${log_path}/$(date -d 'yesterday' '+%Y%m%d')

mkdir -p ${new_log_path}
mv ${log_path}/*.log ${new_log_path}

kill -USR1 `cat /usr/local/webserver/nginx/logs/nginx.pid`
EOF


crontab -e
0 0 * * * /bin/bash /usr/local/webserver/nginx/cut_nginx_log.sh

