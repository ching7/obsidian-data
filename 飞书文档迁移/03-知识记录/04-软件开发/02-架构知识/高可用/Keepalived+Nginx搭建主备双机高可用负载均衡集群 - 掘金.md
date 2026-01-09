---
created: '2024-03-13 09:16:03'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/UO1RdenTkoHqc3x333dclt7Snm5
modified: '2024-03-13 09:19:19'
source: feishu
title: Keepalived+Nginx搭建主备双机高可用负载均衡集群 - 掘金
---

Keepalived+Nginx搭建主备双机高可用负载均衡集群 - 掘金
🔗 原文链接： https://juejin.cn/post/688748743540...
⏰ 剪存时间：2024-03-13 09:16:02 (UTC+8)
✂️ 本文档由 飞书剪存 一键生成
什么是Keepalived？
Keepalived是一款基于vrrp协议的高可用集群软件，通过虚拟IP（VIP）对外提供服务，能够实时监控集群中服务器的运行状态并自动进行故障隔离，这些服务器都启动着相同的服务，当主服务器发生故障时，会自动将虚拟IP漂移到备份服务器，从而实现业务高可用。
高可用集群架构图

c1a8fd257e1091bc812631d5e3fc3cd9.png


实验方案规划


主机名

VIP

IP

Nginx端口

说明

LB-01

192.168.31.250

192.168.31.240

8080

Keepalived+Nginx主负载均衡服务器

LB-02

192.168.31.250

192.168.31.241

8080

Keepalived+Nginx备负载均衡服务器

APP-01

-

192.168.31.242

8080

后端服务器1（Nginx）

APP-02

-

192.168.31.242

8081

后端服务器2（Nginx）

APP-03

-

192.168.31.242

8082

后端服务器3（Nginx）
APP-01 、 APP-02 、 APP-03 都在同一台虚拟机上，使用三个Docker容器模拟三台后端服务器的场景，三台虚拟机系统环境均为Centos+Docker
后端服务器环境搭建
使用以下命令快速创建三个后端服务
docker run -d --name APP-01 -p 8080:80 nginx:alpine \
&& docker run -d --name APP-02 -p 8081:80 nginx:alpine \
&& docker run -d --name APP-03 -p 8082:80 nginx:alpine
修改 /usr/share/nginx/html/index.html 文件，标注出当前容器名称，便于后续测试

d70cb174858c8cf44f02f2e5e80c18a1.png


修改 /etc/nginx/conf.d/default.conf 文件，在server块中添加 add_header LB_IP $remote_addr ，标注出负载均衡服务器IP，便于后续测试

7a999d6cabbfb2976d42f79843ff8b5b.png


主备负载均衡服务器环境搭建
安装Nginx
LB-01 、 LB-02 都使用 docker run -d --name load-balancer --restart=always -p 8080:80 nginx:alpine 创建负载均衡服务，修改 /etc/nginx/nginx.conf 配置文件，配置如下：
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    upstream backend-server {
        server 192.168.31.242:8080;
        server 192.168.31.242:8081;
        server 192.168.31.242:8082;
    }
    server {
        server_name localhost;
        listen 80 ;
        location / {
            proxy_pass http://backend-server;
        }
    }
}
通过访问 192.168.31.240:8080 （LB-01）或 192.168.31.241:8080 （LB-02）即可访问后端服务

a4da17bda160a73e865c5a73eedf93ec.png



7a1e3821e19fd78119c54f396add5468.png


安装Keepalived
这里使用Docker镜像 osixia/keepalived 安装keepalived服务，具体使用说明查看 文档
LB-01主服务器
使用以下命令创建容器
docker run --cap-add=NET_ADMIN --cap-add=NET_BROADCAST --cap-add=NET_RAW --net=host -d --name keepalived --restart=always \
    -e KEEPALIVED_INTERFACE='enp0s3' \
    -e KEEPALIVED_PASSWORD='d0cker' \
    -e KEEPALIVED_STATE='BACKUP' \
    -e KEEPALIVED_ROUTER_ID='51' \
    -e KEEPALIVED_PRIORITY='120' \
    -e KEEPALIVED_UNICAST_PEERS='192.168.31.241' \
    -e KEEPALIVED_VIRTUAL_IPS='192.168.31.250' \
    osixia/keepalived:2.0.20
