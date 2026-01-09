---
created: '2025-03-18 10:13:46'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/RItPd8fE0oX6DRxigqqcgQYQnmd
modified: '2025-03-18 10:14:32'
source: feishu
title: 07-Docker命令
---

07-Docker命令
批量删除镜像

docker images | grep "produce" | awk '{print $3}' | xargs docker rmi

