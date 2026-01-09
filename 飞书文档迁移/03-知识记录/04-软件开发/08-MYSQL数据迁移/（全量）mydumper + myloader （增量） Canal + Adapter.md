---
created: '2025-07-18 09:26:38'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Zy3xdWQNvoY3xBx85OqcheAGnJc
modified: '2025-07-25 09:53:43'
source: feishu
title: （全量）mydumper + myloader （增量） Canal + Adapter
---

（全量）mydumper + myloader （增量） Canal + Adapter

适合云下到云上mysql，依赖云下的binlog
思路
[ 云下 MySQL ]
      |
      |——[ 全量 ]——→ MyDumper  → 云上 MySQL
      |
      |——[ 增量 ]——→ Canal → Canal Adapter → 云上 MySQL
全量：mydumper + myloader 
compose
services:
  mydumper:
    image: artifacts.iflytek.com/docker-repo/mydumper/mydumper:latest
    container_name: mydumper
    network_mode: host
    volumes:
      - ./dump:/dump
      - ./logs:/logs
    restart: "no"

  myloader:
    image: artifacts.iflytek.com/docker-repo/mydumper/mydumper:latest
    container_name: myloader
    network_mode: host
    volumes:
      - ./dump:/dump
      - ./logs:/logs
    restart: "no"

清理容器
docker compose down --remove-orphans
dump
# host port根据实际情况进行调整
# --database=aicc_jxrt_sj  按照实际的情况调整schema名称
docker compose run mydumper mydumper --host=172.30.32.170 --port=3307 --user=root --password=root@2025 --database=aicc_jxrt_sj --outputdir=/dump --verbose=3  --logfile=/logs/mydumper.log --threads=6 --compress --rows=50000
loader
# --overwrite-tables 谨慎使用，会删除库中已有数据
# host port根据实际情况进行调整
# --database=aicc_jxrt_sj  按照实际的情况调整schema名称
docker compose run -d myloader myloader --host=172.30.32.170 --port=3308 --user=root --password=root@2025 --database=aicc_jxrt_sj --directory=/dump --verbose=3  --logfile=/logs/myloader.log --threads=6 --overwrite-tables
metedata
SOURCE_LOG_FILE = "mysql-bin.000005"
SOURCE_LOG_POS = 584231295
增量Canal + Adapter
配置 Canal（在云下 MySQL 侧部署）
目录
canal-adapter-jar/
canal-server-jar/
确保：
云下 MySQL 开启 binlog（必须是 ROW 格式）
云上、云下创建专用 Canal 用户，并授权：
用户创建
CREATE USER 'canal'@'%' IDENTIFIED BY 'canal@123';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
安装canal
canal-server
# https://github.com/alibaba/canal/releases
#1. 下载
canal.deployer-1.1.8.tar.gz
#2  解压
tar zxvf canal.deployer
#3 配置修改 ,conf下example 要和adapter中的相同
vi conf/example/instance.properties

#position info，需要改成源的数据库信息
canal.instance.master.address = 127.0.0.1:3306 
#如果要指定binglog，查看master的情况
# SOURCE_LOG_FILE = "mysql-bin.000005"
canal.instance.master.journal.name = 
# SOURCE_LOG_POS = 584231295
canal.instance.master.position = 
# 也支持按照时间戳
canal.instance.master.timestamp = 
#canal.instance.standby.address = 
#canal.instance.standby.journal.name =
#canal.instance.standby.position = 
#canal.instance.standby.timestamp = 
#username/password，需要改成源数据库，最好是单独的只读同步用户
canal.instance.dbUsername = canal  
canal.instance.dbPassword = canal@123

#4. 启动，日志中的binlog位置会和配置的迟一次。等于会保留最新一次的执行
# 如果要重新同步，需要删除conf/example/下的h2  以及 meta
canal-adapter
# https://github.com/alibaba/canal/releases
#1. 下载
https://github.com/alibaba/canal/releases/download/canal-1.1.8/canal.adapter-1.1.8.tar.gz
#2  解压
tar zxvf canal-adapter
#3 配置修改 

# 修改conf/application.yml为:
server:
  port: 8081
logging:
  level:
    com.alibaba.otter.canal.client.adapter.hbase: DEBUG
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: -1
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 172.30.32.170:11111
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:

  srcDataSources:
    defaultDS:
      url: jdbc:mysql://127.30.32.170:3307/aicc_jxrt_sj?useUnicode=true
      username: canal
      password: canal@123
  canalAdapters:                            
  - instance: example # 与canal-server中的保持一致
    groups:
    - groupId: g1
      outerAdapters:
      - name: rdb                                               # 指定为rdb类型同步
        key: mysql1                                            # 指定adapter的唯一key, 与表映射配置中outerAdapterKey对应
        properties:
          jdbc.driverClassName: com.mysql.jdbc.Driver        # jdbc驱动名, 部分jdbc的jar包需要自行放致lib目录下
          jdbc.url: jdbc:mysql://172.30.32.170:3308/aicc_jxrt_sj?useUnicode=true        # jdbc url
          jdbc.username: mytest                                 # jdbc username
          jdbc.password: m121212                                # jdbc password
          threads: 5            
canal-adapter/conf/rdb/mytest_user.yml
#dataSourceKey: defaultDS
#destination: example
#groupId: g1
#outerAdapterKey: mysql1
#concurrent: true
#dbMapping:
#  database: mytest
#  table: user
#  targetTable: mytest2.user
#  targetPk:
#    id: id
#  mapAll: true
#  targetColumns:
#    id:
#    name:
#    role_id:
#    c_time:
#    test1:
#  etlCondition: "where c_time>={}"
#  commitBatch: 3000 # 批量提交的大小

# 要和adapter中配置的canalAdapters保持一致
## Mirror schema synchronize config
dataSourceKey: defaultDS
destination: example
groupId: g1
outerAdapterKey: mysql1
concurrent: true
dbMapping:
  mirrorDb: true
  database: aicc_jxrt_sj
