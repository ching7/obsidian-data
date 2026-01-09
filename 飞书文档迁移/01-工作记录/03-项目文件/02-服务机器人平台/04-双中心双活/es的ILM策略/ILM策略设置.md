---
created: '2025-01-22 10:45:22'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/YewadbMZWoKtlJxwfPJc7gC1nId
modified: '2025-01-22 11:29:53'
source: feishu
title: ILM策略设置
---

ILM策略设置
Elasticsearch的索引生命周期管理（ILM）是一种自动化管理索引从创建到删除整个生命周期的功能。它允许用户定义策略，自动执行索引的滚动、缩小、分配、只读设置、强制合并分段以及最终的索引删除等操作。
ILM的主要阶段和操作
Hot 阶段：
索引被频繁写入和查询的阶段。
可以执行的操作包括增加副本以提高查询性能，以及触发滚动（rollover）以创建新索引。
Warm 阶段：
索引不再频繁写入，但仍可能被查询的阶段。
可以减少副本数量以节省资源，或者将索引迁移到成本更低的存储介质上。
Cold 阶段：
索引很少被查询，主要用于归档和合规性目的。
可以进一步减少副本数量，或者将索引迁移到更冷的存储层。
Delete 阶段：
索引不再需要，可以被删除的阶段。
索引将被永久删除。
ILM的实际应用例子
假设你运营一个日志分析系统，每天生成大量的日志数据。你可以使用ILM来自动化管理这些日志索引的生命周期。
定义ILM策略：
定义一个策略，当索引达到30天或50GB时，触发滚动操作创建新索引。
在Warm阶段，减少副本数量到1，以节省资源。
在Cold阶段，将索引迁移到成本更低的存储介质上。
在索引达到90天时，自动删除索引。
JSONCopy
PUT _ilm/policy/logstash_policy
{
        "policy": {
                "phases": {
                        "hot": {
                                "min_age": "0ms",
                                "actions": {
                                        "rollover": {
                                                "max_size": "50gb",
                                                "max_age": "30d"
                                        }
                                }
                        },
                        "warm": {
                                "min_age": "30d",
                                "actions": {
                                        "allocate": {
                                                "number_of_replicas": 1
                                        }
                                }
                        },
                        "cold": {
                                "min_age": "60d",
                                "actions": {
                                        "allocate": {
                                                "number_of_replicas": 0
                                        }
                                }
                        },
                        "delete": {
                                "min_age": "90d",
                                "actions": {
                                        "delete": {}
                                }
                        }
                }
        }
}
创建索引模板并关联ILM策略：
创建一个索引模板，将ILM策略应用到所有日志索引上。
设置索引别名，以便统一对外提供服务。
JSONCopy
PUT _index_template/logstash_template
{
        "index_patterns": ["logs-*"],
        "settings": {
                "index": {
                        "lifecycle": {
                                "name": "logstash_policy"
                        }
                }
        },
        "aliases": {
                "logs-write": {}
        }
}
写入日志数据：
每天写入日志数据到logs-write别名。
当索引满足滚动条件时，Elasticsearch将自动创建新索引，并将别名指向新索引。
监控和管理：
定期监控索引的状态和性能。
根据需要调整ILM策略和索引模板。
如果你的索引是 aicc-test，别名是 aicc-test-cur，要确保在 rollover 时别名能够自动更新到最新的索引，可以通过 索引生命周期管理（ILM） 或 手动操作 来实现。下面详细解释如何操作。
1. 使用索引生命周期管理（ILM）自动更新别名
在使用 ILM 策略的情况下，rollover API 会自动更新别名。ILM 会根据你设定的策略（如 max_size 或 max_age）创建新的索引，并且在新的索引创建时，别名会自动指向最新的索引。
步骤 1：创建 ILM 策略
首先，你需要为索引 aicc-test 创建一个 ILM 策略。在这个策略中，你可以设置条件（如大小、时间）来决定何时进行索引滚动。
PUT /_ilm/policy/aicc-test-rollover-policy
{
        "policy": {
                "phases": {
                        "hot": {
                                "actions": {
                                        "rollover": {
                                                "max_size": "50gb",
                                        }
                                }
                        }
                }
        }
}

