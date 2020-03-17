
# Registry-Nacos

📢 本项目是基于[阿里开源的Nacos Docker](https://github.com/nacos-group/nacos-docker)进行部分修改。

📢 简化了官方Docker镜像，相当于一款`纯净版的Nacos-Docker`, 魔改后的`微服务注册、配置中心`。

> 这里用的是基础镜像是`nacos/nacos-server:1.1.3`,如果需要自定义版本，直接修改`DockerFile`即可

# 目录


- [部署方式](#1001)
    1. [ip模式](#1002)
    1. [hostname模式](#1003)
    1. [单机模式部署](#1004)
- [部署操作](#1005)


```
├─official                      # 官方提供的DockerFile和部署示例
│  
└─diydemo                       # 魔改后的部署文件
    ├─env/                      # 环境变量文件夹
    └─init.d/                   # 用于挂载的相关配置
    └─cluster-hostname.yaml     # 基于hostname模式的docker-compose部署文件
    └─cluster-ip.yaml           # 基于ip模式的docker-compose部署文件
    └─Dockerfile                # 用于构建自用镜像的DockerFlie
    └─standalone-mysql.yaml     # 基于Mysql的单机模式docker-compose部署文件

```

> 本文档仅介绍在Docker环境下进行Nacos部署, 其他平台部署相关介绍请参考[Nacos官网](https://nacos.io/zh-cn/docs/quick-start-spring.html)


<a name ="1001"></a>  
# 部署方式

**部署之前需要先构建需要使用的Nacos-Server镜像, 通过修改DockerFile中的基础镜像`tag标签`,可进行基础镜像版本切换。**

目前默认使用的1.1.3版本镜像地址 

> 如你你需要使用其他版本Nacos可自行构建使用

目前支持3种部署模式:`ip模式集群部署` 、 `hostname模式集群部署`和`单机模式部署` 



<a name ="1002"></a>  
## ip模式


即集群中各个节点通过`ip`进行访问, 部署时需要确保以下配置正确

- nacos-ip.env
- cluster-ip.yaml

`nacos-ip.env`中`NACOS_SERVERS`属性必须为各个节点的可访问的ip+port

```yaml
NACOS_SERVERS=10.16.10.166:8848 10.16.10.167:8848 10.16.10.168:8848     #3节点3服务器1端口
or
NACOS_SERVERS=10.16.10.166:8848 10.16.10.167:8849 10.16.10.168:8850     #3节点3服务器3端口
or
NACOS_SERVERS=10.16.10.166:8848 10.16.10.167:8848 10.16.10.167:8849     #3节点2服务器2端口

```

`cluster-ip.yaml`中环境变量中需要配置每个节点的ip和port(`NACOS_SERVER_IP和NACOS_SERVER_PORT`),

这里配置的ip和port必须要与`nacos-ip.env`的节点ipList对应

容器的port必须要与`NACOS_SERVER_PORT`对应

如下所示：

```yaml


  nacos1:
    container_name: nacos1
    build:
      context: .
      dockerfile: Dockerfile
    networks:
      nacos_net:
        ipv4_address: 172.16.238.10
    volumes:
      - ./cluster-logs/nacos1:/home/nacos/logs
      - ./init.d/nacos-logback.xml:/home/nacos/conf/nacos-logback.xml
      - ./init.d/custom.properties:/home/nacos/init.d/custom.properties
    ports:
      - "8848:8848"
    environment:
      - NACOS_SERVER_IP=10.16.10.166
      - NACOS_SERVER_PORT=8848
    env_file:
      - ./env/nacos-ip.env
    restart: on-failure

```


<a name ="1003"></a>  
## hostname模式

> 该部署方式容错率低，配置复杂，了解即可，不推荐使用！，不推荐使用！，不推荐使用！

即集群中各个节点通过hostname进行访问

该方式部署nacos-server集群时，由于nacos中Raft选举Leader时需要相互访问联通，

这样的情况下，多台服务器之前的nacos节点容器配置就相对复杂，倒不如使用ip模式

所以hostname模式，个人认为在单台服务器中配置nacos集群时比较方便快捷

部署前配置方面与ip模式相同，修改和确保`cluster-hostname.yaml`和`nacos-hostname.env`的配置正确



> 如果你想用hostname模式来部署`多个服务器多个节点`，需要进行以下操作

- 确保各个机器间的hosts文件配置正确
- `cluster-hostname.yaml`中配置环境变量`NACOS_SERVER_PORT`为宿主机端口号(发送心跳时使用宿主机端口号)
- 确保`cluster-hostname.yaml`和`nacos-hostname.env`配置正确


<a name ="1004"></a>  
## 单机模式部署
该模式用于部署单机版的nacos，用于开发或测试环境，以及硬件资源不足情况下。

该模式基于mysql持久化

部署前根据不同项目修改`standalone-mysql.yaml`中Mysql的环境变量即可

注：默认采用主从数据库配置，若使用单一数据库，主从配置一致即可，如下：

```yaml
    environment:
      - PREFER_HOST_MODE=ip                       
      - MODE=standalone                             #单机模式
      - SPRING_DATASOURCE_PLATFORM=mysql            #mysql数据源
      - MYSQL_MASTER_SERVICE_HOST=10.16.10.246      #主mysql主机
      - MYSQL_MASTER_SERVICE_DB_NAME=nacos          #主数据库名
      - MYSQL_MASTER_SERVICE_PORT=3306              #端口
      - MYSQL_SLAVE_SERVICE_HOST=10.16.10.246       #从mysql主机
      - MYSQL_SLAVE_SERVICE_PORT=3306               #端口
      - MYSQL_MASTER_SERVICE_USER=root              
      - MYSQL_MASTER_SERVICE_PASSWORD=123456
```




<a name ="1005"></a>  
# 部署

* 克隆项目

  ```powershell
  git clone https://github.com/larscheng/registry-nacos.git
  cd registry-nacos/diydemo
  ```

* 修改配置

  - `nacos-ip.env / nacos-hostname.env`
  - `cluster-ip.yaml / cluster-hostname.yaml`

* ip模式部署

  ```powershell
  docker-compose -f cluster-ip.yaml up -d
  ```


* hostname模式部署

  ```powershell
  docker-compose -f cluster-hostname.yaml up -d
  ```

* 单机模式部署

  ```powershell
  docker-compose -f standalone-mysql.yaml up -d
  ```


