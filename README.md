# php-docker-env
帮助初次接触 docker 的 php 开发者，快速构建 docker 化的 php 开发环境。

### 包涵的组件
* mysql : 5.7.18
* php : 7.1.7
* xdebug : 2.5.5
* apache : 2.4.10

### 关于镜像源
mysql 与 php-apache 均使用[网易蜂巢](https://c.163.com/hub)从 [DockerHub](https://hub.docker.com/explore/)  拉取的官方镜像
* 浏览[网易蜂巢](https://c.163.com/hub)可能需要登录

# 快速开始
请确保已经安装 docker
* [Mac 安装 docker](https://www.docker.com/docker-mac)
* [Windows 安装 docker](https://www.docker.com/docker-windows)
* [其它用户](https://www.docker.com/)
* [docker 快速入门](https://docs.docker.com/get-started/)

### 拉取镜像
执行以下命令，获取[mysql](https://hub.docker.com/_/mysql/)镜像
```sh
docker pull hub.c.163.com/library/mysql:5.7.18
```
执行以下命令，获取[php-apache](https://hub.docker.com/_/php/)镜像
```sh
docker pull hub.c.163.com/library/php:7.1.7-apache
```

### 构建自己的镜像
以上，我们已经获取到了两个基础镜像，使用 docker run 命令就能使用了。但开发环境中，我们还需为PHP安装xdebug插件以方便调试，以及配置 php 和 apache。所以我们要在 php-apache 这个镜像的基础上自定义一个镜像。./docker/Dockerfile 中已经定义好 docker 该如何构建这个镜像。我们只需：
```sh
docker build -t webserver ./docker/
```
就能构建一个名为 webserver 的镜像。（构建镜像可能需要一些时间）

### 创建容器（运行镜像）
#### 启动 mysql
第一次启动 mysql 需执行以下命令：
```sh
docker run --name mydb -v "$PWD"/mysql:/var/lib/mysql -p 3306:3306 -d -e MYSQL_ROOT_PASSWORD=root hub.c.163.com/library/mysql:5.7.18
```
命令详解：
* 这条命令根据 hub.c.163.com/library/mysql:5.7.18 镜像，创建并运行了一个名为 mydb 的容器。
* 将此目录下的 mysql 目录挂载到 ./mydb 容器的 /var/lib/mysql 目录，mydb 运行时所产生的数据将得到保存。
* 将主机的 3306 端口映射到 mydb 容器中的 3306 端口。
* 将 mysql 的 root 用户密码设置为root。以后在 ./mysql/ 目录运行 mysql 镜像 将不用带 -e MYSQL_ROOT_PASSWORD=root 参数。
可直接执行：
```sh
docker run --name mydb -v "$PWD"/mysql:/var/lib/mysql -p 3306:3306 -d -e MYSQL_ROOT_PASSWORD=root hub.c.163.com/library/mysql:5.7.18
```

#### 启动 php-apache
也就是我们构建的 webserver 镜像：
```sh
docker run --name myserver -v "$PWD"/src:/var/www/html -p 8080:80  -e XDEBUG_CONFIG="remote_host=192.168.1.102" -d webserver
```
命令详解：
* 这条命令根据我们自定义的 webserver 镜像，创建并运行了一个名为 myserver  的容器。
* 将此目录下的 ./src 目录挂载到 myserver 容器的 /var/www/html 目录，你的 php 项目代码放入 ./src 目录即可。
* 将主机的 8080 端口映射到 myserver 容器的 80 端口。
* -e XDEBUG_CONFIG="remote_host=192.168.1.102" 此参数中的 ip 地址请更改为你主机的 ip 地址。它设置了 xdebug 插件的 remote_host。

至此，我们已经成功启动了我们的 php 开发环境

## 进行项目开发
将你的 php 项目代码放入 ./src/ 目录，编辑代码，将可得到 myserver 容器的实时响应。
这里我们以 [ThinkPHP5框架](https://github.com/top-think/think) 为例子：
浏览器输入 http://localhost:8080/pulbic/ 即可访问到[ThinkPHP5框架](https://github.com/top-think/think)的欢迎页面。

## 配置文件
#### 文件位置
* apache 主配置文件位于主机 ./docker/conf/apache2.conf 对应于容器的 /etc/apache2/apache2.conf
* xdebug 配置文件位于主机 ./docker/conf/docker-php-ext-xdebug.ini 对应于容器的 /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
* php 主配置文件位于 ./docker/conf/php.ini 对应于容器的 /usr/local/etc/php/php.ini
* 虚拟域名配置文件统一放置于 ./docker/vhost/ 对应于容器的 /etc/apache2/sites-enabled/

#### 配置生效
由于配置文件在docker容器创建时就被打包到容器中，所以无法动态更改。
当更改配置后，需要停止并删除当前运行的容器，重新创建容器即可：
```sh
docker stop myserver 
docker rm myserver
docker build -t webserver ./docker/
docker run --name myserver -v "$PWD"/src:/var/www/html -p 8080:80  -e XDEBUG_CONFIG="remote_host=192.168.1.102" -d webserver
```


## 与容器交互
```sh
docker exec [容器名] [容器执行的命令]
```
* 以上命令可在主机中，让容器执行一条特定的命令。
* 加入 -i 标志可保持主机对容器对输入。
* 加入 -t 标志可让容器打开一个伪终端。
* 更多用法请执行命令  docker exec --help

所以我们可以这样与 mysql 交互：
```sh
docker exec -it mydb mysql -h 127.0.0.1 -u root -p root
```
此命令将打开一个终端，并与 mysql 连接。（假设 mysql root 用户对密码为 root）
```sh
exit
```
可退出终端。

### 进入容器执行更多命令
```sh
docker exec -it myserver /bin/bash
```
以上命令可进入 myserver 容器，并可在容器内执行更多命令，如 ls, cd, ps ...

更多请参阅 [docker 快速入门](https://docs.docker.com/get-started/)

互相学习交流可加 QQ：568499182

学习愉快！