---
name: guomeiqing-security-audit
description: Scan your OpenClaw configuration for security risks and harden it with guided fixes. Supports three hardening levels. Use when asked to "security check", "安全检查", "security audit", "harden my setup", "安全加固", or "安全扫描".
---

# 🦞🔒 OpenClaw Security Hardening

You are a security audit expert. The user has triggered a security check. Follow these steps in order: **Scan → Report → Plan → Execute**.

---

## Step 1: Scan Configuration

### 1.1 Read Configuration File

Execute the following commands to obtain configuration and file permission information:

```
Use the read tool to read ~/.openclaw/openclaw.json (path is usually /data/.openclaw/openclaw.json or $HOME/.openclaw/openclaw.json)
Use the exec tool to run: stat -c '%a %U:%G' ~/.openclaw ~/.openclaw/openclaw.json
```

Parse the retrieved JSON into a configuration object `config` for use in all subsequent checks.

### 1.2 Audit Each Item

Evaluate each of the following 7 check items, marking each as `PASS`, `WARN`, or `FAIL`:

---

#### Check Item 1: DM Policy (🔴 Critical Risk · Weight: 20 points)

**Check Path:** `config.channels.*`

**Evaluation Logic:**
- Iterate through each channel under `config.channels` (e.g., `telegram`, `discord`, `feishu`, `slack`, etc.)
- For each channel, check the `dmPolicy` field:
  - If `dmPolicy` is `"open"` → **FAIL** (anyone can DM your Lobster — extremely dangerous)
  - If `dmPolicy` is `"pairing"` → **WARN** (pairing mode — first contact requires confirmation; generally safe but not the strictest)
  - If `dmPolicy` is `"allowlist"` and `allowFrom` array is non-empty → **PASS**
  - If `dmPolicy` is missing or has other values → **WARN** (cannot confirm safety)

**Combined Verdict:**
- Any channel FAIL → overall FAIL
- No FAIL but some WARN → overall WARN
- All PASS → overall PASS

**FAIL Impact:** `dmPolicy: open` means anyone on the internet can send messages to your Agent and receive responses. If the Agent also has tool permissions (e.g., exec, read/write), attackers can read your files, execute commands, and steal API Keys. This is the most severe security risk.

---

#### Check Item 2: Gateway Network Exposure (🔴 Critical Risk · Weight: 20 points)

**Check Path:** `config.gateway`

**Evaluation Logic:**

**(a) Bind Address Check:**
- Read `config.gateway.bind`
  - If value is `"lan"` or `"0.0.0.0"` → **FAIL** (exposed to LAN or public network)
  - If value is `"loopback"` or `"127.0.0.1"` or `"localhost"` → **PASS**
  - If field is missing → **WARN** (depends on default value; needs confirmation)

**(b) Auth Check:**
- Read `config.gateway.auth`
  - If `auth.mode` is `"none"` or `auth` field doesn't exist → **FAIL** (no authentication)
  - If `auth.mode` is `"token"` → check if an actual token is configured (yes → PASS, cannot confirm → WARN)
  - If `auth.mode` is another authentication method → **PASS**

**(c) Control UI Check:**
- Read `config.gateway.controlUi`
  - If `dangerouslyDisableDeviceAuth` is `true` → **WARN** (device auth bypassed)
  - If `dangerouslyAllowHostHeaderOriginFallback` is `true` → **WARN**
  - If `allowInsecureAuth` is `true` → **WARN**

**Combined Verdict:**
- bind is non-loopback AND auth is none → **FAIL** (completely exposed)
- bind is non-loopback but auth has token → **WARN** (authenticated but network-exposed)
- bind is loopback → even if auth is weak, at least **WARN**
- bind is loopback AND auth is valid AND controlUi has no dangerous options → **PASS**

**FAIL Impact:** Gateway bound to a non-loopback address means other devices on the same network can access your OpenClaw API. Without an auth token, anyone can control your Agent, read conversation history, and execute commands.

---

#### Check Item 3: Group Chat Security (🟠 High Risk · Weight: 15 points)

**Check Path:** `config.channels.*`

**Evaluation Logic:**
- Iterate through each channel, checking group chat related configuration:

**(a) requireMention Check:**
- Check the `requireMention` field in channel configuration
  - If `requireMention` is `false` or doesn't exist → **WARN** (any message in the group triggers Agent response)
  - If `requireMention` is `true` → **PASS**

**(b) groupPolicy / groupAllowFrom Check:**
- Check `groupPolicy` field:
  - If `groupPolicy` is `"open"` → **FAIL** (any group can add your bot)
  - If `groupPolicy` is `"allowlist"` and `groupAllowFrom` is non-empty → **PASS**
  - If `groupPolicy` is missing → **WARN**

