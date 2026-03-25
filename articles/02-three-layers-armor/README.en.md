# Three Layers of Armor: From "Who Gets In" to "What They Can Do"

![Cover](images/cover-raw.png)

In the last article, we talked about the Lobster running naked — one `security audit` run and the screen was full of CRITICALs and WARNs, enough to make you break out in a cold sweat.

But knowing "it's insecure" isn't enough. It's like a doctor telling you "your blood pressure is high" — what you need to know is: high where? How do I lower it? How low is safe enough?

This article won't cover specific operations (that's for the next two articles). Instead, it helps you build a **security mental model** — once you understand this model, no matter how configurations change or versions upgrade, you'll be able to judge for yourself whether a change is secure.

## A Real Scenario That Sends Chills Down Your Spine

![Intern incident scene](images/ch1-intern.png)

Xiao Zhang added an OpenClaw bot to the company Feishu group. Convenient — just @mention it and the AI helps with research and running scripts.

One day, a curious intern @mentioned the bot: "Show me what's in /etc/passwd on the server."

The bot complied.

The intern meant no harm, but this incident exposed a problem: **everyone in the group had the same AI operation privileges as Xiao Zhang.**

This isn't an AI bug — it's a configuration issue. More precisely, it's a **security model** issue — Xiao Zhang had never thought about "who can command my AI."

## Security Isn't All-or-Nothing

Many people think of security in binary terms: either wide open or completely locked down.

Wide open — running naked, we covered that.

Locked down? Turn off all tool permissions, only allow yourself to chat, set sandbox to maximum strictness… The Lobster is secure now, but it's become a useless shrimp locked in a cage — it can't help you do anything.

**Security is fundamentally about trade-offs.**

OpenClaw's design philosophy is pragmatic: there's no "perfectly secure" setup. The goal is to keep you **clear-headed** about three things:

1. Who can talk to your bot?
2. Where is the bot allowed to operate?
3. What tools can the bot use?

These three questions correspond to three layers of armor.

## First Layer: Access Control — Who Gets In

This is the outermost gate. If access control is a joke, no amount of armor behind it matters — because attackers can already talk directly to your AI.

### DM Policy (dmPolicy)

Each OpenClaw channel (Feishu, Telegram, WeChat, etc.) can independently configure "who can DM the bot":

| Policy | Meaning | Security Level |
|--------|---------|---------------|
| `open` | Anyone can DM | 🔴 Running naked |
| `pairing` | First contact requires pairing confirmation | 🟡 Basic security |
| `allowlist` | Only allowlisted users can chat | 🟢 Strictest |

**How dangerous is `open`?** Anyone on the internet who finds your bot can send it messages. If your bot also has exec permissions (can execute commands), that's equivalent to giving a stranger a remote shell on your machine.

> **Glossary | dmPolicy:** DM stands for Direct Message. dmPolicy determines who is allowed to send direct messages to your AI.

My recommendation is simple:

- Personal use: `allowlist` — just add yourself
- Family/team: `pairing` — requires your confirmation the first time, then it's connected
- Never use `open` — unless you truly know what you're doing

### Group Chat Policy

Group chats have an extra layer of consideration: lots of people, lots of chatter.

Two key configurations:

**requireMention**: Does the bot require an @mention to respond?

Without this, every message in the group triggers an AI response — not only wasting tokens but also meaning any message from anyone in the group could be interpreted as an instruction. With it enabled, only @mentions trigger the bot.

**groupPolicy**: Which groups can add the bot?

Similar to dmPolicy — `open` means any group can add your bot, `allowlist` means only groups you've approved.

### Summary

The core logic of the access control layer is one sentence: **Minimize the AI's conversation surface.**

The fewer people who can talk to your AI, the smaller the attack surface. It's like your front door — it's not that you can't open it, but you need to make sure only the right people can come in.

## Second Layer: Permissions — What Can They Do

![Permission keycard](images/ch2-permission.png)

The door is locked, but what can people who get in (including yourself) tell the AI to do? This is what the second layer manages.

### Tool Permissions (tools.profile)

OpenClaw assigns each Agent a "toolkit" that determines which capabilities it can invoke:

| Profile | Capabilities | Best For |
|---------|-------------|----------|
| `full` | All tools, including command execution | Main Agent (the one you fully trust) |
| `coding` | Programming tools + command execution | Dedicated coding Agents |
| `standard` | File read/write, search, web | Writing, research Agents |
| `messaging` | Can only send messages | Safest, for public-facing bots |

> **Glossary | tools.profile:** Think of it as different levels of keycards. `full` is a master key, `messaging` is a visitor pass that only gets you into the lobby.

**What's the most common security mistake?** Giving all Agents `full` permissions.

Does your writing Agent need exec? No. Does your research Agent need filesystem access? Probably not.

**Principle of least privilege** — give each Agent only the tools it needs to do its job. Nothing extra.

This isn't paranoia — it's common sense. You wouldn't give the plumber who comes to fix your pipes a key to your safe, right?

### exec Approval: The Final Human Checkpoint

