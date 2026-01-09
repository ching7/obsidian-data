---
created: '2024-08-29 09:45:41'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/K9wYdimJjoLky5xekLIc7Yrcni1
modified: '2025-01-27 00:23:51'
source: feishu
title: arthas
---

arthas
trace -E 类 方法 -n 30  "#cost>5000"
jstack -l <Java进程PID> > thread_dump.txt

jmap -dump:format=b,file=heap_dump.hprof <PID>

 使用 jcmd
生成线程转储：
bash
复制编辑
jcmd <JVM_PID> Thread.print > thread_dump.txt
生成堆转储：
bash
复制编辑
jcmd <JVM_PID> GC.heap_dump heap_dump.hprof
分析方法：
对于 heap_dump.hprof 文件，直接用 JProfiler 或 VisualVM 加载分析。
对于线程转储文件（thread_dump.txt），可以结合工具如 FastThread.io 或手动分析。
明日代办
磁盘清理，预留春节（10天）期间可用（20、21等）
外呼 100
软交换 100
抓包 100
IM问题跟踪
目前线程趋于稳定
image.png

临时问题版本包准备，先讯飞环境测试。如果明天出现同样的偶现问题，临时升级
1、由于上午业务高峰期出现im历史坐席优先查询导致数据库io占满问题导致系统卡顿，首先响应将全部渠道历史坐席优先策略暂时先关闭了，数据库恢复正常，但im依然出现频繁错误现象
2、初步排查怀疑可能为数据库依然查询缓慢，数据库连接无法释放等问题导致，对im长时间未释放连接进行处理后重启应用，初步缓解了部分压力，但依然有串线，坐席无法进线问题
3、重启一段时间后出现im出现进线缓慢问题，页面表现为socket无法连接或需要重试多次才能完成连接，排查可能为连接数满了，修改配置增加连接数并重启应用，暂时部分恢复业务，但依然有部分坐席无法进线及无访客无法签出问题
4、业务进行一段时间后，又重复出现问题3现象，dump应用jvm日志后排查可能为卡顿时出现部分数据错误导致redis锁在部分情况下一直无法释放并循环查询，占满线程，针对此现象修复了错误数据，并重启应用，暂时恢复业务，待继续观察
针对该问题需进线如下优化
1、针对历史坐席优先查询时间过长问题优化sql查询及索引,减少查询时间
2、查询出现错误数据原因并针对redis锁无法释放问题增加最大尝试次数，超过次数释放锁，以防止锁一直无法释放问题
