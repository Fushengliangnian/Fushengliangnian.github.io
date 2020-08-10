[TOC]



## image镜像操作

**列出已经下载下来的镜像:**  `docker image ls `

**查看镜像、容器、数据卷所占用的空间:**  `docker system df`

**查看虚悬镜像:**  `docker image ls -f dangling=true` 

`虚悬镜像已经失去了存在的价值，是可以随意删除的`

**删除虚悬镜像:** `docker image prune`

**查看中间层镜像:** `docker image ls -a`

```中间层镜像: 为了加速镜像构建、重复利用资源，Docker 会利用 **中间层镜像**。所以在使用一段时间后，可能会看到一些依赖的中间层镜像。中间层镜像也是无标签的镜像，是其它镜像所依赖的镜像。这些无标签镜像不应该删除，否则会导致上层镜像因为依赖丢失而出错。实际上，这些镜像也没必要删除，因为之前说过，相同的层只会存一遍，而这些镜像是别的镜像的依赖，因此并不会因为它们被列出来而多存了一份，无论如何你也会需要它们。只要删除那些依赖它们的镜像后，这些依赖的中间层镜像也会被连带删除。

```

**部分镜像的查找:**

- 根据镜像名查询: `docker image ls 镜像名`
- 过滤器filter查询: `docker image ls -f since=mongo:3.2`

**根据特定的方式显示查找结果:**

- `-q` : 只显示 ID； ep:`docker image ls -q`

  `--filter` 配合 `-q` 产生出指定范围的 ID 列表，然后送给另一个 `docker` 命令作为参数，从而针对这组实体成批的进行某种操作的做法在 Docker 命令行使用过程中非常常见，不仅仅是镜像，将来我们会在各个命令中看到这类搭配以完成很强大的功能。

