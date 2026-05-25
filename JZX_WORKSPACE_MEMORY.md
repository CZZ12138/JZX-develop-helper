# JZX Workspace Memory

适用范围：`/Users/ze/JZX`

本文件是工作区长期记忆库，保存可复用的经验、规范、设施信息、排障结论和项目上下文。

`AGENTS.md` 负责高频执行规则；本文件负责持续沉淀。凡是后续仍可能复用的信息，都应更新到这里，而不是只停留在对话里。

## Maintenance Rules

- 优先记录可复用结论，不记录纯一次性聊天内容
- 同类信息只保留一个最新版本，失效内容直接覆盖
- 尽量写清适用范围、背景、限制条件
- 敏感信息只记录入口、定位方式、使用规则，不直接写密钥
- 新出现的重要经验、开发规范、环境约定、排障结论，应及时补充到本文件

## Workspace Facts

- JZX 相关开发任务默认在 `/Users/ze/JZX` 工作区下进行
- 工作区级自动加载规则文件：`/Users/ze/JZX/AGENTS.md`
- 若子项目存在更近一级 `AGENTS.md`，其规则优先；若没有，再参考该项目的 `CLAUDE.md`

## Core Rules

- `engineering-context` 是工作区级上下文入口，也是上下文 MCP 服务来源
- 遇到跨项目背景、模块职责、历史方案、业务场景问题时，优先检索 `engineering-context`
- `engineering-context/skills` 已同步到 `~/.codex/skills`
- 开发流程默认遵循：需求边界澄清 -> 顶层架构 -> `spec -> plans -> execution`
- spec 写作规范入口：`/Users/ze/JZX/engineering-spec-writing-guideline.md`

## Reusable Prompts

- 需求理解阶段：
  “这是一个 JZX 项目和 xx 功能相关的新需求，先尝试理解下整体需求需要做什么？不做什么？任务的方向和边界在哪里。”
- 顶层架构阶段：
  “接下来帮我产出顶层的架构设计，该顶层架构需要输出文档 `xxx_00_Overview_顶层架构`，这个文档中需要包括整体可能要涉及到改造的项目，以及改造点，它本质上就是一个简化版的面向技术的 PRD 说明。”
- 顶层架构文档至少包含：
  - 顶层模块和架构设计
  - 模块边界与关系
  - 新增模块
  - 现有 JZX 仓库（服务 / 模块 / 端）改造范围

## Project Notes

### `engineering-context`

- 承载工程上下文资料、模块卡片、业务场景文档和相关 MCP 上下文能力
- 任务早期分析阶段应优先使用

### `hanxue-edu-agent`

- 是寒雪对外 OpenAPI 的上游 Agent 服务，不直接面向 RTC 客户端
- 对外核心接口：
  - `GET /open-api/v1/session-id`
  - `POST /open-api/v1/messages`
  - `GET /open-api/v1/sessions/{sessionId}/events`
- 对外事件协议是 `AgentPublicMessage` / `OpenApiSseEvent` 六元组：
  - `seq`
  - `sessionId`
  - `messageId`
  - `eventType`
  - `payload`
  - `publishedAt`
- OpenAPI SSE 不是裸流，内置：
  - session 租约互斥
  - 心跳保活
  - `lastMessageId` 断点恢复
  - 最新事件缓存
- Agent 内部事件转换流水线：
  - `Agent 内部事件 -> SessionStreamState -> TurnCollector -> AgentPublicMessage -> SessionSseDispatcher -> SSE`
- 关键事件类型：
  - `turn.start`
  - `thinking.start` / `think.delta` / `thinking.end`
  - `tool.start` / `tool.end`
  - `speech.start` / `speech.delta` / `speech.end`
  - `note.ready`
  - `action`
  - `task.recommend`
  - `diagnosis.result`
  - `turn.done`
  - `error`
- `POST /messages` 是事件驱动输入，不只是普通聊天文本；`user-txt`、`upload_material`、`tab_choice` 这类都可以作为上游输入语义
- `user-txt` 允许空内容，空文本会被替换为 `[用户未发声]`，让 Agent 感知“用户这一轮开口但没有有效语音”

### 寒雪调用链路

- 整体链路：
  - `ARTC 客户端/RTC 音频/DataChannel -> jzx-artcManager(HanxueAdapter + HanxueAgentBridgeServiceImpl) -> hanxue-edu-agent(OpenAPI) -> SSE 事件流 -> jzx-artcManager -> TTS/RTC 音频 + DataChannel -> ARTC 客户端`
- 角色边界：
  - `hanxue-edu-agent` 负责会话态 Agent 编排和标准事件流输出
  - `jzx-artcManager` 负责寒雪场景桥接：会话建立、SSE 消费、事件翻译、TTS 推流、DataChannel 下发
  - `ARTC 客户端` 同时消费音频流和结构化业务消息流

### `jzx-artcManager` 寒雪桥接

- 寒雪业务入口在 `HanxueAdapter`
- 核心职责：
  - `enter_scene` 时获取或恢复 `sessionId`
  - 启动或续接 SSE 订阅
  - 把用户文本和部分客户端事件转发给 `hanxue-edu-agent`
- `upload_material`、`tab_choice` 会被转成文本事件继续发往上游 Agent，不在本地做最终语义决策
- `exit_scene` 主要关闭本地 SSE，不额外向上游发结束协议
- `ensureSession` 优先复用旧 `sessionId`，避免 SSE 短断后新建 session 导致上下文、照片、技能激活状态丢失

### SSE 到客户端的真实翻译规则

- `HanxueAgentBridgeServiceImpl` 消费上游 SSE，不做原样透传，而是进行协议翻译
- 下游分两路：
  - 语音路：`speech.*` 事件驱动 TTS，再走 RTC 音频输出
  - 数据路：`action`、`task.recommend`、`note.ready`、`scene.ready` 等转 DataChannel JSON 下发给客户端
- `speech.start` / `speech.end` 不直接下发给客户端作为主业务事件
- `speech.delta` 会做按文本去重，防止重复片段导致重复播报
- 同一轮 turn 内的多段 `speech.delta` 共享同一个 `turnToken` 和同一条 TTS 流
- TTS `streamId` 采用 `hx2_speech::sessionId::turnToken` 形式，保证同 turn 内稳定
- 关键原则：
  - `speech.end` 只是文本片段边界
  - `turn.done` 才是一轮老师输出的真正闭环边界
  - TTS 流的真正 END 在 `turn.done` 统一发送
- 结论：判断“老师这一轮是否真的讲完”应看 `turn.done`，不能只看 `speech.end`

### Action / 结构化业务指令

- `action` 事件是结构化客户端指令，不是普通文本
- 已知上游动作映射包括：
  - `request_photo -> OPEN_MATERIAL_UPLOAD`
  - `close -> CLOSE_SESSION`
  - `show_task_card -> SHOW_TASK_CARD`
  - `show_app_card -> SHOW_APP_CARD`
- 因此 DataChannel 的价值不只是展示文本，而是驱动前端交互和状态机

### 理解寒雪链路时的几个关键点

- 寒雪不是“Agent 直接对 RTC 说话”，而是“Agent 输出标准 SSE 事件流，artcManager 做桥接”
- `jzx-artcManager` 是协议适配层，不是上游业务决策层
- 客户端看到的是统一老师体验，但底层是：
  - 音频流负责“听见”
  - DataChannel 负责“看见”和“交互逻辑”
- 排查链路问题时要区分是：
  - 上游 Agent 没产出事件
  - SSE 没收到/没恢复
  - 桥接层没正确翻译
  - TTS/RTC 音频没推成功
  - DataChannel 没正确下发/消费

### `sdd-bugfix`

- 保存了访问集群所需的 key / kube 相关接入信息
- 在该仓库上下文中，`kubectl` 可以完成大多数集群操作
- 可以从远程服务器拉取开发代码，并部署到 K8s 集群进行测试
- 适合作为联调、部署验证、集群问题排查入口
- 涉及真实环境操作时，先确认目标环境边界

## Pending Additions

- 各项目启动方式
- 各项目测试命令
- 账号、环境、网关、配置中心接入说明
- 发布、联调、排查链路
- 常见报错与处理结论

## Update Log

### 2026-05-22

- 初始化工作区记忆文件
- 记录 `engineering-context` 为上下文 MCP 服务入口
- 记录 `engineering-context/skills` 已同步到 Codex 本地技能目录
- 记录默认开发流程：需求边界澄清 -> 顶层架构 -> `spec -> plans -> execution`
- 记录 spec 规范路径：`/Users/ze/JZX/engineering-spec-writing-guideline.md`
- 记录常用需求理解与顶层架构 Prompt
- 记录寒雪 agent 的真实调用链路、SSE 事件协议、桥接层职责与 `turn.done` 闭环语义
- 记录 `sdd-bugfix` 的集群接入、远程拉代码、K8s 测试部署能力
- 记录工作区级规则文件采用 `AGENTS.md`
