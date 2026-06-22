# JZX Skills

这个目录保存 JZX 工作区可复用的 Codex skills。

## `jzx-workorder-debugging`

用途：按工单、SLS 日志、trace/roomId/userId、截图和本地代码证据定位首错点，并输出证据驱动的排障结论或汇报。

目录结构：

- `SKILL.md`：skill 入口、触发场景、工作流和输出风格
- `references/sls-config.md`：SLS project/logstore/container、查询策略和常见坑
- `references/evidence-standards.md`：证据标准、代码证明要求和纠错协议
- `references/report-patterns.md`：正式排查报告、owner 汇报和跨工单对比模板
- `agents/openai.yaml`：OpenAI/Codex 侧展示信息

同步到本机 Codex：

```bash
rsync -a skills/jzx-workorder-debugging/ ~/.codex/skills/jzx-workorder-debugging/
```
