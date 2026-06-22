# JZX Workspace AGENTS

适用范围：`/Users/ze/JZX`

## Rule Order

1. 就近子项目 `AGENTS.md`
2. 子项目 `CLAUDE.md`
3. 本文件
4. 工作区记忆文件：`/Users/ze/JZX/JZX_WORKSPACE_MEMORY.md`

## Must Follow

- JZX 相关任务默认以当前工作区为根上下文
- 先做需求边界澄清，再做顶层架构，再进入 `spec -> plans -> execution`
- 新需求默认先回答：做什么、不做什么、方向、边界
- 理解需求、梳理方案或做交接说明时，优先画任务流图或数据流图，先把业务流、系统流、核心对象关系画清楚，再进入文字分析
- 理解项目、业务模块或跨系统链路时，除画图外，应把必要字段整理成简洁字段表，说明字段位置、业务语义、主要用途、上下游关系，帮助快速熟悉业务上下文
- 开发任务拿到 PRD 后，默认走 superpowers 流程
- 顶层架构文档命名：`xxx_00_Overview_顶层架构`
- 顶层架构至少覆盖：顶层模块、模块边界关系、新增模块、现有仓库改造范围

## Infrastructure Fact Sources

- DevOps 发布单只证明发布动作发生过；发布是否真实生效，以 K8s 当前 `image` / `ReplicaSet` 和 SLS 实际请求 `traceId/container/version` 为准
- `*-new` 表示生产灰度环境，`*-base` 表示生产全量环境
- 配置事实以目标环境 Nacos / Secret / Pod env 为准，不从旧文档或本地代码硬猜
- 业务状态以业务 DB 为准；SLS 证明事件发生，不等于业务状态闭环
- SLS 查询优先级：`traceId` / `EagleEye-TraceID` > 业务主键 / `roomId` / `sessionId` > `userId` / device > path + 时间窗

## Release / Pipeline Rules

- 日常工单 bug 发布流：先修复 -> 发测试流水线 -> 测试验证 -> 发预发 / 填预发发布单 -> 把发布单链接交给陈灵琪发生产
- 生产发布、生产 Nacos、生产重放请求默认不能由 Agent 自行操作，除非用户明确授权
- 验证灰度不能推导全量生效；验证全量不能只看灰度容器日志
- 同一流水线后续发布可能覆盖前一次 bugfix，排查复测结果时必须确认当前运行 tag

## Workorder / Error Debugging

- 工单、报错、生产/测试日志排查默认先使用 `jzx-workorder-debugging` skill
- 排障结论必须区分：现象、日志证据、代码证据、运行版本、首错点、未确认项
- 没有日志证据 + 代码证据时，不能下根因结论；只能写“日志定位到某环节，代码证据未确认”
- 截图、发布单、用户口述只能作为线索，不能单独作为根因事实源
- 被质疑或证据冲突时，回到原始输入输出链路重新定位，不维护旧结论

## Primary Context

- 优先使用 `engineering-context`
- `engineering-context` 是上下文 MCP 服务入口，也是跨项目背景资料主来源
- `engineering-context/skills` 已同步到 `~/.codex/skills`
- `engineering-context/skills` 也是 superpowers 流程的一部分，开发阶段不能只把它当参考资料，需要按需加载并执行其中约定
- spec 写作规范入口：`/Users/ze/JZX/engineering-spec-writing-guideline.md`

## code-review-graph

- 当任务需要理解 `code-review-graph` 代码库、分析其结构、查找调用关系或做变更影响分析时，优先加载该仓库自带的 MCP 服务
- 该仓库已经在 Codex 侧完成配置，相关能力应通过图工具而不是纯文件扫描来获取上下文
- 日常围绕该仓库的工作，先看图状态，再做 build/update/detect-changes 这类图操作

## Memory File

- 长期经验、开发规范、设施信息、排障结论，统一维护在 `JZX_WORKSPACE_MEMORY.md`
- 工作过程中出现值得复用的信息，应更新该记忆文件，而不是只停留在对话里
- 本文件只保留高频、强约束、可执行规则；细节和持续沉淀内容放入记忆文件
- SLS project/logstore/container、Nacos dataId、NodePort、具体工单 trace、发布 tag 等细节不要放在本文件；放到 memory、基建文档或对应 skill reference
- 不在本文件维护普通业务代码仓库清单；只保留 `sdd-bugfix` 这类功能性排障入口

## Repo Notes

- `sdd-bugfix` 可作为集群接入、`kubectl` 操作、远程代码拉取、K8s 测试部署入口
- 进入有专属规则的子仓库前，先读取其本地规则文件
