[TOC]

## **RUN 执行命令:**

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样

  ```Dockerfile
  RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
  ```

- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。



## **COPY 复制文件:** 

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



## ADD 更高级的复制文件:

- shell模式 `ADD [--chown=<user>:<group>] <源路径>... <目标路径>`
- 函数调用式  `ADD [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]`

`源路径>` 可以是一个 `URL`，这种情况下，Docker 引擎会试图去下载这个链接的文件放到 `<目标路径>` 去。下载后的文件权限自动设置为 `600`，如果这并不是想要的权限，那么还需要增加额外的一层 `RUN`进行权限调整，另外，如果下载的是个压缩包，需要解压缩，也一样还需要额外的一层 `RUN` 指令进行解压缩。所以不如直接使用 `RUN` 指令，然后使用 `wget` 或者 `curl` 工具下载，处理权限、解压缩、然后清理无用文件更合理。

 `<源路径>` 为一个 `tar` 压缩文件的话，压缩格式为 `gzip`, `bzip2` 以及 `xz` 的情况下，`ADD` 指令将会自动解压缩这个压缩文件到 `<目标路径>` 去。

`--chown=<user>:<group>` 选项来改变文件的所属用户及所属组。

```Dockerfile
ADD --chown=55:mygroup files* /mydir/
ADD --chown=bin files* /mydir/
ADD --chown=1 files* /mydir/
ADD --chown=10:11 files* /mydir/
```

```
############## ps:

COPY` 和 `ADD` 指令中选择的时候，可以遵循这样的原则，所有的文件复制均使用 `COPY` 指令，仅在需要自动解压缩的场合使用 `ADD`
```



## CMD 容器启动命令:

- `shell` 格式：`CMD <命令>`
- `exec` 格式：`CMD ["可执行文件", "参数1", "参数2"...]`
- 参数列表格式：`CMD ["参数1", "参数2"...]`。在指定了 `ENTRYPOINT` 指令后，用 `CMD` 指定具体的参数。

`CMD` 指令就是用于指定默认的容器主进程的启动命令的。

启动容器实际上就是 启动容器进程，当容器主进程结束时，容器也就退出了，所以，在容器中启动的程序应当跑在前台，而不是后台。

```Dockerfile
# 错误命令
CMD service nginx start
# 容器执行后就立即退出了

# 正确的做法是直接执行 nginx 可执行文件，并且要求以前台形式运行
CMD ["nginx", "-g", "daemon off;"]
```



## ENTRYPOINT 入口点:z



## **ENV **设置环境变量:



## ARG 构建参数:



## VOLUME 定义匿名卷:



## EXPOSE 暴露端口:



## WORKDIR 指定工作目录:



## USER 指定当前用户:



## HEALTHCHECK 健康检查:



## ONBUILD 为他人作嫁衣:

