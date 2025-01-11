## Docker安装ELK日志平台

### 前置条件

- 系统环境：Centos7.6 内核3.10.0以上
- 内存：4G以上
- JDK： jdk1.8以上
- 升级CentOS内核
- 安装docker-ce和docker-compose


### 开启或关闭防火墙
1.关闭防火墙

```shell
systemctl stop firewalld.service      #执行关闭命令
systemctl status firewalld.service    #查看防火墙状态
systemctl disable firewalld.service   #执行开机禁用防火墙自启命令
```

2.开启防火墙
```shell
systemctl start firewalld.service    #启动防火墙
systemctl enable firewalld.service   #防火墙随系统开启启动
```

### docker-compose安装ELK环境

#### 新建目录&修目录权限
```shell
$ mkdir -p /data/elk/{elasticsearch/data,logstash}     #新建目录
#授权目录
$ cd /data/elk
$ chmod 777 elasticsearch/data
```

#### 编写docker-compose配置文件

创建docker-compose.yml文件,在docker-compose.yml文件添加以下内容

```shell
$ vi /data/elk/docker-compose.yml
```

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.7.0  #镜像
    container_name: elk_elasticsearch  #定义容器名称
    restart: always  #开机启动，失败也会一直重启
    environment:
      - "cluster.name=elasticsearch-spring" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx1024m" #设置使用jvm内存大小
    volumes:
      - /data/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /data/elk/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
  kibana:
    image: kibana:7.7.0
    container_name: elk_kibana
    restart: always
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200 #设置访问elasticsearch的地址
    ports:
      - 5601:5601
  logstash:
    image: logstash:7.7.0
    container_name: elk_logstash
    restart: always
    volumes:
      - /data/elk/logstash/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
```

#### 编写logstash配置文件

创建logstash-springboot.conf配置文件并添加以下内容
```conf
input {
  tcp {
    mode => "server"
    host => "0.0.0.0"
    port => 4560
    codec => json_lines
  }
}
output {
  elasticsearch {
    hosts => "elasticsearch:9200"
    index => "boutique-logstash-%{+YYYY.MM.dd}"
  }
}
```

#### docker-compose构建并运行ELK
```shell
$ cd /data/elk
$ docker-compose up -d
$ docker ps
```

#### kibana配置

> 汉化kibana

```shell
$ docker exec -it elk_kibana /bin/bash   # 进入kibanar容器
$ cd config                              # 进入容器的配置文件目录
$ vi kibana.yml                          # 编辑kibana.yml文本 末尾加入i18n.locale: zh-CN
$ docker restart elk_kibana              # 重启kibana
```

> Kibana设置登录密码

kibana默认没有访问的权限控制，如果需要设置访问的账号密码，可以使用nginx配置代理来发布kibana。在 nginx 下，提供了 ngx_http_auth_basic_module 模块实现让用户只有输入正确的用户名密码才允许访问web内容。默认情况下，nginx 已经安装了该模块。所以整体的一个过程就是先用第三方工具设置用户名、密码（其中密码已经加过密），然后保存到文件中，接着在 nginx 配置文件中根据之前事先保存的文件开启访问验证。


#### elasticsearch配置

> 安装ES IK分词器

上面我们安装的是ElasticSearch7.7.0，下载对应的IK分词包 ,elasticsearch挂载的路径是/data/elk/elasticsearch/plugins
1.将下载的ik包解压拷贝到挂载的路径‘/data/elk/elasticsearch/plugins’，并且重命名为ik
2.重启docker容器
```shell
# 重启elasticsearch容器即可
docker restart [elasticsearch容器ID]
# 查看IK是否安装成功
docker exec -it [elasticsearch容器ID] /bin/bash
cd plugins/
ls
```
#### logstash配置
1. logstash中安装json_lines插件并重启logstash
```shell
$ docker exec -it elk_logstash /bin/bash -c  "cd /bin && logstash-plugin install logstash-codec-json_lines"
$ docker restart elk_logstash
```

#### ELK集成kafka日志收集

> [!TIP]
> https://blog.csdn.net/zhang14533/article/details/130325920


#### 问题附录

> max_map_count问题以及解决

1.编辑sysctl.conf文件

```shell
$ vi /etc/sysctl.conf

```
添加代码
```conf
# 添加如下代码
vm.max_map_count=262144
```

2.配置文件生效

```shell
$ sysctl -p #这个命令要加上，否则无效
```
