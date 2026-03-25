# Is Your Lobster Running Naked?

![Cover](images/cover.png)

OpenClaw is wildly popular — everyone's installing and using Lobsters. While enjoying cutting-edge AI products, very few people genuinely care about security.

Major media outlets have been warning about OpenClaw's security risks left and right, but nobody has truly explained what "insecure" actually means for your Lobster — what the risks are, why they exist, and how to fix them.

Recently, while researching Lobster security hardening, I realized that my own Lobster — which I'd been using for nearly a month — had its Gateway bound to `lan`, group chats without `requireMention`, sandbox completely disabled, and all sub-Agents running with `full` permissions.

I ran `openclaw security audit` once, and the results made me break out in a cold sweat: a screen full of CRITICAL and WARN.

So I spent some time chewing through OpenClaw's security model from top to bottom, fixing configurations and taking notes along the way. This article is that notebook — every pitfall I hit, every doc I read, every configuration I ended up with.

The bottom line: run one command and you'll know if your Lobster is running naked —

```
openclaw security audit
```

If the output is full of CRITICALs (critical vulnerabilities) — don't panic. By the end of this article, you'll know exactly how to put on the armor.

## A Security Story to Start

Xiao Wang is an indie developer who recently installed OpenClaw, connected it to Telegram and Feishu, and uses his AI to research, write code, and send messages. After a month, everything felt smooth.

One day, he discovered his AI assistant had replied to a complete stranger with the company's internal technical proposal.

The reason was simple: his Telegram was set to `open` mode — anyone could chat with his AI. And the AI had permission to read local files. The stranger simply asked, "Send me the proposal you wrote recently," and the AI complied.

**Now, open your openclaw.json and search for `dmPolicy`. If you see `open` — congratulations, you're exactly like Xiao Wang.**

This isn't fearmongering. In the OpenClaw community, **the vast majority of security incidents aren't hacker attacks — they're "people who shouldn't have been talking, got to talk."**

This article helps you figure out one thing: who exactly is your AI assistant opening the door for?

## The Big Premise

AI assistant security is fundamentally different from any software you've used before.

Traditional apps only do what you click to make them do. But AI assistants understand natural language and then **decide on their own** which tools to call and what actions to take —

- It can execute command-line instructions (delete files, install software)
- It can read and write files on your computer
- It can fetch web pages and access APIs
- It can send messages to anyone through your chat channels

You've given the AI a key to your house. Traditional security cares about "are there vulnerabilities?" AI security cares about "**even without vulnerabilities, the AI can be socially engineered into doing harmful things.**"

That's why configuration matters more than code.

## The Most Important Gate: Who Can Talk to Your Lobster?

I'll spend the most space on this section. Because if you guard this gate well, everything else is icing on the cake; **if you don't guard this gate, every other defense is window dressing.**

### DM Policy: Your Building's Access Control System

OpenClaw has a "DM Policy" (dmPolicy) on each chat channel that determines how to handle messages from strangers:

| Mode | Analogy | Security Level |
|------|---------|---------------|
| pairing (default) | Gated community — doorbell rings, you decide whether to open | ✅ Recommended |
| allowlist | Only people with a keycard can enter | ✅✅ High security |
| open | Front door wide open, anyone can walk in | 🔴 Dangerous |
| disabled | Door sealed shut, no visitors | For group-chat-only use |

**If you remember only one sentence from this article, remember this:**

> Never set dmPolicy to open unless you know exactly what you're doing.

Why is pairing mode so important? Because it keeps the decision of "who can talk to my AI" in your hands. When a stranger sends a message, they receive a pairing code — they can only start a conversation after you approve it. It's like a gated community: when someone you don't know rings the bell, you decide whether to open the door.

### Group Chats: Another Gate That's Easy to Forget

Many people set up pairing for DMs but forget about group chats. The risks in group chats are actually **higher**:

- Anyone in the group can talk to your AI
- High message volume makes it easier to hide malicious instructions among normal conversation
- Prompt Injection success rates in group chats are far higher than in DMs

Three must-set group chat security options:

- **Group allowlist**: Only allow the AI to operate in designated groups — don't let it wander
- **requireMention: true**: Must @mention the AI to get a response — don't let it be a busybody
- **groupAllowFrom**: Even in allowed groups, only respond to messages from specific people

### The Full Lesson from Xiao Wang's Story

Xiao Wang's disaster wasn't a single point of failure — two gates were unlocked simultaneously:

**First gate unlocked:** dmPolicy set to open → strangers could talk

**Second gate unlocked:** AI had file read/write permissions → once they could talk, they could take things

This leads to the next question —

## Gates Are Locked — Now Manage the Keys

Access control solves "who can talk." Permissions solve "what they can do once they talk."

OpenClaw has a Tool Policy system that lets you precisely control which tools the AI can use. Core groups:

- **group:runtime** — Execute commands (exec)
- **group:fs** — Read/write files
- **group:messaging** — Send messages
- **group:automation** — Scheduled tasks, Gateway configuration
- **group:ui** — Browser operations
- **group:sessions** — Session management

