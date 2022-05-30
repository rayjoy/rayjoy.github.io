# Docker使用总结

## 1.测试所使用环境

- **ubuntu 16.04 server**系统。
- 有些指令在**fish shell**中运行不正常，请使用**bash shell**

## 2.安装

### 添加apt国内源：
- 添加GPG密钥：
    > `curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -`

- 添加清华大学Docker软件源：
    > `sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"`

### 安装`docker-ce`: 
> `sudo apt-get update;sudo apt-get install docker-ce`

### 启动docker服务：
> `sudo systemctl enable docker; sudo systemctl start docker`

### 建立**docker**用户组
> `sudo groupadd docker`

### 将用户加入**docker**用户组：

- 指令：
    > `sudo usermod -aG docker $USER`

- 注: 
    > 这一步可能不需要，因为此时系统可能已经存在**docker**用户，这种情况下运行此指令系统会报用户已存在的警告，忽略即可。

### 使用阿里云镜像加速器:
- 从[阿里云](https://cr.console.aliyun.com/#/accelerator)获取加速器地址为：
    > https://xxxxxxxx.mirror.aliyuncs.com

- 在`/etc/docker/daemon.json`中添加如下内容：

```json
{
    "registry-mirrors": [
         "https://xxxxxxxx.mirror.aliyuncs.com"
    ]
}
```

### 重启**docker**服务：
> `sudo systemctl daemon-reload; sudo systemctl restart docker`.

### 可能需要重新登录终端来获取**docker 用户权限**。

## 3.使用

### 获取镜像：
> `docker image pull ubuntu:16.04`。

### 运行容器：
- 例子1：
    > `docker container run -idt --restart=always -p 80:1080 ubuntu:16.04 bash`。

- 例子2：
    > `docker container run -it --rm ubuntu:16.04 bash`

- 指令选项说明：
    > **-i:** 以交互模式运行容器，通常与 **-t** 同时使用；
    > **-t:** 为容器重新分配一个伪输入终端，通常与 **-i** 同时使用；
    > **-d:** 后台运行容器，并返回容器ID；
    > **--rm:** 容器退出后自动删除该容器并清理容器内部的文件系统；
    >  **--restart=always:** 容器自动重新启动，如果容器已经启动了可以使用如下命令配置：`docker update --restart=always <CONTAINER ID>`；
    >  **-p:** 端口映射，形式为**-p IP:host_port:container_port**，例子1中将本机端口80映射到容器内部端口1080；

### 列出系统中的镜像：
- `docker image ls -a`;
    > **-a** 选项表示全部列出

### 列出系统中的容器：
- 例子：
> `docker container ls -a`

- 说明：
> **-a** 选项表示全部列出

### 启动已终止容器：
> `docker container start [name/id/..]`

### 进入容器
- 例1：
> `docker exec -it <CONTAINER ID> bash`

### 挂载主机目录
- 使用`--mount 标记`

- 例1：
>`docker run -idt --mount type=bind,source=/src/webapp,target=/opt/webapp python:2.7 bash`

### 利用**Dockerfile**定制镜像：
- 在空白目录中建立一个文本文件，命名为**Dockerfile**,
- 写入内容，示例:
```plain
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- 构建镜像：
> `docker image build -t nginx:v3 .`

### 显示docker占用空间的详细信息：
> docker system df

### 查看容器日志

> docker logs -f container-name
> - -f: 跟踪日志输出
> - --since: 显示某个开始时间的所有日志
> - -t: 显示时间戳
> - --tail: 仅列出最新N条容器日志

### 查看容器详细信息

1. ·docker inspect container-id/name

### 容器连接主机硬件：

- [Docker容器内访问宿主机硬件资源——树莓派编译GPIO驱动，通过容器控制GPIO](https://blog.csdn.net/tianhuanqingyun/article/details/91580778)。
- Docker提供了三种访问硬件设备的方式，参考[stackoverflow](https://stackoverflow.com/questions/30059784/docker-access-to-raspberry-pi-gpio-pins):
  - 使用 **–privileged** 选项, 比如 `\docker run --privileged -d whatever`,使用"–privileged=true"会拥有宿主机上root的权限，这对容器的权限管理而言是极度不安全的，在调试过程中使用并无问题，但在生产环境中却不太合适，所以默认情况下–privileged=false；
  - 使用 **–device** 选项，比如 `docker run --device /dev/gpiomem -d whatever`,使用"–device"可以在不打开–privileged选项的情况下访问宿主机的设备，默认情况下容器拥有读，写，创建设备文件的权限，可以使用“r”，“w”，"m"来管理权限。使用这种方式可以很好地配合自己编写的设备驱动，并在容器内灵活的访问设备节点。
  - 使用 **–volume** 选项，比如 `docker run -v /sys:/sys -d whatever`, 挂载数据卷的方式是实现数据持久化最常用的一种方式，它可以通过“rw”，“ro”管理文件的读写权限，因为linux内核文件系统的思想，它同样可以用来访问设备节点，但在实践中访问设备时，有时可能存在访问权限或者其他问题无法在宿主机内直接控制设备节点。

### docker 使用自定义 dns

- 在 `/etc/docker/daemon.json` 文件中添加 `"dns": ["119.29.29.29", "180.76.76.76"]`
- 参考资料：[Using host DNS in Docker container with Ubuntu 18](https://l-lin.github.io/post/2018/2018-09-03-docker_ubuntu_18_dns/)

### end

