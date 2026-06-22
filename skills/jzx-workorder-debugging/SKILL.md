---
name: jzx-workorder-debugging
description: Diagnose JZX work orders by combining ticket details, SLS logs, browser evidence, and local code repositories. Use when the user asks to 排查/复核/定位/汇报 a 工单, traceId, roomId, userId, device SN, production/test log issue, 作业辅导/寒雪/讲题问题, or asks for a concise evidence-backed bug report or owner-facing summary.
---

# JZX Workorder Debugging

## Purpose

Turn a work-order symptom into a defensible diagnosis: what happened, where the first wrong state appeared, which service/repo owns it, what evidence proves it, and what should be reported.

Before any JZX work-order investigation, read:

- `references/sls-config.md` for embedded SLS projects, logstores, containers, query strategy, and operational pitfalls.
- `references/evidence-standards.md` for mandatory evidence rules, code-proof requirements, and correction protocol.

When writing a formal report, owner-facing summary, or cross-ticket comparison, also read `references/report-patterns.md`.

## Required Inputs

Collect or infer these fields before concluding:

| Field | Why it matters |
| --- | --- |
| `workOrderId` / URL | Ticket source of truth and time window. |
| problem time window | Bound SLS queries; avoid attributing window-external logs. |
| environment | `test`, `pre`, `prod-base`, `prod-new`, or unknown. |
| trace identifiers | `traceId`, `EagleEye-TraceID`, `roomId`, `roundId`, `lectureQuestionId`, `pid/qid/subQid`, `userId`, device SN. |
| user-visible symptom | Separate observable behavior from inferred cause. |
| relevant repo/branch | Match runtime version before code attribution. |

If key fields are missing, search available sources first: current Chrome page/appshot, provided screenshots, local docs such as `/Users/ze/JZX/jzx-composition/CLAUDE.md`, and `/Users/ze/JZX/JZX_WORKSPACE_MEMORY.md`.

## Workflow

1. **Restate scope.** Say exactly which ticket/problem is being investigated and which problem is not in scope.
2. **Extract the business flow.** Build a compact flow from user action to downstream services. Prefer a Mermaid flow when the flow crosses services.
3. **Pin the runtime.** Confirm environment, deployment lane, and actual container/version hit by the request. Do not assume a DevOps release means the request used that image.
4. **Collect logs chronologically.** Query by strongest identifier first: `traceId` > `roomId + time` > `userId/device + time` > interface path + time. Use SLS and compare adjacent services.
5. **Create an input/output chain.** For each hop, record request parameters, response payload, important transformed fields, timestamp, container, and trace/span.
6. **Find the first wrong state.** The root cause candidate is the earliest hop where the data differs from the expected business object, not the latest service that displays the bad result.
7. **Map to code.** Search local repos for the method/class/log text handling that hop. Use the branch matching the runtime where possible. Do not claim code root cause without a code reference or a clearly stated runtime-code mismatch.
8. **Separate facts from hypotheses.** Label confirmed evidence, inferred cause, and remaining uncertainty.
9. **Correct aggressively when challenged.** If the user questions a conclusion, rebuild the raw input/output chain and explicitly state whether the old conclusion was wrong, incomplete, or still valid with narrower scope.
10. **Write a short report.** Load `references/report-patterns.md` when the user asks for a report, owner summary, or cross-ticket comparison.

## SLS Usage

Use `references/sls-config.md` as the embedded SLS configuration source. Prefer that file over memory when handing this skill to another agent. If local memory or project docs disagree, verify by querying recent `APPLICATION VERSION`/request logs and state which source was used.

## Code Mapping

Common local repos under `/Users/ze/JZX`:

| Repo | Typical ownership |
| --- | --- |
| `jzx-composition` | 作业辅导/讲题 orchestration, exampaper, TTS/板书 integration. |
| `jzx-ai-dual-mentor` | Legacy or reference implementation for讲题链路. |
| `jzx-question-component` | 搜题、题库、清洗/解题/题目标准化 related processing. |
| `jzx-artcManager` | RTC/ARTC bridge, applyLecture/open-room, SSE/DataChannel bridge. |
| `jzx-server`, `poseidon` | User/study/wrong-question supporting data when involved. |

Always verify branch and runtime version before saying “代码原因”. If code and runtime mismatch, say “code suggests” or “likely” rather than asserting.

## Evidence Rules

Follow `references/evidence-standards.md` strictly.

- Do not conclude from a screenshot alone when logs can confirm the data chain.
- Do not blame the final model response until upstream `targetStem/targetAnswer/targetMethod/questionAnalysis`, cleanData input/output, and selected `pid/qid/subQid` are checked.
- When the user challenges a conclusion, rebuild from raw input/output chain rather than defending the previous wording.
- If code evidence is missing, say `日志证据定位到 <service/hop>，代码证据未确认`, not `代码原因是`.
- For “讲错题/题目不对应”, specifically compare:
  - selected result metadata: `route`, `pid`, `qid`, `subQid`
  - retrieved question object: stem/image/answer/solution
  - normalized/cleaned fields: `originalTarget*`, `target*`, `question*`
  - persisted `AILectureQuestion`
  - prompt input to model / TTS / boardnote
- For “读错/TTS 读错”, specifically compare:
  - model text before TTS
  - `cleanData` request and response
  - TTS request content
  - subject field source
  - whether the specific chain calls cleanData at all

## Output Style

Be concise and evidence-led. A useful answer normally contains:

- `现象`: what the user saw.
- `首错点`: earliest confirmed wrong state.
- `原因判断`: confirmed or likely cause, with confidence wording.
- `证据`: 3-6 high-signal bullets with trace/time/container/code refs.
- `影响/边界`: what this does and does not explain.
- `下一步`: owner or fix direction.
