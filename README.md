# macctl-control

这是 Linux-macctl 的 GitHub **补充控制通道**，不是 SentinelX 的替代品。

当前用于普通 ChatGPT Plus 聊天窗口中的持久、异步、可排队维护任务：

`ChatGPT → GitHub Issue → GitHub Actions → Hermes self-hosted runner → 固定 root dispatcher → macctl / service control → Issue 安全摘要 + Artifact`

## 当前定位

- SentinelX：实时主控制通道。
- GitHub：后台任务、长任务、排队、留档、自主维护、SentinelX 应用层备用通道。
- Remote Desktop Commander：维修 / break-glass。

## R2.1 自主能力

普通聊天中可直接调用固定 typed operations，无需每次人工审批。主要包括：

- 基础检查：`version`、`status`、`health`、`doctor`
- Fleet / Linux：`fleet.status`、`fleet.doctor`、`linux.fleet-status`、`linux.fleet-doctor`
- 备份：`backup.status`、`backup.destinations`、`backup.snapshots`、`backup.apfs-snapshots`、`backup.create-local`
- 审计：`audit.status`、`audit.stats`、`audit.verify`
- 控制面：`sentinelx.health`、`sentinelx.recover`、`sentinelx.restart`、`github-runner.health`
- 长任务：`full.audit`、`ci.committed`、`maintenance.validate`
- Hermes VM 受控重启：`hermes.reboot.preflight`、`hermes.reboot`、`hermes.reboot.status`
- 自描述：`control.capabilities`

`hermes.reboot` 只指向 **Hermes VM**。它使用长期授权，不再要求逐次人工审批；执行前会检查 systemd 状态、GitHub runner、根分区可用空间和重复 reboot 事务，SentinelX 状态作为观察项。通过后写入本地 reboot transaction，使用 systemd 延迟调度重启，启动后由 `macctl-hermes-reboot-reconcile.service` 记录是否进入新的 boot。

该长期授权 **不适用于** NAS 宿主机、Mac mini 或其他 Linux 主机。

## 任务状态

`macctl:request → macctl:running → macctl:passed / macctl:failed`

同一个 `request_id` 重复触发时不会重新执行；同 ID 不同 payload 直接拒绝。成功任务自动关闭 Issue，并生成 7 天安全摘要 Artifact。

## 自主权限模型

- GitHub runner 本身仍是普通用户。
- 唯一固定 `/usr/local/libexec/macctl-github-dispatch` 通过 sudo 以 root 执行，并由 root 拥有、runner 不可修改。
- ChatGPT/GitHub 连接当前可直接维护这个公开控制仓库，仓库未启用 Rulesets。
- R2.1 白名单内操作可直接执行。
- **Hermes VM controlled reboot 已有 standing authorization，无需再次询问。**
- NAS 宿主机 reboot、Mac reboot、其他主机 reboot、shutdown、真实网络切换、安全策略变更、不可逆删除、Immutable Release 发布仍要求新的明确授权。

## 请求格式

```json
{
  "schema_version": 1,
  "request_id": "req-example-0001",
  "operation": "maintenance.validate",
  "args": {}
}
```

在普通聊天中创建 Issue 时直接带 `macctl:request` 标签即可一次调用触发；成功后 GitHub 自动回写、留 Artifact 并关闭 Issue。
