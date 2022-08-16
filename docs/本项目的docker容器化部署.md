# 本项目的docker容器化部署

## 1. 安装配置



docker安装https://www.runoob.com/docker/ubuntu-docker-install.html

docker-compose安装https://developer.aliyun.com/article/763018 (本项目未使用，可以不安装)

dockerhub官网注册一个账号，用于进行push/pull镜像操作https://hub.docker.com/ 



在终端登录docker：

1.  将本机用户添加至docker用户组: `sudo gpasswd -a 本机用户名 docker`
2. 更新用户组: `newgrp docker`
3. 登录: `docker login`



## 2. 创建镜像



创建镜像之前先确保在本机已经把完整的项目文件clone下来了

1. 进入项目根目录，执行 `mvn package` 命令
2. 进入DockerFile所在目录。本项目是GreenFarm/csu-farm-api
3. 执行 `docker build -t csu-farm-api .`

​	docker build 参数：

​		-t 指定镜像的tag（名字、版本）

​		最后面的 ‘.’ 代表dockerfile所在目录



## 3. 镜像使用



docker的使用很简单，仅需要执行一个命令即可

`docker run -d -p 8086:8086 csu-farm-api:latest`

可以使用 `docker ps` 命令查看所有正在运行的容器

*注意*：

1. 使用命令 `docker image ls` 可以查看所有镜像。如果镜像的tag与上例不一样，那么就把上面的改正所查看到的即可
2. -d 后台运行镜像
3. -p 端口映射 : 映射主机（宿主）端口和容器端口

上一步创建的镜像相当于一个超微型的开销极低的linux系统，在docker容器运行此镜像就把这个系统跑起来了，使用-p映射主机（宿主）端口和容器端口。上一步中，在DockerFile中有`EXPOSE 8086`的语句，效果是把这个微型的系统的8086端口暴露，使得 `build` 命令中的 `-p`参数可以将宿主的8086端口与容器的8086端口映射起来。



****

**以上几个步骤已足以将csu-farm-api后端以容器的方式运行起来。接下来介绍的是镜像的管理**



## 4. 镜像管理



1. 以下两个命令任其一可以查看本机主机上所有镜像：

   ```
   docker images
   docker image ls
   ```

   其中各选项的说明：

   - **REPOSITORY：**表示镜像的仓库源
   - **TAG：**镜像的标签
   - **IMAGE ID：**镜像ID
   - **CREATED：**镜像创建时间
   - **SIZE：**镜像大小

2. 获取一个新的镜像

   当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 `docker pull` 命令来下载它。

   比如本项目我们使用

   `docker pull aier233/csu-farm-api  `

   命令来下载镜像 

3. 删除一个镜像

   `docker rmi 镜像tag或id`

   若提示镜像被某个容器使用，那么使用

   `docker rm 容器id`来删除相关容器

4. 将镜像推送至dockerhub

   推送镜像是有命名规范的，必须以 `注册用户名/镜像名` 的形式推送

   所以在push前，先用tag命令修改为规范的镜像：

   ```
    docker tag csu-farm-api:latest aier233/csu-farm-api:latest
   ```

   接着执行命令将其push至dockerhub：

   ~~~
   docker push aier233/csu-farm-api:latest
   ~~~

   网速很慢的情况下大文件不好上传，这是正常现象，静静等待就行。

   服务器带宽也好像不高，所以需要一段时间



