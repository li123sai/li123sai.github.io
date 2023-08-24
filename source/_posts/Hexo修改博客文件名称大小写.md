---
title: Hexo修改博客文件名称大小写
date: 2023-08-24 16:02:12
categories:
- 工具
- Hexo
tags:
- Hexo
---

## 一、背景

在使用Hexo部署博客后，发现博客名称大小写没有统一，于是就修改了大小写重新部署，本地访问没有问题，域名访问就不行了。

## 二、解决方案

1. 查看git是否忽略大小写

   ```sh
   # 查看git文件大小写配置：
   git config --get core.ignorecase
   
   # 当结果是true时，忽略大小写，需要设置为false
   git config core.ignorecase false
   
   # 这块是为了备份博客
   ```

   

2. 修改博客.deploy_git 目录下.git目录里的config

   ![文件位置](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo1.png)

   ```sh
   # 将true 改为 false
   ignorecase = false
   
   # 这块是为了部署
   ```

   

3. 删除博客.deploy_git 目录下除.git目录的其他文件

   ![删除文件范围](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo2.png)

4. 重新部署

   ```sh
   hexo clean && hexo g && hexo d
   ```

   

## 三、说明

因为默认忽略大小写所以本地修改大小写后，git识别不到就提交不了，部署的博客还是以原文件名部署，所以会出现访问不到的情况。

![大小写修改未部署成功](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo5.png)

![大小写修改部署成功](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo3.png)

所以大小写必须一致。
