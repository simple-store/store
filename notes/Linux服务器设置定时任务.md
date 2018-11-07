### Linux服务器相关

- ssh链接

  > mac上连接服务器最便捷的方式无疑是ssh连接了

  - 打开cmd窗口运行

  ```cmd
  sudo ssh username@ip -p port
  ```

  1.提示你输入本机密码

  `Password:`

  2.提示ECDSA key

  ```cmd
  ECDSA key fingerprint is SHA256:CjKMbELB5VvJeM6KeBmthGoi6C2QjbfbmEYlyT8Zew8.
  
  Are you sure you want to continue connecting (yes/no)? yes
  
  ```

  3.输入远程连接密码

  `xxx@xx.x.xx.xxx's password:`

  4.成功连接

  `Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.13.0-43-generic x86_64)`



- 相关命令
  - 查看系统信息: cat /proc/version
  - 创建文件夹: mkdir xxx
  - 创建文件: touch xx.xx(test.js)
  - 删除文件夹： rm -r xxx
  - 强制删除文件夹: rm -r -f xxx
  - 进入文件夹： cd xxx
  - 查看当前路径信息: pwd
  - 查看当前路径下目录信息： ls









- 安装docker

  - sudo apt install docker.io 安装

  - docker version 查看当前版本

  - ```
    docker image ls 查看所有image
    ```

  - ```
    docker image rm [imageName] 删除image
    ```

  - ```
    docker container run -p 8000:3000 -it koa-demo /bin/bash  启动一个image
    ```

  - ```
    exit / ctrl + c 退出容器
    ```

  - 