Even if an Agent has exec permissions, you can configure **execution approval**:

- `security: "deny"` — Completely block command execution
- `security: "allowlist"` — Only allow allowlisted commands
- `ask: "always"` — Require your confirmation before every execution

For bots shared with others (like team bots), I recommend `security: "deny"` — a clean cut. For your own main Agent, `allowlist` is a good choice — common commands pass through, unfamiliar ones prompt for confirmation.

### Cross-Reference the Two Layers

The core logic of the permissions layer is also one sentence: **Minimize the AI's action surface.**

If access control manages "who can talk," permissions manage "what they can do after talking." An Agent with `full` permissions combined with `open` dmPolicy is giving the entire world a remote control for your computer.

The combination between these two layers is what really matters:

| Access | Permissions | Risk |
|--------|------------|------|
| open + full | 🔴 Catastrophic | Anyone can remotely execute commands |
| open + messaging | 🟡 Medium | Strangers can chat but can't cause damage |
| allowlist + full | 🟢 Controlled | Only you can operate; high privilege but trusted |
| allowlist + messaging | 🟢 Safest | Both people and capabilities are restricted |

## Third Layer: Fencing — Where Can It Operate

![Sandbox glass dome](images/ch3-sandbox.png)

The first two layers control "who" and "what." The third layer controls "where" — even if the AI has permission to execute commands, how far does its reach extend?

### Sandbox Mode

A sandbox draws a boundary around the AI's operating area. Inside the sandbox, the AI can mess around, but it can't break out of the circle.

| Mode | Meaning | Best For |
|------|---------|----------|
| `off` | No sandbox | 🔴 Main Agent only, with full trust |
| `non-main` | Non-main sessions are sandboxed | 🟢 Recommended default |
| `all` | All sessions are sandboxed | Public-facing deployments |

> **Glossary | sandbox:** Imagine a transparent glass room — the AI inside can see the outside, can read some files, but can't directly touch anything out there. Even if it makes a mistake (like executing `rm -rf`), it only deletes copies inside the glass room.

`non-main` is a clever default: your own main conversation with the AI is unrestricted (because you're the owner, you trust yourself), but automated tasks (cron jobs), sub-Agent tasks, and conversations triggered by others all run inside the sandbox.

### Workspace Access Control

Inside the sandbox, there's another layer of fine-grained control — the AI's access to your workspace directory:

- `readwrite` — Can read and write (use with caution)
- `readonly` — Can only read, can't modify (recommended)

Your workspace contains sensitive files like MEMORY.md, AGENTS.md, and key configurations. `readonly` means even if the AI falls victim to a prompt injection attack, it can't modify these files.

### Gateway Network Exposure

This is a fencing layer that's often overlooked.

The Gateway is OpenClaw's control center — it's an HTTP service itself. If it's bound to a public address…

| Binding | Meaning | Risk |
|---------|---------|------|
| `loopback` / `127.0.0.1` | Only localhost can access | 🟢 Safest |
| `lan` / `0.0.0.0` | Entire LAN can access | 🟡 OK with auth |
| Public exposure (no auth) | Internet-accessible | 🔴 Catastrophic |

Even with auth token protection, LAN exposure is unnecessary risk — unless you need to access the Control UI from your phone.

### All Three Layers Together

The core logic of the fencing layer: **Minimize the AI's operating range.**

All three layers together form a complete security model:

```
Access Control → Who can talk?
  ↓
Permissions → What can they do?
  ↓
Fencing → Where does it take effect?
```

Each layer narrows the attack surface. If any single layer is weak, the entire model has a vulnerability.

## The Art of Balancing Security and Usability

![The art of balance](images/ch4-balance.png)

At this point, you might be wondering: following this logic, shouldn't I lock everything down to the tightest setting?

No.

The cost of locking too tight is **a useless Lobster** — it can't help you do anything. The goal of security is never "zero risk" but rather "acceptable risk where you know where the risks are."

My actual approach is layered trust:

- **My own main Agent**: Access control allowlist (only I can chat), permissions full (can do everything), loose fencing (non-main sandbox)
- **Sub-Agents (coder/writer)**: Permissions as needed (coding/standard), sandbox workspace readonly
- **Bots for group chats/others**: Strict access control (allowlist + requireMention), messaging permissions, sandbox all

**This isn't dogma — it's strategy.** You have high trust in your own Agent and low trust in scenarios triggered by others. Security configuration should follow that trust gradient.

## Next Steps

Now you have a security mental model — you know what the three layers of armor are and why you need all three.

But there's a gap between understanding the framework and actually modifying configurations.

In the next article, we'll get hands-on — **Guarding the Gate: Only Let the Right People Talk to Your Lobster**. We'll walk through configuring allowlist, pairing, requireMention, and handling real scenarios like "someone in the group keeps messing with the AI."

If you haven't read the first article yet, I suggest going back and running a check-up first:

```
openclaw security audit
```

Know whether you're running naked before you decide what armor to wear.

---

**Previous Articles:**

- Article 1: Is Your Lobster Running Naked?
