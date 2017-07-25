FROM hub.c.163.com/library/php:7.1.7-apache

# 安装xdebug
RUN pecl install xdebug-2.5.5 \\
    && docker-php-ext-enable xdebug
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
