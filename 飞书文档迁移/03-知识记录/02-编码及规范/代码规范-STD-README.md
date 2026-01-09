---
created: '2024-05-15 17:25:35'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/VrXKdUuvmoGOAYxBYZuctDGOn9b
modified: '2024-05-29 10:10:11'
source: feishu
title: 代码规范-STD-README
---

代码规范-STD-README
项目介绍
AICC智能呼叫中心
目录介绍
aicc-main：aicc主体服务，包括aicc平台、aicc应用
aicc-app：aicc应用服务，包含联系计划管理、客户信息管理、坐席问答记录、坐席班表功能、特殊客户管理（VIP黑名单）、服务小结等
aicc-palt：aicc应用平台服务，包含对外坐席服务、外呼接口、坐席状态控制、系统用户管理、坐席联系历史、录音及文本查看、服务路由、坐席路由、路由日志、排队管理、被叫号码设置、IVR流程管理、提示音管理、坐席技能组信息、坐席技能组人员管理、系统配置、运维配置等
aicc-web：aicc主体web，主体web框架以及平台、应用服务的web界面
aicc-sjyy-all：单产品数据运营
aicc-aicsdos-dev：数据运营aicsdos的后台服务
aicc-data-integration-dev：aicc指标开发的jar任务文件
aicc-im-integration-dev：IM指标开发的jar任务文件
aicc-jelly-dev：aicc指标开发的jar任务文件（第一版没有数仓建模）
aicc-jobcontainer-dev：数据运营的任务调度系统
aicc-sda-dev：数据运营报表和指标数据存储后台服务
aicc-sjyy-sql：指标计算和应用初始化sql
aicc-storage-dev：数据运营录音存储后台服务
aicc-thirdparty：aicc集成三方系统
aicc-aicsdos-web：aicc集成数据运营aicsdos界面
aicc-audiostorage-dev：aicc主体服务音频存储服务
aicc-im-dev：IM文字客服服务
aicc-im-web：IM文字客服前端
aicc-msg-dev：短信平台服务
aicc-msg-web：aicc集成短信界面
aicc-rz-web：aicc集成融智界面
aicc-workflow-web：aicc集成讯飞工作流界面
服务目录规范：单体服务必须要有以下目录：aicc-xx-dev、aicc-xx-dev-db、aicc-xx-web
版本控制
image.png

GIT/SVN提交规范：不允许仅提交代码产物，git上不提交文档、集成产物在提交版本控制系统时，仅提交代码，添加commit日志时，添加日志的规范如下：
build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交
ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle等)的提交
docs：文档更新
feat：新增功能
fix：bug 修复
perf：性能优化
refactor：重构代码(既没有新增功能，也没有修复 bug)
style：不影响程序逻辑的代码修改(修改空白字符，补全缺失的分号等)
test：新增测试用例或是更新现有测试
revert：回滚某个更早之前的提交、
chore：不属于以上类型的其他类型(日常事务)
例如：fix:空值针异常修复
# 正则校验
^(fix|build|ci|docs|feat|perf|refactor|style|test|revert|chore):\[\d*\].*
开发过程
数据库
新增表结构时，根据各个模块功能规范命名、例如seat相关数据表：seat_业务名，acd相关数据表：acd_业务名，坐席业务相关：seatbiz_业务名，系统配置相关：sys_业务名
所有数据库表必须有以下基础字段：
id（UUID）
create_time
update_time
create_by（创建人。可为空）
update_by（更新人。可为空）
org_code(组织机构编码。可为空)
pom依赖
新增pom依赖，需要在最外层pom声明依赖，在common模块引入依赖。业务模块不做任何依赖引入
模块新建
模块新建时；除非必要，无须再新建子模块\子包，模块下各个包名命名规范如下
constants：常量
bean\entity: 一般java实体
dao：数据库映射实体类
vo: 接口请求出入参实体
api：服务对外接口SDK
service：服务内部接口
impl：服务接口实现类
mapper：数据库SQL逻辑
controller：业务控制类
代码规范(参考阿里规范)
 
Java开发手册(黄山版).pdf
系统已经集成了swagger、以及标准化返回，新增接口时，注意添加swagger注释及使用标准化返回，便于调试
common模块中定义标准错误码，用于错误查询(整理错误码)
接口书写风格 Restful风格
使用alibaba代码规范插件，扫描代码，规范基础代码编写
测试集成
版本号：根据提测版本修改当前项目中的pom中的，将版本信息改为对应的提测版本号，检查代码规范
标准提测版本： 1.0.X-SNAPSHOT、标准稳定版本： 1.0.X-RELEASE
定制提测版本： V1.0.X_M_YX1.0.X-SNAPSHOT、稳定版本： V1.0.X_M_YX1.0.X-RELEASE，M：是基于哪个标准版本做的定制，YX1.0.X：是局点名称缩写以及当前定制版本信息
数据库
索引
数据库的所有业务表，必须需要有除UUID外的业务唯一索引！！！
唯一索引：表名_uk_索引名（字段名全小写无下划线，表名可缩写）
主键索引：表名_pk_索引名
普通索引：表名_idx_索引名
数据库文件
！！！必须要维护数据库设计文件，使用PDMANER
当前版本首次提测，根据功能修改点，提供全量、增量sql脚本（试行Q2）。DB提测目录以版本名命名，例如：
正式包：AICC_V1.0.3，AICC_V1.0.4
正式补丁包：AICC_V1.0.3.1
定制包：AICC_V1.0.X_M_YX1.0.X
定制补丁包：AICC_V1.0.X_M_YX1.0.X
每个版本AICC应用需要有以下SQL脚本
全量AICC应用脚本
全量UAP脚本
增量AICC应用脚本
增量UAP脚本
特殊脚本（如果有）
且AICC应用的SQL要求可以重复执行，且重复执行不报错。增量SQL模板见SVN。脚本中严禁出现DROP语句！！！
提测流程
普通配置：类似数据库连接地址
在application.properties新增配置
系统配置：
类似座席挂机后调整置闲时长：输入型
挂机是否自动转满意度等：选择型
在sys_config新增配置项：config_type=0:代表输入型，config_type=1:代表选择型，选择性，需要在sys_sub_config新增子项
每个版本构建负责人跟踪当前版本
每个功能修改人更新resource目录的version.txt，包括当前的版本更新信息、构建版本号、提测版本号、版本变动点
每个功能修改人更新当前版本部署文档！！！，更新各自功能的自测说明
提测流程规范，研测需要关注公司质效平台流程，保证每个版本流程正常结束
持续集成
版本TAG
${GIT_BRANCH}/${BUILD_SID}
构建
mvn clean -Dmaven.test.skip=true  -Dbuild.sid=Branch-${GIT_BRANCH}-${BUILD_SID} install -pl aicc-plat/aicc-plat-core -am
