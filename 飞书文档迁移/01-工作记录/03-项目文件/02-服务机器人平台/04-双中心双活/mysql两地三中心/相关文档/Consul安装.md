---
created: '2024-12-09 21:25:21'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/DaOWd4Orxo3HYkxXbR9c8B8SnKe
modified: '2024-12-09 21:35:43'
source: feishu
title: Consul安装
---

Consul安装
要使用 Docker Compose 和 Consul 部署 ProxySQL 高可用集群，可以按照以下步骤进行。这个例子将展示如何使用 Docker Compose 启动 ProxySQL 和 Consul 服务，并配置 ProxySQL 集群。

环境准备

假设有以下环境：

两台 ProxySQL 节点：proxy1 和 proxy2
三台 MySQL 数据库节点：db1, db2, db3 组成 Galera Cluster
所有节点都在同一网络中，并且可以相互通信

步骤 1: 创建 Docker Compose 文件

创建一个 docker-compose.yml 文件，内容如下：

version: '3.7'

services:
  consul:
    image: consul:latest
    container_name: consul
    ports:
      - "8500:8500"
    command: "agent -server -bootstrap-expect=1 -ui -client 0.0.0.0"

  proxy1:
    image: proxysql/proxysql:latest
    container_name: proxy1
    environment:
      - PROXYSQL_CLUSTER_SERVER_ID=1
      - PROXYSQL_CLUSTER_NODES=proxy1:6031,proxy2:6031
      - PROXYSQL_ADMIN_VARIABLES=admin_host=0.0.0.0,admin_port=6032
    ports:
      - "6032:6032"  # Admin port
      - "6033:6033"  # MySQL proxy port
    volumes:
      - proxy1_config:/etc/proxysql
      - proxy1_data:/var/lib/proxysql
    depends_on:
      - consul

  proxy2:
    image: proxysql/proxysql:latest
    container_name: proxy2
    environment:
      - PROXYSQL_CLUSTER_SERVER_ID=2
      - PROXYSQL_CLUSTER_NODES=proxy1:6031,proxy2:6031
      - PROXYSQL_ADMIN_VARIABLES=admin_host=0.0.0.0,admin_port=6032
    ports:
      - "6034:6032"  # Admin port for proxy2
      - "6035:6033"  # MySQL proxy port for proxy2
    volumes:
      - proxy2_config:/etc/proxysql
      - proxy2_data:/var/lib/proxysql
    depends_on:
      - consul

volumes:
  proxy1_config:
  proxy1_data:
  proxy2_config:
  proxy2_data:

步骤 2: 启动 Docker Compose 服务

运行以下命令启动服务：

docker-compose up -d

步骤 3: 配置 Consul

Consul 已经启动并运行在 http://localhost:8500。你可以通过 Consul UI 或 API 进行配置。

步骤 4: 配置 ProxySQL 集群

连接到每个 ProxySQL 管理接口并运行必要的 SQL 命令。

对于 proxy1：

mysql -h 127.0.0.1 -P 6032 -u admin -padmin

运行 SQL 命令：

UPDATE global_variables SET variable_value='1' WHERE variable_name='cluster_enabled';
LOAD ADMIN VARIABLES TO RUNTIME;
SAVE ADMIN VARIABLES TO DISK;

INSERT INTO cluster_nodes (server_id, hostgroup_id, hostname, port, weight) VALUES (2, 0, 'proxy2', 6031, 1);
LOAD CLUSTER NODES TO RUNTIME;

对于 proxy2：

mysql -h 127.0.0.1 -P 6034 -u admin -padmin

运行 SQL 命令：

UPDATE global_variables SET variable_value='1' WHERE variable_name='cluster_enabled';
LOAD ADMIN VARIABLES TO RUNTIME;
SAVE ADMIN VARIABLES TO DISK;

INSERT INTO cluster_nodes (server_id, hostgroup_id, hostname, port, weight) VALUES (1, 0, 'proxy1', 6031, 1);
LOAD CLUSTER NODES TO RUNTIME;

步骤 5: 配置后端 MySQL 服务器（可选）

如果有一个后端 MySQL Galera 集群，可以在 ProxySQL 中配置后端服务器：

INSERT INTO mysql_servers (hostgroup_id, hostname, port, weight) VALUES
(10, 'db1', 3306, 1),
(10, 'db2', 3306, 1),
(10, 'db3', 3306, 1);

LOAD MYSQL SERVERS TO RUNTIME;

步骤 6: 配置用户认证（可选）

配置一个用户用于连接后端 MySQL 服务器：

INSERT INTO mysql_users (username, password, active, default_hostgroup) VALUES
('proxysql_user', 'proxysql_password', 1, 10);

LOAD MYSQL USERS TO RUNTIME;

步骤 7: 配置查询路由规则（可选）

配置查询路由规则，将读写请求路由到不同的服务器组：

INSERT INTO mysql_query_rules (rule_id, active, match_pattern, destination_hostgroup, apply) VALUES
(1, 1, '^SELECT.*', 20, 1),
(2, 1, '.*', 10, 1);

LOAD MYSQL QUERY RULES TO RUNTIME;

步骤 8: 测试高可用性

8.1 测试 ProxySQL 故障转移

停止 proxy1 服务。
检查 proxy2 是否接管服务。
重新启动 proxy1，检查是否重新加入集群并成为备用节点。

8.2 测试 MySQL 故障转移

停止 db1 服务。
检查 ProxySQL 是否自动将请求路由到 db2 和 db3。
重新启动 db1，检查是否自动重新加入集群。

总结

通过以上步骤，你已经成功使用 Docker Compose 和 Consul 部署了一个 ProxySQL 高可用集群。这个集群可以自动处理 ProxySQL 和后端 MySQL 服务器的故障转移，确保系统的高可用性和数据一致性。