步骤 2：创建索引模板并绑定 ILM 策略
然后，为索引 aicc-prd 创建模板，并绑定刚才创建的 ILM 策略。该模板确保所有符合 aicc-prd-* 模式的索引都采用此策略。
PUT /_template/aicc-test-template
{
        "index_patterns": ["aicc-test-*"],
        "settings": {
                "index.number_of_shards": 3,
                "index.number_of_replicas": 2,
                "index.lifecycle.name": "aicc-test-rollover-policy"
        },
        "mappings": {
                "properties": {
                        "businessTagLists": {
                                "properties": {
                                        "id": {
                                                "type": "keyword"
                                        },
                                        "name": {
                                                "type": "keyword"
                                        },
                                        "path": {
                                                "type": "keyword"
                                        }
                                }
                        },
                        "callDuration": {
                                "type": "long"
                        },
                        "callEndTime": {
                                "type": "date",
                                "format": "yyyy-MM-dd HH:mm:ss"
                        },
                        "callId": {
                                "type": "keyword"
                        },
                        "callIn": {
                                "type": "boolean"
                        },
                        "callPhoneNo": {
                                "type": "keyword"
                        },
                        "status": {
                                "type": "keyword"
                        },
                        "duty": {
                                "type": "keyword"
                        },
                        "callStartTime": {
                                "type": "date",
                                "format": "yyyy-MM-dd HH:mm:ss"
                        },
                        "createTime": {
                                "type": "date",
                                "format": "yyyy-MM-dd HH:mm:ss"
                        },
                        "customerName": {
                                "type": "keyword"
                        },
                        "explicitNumber": {
                                "type": "keyword"
                        },
                        "hangUpReason": {
                                "type": "keyword"
                        },
                        "id": {
                                "type": "keyword"
                        },
                        "manualservice": {
                                "type": "keyword"
                        },
                        "myd": {
                                "type": "keyword"
                        },
                        "description": {
                                "type": "keyword"
                        },
                        "phoneCode": {
                                "type": "keyword"
                        },
                        "queueDuration": {
                                "type": "long"
                        },
                        "queueTime": {
                                "type": "keyword"
                        },
                        "repeatCall": {
                                "type": "boolean"
                        },
                        "ringTime": {
                                "type": "long"
                        },
                        "seatNo": {
                                "type": "keyword"
                        },
                        "skillGroupId": {
                                "type": "keyword"
                        },
                        "skillGroupName": {
                                "type": "keyword"
                        },
                        "text": {
                                "type": "text",
                                "fields": {
                                        "keyword": {
                                                "type": "keyword",
                                                "ignore_above": 8192
                                        }
                                }
                        },
                        "vipFlag": {
                                "type": "keyword"
                        }
                },
                "aliases": {
                        "aicc-test": {}
                },
                "meta": {
                        "description": "Template for aicc-prd indices",
                        "author": "Admin IFLYTEK",
                        "date": "2025-01-22"
                }
        }
}
}

步骤 3：创建初始索引并设置别名
在创建初始索引时，为其创建别名 aicc-test。
POST /aicc-test-2025.01.16
{"aliases": 
    {"aicc-test": 
        {}
    }
}
// 别名 aicc-prd-cur 指向当前的活动索引

此时，aicc-test 别名指向 aicc-test-2025.01.16 索引。
步骤 4：使用 rollover 自动滚动索引
ILM 策略会自动触发 rollover 操作。如果索引 aicc-test-2025.01.16 的大小达到 50GB 或时间超过 1 天，ILM 会自动创建新索引（如 aicc-test-2025.01.17），并自动将别名 aicc-test 指向新索引。
此过程无需手动干预，ILM 会自动更新别名。
总结
在使用 ILM 时，rollover 和别名更新是自动执行的。ILM 会根据策略条件滚动索引，并自动将 aicc-prd-cur 别名指向新的索引。你无需手动更新别名或触发 rollover，这整个过程是自动化的。
2. 如果不使用 ILM（手动操作）
如果你没有启用 ILM，依然可以手动管理 rollover 和别名更新。此时，rollover 需要手动触发，别名更新也需要手动操作。
步骤 1：执行 rollover
当你执行 rollover 操作时，索引 aicc-test-2025.01.16 满足滚动条件（如超过大小或时间限制），新的索引 aicc-test-2025.01.17 会被创建：
POST /aicc-test/_rollover
步骤 2：手动更新别名
rollover 会创建新索引，但不会自动更新别名。因此，你需要手动更新别名 aicc-prd-cur，使其指向新创建的索引：
POST /_aliases
{
        "actions": [{
                "remove": {
                        "index": "aicc-test-2025.01.16",
                        "alias": "aicc-test"
                }
        }, {
                "add": {
                        "index": "aicc-test-2025.01.17",
                        "alias": "aicc-test"
                }
        }]
}

步骤 3：定时任务自动化（可选）
如果你希望自动化这一过程，可以使用定时任务（如 Cron 作业）来定期触发 rollover 和别名更新操作。定时任务可以每天触发，确保滚动索引和别名更新。
总结
使用 ILM：rollover 和别名更新是自动执行的。ILM 会管理索引滚动并自动将别名指向新索引。
不使用 ILM：你需要手动触发 rollover 操作并更新别名，或者通过定时任务自动化这些操作。
因此，如果希望完全自动化 rollover 和别名更新，建议启用 ILM，它可以无缝地处理这一过程。

