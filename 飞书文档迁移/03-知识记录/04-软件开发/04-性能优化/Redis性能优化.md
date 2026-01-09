---
created: '2024-04-11 15:51:55'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/B2S9dA2xco0nXgx8XiCc4bEOn6f
modified: '2024-04-15 15:15:46'
source: feishu
title: Redis性能优化
---

Redis性能优化
Redis 提供了几个与过期键扫描相关的配置参数，通过调整这些参数可以影响过期键的清理效率和对服务器性能的影响。以下是主要的过期扫描参数及其设置方法：
hz 参数
hz 参数决定了 Redis 服务器每秒执行的后台任务（如过期键扫描、AOF/WAL 日志同步等）的频率。默认值为 10，表示每秒执行 10 次。增大该值可以提高过期键的清理效率，但会增加 CPU 使用率。
设置方法（在 Redis 配置文件 redis.conf 中修改或使用 CONFIG SET 命令）：
# 在 redis.conf 中添加或修改
hz 20 # 示例值，可根据实际情况调整

# 或者使用 CONFIG SET 命令（运行时动态修改，重启后失效）
redis-cli CONFIG SET hz 20

