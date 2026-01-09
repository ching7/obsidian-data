---
created: '2024-11-20 10:38:16'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/KpmydkwNyo1ESKxDLPkcnEAanr3
modified: '2024-12-26 10:17:37'
source: feishu
title: mysql同城双活，异地灾备
---

mysql同城双活，异地灾备
架构图

部署视图
服务器
同城0.05ms以内。合肥上海异地15ms以内
三台相关服务器如下：已经安装好基础docker环境，挂载500G磁盘在iflytek目录下
172.30.94.25 合肥B3-5 密码：IFlytek2024
172.29.230.6 合肥城市云-1密码：IFlytek2024
172.29.67.75 上海临港-1 密码：IFlytek2024
版本包
使用 ProxySQL配置读写分离、Consul集群模式 和 Orchestrator 集群模式 实现 MySQL 高可用性。详细介绍在centos系统下同时使用docker进行的安装操作步骤、各个组件版本，来源
目前有三台服务器位于三个不同的机房，A服务器在A机房，B服务器在B机房，C服务器在C机房，AB机房是同城的机房，C机房相比AB机房是异地的机房。现在要搭建mysql的同城双活异地灾备的架构。
目前要使用 ProxySQL配置读写分离、Consul集群模式 和 Orchestrator 集群模式实现。
要求你详细介绍三个机房分别如何部署规划相关组件，同时要求在centos系统下同时使用docker进行的安装操作步骤、各个组件版本，来源
Consul
Mysql
Orchestrator

