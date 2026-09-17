# macctl-control

这是 Linux-macctl 的 GitHub **补充控制通道**，不是 SentinelX 的替代品。

当前 R0 只用于普通 ChatGPT 聊天窗口中的持久、异步、只读任务：

`ChatGPT → GitHub Issue → GitHub Actions → Hermes self-hosted runner → 固定 dispatcher → macctl → Issue 结果`

## 当前定位

- SentinelX：实时主控制通道。
- GitHub：后台任务、排队、留档、SentinelX 的应用层备用通道。
- Remote Desktop Commander：维修 / break-glass。

## R0 允许动作

- `version`
- `status`
- `health`
- `doctor`
- `fleet.status`
- `fleet.doctor`
- `backup.status`
- `linux.fleet-status`
- `linux.fleet-doctor`

## 安全边界

- 只接受严格 JSON 请求。
- 不接受 shell、script、任意 path 或任意参数。
- `github-runner` 为普通 Linux 用户，不持有 Mac SSH 私钥。
- 只有固定 `/usr/local/libexec/macctl-github-dispatch` 可通过 sudo 执行。
- Hermes 本地按 `request_id + SHA-256` 做幂等；同 ID 不同内容直接拒绝。
- GitHub 队列顺序不作为业务顺序依据，Hermes 本地锁与 ledger 才是最终仲裁。
- 当前不开放 reboot、shutdown、网络修改、文件写入、GUI、Browser mutation、消息发送等高风险/实时操作。

## 请求格式

```json
{
  "schema_version": 1,
  "request_id": "req-example-0001",
  "operation": "status",
  "args": {}
}
```

创建 Issue 后增加 `macctl:request` 标签即可触发。结果由 `github-actions[bot]` 回写到该 Issue。
