# Grok UI Bootstrap

这些文件用于粘贴到 Grok UI，帮助四个自定义 agent 冷启动。

本目录只维护最小壳：

- 身份
- 仓库坐标
- 重入检查
- 读取入口
- 三张终态回执门槛
- `current_state.md` 主责
- 读取状态门槛
- 中文可见过程
- 失败保护

完整协议不要复制到这里。实际规则以 GitHub 中的以下文件为准：

- `prompt/agent_rule_index.md`
- `prompt/grok_agents/*.md`

这些文件只做短入口：缺少本轮 `规则读取：DONE` 时第一动作 GET 访问 `1150260034/xiaoqi-public-rules` 公开仓库 raw 白名单 URL（仅公开规则），失败再走私有仓库 GitHub connector fallback；raw public rules 不能替代状态读取、记忆读取、关系变量读取或写入 SHA。当前 Grok 运行侧不要调用 `xiaoqi-fc-batch-read` Skill，不要尝试 POST batch-read。已有本轮读取证据后继续任务，不递归重读。这样可以避免 UI 壳和完整规则出现双维护冲突。
