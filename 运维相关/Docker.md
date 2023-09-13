# Docker

## 基本概念

### 镜像

我们都知道，操作系统分为 **内核** 和 **用户空间**。对于 `Linux` 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 **Docker 镜像**（`Image`），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。

**Docker 镜像** 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 **不包含** 任何动态数据，其内容在构建之后也不会被改变。



#### 虚悬镜像

镜像是有镜像名和标签的，原来为 `mongo:3.2`，随着官方镜像维护，发布了新版本后，重新 `docker pull mongo:3.2` 时，`mongo:3.2` 这个镜像名被转移到了新下载的镜像身上，而旧的镜像上的这个名称则被取消，从而成为了 `<none>`。除了 `docker pull` 可能导致这种情况，`docker build` 也同样可以导致这种现象。由于新旧镜像同名，旧镜像名称被取消，从而出现仓库名、标签均为 `<none>` 的镜像。这类无标签镜像也被称为 **虚悬镜像(dangling image)** 



#### 中间层镜像

用于加速构建，重复利用资源所产生的镜像，也是无标签镜像，但不像虚悬镜像一样可以随意删除



### 容器

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的命名空间。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。



#### 分层存储

因为镜像包含操作系统完整的 `root` 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 Union FS 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 `ISO` 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。



### 仓库

Docker Registry 是一个集中的存储、分发镜像的服务。

一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

以 Ubuntu 镜像 为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，`16.04`, `18.04`。我们可以通过 `ubuntu:16.04`，或者 `ubuntu:18.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

仓库名经常以 *两段式路径* 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。



国内镜像地址：

```shell
registry.docker-cn.com # Docker 中国官方镜像
f1361db2.m.daocloud.io # DaoCloud 镜像站
dockerhub.azk8s.cn # Azure 中国镜像
docker.mirrors.ustc.edu.cn # 科大镜像站
https://<your_code>.mirror.aliyuncs.com # 阿里云
reg-mirror.qiniu.com # 七牛云
hub-mirror.c.163.com # 网易云
mirror.ccs.tencentyun.com # 腾讯云
```



win10配置镜像：

- 打开Docker Desktop软件，进入Settings-Docker Engine

- 编辑json文件，添加registry-mirrors项：

  ```json
  {
    ...
    "registry-mirrors": [
      "https://registry.docker-cn.com"
    ]
  }
  ```

  

## 镜像

可以在[docker hub](https://hub.docker.com/search?q=&type=image)上搜索镜像

### 下载镜像

```shell
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

具体的选项可以通过 `docker pull --help` 命令查看



### 列出镜像

```shell
docker image ls [仓库名[:标签]] [选项]
```

列出本地已有的镜像，可以通过指定仓库名和标签名进行筛选

选项：

- `-q`：在结果中列出镜像的id，通常用于送给另一个命令作为参数
- `-f 过滤表达式`：过滤器，过滤表达式有：
  -  `since=mongo:3.2`：筛选出mongo:3.2之后建立的镜像
  - `before=mongo:3.2`：筛选出mongo:3.2之前建立的镜像
  - `label=xxx`：根据镜像构建时定义的LABEL进行筛选
  - `dangling=true`：筛选虚悬镜像
- `--format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"`：根据go的模板语法自定义结果
- `--digests`：结果中显示镜像摘要



### 删除镜像

```shell
docker image rm [选项] <镜像1> [<镜像2> ...]
```

- `<镜像>`：可以是 `镜像短 ID`（即完整id的前n位字符，一般取3位以上）、`镜像长 ID`、`镜像名` 或者 `镜像摘要`



配合 image ls 命令成批删除：

```shell
docker image rm $(docker image ls -q -f before=mongo:3.2)
```



删除所有虚悬镜像：

```shell
docker image prune
```



### 定制镜像

#### Dockerfile介绍

定制镜像需要创建Dockerfile文件，内容如下例：

```shell
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

文件中包含了一条条的指令（Instruction），每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。



构建镜像：

```shell
docker build [选项] 构建上下文路径
```

- 选项：
  - `-t <仓库名>[:<标签>]`：指定镜像名称
- 构建上下文路径：一般指定为 `.`，表示当前目录。在COPY等指令中，可能需要将本地文件复制进镜像中，而在这些指令中，源文件的路径必须是相对路径，这些相对路径就是相对构建上下文路径
  - docker引擎无法获得上下文路径外的文件



#### Dockerfile指令

- FROM 指定基础镜像：

  ```shell
  FROM <镜像>
  ```

  - 指定基础镜像，在该镜像上构建，因此该指令必须是第一条指令

- RUN 执行命令：

  ```shell
  RUN <命令>
  # 或者
  RUN ["可执行文件", "参数1", "参数2"]
  ```

  - RUN命令会像shell脚本一样执行命令

  - 执行多条命令时，不应该使用多个RUN指令，而是用 `&&` 将命令串联。防止多次执行RUN指令而创建多余的层

    ```shell
    RUN 命令1
    RUN 命令2
    
    # 改写为
    RUN 命令1 \
        && 命令2
    ```





## 容器

### 列出容器

```
docker container ls -a
```



查看容器具体信息：

```
docker inspect <容器ID或容器名>
```



### 启动/停止容器

```shell
docker run [选项] <镜像> 命令
```

- 选项：
  - `-it`：这是两个参数，`-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， `-i` 则让容器的标准输入保持打开。
  - `--rm`：容器退出后随之将其删除。
  - `-d`：后台运行，不将结果输出到当前控制台
  - `--mount type=bind,source=/src/webapp,target=/usr/share/nginx/html,readonly`：挂载主机目录
  - `--name 容器名`：指定容器名
  - `-p 80:8080`：映射本机的80端口到容器的8080端口
