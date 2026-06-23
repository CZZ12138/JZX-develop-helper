# JZX Workspace Memory

适用范围：`/Users/chengzuozheng/Desktop/jzx-workspace`

本文件是工作区长期记忆库，保存可复用的经验、规范、设施信息、排障结论和项目上下文。

`AGENTS.md` 负责高频、强约束、可执行规则；本文件负责持续沉淀。凡是后续仍可能复用的信息，应更新到这里，而不是只停留在对话里。

## Maintenance Rules

- 优先记录可复用结论，不记录纯一次性聊天内容
- 同类信息只保留一个最新版本，失效内容直接覆盖
- 尽量写清适用范围、背景、限制条件
- 敏感信息只记录入口、定位方式、使用规则，不直接写密钥、密码、AK/SK、Cookie、数据库真实连接串
- SLS project/logstore/container、Nacos dataId、NodePort、发布 tag 等细节可以写在 memory；普通业务代码仓库清单不要无限扩张，优先沉淀服务边界和排障口径
- 新出现的重要经验、开发规范、环境约定、排障结论，应及时补充到本文件

## Quick Index

- 工作区规则：`/Users/chengzuozheng/Desktop/jzx-workspace/AGENTS.md`
- 基建文档：`/Users/chengzuozheng/Desktop/jzx-workspace/公司基建_00_Overview_流水线集群日志经验.md`
- 基建合并版草稿：`/Users/chengzuozheng/Desktop/jzx-workspace/公司基建_00_Overview_流水线集群日志经验(2).md`
- spec 写作规范：`/Users/chengzuozheng/Desktop/jzx-workspace/engineering-spec-writing-guideline.md`
- 工单排障 skill：`/Users/chengzuozheng/.codex/skills/jzx-workorder-debugging`
- 测试集群/bugfix 操作入口：`/Users/chengzuozheng/Desktop/jzx-workspace/sdd-bugfix`
- 寒雪观测平台交接：`/Users/chengzuozheng/Desktop/jzx-workspace/hanxue-observability-platform/HANDOFF_2026-06-17_寒雪观测平台交接.md`

## Workspace Facts

- JZX 相关开发任务默认在 `/Users/chengzuozheng/Desktop/jzx-workspace` 工作区下进行
- 工作区级自动加载规则文件：`/Users/chengzuozheng/Desktop/jzx-workspace/AGENTS.md`
- 若子项目存在更近一级 `AGENTS.md`，其规则优先；若没有，再参考该项目的 `CLAUDE.md`
- `engineering-context` 是工作区级上下文入口，也是跨项目背景资料主来源
- `engineering-context/skills` 已同步到 `~/.codex/skills`，开发阶段按需加载并执行其中约定
- 开发流程默认遵循：需求边界澄清 -> 顶层架构 -> `spec -> plans -> execution`

## Git / Branching Rules

- 个人功能分支需要保持干净，不要为了处理合入 `test` 的冲突而把 `origin/test` merge 回个人功能分支
- 用户要求“更新远端分支 / 上传同步开发分支”时，默认只推送当前代码分支上已经提交的改动；本地工作文档、spec/overview/plan/summary、`docs/superpowers/` 等未跟踪文档不要自动提交或推送，除非用户明确要求同步这些文档
- 如果代码变更涉及建表、改表、加字段或索引，合入/部署测试环境时必须同步确认测试库 schema 已变更到位；不要只部署代码而遗漏表结构，否则会影响同一测试环境里的其他人。若自动迁移链路不明确，应显式检查目标测试库字段/表并补齐。
- 合并个人开发分支到 `test` 如果出现冲突，推荐流程：
  - 从 `origin/test` 拉一条临时合并分支，命名优先使用 `merge/test-<日期或需求标识>` / `merge/<日期>-merge-into-test`
  - 在该临时合并分支上 merge 个人功能分支并解决冲突
  - 默认只推送临时合并分支到远端，由用户在 Codeup 手动提 MR / 合并到 `test`
  - 除非用户明确说“直接合并/直接推 test”，不要由 Codex 直接推送 `test`
  - 个人功能分支继续只保留自己的功能提交，避免被 `test` 上的测试代码或其他未发布改动污染
