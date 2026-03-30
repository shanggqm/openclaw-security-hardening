> 🇨🇳 [中文版](README.md)

# OpenClaw Security Hardening

Security toolkit for OpenClaw: configuration audit Skill, third-party Skill installation review, and an accompanying security article series.

## Skills

### guomeiqing-security-audit — Configuration Security Audit

Audits `openclaw.json` across 7 security dimensions, produces a scored report, and offers three hardening tiers (Basic / Standard / Strict). Once you pick a tier, it applies the changes and verifies automatically.

Audit items:

| # | Item | Risk Level | Description |
|---|------|-----------|-------------|
| 1 | DM Policy | Critical | Who can DM your bot — `open` means everyone |
| 2 | Gateway Exposure | Critical | Bind address and auth configuration |
| 3 | Group Chat Security | High | requireMention, group allowlist |
| 4 | Tool Permissions | High | Whether Agent tool permissions are overly broad |
| 5 | Sandbox Configuration | Medium | Sandbox mode and isolation level |
| 6 | File Permissions | Medium | Config file chmod |
| 7 | Model Security | High | Whether tool-enabled Agents use under-capable small models |

### guomeiqing-safe-install — Third-Party Skill Security Review

Runs a 7-point security review before installing any community Skill (outbound data, credential access, dangerous commands, disguised behavior, privilege escalation, hidden instructions, script files). Passes clean Skills through; blocks or escalates risky ones.

Both Skills are **prompt-only** (pure SKILL.md) with no script dependencies — all operations go through the Agent's built-in tools (read / edit / exec).

## Installation

**npm (recommended):**

```bash
openclaw skills install npm:guomeiqing-security-audit
```

**Manual:**

```bash
cd ~/.openclaw/skills
mkdir -p guomeiqing-security-audit && cd guomeiqing-security-audit
npm pack guomeiqing-security-audit && tar xzf guomeiqing-security-audit-*.tgz --strip-components=1 package/
rm -f guomeiqing-security-audit-*.tgz
```

Restart Gateway after installation:

```bash
openclaw gateway restart
```

## Usage

Say "security check", "run a security audit", or "security scan" to trigger the audit workflow.

Sample output:

```
OpenClaw Security Scan Report

Scan Time: 2026-03-22 09:00
Config: ~/.openclaw/openclaw.json

Overview
Score: 65 / 100
FAIL: 1 | WARN: 3 | PASS: 3

Details
1. DM Policy        [Critical]  PASS  — allowlist configured
2. Gateway Exposure  [Critical]  WARN  — bind=lan, auth token present
3. Group Chat        [High]      PASS  — groupPolicy=allowlist
4. Tool Permissions  [High]      WARN  — 3 agents with full profile
5. Sandbox Config    [Medium]    FAIL  — no sandbox configured
6. File Permissions  [Medium]    WARN  — dir permission 755, recommend 700
7. Model Security    [High]      PASS  — all tool-enabled agents use large models
```

After the audit you can pick a hardening tier:

| Tier | Time | Use Case |
|------|------|----------|
| Basic | ~5 min | Fix highest-risk items (DM Policy, Gateway Auth, File Permissions) |
| Standard | ~10 min | Basic + Group Chat protection + Sandbox + role-based tool permissions |
| Strict | ~15 min | Standard + full sandboxing + least privilege + Gateway loopback |

## Security Article Series

All 5 articles from the "OpenClaw Deep Dive · Security Series" are archived in [`articles/`](articles/) along with original illustrations.

| # | Title | Topic |
|---|-------|-------|
| 1 | [Is Your Lobster Running Naked?](articles/01-lobster-naked/) | Security overview and hands-on audit |
| 2 | [Three Layers of Armor](articles/02-three-layers-armor/) | Security mental model: Access → Permissions → Sandbox |
| 3 | [Can Your Lobster Be Turned? — Prompt Injection Explained](articles/03-prompt-injection/) | Direct / indirect injection, tool chain attacks, supply chain poisoning |
| 4 | [Is Your Installed Skill Safe?](articles/04-skill-supply-chain/) | Skill supply chain, model risk, hidden instruction auditing |
| 5 | [Don't Let Your Lobster Leak Secrets — Privacy Fences & Data Security](articles/05-privacy-fence/) | Memory leaks, group chat boundaries, key protection, data flow security |

## Roadmap

- [x] guomeiqing-security-audit: one-click security audit + three-tier hardening
- [x] guomeiqing-safe-install: third-party Skill security review
- [x] Security series — all 5 articles complete
- [ ] Threat model documentation
- [ ] Least-privilege assistant Skill
- [ ] Sandbox configuration wizard Skill
- [ ] Security configuration template library

## Contributing

See [CONTRIBUTING.en.md](CONTRIBUTING.en.md). Issues and PRs are welcome.

## Acknowledgments

- [OpenClaw](https://github.com/openclaw/openclaw)

## License

[MIT](LICENSE)