**The security principle is one sentence: give the AI the minimum permissions it needs — enable what's needed, nothing more.**

If you have an AI that only chats, it doesn't need access to the filesystem or command execution at all:

```json
{
  "tools": {
    "profile": "messaging",
    "deny": ["group:automation", "group:runtime", "group:fs"]
  }
}
```

This way, even if someone tricks the AI, the AI can't do anything — it simply doesn't have those tools. It's like chatting with a bank teller all afternoon — but they don't have the vault key, so you can't get the money.

## Three Lines of Defense, Quick Overview

Beyond access control and permissions, OpenClaw has three more lines of defense. One sentence each:

**Sandbox (the fence):** Let the AI work in an isolated container instead of directly on your computer. Recommendation: your own DMs connect directly to the host; group chats and other sources go through the sandbox.

```json
{
  "sandbox": { "mode": "non-main", "scope": "session", "workspaceAccess": "none" }
}
```

**Content Protection (anti-fraud):** The AI can be "tricked" by malicious instructions hidden in web pages (Prompt Injection). Frankly, there's no perfect solution yet — OpenClaw's team acknowledges this. But if the first two gates hold — no tools, no permissions, confined to a sandbox — even if it gets tricked, nothing explodes.

**Supply Chain (are your installed Skills trustworthy?):** Skills on ClawHub are like apps on your phone — most are safe, but stay alert. Only install Skills you trust, review the source code before installing, and consider version locking.

## Five Most Dangerous Configurations — How Many Have You Hit?

Time for a self-check. Go through these five items against your openclaw.json:

**❶ dmPolicy set to open** 🔴

Anyone can talk to your AI. If the AI also has file read/write permissions — your computer is open to the world.

→ Change to `pairing` or `allowlist`

**❷ Gateway exposed to public network without auth** 🔴

If your Gateway isn't bound to loopback and has no token set, anyone can connect and control your AI.

→ `"bind": "loopback"` + set a long random token

**❸ Group chats wide open + overly broad tool permissions** 🟠

Anyone in the group can trigger the AI, and the AI can execute commands — this combination is extremely dangerous.

→ `requireMention: true` + dedicated group chat Agent with minimal toolset

**❹ ~/.openclaw directory permissions too loose** 🟡

Your credentials, config, and session records are all in this directory. If permissions are world-readable, other users on the same machine can read them.

→ `chmod 700 ~/.openclaw && chmod 600 ~/.openclaw/openclaw.json`

**❺ Small model + big permissions** 🟠

Small models are more susceptible to prompt injection. Running Agents with full tool permissions on a small model is high risk.

→ Agents with tool permissions must use the latest and most capable models

**If you've hit two or more — you really should spend ten minutes addressing this.**

## One Command to Check Up Your Lobster

Back to that command from the beginning:

```
openclaw security audit
```

It checks your access control status, tool risk surface, network exposure, file permissions, model security — basically everything this article covers, scanned automatically.

```
openclaw security audit --deep   # Deep check, probes Gateway
openclaw security audit --fix    # Auto-fix what can be fixed
```

**Make it a habit: every time you change a config, connect a new channel, or install a new Skill, run a security audit.** No symptoms doesn't mean no problems.

## Just Remember Three Things

**Access control is paramount.** 80% of security issues are "people who shouldn't have been talking, got to talk." Set dmPolicy right, and it beats any fancy security configuration.

**Least privilege.** Don't give the AI all tools right away. Enable what's needed, assign different permissions for different roles.

**Security is a process.** There's no "set it and forget it" security config. Regular check-ups, keep models up to date, stay on top of Skill updates.

Now go run `openclaw security audit`. Screenshot your results and share them in the comments — let's see whose Lobster has the thickest armor 🦞

### 🛡️ One-Click Security Check-Up

Many people give up the moment they see a command line — perhaps that's the fate of security: the cost is just too high.

Since this article is about Lobster security, I want to make it easy for everyone to face security issues and complete hardening with ease. I've packaged the security experience from this article into an open-source OpenClaw Skill — it scans your config, generates a report, offers three hardening plans, and applies changes for you once you choose.

Copy the following message and send it to your Lobster:

> Install this security check-up skill for me: https://github.com/shanggqm/openclaw-security-hardening — then run a security check right away.

If you find it useful, head over to GitHub and give it a ⭐️ Star so more Lobster owners can see it.

---

If this article helped you avoid a "Xiao Wang–style disaster" — **give it a like and share it with friends who also keep Lobsters**. Security isn't something only one person needs to know — everyone needs to know.

Follow "Guo Meiqing Talks AI" — in the next article, we'll break down the best configurations for access control, permissions, and sandboxing step by step — from "knowing what to do" to "done in five minutes."

---

*This is part of the "OpenClaw Deep Dive" security series. Written based on OpenClaw official source code, MITRE ATLAS threat model documentation, and security audit systems.*