- 如果误把 `test` merge 到个人功能分支，应先备份污染提交，再将个人功能分支回退到合入 `test` 前的最后一个功能提交；必要时使用 `--force-with-lease` 更新个人远端分支

## Reusable Prompts

需求理解阶段：

```text
这是一个 JZX 项目和 xx 功能相关的新需求，先尝试理解下整体需求需要做什么？不做什么？任务的方向和边界在哪里。
```

顶层架构阶段：

```text
接下来帮我产出顶层的架构设计，该顶层架构需要输出文档 `xxx_00_Overview_顶层架构`，这个文档中需要包括整体可能要涉及到改造的项目，以及改造点，它本质上就是一个简化版的面向技术的 PRD 说明。
```

顶层架构文档至少包含：

- 顶层模块和架构设计
- 模块边界与关系
- 新增模块
- 现有 JZX 仓库（服务 / 模块 / 端）改造范围

## Infrastructure / Release Rules

### Fact Sources

- DevOps 发布单只证明发布动作发生过；是否真实生效，以 K8s 当前 `image` / `ReplicaSet` 和 SLS 实际请求 `traceId/container/version` 为准
- `*-new` 表示生产灰度环境，`*-base` 表示生产全量环境
- 配置事实以目标环境 Nacos / Secret / Pod env 为准，不从旧文档或本地代码硬猜
- 业务状态以业务 DB 为准；SLS 证明事件发生，不等于业务状态闭环
- SLS 查询优先级：`traceId` / `EagleEye-TraceID` > 业务主键 / `roomId` / `sessionId` > `userId` / device > path + 时间窗

### Release Flow

- 日常工单 bug 发布流：先修复 -> 发测试流水线 -> 测试验证 -> 发预发 / 填预发发布单 -> 把发布单链接交给陈灵琪发生产
- 生产发布、生产 Nacos、生产重放请求默认不能由 Agent 自行操作，除非用户明确授权
- 验证灰度不能推导全量生效；验证全量不能只看灰度容器日志
- 同一流水线后续发布可能覆盖前一次 bugfix，排查复测结果时必须确认当前运行 tag
- 前端页面仍旧时，优先查流水线、镜像、JAR 静态资源 hash、浏览器加载 hash，而不是先改代码

### K8s / Cluster

- `sdd-bugfix` 保存访问集群所需的 key / kube 相关接入信息
- 在 `sdd-bugfix` 上下文中，`kubectl` 可以完成大多数集群操作
- 可以从远程服务器拉取开发代码，并部署到 K8s 集群进行测试
- 默认 kubeconfig：`.env/credentials/kubeconfig`
- 默认 namespace：`test`
- 适合作为联调、部署验证、集群问题排查入口
- 涉及真实环境操作时，先确认目标环境边界

### SLS Coordinates

工单排障优先使用 `jzx-workorder-debugging/references/sls-config.md`，该文件是当前自包含配置入口。

作业辅导/讲题常用测试环境：

| Field | Value |
| --- | --- |
| region | `cn-hangzhou` |
| project | `k8s-log-ce24edc82a57847669a3929be5d67edc0` |
| logstore | `test-app-log` |

生产环境：

| 环境 | 含义 | project | logstore |
| --- | --- | --- | --- |
| prod-base | 生产全量 | `k8s-log-c292481d697254c0cb6272d5a74f3c5c1` | `prod-app-log` |
| prod-new | 生产灰度 | `k8s-log-c292481d697254c0cb6272d5a74f3c5c1` | `prod-app-log` |

常见容器命名：

| 环境 | 容器命名 |
| --- | --- |
| test | `test-zstt-<service>` |
| prod-base | `prod-zstt-<service>-base` |
| prod-new | `prod-zstt-<service>-new` |

已验证：旧测试 project `k8s-log-cf194c581f4c848d8a051abe4c1b2a8de` 在作业辅导查询中曾返回 `401 The project does not belong to you`；作业辅导优先使用 `ce24...`。但寒雪观测平台历史事实源仍有使用 `cf194...` 的记录，具体场景按服务和日志可达性验证。

