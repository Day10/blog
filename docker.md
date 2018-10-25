### why Docker?
1. 环境配置麻烦，诞生了软件带环境安装的需求
2. 虚拟机虽然使带环境安装的方法，但是占用资源多，启动慢，冗余步骤多。
3. Linux容器（LXC）是另一种带环境安装的方法。并且没有虚拟机的缺点。
4. docker属于 Linux 容器的一种封装，提供简单易用的容器使用接口

### 用途
1. 提供一次性环境（测试环境）
2. 提供弹性的云服务（易扩容缩容）
3. 组建微服务架构

### 基本概念
1. image 文件
    * Docker 把应用程序及其依赖，打包在 image 文件里面
    * image能生成容器实例，可以当作容器的模板
    * 一个image可以继承另一个image
    * 开发中，一般一个image文件 = 继承的image文件 + 个性化设置。有点像子类与父类。 
    * Docker Hub 是image仓库

| 命令 | 作用 |
|---|---|
|docker image pull library/image-name|从仓库抓取image文件|
|docker image pull hello-world |同上，官方的默认组是library所以可以省略|
|docker image ls | 查看image文件 |
|docker container run image-name |（如果没有image-name文件，则自动抓取）从image文件生成一个容器实例|
|docker container kill [containID] / Ctrl + c | 终止不会自动终止的容器 |
2. 容器文件
    * 容器文件是image文件生成的容器实例
    * 关闭容器不会删除容器文件，只是容器停止运行

| 命令 | 作用 |
|---|---|
| docker container ls | 列出所有运行的容器 |
| docker container ls --all | 列出所有容器 |
| docker container rm [containerID] | 删除容器文件 |


### 制作自己的Docker容器
1.  .dockerignore文件内容表示不要打包进入image文件的路径
`
  .git
  node_modules
  npm-debug.log
`
2. Dockerfile文件
    * FROM node:8.4： 该image继承官方8.4版本的node image 
    * COPY . /app ：将当前目录下的所有文件（除.dockerignore排除的路径）拷贝到image文件的/app目录
    * WORKDIR /app ：指定工作路径是/app
    * RUN npm install --registry=https://registry.npm.taobao.org： 在/app目录下，运行npm install命令安装依赖
    * EXPOSE 3000：将容器 3000 端口暴露出来， 允许外部连接这个端口。
    * CMD node demos/01.js：容器启动后自动执行node demos/01.js
3. 创建image文件
    * docker image build （创建image文件） -t image-name:0.0.1（-t指定image文件名和版本）  . （表示Dockerfile文件所在路径，因为在当前路径，所以是一个点·）
4. 生成容器

| 命令/ 参数 | 作用 |
|---|---|
|docker container run -p 8000:3000 -it koa-demo:0.0.1 /bin/bash| 从image生成容器 |
|-p 8000:3000| 容器的3000端口映射到本机的8000端口|
|-it| 容器的 Shell 映射到当前的 Shell，然后你在本机窗口输入的命令，就会传入容器。|
|koa-demo:0.0.1|image 文件名，如果没标签，默认latest|
|/bin/bash|容器启动以后，内部第一个执行的命令。这里是启动 Bash，保证用户可以使用 Shell。|
|--rm|容器终止后自动删除容器文件|

5. 发布image

| 命令 | 作用 |
|---|---|
|docker login|登录|
|docker image tag [imageName] [username]/[repository]:[tag]| 为本地的 image 标注用户名和版本.例子:docker image tag koa-demos:0.0.1 ruanyf/koa-demos:0.0.1|
|docker image push [username]/[repository]:[tag]|发布|

6. 其他命令

| 命令 | 作用 |
|---|---|
|docker container start [containerID]| 启动已经生成、已经停止运行的容器文件|
|bash container stop [containerID]|终止容器，但自行进行收尾清理工作，但也可以不理会这个信号|
|docker container logs [containerID]| 查看 docker 容器的输出|
|docker container exec -it [containerID] /bin/bash| 用于进入一个正在运行的 docker 容器|
|docker container cp [containID]:[/path/to/file] .|从正在运行的 Docker 容器里面，将文件拷贝到本机|

### Docker 微服务
微服务的思想: 软件把任务外包出去，让各种外部服务完成这些任务，软件本身只是底层服务的调度中心和组装层。
#### 用docker-compose
1. Compose是 Docker 公司推出的一个工具软件，可以管理多个 Docker 容器组成一个应用。你需要定义一个 YAML 格式的配置文件`docker-compose.yml`，写好多个容器之间的调用关系。然后，只要一个命令，就能同时启动/关闭这些容器。
2. 启动所有服务：`docker-compose up`；关闭所有服务：`docker-compose stop`
3. wordPress的docker-compose.yml示例
mysql:

    image: mysql:5.7
    
    environment:
    
     \- MYSQL_ROOT_PASSWORD=123456 // 向容器进程传入一个环境变量MYSQL_ROOT_PASSWORD，该变量会被用作 MySQL 的根密码。
     
     \- MYSQL_DATABASE=wordpress  // 向容器进程传入一个环境变量MYSQL_DATABASE，容器里面的 MySQL 会根据该变量创建一个同名数据库（本例是WordPress）。
web:

    image: wordpress  // 基于官方镜像wordpress
    
    links:
    
    \- mysql
    
    environment:
    
     \- WORDPRESS_DB_PASSWORD=123456
     
    ports:
    
     \- "127.0.0.3:8080:80"  // 将容器的 80 端口映射到127.0.0.2的8080端口。
     
    working_dir: /var/www/html
    
    volumes:
    
     \- wordpress:/var/www/html  //  将容器的/var/www/html（Apache 对外访问的默认目录目录映射到当前目录的wordpress子目录。

    [资料来源](http://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html)

