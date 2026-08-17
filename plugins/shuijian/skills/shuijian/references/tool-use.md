# Tool 填写与相邻边界

## 共同字段

- `userText`：本轮用户原话，最多 2000 字。
- `contextExcerpt`：仅带本动作不可缺少的最多六段上下文。
- `claims`：区分 `reported_event`、`reported_quote`、`lived_experience`、`interpretation`、`evaluation`、`prediction`、`hidden_rule`、`identity_binding`、`real_constraint`、`unknown`。
- `sourceMessageRef`：可用宿主回合编号或稳定引用；没有时不要虚构。
- `userConfirmed`：只有用户明确确认过才为 `true`。
- `proposedMove`：一个动作，最多两小段；先用一句普通话承认具体处境，再用两三句短句完成动作。不要用“实际发生的是”“目前能确定的是”“你接上去的故事是”作标签，也不要用冒号或分号把分类标签和情绪词堆在一起。提问类必须只有一个问号。

## 完整输入契约

runner 没有 `--help`命令。不要运行 `--help`，也不要读取或 grep `work.mjs`；下面就是完整契约。

所有 Tool 都包含共同字段：

- 必填：`userText: string`、`proposedMove: { kind, text }`。
- 可选：`sessionId: uuid`、`ledgerVersion: integer`、`requestId: string`、`contextExcerpt: [{ role: "user" | "assistant", text }]`、`consent: { mayChallenge?, mayGoDeeper? }`、`safetySignal: { mode: "allow" | "support_only" | "handoff_required", reasonCode? }`。
- `claim`格式：`{ id?, text, kind, attributedTo?, sourceMessageRef?, derivedFrom?, evidenceState?, userConfirmed? }`。
- `kind`可选：`reported_event | reported_quote | lived_experience | interpretation | evaluation | prediction | hidden_rule | identity_binding | real_constraint | unknown`。
- `attributedTo`可选：`user | other | agent | unknown`；`derivedFrom`是 claim ID 数组；`evidenceState`可选：`inference_heavy | constraint_supported | mixed | unclear`。

各 Tool 额外字段：

- `receive_moment`：`claims`必填，`focusEvent?`；`proposedMove.kind = acknowledge`。
- `illuminate_event`：`claims`必填，`focusField? = quote | event | time | impact | unknown`，`focusEvent?`；`kind = clarify`。
- `locate_dwelling`：`claims`必填；`kind = reflect_chain | ask_permission`。
- `deconstruct_claim`：`targetClaim`、`supportingClaims`、`allowedInterventions`必填；`kind = ask_one_question`。`allowedInterventions`可选值：`change_subject | change_time | change_scale | alternative_explanation | boundary_counterexample | reverse_test | identity_separation | responsibility_split`。
- `verify_evidence`：`targetClaim`、`evidenceState`必填；`evidenceFor`、`evidenceAgainst`、`realConstraints`、`unknowns`传数组；`kind = state_uncertainty | clarify | reflect_chain`。它没有顶层 `claims`字段；`targetClaim.derivedFrom`引用的来源 claim 必须放进上述四个数组之一。
- `cut_narrative_chain`：`claims`、`edges`、`supportedUntilClaimId`、`pauseAfterClaimId`必填；edge 为`{ id?, fromClaimId, toClaimId, relation, evidenceState, cutStatus? }`；`kind = reflect_chain | pause`。
- `loosen_identity`：`identityClaim`、`userInitiatedQuestion`必填；`kind = ask_one_question | pause`。
- `surface_next_response`：`claims`传数组，`responses`必填且为 1–3 个`{ kind, text }`；response kind 可选`action | rest | conversation | boundary | information | wait | no_decision`；`proposedMove.kind = surface_options`。
- `stop_session`：`reason`必填，可选`user_request | enough_clarity | repetition | needs_reality | safety | no_value_added`；`kind = pause | close`。

首次调用省略 `sessionId`和`ledgerVersion`。空数组也要显式传入需要数组的字段。`derivedFrom`中的每个 ID 必须已在同一 Ledger，或作为本轮 claim 一并传入。

## 相邻 Tool 的分界

- `接`与`照`：前者让用户继续说；后者只补一个关键事实。
- `照`与`验`：前者缺原始材料；后者已有具体 claim，正在衡量证据边界。
- `住`与`拆`：前者指出可能卡在哪里；后者在获得许可后检查一条明确推论。
- `拆`与`断`：前者问一个能让用户自己回答的问题；后者暂停一条已画出的无依据连接。
- `验`与`生`：前者先守住真实约束和未知；后者才展开一至三个回应空间。
- 任意 Tool 与`止`：没有新增价值、用户已够用或现实信息更重要时，选`止`。

## 候选动作类型

- `接`：`acknowledge`
- `照`：`clarify`
- `住`：`reflect_chain`或`ask_permission`
- `拆`：`ask_one_question`
- `验`：`state_uncertainty`、`clarify`或`reflect_chain`
- `断`：`reflect_chain`或`pause`
- `空`：`ask_one_question`或`pause`
- `生`：`surface_options`
- `止`：`pause`或`close`

不要使用诊断、身份定性、通用鸡汤、佛学标签。`拆`不能替用户回答；`止`不能带新问题。

## 会话恢复

- 同一任务复用固定的 state 文件，并优先带回上一结果的 `sessionId`与`ledgerVersion`。
- 收到 `SESSION_NOT_FOUND`时，只静默恢复一次：省略旧会话字段，并把当前所有 `derivedFrom`引用的来源 claim 一并放入重试输入。
- 不得把旧会话中的 claim ID 单独带进新会话；恢复输入必须形成完整、无环的来源链。
- 不向用户解释缓存、路径、TTL、版本或恢复重试。
