---
layout: post
title: "[Linux] 安装Java运行环境"
subtitle: 'JDK1.8.0_144'
author: "Backspace"
header-style: text
tags:
  - Linux
  - Java
---


### 解压JDK安装包

1.使用xftp连接Linux，将JDK的安装包上传到`/opt/software`文件夹下

2.使用xshell连接Linux[非root用户]，`cd /opt/sofware`来到software目录

3.解压安装包到/opt/module目录下 `tar -zxvf jdk1.8.0_144.tar.gz -C /opt/module`

### 配置环境变量

**方法①**

1.打开profile文件 `sudo vim /etc/profile`

2.修改文件，在profile文件末尾添加

```shell
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
```

3.保存退出后让文件生效 `source /etc/profile`

**方法②**

1.新建env.sh文件 `sudo vim /etc/profile.d/env.sh`

2.添加JAVA_HOME内容

```shell
#JAVA_HOME
export JAVA_HOME=/opt/module/jdk1.8.0_144
export PATH=$PATH:$JAVA_HOME/bin
```

3.保存退出后，让文件生效`source /etc/profile.d/env.sh`

4.测试是否成功 `java -version`