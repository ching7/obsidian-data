# k8s
「为什么本地开发更推荐用 Kind，而不是直接装 K8s 集群。」
---
1. 本质区别：Kind 和 Kubernetes

|       |                                                    |                                  |
| ----- | -------------------------------------------------- | -------------------------------- |
| 对比维度  | Kind                                               | Kubernetes                       |
| 本质    | Kubernetes in Docker（把 Kubernetes 跑在本地 Docker 容器里） | Kubernetes 是完整的分布式系统，部署需要虚拟机或物理机 |
| 安装复杂度 | ✅ 一条命令就可以创建集群，极简                                   | ❌ 安装复杂，需要配置节点、网络                 |
| 资源消耗  | ✅ 轻量，本地笔记本电脑也能跑                                    | ❌ 占资源，至少需要几台虚拟机                  |
| 适用场景  | 本地开发、学习、测试                                         | 生产环境、性能测试                        |
| 节点扩展  | 有限（开发用途够用）                                         | 灵活，生产环境可扩展上百节点                   |
| 删除集群  | ✅ 删除简单，一条命令清理                                      | ❌ 删除麻烦，需要清理磁盘、节点                 |
2. 打个生活中的比喻 🚗
- **Kind：** 就像「搭积木模型」，你在桌子上先搭一个小型的微缩模型。成本低，调试方便，出错了拆了重来。
- **Kubernetes（生产级）：** 就是「盖真正的大楼」，你要打地基、安装电梯、布置水电。成本高，调试周期长，动静也大。
---
3. 为什么本地更推荐 Kind？
✅ **方便快捷**
- Kind 是给本地开发设计的，1-2 分钟就可以创建一个完整的集群环境。
- 配置非常轻便，不需要复杂的网络配置和硬件资源。
✅ **更接近真实 Kubernetes**
- 虽然是跑在 Docker 容器里的，但它使用的是 Kubernetes 的核心组件，和生产环境高度一致。
- 用 Kind 学习、开发，未来迁移到云端 Kubernetes 几乎无缝对接。
✅ **多集群模拟**
- Kind 甚至可以在一台电脑上创建多个集群，模拟多集群环境，非常适合做多集群调度和网络实验。
✅ **清理简单**
- 一个命令删除集群，不用担心遗留数据或系统污染。
4. 真实开发流程示意图 🎨
```Plain
你的电脑
└── Docker（容器引擎）
    └── Kind（Kubernetes 集群）
        ├── Node 1 (控制平面 + worker)
        └── Node 2 (worker，可选)
你的应用开发流程：
本地代码  →  Docker 镜像 →  Kind 集群部署 → 测试成功 → 云端 Kubernetes 生产环境发布
```
---
5. 结论 ✅
**总结一句话：**
> 本地开发用 Kind，既能节省资源，又能高效模拟真实的 Kubernetes 环境，开发调试成本低，学习效率高！
---
## Kind
创建 Kind 集群就像「启动一个 Docker Compose」，但是它是多容器的 K8s 环境。
### kind安装
```YAML
#下载kind：
https://github.com/kubernetes-sigs/kind/releases
#下载镜像：
docker pull kindest/node:v1.32.2
#启动kind：
kind create cluster --image kindest/node:v1.32.2 --name my-k8s-cluster 

kind get clusters
```


