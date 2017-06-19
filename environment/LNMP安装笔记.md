# LNMP安装教程

## Nginx

	./configure --prefix=/home/work/web/nginx \
	--conf-path=conf/nginx.conf \
	--sbin-path=sbin/nginx \
	--pid-path=var/nginx.pid \
	--error-log-path=logs/error.log \
	--http-log-path=logs/access.log \
	--with-http_ssl_module \
	--with-pcre=/usr/local/src/pcre \
	--with-zlib=/usr/local/src/zlib \
	--with-openssl=/usr/local/src/openssl

## Openresty

	./configure --prefix=/home/work/web/openresty \
	--conf-path=conf/nginx.conf \
	--sbin-path=sbin/nginx \
	--pid-path=var/nginx.pid \
	--error-log-path=logs/error.log \
	--http-log-path=logs/access.log \
	--with-http_ssl_module \
	--with-pcre=/usr/local/src/pcre \
	--with-zlib=/usr/local/src/zlib \
	--with-openssl=/usr/local/src/openssl-1.0.2j \
	--with-luajit \
	--without-http_redis2_module \
	--with-http_iconv_module

## PHP

	./configure --prefix=/home/work/web/php \
	--enable-fpm \
	--with-mcrypt \
	--enable-mbstring \
	--disable-pdo \
	--with-curl \
	--disable-debug \
	--disable-rpath \
	--enable-inline-optimization \
	--with-bz2  \
	--with-zlib \
	--enable-sockets \
	--enable-sysvsem \
	--enable-sysvshm \
	--enable-pcntl \
	--enable-mbregex \
	--with-mhash \
	--enable-zip \
	--with-pcre-regex \
	--with-mysqli \
	--with-pdo-mysql \
	--with-gd \
	--with-jpeg-dir 

## 依赖

	yum -y install libmcrypt-devel mhash-devel libxslt-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5 krb5-devel libidn libidn-devel openssl openssl-devel
