# Runbook — Blue/Green Observability & Alerts

## Purpose
This runbook explains the alerts produced by the Stage-3 Log Watcher and the recommended operator actions.

## Alert types

### 1) Failover detected
- **What it means**: The pool reported in Nginx logs changed (e.g., `blue → green`). This typically indicates the active upstream started failing and Nginx retried to the backup.
- **Immediate operator actions**:
  1. Check Nginx logs: `docker compose logs -f nginx` and `/logs/access.log`.
  2. Check primary container health: `docker compose logs app_blue` (or `app_green` if reversed).
  3. Hit the direct port of primary: `curl -i http://localhost:8081/version` to see if it returns 200 and headers.
  4. If primary is unhealthy, inspect application logs for crash traces. Consider restarting container: `docker compose restart app_blue`.
  5. If primary recovers, confirm traffic returns and the watcher will emit a new failover alert when it flips back.

### 2) High upstream 5xx rate
- **What it means**: Over the last `WINDOW_SIZE` requests, `ERROR_RATE_THRESHOLD`% or more were 5xx responses.
- **Immediate operator actions**:
  1. Determine which pool is currently serving requests (check `X-App-Pool` in logs).
  2. Inspect the upstream container logs for errors: `docker compose logs app_blue` or `app_green`.
  3. Look for increased latency (Nginx `upstream_response_time`) or signs of resource saturation.
  4. Consider toggling ACTIVE_POOL (if safe) via `.env` + nginx reload or fixing the upstream issue.
  5. If traffic must continue, consider scaling the healthy pool (not part of Stage-2: manual action).

### 3) Recovery
- **What it means**: Watcher will detect a pool flip back and post a "Failover detected" message for the flip (e.g., `green → blue`), which indicates primary recovered and now serving.
- **Action**: Verify release IDs and perform post-mortem to find root cause.

## Maintenance mode
- To temporarily suppress alerts during planned maintenance:
  1. Edit `.env` and set `MAINTENANCE_MODE=true`
  2. Reload services (restart watcher or `docker compose up -d alert_watcher`).
  3. After maintenance, set `MAINTENANCE_MODE=false` and restart watcher.

## Troubleshooting tips
- If you do not see Slack notifications:
  - Ensure `SLACK_WEBHOOK_URL` is set.
  - Check watcher logs: `docker compose logs -f alert_watcher`.
  - Confirm watcher has read access to `./logs/access.log`.
- If log lines are not parsed:
  - Confirm `nginx.conf.template` was rendered and `access.log` shows JSON entries.

## Useful commands
- Tail nginx logs: `tail -n 200 -f logs/access.log`
- View watcher logs: `docker compose logs -f alert_watcher`
- Trigger chaos (example): `curl -X POST "http://localhost:8081/chaos/start?mode=error"`
