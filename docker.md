# docker

* 非root用户使用docker出现权限问题

``` bash
sudo groupadd docker #添加docker用户组
sudo gpasswd -a $USER docker #将当前用户添加至docker用户组
newgrp docker #切换docker用户组 或登出用户
```

* 查看镜像

``` bash
docker images
docker image ls
```

* 拉取镜像

``` bash
docker image pull IMAGE[:VERSION]
```

* 删除镜像

``` bash
docker image rm ID
# 按条件删除
docker image rm $(docker images --filter=reference='name*' -q)
```

* 运行镜像

> docker run -itd [--env-file env-file] [--name 容器名称] 镜像名称
```sh
docker run 
  -u root \
  -d \
  -p 80:3000 \ #port 映射端口
  --name name \    
  --restart=always \
  -v /var/run/docker.sock:/var/run/docker.sock \  #volume 挂载目录
  -v /data:/data \
  imageName
```

* 查看容器

> docker ps

* 进入容器

> docker exec -it Id bash

* 删除容器

``` bash
docker rm [-f] Id 
#删除所有
docker rm -f $(docker ps -aq)
```

* 按dockerfile生成镜像

> docker build -t name:tag [-f dockerfile] .
