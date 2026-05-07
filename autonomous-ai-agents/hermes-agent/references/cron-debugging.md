# Cron Job Debugging — Session Notes

## Diagnosing a Cron Job That Never Runs

### Symptom
`cronjob list` shows job with `state: scheduled`, `last_run_at: null`, and `next_run_at` in the past. The scheduler never ticks.

### Diagnostic Checklist

1. **Check scheduler health**
   - `cronjob list --all` shows all jobs
   - If ALL jobs have `last_run_at: null`, the scheduler process itself is dead/stuck

2. **Check gateway log**
   ```
   tail -100 ~/.hermes/logs/gateway.log
   grep -i "cron\|scheduler\|job" ~/.hermes/logs/gateway.log
   ```

3. **Restart gateway** (user must run manually):
   ```bash
   hermes gateway restart
   ```

4. **Distinguish execution error vs delivery error**
   - `last_status: error` + `last_delivery_error: null` → task execution failed (cron ran but the command errored)
   - `last_status: error` + `last_delivery_error: non-null` → task succeeded but push failed
   - `last_run_at: null` → scheduler never ran the job at all

## Distinguishing Task Failure from Delivery Failure

| Field | Meaning |
|-------|---------|
| `last_status` | Whether the command/prompt itself succeeded or errored |
| `last_delivery_error` | Whether the result was successfully delivered to the target |

A job can have:
- `last_status: error` + `last_delivery_error: null` → command failed, no push attempted
- `last_status: error` + `last_delivery_error: non-null` → command succeeded but push failed
- `last_status: success` + `last_delivery_error: non-null` → rare, edge case

## Verifying WeChat Delivery Channel

Use `send_message` directly (not cron) to test:
```python
send_message(action="send", target="weixin:o9cq802VIxTaQTjfEmDX7CeCVJcQ@im.wechat", message="test")
```
If this succeeds but cron WeChat delivery fails, the issue is in the cron execution environment, not the channel.

## Cron Execution Environment

Cron tasks run in a **headless sub-agent context**, not the current CLI session. This means:
- No PTY attached
- GitHub SSH keys must be available (`~/.ssh/id_ed25519`, etc.)
- No interactive prompts
- Network must be available (git ls-remote to github.com can timeout)

## delegate_task Limitations

`delegate_task` with `toolsets: ["terminal"]` does NOT provide shell access in this environment. The actual cron execution runs in a separate process managed by the cron scheduler. To test cron task logic directly, use `cronjob run` and check results.

## Known Issues

- **croniter must be in hermes venv**: Installing via system pip doesn't work
  ```bash
  /home/ubuntu/.hermes/hermes-agent/venv/bin/python -m pip install croniter
  ```
- **WeChat iLink gateway ret=-2**: Previously seen. Currently resolved. If it recurs, `send_message` will return `ret=-2`.
