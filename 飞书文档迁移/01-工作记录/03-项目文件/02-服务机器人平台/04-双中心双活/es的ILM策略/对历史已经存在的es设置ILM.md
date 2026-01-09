---
created: '2025-01-22 11:30:01'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/MIAJdKV6noYp1gxk11Tcb0HXnlc
modified: '2025-01-22 15:18:18'
source: feishu
title: 对历史已经存在的es设置ILM
---

对历史已经存在的es设置ILM
如果已有的索引 aicc-test 没有日期后缀，并且希望将其纳入 ILM 策略进行管理，最优方案是手动进行数据迁移和索引别名调整，以适配 ILM 的 rollover 要求。以下是具体步骤：

步骤 1：手动创建第一个符合 ILM 的索引
因为 ILM 的 rollover 必须绑定到一个别名上，且该别名需要指向一个启用了 is_write_index 的索引，因此你需要创建一个新的索引（带日期后缀），并将旧索引的别名迁移至新索引。
示例：
PUT /aicc-test-000001

{"aliases": {"aicc-test": {"is_write_index": true}}}
这里：
aicc-test-000001 是新索引，带有版本号或日期后缀。
别名 aicc-test 作为写索引别名，绑定到新索引。

步骤 2：将旧索引的数据迁移到新索引
方法 1：使用 Elasticsearch 的 _reindex API
将 aicc-test 索引中的数据迁移到 aicc-test-000001：
POST /_reindex
{"source": {"index": "aicc-test"},"dest": {"index": "aicc-test-000001"}}
迁移完成后，所有数据会被复制到 aicc-test-000001。
方法 2：通过外部工具迁移（如 Logstash 或 Beats）
如果数据量较大且需要对数据进行变更或过滤，可以使用 Logstash、Filebeat 等工具重新导入数据到新索引。

步骤 3：更新旧索引的别名设置
将旧索引 aicc-test 的别名解绑，同时确保新索引 aicc-test-000001 保持别名 aicc-test。
解绑别名：
POST /_aliases
{"actions": [{"remove": {"index": "aicc-test","alias": "aicc-test"}}]}
新索引保持别名（已在创建时完成）：
POST /_aliases
{"actions": [{"add": {"index": "aicc-test-000001","alias": "aicc-test","is_write_index": true}}]}

步骤 4：为新索引绑定 ILM 策略
为 aicc-test-000001 绑定 ILM 策略：
PUT /aicc-test-000001/_settings
{"index.lifecycle.name": "aicc-test-rollover-policy","index.lifecycle.rollover_alias": "aicc-test"}

步骤 5：清理旧索引（可选）
如果旧索引 aicc-test 不再需要，可以删除它以释放存储空间：
DELETE /aicc-test

步骤 6：后续自动 rollover 操作
当 aicc-test-000001 达到 ILM 策略中的条件（例如 50GB 或 1天），Elasticsearch 会自动：
创建新索引（如 aicc-test-000002）。
将别名 aicc-test 自动更新到新索引。
旧索引（aicc-test-000001）变为只读。

注意事项
数据一致性: 在迁移期间，暂停对 aicc-test 索引的写入操作，避免数据丢失。
性能影响: 大规模数据迁移可能会对集群性能产生影响，建议在业务低峰期执行。
索引结构一致性: 确保 aicc-test 和 aicc-test-000001 的映射一致。如果需要调整映射，可在迁移过程中更新。
测试环境验证: 在生产环境操作前，先在测试环境验证流程。
