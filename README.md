# gcs-claude-stats

A two-line statusline for [Claude Code](https://claude.com/claude-code) that shows model, project, context usage, 5-hour and 7-day quotas, and session duration — all counting in the same direction so you can read it at a glance.

```
◆ Opus 4.7 │ gcs-claude-stats/Development
▱▱▱▱▱ 0% │ 5h: 22% (3h) │ 7d: 39% (5d) │ ⏱ 2h44m
```

## What you see

**Line 1:** model · project / git branch

**Line 2:**
- `▰▰▱▱▱ 40%` — context window **used** (bar fills as you consume)
- `5h: 22% (3h)` — 5-hour rolling quota used, with time until reset
- `7d: 39% (5d)` — 7-day rolling quota used, with time until reset
- `⏱ 2h44m` — current Claude Code session duration

All three percentages count **up** as you use them. The countdown (`(3h)`, `(5d)`) is time-until-reset.

## Install

Requires Python 3 (any 3.x).

```bash
git clone https://github.com/gcampton/gcs-claude-stats.git ~/.claude/hooks/gcs-claude-stats
```

Then add this to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "python3 /home/YOUR_USER/.claude/hooks/gcs-claude-stats/statusline.py",
    "padding": 0
  }
}
```

(Or use the included `statusline.sh` / `statusline.cmd` launchers, which auto-detect `python3` / `python` / `py`.)

Restart Claude Code to load the new statusline.

## Configuration

Toggle features via environment variables. Set them in your shell profile or in Claude Code's settings.

| Variable | Default | What it does |
|---|---|---|
| `CQB_CONTEXT_SIZE` | `0` | Show `of 200K` next to the context % |
| `CQB_TOKENS` | `1` | Show `↑in ↓out` token counts |
| `CQB_PACE` | `0` | Show pace indicator (you're ahead/behind expected usage) |
| `CQB_RESET` | `1` | Show reset countdowns `(3h)` `(5d)` |
| `CQB_DURATION` | `1` | Show session duration `⏱ 2h44m` |
| `CQB_COST` | `0` | Show session cost in USD |
| `CQB_BRANCH` | `1` | Show git branch on line 1 |

(Defaults match what's shown in the example above.)

## How it gets quota data

The script calls Anthropic's usage API on first run and caches the response in `/tmp/claude-sl-usage.json` for 5 minutes. The fetch happens in a background thread so it doesn't slow the statusline render. Subsequent renders read from cache.

If the API call fails (network down, auth issue, etc.) you get `5h: --` / `7d: --` and everything else still works.

## What's different from upstream

This is a fork of [aiedwardyi/claude-usage-monitor](https://github.com/aiedwardyi/claude-usage-monitor). All upstream functionality is preserved. Changes:

1. **Context bar fills as USED, not remaining.** Upstream shows `▰▰▰▰▰ 100%` for an empty context (full bar = lots of room left). This fork shows `▱▱▱▱▱ 0%` for an empty context (empty bar = nothing used). Now all three percentages on the line count in the same direction.

2. **7-day reset countdown always shown.** Upstream hides the `(Xd)` reset for 7d unless you're past 70% used. This fork always shows it, so you don't wonder whether the missing time means "you're fine" or "I don't know."

3. **Session duration prefixed with `⏱`.** Upstream renders the duration as a bare value at the end of line 2 (e.g. `2h44m`), which is easy to misread as the 7-day reset time. The clock glyph makes it visually distinct.

## Credits

- Upstream: **[aiedwardyi/claude-usage-monitor](https://github.com/aiedwardyi/claude-usage-monitor)** by Edward Yi (MIT)
- Fork modifications: gcampton

## License

MIT — see [LICENSE](LICENSE).
