# Docker2

## 1.docker-compose

### 1.docker-compose介绍

![image-20221117125605292](iamge/image-20221117125605292.png)

### 2.docker-compose的安装

Windows和Mac在默认安装了docker desktop以后，docker-compose随之自动安装

```powershell
PS C:\Users\Peng Xiao\docker.tips> docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```



Linux用户需要自行安装

最新版本号可以在这里查询 https://github.com/docker/compose/releases

```powershell
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```



熟悉python的朋友，可以使用pip去安装docker-Compose

```powershell
$ pip install docker-compose
```



### 3.compose 文件的结构和版本

docker compose文件的语法说明 https://docs.docker.com/compose/compose-file/

#### 基本语法结构

```yml
version: "3.8"

services: # 容器
  servicename: # 服务名字，这个名字也是内部 bridge网络可以使用的 DNS name
    image: # 镜像的名字
    command: # 可选，如果设置，则会覆盖默认镜像里的 CMD命令
    environment: # 可选，相当于 docker run里的 --env
    volumes: # 可选，相当于docker run里的 -v
    networks: # 可选，相当于 docker run里的 --network
    ports: # 可选，相当于 docker run里的 -p
  servicename2:

volumes: # 可选，相当于 docker volume create

networks: # 可选，相当于 docker network create
```



以 Python Flask + Redis练习：为例子，改造成一个docker-compose文件

```sh
docker image pull redis
docker image build -t flask-demo .

# create network
docker network create -d bridge demo-network

# create container
docker container run -d --name redis-server --network demo-network redis
docker container run -d --network demo-network --name flask-demo --env REDIS_HOST=redis-server -p 5000:5000 flask-demo
```



docker-compose.yml 文件如下

```yml
version: "3.8"

services:
  flask-demo:
    image: flask-demo:latest
    environment:
      - REDIS_HOST=redis-server
    networks:
    - demo-network
    ports:
      - 8080:5000

  redis-server:
    image: redis:latest
    networks:
     - demo-network

networks:
  demo-network:
```



#### docker-compose 语法版本

向后兼容

https://docs.docker.com/compose/compose-file/

dockerfile 也可以指定

```yml
version: '3.8'

services:
  express-demo:
    build:
      context: ./server
      dockerfile: dockerfile.env
    image: express-demo-env
    entrypoint: -REDIS_HOST=redis-serever
    networks:
      - demo-network
    ports:
      - 3000:3000

networks:
  demo-network:

```

```powershell
docker-compose pull #可以提前拉镜像
```

```
docker-compose build
```

```
docker-compose up -d --build
```



### 4更新



### 5.水平拓展（负载均衡）

首先启动

```
docker-compose up -d --build
```

启动三组

```powershell
 docker-compose up -d --scale express-demo=3
```

```powershell
docker-compose logs -f -t
```

```powershell
docker-compose restart nginx
```

重启nginx后负载均衡生效

github地址

```
https://github.com/ityifan/docker_demo
```



### 6.环境变量

Docker-compose.yml

```yml
version: "3.8"

services:
  express:
    build:
      context: ./server
      dockerfile: dockerfile
    image: express-demo:latest
    environment:
      - REDIS_HOST=redis-server
      - NODE_ENV=v0
      - REDIS_PASS=${REDIS_PASSWORD}
    networks:
      - backend
      - frontend

  redis-server:
    image: redis:latest
    command: redis-server --requiepass ${REDIS_PASSWORD}
    networks:
      - backend

  nginx:
    image: nginx:stable-alpine
    ports:
      - 8000:80
    depends_on:
      - express
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./var/log/nginx:/var/log/nginx
    networks:
      - frontend

networks:
  backend:
  frontend:

```

.env 与docker—compose.yml同级

```.env
REDIS_PASSWORD=abc123
```

查看docker-compose config

```powershell
docker-compose config
```

.env文件可以更改 如果更改 需要指定

```
docker-compose --env-file .\myenv config
```

如果进行up

```
docker-compose --env-file .\myenv up -d
```



### 7.服务依赖和健康检查

docker的服务依赖和健康检查

Dcokerfile

```sh
HEALTHCHECK --interval=30s --timeout=30s \
    CMD curl -f https://localhost:5000/ || exit 1
```

Dockerfile

```dockerfile
FROM node

RUN apt-get update && \
    apt-get install -y curl

WORKDIR /app/

COPY package.json /app/

COPY env.js /app/

COPY app.js /app/

COPY cRedis.js /app/

RUN npm install

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=30s \
    CMD curl -f https://localhost:5000/ || exit 1

CMD ["node","app.js","-h","0.0.0.0"]
```



```sh
docker container run -d --network mybridge --env REDIS_PASS=abc123 flask-demo
```



