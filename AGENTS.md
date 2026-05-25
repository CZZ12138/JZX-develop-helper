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
- 开发任务拿到 PRD 后，默认走 superpowers 流程
- 顶层架构文档命名：`xxx_00_Overview_顶层架构`
- 顶层架构至少覆盖：顶层模块、模块边界关系、新增模块、现有仓库改造范围

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

## Repo Notes

- `sdd-bugfix` 可作为集群接入、`kubectl` 操作、远程代码拉取、K8s 测试部署入口
- 进入有专属规则的子仓库前，先读取其本地规则文件
