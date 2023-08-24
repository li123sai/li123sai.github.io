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

## 二、解决方案A

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

   ```
   hexo clean && hexo g && hexo d
   ```

   

5. 但是后边在写博客部署时出现问题了

   ![报错](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo7.png)

   ```sh
   # 执行hexo clean 也不管用
   hexo clean
   ```

   

6. 删除博客.deploy_git 目录（备注：如果第三部直接删除.deploy_git目录，不知道还会不会出现第五步的问题，所以增加解决方案B）

   ![删除范围](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo6.png)

7. 重新部署就没有问题了

   ```
   hexo clean && hexo g && hexo d
   ```

   

## 三、解决方案B

1. 查看git是否忽略大小写

   ```sh
   # 查看git文件大小写配置：
   git config --get core.ignorecase
   
   # 当结果是true时，忽略大小写，需要设置为false
   git config core.ignorecase false
   
   # 这块是为了备份博客
   ```

   

2. 删除博客.deploy_git 目录

   ![删除范围](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo6.png)

3. 重新部署

   ```sh
   hexo clean && hexo g && hexo d
   ```

   

​	



## 三、说明

- 静态文件问题

因为在本地在执行`hexo g`后，会在博客根目录下生成一个`public`文件夹。

接着执行`hexo d`，就会把这个`public`文件夹的东西完完整整地拷贝到`.deploy_git`文件夹里，并把该文件夹里的所有文件全部推送push到远程库。

之后会触发Pages服务的钩子去build项目，然后部署到网站上。

markdown文章在之前的`hexo g`之后，把生成的静态文件拷贝到了`.deploy_git`文件，但`hexo clean`并没能删除`.deploy_git`里的markdown的静态文件。在解决方案A中只删除博客.deploy_git 目录下除.git目录的其他文件，所以后边在部署的时候，就报错了。

所以同时删掉`.deploy_git`文件夹即可。



- 大小写问题

因为默认忽略大小写所以本地修改大小写后，git识别不到就提交不了，部署的博客还是以原文件名部署，所以会出现访问不到的情况。

![大小写修改未部署成功](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo5.png)

![大小写修改部署成功](https://raw.githubusercontent.com/li123sai/myPictures/main/img/hexo3.png)

所以大小写必须一致。
