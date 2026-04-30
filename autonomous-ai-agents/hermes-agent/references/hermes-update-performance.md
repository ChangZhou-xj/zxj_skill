# Hermes Update Performance — Fix Applied

## Problem

`hermes update` took ~4.7s every time, even when already up to date.

## Root Cause

| Operation | Latency | Why |
|-----------|---------|-----|
| Banner check: `git ls-remote https://github.com/...` | ~5s | HTTPS timeout to github.com |
| `git fetch origin` in `cmd_update` | ~4s | SSH跨境到github.com |
| Total | ~9-10s potential | Banner + fetch combined |

## Fixes Applied (code changes this session)

### 1. Banner cache: 6h → 24h
**File:** `~/.hermes/hermes-agent/hermes_cli/banner.py`

```python
# Before:
_UPDATE_CHECK_CACHE_SECONDS = 6 * 3600

# After:
_UPDATE_CHECK_CACHE_SECONDS = 24 * 3600
```

### 2. `hermes update` fast path
**File:** `~/.hermes/hermes-agent/hermes_cli/main.py`

Added to `cmd_update()` — if `check_for_updates()` returns 0 (up-to-date) and `--force` not passed, exit immediately without `git fetch`:

```python
if not getattr(args, "force", False):
    try:
        from hermes_cli.banner import check_for_updates
        cached = check_for_updates()
        if cached == 0:
            print("✓ Already up to date (cached).")
            print("  Use `hermes update --force` to force a fresh check.")
            return
    except Exception:
        pass  # Cache unavailable — proceed with normal update
```

Added `--force` flag to the update parser:
```python
update_parser.add_argument(
    "--force",
    action="store_true",
    default=False,
    help="Force a fresh update check even when cached data says we're up to date",
)
```

## Performance Results

| Scenario | Before | After |
|----------|--------|-------|
| `hermes update` (up-to-date, cached) | ~4.7s | **0.38s** |
| `hermes update --force` | 4.7s | 4.6s |
| `hermes --version` (warm, gateway running) | varies | ~0.75s |

## Key Files Modified

- `~/.hermes/hermes-agent/hermes_cli/banner.py` — cache duration
- `~/.hermes/hermes-agent/hermes_cli/main.py` — cmd_update fast path + --force flag

## Cache File

`~/.hermes/.update_check` — JSON with `ts`, `behind`, `rev` fields. 24-hour TTL.

## Manual Cache Override

If github.com is completely unreachable, write the cache to skip all network calls:
```
{"ts": <current_unix_epoch>, "behind": 0, "rev": null}
```
Example epoch: `1777535538` (2026-04-30 07:52:18 UTC).
