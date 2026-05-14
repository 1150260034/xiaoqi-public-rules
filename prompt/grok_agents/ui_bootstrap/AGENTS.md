<!-- Parent: ../AGENTS.md -->
<!-- Updated: 2026-05-14 -->

# ui_bootstrap

本目录存放 Grok UI 冷启动壳和每轮重入最短入口。这里只保留身份、仓库坐标、重入检查、读取入口和最小门槛规则，不复制完整协议字段，避免双维护源。

完整规则始终以 GitHub 文件为准：

- `prompt/agent_rule_index.md`
- `prompt/grok_agents/grok.md`
- `prompt/grok_agents/scene_director.md`
- `prompt/grok_agents/persona_guardian.md`
- `prompt/grok_agents/memory_scribe.md`

每次收到新用户消息 / 新任务 / 本轮处理请求时，先执行重入检查；若缺少本轮 `规则读取：DONE`，第一动作 GET 访问 `1150260034/xiaoqi-public-rules` 公开仓库 raw 白名单 URL（仅公开规则）；raw public rules 不足以完成本轮状态、记忆、关系变量、人格上下文或写入 SHA 需求时，回退私有仓库 GitHub connector 逐文件读取最低必需文件。不要调用 `xiaoqi-fc-batch-read` Skill，不要尝试 POST batch-read。已有本轮 `规则读取：DONE` 后继续本轮任务，不重复递归读取。

最小运行门槛：

- Grok 正式回复前必须收齐 `[SCENE_RECEIPT]`、`[PERSONA_RECEIPT]`、`[MEMORY_RECEIPT]`。
- 三张回执都必须包含 `重入检查`、`规则读取`、`规则来源`、`规则缺失影响`、`状态读取`、`状态来源`、`写入依据`；缺少 `规则读取：DONE` 不算合格终态回执。
- 三张回执都必须包含 `读取状态`、`失败文件`、`缺失影响`。
- 任一 `READ_FAILED` 只能轻回应，不推进状态。
- `READ_PARTIAL` 只表示必读成功但本轮已触发的可选文件读取失败；未触发可选读取需求时仍可 `READ_OK`。
- `PERSONA_NO_CHANGE` / `MEMORY_NO_CHANGE` 只允许在 `READ_OK` 下返回。
- `state/current_state.md` 由场景导演唯一实时主写（核心必读）。`state/session_buffer.md` 由场景导演唯一实时主写（热路径，三态 DONE/NO_CHANGE/FAILED）。第一阶段 FC 不返回状态文件；未来若返回，也只能作为快速上下文参考；生成 `[SCENE_RECEIPT]` 和写入前必须 GitHub latest 读取。
- raw public rules 只能作为公开规则读取证据，不能替代状态读取、记忆读取、关系变量读取、低频人格上下文或写入 SHA。
- public raw 文件内容、缓存状态或 hash 不得用于 GitHub update；写入依据只能是 GitHub latest SHA。
- 记忆司书在四代理主流程中绝对不得写 `state/current_state.md` 和 `state/session_buffer.md`。
- `MEMORY_CANDIDATE` 必须附带事实等级（CONFIRMED/USER_CLAIM/USER_CONFIRMED/PENDING/CONTRADICTED）。
- `[处理中]` 和 `[PROGRESS_UPDATE]` 不算终态。
- 可见思考摘要、进度消息、回执、工具调用说明默认中文。
