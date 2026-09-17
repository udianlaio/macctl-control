# R2 Autonomous Maintenance

R2 extends the GitHub supplementary control path from read-only inspection to fixed, typed low/medium-risk maintenance operations. The dispatcher remains the only privileged entry point on Hermes and records every request in the local idempotent ledger.

High-impact operations such as reboot, shutdown, real network switching, security-policy changes, irreversible deletion, and immutable release publication remain outside standing autonomy and require fresh authorization.