## kubectl
管理kubernetes的工具
### 类比图：Kubernetes 集群 = 小区管理系统 🏘️
|                          |                                                    |                                                                                 |
| ------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------- |
| Kubernetes 概念            | 小区比喻                                               | 解释                                                                              |
| Cluster（集群）              | 小区                                                 | 一整个住宅小区，里面有很多楼栋和住户。                                                             |
| Node（节点）                 | 楼栋                                                 | 小区里的每一栋大楼，里面有很多房间。可以是物理机或虚拟机。                                                   |
| Pod                      | 房间                                                 | 楼里的一个房间，里面可以住一户或者几户（多个容器）。                                                      |
| Container（容器）            | 居民                                                 | 住在房间里的居民，真正干活的单位。                                                               |
| Service                  | 小区大门 / 快递柜 / 便利店入口                                 | 房子对外的出入口，让外部的人能找到住户。                                                            |
| Deployment / StatefulSet | 物业公司装修合同                                           | 告诉施工队怎么装修房子、配置住户。                                                               |
| Namespace（命名空间）          | 小区里的不同区                                            | 小区有东区、西区、南区，各个区域互不影响，管理不同类型的房子。                                                 |
| ConfigMap / Secret       | 住户手册 / 保险箱密码                                       | 房间里的配置说明书或者私密信息。                                                                |
| kubectl                  | 小区物业管理的对讲机                                         | 管理员用它来调度、查询小区情况。                                                                |
|                          |                                                    |                                                                                 |
| Kubernetes 概念            | Docker 类比                                          | 解释                                                                              |
| Pod                      | Container（容器）                                      | Pod 是 K8s 最小调度单元，里面可以包含一个或多个容器。Docker 里只有容器的概念；你可以把 Pod ≈ Docker 容器 + 一层管理壳。    |
| Node                     | Docker Host（运行 Docker Engine 的机器）                  | Node 是运行 Pod 的节点，就像 Docker 的宿主机，里面可以启动多个容器。                                     |
| Deployment               | docker run --restart=always + docker-compose scale | Deployment 管理 Pod 副本数，保证数量一致，类似你在 Docker 里用参数让容器自动重启，或者用 docker-compose 扩容多个容器。 |
| Service                  | 容器的暴露端口 / Docker 网络                                | Service 提供稳定访问入口，隐藏 Pod 数量和变化。相当于在 Docker 里 -p 8080:80，或者建一个网络让容器互联。            |
| Namespace                | Docker 命名空间 / Compose project name                 | Namespace 用来隔离不同应用的资源，就像在 Docker Compose 里用 -p myproject 给一组容器打标签分组。            |
| ConfigMap                | 挂载配置文件 / 环境变量                                      | ConfigMap 存放应用配置，类似 docker run -e VAR=value 或者挂载 -v config:/etc/app/config。     |
| Secret                   | Docker Secrets（Swarm 模式）                           | Secret 存放敏感信息（密码、密钥），相当于 Docker Swarm 里的 secrets，或者用环境变量传递敏感信息。                 |
常用的命令如下：
```YAML
## 查看集群信息
## https://www.yuque.com/atguigu-team/frzi7z?# 《k8s专区》 密码：trk3
# 本地有哪些 context（即集群 + 用户 + 命名空间的组合）
kubectl config get-contexts
kubectl config use-context minikube
# 相当于打电话给物业问一下：“我们小区的基本情况是啥？”
# 比如管理中心地址，DNS 地址等。
kubectl cluster-info
# 查看节点信息
kubectl get nodes -o wide

## 创建pod docker run
kubectl run mynginx --image=nginx
# pods的详细详细
kubectl describe pod <pod-name>
# 删除
kubectl delete pod <pod-name>
# docekr ps
kubectl get pods -A
# 同一类标签的pods
kubectl get pods -n kube-system

# 查看命名空间
kubectl config view --minify
# 命名空间
kubectl get ns
# service
kubectl get svc
# 配置端口转发
kubectl port-forward svc/foo-service 5678:5678
# 创建 pod或者pod和其对应的service
kubectl apply -f <file.yaml>
kubectl delete -f <file.yaml>
kubectl delete pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl exec -it <pod-name> -- /bin/bash
# 监控
kubectl get pods -w
# 导出设计图
kubectl get deployment <deployment-name> -o yaml > deployment.yaml

# 重启命名空间下
kubectl rollout restart deployment -n higress-system
# 重装
kubectl delete pod --all -n higress-system

kubectl config use-context your-context
kubectl config set-context your-context --namespace=your-namespace

# 配置
kubectl -n higress-system get cm higress-config -o yaml > higress-config.yaml
kubectl -n higress-system apply -f higress-config.yaml

kubectl -n higress-system describe cm higress-config
kubectl -n higress-system edit cm higress-config
```
## helm
### Helm 和 YAML 区别
|   |   |   |
|---|---|---|
|类比角色|YAML 文件手动部署（kubectl apply -f）|Helm 安装（helm install）|
|生活类比|你亲自画设计图，找工人，一步步盖房|找专业装修公司：套餐装修，拎包入住|
|操作方式|自己写好每个 YAML，手动 apply|命令一行，模板化部署，参数化安装|
|灵活性|灵活，但需要你写所有配置文件|默认模板好，灵活修改参数即可|
|维护升级|升级麻烦，需要自己改 YAML 文件|Helm 可以一键升级、回滚|
|重复部署|多环境部署麻烦，要复制很多 YAML|Helm 直接通过参数化一键搞定|
|成熟度|初学者容易上手，适合学习原理|生产环境建议使用，自动化更强|
---
#### 通俗场景：
#### ✅ YAML 手动部署：
就像你自己做蛋糕 🍰
- 面粉、糖、鸡蛋都要自己买好。
- 做出来有成就感，但要花很多时间。
- 每做一次都要从头来一遍。
#### ✅ Helm 安装：
就是你去蛋糕店买现成的 🎂
- 你只用说口味，几人份，店家全帮你做好。
- 有模板化配方，还能快速多次复用。
- 后期加配料、升级版本很方便。
---
针对你问的：部署 Higress
1. 如果你用 YAML：
你需要手动下载好所有 Higress 的相关 YAML 文件：
- CRD（自定义资源定义）
- Deployment
- Service
- ConfigMap
- RBAC
- … 还有相关的监控、网关配置
执行：
```Bash
kubectl apply -f crds.yaml
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
...
```
2. 用 Helm：
Helm 已经帮你准备好了所有模板，你只需要写参数：
```Bash
helm install higress -n higress-system higress.io/higress \
  --create-namespace \
  --render-subchart-notes \
  --set global.local=true \
  --set global.o11y.enabled=false
```

