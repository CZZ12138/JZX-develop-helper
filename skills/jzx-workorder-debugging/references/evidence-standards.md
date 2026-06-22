# Evidence Standards and Correction Protocol

Use this file to keep JZX work-order diagnoses rigorous. The user has explicitly observed that prior conclusions can be wrong when they are not grounded in enough code/log evidence.

## Non-Negotiable Rules

1. **No root-cause claim without evidence.**
   - A root cause requires at least one decisive log/input-output artifact and one code reference, unless the user only asked for a preliminary hypothesis.
   - If code evidence is not yet found, write: `日志证据定位到 <hop/service>，代码证据未确认`.

2. **Find the first wrong state.**
   - The first wrong state is the earliest observed field/object/payload that differs from expected business state.
   - The service that displays or speaks the bad content is not automatically the cause.

3. **Separate observation, evidence, and judgment.**
   - `现象`: what user/screenshot/ticket shows.
   - `日志证据`: raw fields and timestamps.
   - `代码证据`: file/method/line or exact logic.
   - `判断`: what the evidence supports.
   - `未确认`: what has not been proven.

4. **Do not defend stale conclusions.**
   - If the user challenges a conclusion, restart from raw chain evidence.
   - Explicitly state whether the old conclusion was wrong, incomplete, or still valid with narrowed scope.

5. **Code evidence must be concrete.**
   - Cite repository, branch/runtime version, file path, method/class, and the relevant conditional/assignment/call.
   - Prefer clickable local file links in final answers when available.
   - If line numbers may shift, include method names and exact code tokens.

6. **Runtime-code match must be checked.**
   - Confirm branch/image/version used by the request before blaming current local code.
   - If runtime cannot be confirmed, mark code attribution as provisional.

7. **Avoid over-compressing evidence.**
   - Keep decisive raw fields: `traceId`, `roomId`, `roundId`, `lectureQuestionId`, `pid`, `qid`, `subQid`, `targetStem`, `targetAnswer`, `questionAnswer`, `subject`, `text`.
   - Redact secrets and personal contact data.

## Evidence Ladder

Use the strongest evidence available:

1. Same trace, same timestamped chain, same payload field before/after transformation.
2. Same roomId/lectureQuestionId and narrow time window across services.
3. Same user/device/time plus interface path.
4. Screenshot/ticket text only.

Do not present level 4 as root cause when level 1-3 can be obtained.

## Code-Proof Checklist

Before finalizing a diagnosis, answer:

- Which service first produced the wrong value?
- Which field first became wrong?
- What was the previous correct value?
- What method/class wrote or transformed it?
- What code path makes that transformation possible?
- What branch/image version was running?
- Does a downstream service merely consume the wrong value?
- Is there any alternative path in the same time window?

If any answer is missing, mark it in `未确认`.

## Correction Protocol

When a conclusion is challenged or evidence contradicts it:

1. Quote the exact challenged conclusion briefly.
2. State the new question being tested.
3. Rebuild the chain table from raw logs:
   - upstream input
   - transformation output
   - persistence
   - downstream prompt/TTS/boardnote
4. Identify where the previous reasoning skipped a hop or confused similar records.
5. Replace the conclusion with one of:
   - `确认修正：...`
   - `结论收窄：...`
   - `仍未确认：...`
6. Preserve both old and new evidence if useful, but make the final conclusion unambiguous.

## Common Wrong Reasoning Patterns

- Mistaking a similar later request for the ticket’s actual request.
- Treating UI/screenshot mismatch as proof of the upstream cause.
- Blaming `模型幻觉` without checking `target*` / prompt input.
- Blaming `清洗问题` without checking cleanData input/output and surrounding solve/merge steps.
- Blaming `召回错误` without proving the selected `pid/qid/subQid` returned the wrong object.
- Assuming production gray release was hit because DevOps says published.
- Ignoring that `imageUrls`, `mergeImageUrl`, sparse arrays, and selected question index can diverge.
- Ignoring tests or user retest may have hit a different pipeline/image/version.

## Domain-Specific Proof Requirements

### Wrong Question / 讲错题

Need proof for:

- selected `route/pid/qid/subQid`
- retrieved question stem/image/answer/solution
- persisted `AILectureQuestion` fields
- any cleanup/solve/merge stage that overwrote fields
- model prompt input

Do not say `Quark 题库错` unless the raw Quark/API response itself contains the mismatched text/image/answer and no local transformation explains it.

Do not say `清洗导致` unless cleanData input is correct and cleanData output is wrong, or code shows cleanData response overwrites the field incorrectly.

### TTS / 读错

Need proof for:

- text before cleanData
- cleanData request subject/text
- cleanData response
- TTS request content
- code path that calls or omits cleanData

If the chain never calls cleanData before TTS, cite the method path that builds the TTS request directly from model text.

If cleanData is called with wrong subject, trace subject source and cite code.

### Slow Response / 卡顿

Need proof for:

- exact user-visible phase
- start/end timestamps for each service hop
- whether blocking is synchronous or asynchronous
- timeout/error payloads
- retry or downstream record update failures

Do not use total ticket time or one endpoint duration as the entire cause.

## Report Language

Use calibrated language:

- `可以确认`: log and code evidence both support it.
- `日志上可以确认`: logs support it, code not yet checked.
- `代码上存在风险`: code path could cause it, but matching runtime/log not confirmed.
- `倾向判断`: plausible but still missing one proof.
- `不能确认`: evidence does not support assertion.

Avoid:

- `肯定是` unless proven.
- `应该是` without saying what is missing.
- `代码没问题` unless relevant code path and runtime were checked.
