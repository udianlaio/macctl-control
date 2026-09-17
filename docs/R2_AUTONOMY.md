# R2 自主维护

R2 把 GitHub 补充控制通道从只读查询升级为普通 ChatGPT Plus 聊天中可持续调用的后台维护通道。

当前链路：

`ChatGPT → GitHub Issue → Actions → Hermes runner → root-owned typed dispatcher → macctl / service control → Issue + Artifact`

## 已验证能力

- `control.capabilities`：运行时自描述，避免 AI 猜能力。
- `maintenance.validate`：Mac/Linux/备份/审计/SentinelX/runner + committed-head CI 一次验证。
- `ci.committed`：从 `/opt/macctl` 当前已提交 HEAD 隔离 clone 后跑 CI，不碰脏工作树；当前实测 480 tests PASS。
- `backup.create-local`：可由 GitHub 后台创建本地备份。
- `sentinelx.restart` / `sentinelx.recover`：GitHub 可维护 SentinelX，自身形成应用层备用控制通道。
- 其余 read-only 状态、审计、备份列表和 fleet 检查由 `control.capabilities` 返回最新清单。

## 执行模型

GitHub runner 是普通用户，但只能 sudo 调用固定 `/usr/local/libexec/macctl-github-dispatch`；dispatcher 为 root-owned、runner 不可修改，并只接受严格 JSON typed operation。请求以 `request_id + SHA-256` 在 Hermes 本地做幂等，完整原始结果只保存在本地 ledger，公开 GitHub 只显示安全摘要。

## 自主权限边界

R2 白名单内的低/中风险维护操作无需逐次人工审批。reboot、shutdown、真实网络切换、安全策略修改、不可逆删除、Immutable Release 发布仍要求新的明确授权，不能用长期授权替代当次授权。
