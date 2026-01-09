---
created: '2024-12-05 14:37:29'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/MsK3dWCW8o5L3bx7gybcUdutnoh
modified: '2025-01-14 19:25:41'
source: feishu
title: MySQL 两地三中心部署文档
---

MySQL 两地三中心部署文档
背景 
为了实现 MySQL 的同城双活异地灾备架构，计划在 A 机房 （主节点）、 B 机房 （备份节点）和 C 机房 （异地灾备）之间搭建一个高可用的 MySQL 集群
ProxySQL ：实现读写分离，通过读取 Consul 的健康检查信息，动态调整数据库负载，当某个节点故障时，自动将流量转发到其他健康节点
Consul ：提供 MySQL 实例的健康检查。当某个节点失败时，ProxySQL 和 Orchestrator 会根据 Consul 的状态自动更新配置，确保流量转发到健康的节点
Orchestrator：自动监控 MySQL 集群的主从拓扑，一旦发现主节点故障，Orchestrator 会自动执行故障转移。
Percona-toolkit ：实现主从数据校验比对和异常数据处理
架构设计概述
A 机房 和 B 机房 作为同城双活机房，C 机房 作为异地灾备机房，配置 MySQL 一主多从复制，各个机房使用 ProxySQL 实现读写分离。
通过各中心机房配置 Orchestrator 用于 MySQL 主从集群自动故障切换。
Consul 用于提供服务发现、健康检查功能，并与 ProxySQL 和 Orchestrator 集成
各中心机房中的 Percona-toolkit工具进行主从数据比对和异常数据处理
架构设计图

组件版本
MySQL ：8.0.39（来源：Docker Hub artifacts.iflytek.com/docker-repo/library/mysql:8.0.39 镜像）
ProxySQL ：2.7.1-16-g2726c27（来源：Docker Hub artifacts.iflytek.com/docker-repo/proxysql/proxysql:latest 镜像）
Consul ：1.15.4（来源：Docker Hub artifacts.iflytek.com/docker-repo/library/consul镜像）
Orchestrator ：3.1.0（来源：Docker Hub artifacts.iflytek.com/docker-repo/vitess/orchestrator:latest 镜像）
Percona-toolkit ：3.1.0（来源：https://www.percona.com/downloads/percona-toolkit/3.1.0/binary/redhat/7/x86_64/percona-toolkit-3.1.0-2.el7.x86_64.rpm）
系统要求
操作系统：CentOS 7.9
Docker：安装最新版的 Docker 和 Docker Compose
部署步骤
一、Docker-CE 与Docker-Compose 安装与配置
以下操作需要在各个机房的服务器上进安装
安装 Docker

1.下载关于docker的依赖环境
yum -y install yum-utils device-mapper-persistent-data lvm2

2.设置下载Docker的镜像源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

3.安装Docker缓存
yum makecache fast

4.安装Docker的服务（务必安装的是docker-ce，docker会导致无法使用公司的内网docker仓库）
yum -y install docker-ce

5.测试Docker是否安装成功
docker version

sudo vim /etc/docker/daemon.json
#增加或修改以下配置内容：
{
  "registry-mirrors": [
    "https://hub.c.163.com",
    "https://hub.daocloud.io",
    "https://registry-1.docker.io/"
  ]
}
# 重启
systemctl restart docker

# 是否生效
docker info

# 拉取镜像的参考模板
docker pull artifacts.iflytek.com/docker-repo/xxx/xxximagename:version
docker pull artifacts.iflytek.com/docker-repo/imagename:version
安装 Docker Compose
#下载版本包
https://github.com/docker/compose/releases 
#点开后下载带这个后缀的文件：docker-compse-linux-X86_64
#如果上面的版本没有这个后缀，就往下翻找之前的版本

#上传版本包、下载好之后，将这个文件传给我们对应的服务器
# 然后将文件更改名字
mv docker-compose-linux-x86_64 docker-compose

# 移动到user/local/bin中
mv docker-compose /usr/local/bin

# 切换至对应文件夹中
cd /usr/local/bin

# 赋权最高的权限
chmod +x /usr/local/bin/docker-compose

# 查看版本，输出版本信息，安装成功
docker-compose version

二、MySQL 集群部署
MySQL 配置
新增宿主机相关目录以及文件
mkdir -p /iflytek/mysql/{conf,logs,data}
我们在 A 、B、C 机房（A 机房主节点、B 、C机房从节点）分别部署 MySQL ，并设置一主两从复制。
在 A 机房 （主节点）的 MySQL 配置文件 /iflytek/mysql/conf/my.cnf 中，配置：
[mysqld]
server-id=1
log-bin=mysql-bin
# 以下3个配置务必启用，是orch进行主从切换的关键
gtid_mode=on
enforce-gtid-consistency=true
binlog-format=ROW
report_host=172.30.94.25
report_port=3306
expire_logs_days=30
max_binlog_size=1G
在 B 机房 （从节点）的 MySQL 配置文件 /iflytek/mysql/conf/my.cnf 中，配置：
[mysqld]
server-id=2
log-bin=mysql-bin
# 以下3个配置务必启用，是orch进行主从切换的关键
gtid_mode=ON
enforce-gtid-consistency=true
binlog-format=ROW
report_host=172.29.230.60
report_port=3306
expire_logs_days=30
max_binlog_size=1G
在 C 机房 （从节点）的 MySQL 配置文件 /iflytek/mysql/conf/my.cnf 中，配置：
[mysqld]
server-id=3
log-bin=mysql-bin
# 以下3个配置务必启用，是orch进行主从切换的关键
gtid_mode=ON
enforce-gtid-consistency=true
binlog-format=ROW
report_host=172.29.67.75
report_port=3306
expire_logs_days=30
max_binlog_size=1G
编写docker-compose.yml
在 A 和 B 机房创建 Docker Compose 配置文件 docker-compose-mysql.yml
各个机房按照以下的docker-compose.yml，进行部署（调整container_name，区分各个机房的mysql）
version: '3'
services:
  mysql:
    image: artifacts.iflytek.com/docker-repo/library/mysql:8.0.39
    container_name: mysql-1
    environment:
      MYSQL_ROOT_PASSWORD: fwjqr@2024
      MYSQL_REPLICATION_USER: replica
      MYSQL_REPLICATION_PASSWORD: replica_fwjqr@2024
    networks:
      - mysql-ha-network
    ports:
      - "3306:3306"
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/mysql/log:/var/log/mysql
      - /iflytek/mysql/data:/var/lib/mysql
      - /iflytek/mysql/conf:/etc/mysql/conf.d
    command: --default-authentication-plugin=mysql_native_password