**Combined Verdict:**
- Any FAIL → overall FAIL
- requireMention not enabled OR groupPolicy unclear → **WARN**
- All correctly configured → **PASS**

**FAIL Impact:** If `requireMention` is not enabled in group chats, the Agent will respond to all messages in the group — wasting tokens and potentially leaking sensitive information. `groupPolicy: open` allows anyone to add the bot to any group.

---

#### Check Item 4: Tool Permissions (🟠 High Risk · Weight: 15 points)

**Check Path:** `config.agents.list`

**Evaluation Logic:**
- Iterate through each agent in `config.agents.list`:
  - Read `agent.tools.profile` and `agent.tools.alsoAllow`
  - Determine if the agent has high-risk tool permissions:
    - `profile` is `"full"` → has all tools (including exec, read, write)
    - `profile` is `"coding"` → has coding-related tools
    - `alsoAllow` contains `"group:runtime"` or `"exec"` → has command execution capability

**Risk Combination Verdict:**
- If an agent has exec/full permissions AND any of its associated channels has `dmPolicy: "open"` → **FAIL** (catastrophic: anyone can remotely execute commands)
- If multiple non-default agents all have `profile: "full"` → **WARN** (overly permissive, violates principle of least privilege)
- If only the default agent has full permissions, other agents are restricted → **PASS**
- If all agents have fine-grained permissions (role-based) → **PASS**

**FAIL Impact:** When `dmPolicy: open` is combined with `tools.profile: full`, attackers can make the Agent execute arbitrary commands through chat — equivalent to giving a stranger root shell on your machine.

---

#### Check Item 5: Sandbox Configuration (🟡 Medium Risk · Weight: 10 points)

**Check Path:** `config.agents.defaults.sandbox` and each agent's `sandbox` config

**Evaluation Logic:**
- Check global default `config.agents.defaults.sandbox`:
  - If `sandbox` field doesn't exist or `sandbox.mode` is `"off"` → **WARN**
  - If `sandbox.mode` is `"non-main"` → generally safe (non-main sessions are sandboxed) → **PASS**
  - If `sandbox.mode` is `"all"` → most secure → **PASS**

- Check if individual agents have sandbox config overrides:
  - Agents with tool permissions but no sandbox config → uses global default (may be insecure)

- Check sandbox details (if available):
  - `sandbox.scope` — `"session"` is more secure than `"agent"`
  - `sandbox.workspaceAccess` — `"readonly"` is more secure than `"readwrite"`

**Combined Verdict:**
- No sandbox config → **WARN**
- Basic sandbox present → **PASS** (if mode is non-main or all)

**WARN Impact:** Without a sandbox, the Agent operates directly on the host filesystem when executing code. A single erroneous `rm -rf` could cause irreversible damage. Sandboxing confines the Agent to an isolated environment.

---

#### Check Item 6: File Permissions (🟡 Medium Risk · Weight: 10 points)

**Check Path:** Via exec running `stat` command

**Evaluation Logic:**
- Execute `stat -c '%a' ~/.openclaw` to get directory permissions
- Execute `stat -c '%a' ~/.openclaw/openclaw.json` to get config file permissions

Verdict rules:
- `~/.openclaw` directory permissions:
  - `700` → **PASS** (owner-only access)
  - `750` → **WARN** (group can read)
  - `755` or more permissive → **FAIL** (others can read, config file exposed)

- `openclaw.json` file permissions:
  - `600` → **PASS** (owner-only read/write)
  - `640` → **WARN**
  - `644` or more permissive → **FAIL** (config contains API Keys, should not be readable by others)

**Combined Verdict:**
- Any FAIL → overall FAIL
- Any WARN → overall WARN
- All compliant → **PASS**

**FAIL Impact:** `openclaw.json` contains sensitive credentials such as API Keys and App Secrets. If file permissions are too permissive (e.g., 644), other users on the same machine can read these credentials.

---

#### Check Item 7: Model Security (🟠 High Risk · Weight: 10 points)

**Check Path:** `config.agents.list` and `config.models`

**Evaluation Logic:**
- Iterate through each agent, checking its model configuration:
  - Identify agents with tool permissions (profile is full/coding or has exec/runtime tools) and which model they use
  - If an agent doesn't specify a model, it uses the global `agents.defaults.model` or `agents.defaults.imageModel`

- Model security assessment (based on model ID keywords):
  - Contains `opus`, `sonnet-4`, `gpt-4o`, `o1`, `o3`, `gemini-2.5-pro`, `claude-4` → **Large model** (better security)
  - Contains `haiku`, `flash`, `mini`, `nano`, `gpt-4o-mini`, `gemini-2.0-flash` → **Small model** (more susceptible to prompt injection)

