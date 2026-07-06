# 零、业务相关

```
# 查看正在运行的容器
docker ps

# 查看所有的容器
docker ps -a

# 查看所有的镜像
docker images

# 运行容器 挂载 命名 镜像 初始路径
docker run --privileged -dit --name megatron-test -v pip-build-deps:/home/codespace/pip-build-deps --shm-size=128g -v deb-upgrade:/home/codespace/deb-upgrade -v /sys:/sys -v /dev:/dev -v /lib/modules:/lib/modules -v model_home:/data/model_home -v pip-venv:/data/pip-venv -v /login_home/wx_liwei:/login_home/wx_liwei  hub.tsingmicro.com/tx8/ubuntu/daily:tsingmicro_release_tsm8.``260603221224` `/bin/bash

# 将容器提交为镜像（记得加 --no-pause 避免暂停卡住）
docker commit --no-pause <容器名或ID> <新镜像名>:<标签>

# 将镜像导出为 .tar 文件
docker save -o <文件名>.tar <镜像名>:<标签>

# 从 .tar 文件加载镜像
docker load -i <文件名>.tar

# 检查所有正在运行的容器的挂载情况
docker inspect $(docker ps -q) --format=``'{{printf "\n=== %s ===" .Name}}{{range .Mounts}}{{printf "\n%-8s %-40s -> %s" .Type .Source .Destination}}{{end}}'
```
# 一、容器生命周期管理


|     |     |
| --- | --- |
|     |     |

|**运行容器**|`docker run -d --name mynginx nginx`|`-d` 后台运行，`--name` 指定名称|
|运行并交互|`docker run -it ubuntu /bin/bash`|`-it` 交互式终端|
|运行并映射端口|`docker run -d -p 8080:80 nginx`|宿主机 8080 → 容器 80|
|**启动容器**|`docker start <容器ID/名称>`|启动已停止的容器|
|**停止容器**|`docker stop <容器ID/名称>`|优雅停止（发送 SIGTERM）|
|**强制停止**|`docker kill <容器ID/名称>`|立即停止（发送 SIGKILL）|
|**重启容器**|`docker restart <容器ID/名称>`|重启|
|**暂停/恢复**|`docker pause / unpause <容器ID/名称>`|暂停/恢复容器进程|
|**删除容器**|`docker rm <容器ID/名称>`|删除已停止的容器|
|强制删除|`docker rm -f <容器ID/名称>`|强制删除运行中的容器|
|删除所有停止的容器|`docker container prune`|清理|

  

---

# 二、容器查看与监控

|**列出运行中的容器**|`docker ps`|只显示运行中的|
|**列出所有容器**|`docker ps -a`|包含已停止的|
|列出所有容器 ID|`docker ps -aq`|配合其他命令使用|
|查看最后创建的容器|`docker ps -l`|最后一个|
|**查看容器日志**|`docker logs <容器ID/名称>`|查看全部日志|
|实时跟踪日志|`docker logs -f <容器ID/名称>`|类似 `tail -f`|
|查看最近 100 行|`docker logs --tail 100 <容器ID/名称>`|只显示尾部|
|查看容器内进程|`docker top <容器ID/名称>`|类似 `ps`|
|**查看容器资源使用**|`docker stats`|实时 CPU/内存/网络|
|查看容器详细信息|`docker inspect <容器ID/名称>`|JSON 格式全部配置|
|查看容器挂载|`docker inspect --format='{{json .Mounts}}' <容器ID>`|格式化查看挂载|

---

# 三、进入容器与执行命令

|**进入容器（推荐）**|`docker exec -it <容器ID/名称> /bin/bash`|进入已有容器|
|进入容器（sh）|`docker exec -it <容器ID/名称> /bin/sh`|Alpine 等轻量镜像用 sh|
|**在容器内执行命令**|`docker exec <容器ID> ls -la /app`|不进入，直接执行|
|以特定用户执行|`docker exec -it -u root <容器ID> /bin/bash`|以 root 身份进入|
|旧式进入方式|`docker attach <容器ID/名称>`|不推荐，会阻塞|
|容器内复制文件到宿主机|`docker cp <容器ID>:/app/log.txt ./`|从容器复制出来|
|宿主机复制文件到容器|`docker cp ./file.txt <容器ID>:/app/`|复制进去|

---

# 四、镜像操作