常用关键词：

- `RequestMonitor`
- `APPLICATION VERSION`
- `traceId`
- `EagleEye-TraceID`
- 接口路径
- 设备 ID / 用户 ID
- 业务错误码，如 `B0001`

## Workorder / Error Debugging

### Skill

- `jzx-workorder-debugging`：路径 `/Users/chengzuozheng/.codex/skills/jzx-workorder-debugging`
- 用于按工单、SLS 日志、trace/roomId/userId、截图和本地代码证据定位首错点，并输出证据驱动的简洁汇报
- 使用时先读：
  - `references/sls-config.md`
  - `references/evidence-standards.md`
  - 写正式报告时再读 `references/report-patterns.md`

### Evidence Rules

- 工单、报错、生产/测试日志排查默认先使用 `jzx-workorder-debugging` skill
- 排障结论必须区分：现象、日志证据、代码证据、运行版本、首错点、未确认项
- 没有日志证据 + 代码证据时，不能下根因结论；只能写“日志定位到某环节，代码证据未确认”
- 截图、发布单、用户口述只能作为线索，不能单独作为根因事实源
- 被质疑或证据冲突时，回到原始输入输出链路重新定位，不维护旧结论
- 首错点是字段/对象第一次从正确变错误的地方，不一定是最后报错、最后播报或最后展示的服务
- 归因时严格按工单填写的问题发生时间窗收敛证据；窗口外长尾只能作为前置上下文/风险点，不能直接当成窗口内根因

### Error Workorder Checklist

报错类工单优先收集：

- 用户可见现象 / 错误码 / 截图
- 发生时间窗和环境
- 接口 path
- `traceId` / `EagleEye-TraceID` / `span_id`
- 命中的 container / image / `APPLICATION VERSION`
- 异常堆栈首个业务方法
- 下游 HTTP / Dubbo / DB / Redis / MQ 错误
- 上游入参关键字段和下游响应关键字段
- 对应代码路径和运行版本是否一致

### Common Wrong Reasoning Patterns

- 把相似的后续请求当成当前工单请求
- 把截图错配当成上游根因
- 没查 `target*` / prompt 输入就说模型幻觉
- 没查 cleanData 输入输出就说清洗问题
- 没证明 `pid/qid/subQid` 返回错误对象就说召回错误
- 看到发布单已发布就认为灰度已命中
- 忽略测试复测可能命中了另一个后续发布版本
- 灰度验证通过就推导全量已经生效，或全量问题只查灰度容器

## Service / Project Context

### `engineering-context`

- 承载工程上下文资料、模块卡片、业务场景文档和相关 MCP 上下文能力
- 任务早期分析阶段应优先使用
- 基建合并文档中引用过 engineering-context 快照 sha：`11427ded4694fe13df7e2e1b3db1a7ca8abfb434`；sha 只是整理时快照，后续以 `engineering-context` 最新内容为准
- 已知高层模块：
  - `jzx-devops`：发布管理与 SRE 可观测性中枢
  - `jzx-gateway`：统一入口，负责认证、动态路由和灰度打标
  - `jzx-burial-trace`：埋点日志采集入口，常见业务观测经 `BurialUtil.modelLog` 上报
  - `jzx-metrics-sentinelX`：聚合 SLS、ARMS Prometheus、自建 Prometheus、Redis 埋点和告警事件

### `jzx-server`

