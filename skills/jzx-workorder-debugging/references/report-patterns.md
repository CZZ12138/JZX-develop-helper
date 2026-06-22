# JZX Workorder Report Patterns

Use these patterns when producing a markdown/html report, owner-facing summary, or cross-ticket comparison.

## Minimal Report

```markdown
# 工单排查报告：<workOrderId>

## 结论
<1-3 句话。先说用户现象，再说首错点和责任链路。若代码证据未确认，必须明说。>

## 工单信息
| 字段 | 值 |
| --- | --- |
| 工单 | <id/url> |
| 时间窗 | <start ~ end> |
| 环境 | <prod-base/prod-new/test/unknown> |
| 用户/设备 | <userId/device SN; redact phone> |
| 关键 ID | <traceId/roomId/roundId/lectureQuestionId/pid/qid/subQid> |

## 链路与证据
| 时间 | 服务/容器 | 方法/接口 | 输入关键字段 | 输出关键字段 | 判断 |
| --- | --- | --- | --- | --- | --- |
| <ts> | <container> | <method/path> | <raw key fields> | <raw key fields> | <normal/wrong> |

## 首错点
<明确指出第一处从正确变错误的字段/服务/方法。>

## 代码定位
| 仓库 | 分支/版本 | 文件/方法 | 说明 |
| --- | --- | --- | --- |
| <repo> | <branch/image> | <path:line or method> | <why it owns the behavior> |

## 证据充分性
| 类型 | 状态 | 说明 |
| --- | --- | --- |
| 日志证据 | <已确认/未确认> | <trace/time/container/raw fields> |
| 代码证据 | <已确认/未确认> | <repo/branch/file/method> |
| 运行版本 | <已确认/未确认> | <image/version/container> |

## 边界
- 已确认：...
- 未确认：...
- 不属于本次问题：...

## 建议
<修复方向、补日志、回归用例、找哪个 owner。>
```

## Owner-Facing Summary

Use this when the user wants to report to another repository owner:

```text
这个工单现象是：<用户选择/操作> 后，<实际错误表现>。
我按 trace/roomId 查了链路，首错点出现在 <service/method/interface>：输入仍是 <正确对象/字段>，但输出已经变成 <错误对象/字段>。
下游 <composition/artc/tts/boardnote/model> 只是消费了这个错误字段，所以不是最终讲题模型局部幻觉。
日志证据：<traceId>，<timestamp>，<container>，<field A -> field B>。
代码证据：<repo/branch/file/method> 中 <具体调用/赋值/覆盖逻辑>。
建议你们重点看 <repo/file/method> 的 <召回/清洗/标准化/合并/覆盖/映射> 逻辑。
```

## Cross-Ticket Comparison

When comparing two or more tickets, use this table:

| 工单 | 表面现象 | 首错点 | 是否同因 | 证据差异 |
| --- | --- | --- | --- | --- |
| <id> | <symptom> | <first bad hop> | <yes/no/uncertain> | <trace/field/code> |

Rules:

- Same user-facing symptom does not mean same root cause.
- Same final wrong field does not mean same first wrong hop.
- Do not merge tickets unless the first wrong state and owning code path match.
- If one ticket lacks code evidence, mark `是否同因` as `uncertain`.

## Evidence Snippets

Prefer compact snippets:

```json
{
  "traceId": "...",
  "roomId": "...",
  "method": "...",
  "input": {
    "pid": "...",
    "qid": "...",
    "subject": "...",
    "targetStem": "..."
  },
  "output": {
    "targetStem": "...",
    "targetAnswer": "..."
  }
}
```

Avoid dumping full payloads unless the mismatch is only visible in the full object. Redact phone numbers, credentials, cookies, and unrelated personal data.
