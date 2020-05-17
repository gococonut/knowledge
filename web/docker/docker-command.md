### 列出所有的容器 ID

docker ps -aq

### 停止所有的容器

docker stop $(docker ps -aq)

### 删除所有的容器

docker rm $(docker ps -aq)

### 删除所有的镜像

docker rmi $(docker images -q)

### 复制文件

docker cp mycontainer:/opt/file.txt /opt/local/
docker cp /opt/local/file.txt mycontainer:/opt/