- Java 8 / Spring Boot 2.5.15 Maven 多模块服务，父工程版本 `4.7.8`
- 远端：`git@codeup.aliyun.com:91jzx_code/k12-jzx-server/business/jzx-server.git`
- 主启动模块：`jzx-admin`
- 启动类：`jzx-admin/src/main/java/com/jzx/JzxApplication.java`
- 默认服务名/路径：`spring.application.name=jzx-server`，`server.port=10089`，`server.servlet.context-path=/jzx-server`
- 配置入口：`jzx-admin/src/main/resources/bootstrap.properties` + 环境文件 `bootstrap-{local,dev,test,pre,prod,remote_dev}.properties`，配置中心/注册中心走 Nacos
- 关键基础设施：MySQL + MyBatis/MyBatis-Plus，Redis/JetCache/Redisson，Nacos，Dubbo，OpenFeign，RocketMQ starter，Kafka，Activiti 6，Aliyun OSS，XXL-JOB
- 主要业务域：学习中心、Activiti 学习流、错题、英语背词、反馈/OSS/公式/批改辅助、内部 DTO/Facade
- 典型链路：
  - 精准学：`/hera/rest/task/start` -> `/hera/rest/lesson/task/nextQuestion` -> `/hera/rest/lesson/task/submit`
  - 计算训练：`/hera/rest/computeTraining/catalogByTeachType` -> `/session/start` -> `/answer/upload` -> `/session/finish` -> `/report/{sessionId}`
  - 错题：`/wrongQuestion/rest/search|delete|batchAddWrongQst`、`/study/rest/wrongQuestionBook/list|correct`、`/wrong/dailyClear/*`
  - 英语背词：`/english/rest/workflow/startLearn` -> `/nextQuestion` -> `/submitAnswer|tip|skipped`
- S5 数学同步题型过关入口：客户端 `sceneType=S5` 的 `chapterId + patternId[]` 请求落到 `jzx-activiti` 的 `/rest/lesson/task/start`，由 `StudentSelfTaskServiceImpl.startSelfTaskS5` 创建/续接 `hera_student_practice_task` 与 `hera_student_pattern_job`
- S5 排查“未解锁/未掌握”优先看 `hera_pre_test_pattern_master`、算法掌握度查询、`hera_answer_record`、S5 job 状态，而不是语英一对一的 `teachable` 或精品课 `lock`
- 代码图工具历史上对该仓库返回 0 nodes/0 edges，理解结构时以源码扫描为主；后续需要可先构建/更新图谱

### `poseidon`

- 路径：`/Users/chengzuozheng/Desktop/jzx-workspace/poseidon`
- Maven 多模块 Java 8 / Spring Boot 2.5.15 项目，父 POM 描述为 `AI双师课`
- 主启动入口：`jzx-admin/src/main/java/com/jzx/JzxApplication.java`
- 默认应用名：`poseidon`，默认端口和路径在 `jzx-admin/src/main/resources/bootstrap.properties` 中配置为 `10090`、`/poseidon`
- 主要模块边界：启动聚合、学习中心、Activiti/BPMN AI 数学、Redbird 图谱/学习概览、SK 高光/思维导图、错题、英语背词、判题/上传/反馈/渲染、跨服务 API、通用基础设施、掌握度/ZPD
- 重点入口：暖场、Redbird 图谱/学习概览、SK 高光/思维导图、错题、英语背词
- 本地 `bootstrap-local.properties` 含真实环境连接与密钥类配置；总结、提交、文档中不要复制具体值

### Hanxue Agent

- `hanxue-edu-agent` 是寒雪对外 OpenAPI 的上游 Agent 服务，不直接面向 RTC 客户端
- `engineering-context` 中应作为独立服务 `hanxue-edu-agent` 维护，不要套用旧 `hanxue-claw` 的 Dubbo 主入口结论；截至 2026-06-22 已按 `origin/release/20260618_v3` 快照补 README、OpenAPI Runtime、Agent Tools、Observability Runtime 和 task_by_module，记录 Agent Trace Kafka、课程服务任务富化、负一屏任务创建扩展字段与 TTS `audio-setting-format`
- 远端存在 `test` 分支；若某个本地 worktree 的 `remote.origin.fetch` 只配置了白名单分支，普通 `git fetch` 可能不会生成/更新 `origin/test`，可显式执行 `git fetch origin refs/heads/test:refs/remotes/origin/test`
- 对外核心接口：
  - `GET /open-api/v1/session-id`
  - `POST /open-api/v1/messages`
  - `GET /open-api/v1/sessions/{sessionId}/events`