相当于你告诉 Helm：

> “我需要一个本地模式的 Higress，不用监控模块，顺便帮我创建命名空间哈！”

Helm 会自动帮你生成对应的 YAML 文件并 apply。

---

那，最关键的区别总结一句话：

> **YAML 是手动盖房，Helm 是套餐化自动盖房。**

- 学习阶段用 YAML 能帮助你理解底层资源关系。
    
- 实际生产、企业部署推荐用 Helm，省时省力，升级维护方便。
    

---

附赠：如果你用 Helm 安装 Higress，后续还能：

- **查看安装内容：**
    

```Bash
helm get manifest higress -n higress-system
```
- 👉 这会打印出实际 apply 的 YAML 文件，想学 YAML 的话，这招很有用！
- **升级版本：**
```Bash
helm upgrade higress higress.io/higress -n higress-system --set xxx=xxx
```
- **卸载清理：**
```Bash
helm uninstall higress -n higress-system
```
---
结论总结
|   |   |
|---|---|
|场景|推荐使用|
|学习 Kubernetes 基础，理解资源关系|YAML|
|快速搭建环境，避免出错，易维护|Helm|
|生产环境，多集群多环境管理|强烈推荐 Helm|
---
### 使用helm安装higress
```Bash
helm repo add higress.io https://higress.io/helm-charts
helm install higress -n higress-system higress.io/higress \
  --create-namespace \
  --render-subchart-notes \
  --set global.local=true \
  --set global.o11y.enabled=false
```
https://github.com/helm/helm/releases
### 常用命令
# 查看已安装发布
helm list -A
```
# higress
https://higress.cn/docs/latest/user/quickstart/
## hgctl
使用kubernetes暴露higress-gateway的端口
```Bash
# 将 K8s 集群内的端口映射出来
kubectl port-forward service/higress-gateway -n higress-system 80:80 443:443
```
用于负载k8s中的服务
```Bash
# nacos注册
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=ip&groupName=ip&ip=ipinfo.io&port=80&ephemeral=false'
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=amap&groupName=amap&ip=restapi.amap.com&port=80&ephemeral=false'
curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=fwjqr&groupName=fwjqr&ip=10.10.138.202&port=9011&ephemeral=false'
curl -X DELETE 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=fwjqr&groupName=fwjqr&ip=10.10.138.202&port=9011&ephemeral=false'
# 启动可视化的higress配置界面
hgctl dashboard console
```
## MCP
kubectl -n higress-system edit cm higress-config
```YAML
apiVersion: v1
data:
  higress: |-
    mcpServer:
      sse_path_suffix: /sse
      enable: true
      redis:
        address: 10.10.138.202:6379
      match_list:
        - match_rule_domain: "*"
          match_rule_path: /registry
          match_rule_type: "prefix"
      servers:
        - name: nacos-registry
          type: nacos-mcp-registry
          path: /registry
          config:
            serverAddr: 10.10.138.202
            namespace: ""
            serviceMatcher:
              amap: ".*"
              ip: ".*"
              fwjqr: ".*"
    downstream:
      connectionBufferLimits: 32768
      http2:
        initialConnectionWindowSize: 1048576
        initialStreamWindowSize: 65535
        maxConcurrentStreams: 100
      idleTimeout: 180
      maxRequestHeadersKb: 60
      routeTimeout: 0
    upstream:
      connectionBufferLimits: 10485760
      idleTimeout: 10
```