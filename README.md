# macctl-control

这是 Linux-macctl 的 GitHub **补充控制通道**，不是 SentinelX 的替代品。

当前用于普通 ChatGPT Plus 聊天窗口中的持久、异步、可排队维护任务：

`ChatGPT → GitHub Issue → GitHub Actions → Hermes self-hosted runner → 固定 root dispatcher → macctl / service control → Issue 安全摘要 + Artifact`

## 当前定位

- SentinelX：实时主控制通道。
- GitHub：后台任务、长任务、排队、留档、自主维护、SentinelX 应用层备用通道。
- Remote Desktop Commander：维修 / break-glass。

## R2 自主能力

普通聊天中可直接调用以下固定 typed operations，无需每次人工审批：

- 基础检查：`version`、`status`、`health`、`doctor`
- Fleet：`fleet.status`、`fleet.doctor`
- Linux：`linux.fleet-status`、`linux.fleet-doctor`
- 备份：`backup.status`、`backup.destinations`、`backup.snapshots`、`backup.apfs-snapshots`、`backup.create-local`
- 审计：`audit.status`、`audit.stats`、`audit.verify`
- 控制面：`sentinelx.health`、`sentinelx.recover`、`sentinelx.restart`、`github-runner.health`
- 长任务：`full.audit`、`ci.committed`、`maintenance.validate`
- 自描述：`control.capabilities`

`ci.committed` 会从 `/opt/macctl` 当前已提交 HEAD 单独 clone 到隔离目录后运行 CI，避免碰现有脏工作树。`maintenance.validate` 会把 Mac/Linux/备份/审计/控制面检查与 committed-head CI 合并成一次后台验证任务。

## 任务状态

`macctl:request → macctl:running → macctl:passed / macctl:failed`

同一个 `request_id` 重复触发时不会重新执行，直接返回 Hermes 本地 ledger 中已有结果；同 ID 不同 payload 直接拒绝。

## 自主权限模型

- GitHub runner 本身仍是普通用户。
- 唯一固定 `/usr/local/libexec/macctl-github-dispatch` 通过 sudo 以 root 执行，并由 root 拥有、runner 不可修改。
- ChatGPT/GitHub 连接当前可直接维护这个公开控制仓库，仓库未启用 Rulesets。
- R2 白名单内的低/中风险维护操作可以直接执行。
- reboot、shutdown、真实网络切换、安全策略变更、不可逆删除、Immutable Release 发布仍要求新的明确授权；长期授权不会替代这类动作的当次授权。

## 结果与 Artifact

- Issue 只回传非敏感摘要和 PASS / FAIL 明细。
- 每次任务生成安全摘要 JSON Artifact，默认保留 7 天。
- 完整原始执行结果只保留在 Hermes 本地 ledger，不上传公开仓库。
- Artifact 使用官方 `actions/upload-artifact` 并固定完整 commit SHA。

## 请求格式

```json
{
  "schema_version": 1,
  "request_id": "req-example-0001",
  "operation": "maintenance.validate",
  "args": {}
}
```

创建 Issue 后增加 `macctl:request` 标签即可触发。结果由 `github-actions[bot]` 回写到该 Issue。