- 对外事件协议是 `AgentPublicMessage` / `OpenApiSseEvent` 六元组：`seq`、`sessionId`、`messageId`、`eventType`、`payload`、`publishedAt`
- OpenAPI SSE 不是裸流，内置 session 租约互斥、心跳保活、`lastMessageId` 断点恢复、最新事件缓存
- Agent 内部事件转换流水线：`Agent 内部事件 -> SessionStreamState -> TurnCollector -> AgentPublicMessage -> SessionSseDispatcher -> SSE`
- 关键事件类型：`turn.start`、`thinking.*`、`tool.*`、`speech.*`、`note.ready`、`action`、`task.recommend`、`diagnosis.result`、`turn.done`、`error`
- `POST /messages` 是事件驱动输入，不只是普通聊天文本；`user-txt`、`upload_material`、`tab_choice` 都可以作为上游输入语义
- `user-txt` 允许空内容，空文本会被替换为 `[用户未发声]`

### Hanxue RTC Bridge

- 整体链路：`ARTC 客户端/RTC 音频/DataChannel -> jzx-artcManager(HanxueAdapter + HanxueAgentBridgeServiceImpl) -> hanxue-edu-agent(OpenAPI) -> SSE 事件流 -> jzx-artcManager -> TTS/RTC 音频 + DataChannel -> ARTC 客户端`
- `hanxue-edu-agent` 负责会话态 Agent 编排和标准事件流输出
- `jzx-artcManager` 负责寒雪场景桥接：会话建立、SSE 消费、事件翻译、TTS 推流、DataChannel 下发
- 客户端同时消费音频流和结构化业务消息流
- 寒雪业务入口在 `HanxueAdapter`
- `upload_material`、`tab_choice` 会被转成文本事件继续发往上游 Agent，不在本地做最终语义决策
- `exit_scene` 主要关闭本地 SSE，不额外向上游发结束协议
- `ensureSession` 优先复用旧 `sessionId`，避免 SSE 短断后新建 session 导致上下文、照片、技能激活状态丢失
- 2026-06-22 更新 `engineering-context/jzx-artcManager`：以远端 `origin/release/20260610` (`45b091aa082af4ba6e3731da904f77b411d15061`) 为最新事实；该 release 包含新板书 `board_control` 总控、`hanxue_call_v2` / `hanxue_call_v3` 路由拆分、`teacher_ready` / `readyId` 编排、预录音频首轮和音频 OSS 留存

SSE 到客户端的真实翻译规则：

- 上游 SSE 不原样透传，会进行协议翻译
- 语音路：`speech.*` 事件驱动 TTS，再走 RTC 音频输出
- 数据路：`action`、`task.recommend`、`note.ready`、`scene.ready` 等转 DataChannel JSON 下发给客户端
- `speech.start` / `speech.end` 不直接下发给客户端作为主业务事件
- `speech.delta` 会按文本去重，防止重复播报
- 同一轮 turn 内多段 `speech.delta` 共享同一个 `turnToken` 和同一条 TTS 流
- TTS `streamId` 采用 `hx2_speech::sessionId::turnToken` 形式，保证同 turn 内稳定
- `speech.end` 只是文本片段边界，`turn.done` 才是一轮老师输出的真正闭环边界
- TTS 流的真正 END 在 `turn.done` 统一发送
- 判断“老师这一轮是否真的讲完”应看 `turn.done`，不能只看 `speech.end`

Action / 结构化业务指令：

- `action` 事件是结构化客户端指令，不是普通文本
- 已知上游动作映射包括：`request_photo -> OPEN_MATERIAL_UPLOAD`、`close -> CLOSE_SESSION`、`show_task_card -> SHOW_TASK_CARD`、`show_app_card -> SHOW_APP_CARD`
- DataChannel 的价值不只是展示文本，而是驱动前端交互和状态机

### Phoenix / Agent Observability

- 寒雪 Agent 观测平台二期当前边界：不改 Agent 业务逻辑，不改首讲 prompt，不直接改线上配置；目标是给测试环境多个 Agent 副本绑定各自 Phoenix 观测平台副本
- Phoenix 代码中未发现直接读取 Nacos 的逻辑；Agent 上报 Phoenix 的 endpoint 在 Agent 的 Nacos 配置里
- MSE 测试 Nacos：
  - 实例：`zstt-dev-test`
  - 服务地址：`mse-aee88fd0-nacos-ans.mse.aliyuncs.com:8848`
  - 配置 namespace：显示名 `配置文件`，tenant/id `config`
  - Group：`DEFAULT_GROUP`