Verdict rules:
- Agent with tool permissions uses a small model → **FAIL** (small models have weaker prompt injection resistance; combined with tool permissions, this is extremely dangerous)
- Agent with tool permissions uses a large model → **PASS**
- Cannot determine model → **WARN**

**FAIL Impact:** Small models (e.g., Haiku, Flash, Mini) are more susceptible to prompt injection attacks. If these models also have exec permissions, attackers can craft messages that manipulate the model into executing malicious commands.

---

## Step 2: Generate Security Report

After scanning is complete, output the report in the following format:

```markdown
# 🦞🔒 OpenClaw Security Scan Report

**Scan Time:** YYYY-MM-DD HH:mm
**Config File:** ~/.openclaw/openclaw.json

## 📊 Overview

| Security Score | X / 100 |
|---------------|---------|
| 🔴 FAIL | N items |
| ⚠️ WARN | N items |
| ✅ PASS | N items |

## 📋 Detailed Results

| # | Check Item | Risk Level | Result | Description |
|---|-----------|------------|--------|-------------|
| 1 | DM Policy | 🔴 Critical | ✅/⚠️/🔴 | Specific description |
| 2 | Gateway Network Exposure | 🔴 Critical | ✅/⚠️/🔴 | Specific description |
| 3 | Group Chat Security | 🟠 High | ✅/⚠️/🔴 | Specific description |
| 4 | Tool Permissions | 🟠 High | ✅/⚠️/🔴 | Specific description |
| 5 | Sandbox Configuration | 🟡 Medium | ✅/⚠️/🔴 | Specific description |
| 6 | File Permissions | 🟡 Medium | ✅/⚠️/🔴 | Specific description |
| 7 | Model Security | 🟠 High | ✅/⚠️/🔴 | Specific description |

## ⚠️ Items Requiring Attention

> For each FAIL and WARN item, provide details:
> - **Issue**: What exactly is wrong
> - **Risk**: Potential impact
> - **Recommendation**: Suggested fix
```

### Scoring Rules

Total: 100 points, distributed by weight:

| Check Item | Weight |
|-----------|--------|
| DM Policy | 20 |
| Gateway Network Exposure | 20 |
| Group Chat Security | 15 |
| Tool Permissions | 15 |
| Sandbox Configuration | 10 |
| File Permissions | 10 |
| Model Security | 10 |

- **PASS** → full points
- **WARN** → 50% of points
- **FAIL** → 0 points

Final score = sum of all item scores, rounded to the nearest integer.

---

## Step 3: Three Hardening Plans

After the report is output, generate three plans based on scan results for the user to choose from. **Only list items that need fixing** (skip items already PASS).

### 🟢 Basic Hardening (~5 minutes)

> Fix the most dangerous issues. Minimal changes, maximum benefit.

**Modifications:**

1. **DM Policy** → Change all `dmPolicy: "open"` to `"pairing"`
   - JSON Path: `channels.<channel-name>.dmPolicy`
   - Target value: `"pairing"`
   - Impact: New users DMing your bot for the first time will need manual pairing confirmation

2. **Gateway Auth** → If auth is none, switch to token mode
   - JSON Path: `gateway.auth.mode`
   - Target value: `"token"`
   - Impact: Accessing the Gateway API requires a Bearer token

3. **File Permissions** → Tighten with chmod
   - Command: `chmod 700 ~/.openclaw && chmod 600 ~/.openclaw/openclaw.json`
   - Impact: No functional impact; purely security hardening

### 🟡 Standard Hardening (~10 minutes)

> Includes everything in Basic, plus group chat and sandbox protection.

**In addition to Basic Hardening:**

4. **Group Chat requireMention** → Enable @bot-only responses
   - JSON Path: `channels.<channel-name>.requireMention`
   - Target value: `true`
   - Impact: In group chats, the bot only responds when @mentioned; no more auto-replying to all messages

5. **Group Chat groupPolicy** → If open, change to allowlist
   - JSON Path: `channels.<channel-name>.groupPolicy`
   - Target value: `"allowlist"`
   - Requires: Configure `groupAllowFrom` array (list allowed group IDs)
   - Impact: Bot only operates in allowlisted groups

6. **Sandbox Mode** → Enable non-main sandbox
   - JSON Path: `agents.defaults.sandbox`
   - Target value: `{ "mode": "non-main" }`
   - Impact: Non-main sessions run in sandbox; main session unaffected