networks:
  mysql-ha-network:
启动 MySQL 容器：
docker-compose -f docker-compose.yml up -d
MySQL 主从复制配置
在 A\B\C机房均创建复制、监控、代理账户：
-- 主从复制用户
CREATE USER 'replica'@'%' IDENTIFIED BY 'replica_fwjqr@2024';
GRANT REPLICATION SLAVE ON *.* TO 'replica'@'%';

-- 监控用户
CREATE USER 'monitor'@'%' IDENTIFIED BY 'fwjqr@2024';
grant select on *.* to 'monitor'@'%';

-- proxysql 代理
CREATE USER 'proxysql'@'%' IDENTIFIED BY 'fwjqr@2024';
grant all on *.* to 'proxysql'@'%';

FLUSH PRIVILEGES;

在 B 机房 （从节点）上配置复制：
CHANGE MASTER TO MASTER_HOST = '172.30.94.25',
MASTER_USER = 'replica',
MASTER_PASSWORD = 'replica_fwjqr@2024',
-- 此处必须为0，是orch进行主从切换的关键
MASTER_AUTO_POSITION = 0;
START SLAVE;
-- 确认主从同步是否正常
SHOW SLAVE STATUS;
在 C机房 （从节点）上配置复制：
CHANGE MASTER TO MASTER_HOST = '172.30.94.25',
MASTER_USER = 'replica',
MASTER_PASSWORD = 'replica_fwjqr@2024',
-- 此处必须为0，是orch进行主从切换的关键
MASTER_AUTO_POSITION = 0;
START SLAVE;
-- 确认主从同步是否正常
SHOW SLAVE STATUS;
三、ProxySQL 配置
ProxySQL 用于实现读写分离。在 A 机房中部署 ProxySQL。整体proxysql的逻辑图如下
image.png

创建 ProxySQL 配置
配置宿主机相关目录文件
mkdir -p /iflytek/proxysql/{cnf,data}
创建 docker-compose.yml 文件：
version: '3'

services:
  proxysql:
    image: artifacts.iflytek.com/docker-repo/proxysql/proxysql
    container_name: proxysql
    networks:
      - mysql-ha-network
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/proxysql/cnf/proxysql.cnf:/etc/proxysql.cnf
      - /iflytek/proxysql/data:/var/lib/proxysql
    ports:
      - "6033:6033" # Admin port
      - "6032:6032" # MySQL proxy port
networks:
  mysql-ha-network:
ProxySQL 配置文件内容
datadir="/var/lib/proxysql"

admin_variables =
{
    admin_credentials="admin:admin;root:fwjqr@2024"
    mysql_ifaces="0.0.0.0:6032"
}

mysql_variables =
{
    threads=4
    max_connections=2048
    default_query_delay=0
    default_query_timeout=36000000
    have_compress=true
    poll_timeout=2000
    interfaces="0.0.0.0:6033"
    default_schema="information_schema"
    stacksize=1048576
    # proxysql模拟的mysql版本，此处关键，会影响应用的连接驱动的选择
    server_version="8.0.39"
    connect_timeout_server=3000
    monitor_username="monitor"
    monitor_password="fwjqr@2024"
    monitor_history=600000
    monitor_connect_interval=60000
    monitor_ping_interval=10000
    monitor_read_only_interval=1500
    monitor_read_only_timeout=500
    ping_interval_server_msec=120000
    ping_timeout_server=500
    commands_stats=true
    sessions_sort=true
    connect_retries_on_failure=10
}
ProxySQL安装
连接到 ProxySQL 的管理端口并配置数据库：
添加 MySQL 实例到 ProxySQL：
insert into mysql_servers(hostgroup_id,hostname,port,weight,comment) values(10,'172.30.94.25',3306,1,'Write Group Master');

insert into mysql_servers(hostgroup_id,hostname,port,weight,comment) values(20,'172.29.67.75',3306,1,'Read Group Slave1');

 
insert into mysql_servers(hostgroup_id,hostname,port,weight,comment) values(30,'172.29.230.6',3306,1,'Read Group Slave2');


启动 ProxySQL：
docker-compose -f docker-compose-proxysql.yml up -d
调整proxysql模拟的mysql版本
image.png

配置ProxySQL相关用户
启动Proxysql
docker-compose -f docker-compose.yml up -d
新增管理用户用于远程连接访问：由于proxysql的安全设置，默认的admin管理用于只允许本地连接
# 进入容器内部
dockect exec -it proxysql/bin/bash

