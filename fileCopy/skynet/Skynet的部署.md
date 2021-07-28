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