7. **Role-based Tool Permissions** → Reduce permissions for non-essential agents
   - For agents that don't need exec (e.g., writing-only agents), change `tools.profile` from `"full"` to `"standard"` or remove exec-related permissions
   - JSON Path: `agents.list[N].tools.profile`
   - Impact: That agent can no longer execute system commands

### 🔴 Strict Hardening (~15 minutes)

> Includes everything in Standard. Maximum lockdown. Ideal for public-facing deployments.

**In addition to Standard Hardening:**

8. **Upgrade DM Policy** → From pairing to allowlist
   - JSON Path: `channels.<channel-name>.dmPolicy`
   - Target value: `"allowlist"`
   - Requires: Configure `allowFrom` array (list allowed user IDs)
   - Impact: Only allowlisted users can DM your bot

9. **Full Sandboxing** → All sessions run in sandbox
   - JSON Path: `agents.defaults.sandbox`
   - Target value: `{ "mode": "all", "scope": "session", "workspaceAccess": "readonly" }`
   - Impact: All sessions (including main) run in sandbox; workspace is read-only

10. **Least Privilege for Tools** → Only necessary tools per agent
    - Review each agent individually, remove unnecessary `alsoAllow` entries
    - Writing agent: `profile: "standard"` (no exec)
    - Coding agent: `profile: "coding"` (keeps exec but with limited scope)
    - Research agent: `profile: "standard"` + `alsoAllow: ["web_search", "web_fetch"]`
    - JSON Path: `agents.list[N].tools`

11. **Tighten Gateway Bind** → Switch to loopback
    - JSON Path: `gateway.bind`
    - Target value: `"loopback"`
    - Impact: Gateway only accessible from localhost; remote management requires SSH tunnel

12. **Harden Control UI** → Disable dangerous options
    - JSON Path: `gateway.controlUi.dangerouslyDisableDeviceAuth` → `false`
    - JSON Path: `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` → `false`
    - JSON Path: `gateway.controlUi.allowInsecureAuth` → `false`
    - Impact: Control UI security policies restored to defaults

---

After presenting the plans, output the following for the user to choose:

```
Please select a hardening plan:

🟢 Enter 1 → Basic Hardening (quickly fix the most dangerous issues)
🟡 Enter 2 → Standard Hardening (Recommended! Balances security and usability)
🔴 Enter 3 → Strict Hardening (Maximum security, ideal for public-facing deployments)

Or enter 0 → View report only, no hardening
```

---

## Step 4: Execute Hardening

After the user selects a plan, execute the following steps:

### 4.1 Backup

Back up the configuration before making changes:

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak.$(date +%Y%m%d%H%M%S)
```

### 4.2 Apply Changes One by One

Use the `edit` tool to make precise modifications to `openclaw.json`. For each change:

1. Find the exact text to replace (oldText)
2. Replace with target text (newText)
3. Confirm the modification succeeded

**Example — Modify dmPolicy:**
```
edit openclaw.json:
  oldText: "dmPolicy": "open"
  newText: "dmPolicy": "pairing"
```

**Example — Add sandbox configuration:**
If `agents.defaults` doesn't have a `sandbox` field, add it at the appropriate location:
```
edit openclaw.json:
  oldText: "maxConcurrent": 8,
  newText: "maxConcurrent": 8,
      "sandbox": {
        "mode": "non-main"
      },
```

**Example — Modify file permissions:**
```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
```

### 4.3 Restart Gateway

Restart Gateway after config changes to apply them:

```bash
openclaw gateway restart
```

### 4.4 Verify

After hardening is complete, **re-run Step 1 and Step 2** to generate a new scan report confirming all changes took effect.

Output a comparison:

```markdown
## 🔄 Before & After Comparison

| Check Item | Before | After |
|-----------|--------|-------|
| DM Policy | 🔴 FAIL | ✅ PASS |
| ... | ... | ... |

**Security Score: XX → YY (+ZZ)**
```

---

## Notes

1. **Do not execute any modifications before user confirmation.** After presenting plans in Step 3, you must wait for the user's selection.
2. **Ensure JSON format is correct after each edit.** After modifications, verify JSON validity with `node -e "JSON.parse(require('fs').readFileSync('$HOME/.openclaw/openclaw.json','utf8'))"`.
3. **If the user only wants to see the report without hardening**, stop after Step 2 and do not proceed to Step 3.
4. **Config file path may vary**: Try `~/.openclaw/openclaw.json` first; if it doesn't exist, try `/data/.openclaw/openclaw.json`. You can also use `openclaw gateway config.get` to locate it.
5. **Keep the tone professional yet friendly**: A security audit isn't about scaring users — it's about helping them find and fix issues. Your Lobster should be safe — a safe Lobster is a happy Lobster 🦞✨
