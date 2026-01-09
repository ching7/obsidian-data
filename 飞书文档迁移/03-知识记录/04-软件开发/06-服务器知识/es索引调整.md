---
created: '2024-07-04 09:35:17'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/MMdtdza2nonsCRxXXHHcQEQincg
modified: '2024-07-04 09:35:50'
source: feishu
title: es索引调整
---

es索引调整
看下这两个配置：
elasticsearch.aicc.index.seatCallRecord.name=
elasticsearch.aicc.index.name=
然后，在对应的es执行下：curl -XPUT http://127.0.0.1:9200/aicc-yace/_settings -d '{ "index" : { "max_result_window" : 300000}}'
127.0.0.1 和 aicc-yace换成对应的索引和 ip
