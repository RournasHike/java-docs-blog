## Docker安装与配置

### 前置工作

> 1.检查操作系统内核版本

这里使用的是CentOS7 Linux系统，系统内核版本必须大于3.10
```shell
uname -r
```

> 2.安装好jdk

推荐安装jdk8以上

### Docker安装

> 移除旧版本docker

```shell
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

> 安装系统工具

```shell
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```

> 添加软件源信息

```shell
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```


> 更新 yum 缓存

```shell
# centos 7
sudo yum makecache fast
```

> 安装 Docker-ce

安装docker-ce版本
```shell
sudo yum -y install docker-ce
```

查看已安装docker版本

```shell
docker version
```

> 启动命令

启动 Docker 后台服务

```shell
sudo systemctl start docker
```

开机启动

```shell
sudo systemctl enable docker
```

> 镜像加速

请在`/etc/docker/daemon.json`该配置文件中加入`registry-mirrors`：

```json
{
  "registry-mirrors": ["https://docker.1panel.live"]
}
```

> [!NOTE]
> 其他镜像地址

```json
{
    "registry-mirrors": [
            "https://docker.211678.top",
            "https://docker.1panel.live",
            "https://hub.rat.dev",
            "https://docker.m.daocloud.io",
            "https://do.nark.eu.org",
            "https://dockerpull.com",
            "https://dockerproxy.cn",
            "https://docker.awsl9527.cn"
      ]
}
```

```json
{
"registry-mirrors": ["https://docker.m.daocloud.io"]
}
 
```


重新加载
```shell
systemctl daemon-reload
```

重启docker服务
```shell
systemctl restart docker
```

### Docker-Compose安装

安装到全局
```shell
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-`uname -s`-`uname -m` > docker-compose
如果上面下载很慢可以用一下命令找到适合本系统的docker-compose url
echo https://github.com/docker/compose/releases/latest/download/docker-compose-`uname -s`-`uname -m`
将输出结果在浏览器打开，这样就会自动下载命名为docker-compose再上传到服务器上

sudo mv docker-compose /usr/libexec/docker/cli-plugins
sudo chmod +x /usr/libexec/docker/cli-plugins/docker-compose
sudo chown root:root /usr/libexec/docker/cli-plugins/docker-compose
sudo mv docker-compose /usr/local/bin/
```