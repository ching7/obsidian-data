---
created: '2024-04-11 16:02:07'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/FvQkdKPcaou8mixbbfPcpixhnjd
modified: '2024-04-11 16:04:14'
source: feishu
title: Nginx+Tomcat性能优化
---

Nginx+Tomcat性能优化
调整IM的minio地址到天翼云
spring:
  tomcat:
    max-threads: 2000
dubbo:
  protocol:
    threads: 50
    thread-pool: 100
    iothreads: 50
    queues: 2000
spring.datasource.initialSize=50
spring.datasource.minIdle=100
spring.datasource.maxActive=250
# 1.work进程从1调整到16
# 2.调整log格式如下
log_format main escape=json '{ "dateTime":"
$time_local","xForwardedFor":"$
http_x_forwarded_for",'
        '"sourceAddress":"
$remote_addr","dstAddress":"$
server_addr",'
        '"dstHostName":"
$server_name","dstPort":"$
server_port",'
        '"userAgent":"
$http_user_agent","bytesOutInfo":"$
bytes_sent",'
        '"responseCode":"
$status","httpRerfer":"$
http_referer","requestBody":"
$request_body",'
        '"responseTimems":"$
request_time","requestRInfo":"
$request","webCookie":"$
http_cookie" }';
# 调整nginx的连接超时时间，防止由于接口超时导致的502
location块
  proxy_read_timeout30s;
  proxy_connect_timeout30s
  proxy_send_timeout30s;
#调整nginx 的upstream的默认负载方式
#    改为最小连接数
http {
    upstream my_backend {
        least_conn;
        server backend1.example.com;
        server backend2.example.com;
        server backend3.example.com;
    }

    server {
        listen 80;

        location / {
          proxy_pass http://my_backend;
        }
    }
}

