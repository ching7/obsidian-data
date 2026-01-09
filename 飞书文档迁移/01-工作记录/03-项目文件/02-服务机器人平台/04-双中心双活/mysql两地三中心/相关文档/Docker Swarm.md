---
created: '2024-12-16 17:01:32'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/SPYudDFT4omG8kxy3DmcYg1mnVb
modified: '2024-12-16 18:13:42'
source: feishu
title: ' Docker Swarm '
---

 Docker Swarm 
使用 Docker Swarm 和 Overlay 网络
Docker Swarm 是 Docker 的集群管理工具，它允许你在多个机器上创建一个集群，容器可以跨多个主机相互通信。Docker Swarm 通过 Overlay 网络 实现跨主机的容器间通信。使用 Swarm 模式，容器就像在同一网络中的容器一样，可以彼此直接访问。
步骤 1: 初始化 Docker Swarm 集群
在其中一台机器上初始化 Swarm 集群。假设该机器的 IP 是 172.30.94.25，你可以运行以下命令：
bash
复制代码
docker swarm init --advertise-addr 172.30.94.25
这将初始化 Swarm 集群，并指定主机 IP 地址作为广告地址。
Swarm initialized: current node (w47q75aexenmygi3luo9jo5pz) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-58zjygyam2t4223y7dprdjokrqxumd32fo4rrs3jzh9o6hy0gn-3ze7x93edjqorlf0t42irlbtn 172.30.94.25:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
步骤 2: 将其他节点加入到 Swarm 集群
在其他机器上，通过获取 Swarm 主节点的 join-token 来将它们加入 Swarm 集群。首先，在主节点上获取 join-token：
bash
复制代码
docker swarm join-token worker
这将输出一个命令，你可以在其他机器上运行这个命令来加入 Swarm 集群：
bash
复制代码
docker swarm join --token <worker-join-token> 172.30.94.25:2377
在其他机器上运行该命令，机器就会加入到 Swarm 集群。
This node joined a swarm as a worker.
步骤 3: 创建 Overlay 网络
在 Swarm 集群中，使用 Overlay 网络使容器能够跨机器通信：
bash
复制代码
docker network create --driver   overlay --attachable mysql-ha-network-all
这个命令将在 Swarm 集群中创建一个名为 mysql-ha-network-all 的 Overlay 网络。
可以成功启用 attachable 功能，允许手动将容器附加到 Overlay 网络。
步骤 4: 部署服务或容器
现在，你可以在这个 Overlay 网络中部署服务或容器，容器将能够跨多个主机进行通信。以下是如何在 Docker Compose 中使用 Overlay 网络的例子：


docker inspect proxysql | grep "IPAddress"
