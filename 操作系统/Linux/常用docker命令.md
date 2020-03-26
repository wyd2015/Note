---
title: '常用docker命令'
date: 2019-11-30
categories: 'Linux'
---
# 镜像
1. 查看镜像：`docker images`
    ```bash
    # 查看镜像
    [root@zabbix ~]# docker images
    REPOSITORY       TAG     IMAGE ID            CREATED             SIZE
    ckm_similarity   8.0     68c27838ddb1        23 hours ago        5.16 GB

    # 查看所有的镜像（包含历史镜像）
    docker images -a
    ```
2. 搜索镜像：
   ```sh
   docker search image
   ```
3. 创建镜像：
    ```sh
    # 利用Dockerfile创建镜像
    docker build -t repository:tag Dockerfile_path
    ```
4. 提交修改后的镜像：
    ```sh
    docker commit 
    -m "commit msg"
    -a "author info"
    container_id
    repository:tag
    ```
5. 上传镜像：
   ```sh
    docker push repository[:tag]
   ```
6.  导出镜像到本地：  
    ```sh
    # 导出镜像的快照文件，此操作会丢弃所有的历史记录和元数据（配置）信息
    docker export container_id > export_file_name.tar

    # 使用save保存镜像到本地时可保证元数据信息不会丢失
    docker save -o /home/wcg/ repository:tag
    ```
7. 导入镜像到本地镜像库
   ```sh
    docker load --input image_file_name
    docker load < image_file_name
   ```
8. 移除镜像
   ```sh
    docker rmi repository
   ```

# 容器
1. 查看正在运行的容器：
   ```sh
   docker ps
   ```
2. 进入容器：
   ```sh
   #连接一个正在运行的容器实例
   docker attach container_id

   # 进入后可在容器内执行命令
   docker exec -ti container_id /bin/bash
   ```
3. 查看处于终止状态的容器：
   ```sh
   docker ps -a
   ```
4. 启动一个新建容器：
   ```sh
   docker run -ti --name container_name -p 8000:8000 repository:tag /bin/bash
   ```

   使用docker run来创建并启动容器时，后台运行的标准流程如下：
   - 检查本地是否存在制定的镜像，不存在就从共有仓库下载；
   - 利用镜像创建并启动一个容器；
   - 分配一个文件系统，并在只读的镜像层外面挂载一层可读可写层；
   - 从宿主机配置的网桥接口中桥接一个虚拟接口到容器；
   - 从地址池配置一个ip地址给容器；
   - 执行用户指定的应用程序；
   - 执行完毕后容器被终止。


5. 启动一个已经存在的容器：
   ```sh
   docker start container_id
   ```
6. 终止一个容器：
   ```sh
   docker stop container_id
   ```
7. 重启一个正在运行的容器：
   ```sh
   docker restart container_id
   ```
8. 删除正在运行的容器：
   ```sh
   docker rmf container_id
   ```
9. 删除已经终止的容器：
   ```sh
   docekr rm container_id
   ```
10. 导出容器
    ```sh
    docker export container_id > export_file_name.tar
    ```
11. 导入容器为镜像
    ```sh
    cat export_file_name.tar | docker import - repository:tag
    ```
12. 提交修改后的容器
    ```sh
    # 将一个容器固化为一个新的镜像
    docker commit <container_id> [repository:tag]
    ```

# 仓库
1. 把本地标记的镜像推送到仓库
   ```sh
   docker push 192.168.1.182:5000/ckm
   ```