keepalived配置如下：
global_defs {
  default_interface enp0s3
}
vrrp_script check_nginx {
  script "/container/service/keepalived/assets/check_nginx.sh" #检测脚本文件
  interval 7           #检测时间间隔
  weight -15           #权重
}
vrrp_instance VI_1 {
  interface enp0s3     #设置实例绑定的网卡
  state BACKUP         #设置实例初始状态，实际的MASTER和BACKUP是选举决定的
  virtual_router_id 51 #同一实例下virtual_router_id必须相同
  priority 120         #设置优先级，优先级高的会被竞选为Master
  nopreempt            #非抢占模式
  unicast_peer {       #单播模式，设置对端ip
    192.168.31.241
  }
  virtual_ipaddress {  #设置VIP，可以设置多个
    192.168.31.250
  }
  authentication {     #设置认证
    auth_type PASS     #认证方式，支持PASS和AH
    auth_pass d0cker   #认证密码
  }
  track_script {       #设置追踪脚本
    check_nginx
  }
  notify "/container/service/keepalived/assets/notify.sh"
}
LB-02备服务器
使用以下命令创建容器
docker run --cap-add=NET_ADMIN --cap-add=NET_BROADCAST --cap-add=NET_RAW --net=host -d --name keepalived --restart=always \
    -e KEEPALIVED_INTERFACE='enp0s3' \
    -e KEEPALIVED_PASSWORD='d0cker' \
    -e KEEPALIVED_STATE='BACKUP' \
    -e KEEPALIVED_ROUTER_ID='51' \
    -e KEEPALIVED_PRIORITY='110' \
    -e KEEPALIVED_UNICAST_PEERS='192.168.31.240' \
    -e KEEPALIVED_VIRTUAL_IPS='192.168.31.250' \
    osixia/keepalived:2.0.20
keepalived配置如下：
global_defs {
  default_interface enp0s3
}
vrrp_script check_nginx {
  script "/container/service/keepalived/assets/check_nginx.sh"
  interval 7
  weight -15
}
vrrp_instance VI_1 {
  interface enp0s3
  state BACKUP
  virtual_router_id 51
  priority 110
  nopreempt
  unicast_peer {
    192.168.31.240
  }
  virtual_ipaddress {
    192.168.31.250
  }
  authentication {
    auth_type PASS
    auth_pass d0cker
  }
  track_script {
    check_nginx
  }
  notify "/container/service/keepalived/assets/notify.sh"
}
创建检测脚本文件
由于 LB-01 、 LB-02 keepalived配置为非抢占式，当 LB-01 的Nginx服务出现问题时，优先级降为105，而此时 LB-02 Nginx服务正常，优先级为110，尽管 LB-02 的优先级比 LB-01 高， LB-02 也并不会抢占成为Master状态，所以需要在检测Nginx状态脚本中添加关闭keepalived的命令，使Nginx出现问题时能正常完成主备切换。 /container/service/keepalived/assets/check_nginx.sh 如下：
#!/bin/bash
nc -z localhost 8080 || kill 1
使用 chmod +x /container/service/keepalived/assets/check_nginx.sh 添加脚本执行权限
访问测试
配置完成之后，重启 LB-01 、 LB-02 的load-balancer和keepalived容器，依次进行以下测试
优化（LVS+Keepalived+Nginx）
上述方案依旧是单点接入，若接入的流量超出了Nginx性能的上限，那业务还是无法达到高可用，这时候可以使用LVS（Linux虚拟服务器），它是一个虚拟服务器集群系统，是Linux系统中的 IP_VS 内核模块，能够用来做负载均衡，性能比Nginx强很多，可以承接更多的客户端访问流量，架构图如下：
6f2d2791b311974945869d4935b11278.png


