# 说明
详细信息请参见 [lab0 documents](https://edu.n2sys.cn/#/tut_lab/lv0/README)
# 操作
1. 准备Docker
    1. 安装Docker
    2. 编写Dockerfile
        ```dockerfile
        FROM docker.mirrors.sjtug.sjtu.edu.cn/library/ubuntu:22.04
        LABEL maintainer="yuezhu@pku.edu.cn"
        COPY sources.list.x86 /etc/apt/sources.list
        RUN apt update && apt install cmake gcc g++ ninja-build vim openssh-server ca-certificates curl git --yes --force-yes && echo 'root:123456' | chpasswd
        COPY sshd_config.txt /etc/ssh/sshd_config
        ```
    3. 构建镜像
        ```bash
        docker buildx build -t netlab2023:v1 -f .\Dockerfile . --platform linux/amd64
        ```
    4. 启动容器
        ```yml
        services:
            netlab-lab1:
                image: netlab2023:v1
                container_name: netlab-lab0
                volumes:
                - ./workspace:/workspace
                ports:
                - 20729:22
                tty: true
        ```
    5. 配置环境
        1. SSH
            ```bash
            docker exec netlab-lab0 /bin/bash -c "service ssh restart"
            ssh-keygen
            //并复制id_rsa.pub
            
            ssh root@127.0.0.1 -p 20729

            //在.ssh下新建authorized_keys下粘贴
            ```
        2. Dev Container
            > 使用VS Code插件快速打开
2. 编译准备
    ```bash
    mkdir build
    cd build
    cmake .. -G "Ninja"
    ```
3. 修改CMakeLists
    ```cmake
    add_executable(hellonetwork hellonetwork.cpp)
    ```
4. 修改源代码
    > EZ
5. 测试
    ```zsh
    ninja
    hellonetwork 1
    ```
6. 提交
    ```zsh
    git add .
    git commit -am "Update"
    git push
    ```