- 已确认的测试 Agent 副本配置映射：
  - `test-zstt-hanxue-edu-agent-3` 加载 `hanxue-edu-agent-test_2.yml`
  - `test-zstt-hanxue-edu-agent-4` 加载 `hanxue-edu-agent-test_3.yml`
- 关键字段：`phoenix.observability.endpoint`，形如 `http://<phoenix-service>:6006/v1/traces`
- Phoenix 副本部署 `test-zstt-phoenix-v1/v2/v3` 创建后可能因 K8s service 名 `phoenix` 自动注入 `PHOENIX_PORT=tcp://...` 导致启动失败；处理方式是在流水线/部署显式设置 `PHOENIX_PORT=6006`，或避免创建名为 `phoenix` 的 service
- 临时 NodePort 历史记录：老 Phoenix `30998`，v2 `32022`，v3 `31221`，节点 `192.168.12.222`；若流水线重建 Service，端口可能被覆盖，需重新确认

### Hanxue Observability Platform

- 平台路径：`/Users/chengzuozheng/Desktop/jzx-workspace/hanxue-observability-platform`
- 测试 namespace deployment：`test-zstt-hanxue-observability-service`、`test-zstt-hanxue-observability-web`
- Web NodePort 历史值：`31681`
- 跳板机 `192.168.15.200` 不是 K8s node；访问 NodePort 要用测试网络内集群节点 IP，例如历史记录 `192.168.41.1:31681`
- 后端 context path：`/api/hanxue-observability`
- readiness：`/api/hanxue-observability/ops/data-sources/readiness`
- runtime datasource 来自 Secret，不要把真实连接串复制进 manifests 或文档
- Spring Boot Actuator 曾因 Redis health 探测 `localhost:6379` 导致 rollout 失败；无 Redis 真实依赖时设置 `MANAGEMENT_HEALTH_REDIS_ENABLED=false`
- 前端构建必须同步到后端静态资源：`npm run build --prefix hanxue-observability-web` 后执行 `rsync -a --delete hanxue-observability-web/dist/ hanxue-observability-service/src/main/resources/static/hanxue-observability/`，再打包后端
- 打部署包前优先 `mvn -q clean -DskipTests package -f hanxue-observability-service/pom.xml`；仅 `package` 可能保留旧 hash
- 页面仍旧时，优先查流水线/镜像/JAR 静态资源 hash/浏览器加载 hash

数据模型和事实源：

- 主 join 优先级：`csTaskId` > `interactionId` > `sessionId` > `traceId`
- `csTaskId` 表示一次 CS 工具命中，优先用于 CS 推荐/表现/任务/状态事件钻取
- `taskId` 是 encourage 负一屏任务 row id，不能替代 `csTaskId`
- `readyId` 只用于本地 ready/speech 对齐，不能作为跨系统观测 id
- `hanxue_task_status_event` 的真实位置是测试环境 MySQL 的 `encourage.hanxue_task_status_event`，来源为 `test-zstt-encourage` -> Nacos `encourage-test.yml` -> `ds_mysql_master_url`
- 观测平台不要用 `rtcmanager` datasource 查询 `encourage` 表；应使用独立 `HANXUE_ENCOURAGE_DATASOURCE_URL/USERNAME/PASSWORD` 或等价 Secret 接入 `encourage` 库

SLS source：

- 测试环境 SLS project：`k8s-log-cf194c581f4c848d8a051abe4c1b2a8de`
- logstore：`test-app-log`
- Phase A 只消费 `COMMON-BASE模型调用日志`，不默认消费 `COMMON-TRACE模型调用日志`，避免重复统计
- Agent 观测日志关键词：`HANXUE_OBSERVE`
- task 观测日志关键词：`HANXUE_TASK_OBSERVE`
- SLS 原始日志主要在 `content` 字段，`COMMON-BASE模型调用日志:` 后是 BurialUtil 外层 JSON，业务 payload 在外层 `inParam` 字符串 JSON 中
- SLS 清洗 schema：`/Users/chengzuozheng/Desktop/jzx-workspace/hanxue-observability-prototype/hanxue_data_statistics_11_Schema_SLS观测日志清洗.md`
- 不要把 SLS AK/SK 写入记忆文件；测试阶段如需临时写配置，后续迁移到 Nacos/K8s Secret/流水线变量

