---
title: 使用opengrok高效率阅读源码
layout: page
categories: Other
tags: [OpenGrok]
---

## 1. 环境安装
### 1.1 安装JDK
从如下地址下载JDK安装包：http://www.oracle.com/technetwork/java/javase/downloads/jdk10-downloads-4416644.html

安装完成后，通过```java --version```命令查看是否安装成功：
>java 10.0.1 2018-04-17
>Java(TM) SE Runtime Environment 18.3 (build 10.0.1+10)
>Java HotSpot(TM) 64-Bit Server VM 18.3 (build 10.0.1+10, mixed mode)

### 1.2 安装Tomcat
Tomcat可以通过如下命令安装：```brew install tomcat```

### 1.3 设置环境变量并验证Tomcat
在～／.bashrc或者~/.bash_profile中添加如下环境变量
a) 设置JAVA环境：```export JAVA_HOME=$(/usr/libexec/java_home)```
b）设置Tomcat环境：```export PATH=$PATH:/usr/local/Cellar/tomcat/9.0.7/bin/```
c）设置OpenGrok的Tomcat路径：```export OPENGROK_TOMCAT_BASE=/usr/local/Cellar/tomcat/9.0.7/libexec/```

启动tomcat：```catalina run```，看到如下页面表示环境安装成功：
![tomcatindex](/assets/jekyll/tomcat_index.png){:class="img-responsive center-block" width="600px"}

### 1.4 安装Ctags
MacOS下可以用如下命令安装ctags：```brew install ctags```

### 1.4 安装OpenGrok
从如下地址下载OpenGrok包：https://github.com/oracle/opengrok/releases

然后解压到/usr/local目录下，并进入目录```cd /usr/local/opengrok-0.12.1.5/bin```运行```./OpenGrok deplo```。

通过```localhost:8080/source```即可看到如下页面，表示环境均安装成功：
![opengrokstart](/assets/jekyll/opengrok_start.png){:class="img-responsive center-block" width="600px"}

## 2. 源码管理
给需要管理的源码建索引：```./OpenGrok index ~/Documents/ProjectCode/Github/src/```，其中最后的绝对路径需要替换成自己项目路径。

完成操作后，即可开始遍历源码：
![opengrokbianli](/assets/jekyll/opengrok_bianli.png){:class="img-responsive center-block" width="600px"}

搜索具体的函数或者变量：
![opengroksearch](/assets/jekyll/opengrok_search.png){:class="img-responsive center-block" width="600px"}
