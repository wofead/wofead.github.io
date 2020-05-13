# Skynet的部署

[toc]

## Docker

学习参照：https://blog.csdn.net/qq_26870933/article/details/81675201

docker是一个开源的应用容器引擎，可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口，更重要的是容器性能开销极低。

docker包含三个基本概念：

* **镜像：**docker镜像就相当于是一个root文件系统
* **容器：**镜像（image）和容器（container）的关系，就像是面向对象程序设计中的类和实例一样。镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
* **仓库：**仓库可看着一个代码控制中心，用来保存镜像。

还有其他两个概念：

* **客户端**：客户端通过命令行或者其他工具使用docker sdk与docker的守护进程通信。
* **主机**：一个物理或者虚拟的机器用于执行Docker守护进程和容器。

docker使用C/S架构模式，使用远程API来管理和创建docker容器。

安装：可以通过下载阿里云的镜像来下载：http://mirrors.aliyun.com/docker-toolbox/windows/docker-toolbox/

为了在docker中更快的安装各种插件，可以在Docker中设置镜像加速，在setting中，的Docker Engine中，registry-mirrors中添加加速镜像。

```
"registry-mirrors": [
    "https://hub-mirror.c.163.com/",
    "https://reg-mirror.qiniu.com"
  ]
```

### docker中常见的命令：

1. docker ps : 查看docker容器的详细信息
2. docker images: 查看本地的镜像

win下docker的常用配置：

1. 启动一个的PowerShell（即以管理员身份运行）。搜索PowerShell，右键单击，然后选择以管理员身份运行。在PowerShell提示符下键入：`Set-ExecutionPolicy RemoteSigned`
2. 安装posh-dockerPowerShell模块以自动完成Docker命令，键入：`Install-Module posh-docker`
3. 配置阿里云镜像加速：https://cr.console.aliyun.com/ ，在镜像加速器中，拷贝地址到上面设置镜像加速的地方。

### 用Dockerfile定义一个镜像

使用docker，你可以将一个可移植的python运行库作为一个映射，不需要安装。然后，你的构建将以基础python镜像与应用程序代码一起包括在内，确保你的应用程序，依赖项和运行时都一起运行。

这些可移植的镜像是由一个叫做Dockfile的东西来定义的。

创建一个Docker文件夹：里面放三个文件 `app.py , Dockerfile 和 requirements`

```python
1.Dockerfile：
# Use an official Python runtime as a parent image
FROM python:2.7-slim
 
# Set the working directory to /app
WORKDIR /app
 
# Copy the current directory contents into the container at /app
ADD . /app
 
# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt
 
# Make port 80 available to the world outside this container
EXPOSE 80
 
# Define environment variable
ENV NAME World
 
# Run app.py when the container launches
CMD ["python", "app.py"]
 
2.app.py：
from flask import Flask
from redis import Redis, RedisError
import os
import socket
 
# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)
 
app = Flask(__name__)
 
@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"
 
    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)
 
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
 
3. requirements.txt：
Flask
Redis
```

接下来构建镜像:在我们的docker目录下打开powershell:然后运行`docker build -t friendlyhello .` ，ps:千万不要忘记点`.`。

