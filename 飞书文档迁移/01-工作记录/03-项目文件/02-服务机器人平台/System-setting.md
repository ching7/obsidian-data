---
created: '2025-05-08 14:16:06'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/W5CNdt5thosrepxly4bcuViqnof
modified: '2025-05-08 15:49:53'
source: feishu
title: System-setting
---

System-setting
背景
cbb-search，检索CBB组件（改成问答CBB）
需求：cbb-search合并到不同应用中？取除jar包
功能：
对接docqa代理？
QA抽取、采编
一问多答
标签问答
还提供给三方的使用？理解引擎
代办
需求：system-setting组件的剥离知识库、机器人、cbb-search的引用
目前有三个方案：
依赖的知识库、机器人API接口做异常保护，保证独立部署system-setting组件时，组件功能正常不报错
将依赖知识库和机器人的功能做功能前置和独立入口
移除system-setting的cbb-search的引用
结论：
移除system-setting的cbb-search的引用不依赖整体的cbb-search改造
system-setting的cbb-search的引用改成common-engine
目前评估的方案和可行性
方案组合：1必做，单独2+3不可行
1+3（约7人日，1的rest接口改造+ 内部接口改造共约5人日，3的2人日依赖调整）
1+2+3（约10人日，1的5人日、2的3人日、3的2人日）同时涉及前端功能调用前置

