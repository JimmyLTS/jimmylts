---
layout: post
title: Java学习(一) --- Mac OS X 搭建Java环境
date: 2017-03-27 20:50:24.000000000 +08:00
tags: 学习之路
---

![](/assets/images/2017/torrow_better.jpg)

## 下载JDK
Oracle官方下载链接：http://www.oracle.com/technetwork/java/javase/downloads/index.html

## 安装JDK
双击下载好的jdk.dmg文件，打开文件后双击.pkg文件进行安装
![](/assets/images/2017/java_study_1.png)
然后continue、install、输入用户密码、安装完毕close。

## 检测JDK是否安装好
在terminal输入java -version查看安装的jdk版本信息
![](/assets/images/2017/java_study_2.png)

## 查看Shell命令
查看系统使用何种Shell命令，在terminal输入echo $SHELL 。若输出bash，则为Bourne shell命令，可以通过编辑profile配置环境变量。
![](/assets/images/2017/java_study_3.png)

## 配置Java环境变量
输入sudo vim /etc/profile，然后按下 i ，进入插入模式，在文件尾部添加Java路径:
JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/"
export JAVA_HOME
CLASS_PATH="$JAVA_HOME/lib"
PATH=".$PATH:$JAVA_HOME/bin"
![](/assets/images/2017/java_study_4.png)
添加完之后，按esc推出插入模式，再输入 :wq! 强制写入并保存退出。
要想马上生效，输入 source /etc/profile

## 检查环境
在terminal输入echo $JAVA_HOME
![](/assets/images/2017/java_study_5.png)
配置完毕！