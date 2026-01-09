---
created: '2024-05-30 15:03:11'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Y5eEdKBjqoD8dUxTUINceTrRnwf
modified: '2024-12-06 15:32:57'
source: feishu
title: docker安装mysql
---

docker安装mysql

sudo systemctl start docker
sudo systemctl enable docker
{
    "registry-mirrors": [
        "https://mirrors.tuna.tsinghua.edu.cn/"
    ]
}

sudo systemctl restart docker

sudo docker info
docer pull mysql
不同的mysql版本，/etc/mysql目录下的结构不一样 
常见 MySQL 版本的 Docker 容器配置文件路径
MySQL 8.0+
/etc/mysql/my.cnf
/etc/mysql/mysql.conf.d/
/etc/mysql/conf.d/
MySQL 5.7
/etc/mysql/my.cnf
/etc/mysql/conf.d/
/etc/mysql/mysql.conf.d/
MariaDB
/etc/my.cnf
/etc/mysql/my.cnf
/etc/my.cnf.d/
# 先登录docker仓库
# docker login artifacts.iflytek.com -u YOUR_USERNAME -p YOUR_PASSWORD
docker pull artifacts.iflytek.com/docker-repo/library/mysql
docker 5.X挂载
docker run -p 3306:3306 --name  xn-mysql \
-v /iflytek/mysql/mysqldocker/log:/var/log/mysql \
-v /iflytek/mysql/mysqldocker/data:/var/lib/mysql \
-v /iflytek/mysql/mysqldocker/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=<your_password> \
-d mysql:5.7.24
docker 8.X挂载
docker run -p 3306:3306 --name  mysql1 \
-v /iflytek/mysql/log:/var/log/mysql \
-v /iflytek/mysql/data:/var/lib/mysql \
-v /iflytek/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=<your_password> \
-d mysql:8.0.40
docker设置镜像自启动
docker update redis --restart=always
docker update mysql --restart=always

查看Docker镜像latest版本号的具体版本
docker image inspect mysql:latest | grep -i version


Elasticsearch
elasticsearch.aicc.index.seatCallRecord.name=
elasticsearch.aicc.index.name=
curl -XPUT http://127.0.0.1:9200/aicc-yace/_settings -d '{ "index" : { "max_result_window" : 300000}}'
curl -XPUT http://127.0.0.1:9200/aicc-wdx-seatcalrecord/_settings -d '{ "index" : { "max_result_window" : 300000}}'
