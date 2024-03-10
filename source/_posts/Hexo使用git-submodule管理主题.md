---
title: Hexo使用git submodule管理主题
date: 2024-03-10 22:02:43
categories:
- 工具
- Hexo
tags:
- Hexo
---

## 一、背景

使用hexo时可以选择很多主题，之前都是将主题复制到theme文件夹中直接使用，一次查看自己博客时发现一个主题的bug，去主题仓库发现大佬已经修改了，于是进行主题升级，这时就麻烦了。

1. 主题基本都是一个独立的项目，将主题直接复制到项目中是没有原本的版本控制功能。
2. 使用的主题如果自己个性化处理了，升级时需要将自己的修改在重复搞一遍。



## 二、Git submodules的作用

Git submodules 可以叫git的子模块，子模块就是将一个git仓库作为另一个git仓库的子目录。可以将一个仓库克隆到自己项目中，同时还保持提交更新的独立。

### 基础用法

使用Git submodules的方式，将一个主题作为子项目。

## 三、Git submodules的使用

### fork别人主题并保持同步更新

```shell
# 将fork的项目克隆到本地仓库中
git clone https://github.com/li123sai/hexo-theme-3-hexo.git

# 给fork的项目增加一个远程地址（upstream是远程仓库的别名，可以更还）
git remote add upstream https://github.com/yelog/hexo-theme-3-hexo

# 查看状态（下边有两个upstream说明第二部成功了）
git remote -v

# 从设置的远程仓库更新
git fetch upstream

# 查看远程仓库更新（Q退出）
git log upstream/master

# 将 upstream 远程仓库的更新合并到当前的本地分支中
git merge upstream/master

# 更新到自己fork的仓库中
git push origin master
```

![fork操作图示](https://raw.githubusercontent.com/li123sai/myPictures/main/img/submodule1.png)

### 将fork的代码作为子项目

#### 首次添加子模块

```shell
# 将父项目克隆到本地
git clone https://github.com/li123sai/li123sai.github.io.git

# 进入父项目目录，添加子模块
git submodule add https://github.com/li123sai/hexo-theme-3-hexo.git

# 查看添加子模块的状态
git submodule

# 提交和推送父项目变更
git status
git add .
git commit -m 'new file'
git push
```



#### 拉取包含子模块的父项目

```shell
# 克隆父项目
git clone https://github.com/li123sai/li123sai.github.io.git
# 查看子模块状态（这时子模块文件夹式空的，又一个-号，说明没有初始化）
git submodule
# 初始化子模块
git submodule init
# 更新子模块
git submodule update
# 查看子模块状态
git submodule
```



#### 父项目中子模块更新

```shell
# 初始化版本
git submodule update --init
# 更新子模块代码为远程仓库最新版本
git submodule update --remote
# 查看子模块的状态
# 如果有+号，说明子模块变了，父模块还没有将改变提交（可以git add . 、git commit -m '提交子模块更新'）
git submodule
```



![操作图示](https://raw.githubusercontent.com/li123sai/myPictures/main/img/submodule1.png)
