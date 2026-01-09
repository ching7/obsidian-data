---
created: '2025-03-03 11:19:17'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/M63qdVUfUo9qfVxp8Koc5TEKned
modified: '2025-03-04 17:14:13'
source: feishu
title: JDK降级与可观测版本升级注意事项
---

JDK降级与可观测版本升级注意事项
JDK17降级到JDK8
目前以下服务的JDK版本已经从JDK17降级到JDK8

注意事项
调整非国产化下dockerfile，将JDK依赖改为8
JDK降级到8以后，部分JAVA语言特性不再支持，需要调整为JDK8的格式，存量代码已经更新，后续新增代码需要各位研发同事注意
1. java.io.Serial;无法使用，jdk14引入。删除改为手动声明序列化
2. List.of/Map.of无法使用，jdk9引入，换成Arrays.asList,手动声明map
3. Stream.toList() 方法是在 JDK 16引入，换成Stream.collect(Collectors.toList())
4. msg instanceof FullHttpResponse response不支持，换成显式的强制类型转换(FullHttpResponse 5. response = (FullHttpResponse) msg)
6. JDK 11 开始，StringBuilder 类才引入了 isEmpty()，改用.length()判断
7. URLEncoder.encode(param.getKey(), StandardCharsets.UTF_8.name())，改用.name()判断
8. switch写法调整为jdk8支持的方法
9. StringBuilder 无isEmpty方法，jdk11才引入，使用.length() == 0替换
\s转义符不支持，jdk15才支持，jdk8换成\\s

未涉及的适配改动点，请参考编译器的报错提示！！
删除的V1.1不再使用的模块，同时调整了部分代码的依赖
1. 删除系统中对于：srp-robot-manager、srp-dialog-design模块的依赖
2. system-setting的VariableDubboService已替换为VariableServiceApi
ibuild的构建jdk版本，需要改为JDK1.8
如果需要远程debug，需要调整start.sh中的远程端口的写法
-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
对接可观测平台
目前V1.1的所有服务已经接入研发环境的可观测平台，主要接入的功能有2点：
对接可观测平台的链路日志
对接可观测平台的日志
研发环境可观测账号信息：http://172.29.234.92:8081/login，admin/star@1q2w
注意事项
集成可观测的过程中，发现部分组件代码无bin目录下的compose.yml、restart_container.sh，目前参考服务器上已有的内容进行补充。同时调整了构建的build.xml，增加打包文件
已有服务调整compose.yml
其中version版本号的调整适配V1.1的版本号
增加可观测平台对接的配置，默认关闭
调整非国产化下dockerfile，删除开源的skywalking-agent依赖（改为公司版本需要手动安装）
调整logback.xml，将日志输出改为公司skywalking-agent的格式（此处强依赖公司apm-toolkit-logback-1.x的pom依赖）如果无须对接，可以调整日志输出的格式layout
由于对接了可观测平台，日志位置：默认在logs/default.log中

