---
created: '2025-04-07 17:29:27'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Hez0dCmQwo6Qu5x7TEPcIe6Ynzd
modified: '2025-04-08 17:47:35'
source: feishu
title: 使用nacos自建mcp服务
---

使用nacos自建mcp服务
docker搭建nacos
docker-compose.yml
version: '3'
services:
  nacos:
    image: nacos/nacos-server:2.2.3
    container_name: nacos-standalone
    environment:
      - MODE=standalone
      - NACOS_AUTH_TOKEN=mcpdemo  # 替换为实际的认证令牌
      - NACOS_AUTH_IDENTITY_VALUE=mcpdemo  # 替换为实际的身份值
    ports:
      - "8848:8848"
      - "9848:9848"
      - "9849:9849"
    volumes:
      - ./nacos-data:/home/nacos/data
      - ./nacos-logs:/home/nacos/logs
    restart: always
docker搭建k8s集群
本地Kind

chmod +x ./hgctl
sudo mv ./hgctl /usr/local/bin
下载kind：https://github.com/kubernetes-sigs/kind/releases
下载镜像：docker pull kindest/node:v1.32.2
启动kind：kind create cluster --image kindest/node:v1.32.2 --name my-k8s-cluster 
下载kubectl：https://higress.cn/docs/latest/user/quickstart/
下载hgctl
新建service  foo.yaml

kubectl apply -f foo.yaml
#检查 Pod 和 Service 是否创建成功：

kubectl get pod
kubectl get service

#由于 kind 是本地集群，默认是 NodePort 或 port-forward 才能访问 Service。
#可以用端口转发方式来访问：
kubectl port-forward service/foo-service 5678:5678

docker搭建mcp服务

