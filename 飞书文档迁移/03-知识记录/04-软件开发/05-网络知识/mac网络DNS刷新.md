---
created: '2024-06-24 17:17:38'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/WnbxdUdOwoJAL9xRN8PceKBtnBc
modified: '2024-08-05 14:17:08'
source: feishu
title: mac网络DNS刷新
---

mac网络DNS刷新
sudo killall -HUP mDNSResponder
sudo dscacheutil -flushcache

