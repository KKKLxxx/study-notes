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

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

如果失败，提示可能如下（表示因网络原因无法访问镜像）

```
docker: Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers).
```

（25年5月更新）目前的网络环境大概率是失败的，需要先按照如下步骤配置镜像

## 7、配置镜像加速

（25年5月更新）实测目前阿里云的镜像也不可用了，可执行如下指令配置镜像源

```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
"registry-mirrors": [
  "https://registry.docker-cn.com",
  "https://s4uv0fem.mirror.aliyuncs.com",
  "https://docker.1ms.run",
  "https://registry.dockermirror.com",
  "https://docker.m.daocloud.io",
  "https://docker.kubesre.xyz",
  "https://docker.mirrors.ustc.edu.cn",
  "https://docker.1panel.live",
  "https://docker.kejilion.pro",
  "https://dockercf.jsdelivr.fyi",
  "https://docker.jsdelivr.fyi",
  "https://dockertest.jsdelivr.fyi",
  "https://hub.littlediary.cn",
  "https://proxy.1panel.live",
  "https://docker.1panelproxy.com",
  "https://image.cloudlayer.icu",
  "https://docker.1panel.top",
  "https://docker.anye.in",
  "https://docker-0.unsee.tech",
  "https://hub.rat.dev",
  "https://hub3.nat.tf",
  "https://docker.1ms.run",
  "https://func.ink",
  "https://a.ussh.net",
  "https://docker.hlmirror.com",
  "https://lispy.org",
  "https://docker.yomansunter.com",
  "https://docker.xuanyuan.me",
  "https://docker.mybacc.com",
  "https://dytt.online",
  "https://docker.xiaogenban1993.com",
  "https://dockerpull.cn",
  "https://docker.zhai.cm",
  "https://dockerhub.websoft9.com",
  "https://dockerpull.pw",
  "https://docker-mirror.aigc2d.com",
  "https://docker.sunzishaokao.com",
  "https://docker.melikeme.cn"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

参考链接：https://blog.csdn.net/qq827245563/article/details/146085734

配置完成后重新执行步骤6即可