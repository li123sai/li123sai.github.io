---
title: Mac优雅使用easyconnect
date: 2024-01-27 17:53:08
categories:
- 工具
- Mac
tags:
- easyconnect
---

## EasyConnect的魔法

------

​	深信服的easyconnect应该都用过，方便连接内网，但是它的一些常驻进程就有点意思了。例如：两个以root权限的常驻进程和Sangfor根证书。

- EasyMonitor和ECAgentProxy后台常驻，手动关闭后还会拉起

- 自动安装系统根证书，即使删除了，只要上边两个进程在还是会重新安装

  有了这样的加持，就不知道能发出什么魔法了。

  

  | 名称     | 环境               |
  | -------- | ------------------ |
  | 操作系统 | macOS Ventura 13.4 |
  | CPU      | Apple M2           |
  | 终端     | zsh (oh-my-zsh)    |
  | 终端工具 | iTerm2             |

  

  ## 方案一（Docker魔法对魔法）

  ------

   对魔法深入研究的大佬搞了Docker镜像来对抗魔法的影响。

  [docker-easyconnect](https://github.com/docker-easyconnect/docker-easyconnect ) 项目目的很明确：

  将深信服的EasyConnect和aTrust运行在docker或者podman中。

  项目提供了纯命令行版和图形桌面版。尽量本地安装使用socks5代理，避免使用云服务产生的安全性问题。

  ```sh
  docker run --rm --device /dev/net/tun --cap-add NET_ADMIN -ti -p 127.0.0.1:1080:1080 -p 127.0.0.1:8888:8888 -e EC_VER=7.6.3 -e CLI_OPTS="-d vpnaddress -u username -p password" hagb/docker-easyconnect:cli
  ```
  
  

  | 参数                                                | 说明                                                         |
  | --------------------------------------------------- | ------------------------------------------------------------ |
  | --rm                                                | 容器停止后自动删除容器。这个选项可以确保每次运行容器时都是全新的状态。 |
  | --device /dev/net/tun                               | 将 `/dev/net/tun` 设备挂载到容器中，以便容器可以使用 TUN/TAP 设备。 |
  | --cap-add NET_ADMIN                                 | 为容器添加 `NET_ADMIN` 权限，以便容器可以进行网络配置。      |
  | -ti                                                 | 分配一个伪终端 (pseudo-TTY) 并保持 STDIN 打开，以便在容器内交互。 |
  | -p 127.0.0.1:1080:1080                              | 将主机的 127.0.0.1 地址的 1080 端口映射到容器内的 1080 端口。这样可以通过主机的 1080 端口访问容器内的服务。 |
  | -p 127.0.0.1:8888:8888                              | 将主机的 127.0.0.1 地址的 8888 端口映射到容器内的 8888 端口。同样地，这样可以通过主机的 8888 端口访问容器内的服务。 |
  | -e EC_VER=7.6.3                                     | 设置一个名为 EC_VER 的环境变量，并将其值设置为 7.6.3。表示使用 `7.6.7` 版本的 EasyConnect，请根据实际情况修改版本号。  `7.6.3`：适用于连接 <7.6.7 版本的 EasyConnect 服务端。 `7.6.7`：适用于连接 >= 7.6.7 版本的 EasyConnect 服务端。 `cli`：来源于 [@shmille](https://github.com/shmilee) 提供的[命令行版客户端 deb 包](https://github.com/shmilee/scripts/releases/download/v0.0.1/easyconn_7.6.8.2-ubuntu_amd64.deb)。适用于所有版本的 EasyConnect 服务端（需配合环境变量参数 `-e EC_VER=7.6.3` 或 `-e EC_VER=7.6.7`），但只有 amd64 版本，只能使用用户名、密码来登录。 |
  | -e CLI_OPTS="-d vpnaddress -u username -p password" | 设置一个名为 CLI_OPTS 的环境变量，并将其值设置为 "-d vpnaddress -u username -p password"。这个环境变量可能是用于传递命令行参数给容器中运行的程序。     vpnaddress：vpn地址，username：用户名【支持中文】，password：密码。 |
  
  

  执行完毕后，容器会在前台运行，关闭后容器自动销毁数据。

  
  
  ## 封装脚本
  
  觉得命令行麻烦可以封装为脚本执行
  
  1. 新建一个 `~/myfile/myshell/connvpn.sh` 并输入以下内容
  
     ```shell
     #!/bin/bash
     
     echo "Start Docker EasyConnect..."
     username="用户名"
     vpnaddr="VPN 连接地址"
     echo "Please enter $username password:"
     read -s password
     docker run --rm --device /dev/net/tun --cap-add NET_ADMIN -ti -p 127.0.0.1:1080:1080 -p 127.0.0.1:8888:8888 -e EC_VER=7.6.3 -e CLI_OPTS="-d $vpnaddr -u $username -p $password" hagb/docker-easyconnect:cli
     ```
  
     
  
  2. 编辑`~/.zshrc`文件，末尾加入
  
     ```shell
     alias connvpn="/bin/sh ~/myfile/myshell/connvpn.sh"
     ```
  
     
  
  3. 之后使配置生效`source ~/.zshrc`，此时，尝试执行`connvpn`，会自动启动一个 Docker 容器，并自动登录相关账号
  
     
  
  ## 配置代理
  
  上边已经暴漏一个socks5代理的1080端口，接下来配置并使暴露出来的代理。
  
  ### 浏览器配置代理
  
  1. 浏览器使用代理
  
     使用使用 [SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega) 插件实现，可自行安装该插件，安装完毕后，「新建情景模式」。
  
     ![新建情景模式](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy.png)
  
  2. 配置代理服务器
  
     ![配置](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy1.png)
  
  3. 使用
  
     ![使用](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy2.png)
  
     就可以访问vpn地址了
  
  
  
  ### SSH代理
  
  1. 直接使用ssh命令
  
     ```sh
     ssh -o ProxyCommand="nc -X 5 -x 127.0.0.1:1080 %h %p" root@xxx.xxx.xxx.xxx
     ```
  
     
  
  2. 使用SSH config的方式管理，编辑~/.ssh/config
  
     ```sh
     # 示例配置
     Host myeasyconnect
         HostName 192.168.111.222
         User root
         Port 60022
         IdentityFile ~/.ssh/id_ed25519
         ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p
        
      # 使用
      ssh myeasyconnect
     ```
  
  ### Jetbrains代理
  
  配置vpn拉取或推送代码
  
  ![ide配置](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy3.png)
  
  
  
  
  
  ## 方案二（windows虚拟机安装）
  
  1. 安装Parallels Desktop
  
     ![安装](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy4.png)
  
  2. 在windows虚拟机中使用easyconnect
  
     ![使用ccproxy](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy5.png)
  
  3. 在浏览器中使用
  
     ![浏览器使用](https://raw.githubusercontent.com/li123sai/myPictures/main/img/easy6.png)
  
  4. SSH使用
  
     ```sh
     export https_proxy=http://10.211.55.3:808 http_proxy=http://10.211.55.3:808 all_proxy=socks5://10.211.55.3:808
     ```
  
     
  
  

****