钉钉认证：

- Nacos dataId：`hanxue-observability-service-test.yml`
- 配置路径：`hanxue.observability.auth.*`
- 认证回调地址规则：`{hanxue.observability.auth.public-base-url}/api/hanxue-observability/auth/callback`
- 当前 NodePort 临时 base URL 历史值：`http://192.168.12.222:31681`
- 企业钉钉授权成功即可访问，不落用户表，不做白名单/角色；服务端签发 `hanxue_obs_session` httpOnly Cookie
- 退出入口：`POST /api/hanxue-observability/auth/logout`
- 拦截器只精确放行 `/api/hanxue-observability/auth/**`，不要用路径包含 `/auth/` 做放行
- CORS 配 `*` 时不允许跨站带凭证
- 不要记录 DingTalk client secret、JWT secret、数据库密码、SLS AK/SK

## Domain Debugging Notes

### 作业辅导 / 讲题

核心链路：

```text
RTC 客户端 -> RTC 桥接服务 -> 作业辅导服务 -> 题库/解题/清洗/板书/AI/TTS -> RTC 桥接服务 -> 客户端
```

通用经验：

- 排查“拍照讲题反应慢/停顿”时，不要只看 `applyLectureNew` 同步接口耗时；该接口返回后还有 AI 回复生成、TTS/文本 LongPoll/SSE、结构化板书流等异步阶段
- `applyLectureNew 60s timeout` 只能覆盖开房间 HTTP 同步返回，不能覆盖“首个可感知老师回复/首个板书/板书完成”慢
- 板书慢要同时看业务网关接口耗时和完整 SSE/处理耗时
- `updateLecturePlayedRecord` 上传已播放 TTS/板书失败会影响下一轮学生发言；重点查非 JSON timeout 响应是否被直接 `JSONObject.parseObject`，以及业务侧是否出现 played boardnote record not found

讲错题 / 题目不对应：

- 先查 `applyLectureNew` 入参 `route/pid/qid/subQid`
- 对比搜索结果的 stem/image/answer/solution
- 对比 `originalTarget*`、`target*`、`question*`
- 对比持久化题目、prompt 输入、TTS 输入、板书输入
- 不能直接归因为模型幻觉；若 prompt/TTS/板书已经围绕错误题，下游模型只是消费错误输入
- 不能说 `Quark 题库错`，除非原始 API 响应就是错的，且没有本地转换导致错配
- 不能说 `清洗导致`，除非 cleanData 输入正确而输出错误，或代码证明 cleanData 响应覆盖字段有问题

TTS / 读错：

- 对比模型文本、cleanData request/response、TTS request content、subject 来源、该路径是否调用 cleanData
- 如果链路未调用 cleanData，结论应指向构造 TTS 请求的代码路径，而不是清洗服务
- 如果 cleanData subject 错了，要继续追 subject 来源

### 作业辅导历史案例结论

