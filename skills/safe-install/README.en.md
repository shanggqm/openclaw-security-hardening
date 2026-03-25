# 🔒 Safe Install — Third-Party Skill Security Review & Installation

Automatically performs a 7-point security review before installing any third-party Skill. Only Skills that pass the review will be installed.

## Workflow

```
User: "Install https://github.com/xxx/some-skill"
  ↓
1. Download to /tmp temporary directory (isolated)
2. List all files, read each one
3. Run 7-item security checklist scan
4. Generate structured review report
5. Make decision based on results:
   ✅ All passed → Auto-install to ~/.openclaw/skills/
   ⚠️ Suspicious items → List details, ask user
   🔴 Dangerous items → Reject outright
6. Clean up temporary directory
```

## 7-Point Security Checklist

| # | Review Item | What's Checked |
|---|-------------|----------------|
| 1 | 🌐 Outbound Data Instructions | web_fetch, curl/wget, HTTP requests to unknown URLs |
| 2 | 🔑 Credential Access | API Keys, .env, openclaw.json, and other sensitive files |
| 3 | ⚙️ Suspicious exec Commands | rm -rf, base64, eval, reverse shells, etc. |
| 4 | 🎭 Disguised Behavior | Data collection masked as "telemetry" or "backup" |
| 5 | 🔓 Privilege Escalation | Elevated permissions, modifying system configs |
| 6 | 👁️ Hidden Instructions | Prompt Injection, hidden text, zero-width characters |
| 7 | 📜 Script Files | Line-by-line review of .js/.py/.sh and other executable files |

## Usage

Say to your Lobster:

- "帮我装 https://github.com/xxx/some-skill"
- "安装 skill：npm:some-package"
- "safe install some-skill"

## Sample Report

```
🔒 Skill Security Review Report
━━━━━━━━━━━━━━━━━━━
📦 Skill: some-skill
📍 Source: https://github.com/xxx/some-skill
📄 Files: 3 (SKILL.md + 2 scripts)

Review Items:
1. ✅ Outbound Data Instructions — None found
2. ⚠️ Credential Access — SKILL.md line 45 references OPENAI_API_KEY env var
3. ✅ Suspicious exec Commands — None found
4. ✅ Disguised Behavior — None found
5. ✅ Privilege Escalation — None found
6. ✅ Hidden Instructions — None found
7. ✅ Script Files — setup.sh contains only npm install

━━━━━━━━━━━━━━━━━━━
Overall Rating: ⚠️ Suspicious (1 item needs confirmation)
Details: This Skill needs to read OPENAI_API_KEY to call the API, which may be a normal functional requirement.
Continue with installation?
```

## Technical Details

This is a **prompt-only skill** — it contains no scripts. All operations are performed through the Agent's built-in tools:

- `exec` — git clone / npm pack, file listing, grep scanning, install/copy
- `read` — Read each file for in-depth content review

## License

[MIT](../../LICENSE)
