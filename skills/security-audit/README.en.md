# 🦞🔒 Security Audit Skill

One command to run a complete security self-check and hardening for your Lobster.

## Installation

```bash
openclaw skills install npm:guomeiqing-security-audit
```

## Usage

Say any of the following to your Lobster:

- "安全检查"
- "security check"
- "帮我做个安全审计"
- "安全扫描"

## How It Works

This is a **prompt-only skill** (pure SKILL.md) — it contains no scripts. All operations are performed through the Agent's built-in tools:

- `read` — Read configuration files
- `exec` — Check file permissions, back up configs, restart Gateway
- `edit` — Precisely modify configuration items

The Agent will automatically:
1. **🔍 Scan** — Read `openclaw.json` and audit across 7 security dimensions
2. **📊 Report** — Output a scored security report (0–100)
3. **💡 Recommend** — Generate three hardening plans (Basic / Standard / Strict)
4. **🔧 Execute** — Automatically apply config changes and verify after you select a plan

## Three Hardening Plans

| Plan | Time | Best For |
|------|------|----------|
| 🟢 Basic | ~5 min | Fix the most critical issues |
| 🟡 Standard | ~10 min | Recommended! Balances security and usability |
| 🔴 Strict | ~15 min | Maximum security, ideal for public-facing deployments |

---

For more information, see the [repository home](../../README.en.md) and [security article series](../../articles/README.en.md).
