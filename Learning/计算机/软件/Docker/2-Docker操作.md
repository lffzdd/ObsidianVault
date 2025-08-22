# `run`
`run` 是 Docker 命令的一部分，用于创建并启动一个新的容器。以下是对 `docker run` 命令的详细解释：
1. **基本用法**：
   - `docker run` 命令用于从指定的镜像创建并启动一个新的容器。它是 Docker 中最常用的命令之一。
2. **命令格式**：
   - 基本的命令格式如下：
     ```
     docker run [OPTIONS] IMAGE [COMMAND] [ARG…]
     ```
   - 其中，`IMAGE` 是您要使用的 Docker 镜像，`COMMAND` 是可选的命令，`ARG…` 是可选的参数。
3. **常用选项**：
   - `-d`：在后台运行容器（分离模式）。
   - `-it`：交互式运行容器并分配一个伪终端，通常用于启动交互式 shell。
   - `--name`：为容器指定一个名称。
   - `-p`：将宿主机的端口映射到容器的端口。
   - `-v`：将宿主机的目录挂载到容器中，通常用于数据持久化。
4. **应用场景**：
   - `docker run` 常用于启动新的应用程序容器。例如，您可以使用它来启动一个 Web 服务器、数据库或任何其他应用程序。
 **示例**：
假设您想要从 `nginx` 镜像启动一个新的容器，并将其命名为 `my_nginx`，同时将宿主机的 8080 端口映射到容器的 80 端口，可以使用以下命令：
```bash
docker run -d --name my_nginx -p 8080:80 nginx
```
这将启动一个新的 Nginx 容器，并在后台运行。
## `ps``
您可以使用 `docker ps` 命令查看当前运行的容器及其名称
# `exec`
`exec` 是 Docker 命令的一部分，用于在运行中的容器内执行命令。以下是对 `docker exec` 命令的详细解释：
1. **基本用法**：
   - `docker exec` 命令允许您在一个已经运行的容器中执行新的命令。例如，您可以在容器内启动一个新的 shell 会话，或者运行特定的应用程序。
2. **命令格式**：
   - 基本的命令格式如下：
     ```
     docker exec [OPTIONS] CONTAINER COMMAND [ARG…]
     ```
   - 其中，`CONTAINER` 是您要在其中执行命令的容器的名称或 ID，`COMMAND` 是您要执行的命令，`ARG…` 是可选的参数。
3. **常用选项**：
   - `-it`：这两个选项通常一起使用，`-i` 表示交互式，`-t` 表示分配一个伪终端。这样可以让您在容器内与命令进行交互。
   - 例如，您可以使用以下命令在容器中启动一个交互式的 Bash shell：
     ```
     docker exec -it my_container /bin/bash
     ```
4. **应用场景**：
   - `docker exec` 常用于调试和管理容器。例如，您可以在容器内查看日志、检查文件、运行数据库命令等。
### 示例：
假设您有一个名为 `my_container` 的正在运行的容器，您可以使用以下命令在其中启动一个交互式的 shell：
```bash
docker exec -it my_container /bin/bash
```
这将使您能够在 `my_container` 内部执行命令并与其交互。
### 总结：
`docker exec` 命令用于在运行中的容器内执行命令，常用于调试和管理容器。通过使用 `-it` 选项，您可以启动一个交互式的 shell 会话，方便地与容器进行交互。
# `diff`
`diff` 是 Docker 命令的一部分，用于比较容器的文件系统与其基础镜像之间的差异。以下是对 `docker diff` 命令的详细解释：
1. **基本用法**：
   - `docker diff` 命令用于查看一个容器的文件系统与其基础镜像之间的变化。这可以帮助您了解在容器运行期间对文件系统所做的更改。
2. **命令格式**：
   - 基本的命令格式如下：
     ```
     docker diff CONTAINER
     ```
   - 其中，`CONTAINER` 是您要检查的容器的名称或 ID。
3. **输出结果**：
   - `docker diff` 命令的输出会列出文件系统中发生的变化，通常包括以下几种状态：
     - `A`：表示文件或目录被添加。
     - `C`：表示文件或目录被修改。
     - `D`：表示文件或目录被删除。
4. **应用场景**：
   - `docker diff` 常用于调试和审计，帮助开发者了解容器在运行期间对文件系统的更改。这对于排查问题或优化容器的使用非常有用。
