---
created: '2025-07-17 16:21:59'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Q0kCd1k6HojQyVxzqbSctjw4nGf
modified: '2025-07-24 16:18:43'
source: feishu
title: （全量）Mysql-xtrabackup-demo
---

（全量）Mysql-xtrabackup-demo

依赖mysql的data目录，不适合云上mysql
MYSQL部署安装

依赖：2个数据库必须开启了binglog，依赖xtrabackup插件
目录脚本
mkdir mysql-xtrabackup-demo && cd mysql-xtrabackup-demo

mkdir backup
my.conf
[mysqld]
# 以下在增量同步是必须，id集群中不能重复
server-id=1
log-bin=mysql-bin
binlog-format=ROW
gtid-mode=ON
enforce-gtid-consistency=ON

# 避免因数据导入太快而主从中断（可选）
sync_binlog=1
innodb_flush_log_at_trx_commit=1

# 容器环境兼容
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
pid-file=/var/run/mysqld/mysqld.pid

[client]
default-character-set=utf8mb4
源1 docker-compose.yml
services:
  mysql57:
    image: artifacts.iflytek.com/docker-repo/library/mysql:5.7.40
    container_name: mysql57
    environment:
      MYSQL_ROOT_PASSWORD: root@2025
      MYSQL_DATABASE: testdb
    ports:
      - "3307:3306"
    volumes:
      - ./data:/var/lib/mysql
      - ./conf/my.cnf:/etc/mysql/conf.d/my.cnf

源2 docker-compose.yml
services:
  mysql58:
    image: artifacts.iflytek.com/docker-repo/library/mysql:5.7.40
    container_name: mysql57
    environment:
      MYSQL_ROOT_PASSWORD: root@2025
      MYSQL_DATABASE: testdb
    ports:
      - "3307:3306"
    volumes:
      - ./data:/var/lib/mysql
      - ./conf/my.cnf:/etc/mysql/conf.d/my.cnf
数据初始化（验证可选）：mysqldump
# 整个数据库
mysqldump -h 172.30.33.60 -u root -p \
  --single-transaction \
  --routines --triggers \
  --all-databases \
  > full_dump.sql
# 单个数据库
mysqldump -h 172.30.33.60 -u root -p \
  --single-transaction \
  --routines --triggers \
  aicc_jxrt_sj> aicc_jxrt_sj-full_dump.sql

# 复制到容器内  
docker cp aicc_jxrt_sj-full_dump.sql  mysqlsource:/tmp/

docker exec -it mysqlsource /bin/bash
# 导入数据库
# 先创建对应的数据库
mysql -h127.0.0.1 -uroot -p 
CREATE DATABASE aicc_jxrt_sj DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
EXIT;
# 无进度显示，可能会比较慢
mysql -h127.0.0.1 -uroot -p aicc_jxrt_sj < /tmp/aicc_jxrt_sj-full_dump.sql

percona-xtrabackup
services:
  xtrabackup:
    image: artifacts.iflytek.com/docker-repo/percona/percona-xtrabackup:2.4
    container_name: xtrabackup
    network_mode: host
    volumes:
      - ./backup:/backup
      - ./mysqlsource/data:/var/lib/mysql:ro    # 关键：挂载 源MySQL 数据目录
    command: >
      xtrabackup --backup
      --host=127.0.0.1
      --port=3306
      --user=root
      --password=root@2025
      --target-dir=/backup/full
      --slave-info
    restart: "no"

补充从源表迁移到新表的操作
