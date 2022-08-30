---
title: Docker实现前后端自动化部署
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