# 本地使用admin管理ProxySQL(admin是默认管理用户，只允许本地登录)
mysql -uadmin -padmin -h127.0.0.1 -P6032

# 添加管理员帐户
select @@admin-admin_credentials;

set admin-admin_credentials='admin:admin;rootadmin:fwjqr@2024';
# 确认是否添加成功
select @@admin-admin_credentials;

# 加载配置到内存
load admin variables to runtime;
# 加载配置到磁盘
save admin variables to disk;
添加被代理mysql集群的相关用户
# 增加代理用户
# 此处的配置需要与 二.3节 相同
# default_hostgroup 参考 三.4 节
insert into mysql_users(username,password,default_hostgroup,transaction_persistent) values('proxysql','fwjqr@2024',10,1);

# 查看是否添加成功
select * from mysql_users\G

# 增加监控用户
select @@mysql-monitor_username;
select @@mysql-monitor_password;
# 此处的配置需要与 二.3节 相同
set mysql-monitor_password='fwjqr@2024';
select @@mysql-monitor_password;

# 加载配置到内存
load admin variables to runtime;
# 加载配置到磁盘
save admin variables to disk;
配置读写分离规则
添加路由规则(基于SQL语句)
# 写库：update|insert|delete相关操作
insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply)values(10,1,'^(update|insert|delete)',10,1);

# 指定读 
insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply)values(15,1,'^select.*from.srp*',20,1);


# 读库：select|show|desc相关操作
insert into mysql_query_rules(rule_id,active,match_digest,destination_hostgroup,apply)values(20,1,'^(select|show|desc)',30,1);

# 加载配置到内存
load admin variables to runtime;
# 加载配置到磁盘
save admin variables to disk;
proxysql登录使用master创建的proxysql
# 登录代理用户
mysql -uproxysql -pfwjqr@2024 -h127.0.0.1 -P6033

# 测试查询
SELECT * from srp.knowledge_search_record LIMIT 0,5
确认路由日志
# 登录管理端 使用 三.2 新增的管理用户，可以使用navicat的连接
mysql -prootadmin -ufwjqr@2024 -h127.0.0.1 -P6032;

# 查看统计路由信息
select * from stats_mysql_query_digest\G


报错排查
如果出现应用连接出现
autoconfigure/DruidDataSourceAutoConfigure.class]: Unknown system variable 'query_cache_size'

# 1.登录proxysql管理端进行proxysql 模拟代理客户端调整
SET mysql-server_version='8.0.39'

# 加载配置到内存
load admin variables to runtime;
# 加载配置到磁盘
save admin variables to disk;
四、Consul 配置
Consul 用于记录当前mysql集群的主节点的服务发现，目前在A机房部署。
创建 Consul 配置文件
配置相关目录以及文件
mkdir -p /iflytek/consul/{config,data1,data2,data3}
创建 docker-compose.yml 文件：
mkdir -p  /iflytek/consul-cluster/{config,data}
version: '3'
services:
  consul-server-1:
    image: artifacts.iflytek.com/docker-repo/library/consul:1.15.4
    container_name: consul-server-1
    networks:
      - mysql-ha-network
    ports:
      - "8510:8500"      # HTTP API 和 Web UI
      - "8310:8300"      # Server RPC 通信
      - "8311:8301"      # LAN Gossip (TCP)
      - "8311:8301/udp"  # LAN Gossip (UDP)
      - "8312:8302"      # WAN Gossip (TCP)
      - "8312:8302/udp"  # WAN Gossip (UDP)
      - "8610:8600"      # DNS 查询 (TCP)
      - "8610:8600/udp"  # DNS 查询 (UDP)
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/consul/config:/consul/config
      - /iflytek/consul/data1:/consul/data
    command: agent -server -ui -bootstrap-expect=3 -bind=0.0.0.0 -client=0.0.0.0
  consul-server-2:
    image: artifacts.iflytek.com/docker-repo/library/consul:1.15.4
    container_name: consul-server-2
    networks:
      - mysql-ha-network
    ports:
      - "8520:8500"      # HTTP API 和 Web UI
      - "8320:8300"      # Server RPC 通信
      - "8321:8301"      # LAN Gossip (TCP)
      - "8321:8301/udp"  # LAN Gossip (UDP)
      - "8322:8302"      # WAN Gossip (TCP)
      - "8322:8302/udp"  # WAN Gossip (UDP)
      - "8620:8600"      # DNS 查询 (TCP)
      - "8620:8600/udp"  # DNS 查询 (UDP)
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/consul/config:/consul/config
      - /iflytek/consul/data2:/consul/data
    command: agent -server -ui -bootstrap-expect=3 -bind=0.0.0.0  -client=0.0.0.0  -retry-join="consul-server-1"
  consul-server-3:
    image: artifacts.iflytek.com/docker-repo/library/consul:1.15.4
    container_name: consul-server-3
    networks:
      - mysql-ha-network
    ports:
      - "8530:8500"      # HTTP API 和 Web UI
      - "8330:8300"      # Server RPC 通信
      - "8331:8301"      # LAN Gossip (TCP)
      - "8331:8301/udp"  # LAN Gossip (UDP)
      - "8332:8302"      # WAN Gossip (TCP)
      - "8332:8302/udp"  # WAN Gossip (UDP)
      - "8630:8600"      # DNS 查询 (TCP)
      - "8630:8600/udp"  # DNS 查询 (UDP)
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/consul/config:/consul/config
      - /iflytek/consul/data3:/consul/data
    command: agent -server -ui -bootstrap-expect=3 -bind=0.0.0.0  -client=0.0.0.0  -retry-join="consul-server-1"
