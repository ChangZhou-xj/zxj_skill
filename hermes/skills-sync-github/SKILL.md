---
name: skills-sync-github
description: "Sync local ~/.hermes/skills directory to a GitHub repository on a schedule."
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  hermes:
    tags: [hermes, skills, git, github, sync, cron]
    trigger: "sync.*skills.*github|publish.*skills.*github|skills.*backup.*github"
---

# Skills Sync to GitHub

Sync the local `~/.hermes/skills` directory to a GitHub repository on a schedule (e.g., weekly).

## Prerequisites

1. **GitHub repository** must exist (create at https://github.com/new)
2. **SSH key or HTTPS token** configured for GitHub push access
   - SSH: `~/.ssh/id_ed25519` with public key added to GitHub
   - Test: `ssh -T git@github.com`
3. **Git identity** set in the skills directory:
   ```bash
   git config user.name "Hermes Agent"
   git config user.email "hermes@agent.local"
   ```

## Cron Job Setup

```bash
hermes cron create "0 10 * * 6"
# Name: zxj-skills-sync
# Prompt: (see below)
# Deliver: weixin:o9cq802VIxTaQTjfEmDX7CeCVJcQ@im.wechat
```

## Sync Prompt (verbatim)

```
将 ~/.hermes/skills 目录下的所有 skill 文件同步发布到 GitHub 仓库 https://github.com/ChangZhou-xj/zxj_skill

步骤：
1. cd ~/.hermes/skills && pwd 确认目录
2. 检查是否为 git 仓库：git rev-parse --git-dir
3. 如果不是 git 仓库，初始化：git init && git remote add origin git@github.com:ChangZhou-xj/zxj_skill.git
4. 设置 git 用户（如果未设置）：git config user.name "Hermes Agent" && git config user.email "hermes@agent.local"
5. git add -A 添加所有变更
6. 检查是否有变更：git diff --cached --stat
7. 如果有变更，提交：git commit -m "skills sync $(date '+%Y-%m-%d %H:%M')"
8. 推送：git push origin main（或master，视远程分支而定）
9. 报告推送结果（成功或失败）

如果 git push 需要认证，先尝试 ssh -T git@github.com 验证连接。
报告完整的执行输出。
```

## Troubleshooting

### Scheduler not running
- If `last_run_at: null` but `next_run_at` is in the past, the scheduler itself is stuck
- User must run: `hermes gateway restart`
- Then re-trigger with: `hermes cron run <job_id>`

### Push fails with authentication error
- Check SSH: `ssh -T git@github.com`
- If SSH works but git push fails, the skills dir may have a different remote URL configured
- Check: `cd ~/.hermes/skills && git remote -v`
- Fix remote: `git remote set-url origin git@github.com:ChangZhou-xj/zxj_skill.git`

### Git not initialized in skills dir
- `~/.hermes/skills` is NOT automatically a git repo
- The prompt handles init, but if remote already exists with wrong URL, init step will fail
- Fix: `cd ~/.hermes/skills && git remote set-url origin git@github.com:ChangZhou-xj/zxj_skill.git`

## Notes

- Skills sync does NOT use `hermes skills install` — it directly `git add -A` all files
- This backs up the raw SKILL.md files and support files verbatim
- Each commit message includes a timestamp for traceability