- 命令：启动后执行的命令，可以使用`bash`来打开控制台



启动已终止的容器：

```shell
docker container start <容器ID或容器名>
```



终止容器：

```shell
docker container stop <容器ID或容器名>
```



### 修改容器

进入容器：

```shell
docker attach <容器ID或容器名>
# 或
docker exec -it <容器ID或容器名> bash
```

- 进入容器后，可以执行命令，修改容器内容



查看容器的改动：

```shell
docker diff <容器ID或容器名>
```



保存改动：

> 一般情况下，不应该通过commit将改动重新生成镜像，这种镜像可维护性差。若要定制镜像，需要使用Dockerfile完成

```shell
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

- 选项：
  - `--author "xx"`：指定作者
  - `--message "xx"`：指定修改的内容
- 指定仓库名与标签名，则会生成新镜像



### 导入/导出容器

```shell
# 导出容器快照至本地文件
docker export <容器ID或容器名> > xxx.tar
# 将容器快照导入为镜像
cat xxx.tar | docker import - <新的镜像>
# 也可以通过url或某个目录导入
docker import http://example.com/exampleimage.tgz example/imagerepo
```



### 删除容器

```shell
docker container rm <容器ID或容器名>
```



清除所有处于终止状态的容器：

```shell
docker container prune
```



## 数据卷

> `数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UnionFS，可以提供很多有用的特性：
>
> - `数据卷` 可以在容器之间共享和重用
> - 对 `数据卷` 的修改会立马生效
> - 对 `数据卷` 的更新，不会影响镜像
> - `数据卷` 默认会一直存在，即使容器被删除
>
> `数据卷` 的使用，类似于 Linux 下对目录或文件进行 mount，镜像中的被指定为挂载点的目录中的文件会复制到数据卷中（仅数据卷为空时会复制）

创建数据卷：

```shell
docker volume create <数据卷名>
```



查看所有数据卷：

```shell
docker volume ls
```



查看数据卷详细信息：

```shell
docker volume inspect <数据卷名>
```

- windows系统的下数据卷存储于：`\\wsl$\docker-desktop-data\data\docker\volumes`



删除数据卷：

```shell
docker volume rm <数据卷名>
```



清除无主数据卷：

```shell
docker volume prune
```



## 网络

docker默认情况下使用桥接模式网络，容器之间访问网络可以使用 `容器名:端口号` 作为ip端口来访问



## Docker Compose

> 是docker官方的开源项目，用于对docker容器集群进行快速编排
>
> 一个项目可以由多个服务（容器）关联而成，Compose 面向项目进行管理
>
> - 服务 (`service`)：一个应用容器，实际上可以运行多个相同镜像的实例。
> - 项目 (`project`)：由一组关联的应用容器组成的一个完整业务单元。

### 基本使用

安装：

- windows下，安装docker desktop时会一并安装docker-compose
- [其他平台的安装](https://yeasy.gitbook.io/docker_practice/compose/install)



完整说明：https://yeasy.gitbook.io/docker_practice/compose/commands

```shell
docker-compose [选项] [命令] [参数...]
```

- 选项：
  - `-f, --file 文件`：指定使用的 Compose 模板文件，默认为 `docker-compose.yml`，可以多次指定。
  - `-p, --project-name 名称`：指定项目名称，默认将使用所在目录名称作为项目名。
  - `--verbose`：输出更多调试信息。
  - `-v, --version`：打印版本并退出。
- 命令：
  - `build`：构建（重新构建）项目中的服务容器。
    - 服务容器一旦构建后，将会带上一个标记名，例如对于 web 项目中的一个 db 容器，可能是 web_db。
    - 参数：需要重新构建的服务容器
  - `up`：尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联服务相关容器的一系列操作
    - 选项：
      - `-d` 在后台运行服务容器。
      - `--force-recreate` 强制重新创建容器，不能与 `--no-recreate` 同时使用。
      - `--no-recreate` 如果容器已经存在了，则不重新创建，不能与 `--force-recreate` 同时使用。
  - `kill`：通过发送 `SIGKILL` 信号来强制停止服务容器。
    - 选项：
      - `-s 信号`：指定发送的信号，如`-s SIGINT`
    - 参数：需要停止的服务容器
  - `pause`：暂停服务容器
    - 参数：需要暂停的服务容器
  - `restart`：重启服务
    - 选项：
      - `-t, --timeout TIMEOUT`：指定重启前停止容器的超时（默认为 10 秒）



### Compose模板文件

详见：https://yeasy.gitbook.io/docker_practice/compose/compose_file

示例：

```yaml
version: '3'

services:
  nginx:
    image: nginx:1.25.2
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/html:/usr/share/nginx/html:ro

  java-server:
    image: eclipse-temurin:17.0.6_10-jre-focal
    ports:
      - "8080:8080"
    command: java -jar /opt/app/sandpile-1.0-SNAPSHOT.jar --spring.config.location=/opt/app/application.yml
    volumes:
      - ./java-server:/opt/app

  mysql:
    image: mysql:8.1.0
    environment:
      - MYSQL_ROOT_PASSWORD=1234
    ports:
      - "30700:3306"
    volumes:
      - mysql-files:/var/lib/mysql-files
      - mysql-conf:/etc/mysql
      - mysql-logs:/var/log
      - mysql-data:/var/lib/mysql


volumes:
  mysql-files:
  mysql-conf:
  mysql-logs:
  mysql-data:
```