- 自定义展示结果 `--format` : [Go 的模板语法](https://gohugo.io/templates/go-templates/)

  ```bash
  # 只包含镜像ID和仓库名：
  $ docker image ls --format "{{.ID}}: {{.Repository}}"
  5f515359c7f8: redis
  05a60462f8ba: nginx
  fe9198c04d62: mongo
  00285df0df87: <none>
  f753707788c5: ubuntu
  f753707788c5: ubuntu
  1e0c3dd64ccd: ubuntu
  
  # 以表格等距显示，并且有标题行，和默认一样
  $ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
  IMAGE ID            REPOSITORY          TAG
  5f515359c7f8        redis               latest
  05a60462f8ba        nginx               latest
  fe9198c04d62        mongo               3.2
  00285df0df87        <none>              <none>
  f753707788c5        ubuntu              18.04
  f753707788c5        ubuntu              latest
  ```

**删除本地镜像:** `docker image rm`

- 根据ID、镜像名、摘要删除镜像:

  ```bash
  # 根据ID
  $ docker image rm 501
  
  # 根据镜像名
  $ docker image rm centos
  
  # 根据镜像摘要； 先查到摘要再进行删除
  $ docker image ls --digests
  REPOSITORY                  TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
  node                        slim                sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228   6e0c4c8e3913        3 weeks ago         214 MB
  
  $ docker image rm node@sha256:b4f0e0bdeb578043c1ea6862f0d40cc4afe32a4a582f3be235a3b164422be228
  ```

- 用 docker image ls 命令来配合:

  ```bash
  # 删除所有仓库名为 redis 的镜像
  $ docker image rm $(docker image ls -q redis)
  
  # 删除所有在 mongo:3.2 之前的镜像：
  $ docker image rm $(docker image ls -q -f before=mongo:3.2)
  ```

#### 使用 DockerFile 定制镜像

详情:<https://yeasy.gitbooks.io/docker_practice/content/image/build.html>

**FROM 指定基础镜像:**  必备的指令，并且必须是第一条指令:

在 [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如 [`nginx`](https://hub.docker.com/_/nginx/)、[`redis`](https://hub.docker.com/_/redis/)、[`mongo`](https://hub.docker.com/_/mongo/)、[`mysql`](https://hub.docker.com/_/mysql/)、[`httpd`](https://hub.docker.com/_/httpd/)、[`php`](https://hub.docker.com/_/php/)、[`tomcat`](https://hub.docker.com/_/tomcat/) 等；也有一些方便开发、构建、运行各种语言应用的镜像，如 [`node`](https://hub.docker.com/_/node)、[`openjdk`](https://hub.docker.com/_/openjdk/)、[`python`](https://hub.docker.com/_/python/)、[`ruby`](https://hub.docker.com/_/ruby/)、[`golang`](https://hub.docker.com/_/golang/) 等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 [`ubuntu`](https://hub.docker.com/_/ubuntu/)、[`debian`](https://hub.docker.com/_/debian/)、[`centos`](https://hub.docker.com/_/centos/)、[`fedora`](https://hub.docker.com/_/fedora/)、[`alpine`](https://hub.docker.com/_/alpine/) 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

```dockerfile
FROM scratch
...
```

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

**RUN 执行命令:**

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样

  ```Dockerfile
  RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
  ```

- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

**构建镜像:**

...

**镜像构建上下文:**

...

**COPY 复制文件:** 

- shell模式 `COPY [--chown=<user>:<group>] <源路径>... <目标路径>`
- 函数调用式  `COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

`COPY` 指令将从构建上下文目录中 `<源路径>` 的文件/目录复制到新的一层的镜像内的 `<目标路径>` 位置。

```Dockerfile
COPY package.json /usr/src/app/
```

`<源路径>` 可以是多个，甚至可以是通配符，其通配符规则要满足 Go 的 [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) 规则，如：

```Dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

`<目标路径>` 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 `WORKDIR`指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。

此外，还需要注意一点，使用 `COPY` 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这个特性对于镜像定制很有用。特别是构建相关文件都在使用 Git 进行管理的时候。

在使用该指令的时候还可以加上 `--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```Dockerfile
COPY --chown=55:mygroup files* /mydir/
COPY --chown=bin files* /mydir/
COPY --chown=1 files* /mydir/
COPY --chown=10:11 files* /mydir/
```

**ADD 更高级的复制文件:**

**CMD 容器启动命令:**

**ENTRYPOINT 入口点:**

**ENV 设置环境变量:**

**ARG 构建参数:**

**VOLUME 定义匿名卷:**

**EXPOSE 暴露端口:**

**WORKDIR 指定工作目录:**

**USER 指定当前用户:**

**HEALTHCHECK 健康检查:**

**ONBUILD 为他人作嫁衣:**

**Dockerfile 多阶段构建:**

...

**其他制作镜像的方式:**

...



## container 容器操作

**新建并启动容器:** 命令主要为 `docker run`

```bash
# 依靠ubuntu镜像生成容器后 命令输出一个 “Hello World”，之后终止容器。
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
Hello world

# 启动一个 bash 终端，允许用户进行交互
# -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
# -i 让容器的标准输入保持打开
$ docker run -t -i ubuntu:18.04 /bin/bash
root@af8bae53bdd3:/#
root@af8bae53bdd3:/# pwd
/
root@af8bae53bdd3:/# ls
bin boot dev etc home lib lib64 media mnt opt proc root run sbin srv sys tmp usr var
```

利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括:

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

**启动已终止的容器:** `docker container start`

**后台运行:** `-d`

```bash
# 容器会在后台运行并不会把输出的结果 (STDOUT) 打印到宿主机上面
docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a

# 查看后台运行的程序的结果 docker logs [container ID or NAMES] 
docker container logs a211		# a211 为短ID
```

**终止容器:** `docker container stop [container ID]`

**查看终止状态的容器:** `docker container ls -a`  (**Exited (0)**)

**重新启动终止状态的容器:** `docker container start [container ID]`

**将一个运行态的容器终止，然后再重新启动:** `docker container restart`

**进入在后台运行的容器中:**

- `docker attach` 命令: ( **不推荐**使用 )

  ```bash
  $ docker run -dit ubuntu
  243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
  
  $ docker container ls
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
  
  $ docker attach 243c
  root@243c32535da7:/#
  
  ####### 注意 #######
  # 如果从这个 stdin 中 exit，会导致容器的停止
  ```

- `docker exec` 命令: ( **推荐**使用 )

  ```bash
  # 执行 -d 后台 i 持续 t 终端输出 的容器
  $ docker run -dit ubuntu
  69d137adef7a8a689cbcb059e94da5489d3cddd240ff675c640c8d96e84fe1f6
  
  $ docker container ls
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  69d137adef7a        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           zealous_swirles
  
  # -i 界面没有 Linux 命令提示符，但命令执行结果仍然可以返回。
  $ docker exec -i 69d1 bash
  ls
  bin
  boot
  dev
  ...
  
  # 当 -i -t 参数一起使用时，则可以看到 Linux 命令提示符
  $ docker exec -it 69d1 bash
  root@69d137adef7a:/#
  root@69d137adef7a:/# exit
  # 从这个 stdin 中 exit，不会导致容器的停止
  ```

**导出本地容器快照到本地:** `docker export`

```bash
$ docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                    PORTS               NAMES
7691a814370e        ubuntu:18.04        "/bin/bash"         36 hours ago        Exited (0) 21 hours ago                       test
$ docker export 7691a814370e > ubuntu.tar
```

**从容器快照文件中再导入为镜像:** `docker import`

```bash
$ cat ubuntu.tar | docker import - test/ubuntu:v1.0
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
test/ubuntu         v1.0                9d37a6082e97        About a minute ago   171.3 MB

# 通过指定 URL 或者某个目录来导入
$ docker import http://example.com/exampleimage.tgz example/imagerepo
```

**删除容器:** `docker container rm`

```bash
# 删除已终止的容器
$ docker container rm  trusting_newton
trusting_newton

# 删除正在运行的容器，需要添加参数 -f 
$ docker container rm -f trusting_newton
trusting_newton

# 清理所有处于终止状态的容器
$ docker container prune
```



## 访问仓库 Repository

**登陆:** `docker login`

**退出登录:** `docker logout`

**拉取镜像:** 

- `docker search` 查找官方的镜像

- `docker pull` 下载到本地

  ```bash
  $ docker search centos
  NAME                                            DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
  centos                                          The official build of CentOS.                   465       [OK]
  tianon/centos                                   CentOS 5 and 6, created using rinse instea...   28
  blalor/centos                                   Bare-bones base CentOS 6.5 image                6                    [OK]
  saltstack/centos-6-minimal                                                                      6                    [OK]
  tutum/centos-6.4                                DEPRECATED. Use tutum/centos:6.4 instead. ...   5                    [OK]
  # DESCRIPTION 描述; STARS 收藏数; OFFICIAL 是否官方创建; AUTOMATED 是否自动创建
  
  $ docker pull centos
  ```

**推送镜像:**

```bash
# 使用镜像源创建一个指定镜像的标签
$ docker tag ubuntu:18.04 username/ubuntu:18.04

$ docker image ls

REPOSITORY                                               TAG                    IMAGE ID            CREATED             SIZE
ubuntu                                                   18.04                  275d79972a86        6 days ago          94.6MB
username/ubuntu                                          18.04                  275d79972a86        6 days ago          94.6MB

# 推送到 hub 上
$ docker push username/ubuntu:18.04

# 查询刚刚推上去的镜像
$ docker search username

NAME                      DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
username/ubuntu
```

**自动创建:**

- 创建并登录 Docker Hub，以及目标网站；
- 在目标网站中连接帐户到 Docker Hub；
- 在 Docker Hub 中 [配置一个自动创建](https://registry.hub.docker.com/builds/add/)；
- 选取一个目标网站中的项目（需要含 `Dockerfile`）和分支；
- 指定 `Dockerfile` 的位置，并提交创建。

**私有仓库的构建**

...



## 数据管理

...



## 使用网络

**外部访问容器:** 通过 `-P` 或 `-p` 参数来指定端口映射。

```bash
# -P Docker 会随机映射一个 `49000~49900` 的端口到内部容器开放的网络端口
$ docker run -d -P training/webapp python app.py

# 查看映射关系
$ docker container ls -l
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS                    NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  nostalgic_morse
# 本地主机的 49155 被映射到了容器的 5000 端口

# 查看应用的信息
$ docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -
```

`-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。

- **映射所有接口地址: ** `hostPort:containerPort` 

  ```bash
  # 将本地的 5000 端口映射到容器的 5000 端口
  $ docker run -d -p 5000:5000 training/webapp python app.py
  ```

- **映射到指定地址的指定端口:** `ip:hostPort:containerPort`

  ```bash
  # 指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1
  $ docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
  ```

- **映射到指定地址的任意端口:** `ip::containerPort` 

  ```bash
  # 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。
  $ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
  
  # 使用 udp 标记来指定 udp 端口
  $ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
  ```

**查看映射端口的配置:** `docker port`

```bash
$ docker port nostalgic_morse 5000
127.0.0.1:49155.

# 注意: 容器有自己的内部网络和 ip 地址（使用 docker inspect 可以获取所有的变量，Docker 还可以有一个可变的网络配置。）

# -p 可以多次使用来绑定多个端口
$ docker run -d \
    -p 5000:5000 \
    -p 3000:80 \
    training/webapp \
    python app.py
```



### 容器互联: 

**新建网络:** 

```bash
$ docker network create -d bridge my-net
# -d 参数指定 Docker 网络类型，有 bridge overlay; overlay 网络类型用于 Swarm mode
```

**连接容器: ** 

```bash
# 运行一个容器并连接到新建的 `my-net` 网络
$ docker run -it --rm --name busybox1 --network my-net busybox sh

# 打开新的终端，再运行一个容器并加入到 my-net 网络
$ docker run -it --rm --name busybox2 --network my-net busybox sh

# 再打开一个新的终端查看容器信息
$ docker container ls

CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
b47060aca56b        busybox             "sh"                11 minutes ago      Up 11 minutes                           busybox2
8720575823ec        busybox             "sh"                16 minutes ago      Up 16 minutes                           busybox1

# 通过 ping 来证明 busybox1 容器和 busybox2 容器建立了互联关系。
# 在 busybox1 容器输入以下命令
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms

# 用 ping 来测试连接 busybox2 容器，它会解析成 172.19.0.3。
# 在 busybox2 容器执行 ping busybox1，也会成功连接到
/ # ping busybox1
PING busybox1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.064 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.143 ms
```



**配置DNS:** 配置全部容器的 DNS ，可以在 `/etc/docker/daemon.json` 文件中增加以下内容来设置

```json
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```