- 2026-06-01：数学计算题/竖式谜分支会创建 `ai_lecture_question_queue` 并由 `getCalcQuestion` 同步轮询 40s；如果 LLM `解题结果` 已出现且 `questionSolutionStatus=1`，但轮询 DB 仍为 `solutionStatus=0`，且没有后续更新日志，优先怀疑解题结果落库失败或异步任务异常被 `CompletableFuture.runAsync` 吞掉
- 2026-06-10：工单 `2064600087574024194` 主链路不是望远镜题，而是皮艇浮力题链路中 Quark 候选文本与内嵌图片不一致，cleanrender 非 OCR 路线解析 HTML `<img>` 后把 `targetStem` 清洗成“防溺水手环”题，answer/method 仍混入皮艇题；同类问题重点看 Quark 候选 `stem` 的文本与内嵌图片是否同题、OCR 后文本、`originalTarget*` 与 `target*` 分叉
- 2026-06-11：向量多选题“读错/讲错”不是题目对象错配，也不是清洗换题；首次污染发生在通用解题覆盖阶段：通用解题把原始 `AC/B错误` 改写成 `ABC/B正确`，并作为有效结果覆盖下游 DTO。同类问题按 `Quark原题 -> 清洗/标准化 -> 并行解题覆盖 -> 落库 -> 老师回复` 分层定位
- 2026-06-12：向量题读题 TTS 读出 LaTeX 的首错点是读题链路未调用 `cleanData`，直接把读题文本写入 `TTS_AUDIO` 和 `KeywordTtsRequest.content`
- 2026-06-12：读题 TTS 清洗已接入读题入口；清洗后文本只用于 text/regular TTS audio 与 keyword TTS，`bizData.response` 和板书仍保留原始读题文本
- 2026-06-12：cleanData 学科兜底修复引入 subject resolver；当文本包含明确数学公式且上游 subject 为空/`chinese`/`general` 时按 `math` 清洗，但保留 physics/chemistry/science 等真实学科
- Fastjson2 JavaBean getter 序列化 `pId` / `qId` 时可能输出 `PId` / `QId`；若下游 Jackson 按 `pId` / `qId` 反序列化，会导致字段丢失并触发入库错误

## Observability / Logging Engineering

- SLS 可证明某事件发生过，但不能单独证明业务 DB 状态已完成
- 历史日志如果只记录 digest/hash/length，没有原文或完整 payload，不能从 digest 反推历史内容
- 新增 `include-text`、`include-payload` 类日志开关通常只影响未来新日志，不能补历史数据
- `sink-mode: dual` 通常表示保留本地日志，同时上报 Burial/SLS；这是服务侧配置，不是 SLS 控制台配置
- 统计前先选定一类日志来源，避免同时消费 base 和 trace 导致重复统计
- 异步任务里要复制 traceId、业务主键等只读上下文，否则排障时链路容易断
- 只是排障日志时，不顺手改对外接口或客户端 payload

## Documentation Boundary

- `AGENTS.md` 只放高频、强约束、可执行规则
- 本文件保存长期经验、设施入口、排障结论和项目上下文
- 基建细节和较完整说明放到 `公司基建_00_Overview_流水线集群日志经验.md` 或合并版草稿
- 工单排障流程、SLS 配置和报告模板优先沉淀到 `jzx-workorder-debugging` skill reference
- 不在 `AGENTS.md` 或本文件复制密钥、密码、AK/SK、Cookie、数据库真实连接串

## Pending Additions

- 各项目启动方式
- 各项目测试命令
- 账号、环境、网关、配置中心接入说明
- 发布、联调、排查链路的权责边界
- 常见报错与处理结论
- SLS 测试 project 按服务维度整理成明确映射
- 各服务流水线和 deployment 的完整映射，优先从 `sdd-bugfix` 的 `backend-services.json` 和现场实时结果生成

## Update Log

### 2026-06-19

- 系统性整理 memory，按基础设施、排障、项目上下文、领域经验、观测工程重新分组
- 将 `*-new` 灰度、`*-base` 全量、日常工单发布流、事实源优先级同步到 `AGENTS.md` 和本文件
- 明确 `AGENTS.md` 只放强约束，细节进入 memory、基建文档或 skill reference

### 2026-06-17

- 记录寒雪观测平台架构收敛、静态资源 hash 排查、任务口径复核、钉钉认证和 Nacos 配置边界

### 2026-06-11 / 2026-06-12

- 记录寒雪观测平台测试部署、datasource split、SLS source、作业辅导向量题和 TTS 清洗排障结论

### 2026-06-03 / 2026-06-04

- 记录作业辅导 SLS 固定配置、生产灰度命中验证和 TTS/板书链路排障口径

### 2026-05-22 ~ 2026-05-29

- 初始化工作区记忆文件
- 记录 `engineering-context`、默认开发流程、spec 规范路径、寒雪链路、`sdd-bugfix` 集群入口、`jzx-server` 和 `poseidon` 项目上下文、Phoenix 多副本配置经验
