FROM hub.c.163.com/library/php:7.1.7-apache

RUN apt-get update && apt-get install -y libfreetype6-dev  libjpeg62-turbo-dev  libpng12-dev libmcrypt-dev && docker-php-ext-install -j$(nproc) iconv mcrypt && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ && docker-php-ext-install -j$(nproc) gd

# 安装xdebug
RUN pecl install install xdebug-2.5.5 && docker-php-ext-enable xdebug
RUN pecl install install redis-3.1.5 && docker-php-ext-enable redis
RUN docker-php-ext-install pdo_mysql

# 添加php配置文件
# php默认配置文件
# ADD conf/php.ini /usr/local/etc/php/
# xdebug配置文件
ADD conf/docker-php-ext-xdebug.ini /usr/local/etc/php/conf.d/


## 添加apache配置文件
# apache默认配置文件
ADD conf/apache2.conf  /etc/apache2/
# 虚拟域名配置文件
ADD vhost/*.conf /etc/apache2/sites-enabled/
