# Docker安装RocketMQ

### RocketMQ简介

RocketMQ 是一个分布式消息中间件，最初由阿里巴巴集团开发，并于2012年开源。自2016年起，RocketMQ成为了Apache软件基金会的顶级项目。它设计用于支持大规模分布式系统中的可靠消息传递，具备高吞吐量、低延迟以及高可用性的特点，适用于诸如日志收集、监控数据聚合、流处理等多种场景。

### 创建文件

```shell
$ mkdir -p /data/rocketmq/conf
# 配置 Broker 的IP地址 192.168.230.83-> 宿主机IP
$ echo "brokerIP1=192.168.230.83" > broker.conf
```



### 拉取镜像

```shell
$ docker pull apache/rocketmq:5.3.1
$ docker pull apacherocketmq/rocketmq-dashboard:latest
```

### 创建容器网络

```shell
$ docker network create rocketmq
```



### 启动nameserve

```shell
# 启动 NameServer
docker run -d --name rmqnamesrv -p 9876:9876 --network rocketmq apache/rocketmq:5.3.1 sh mqnamesrv

# 验证 NameServer 是否启动成功
docker logs -f rmqnamesrv
```



### 启动Broker+Proxy

```shell
$ docker run -d \
--name rmqbroker \
--network rocketmq \
-p 10912:10912 -p 10911:10911 -p 10909:10909 \
-p 8086:8080 -p 8087:8081 \
-e "NAMESRV_ADDR=rmqnamesrv:9876" \
-v ./conf/broker.conf:/home/rocketmq/rocketmq-5.3.1/conf/broker.conf \
apache/rocketmq:5.3.1 sh mqbroker --enable-proxy \
-c /home/rocketmq/rocketmq-5.3.1/conf/broker.conf

# 验证 Broker 是否启动成功
$ docker exec -it rmqbroker bash -c "tail -n 10 /home/rocketmq/logs/rocketmqlogs/proxy.log"
```



### 启动dashboard

```shell
$ docker run -d --name rocketmq-dashboard --network rocketmq -e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.230.83:9876" -p 9999:8080 -t apacherocketmq/rocketmq-dashboard:latest
```

> [!NOTE]
> 这个需要和rocketmq namesrv和broker相同的容器网络，它们之间才可以互相连通，否则dashboard连接不上broker和nameserve
>
> 

### 访问测试

http://192.168.230.83:9999