### 示例：
假设您有一个名为 `my_container` 的正在运行的容器，您可以使用以下命令查看其文件系统的变化：
```bash
docker diff my_container
```
这将列出自容器启动以来对文件系统所做的所有更改。
### 总结：
`docker diff` 命令用于比较容器的文件系统与其基础镜像之间的差异，列出文件系统中发生的变化。它可以帮助开发者了解容器在运行期间对文件系统的更改，常用于调试和审计。
# `commit`
现在我们定制好了变化，我们希望能将其保存下来形成镜像。
要知道，当我们运行一个容器的时候（如果不使用卷的话），我们做的任何文件修改都会被记录于容器存储层里。而 Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。
`docker commit` 的语法格式为：
```shell
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```
我们可以用下面的命令将容器保存为镜像：
```shell
$ docker commit \
    --author "Tao Wang <twang2218@gmail.com>" \
    --message "修改了默认网页" \
    webserver \
    nginx:v2
sha256:07e33465974800ce65751acc279adc6ed2dc5ed4e0838f8b86f0c87aa1795214
```
其中 `--author` 是指定修改的作者，而 `--message` 则是记录本次修改的内容。这点和 `git` 版本控制相似，不过这里这些信息可以省略留空。
我们可以在 `docker image ls` 中看到这个新定制的镜像：
```
$ docker image ls nginx
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v2                  07e334659748        9 seconds ago       181.5 MB
nginx               1.11                05a60462f8ba        12 days ago         181.5 MB
nginx               latest              e43d811ce2f4        4 weeks ago         181.5 MB
```
我们还可以用 `docker history` 具体查看镜像内的历史记录，如果比较 `nginx:latest` 的历史记录，我们会发现新增了我们刚刚提交的这一层。
```
$ docker history nginx:v2
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
07e334659748        54 seconds ago      nginx -g daemon off;                            95 B                修改了默认网页
e43d811ce2f4        4 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon    0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  EXPOSE 443/tcp 80/tcp        0 B
<missing>           4 weeks ago         /bin/sh -c ln -sf /dev/stdout /var/log/nginx/   22 B
<missing>           4 weeks ago         /bin/sh -c apt-key adv --keyserver hkp://pgp.   58.46 MB
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.11.5-1   0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  MAINTAINER NGINX Docker Ma   0 B
<missing>           4 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/bash"]            0 B
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:23aa4f893e3288698c   123 MB
```
新的镜像定制好后，我们可以来运行这个镜像。
```
docker run --name web2 -d -p 81:80 nginx:v2
```
这里我们命名为新的服务为 `web2`，并且映射到 `81` 端口。访问 `http://localhost:81` 看到结果，其内容应该和之前修改后的 `webserver` 一样。
至此，我们第一次完成了定制镜像，使用的是 `docker commit` 命令，手动操作给旧的镜像添加了新的一层，形成新的镜像，对镜像多层存储应该有了更直观的感觉。
## ## 慎用 `docker commit`
使用 `docker commit` 命令虽然可以比较直观的帮助理解镜像分层存储的概念，但是实际环境中并不会这样使用。
首先，如果仔细观察之前的 `docker diff webserver` 的结果，你会发现除了真正想要修改的 `/usr/share/nginx/html/index.html` 文件外，由于命令的执行，还有很多文件被改动或添加了。这还仅仅是最简单的操作，如果是安装软件包、编译构建，那会有大量的无关内容被添加进来，将会导致镜像极为臃肿。
此外，使用 `docker commit` 意味着所有对镜像的操作都是黑箱操作，生成的镜像也被称为 **黑箱镜像**，换句话说，就是除了制作镜像的人知道执行过什么命令、怎么生成的镜像，别人根本无从得知。而且，即使是这个制作镜像的人，过一段时间后也无法记清具体的操作。这种黑箱镜像的维护工作是非常痛苦的。
而且，回顾之前提及的镜像所使用的分层存储的概念，除当前层外，之前的每一层都是不会发生改变的，换句话说，任何修改的结果仅仅是在当前层进行标记、添加、修改，而不会改动上一层。如果使用 `docker commit` 制作镜像，以及后期修改的话，每一次修改都会让镜像更加臃肿一次，所删除的上一层的东西并不会丢失，会一直如影随形的跟着这个镜像，即使根本无法访问到。这会让镜像更加臃肿。