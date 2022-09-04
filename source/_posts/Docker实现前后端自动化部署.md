---
title: Docker实现前后端自动化部署
date: 2022-08-30
---

用Docker实现Github上提交代码后自动化部署，让购买的`百度云`服务器派上用场~
<!-- more -->

## Docker部署

### 1、安装Docker

### 2、项目根目录创建`nginx.conf`文件

容器监听80端口，代理到/app文件夹中index.html文件

```
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid 
       /var/run/nginx.pid;
events {
  worker_connections  1024;
}

http {
  include       /etc/nginx/mime.types;
  default_type  application/octet-stream;
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
  access_log  /var/log/nginx/access.log  main;
  sendfile        on;
  keepalive_timeout  65;

  server {
    listen       80;
    server_name  localhost;

    location / {
      root   /app;
      index  index.html;
      try_files $uri $uri/ /index.html;
    }

    error_page   500 502 503 504  /50x.html;
    
    location = /50x.html {
      root   /usr/share/nginx/html;
    }
    
  }
}
```

### 3、在项目根目录创建`Dockerfile`文件

```
FROM node:14

WORKDIR /app
COPY ./ /app
RUN npm install && npm run build

FROM nginx

RUN mkdir /app
# --from=0获取第一阶段构建完成的目录
COPY --from=0 /app/dist /app
# COPY /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
```

### 4、打包docker镜像并上传

上传到docker Hub之前需要登录docker

```
docker build -t docker仓库/镜像名称:latest .
docker push docker仓库/镜像名称:latest
```

### 5、云服务器拉取镜像并生成容器运行

```
# deploy.sh

docker pull docker仓库/镜像名称:latest
docker stop fe-container
docker rm fe-container
docker run -itd -p 8081:80 --name fe-container docker仓库/镜像名称:latest
# 删除所有未被 tag 标记和未被容器使用的镜像
docker image prune -f
# 清理所有已经停止的容器
docker container prune -f
```

## Docker部署后端服务

### 安装Mysql

仓库拉取mysql8.0

```
docker pull mysql:8.0
```

安装并运行mysql容器

```
docker run -p 3307:3306 --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
备注：
-p 将本地主机的端口映射到docker容器端口（因为本机的3306端口已被其它版本占用，所以使用3307）
--name 容器名称命名
-e 配置信息，配置root密码
-d 镜像名称
```

让mysql支持远程连接

```
// 进入mysql后
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
flush privileges; //刷新权限
```

若云服务器存在防火墙，需要将3306等端口开放才能连接

![image-20220904164227943](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20220904164227943.png)

之后需要被node服务连接数据库，可以先创建测试的数据库。

### 启动后端服务

后端服务和前端类似，Dockerfile有些改动

测试使用的是midway框架，npm run build && npm start

```
FROM node
WORKDIR /app
COPY package*.json /app/
RUN npm install
COPY . .

EXPOSE 7001
CMD npm run build && npm start
```

用软链的方式，将node容器和mysql容器进行互联。

连接的是mysql容器，将mysql配置修改

此时已经互联，所以port为mysql内部端口3306，host为mysql容器

```
// 添加orm配置
  orm: {
    type: 'mysql',
    host: 'mysql',
    // host: '127.0.0.1', // 改成你的mysql数据库IP
    port: 3306, // 改成你的mysql数据库端口
    ...
  },
```

启动node容器并将mysql容器软链

```
docker run -dti -p 7007:7001 --name node-container --link mysql:mysql 1440054388/midway-image
// mysql:mysql 前端为容器名，后面为别名
```

**单独连接fe容器、node容器、mysql容器比较麻烦，不过docker已有相关的编排工具docker-compose**

### docker-compose.yml配置

以上，存在3个容器，他们的依赖关系是：前端容器=》后端容器=》数据库容器，需要按启动顺序启动。

配置3项，用depends_on确定依赖关系，并且将它们都添加到同一个network中，才能互联