networks:
  mysql-ha-network:
启动相关容器
docker-compose -f docker-compose.yml up -d
Consul 相关命令
# 可视化界面
http://172.30.94.25:8510/ui/dc1/kv

# 删除相关kv：proxysql
docker exec -it consul-server-1 curl --request PUT http://172.30.94.25:8510/v1/agent/service/deregister/proxysql

# 查询相关kv：proxysql
curl http://172.30.94.25:8510/v1/catalog/service/proxysql
五、Orchestrator 配置
Orchestrator 负责 MySQL 集群的拓扑管理和故障转移，部署在 A 机房 。
创建自定义镜像
创建 docker-compose.yml 文件：
mkdir -p /iflytek/orchestrator/{conf,logs,data}
自定义镜像原有官方镜像，增加：mysql-client 和 curl
主要用于mysql触发切换事件后：基于orch自动更新consul中mysql的主节点，来更新proxysql中的代理的主写入节点
Dockerfile如下
FROM artifacts.iflytek.com/docker-repo/vitess/orchestrator

# 替换 APT 源为阿里云源
RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse" > /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse" >> /etc/apt/sources.list && \
    echo "deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse" >> /etc/apt/sources.list

# 更新软件包列表并安装 mysql-client 和 curl
RUN apt-get update && \
    apt-get install -y --allow-unauthenticated mysql-client curl && \
    rm -rf /var/lib/apt/lists/*
构建自定义镜像
docker build -t orchestrator-custom .
创建 Orchestrator容器
新增相关目录
mkdir -p /iflytek/orchestrator/{conf,logs,data}
创建 docker-compose.yml 文件：
version: '3'

services:
  orchestrator1:
    image: orchestrator-custom
    container_name: orchestrator1
    ports: 
      - "3001:3000"          # Web UI 端口
      - "10018:10008"        # Raft 通信端口
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/orchestrator1/conf/orchestrator.conf.json:/conf/orchestrator.conf.json
      - /iflytek/orchestrator1/logs:/var/log/orchestrator
      - /iflytek/orchestrator1/data:/orchestrator-data
    networks:
      - mysql-ha-network

  orchestrator2:
    image: orchestrator-custom
    container_name: orchestrator2
    ports: 
      - "3002:3000"          # Web UI 端口
      - "10028:10008"        # Raft 通信端口
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/orchestrator2/conf/orchestrator.conf.json:/conf/orchestrator.conf.json
      - /iflytek/orchestrator2/logs:/var/log/orchestrator
      - /iflytek/orchestrator2/data:/orchestrator-data
    networks:
      - mysql-ha-network

  orchestrator3:
    image: orchestrator-custom
    container_name: orchestrator3
    ports: 
      - "3003:3000"          # Web UI 端口
      - "10038:10008"        # Raft 通信端口
    volumes:
      # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /etc/localtime:/etc/localtime:ro
      - /iflytek/orchestrator3/conf/orchestrator.conf.json:/conf/orchestrator.conf.json
      - /iflytek/orchestrator3/logs:/var/log/orchestrator
      - /iflytek/orchestrator3/data:/orchestrator-data
    networks:
      - mysql-ha-network
networks:
  mysql-ha-network:
Orchestrator依赖环境准备
Orchestrator的元数据库
元数据存储需要单独的mysql数据库，不能使用被管理的mysql机器，参考第二节独立在A机房部署元数据库：docker-compose.yml 
version: '3'
services:
  mysql:
    image: artifacts.iflytek.com/docker-repo/library/mysql:8.0.39
    container_name: orch-mysql-1
    environment:
      MYSQL_ROOT_PASSWORD: fwjqr@2024
      MYSQL_REPLICATION_USER: replica
      MYSQL_REPLICATION_PASSWORD: replica_fwjqr@2024
    networks:
      - mysql-ha-network
    ports:
      - "3307:3306"
    volumes:
      - /etc/localtime:/etc/localtime:ro # 使用宿主机的时间，避免宿主机和容器内时间不一致
      - /iflytek/mysql-orch/log:/var/log/mysql
      - /iflytek/mysql-orch/data:/var/lib/mysql
      - /iflytek/mysql-orch/conf:/etc/mysql/conf.d
    command: --default-authentication-plugin=mysql_native_password
networks:
  mysql-ha-network:
Orchestrator的集群信息
需要在orch管理的数据库初始化写入当前mysql集群信息
-- 1.在被管理mysql主节点新增_meta数据库

-- 2.新增 instance_info、cluster_info 表
-- instance_info
CREATE TABLE `instance_info` (
  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
  `report_host` varchar(128) NOT NULL DEFAULT '',
  `report_port` int(10) unsigned NOT NULL,
  `instance_name` varchar(128) NOT NULL DEFAULT '',
  `datacenter` varchar(30) NOT NULL DEFAULT '',
  `promotion_rule` varchar(20) NOT NULL DEFAULT '' COMMENT 'prefer/neutral/prefer_not/must_not',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8

-- cluster_info
CREATE TABLE `cluster_info` (
  `id` tinyint(3) unsigned NOT NULL,
  `cluster_name` varchar(128) CHARACTER SET ascii NOT NULL DEFAULT '',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8

-- 3.集群节点信息初始化 promotion_rule：主从切换的规则
-- 节点信息 
INSERT INTO `_meta`.`instance_info` (`id`, `report_host`, `report_port`, `instance_name`, `datacenter`, `promotion_rule`) VALUES (1, '172.30.94.25', 3306, '172.30.94.25:3306', 'zone-hfb35-01', 'prefer');
INSERT INTO `_meta`.`instance_info` (`id`, `report_host`, `report_port`, `instance_name`, `datacenter`, `promotion_rule`) VALUES (2, '172.29.67.75', 3306, '172.29.67.75:3306', 'zone-sh-01', 'prefer');
INSERT INTO `_meta`.`instance_info` (`id`, `report_host`, `report_port`, `instance_name`, `datacenter`, `promotion_rule`) VALUES (3, '172.29.230.6', 3306, '172.29.230.6:3306', 'zone-hfchy-01', 'prefer');
--集群信息
INSERT INTO `_meta`.`cluster_info` (`id`, `cluster_name`) VALUES (1, 'fwjqr-mysql-cluster');
Mysql Failover 通知脚本
该脚本主要用于mysql触发切换事件后：基于orch自动更新consul中mysql的主节点，来更新proxysql中的代理的主写入节点：update_proxysql_master.sh
该脚本需要复制到orch的各个节点的：/iflytek/orchestrator1/data目录下
#!/bin/bash

# 配置项：五.3.2节的集群名称
CLUSER_NAME="fwjqr-mysql-cluster"
# Consul KV 存储 MySQL master 的路径
CONSUL_URL="http://172.30.94.25:8510/v1/kv/mysql/master/${CLUSER_NAME}/hostname"   
PROXYSQL_HOST="172.30.94.25"                               # ProxySQL 地址
PROXYSQL_PORT=6032                                         # ProxySQL 管理端口
PROXYSQL_USER="rootadmin"                                  # ProxySQL 用户
PROXYSQL_PASSWORD="fwjqr@2024"                             # ProxySQL 密码
PROXYSQL_HOSTGROUP_ID=10                                   # ProxySQL 主节点的 hostgroup ID
MYSQL_PORT=3306                                            # MySQL 默认端口
LOG_FILE="/var/log/orchestrator/update_proxysql_master.log" # 日志文件
MYSQL_CMD="mysql -h $PROXYSQL_HOST -P $PROXYSQL_PORT -u $PROXYSQL_USER -p$PROXYSQL_PASSWORD" # MySQL 命令封装
# mysql -h 172.30.94.25 -P 6032 -u rootadmin -pfwjqr@2024
# 日志函数
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

# 1. 获取 Consul 中当前的 master 节点的hostname
log "Fetching MySQL master node from Consul with ${CONSUL_URL}?raw..."
CONSUL_MASTER=$(curl -s "${CONSUL_URL}?raw")

if [[ -z "$CONSUL_MASTER" ]]; then
    log "ERROR: Unable to retrieve master node from Consul."
    exit 1
fi

log "Current master node from Consul: $CONSUL_MASTER"

# 2. 获取 ProxySQL 当前主节点
log "Checking current ProxySQL master configuration..."
CURRENT_PROXYSQL_MASTER=$($MYSQL_CMD -se \
"SELECT hostname FROM mysql_servers WHERE hostgroup_id=$PROXYSQL_HOSTGROUP_ID AND status='ONLINE' LIMIT 1;")

if [[ -z "$CURRENT_PROXYSQL_MASTER" ]]; then
    log "WARNING: Unable to retrieve current master node from ProxySQL."
fi

log "Current ProxySQL master: $CURRENT_PROXYSQL_MASTER"

# 3. 比较 Consul 和 ProxySQL 中的 master 信息
if [[ "$CONSUL_MASTER" == "$CURRENT_PROXYSQL_MASTER" ]]; then
    log "Master node is already up-to-date in ProxySQL. No changes needed."
    exit 0
fi

# 4. 更新 ProxySQL 中的主节点信息
log "Updating ProxySQL master to $CONSUL_MASTER..."

# 禁用旧主节点并记录切换信息
log "Disabling old master: $CURRENT_PROXYSQL_MASTER"
$MYSQL_CMD -e "UPDATE mysql_servers
SET status='OFFLINE_SOFT', comment='Switched at $(date '+%Y-%m-%d %H:%M:%S') 
from old master: $CURRENT_PROXYSQL_MASTER:$MYSQL_PORT'
WHERE hostgroup_id=$PROXYSQL_HOSTGROUP_ID AND hostname='$CURRENT_PROXYSQL_MASTER';"

# 启用新主节点并添加切换信息到 comment
log "Enabling new master: $CONSUL_MASTER"
$MYSQL_CMD -e "DELETE FROM mysql_servers
WHERE hostgroup_id=$PROXYSQL_HOSTGROUP_ID";
$MYSQL_CMD -e "
INSERT INTO mysql_servers (hostgroup_id, hostname, port, status, comment)
VALUES ($PROXYSQL_HOSTGROUP_ID, '$CONSUL_MASTER', $MYSQL_PORT, 'ONLINE', 'New master set on $(date '+%Y-%m-%d %H:%M:%S') 
from old master: $CURRENT_PROXYSQL_MASTER:$MYSQL_PORT');"

# 应用配置更改
log "Applying changes to ProxySQL runtime and saving to disk..."
$MYSQL_CMD -e "LOAD MYSQL SERVERS TO RUNTIME; SAVE MYSQL SERVERS TO DISK;"

log "ProxySQL master successfully updated to $CONSUL_MASTER."
主从复制延迟配置
orch支持检测主从复制延迟。建议最后进行配置，会影响主从切换
-- 登录被管理的mysql主节点
-- 创建时间戳表
CREATE TABLE IF NOT EXISTS mysql.rpl_monitor (
    ts TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
-- 创建定时任务
CREATE EVENT IF NOT EXISTS update_rpl_monitor
ON SCHEDULE EVERY 1 SECOND
DO
  INSERT INTO mysql.rpl_monitor (ts) VALUES (NOW())
  ON DUPLICATE KEY UPDATE ts=NOW();
-- 开启定时任务
SET GLOBAL event_scheduler = ON;
配置 Orchestrator
增加各个orch节点配置
{
  "Debug": true,
  "ListenAddress": ":3000",
  "RaftEnabled": true,
  "RaftDataDir": "/orchestrator-data",
  # 以下2点，根据各个节点调整
  "RaftBind": "0.0.0.0:10018",
  "RaftAdvertise": "orchestrator1:10018",
  "RaftNodes": ["orchestrator1:10018", "orchestrator2:10028", "orchestrator3:10038"],
  # 被管理的mysql信息
  "MLTopologyUser": "root",
  "MySQLTopologyPassword": "fwjqr@2024",
  # 元数据存储的mysql
  "BackendDB": "mysql",
  "MySQLOrchestratorHost": "172.30.94.25",
  "MySQLOrchestratorPort": 3307,
  "MySQLOrchestratorDatabase": "orchestrator",
  "MySQLOrchestratorUser": "root",
  "MySQLOrchestratorPassword": "fwjqr@2024",
  "SQLite3DataFile": "/orchestrator-data/orchestrator.db",
  # consul地址，参考第四节地址
  "ConsulAddress": "172.30.94.25:8510",
  "HostnameResolveMethod": "@@report_host",
  "MySQLHostnameResolveMethod": "@@report_host",
  # 参考 五.3 节配置
  "DetectClusterAliasQuery" : "SELECT cluster_name FROM _meta.cluster_info  WHERE id=1 and exists (select 1 from _meta.instance_info WHERE report_host=@@report_host and report_port=@@report_port);",
  "DetectPromotionRuleQuery": "SELECT promotion_rule FROM _meta.instance_info WHERE report_host=@@report_host and report_port=@@report_port",
  "DetectDataCenterQuery": "SELECT datacenter FROM _meta.instance_info WHERE report_host=@@report_host and report_port=@@report_port",
  "DetectInstanceAliasQuery": "SELECT instance_name FROM _meta.instance_info WHERE report_host=@@report_host and report_port=@@report_port",
  "LogFile": "/var/log/orchestrator/orchestrator.log",
  # 参考 五.3 节配置，建议先置为空
  "ReplicationLagQuery": "SELECT ROUND(TIMESTAMPDIFF(SECOND, MAX(ts), NOW())) AS `lag` FROM mysql.rpl_monitor;",
  "ReasonableReplicationLagSeconds": 5,
  "KVClusterMasterPrefix": "mysql/master",
  "DefaultInstancePort": 3306,
  "DiscoverByShowSlaveHosts": true,
  "InstancePollSeconds": 5,
  "DiscoveryIgnoreReplicaHostnameFilters": [
    "a_host_i_want_to_ignore[.]example[.]com",
    ".*[.]ignore_all_hosts_from_this_domain[.]example[.]com",
    "a_host_with_extra_port_i_want_to_ignore[.]example[.]com:3307"
  ],
  "UnseenInstanceForgetHours": 240,
  "SnapshotTopologiesIntervalHours": 0,
  "InstanceBulkOperationsWaitTimeoutSeconds": 10,
  "StatusSimpleHealth": true,
  "StatusOUVerify": false,
  "AgentPollMinutes": 60,
  "UnseenAgentForgetHours": 6,
  "StaleSeedFailMinutes": 60,
  "SeedAcceptableBytesDiff": 8192,
  "PseudoGTIDPattern": "",
  "PseudoGTIDPatternIsFixedSubstring": false,
  "PseudoGTIDMonotonicHint": "asc:",
  "DetectPseudoGTIDQuery": "",
  "BinlogEventsChunkSize": 10000,
  "SkipBinlogEventsContaining": [],
  "ReduceReplicationAnalysisCount": true,
  "FailureDetectionPeriodBlockMinutes": 60,
  "FailMasterPromotionOnLagMinutes": 0,
  "AutoMasterFailover": true,
  "RecoveryEnabled": true,
  "RecoveryPeriodBlockSeconds": 60,
  "RecoveryIgnoreHostnameFilters": [],
  "RecoverMasterClusterFilters": [
    "*"
  ],
  "RecoverIntermediateMasterClusterFilters": [
    "*"
  ],
  "OnFailureDetectionProcesses": [
    "echo 'Detected {failureType} on {failureCluster}. Affected replicas: {countSlaves}' >> /tmp/recovery.log"
  ],
  "PreGracefulTakeoverProcesses": [
    "echo 'Planned takeover about to take place on {failureCluster}. Master will switch to read_only' >> /tmp/recovery.log"
  ],
  "PreFailoverProcesses": [
    "echo 'Will recover from {failureType} on {failureCluster}' >> /tmp/recovery.log"
  ],
  "PostFailoverProcesses": [
    "echo '(for all types) Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostUnsuccessfulFailoverProcesses": [],
  # 参考 五.3.3 节的脚本设置
  "PostMasterFailoverProcesses": [
    "echo 'Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Promoted: {successorHost}:{successorPort}' >> /tmp/recovery.log",
    "/orchestrator-data/update_proxysql_master.sh"
  ],
  "PostIntermediateMasterFailoverProcesses": [
    "echo 'Recovered from {failureType} on {failureCluster}. Failed: {failedHost}:{failedPort}; Successor: {successorHost}:{successorPort}' >> /tmp/recovery.log"
  ],
  "PostGracefulTakeoverProcesses": [
    "echo 'Planned takeover complete' >> /tmp/recovery.log"
  ],
  "PostponeReplicaRecoveryOnLagMinutes": 0
}
启动 Orchestrator 容器：
docker-compose -f docker-compose-orchestrator.yml up -d
Web界面访问
Orchestrator 会自动检测 MySQL 集群的拓扑，并支持自动故障转移。
通过 Web UI ( http://172.30.94.25:3001/web/clusters/ ) 进行监控和管理，如果没有mysql集群信息在页面上填写主节点信息
image.png

正常发现拓扑后如下显示：
image.png

Orchestraor

curl -s "http://172.30.94.25:3001/api/cluster-info/fwjqr-mysql-cluster" | jq .

curl -s "http://172.30.94.25:3001/api/health" | jq .

curl -s "http://172.30.94.25:3001/api/graceful-master-takeover/172.30.94.25/3306/172.29.230.6/3306" | jq .

curl -s "http://172.30.94.25:3001/api/graceful-master-takeover-auto/172.30.94.25/3306/172.29.230.6/3306" | jq .
六、Percona-toolkit配置
提供MySQL 管理工具包，可以执行各种复杂和麻烦的 MySQL 和系统任务，可以用于整个mysql机器的数据比对、异常数据处理
安装Percona-toolkit
## 下载版本包
wget https://www.percona.com/downloads/percona-toolkit/3.1.0/binary/redhat/7/x86_64/percona-toolkit-3.1.0-2.el7.x86_64.rpm 


http://mirrors.aliyun.com/centos/7/os/x86_64/Packages/perl-DBI-1.627-4.el7.x86_64.rpm
http://mirrors.aliyun.com/epel/7/x86_64/Packages/p/perl-RPC-PlClient-0.20190529-1.el7.noarch.rpm

sudo yum install --downloadonly --downloaddir=./ percona-toolkit
yumdownloader -resolve percona-toolkit
## 安装
yum install -y percona-toolkit-3.1.0-2.el7.x86_64.rpm
使用Percona-toolkit
# 主从数据一致性校验 = 整个数据库扫描
pt-table-checksum --replicate=percona.checksums --user=root --password=fwjqr@2024 --host=172.30.94.25  --no-check-binlog-format
# 指定数据库 --databases=srp 
# 指定数据库表 -tables=knowledge_search_record 支持逗号拼接
pt-table-checksum --replicate=percona.checksums --user=root --password=fwjqr@2024 --host=172.30.94.25   --databases=srp --tables=knowledge_search_record --no-check-binlog-format

# 数据库、表差异检测、修复
# --execute 执行
# --print 仅查看,输出即将执行的sql但是不执行
# 指定的表 mysql user 
pt-table-sync --replicate=percona.checksums --print --sync-to-master h=172.29.230.6,u=root,p=fwjqr@2024,D=mysql,t=user
# 整个库 srp
pt-table-sync --replicate= percona.checksums --print --sync-to-master --databases=srp h=172.29.230.6,u=root,p=fwjqr@2024

# 整个库 srp
pt-table-sync --replicate=percona.checksums --print --sync-to-master --databases=srp h=172.29.230.6,u=root,p=fwjqr@2024


pt-table-sync --print h=172.29.230.6,u=root,P=3306,p=fwjqr@2024 h=172.30.94.25,u=root,P=3306,p=fwjqr@2024 --databases=srp

pt-table-sync --print h=172.30.94.25,u=root,P=3306,p=fwjqr@2024 h=172.29.230.6,u=root,P=3306,p=fwjqr@2024 --databases=srp

pt-table-sync --print  --replicate --sync-to-source h=172.29.67.75,u=root,P=3306,p=fwjqr@2024  --databases=icms --tables=fs_reg
pt-table-sync  --print --sync-to-master --databases=srp h=172.29.67.75,u=root,p=fwjqr@2024
pt-table-sync --print  --sync-to-master --databases=srp h=172.29.67.75,u=root,p=fwjqr@2024

# 主从延迟、也可以通过orch设置  show slave status  
# 创建心跳表，并且更新时间戳
# --daemonize 后台
# pt-heartbeat --stop  --create-table --database=srp --table=heartbeat h=172.30.94.25,u=root,p=fwjqr@2024 --interval=1 停止后台运行
# --print-master-server-id 打印主节点id
pt-heartbeat --update --create-table --database=srp --table=heartbeat h=172.30.94.25,u=root,p=fwjqr@2024 --interval=1

# 检查从库延迟
pt-heartbeat --monitor --database=srp --table=heartbeat --interval=1 h=172.29.230.6,u=root,p=fwjqr@2024
pt-heartbeat --monitor --database=srp --table=heartbeat --interval=1 h=172.29.67.75,u=root,p=fwjqr@2024

#  发现主从关系
pt-slave-find h=172.30.94.25,u=root,p=fwjqr@2024

#用于分析mysql系统变量可能存在的一些问题，可以据此评估有关参数的设置正确与否。
pt-variable-advisor h=172.30.94.25,u=root,p=fwjqr@2024

#用于比较mysql配置文件和服务器变量
#至少2个配置源需要指定，可以用于迁移或升级前后配置文件进行对比
pt-config-diff h=172.30.94.25,u=root,p=fwjqr@2024 h=172.29.230.6,u=root,p=fwjqr@2024

#功能为从mysql表中找出重复的索引和外键，这个工具会将重复的索引和外键都列出来
#同时也可以生成相应的drop index的语句
# --dry-run 项进行预览
pt-duplicate-key-checker --sql  h=172.30.94.25,u=root,p=fwjqr@2024 --databases=srp 
pt-duplicate-key-checker --sql  h=172.30.33.60,u=root,p=0JsIs7n9Uy7FNZN9tA --databases=aicc_jxrt_sj --tables=call_record


pt-heartbeat --monitor --database=srp --table=heartbeat --interval=1 h=172.29.67.75,u=root,p=fwjqr@2024 --daemonize --log=heart.log
其余命令
如果出现相关命令无输出
# 1.找到锁文件位置
# 锁文件通常位于以下路径之一：
/tmp/pt-heartbeat.*
/var/run/pt-heartbeat.*
# 运行以下命令查找锁文件：
ls -l /tmp/pt-heartbeat*
ls -l /var/run/pt-heartbeat*
# 2. 删除锁文件
# 确认后手动删除锁文件：
rm -f /tmp/pt-heartbeat*
rm -f /var/run/pt-heartbeat*

pt-upgrade
  #该命令主要用于对比不同mysql版本下SQL执行的差异，通常用于升级前进行对比。
  #会生成SQL文件或单独的SQL语句在每个服务器上执行的结果、错误和警告信息等。
 
pt-online-schema-change
  #功能为支持在线变更表构，且不锁定原表，不阻塞原表的DML操作。
  #该特性与Oracle的dbms_redefinition在线重定义表原理基本类似。
   
pt-mysql-summary
  #对连接的mysql服务器生成一份详细的配置情况以及sataus信息
  #在尾部也提供当前实例的的配置文件的信息
   
pt-mext
  #并行查看SHOW GLOBAL STATUS的多个样本的信息。
  #pt-mext会执行你指定的COMMAND，并每次读取一行结果，把空行分割的内容保存到一个一个的临时文件中，最后结合这些临时文件并行查看结果。
 
pt-kill
  #Kill掉符合指定条件mysql语句
 
pt-ioprofile
  #pt-ioprofile的原理是对某个pid附加一个strace进程进行IO分析
   
pt-fingerprint
  #用于生成查询指纹。主要将将sql查询生成queryID，pt-query-digest中的ID即是通过此工具来完成的。
  #类似于Oracle中的SQL_ID，涉及绑定变量，字面量等
 
pt-find
  #用与查找mysql表并执行指定的命令，类似于find命令
 
pt-fifo-split
  #模拟切割文件并通过管道传递给先入先出队列而不用真正的切割文件
   
pt-deadlock-logger
  #用于监控mysql服务器上死锁并输出到日志文件，日志包含发生死锁的时间、死锁线程id、死锁的事务id、发生死锁时事务执行时间等详细信息。
 
pt-archiver
  #将mysql数据库中表的记录归档到另外一个表或者文件
  #该工具具只是归档旧的数据，对线上数据的OLTP查询几乎没有影响。
  #可以将数据插入另外一台服务器的其他表中，也可以写入到一个文件中，方便使用load data infile命令导入数据。
 
pt-agent
  #基于Percona Cloud的一个客户端代理工具
 
pt-visual-explain
  #用于格式化explain的输出
 
pt-variable-advisor
  #用于分析mysql系统变量可能存在的一些问题，可以据此评估有关参数的设置正确与否。
 
pt-stalk
  #用于收集mysql数据库故障时的相关信息便于后续诊断处理。
   
pt-slave-delay
  #用于设定从服务器落后于主服务器的时间间隔。
  #该命令行通过启动和停止复制sql线程来设置从落后于主指定时间。
 
pt-sift
  #用于浏览pt-stalk生成的文件。
   
pt-show-grants
  #将当前实例的用户权限全部输出，可以用于迁移数据库过程中重建用户。
   
pt-query-digest
  #用于分析mysql服务器的慢查询日志，并格式化输出以便于查看和分析。
 
pt-pmp
  #为查询程序执行聚合的GDB堆栈跟踪，先进性堆栈跟踪，然后将跟踪信息汇总。
   
pt-index-usage
  #从log文件中读取查询语句，并用分析当前索引如何被使用。
  #完成分析之后会生成一份关于索引没有被查询使用过的报告，可以用于分析报告考虑剔除无用的索引。
   
pt-heartbeat
  #用于监控mysql复制架构的延迟。
  #主要是通过在主库上的--update线程持续更新指定表上的一个时间戳，从库上--monitor线程或者--check线程检查主库更新的时间戳并与当前系统时间对比，得到延迟值。
 
pt-fk-error-logger
  #将外键相关的错误信息记录到日志或表。
 
pt-duplicate-key-checker
   #功能为从mysql表中找出重复的索引和外键，这个工具会将重复的索引和外键都列出来
   #同时也可以生成相应的drop index的语句
   
pt-diskstats
  #类似于iostat，打印磁盘io统计信息，但是这个工具是交互式并且比iostat更详细。可以分析从远程机器收集的数据。
 
pt-config-diff
  #用于比较mysql配置文件和服务器变量
  #至少2个配置源需要指定，可以用于迁移或升级前后配置文件进行对比
   
pt-align
  #格式化输出
   
pt-slave-find
  #连接mysql主服务器并查找其所有的从，然后打印出所有从服务器的层级关系。
   
pt-table-checksum
  #用于校验mysql复制的一致性。
  #该工具主要是高效的查找数据差异，如果存在差异性，可以通过pt-table-sync来解决
主从切换验证
手动切换
异常切换
应用切换验证
遗留问题

Q&A
orch使用问题
出现NoFailoverSupportStructureWarning，导致无法进行手动主从切换
image.png

检查主从节点是否开启GTID模式（on），GTID是否生效（生效为true）
image.png

image.png


如果出现主从不一致的问题
