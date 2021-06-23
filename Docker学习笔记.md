# Docker学习笔记



## 1. Docker基本组成

**镜像(Image)：**

docker镜像类似于一个Class，通过镜像来创建容器，同一个镜像可以创建多个容器。

**容器(container)：**

通过镜像创建，启动、停止、删除命令。

**仓库：**

存储docker镜像。



## 2. Docker常用命令

### 2.1 帮助命令

```shell
docker version		# 显示docker版本信息
docker info 			# 显示docker系统信息，包括镜像和容器数量
```

### 2.2 镜像命令

```shell
docker images		# 查看本地所有镜像
docker search 镜像名           # 搜索镜像
docker pull 镜像名[:tag]      # 下载镜像   --不写tag 则默认latest
docker rmi 容器ID / 镜像名             # 删除指定容器  
                           -f                                     # 强制删除
```

### 2.3 容器命令

```shell
docker run [可选参数] image      # 通过镜像 启动容器
# 参数说明
--name="Name"                              # 容器名字  用于区分容器
-d                                                            # 后台方式运行
-it                                                            # 使用交互方式运行，进入到容器内部查看内容  
                          exit                               # 从交互容器中退出，并停止容器
                          Ctrl+P+Q                    # 从交互容器中退出，不停止容器
-p                                                            # 指定容器端口    -p 8080:8080
                          -p ip : 主机端口 : 容器端口
                          -p 主机端口 : 容器端口
                          -p 容器端口
                          容器端口
-P                                                            # 大写P，随机指定端口

docker ps                                            # 列出所有运行中的容器
                           -a                                 # 列出当前+历史运行过的容器
                           -n=?                            # 列出最近运行过的n个容器

docker rm 容器ID                             # 根据Id删除容器， 不能删除正在运行的容器
                           -f                                   # 强制删除容器，包括正在运行的容器
docker rm -f $(docker ps -aq)     # 递归删除所有容器

docker start 容器ID                          # 启动一个停止的容器
docker restart 容器ID                      # 重启容器
docker stop 容器ID                           # 停止容器
docker kill 容器ID                              # 强制停止(杀掉)容器
```

### 2.4 常用其他命令

```shell
# 以后台方式运行一个centos系统  (必须要有一个进程在运行，不然docker自动结束后台容器)
docker run -d centos /bin/sh -c "while true;do echo jinyigao;sleep 1;done"

# 输出日志
docker logs  容器ID                           
                          -tf                                    # 显示日志
                          --tail n                           # 要显示的日志条数
                          
# 查看容器内进程信息                         
docker top 容器ID

# 查看容器的元数据
docker inspect 容器ID

# 进入当前正在运行的容器



```

