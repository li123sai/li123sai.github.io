---
title: Kettle之变量的作用范围
date: 2023-09-09 14:08:03
categories:
- 工具
- Kettle
tags:
- Kettle
---

## 一、Kettle的变量

Kettle可以在作业和转换中设置变量，变量设置不同作用范围也不同。

作业和转换中设置变量的差异：

作业中设置变量当前作业就可以使用。但是转换中设置变量，使用${}当前转换是拿不到的。需要在子转换或者同作业调用链转换可以获取。（这是一个巨坑）

------

Kettle变量对应着四个不同的作用域范围"s"ystem, "r"oot, "g"randparent, "p"arent。

| 变量类型选项                        | 作用域 | 作用域范围说明                                 |
| ----------------------------------- | ------ | ---------------------------------------------- |
| 在JVM中有效（Java Virtual Machine） | s      | 在一个Java虚拟机下运行的线程都生效。           |
| 在根作业中有效（root iob）          | r      | 在根作业下运行的都是生效的。                   |
| 在父作业中有效（grand-parent job）  | g      | 在当前作业的父作业下都是生效的。               |
| 在当前作业中有效（parent job）      | p      | 在当前作业下是生效的，当前作业的父作业不生效。 |

变量范围示意图:

![kettle作业变量作用范围](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable18.png)



![kettle转换变量作用范围](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable11.png)

## 二、作业设置变量

![kettle作业设置变量](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable13.png)

------

![在JVM中有效](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable14.png)



------

![在当前作业有效](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable15.png)

从上图可见，变量有效范围设置【当前作业有效】，当前作业的父作业使用${}获取的是历史变量，没有获取到最新的。如果没有历史变量就什么都获取不到。

------

![在父作业中有效](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable16.png)



------

![在根作业中有效](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable17.png)





## 三、转换设置变量

![在转换设置变量](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable12.png)

------

![Valid in the Java Virtual Machine](https://raw.githubusercontent.com/li123sai/myPictures/main/img/varable6.png)

从上图可见，变量有效范围设置【Valid in the Java Virtual Machine】，当前转换使用${}获取不到最新的。其他的都可以。

------

![Valid in the root iobl Machine](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable4.png)



------

![Valid in the grand-parent job](https://raw.githubusercontent.com/li123sai/myPictures/main/img/varable8.png)



------

![Valid in the root iob](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable9.png)



四、JS脚本设置变量

![Js脚本设置变量需要设置作用域](https://raw.githubusercontent.com/li123sai/myPictures/main/img/variable1.png)
