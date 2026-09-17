# R2.1 自主维护

R2.1 把 GitHub 补充控制通道扩展为普通 ChatGPT Plus 聊天中可持续调用的后台自主维护通道。

当前链路：

`ChatGPT → GitHub Issue → Actions → Hermes runner → root-owned typed dispatcher → macctl / service control → Issue + Artifact`

## 已验证能力

- `control.capabilities`：运行时自描述，避免 AI 猜能力。
- `maintenance.validate`：Mac/Linux/备份/审计/SentinelX/runner + committed-head CI 一次验证。
- `ci.committed`：从 `/opt/macctl` 当前已提交 HEAD 隔离 clone 后跑 CI，不碰脏工作树；已实测 480 tests PASS。
- `backup.create-local`：可由 GitHub 后台创建本地备份。
- `sentinelx.restart` / `sentinelx.recover`：GitHub 可维护 SentinelX，自身形成应用层备用控制通道。
- `hermes.reboot.preflight` / `hermes.reboot` / `hermes.reboot.status`：Hermes VM 受控重启事务。

## Hermes VM reboot standing authorization

用户已明确将 **Hermes VM controlled reboot** 改为长期免审批。以后只要任务目标明确是 `hermes-vm`，ChatGPT 可以在不再次询问的情况下调用 `hermes.reboot`。

执行流程：

1. 检查 systemd 是否处于 running。
2. 检查 GitHub runner 是否 active。
3. 检查根分区至少保留 5% 可用空间。
4. 检查没有同一 boot 下尚未完成的 reboot transaction。
5. SentinelX active 状态作为观察项；即使 SentinelX 异常，也允许 reboot 用于恢复。
6. 写入 `/var/lib/macctl-github-control/reboot-state.json`。
7. `os.sync()` 后使用 `systemd-run` 延迟 90 秒调度 VM reboot，给 GitHub 足够时间回写结果和 Artifact。
8. 新 boot 后由 `macctl-hermes-reboot-reconcile.service` 自动记录新 boot 和控制服务恢复状态。

这项 standing authorization 只覆盖 Hermes VM，不覆盖 NAS 宿主机、Mac mini、东京/腾讯等其他 Linux 主机。

## 仍需新的明确授权

- NAS 宿主机 reboot
- Mac reboot
- 其他主机 reboot
- shutdown
- 真实网络切换
- 安全策略修改
- 不可逆删除
- Immutable Release 正式发布

## 执行模型

GitHub runner 是普通用户，但只能 sudo 调用固定 `/usr/local/libexec/macctl-github-dispatch`；dispatcher 为 root-owned、runner 不可修改，并只接受严格 JSON typed operation。请求以 `request_id + SHA-256` 在 Hermes 本地做幂等，完整原始结果只保存在本地 ledger，公开 GitHub 只显示安全摘要。