然后我们可以通过docker images命令看到我们的friendlyhello镜像了，我们使用`docker run -p 4000:80 friendlyhello`来运行镜像，可以看到Python正在为应用程序提供消息的[http://0.0.0.0:80](http://0.0.0.0/)。但是，这个消息来自容器内部，它不知道我们将该容器的端口80映射到4000，从而打开URL： [http://localhost:4000](http://localhost:4000/) 。

停止容器运行调用`docker stop <id>`就行了，至于id可以使用 `docker ps`来查看，或者使用 `docker container ls`。

### Docker Hub的常用操作

我们登录到docker hub上，然后创建一个仓储，很像github。

1. 登录docker：`docker login`
2. 标记镜像：就是要把镜像push那个仓储中：`docker tag friendlyhello 用户名/仓储名:标记` 用户名类似于github的用户名，仓储名和github的仓储名意义一致，标记对应远程仓储中的tag。
3. 接下来我们就可以从远程仓储中提取并运行了： `docker run -p 4000:80 614667387/jow:test`

### 服务

在分布式的应用程序中，应用程序的不同部分被称为服务，例如我们常说的游戏中的game服、登录服以及聊天服等等。服务实际上是“生产中的容器”。服务只是运行一个镜像，但他编码镜像运行的方式-应该使用哪个端口，容器应该运行多少个副本，一遍服务具有所需的容量，以及等等。缩放服务会更改运行该软件的容量实例数量，从而为流程中的服务分配更多的计算资源。

使用docker平台定义，运行和扩展服务非常简单-只需编写docker-compose.yml文件即可。

```dockerfile
version: "3"
services:
  web:
    # replace username/repo:tag with your name and image details
    image: 614667387/jow:test #镜像
    deploy:
      replicas: 5 #5个实例
      resources:
        limits:
          cpus: "0.1" #10%的cpu
          memory: 50M #50M的RAM
      restart_policy:
        condition: on-failure #一个失败重新启动容器
    ports:
      - "80:80" #主机端口80映射web端口80
    networks:
      - webnet #默认设置(负载平衡覆盖网络)
networks:
  webnet:
```

接下来运行新的负载均衡应用程序：

1. 运行命令`docker swarm init`
2. 运行`docker stack deploy -c docker-compose.yml getstartedlab`.这个负载均衡应用程序命名为getstartedlab
3. 这个时候我们就可以看主机上有几个容器实例了：`docker service ls`
4. 在服务中运行的单个容器称为任务，可以看到上面有个getstartedlab_web服务，通过`docker service ps getstartedlab_web`查看此服务下的任务
5. 也可以使用`docker container ls -q`来看任务
6. 关闭应用程序使用`docker stack rm getstartedlab`
7. 关闭群`docker swarm leave --force`

### 集群

swarm是运行docker并加入到集群中的一组机器。但是现在它们将由群集管理器在群集上执行。群体中的机器可以是物理的或虚拟的。加入群体后，它们被称为节点。

swarm管理人员可以使用多种策略来运行容器，比如“最空的节点” -它使用容器填充使用最少的机器。或‘全局’，这确保了每个机器只能得到指定容器的一个实例。你可以指示swarm manager在compose文件中使用这些策略。

群体管理者是群体中唯一可以执行你的命令的机器，或者授权其它机器作为工作者加入群里。工人提供能力，并没有权利告诉任何其它机器可以做什么和不可以做什么。

到目前为止，之前都是在本地机器上以单主机模式使用Docker。但是Docker也可以切换到群集模式，这就是使用群集的原因。启用群模式使当前机器成为群管理器。则Docker将运行您正在管理的群集上执行的命令，而不仅仅是在当前的机器上。

创建个集群。一个群由多个节点组成，可以是物理机或虚拟机。基本概念很简单：运行docker swarm init启用群模式，使当前的机器称为群管理器，然后docker swarm join在其他机器上运行，让它们作为工人加入群体。

1. 启动Hyper-V管理器 
2. 单击右侧菜单中的虚拟交换机管理器 
3. 单击创建类型为外部网络的虚拟交换机，给它的名称myswitch，并检查框共享您的主机的活动网络适配器 
4. `docker-machine create -d hyperv –hyperv-virtual-switch “myswitch” myvm1 ` 和`docker-machine create -d hyperv –hyperv-virtual-switch “myswitch” myvm2 `
5. docker-machine ls 列出机器并获取其IP地址。

接下来初始化群并添加节点：

1. docker-machine ssh myvm31
2. 然后让myvm1为一个管理员：docker swarm init 
3. docker node ls ，可以看到myvm1 已经成为管理员了
4. 以管理员身份再运行一个cmd.exe.然后运行命令：docker-machine ssh myvm2
5. docker swarm join --token SWMTKN-1-0csyw4yz6uxob90h0b8ejoimimrgisiuy9t2ugm8c1mxfvxf99-7q7w5jw1mrjk1jlri2bcgqmu8 10.211.106.194:2377(对应你的tocken，在myvm1，成为管理员执行成功之后的输出里面)
6. 然后再切换到myvm1 的cmd.exe中执行命令：docker node ls 
7. docker swarm leave：离开集群

在集群上部署应用程序：

1. docker-machine为swarm管理器配置一个shell ，运行docker-machine env myvm31
2. @FOR /f "tokens=*" %i IN ('docker-machine env myvm1') DO @%i
3. 再运行docker-machine ls以验证它myvm1 是否为活动机器 
4. 在swarm管理器上部署应用程序
5. 使用之前的docker-compose.yml.`docker stack deploy -c docker-compose.yml getstartedla`
6. docker stack ps getstartedlab 查看服务详情
7. 使用docker-machine ls,来看集群的信息



```dockerfile
比如说如果修改了docker-compose.yml文件后，执行命令：
docker stack deploy -c docker-compose.yml getstartedlab
再次运行以部署这些更改即可
比如说前面提到的移除应用程序：docker stack rm getstartedlab
离开群：docker swarm leave –force
重新启动已停止的虚拟机，执行：
docker-machine start <machine-name>
```

