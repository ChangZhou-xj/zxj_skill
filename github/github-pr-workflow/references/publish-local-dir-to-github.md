# Publishing a Local Directory to a New GitHub Repo

Use when: you have a local directory (e.g. `~/.hermes/skills`) that is NOT a git repo, and you want to publish it to a GitHub repository the user owns (e.g. `https://github.com/ChangZhou-xj/zxj_skill`).

## Prerequisites

- GitHub user has SSH key configured and added to their GitHub account, OR
- GitHub user has a personal access token (HTTPS method)
- Target repo URL: `git@github.com:ChangZhou-xj/zxj_skill.git` (SSH) or `https://github.com/ChangZhou-xj/zxj_skill.git`

## Step-by-Step

```bash
# 1. Go to the directory
cd ~/.hermes/skills

# 2. Initialize git repo (ONLY if not already a git repo)
git rev-parse --git-dir 2>/dev/null || {
    echo "Not a git repo, initializing..."
    git init
    git remote add origin git@github.com:ChangZhou-xj/zxj_skill.git
}

# 3. Set git identity (required for commits)
git config user.name "Hermes Agent"    # or user's actual name
git config user.email "hermes@agent.local"

# 4. Verify remote is correct
git remote -v

# 5. Add all files
git add -A

# 6. Check what will be committed
git diff --cached --stat

# 7. Commit with timestamp
git commit -m "skills sync $(date '+%Y-%m-%d %H:%M')"

# 8. Try to push
# For SSH:
git push origin main
# If the repo is empty and has no default branch, you may need:
git push -u origin HEAD:main
```

## If the Repo Doesn't Exist Yet (Fresh Repo)

If `git push` fails with `src refspec main does not match any` or the remote repo is completely empty:

```bash
# First check if origin repo even exists / is reachable
ssh -T git@github.com 2>&1

# If push fails because repo has no main branch yet, use:
git push -u origin HEAD:main
# Or if the local default branch is master:
git push -u origin master
```

## Cron Job Integration

When creating a cron job for periodic sync, the job prompt should:
1. Check if `~/.hermes/skills` is a git repo (step 2 above)
2. Conditionally `git init` + `git remote add` only if needed
3. `git add -A && git commit && git push`
4. Report full output so failures are visible

Sample cron prompt fragment:
```
cd ~/.hermes/skills && git rev-parse --git-dir 2>/dev/null || { git init && git remote add origin git@github.com:ChangZhou-xj/zxj_skill.git; }
git add -A && git diff --cached --stat && git commit -m "sync $(date)" && git push origin main
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `src refspec main does not match any` | Repo is empty, no default branch | `git push -u origin HEAD:main` |
| `Permission denied (publickey)` | No SSH key added to GitHub | Add SSH key at github.com/settings/keys |
| `remote: Repository not found` | Repo doesn't exist yet | Create repo on GitHub first or use HTTPS with token |
| `fatal: not a git repository` | Directory not initialized | Run `git init` first |