```
version: '3'

networks:
  app-web:
   driver: bridge

services:
  mysql:
    container_name: mysql
    image: mysql:8.0
    ports:
     - 3306:3306
    restart: always
    networks:
     - app-web
    environment:
     # 等同于 -e MYSQL_ROOT_PASSWORD指定root的登录密码
     MYSQL_ROOT_PASSWORD: '123456'
     MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
     # 这里这个指令compose启动成功后会自动创建名为node的数据库
     MYSQL_DATABASE: 'node'
     # 此处就是相当于 mysql create user，创建了数据库的登录用户。
     # mysql8.0不用添加以下root，因为默认就已经添加了
    #  MYSQL_USER: 'root'
    #  MYSQL_PASSWORD: '123456' 
    # volumes:
      # - /root/docker/compose/mysql/data:/var/lib/mysql
      # 这里的my.cnf可以从原来的安装的MySQL里面找，如果没有不配置也不影响，只是为了方便外部更改
      # - /root/docker/compose/mysql/conf/my.cnf:/etc/my.cnf
      # - /root/docker/compose/mysql/init:/docker-entrypoint-initdb.d
    # 解决外部无法访问
    command: --default-authentication-plugin=mysql_native_password
  backend:
    container_name: node-container
    image: 1440054388/midway-image
    ports:
     - 7001:7001
    depends_on:
     - mysql
    networks:
     - app-web
  frontend:
    container_name: fe-container
    image: 1440054388/fe-image
    ports:
     - 8081:80
    depends_on:
     - backend
```

并且互联后，可以把host改成127.0.0.1

```
 orm: {
    type: 'mysql',
    // host: 'mysql',
    host: '127.0.0.1', // 改成你的mysql数据库IP
```

启动服务，-d表示以守护进程的方式运行

```
docker compose up -d
```

启动后有一些异常关闭的话，可以查看日志进行排查

```
docker logs 容器名
```

## Github Action自动化部署

**通过Github Action，监听分支代码变化，触发自定义的任务。**

**自动打包代码，镜像上传并登录云服务器执行部署脚本。**

### Github Action

当我们想往自己的项目里接入**Github Actions**时，要在根项目目录里新建`.github/workflows`目录。然后通过编写`yml`格式文件定义**Workflow(工作流程)去实现`CI`。`yml`文件Workflow**中一些比较重要的概念：

- **Event(触发事件)**：指触发 **Workflow(工作流程)** 运行的事件。
- **Job(作业)**：一个**工作流程**中包含一个或多个**Job**，这些**Job**默认情况下并行运行，但我们也可以通过设置让其按顺序执行。每个**Job**都在指定的环境(虚拟机或容器)里开启一个**Runner**(可以理解为一个进程)运行，包含多个**Step(步骤)**。
- **Step(步骤)**：**Job**的组成部分，用于定义每一部的工作内容。每个**Step**在运行器环境中以其单独的进程运行，且可以访问工作区和文件系统。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27deef91333747ab8f7e09bd2649b5bb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

在项目根目录创建`.github/workflows/deploy.yml`目录及文件

```
name: CI/CD
# 触发条件为 push
on:
  push:
    branches:
      - master
    # 修改文件范围, 不在范围内不触发
    paths:
      - '.github/workflows/**'
      - '__test__/**'
      - 'src/**'
      - '**/**'
# 任务
jobs:
  test:
    # 运行的环境
    runs-on: ubuntu-latest
    # 步骤
    steps:
      - uses: actions/checkout@v2 # git pull
      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: 14
      - name: 打包镜像, 上传 Docker Hub
        run: |
          docker login -u ${{ secrets.REGISTRY_USERNAME }} -p ${{ secrets.REGISTRY_PASSWORD }}
          docker build -t ${{ secrets.DOCKER_REPOSITORY }} .
          docker push ${{ secrets.DOCKER_REPOSITORY }}

      - name: 登录服务器, 执行脚本
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.REMOTE_HOST }}
          username: root
          password: ${{ secrets.REMOTE_PASSWORD }}
          # 执行脚本
          script: cd ~ && sh deploy.sh
          
```

deploy.yml文件中的敏感信息用变量表示，在setting中可以配置变量

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddaf12fad12b43be94ea330648f2d349~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp?)

修改代码并提交分支，即可触发deploy.yml流程，大功告成！