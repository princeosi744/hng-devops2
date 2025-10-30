## Stage 3 — Observability & Slack Alerts (testing)

1. Copy `.env.example` to `.env` and set:
   - `BLUE_IMAGE` and `GREEN_IMAGE` (as grader supplies)
   - `SLACK_WEBHOOK_URL` (for test Slack channel)
   - optionally adjust `ERROR_RATE_THRESHOLD`, `WINDOW_SIZE`, `ALERT_COOLDOWN_SEC`.

2. Start services:
   ```bash
   docker compose up -d

