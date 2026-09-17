# macctl-control

这是 Linux-macctl 的 GitHub **补充控制通道**，不是 SentinelX 的替代品。

当前用于普通 ChatGPT Plus 聊天窗口中的持久、异步、只读任务：

`ChatGPT → GitHub Issue → GitHub Actions → Hermes self-hosted runner → 固定 dispatcher → macctl / service health → Issue 安全摘要 + Artifact`

## 当前定位

- SentinelX：实时主控制通道。
- GitHub：后台任务、排队、长任务、留档，以及 SentinelX 的应用层备用检查通道。
- Remote Desktop Commander：维修 / break-glass。

## R1 当前允许动作

- `version`
- `status`
- `health`
- `doctor`
- `fleet.status`
- `fleet.doctor`
- `backup.status`
- `linux.fleet-status`
- `linux.fleet-doctor`
- `sentinelx.health`
- `github-runner.health`
- `full.audit`：一次完成 Mac、Linux、备份、SentinelX、GitHub runner 的固定只读巡检。

## R1 任务状态

`macctl:request → macctl:running → macctl:passed / macctl:failed`

同一个 `request_id` 重复触发时不会重新执行，直接返回 Hermes 本地 ledger 中的已有结果；同一个 ID 如果被换成不同请求内容则直接拒绝。

## 结果与 Artifact

- Issue 只回传非敏感摘要和 PASS / FAIL 明细。
- 每次任务生成一份安全摘要 JSON Artifact，默认保留 7 天。
- 完整原始执行结果只保留在 Hermes 本地 ledger，不上传公开仓库。
- Artifact 使用官方 `actions/upload-artifact` 并固定完整 commit SHA。

## 当前仓库策略

- 仓库为 public。
- 当前未启用 GitHub Rulesets，保留 ChatGPT / GitHub 连接对仓库的最大直接维护能力。
- workflow 仅响应仓库所有者本人创建、本人触发的 `macctl:request` 事件。

## 执行边界

- 只接受严格 JSON 请求。
- 不接受 shell、script、任意 path 或任意参数。
- `github-runner` 为普通 Linux 用户，不持有 Mac SSH 私钥。
- 只有固定 `/usr/local/libexec/macctl-github-dispatch` 可通过 sudo 执行。
- Hermes 本地按 `request_id + SHA-256` 做幂等，并通过本地锁串行仲裁。
- 当前不开放 reboot、shutdown、网络修改、文件写入、GUI、Browser mutation、消息发送等实时或高风险动作。

## 请求格式

```json
{
  "schema_version": 1,
  "request_id": "req-example-0001",
  "operation": "full.audit",
  "args": {}
}
```

创建 Issue 后增加 `macctl:request` 标签即可触发。结果由 `github-actions[bot]` 回写到该 Issue。