笔记：docker 命令，安装 mysql

# 安装
关闭防火墙
```bash
$ systemctl stop firewalld
$ setenforce 0
```
如果没有  yum-utils
```bash
$ yum install -y yum-utils
```
换安装的 docker-ce 源
```bash
$ yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
```
安装
```bash
yum install -y docker-ce docker-ce-cli containerd.io
```
# Docker 命令
查看版本
```bash
$ docker -v
```
启动 docker 才会创建 `/etc/docker/` 目录
```bash
$ systemctl  start docker
$ vim /etc/docker/daemon.json
```
写入仓库镜像后重启
```json
{
    "registry-mirrors": ["https://mirror.ccs.tencentyun.com"]
}
```
```bash
$ systemctl restart docker
```
docker 三层帮助
```bash
$ docker --help
$ docker container --help
$ docker container rm --help
```
# MySQL
拉取 mysql 镜像, 默认 @latest
```bash
$ docker pull mysql
```
查看当前 docker 的所有镜像
```bash
$ docker images
```
启动mysql镜像创建容器， 名称 some-mysql  -d 表示背景运行    tag是版本，最新填 latest
```bash
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```
# volume
```bash
# 创建volume：
docker volume create【volume名称】

# 查看volume参数：
docker volume inspect【volume名称】

# 使用volume：
# -v 宿主机本地磁盘路径：容器内路径
docker run -dit --name 【容器名称】 -v【volume名称】:/volume 【镜像名称】
# 例如：
docker run -d -p 3306:3306 --name mysql_test -e MYSQL_ROOT_PASSWORD=root -v volume-mysql:/var/lib/mysql mysql:5.7

#查看docker数据卷
docker volume ls

# 删除volume：Remove one or more volumes
docker volume rm 【volume名称】

# 删除没使用的数据卷(谨慎使用)：Remove all unused local volumes
docker volume prune
```
创建 docker 卷  
列出存在的 docker 卷  
查看特定的 卷
运行指定卷挂载到container中的地址，并运行 mysql
```bash
$ docker volume create --name portal_mysql
$ docker volume ls
$ docker volume inspect data_volume
$ docker run -v portal_mysql:/data mysql
```
```json
{
    "CreatedAt": "2023-01-28T22:30:53+08:00",
    "Driver": "local",
    "Labels": {},
    "Mountpoint": "/var/snap/docker/common/var-lib-docker/volumes/portal_mysql/_data",
    "Name": "portal_mysql",
    "Options": {},
    "Scope": "local"
}
```

-- 备用的 -- 
```bash
$ docker run --name mysql -v portal_mysql:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```
查看 当前 docker 正在运行的容器,  注意 container_id
```bash
$ docker ps === docker container ls
```
查看某个容器状态 ，id不需要输全 ， 注意 IPAddress， Ports
```bash
$ docker inspect  container_id  
```

进入 mysql 终端
```bash
docker exec -it 【容器id】 bash
```

#输入命令,并按提示输入密码root
```bash
mysql -u root -p
```
# DBeaver
安装 mysql 查看工具 DBeaver
```bash
$ docker pull dbeaver/cloudbeaver:latest
```
启动 DBeaver 镜像, -d 背景运行， -rm 唯一实例  -p 本地端口:容器端口   -v 本地工作空间:容器工作空间映射
```bash
$ docker run -d --name cloudbeaver --rm -ti -p 8088:8978 -v /var/cloudbeaver/workspace:/opt/cloudbeaver/workspace dbeaver/cloudbeaver:latest
```
useSSL =  false 
allow public key = true  

删除某个容器
```bash
$ docker container rm c3* -f
```