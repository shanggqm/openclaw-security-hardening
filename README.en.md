> 🇨🇳 [中文版](README.md)

# 🦞🔒 OpenClaw Security Hardening

OpenClaw security ecosystem: configuration security audit, third-party Skill security review, and security article series. From the ["OpenClaw Deep Dive" Security Series](https://mp.weixin.qq.com/).

## What Is This?

A collection of OpenClaw security Skills, containing two prompt-only Skills:

### 🔍 Security Audit — Configuration Health Check

Just say "security check" and the Agent will automatically:

1. **🔍 Scan** — Read `openclaw.json` and audit across 7 security dimensions
2. **📊 Report** — Output a scored security report (0-100), each item marked ✅ PASS / ⚠️ WARN / 🔴 FAIL
3. **💡 Plan** — Generate three-tier hardening plans based on scan results (Basic / Standard / Strict)
4. **🔧 Execute** — After you choose a plan, the Agent automatically modifies configuration and verifies

### 🔒 Safe Install — Third-Party Skill Security Review

Just say "install xxx skill" and the Agent won't install directly. Instead:

1. **📥 Isolated Download** — Downloads to a /tmp temporary directory
2. **🔍 7-Point Review** — Outbound data, credential access, dangerous commands, disguised behavior, privilege escalation, hidden instructions, script files
3. **📊 Review Report** — Each item marked ✅ Safe / ⚠️ Suspicious / 🔴 Dangerous
4. **✅ Decision** — Safe → auto-install, Suspicious → user confirmation, Dangerous → reject

**Zero scripts, zero dependencies** — relies entirely on the Agent's built-in tools (read/edit/exec) for all operations.

## Audit Items

| # | Audit Item | Risk Level | Description |
|---|-----------|------------|-------------|
| 1 | DM Policy | 🔴 Critical | Who can DM your bot? `open` = everyone |
| 2 | Gateway Network Exposure | 🔴 Critical | Gateway bind address and auth configuration |
| 3 | Group Chat Security | 🟠 High | requireMention, group allowlist |
| 4 | Tool Permissions | 🟠 High | Whether Agent tool permissions are overly broad |
| 5 | Sandbox Configuration | 🟡 Medium | Sandbox mode and isolation level |
| 6 | File Permissions | 🟡 Medium | chmod permissions for config files |
| 7 | Model Security | 🟠 High | Whether tool-enabled Agents use small models |

## Installation

### Option 1: npm Install (Recommended)

```bash
openclaw skills install npm:guomeiqing-security-audit
```

### Option 2: Let Your Lobster Install It

Just tell your Lobster:

> Install this security audit skill: npm:guomeiqing-security-audit, then run a security check for me right away.

### Option 3: Manual Installation

```bash
# Download and extract to skills directory
cd ~/.openclaw/skills
mkdir -p security-audit && cd security-audit
npm pack guomeiqing-security-audit && tar xzf guomeiqing-security-audit-*.tgz --strip-components=1 package/
rm -f guomeiqing-security-audit-*.tgz
```

Restart Gateway after installation:

```bash
openclaw gateway restart
```

Verify installation:

```bash
openclaw skills list | grep security
```

## Usage

Say any of the following to your Lobster:

- "security check"
- "run a security audit"
- "harden my setup"
- "security scan"

The Agent will automatically start scanning and output a report.

## Sample Output

### Scan Report

```
🦞🔒 OpenClaw Security Scan Report

Scan Time: 2026-03-22 09:00
Config File: ~/.openclaw/openclaw.json

📊 Overview

| Security Score | 65 / 100 |
|---------------|----------|
| 🔴 FAIL | 1 item   |
| ⚠️ WARN | 3 items  |
| ✅ PASS | 3 items  |

📋 Detailed Results

| # | Audit Item        | Risk Level | Result | Description                          |
|---|-------------------|-----------|--------|--------------------------------------|
| 1 | DM Policy         | 🔴 Critical | ✅   | allowlist configured                  |
| 2 | Gateway Exposure  | 🔴 Critical | ⚠️   | bind=lan, but auth token configured   |
| 3 | Group Chat        | 🟠 High    | ✅   | groupPolicy=allowlist configured      |
| 4 | Tool Permissions  | 🟠 High    | ⚠️   | 3 agents have full profile            |
| 5 | Sandbox Config    | 🟡 Medium  | 🔴   | No sandbox configured                 |
| 6 | File Permissions  | 🟡 Medium  | ⚠️   | Directory permission 755, suggest 700 |
| 7 | Model Security    | 🟠 High    | ✅   | All tool-enabled agents use large models |
```

### Hardening Plan Selection

```
Choose a hardening plan:

🟢 Enter 1 → Basic Hardening (quick fix for the most dangerous issues)
🟡 Enter 2 → Standard Hardening (Recommended! Balance security and usability)
🔴 Enter 3 → Strict Hardening (Maximum security, for public-facing deployments)

Or enter 0 → View report only, skip hardening
```

## Three-Tier Hardening Plans

| Plan | Time | Use Case |
|------|------|----------|
| 🟢 Basic Hardening | ~5 min | Fix the most dangerous issues (DM Policy, Gateway Auth, File Permissions) |
| 🟡 Standard Hardening | ~10 min | Basic + Group Chat Protection + Sandbox + Role-based Tool Permissions |
| 🔴 Strict Hardening | ~15 min | Standard + Full Sandboxing + Least Privilege + Gateway Loopback |

## Technical Details

This is a **prompt-only skill** (pure SKILL.md) with no scripts. All operations are performed through the Agent's built-in tools:

- `read` — Read configuration files
- `exec` — Check file permissions, backup config, restart Gateway
- `edit` — Precisely modify configuration items

The Agent automatically backs up the config file before modifications and re-scans after hardening to confirm.

---

## 📚 Security Article Series

This repository also hosts all articles from the "OpenClaw Deep Dive · Security Series", with original illustrations archived.

| # | Title | Core Topic | Status |
|---|-------|-----------|--------|
| 1 | [Is Your Lobster Running Naked?](articles/01-lobster-naked/) | Security overview: run an audit to see how exposed you are | ✅ Published |
| 2 | [Three Layers of Armor](articles/02-three-layers-armor/) | Security mental model: Access Control → Permissions → Sandbox | ✅ Published |
| 3 | [Can Your Lobster Be Turned? — Prompt Injection Explained](articles/03-prompt-injection/) | Direct injection, indirect injection, tool chain attacks, Skill supply chain | ✅ Published |
| 4 | [Is Your Installed Skill Safe?](articles/04-skill-supply-chain/) | Skill supply chain attacks, model security, hidden instruction auditing | ✅ Published |
| 5 | Don't Let Your Lobster Leak Secrets — Privacy Fences & Data Security | Memory leaks, group chat boundaries, key protection, data flow security | 📝 Planned |

---

## 🗺️ Roadmap

- [x] security-audit Skill: One-click security health check + three-tier hardening
- [x] safe-install Skill: Third-party Skill security review and installation
- [x] Security series (Articles 1-2: Configuration Security)
- [x] Security series (Article 3: Prompt Injection)
- [x] Security series (Article 4: Trust Chain & Model Security)
- [ ] Security series (Article 5: Privacy Fences)
- [ ] Threat model documentation
- [ ] Least-privilege assistant Skill
- [ ] Sandbox configuration wizard Skill
- [ ] Security configuration template library

## Contributing

See [CONTRIBUTING.en.md](CONTRIBUTING.en.md). Issues, PRs, and security hardening tips are all welcome.

## Acknowledgments

- [OpenClaw](https://github.com/openclaw/openclaw) — Making AI Agents accessible
- Readers of the "OpenClaw Deep Dive" Security Series

Lobsters need security — secure Lobsters are happy Lobsters 🦞✨

## License

[MIT](LICENSE)
