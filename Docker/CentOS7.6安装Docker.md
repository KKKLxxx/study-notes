# CentOS7.6安装Docker

参考官网：https://docs.docker.com/engine/install/centos/

## 1、卸载旧版本

```
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

保险起见，不管之前有没有安装过，执行一下总不会出错

## 2、安装`yum-utils`包（提供`yum-config-manager` 实用程序）

```
sudo yum install -y yum-utils
```

## 3、设置`stable`存储库

```
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

注意换成阿里云镜像

## 4、安装Docker引擎

```
sudo yum -y install docker-ce docker-ce-cli containerd.io
```

## 5、启动Docker

```
sudo systemctl start docker
```

## 6、测试

```
sudo docker version
sudo docker run hello-world
```

如果启动成功，会显示对应的消息

## 7、配置镜像加速

首先打开阿里云《容器镜像服务 ACR》的控制台，找到镜像工具->镜像加速器，下面有一组指令，直接复制粘贴运行即可

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

每个人的地址都不一样，这里我隐藏了自己的地址