Dockerfile healthcheck https://docs.docker.com/engine/reference/builder/#healthcheck

docker compose https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck

健康检查是容器运行状态的高级检查，主要是检查容器所运行的进程是否能正常的对外提供“服务”，比如一个数据库容器，我们不光 需要这个容器是up的状态，我们还要求这个容器的数据库进程能够正常对外提供服务，这就是所谓的健康检查。

容器本身有一个健康检查的功能，但是需要在Dockerfile里定义，或者在执行docker container run 的时候，通过下面的一些参数指定

```
--health-cmd string              Command to run to check health
--health-interval duration       Time between running the check
                                (ms|s|m|h) (default 0s)
--health-retries int             Consecutive failures needed to
                                report unhealthy
--health-start-period duration   Start period for the container to
                                initialize before starting
                                health-retries countdown
                                (ms|s|m|h) (default 0s)
--health-timeout duration        Maximum time to allow one check to
```



#### 示例源码

我们以下面的这个flask容器为例，相关的代码如下

```dockerfile
PS C:\Users\Peng Xiao\code-demo\compose-env\flask> dir


    目录: C:\Users\Peng Xiao\code-demo\compose-env\flask


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2021/7/13     15:52            448 app.py
-a----         2021/7/14      0:32            471 Dockerfile


PS C:\Users\Peng Xiao\code-demo\compose-env\flask> more .\app.py
from flask import Flask
from redis import StrictRedis
import os
import socket

app = Flask(__name__)
redis = StrictRedis(host=os.environ.get('REDIS_HOST', '127.0.0.1'),
                    port=6379, password=os.environ.get('REDIS_PASS'))


@app.route('/')
def hello():
    redis.incr('hits')
    return f"Hello Container World! I have been seen {redis.get('hits').decode('utf-8')} times and my hostname is {socket.gethostname()}.\n"

PS C:\Users\Peng Xiao\code-demo\compose-env\flask> more .\Dockerfile
FROM python:3.9.5-slim

RUN pip install flask redis && \
    apt-get update && \
    apt-get install -y curl && \
    groupadd -r flask && useradd -r -g flask flask && \
    mkdir /src && \
    chown -R flask:flask /src

USER flask

COPY app.py /src/app.py

WORKDIR /src

ENV FLASK_APP=app.py REDIS_HOST=redis

EXPOSE 5000

HEALTHCHECK --interval=30s --timeout=3s \
    CMD curl -f http://localhost:5000/ || exit 1

CMD ["flask", "run", "-h", "0.0.0.0"]
```



上面Dockerfili里的HEALTHCHECK 就是定义了一个健康检查。 会每隔30秒检查一次，如果失败就会退出，退出代码是1

#### 构建镜像和创建容器

构建镜像，创建一个bridge网络，然后启动容器连到bridge网络

```sh
$ docker image build -t flask-demo .
$ docker network create mybridge
$ docker container run -d --network mybridge --env REDIS_PASS=abc123 flask-demo
```



查看容器状态

```sh
$ docker container ls
CONTAINER ID   IMAGE        COMMAND                  CREATED       STATUS                            PORTS      NAMES
059c12486019   flask-demo   "flask run -h 0.0.0.0"   4 hours ago   Up 8 seconds (health: starting)   5000/tcp   dazzling_tereshkova
```



也可以通过docker container inspect 059 查看详情， 其中有有关health的

```sh
"Health": {
"Status": "starting",
"FailingStreak": 1,
"Log": [
    {
        "Start": "2021-07-14T19:04:46.4054004Z",
        "End": "2021-07-14T19:04:49.4055393Z",
        "ExitCode": -1,
        "Output": "Health check exceeded timeout (3s)"
    }
]
}
```



经过3次检查，一直是不通的，然后health的状态会从starting变为 unhealthy

```sh
docker container ls
CONTAINER ID   IMAGE        COMMAND                  CREATED       STATUS                     PORTS      NAMES
059c12486019   flask-demo   "flask run -h 0.0.0.0"   4 hours ago   Up 2 minutes (unhealthy)   5000/tcp   dazzling_tereshkova
```



#### 启动redis服务器

启动redis，连到mybridge上，name=redis， 注意密码

```sh
$ docker container run -d --network mybridge --name redis redis:latest redis-server --requirepass abc123
```



经过几秒钟，我们的flask 变成了healthy

```sh
$ docker container ls
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS                   PORTS      NAMES
bc4e826ee938   redis:latest   "docker-entrypoint.s…"   18 seconds ago   Up 16 seconds            6379/tcp   redis
059c12486019   flask-demo     "flask run -h 0.0.0.0"   4 hours ago      Up 6 minutes (healthy)   5000/tcp   dazzling_tereshkova
```