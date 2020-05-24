---
layout: post
title: "[Hadoop] Linux环境下的集群搭建"
subtitle: 'Hadoop2.7.2'
date: 2020-05-22 11:11:00
author: "Backspace"
header-img: "img/post-bg-apple-event-2015.jpg"
tags:
  - Hadoop
---

我是用3台服务器搭建的Hadoop集群，搭建集群之前首先要明确的就是集群的配置。

|      |       Hadoop102       |            Hadoop103             |           Hadoop104            |
| :--: | :-------------------: | :------------------------------: | :----------------------------: |
| HDFS | `NameNode`,`DataNode` |            `DataNode`            | `SecondaryNameNode`,`DataNode` |
| Yarn |     `NodeManager`     | `ResourcesManager`,`NodeManager` |         `NodeManager`          |
|      |     `jobhistory`      |                                  |                                |

**注意：**

1. `NameNode`和`SecondaryNameNode`不要安装在同一台服务器上
2. `ResourcesManager`也很消耗性能，不能跟`NameNode`和`SecondaryNameNode`安装在同一台服务器上

### 安装步骤分析：

1. 单台服务器安装Hadoop
2. SSH无密登录配置
3. 编写集群分发脚本
4. 配置Hadoop集群
5. 群起集群
6. 集群时间同步



