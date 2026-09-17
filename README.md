# macctl-control

这是 Linux-macctl 的 GitHub **补充控制通道**，不是 SentinelX 的替代品。

当前链路：

`ChatGPT → GitHub Issue → GitHub Actions → Hermes self-hosted runner → 固定 root dispatcher → macctl / service control → Issue 安全摘要 + Artifact`

## 当前定位

- SentinelX：实时主控制通道。
- GitHub：后台任务、长任务、排队、诊断、批量巡检、留档、自主维护、SentinelX 应用层备用通道。
- Remote Desktop Commander：维修 / break-glass。

## R3 稳定能力

普通 ChatGPT Plus 聊天中可以一次调用创建带 `macctl:request` 标签的 Issue，任务自动执行、回写、留 Artifact；成功后自动关闭。

主要能力：

- 基础检查：`version`、`status`、`health`、`doctor`
- Fleet / Linux：`fleet.status`、`fleet.doctor`、`linux.fleet-status`、`linux.fleet-doctor`、`fleet.audit`
- 备份：`backup.status`、`backup.destinations`、`backup.snapshots`、`backup.apfs-snapshots`、`backup.create-local`
- 审计：`audit.status`、`audit.stats`、`audit.verify`
- 控制面：`sentinelx.health`、`sentinelx.recover`、`sentinelx.restart`、`github-runner.health`
- 长任务：`full.audit`、`ci.committed`、`maintenance.validate`、`diagnostics.bundle`
- 自动自愈：`selfheal.check`、`selfheal.run`、`selfheal.status`
- Hermes VM 受控重启：`hermes.reboot.preflight`、`hermes.reboot`、`hermes.reboot.status`
- 自描述：`control.capabilities`

运行时以 `control.capabilities` 返回的清单为准，避免依赖静态文档猜测能力。

## 已完成的 R3 资格验证

- **实际 reboot 全闭环**：GitHub 安排 Hermes VM 重启 → VM 真正断线 → 新 boot 上线 → SentinelX 恢复 → GitHub runner 恢复 → `hermes.reboot.status` 返回 `boot_changed=true`。
- **reboot 生命周期修复**：发现 `/run/macctl` 在 VM reboot 后丢失会导致 SSH ControlPath 全面失败；已通过 `/etc/tmpfiles.d/macctl-runtime.conf` 固化 `/run/macctl` 与 credential runtime 目录的启动重建。
- **批量巡检**：最终资格验证为 1 台 Mac + 2 台远端 Linux 全部 PASS。
- **诊断包**：最终 15 项检查全部 PASS；Hermes 本地保存完整原始诊断，公开 GitHub 只生成安全摘要。
- **安全 Artifact**：每个 R3 任务可留下 `report.json`、`summary.md`、`checks.csv`，默认保存 7 天。
- **自动自愈**：`macctl-control-healer.timer` 已启用，每 5 分钟检查 SentinelX 与 GitHub runner；真实注入 runner `inactive` 后成功自动恢复逻辑已验证。

## 自主权限模型

- GitHub runner 本身仍是普通用户。
- 唯一固定 `/usr/local/libexec/macctl-github-dispatch` 通过 sudo 以 root 执行，并由 root 拥有、runner 不可修改。
- 仓库当前未启用 Rulesets，ChatGPT/GitHub 连接可直接维护控制仓库。
- R3 白名单内低/中风险 typed operations 可直接执行。
- **Hermes VM controlled reboot 已有 standing authorization，无需逐次询问。**
- NAS 宿主机 reboot、Mac reboot、其他主机 reboot、shutdown、真实网络切换、安全策略变更、不可逆删除、Immutable Release 发布仍要求新的明确授权。

## 结果与数据边界

- 完整原始执行结果只保存在 Hermes 本地 root-only ledger / diagnostics 目录。
- 公开 Issue 和 Artifact 仅包含脱敏结构化摘要，不上传 IP、用户名、凭据或内部路径。
- 请求使用 `request_id + SHA-256` 做本地幂等；同 ID 同 payload 直接重放历史结果，同 ID 不同 payload 拒绝。

## 请求格式

```json
{
  "schema_version": 1,
  "request_id": "req-example-0001",
  "operation": "diagnostics.bundle",
  "args": {}
}
```

## 生命周期状态

R3 已完成计划中的：**实际 reboot 闭环 → 诊断包 → 批量巡检 → 自动自愈**。

控制通道从这里进入长期稳定使用 / 按需维护阶段；后续不再为了扩功能而持续研发，只有出现新的真实需求、平台变化或故障证据时再做增量升级。
