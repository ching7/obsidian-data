---
created: '2024-10-23 11:21:11'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/HvVEd7XFIodX28xR7cAcrkxYnPe
modified: '2024-10-25 15:36:02'
source: feishu
title: 使用sping-ai-alibaba作为事业部技术选型的分析报告
---

使用sping-ai-alibaba作为事业部技术选型的分析报告

目标
是否适合作为事业部的技术选型？
11月初给个分析报告，
在事业部架构师范围做分享
一、 概述
什么是spring-ai-aibaba
https://sca.aliyun.com/docs/ai/concepts/

https://mp.weixin.qq.com/s/vCjo1m58InZamd7Wzy9ZJg
使用spring-ai 搭建rag应用
https://docs.spring.io/spring-ai/reference/index.html

https://mp.weixin.qq.com/s/vIG2KzMV30P9yHryfJkOrQ
二、什么是spring-ai

Spring AI 提供了抽象能力，把以下这些能力做了抽象，具有多种实现，支持以最少的代码更改轻松交换组件。
支持文本、语音、图像聊天以及同步、流程api接口访问各类ai大模型：星火大模型
支持各类大模型的：星火大模型多模态
chat completion：完整的对话流程
Embedding ：向量化
Text to image：文本生成图片
Audio Transcription：音频转换文本
Text to Speech：文本转化成语音
Moderation：ai生成内容审核：大模型能力平台-内容审核
支持将大模型输出结构化：大模型能力平台
支持向量化各类数据库Vector Store：docqa
支持向量化数据库的API-SQL：docqa
支持向量化数据库的自动配置：springboot
支持自定义工具/函数链Function Calling：星链
支持AI能力监测：可观测平台，或者是能够监测大模型状态的服务
支持文档ETL：docqa 知识管理\知识采编
支持模型调优AI Model Evaluation ,模型效果评估、模型调优：内核引擎
支持webclient Fluent api 调用：springboot webclient
支持模型封装advisors apis：服务机器人的机器人管理/大模型能力平台的助手管理
支持对话上下文记忆（chat conversation memory ）,RAG 搜索增强：大模型能力平台的助手管理/服务机器人平台知识库功能

三、什么是spring-ai-alibaba
开发复杂 AI 应用的高阶抽象 Fluent API — ChatClient：springboot webclient
提供多种大模型服务对接能力，包括主流开源与阿里云通义大模型服务（百炼）等各类ai大模型：星火大模型
支持的模型类型包括聊天、文生图、音频转录、文生语音等：星火大模型多模态
支持同步和流式 API，在保持应用层 API 不变的情况下支持灵活切换底层模型服务，支持特定模型的定制化能力（参数传递）：星火大模型流式接口
支持 Structured Output，即将 AI 模型输出映射到 POJOs：大模型能力平台
支持矢量数据库存储与检索：docqa
支持函数调用 Function Calling：星链
支持构建 AI Agent 所需要的工具调用和对话内存记忆能力；大模型能力平台的助手管理
支持 RAG 开发模式，包括离线文档处理如 DocumentReader、Splitter、Embedding、VectorStore 等，支持 Retrieve 检索：服务机器人平台知识库功能

image.png

四、类比分析
Chat Client=openapi
对话模型(Chat Model)
嵌入模型 (Embedding Model)
工具(Function Calling)
对话记忆（Chat Memory）
提示词 (Prompt)
文档检索 (Document Retriever)
格式化输出(Structured Output)
向量存储(Vector Store)

五、结论





附件

输入 @ 插入相关文档


