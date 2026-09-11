# ⚠️ 待并入 `DECISION_LOG.md` 的条目

## 现状（2026-09-11 发现）

`DECISION_LOG.md` 当前 **175KB**，本会话**无法直接追加**。原因有两层，都已核实：

1. **本会话的 git 凭证是只读的。** `git push` 返回 `RPC failed; HTTP 403`；GitHub API 查询本会话 token 对本仓库的权限，明确返回 `permissions.push = false`。
   - ⚠️ 注意：git **第一次**给出的错误是误导性的 —— 本地 HEAD 的父提交就是 remote main（标准 fast-forward），git 却报 `non-fast-forward ... tip is behind its remote counterpart`。只有看完整输出才会显出真实的 403。
   - 已排除 egress 代理的嫌疑：`git fetch` / `git pull` 全程正常，代理的 `recentRelayFailures` 为空。
2. **唯一可用的写回路径（GitHub MCP API `create_or_update_file`）必须整文件重写**，而 175KB 超出单次可写出的体量。`PORTFOLIO.md`（30KB）与 `SPRINT_1700.md`（10KB）都在可写范围内，已正常更新。

## 待办

| 文件 | 内容 | 处理方式 |
|---|---|---|
| `DECISION_LOG_2026-09-11.md` | 2026-09-11 完整决策日志条目 | 把文件内容（去掉顶部的说明引用块）**原样贴到 `DECISION_LOG.md` 文件头 `---` 之后**，即最新在最上；然后删除该文件 |

## ⚠️ 一处待更正的措辞（`PORTFOLIO.md`）

`PORTFOLIO.md` 的「数据管道说明」里 2026-09-11 那条，把 403 的原因写成了 **「被 egress 策略禁止」**，并称「代理 README 明确说明这是组织 egress 策略拒绝」。

**这个归因是错的，以本文件上面第 1 条为准：真实原因是本会话的 git 凭证只读（`permissions.push = false`），403 来自 GitHub 本身，不是 egress 代理。**

写那条时我只看了代理 README 里「403/407 = egress 策略拒绝」的说明就下了结论；随后查代理的 `recentRelayFailures` 为空、并直接查到 token 权限，才定位到真正原因。**下次重写 `PORTFOLIO.md` 时把那段改过来**（本次没有为了改一段话而整文件重写，避免在一个其余内容正确的文件上引入新的转写错误）。

## 建议（需用户决定，本次未擅自执行）

`DECISION_LOG.md` 每天增长约 7KB，即使以后拿到 push 权限，单文件也会越来越难维护。两个可选方向：

- **A（推荐）**：按月切分 —— `DECISION_LOG.md` 只保留当月，历史归档为 `DECISION_LOG_2026-08.md` 等。这样单文件始终在 API 可写范围内，本问题永久消失。
- **B**：保持单文件，但确保每天的运行会话具备 git push 权限（在环境配置里给本仓库 push access）。

**本次没有动 `DECISION_LOG.md` 的任何历史内容** —— 它在远端仍是 2026-09-10 之前的完整原样。

## 每日运行的注意事项（写给下一次运行）

- **不要再试 `git push`**，直接用 GitHub MCP API `create_or_update_file`（需带当前 blob 的 `sha`）。
- **判断写回成败，一律以 `git ls-remote` 的实际 ref 或 API 返回的 commit sha 为准，绝不看命令输出的措辞。**
- 价格过期时，改写根目录 `.fetch-request`（**不要动 `.github/`**）即可强制抓一轮，约 2 分钟见效；比 `workflow_dispatch` 省事。