|**列出镜像**|`docker images` 或 `docker image ls`|列出本地所有镜像|
|**拉取镜像**|`docker pull nginx:latest`|从仓库下载|
|**构建镜像**|`docker build -t myapp:v1 .`|基于 Dockerfile 构建|
|构建时指定文件|`docker build -f Dockerfile.prod -t myapp .`|指定 Dockerfile|
|**删除镜像**|`docker rmi <镜像ID/名称>`|删除镜像|
|强制删除|`docker rmi -f <镜像ID>`|强制删除|
|删除所有未使用镜像|`docker image prune`|清理|
|删除所有未使用镜像（强制）|`docker image prune -a -f`|全部清理|
|**给镜像打标签**|`docker tag myapp:v1 myregistry/myapp:v1`|为推送做准备|
|**推送镜像**|`docker push myregistry/myapp:v1`|推送到仓库|
|查看镜像历史|`docker history <镜像名>`|查看构建层历史|
|导出镜像|`docker save -o myapp.tar myapp:v1`|导出为 tar 文件|
|导入镜像|`docker load -i myapp.tar`|从 tar 文件导入|

---

# 五、卷（Volume）操作

|**列出卷**|`docker volume ls`|列出所有 volume|
|**创建卷**|`docker volume create mydata`|创建命名卷|
|**查看卷详情**|`docker volume inspect mydata`|查看挂载位置等|
|**删除卷**|`docker volume rm mydata`|删除指定卷|
|删除未使用卷|`docker volume prune`|清理无用卷|
|查看卷占用|`docker system df`|查看磁盘占用|
|带卷运行容器|`docker run -v [mydata:/app/data](http://mydata/app/data) nginx`|挂载 volume|
|Bind mount|`docker run -v /host/[path:/container/path](http://path/container/path) nginx`|挂载宿主机目录|

---

# 六、网络操作

|**列出网络**|`docker network ls`|列出所有网络|
|**创建网络**|`docker network create mynet`|创建桥接网络|
|查看网络详情|`docker network inspect mynet`|查看配置|
|连接容器到网络|`docker network connect mynet <容器ID>`|将容器加入网络|
|断开容器网络|`docker network disconnect mynet <容器ID>`|从网络移除|
|删除网络|`docker network rm mynet`|删除网络（需无容器使用）|
|启动时指定网络|`docker run --network mynet nginx`|加入指定网络|

---

# 七、清理与系统维护

|**查看磁盘使用**|`docker system df`|查看镜像/容器/卷占用|
|**一键清理**|`docker system prune`|删除停止的容器、未使用的网络、悬空镜像|
|完整清理|`docker system prune -a -f`|包含所有未使用镜像|
|清理镜像|`docker image prune -a`|只清理镜像|
|清理容器|`docker container prune`|只清理停止的容器|
|清理卷|`docker volume prune`|只清理未使用的卷|
|清理网络|`docker network prune`|只清理未使用的网络|

---

# 八、常用组合命令（实用技巧）

|停止所有容器|`docker stop $(docker ps -q)`|
|删除所有容器|`docker rm $(docker ps -aq)`|
|删除所有镜像|`docker rmi $(docker images -q)`|
|停止并删除所有|`docker rm -f $(docker ps -aq)`|
|查看所有容器挂载|`for c in $(docker ps -q); do docker inspect $c --format='{{.Name}} {{.Mounts}}'; done`|
|查看容器 IP|`docker inspect <容器ID> \| grep IPAddress`|
|进入容器并自动 bash|`docker exec -it $(docker ps -ql) /bin/bash`（进入最新容器）|
|查看端口映射|`docker port <容器ID>`|
|查看容器重启策略|`docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' <容器ID>`|
|修改容器重启策略|`docker update --restart=always <容器ID>`|

---

# 九、常用参数速记

|`-d`|后台运行|`docker run -d nginx`|
|`-it`|交互式 + TTY|`docker run -it ubuntu bash`|
|`-p`|端口映射|`-p 8080:80`|
|`-P`|随机端口映射|`-P`|
|`-v`|挂载卷/目录|`-v /[data:/app/data](http://data/app/data)`|
|`--mount`|更灵活的挂载|`--mount type=bind,src=/host,dst=/container`|
|`--name`|容器命名|`--name myapp`|
|`--restart`|重启策略|`--restart=always`|
|`-e`|环境变量|`-e ENV=production`|
|`--env-file`|从文件加载环境变量|`--env-file .env`|
|`--network`|指定网络|`--network mynet`|
|`--rm`|退出自动删除|`docker run --rm`|
|`-u`|指定用户|`-u root`|
|`-w`|指定工作目录|`-w /app`|

---

# 十、常用重启策略

|`no`|不自动重启（默认）|
|`always`|总是重启|
|`unless-stopped`|总是重启，除非手动停止|
|`on-failure`|仅在异常退出时重启|