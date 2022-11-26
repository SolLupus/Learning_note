# Docker镜像制作学习笔记

​	

* 镜像：image[用户昵称]/镜像名称:tag
* 容器:  container

## pull

> 针对镜像

``` 
docker pull php:8.0.22RC1-apache-buster
```

## run

```
docker run -it -d -p 13579:80 -e FLAG=flag{1234564564-4579-4534-4534} id/name
```

- -it 以交互终端形式运行
- -d 以后台形式运行
- -p 端口映射 （宿主机端口:镜像内部端口）
- -e 环境变量
- lxxxin/qwb_2022_rcefile 镜像

## exec

``` 
docker exec -it bugd446576 /bin/bash
```

- -it 以交互式终端运行
- bugd446576 是容器的id
- /bin/bash 是容器内部的bash /bin/sh

## build

> 做一个镜像

- 首先要进入到含有Dockerfile文件的目录(images命名:小写字母，数字，下划线，短横杠)

  ```
  docker build -t image:tag --platform=amd64 .
  ```

- -t 设置镜像tag，sollupus/xxxxx:latest 镜像名称

​	制作完成以后

​	docker images可以看到自己做的镜像

## push

 确保自己docker hub登录：

- docker login登录

```
docker push id /image:tag
```

## 注意点

- platform,架构要清楚

## 做镜像(Dockerfile)

> 动态flag的特性

- 确保容器不会被退出，使用`tail -f /dev/null`
- EXPOSE 80 镜像内部开放端口
- sed -i "s#FLAGFLAGFLAGFLAG#" /var/www/html/flag.php || ture 设置动态flag
- chmod +x /flag.sh 给flag.sh 和start.sh 加权限



