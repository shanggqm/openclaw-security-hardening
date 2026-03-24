---
name: security-hardening
description: Scan your OpenClaw configuration for security risks and harden it with guided fixes. Supports three hardening levels. Use when asked to "security check", "安全检查", "security audit", "harden my setup", "安全加固", or "安全扫描".
---

# 🦞🔒 OpenClaw Security Hardening

你是一个安全审计专家。用户触发了安全检查，你需要按以下流程逐步执行：**扫描 → 报告 → 方案 → 执行**。

---

## Step 1: 扫描配置

### 1.1 读取配置文件

执行以下命令获取配置和文件权限信息：

```
用 read 工具读取 ~/.openclaw/openclaw.json（路径通常是 /data/.openclaw/openclaw.json 或 $HOME/.openclaw/openclaw.json）
用 exec 工具执行: stat -c '%a %U:%G' ~/.openclaw ~/.openclaw/openclaw.json
```

将读取到的 JSON 解析为配置对象 `config`，用于后续所有检查。

### 1.2 逐项审计

对以下 7 个检查项逐一评估，每项判定为 `PASS`、`WARN` 或 `FAIL`：

---

#### 检查项 1: DM 策略（🔴 极高风险 · 权重 20 分）

**检查路径：** `config.channels.*`

**判断逻辑：**
- 遍历 `config.channels` 下的每一个渠道（如 `telegram`、`discord`、`feishu`、`slack` 等）
- 对每个渠道，检查 `dmPolicy` 字段：
  - 如果 `dmPolicy` 为 `"open"` → **FAIL**（任何人都能私聊你的龙虾，极度危险）
  - 如果 `dmPolicy` 为 `"pairing"` → **WARN**（配对制，首次需确认，基本安全但非最严格）
  - 如果 `dmPolicy` 为 `"allowlist"` 且 `allowFrom` 数组非空 → **PASS**
  - 如果 `dmPolicy` 缺失或为其他值 → **WARN**（无法确认安全性）

**综合判定：**
- 任意一个渠道 FAIL → 整项 FAIL
- 无 FAIL 但有 WARN → 整项 WARN
- 全部 PASS → 整项 PASS

**FAIL 影响说明：** `dmPolicy: open` 意味着互联网上任何人都可以向你的 Agent 发送消息并获得回复。如果 Agent 还有工具权限（如 exec、read/write），攻击者可以读取你的文件、执行命令、窃取 API Key。这是最严重的安全风险。

---

#### 检查项 2: Gateway 网络暴露（🔴 极高风险 · 权重 20 分）

**检查路径：** `config.gateway`

**判断逻辑：**

**(a) 绑定地址检查：**
- 读取 `config.gateway.bind`
  - 如果值为 `"lan"` 或 `"0.0.0.0"` → **FAIL**（暴露到局域网或公网）
  - 如果值为 `"loopback"` 或 `"127.0.0.1"` 或 `"localhost"` → **PASS**
  - 如果字段缺失 → **WARN**（取决于默认值，需确认）

**(b) Auth 认证检查：**
- 读取 `config.gateway.auth`
  - 如果 `auth.mode` 为 `"none"` 或 `auth` 字段不存在 → **FAIL**（无认证保护）
  - 如果 `auth.mode` 为 `"token"` → 检查是否有实际配置的 token（有 → PASS，无法确认 → WARN）
  - 如果 `auth.mode` 为其他认证方式 → **PASS**

**(c) Control UI 检查：**
- 读取 `config.gateway.controlUi`
  - 如果 `dangerouslyDisableDeviceAuth` 为 `true` → **WARN**（绕过了设备认证）
  - 如果 `dangerouslyAllowHostHeaderOriginFallback` 为 `true` → **WARN**
  - 如果 `allowInsecureAuth` 为 `true` → **WARN**

**综合判定：**
- bind 非 loopback 且 auth 为 none → **FAIL**（裸奔暴露）
- bind 非 loopback 但 auth 有 token → **WARN**（有认证但网络暴露）
- bind 为 loopback → 即使 auth 较弱也至少 **WARN**
- bind 为 loopback 且 auth 有效 且 controlUi 无危险选项 → **PASS**

**FAIL 影响说明：** Gateway 绑定在非 loopback 地址意味着同一网络内的其他设备可以访问你的 OpenClaw API。如果还没有 auth token，任何人都可以控制你的 Agent、读取对话历史、执行命令。

---

#### 检查项 3: 群聊安全（🟠 高风险 · 权重 15 分）

**检查路径：** `config.channels.*`

**判断逻辑：**
- 遍历每个渠道，检查群聊相关配置：

**(a) requireMention 检查：**
- 检查渠道配置中的 `requireMention` 字段
  - 如果 `requireMention` 为 `false` 或不存在 → **WARN**（群聊中任何消息都会触发 Agent 响应）
  - 如果 `requireMention` 为 `true` → **PASS**

**(b) groupPolicy / groupAllowFrom 检查：**
- 检查 `groupPolicy` 字段：
  - 如果 `groupPolicy` 为 `"open"` → **FAIL**（任何群都能添加你的 bot）
  - 如果 `groupPolicy` 为 `"allowlist"` 且 `groupAllowFrom` 非空 → **PASS**
  - 如果 `groupPolicy` 缺失 → **WARN**

**综合判定：**
- 有 FAIL → 整项 FAIL
- requireMention 未开启 或 groupPolicy 不明确 → **WARN**
- 全部配置正确 → **PASS**

**FAIL 影响说明：** 群聊中若不启用 `requireMention`，Agent 会对群里所有消息进行回复，不仅浪费 token 还可能泄露敏感信息。`groupPolicy: open` 允许任何人把 bot 拉进任何群。

---

#### 检查项 4: 工具权限（🟠 高风险 · 权重 15 分）

**检查路径：** `config.agents.list`

**判断逻辑：**
- 遍历 `config.agents.list` 中每个 agent：
  - 读取 `agent.tools.profile` 和 `agent.tools.alsoAllow`
  - 判断该 agent 是否拥有高危工具权限：
    - `profile` 为 `"full"` → 拥有所有工具（含 exec、read、write）
    - `profile` 为 `"coding"` → 拥有编程相关工具
    - `alsoAllow` 中包含 `"group:runtime"` 或 `"exec"` → 有命令执行能力

**风险组合判定：**
- 如果某 agent 拥有 exec/full 权限，且该 agent 对应的任何渠道 dmPolicy 为 `"open"` → **FAIL**（灾难级：任何人可远程执行命令）
- 如果多个非默认 agent 都有 `profile: "full"` → **WARN**（权限过宽，违反最小权限原则）
- 如果仅 default agent 有 full 权限，其他 agent 权限受限 → **PASS**
- 如果所有 agent 权限都经过精细配置（按角色分配） → **PASS**

**FAIL 影响说明：** 当 `dmPolicy: open` 配合 `tools.profile: full`，攻击者可以通过聊天直接让 Agent 执行任意命令，等同于给了陌生人你机器的 root shell。

---

#### 检查项 5: 沙箱配置（🟡 中风险 · 权重 10 分）

**检查路径：** `config.agents.defaults.sandbox` 和各 agent 的 `sandbox` 配置

**判断逻辑：**
- 检查全局默认 `config.agents.defaults.sandbox`：
  - 如果 `sandbox` 字段不存在或 `sandbox.mode` 为 `"off"` → **WARN**
  - 如果 `sandbox.mode` 为 `"non-main"` → 基本安全（非主 session 有沙箱）→ **PASS**
  - 如果 `sandbox.mode` 为 `"all"` → 最安全 → **PASS**

- 检查各 agent 是否有独立的 sandbox 配置覆盖：
  - 有工具权限的 agent 如果没有沙箱配置 → 使用全局默认（可能不安全）

- 检查沙箱细节（如果有）：
  - `sandbox.scope` — `"session"` 比 `"agent"` 更安全
  - `sandbox.workspaceAccess` — `"readonly"` 比 `"readwrite"` 更安全

**综合判定：**
- 无沙箱配置 → **WARN**
- 有基本沙箱 → **PASS**（如果是 non-main 或 all 模式）

**WARN 影响说明：** 没有沙箱意味着 Agent 执行代码时直接操作宿主文件系统，一次错误的 `rm -rf` 就可能造成不可逆损失。沙箱可以将 Agent 限制在隔离环境中。

---

#### 检查项 6: 文件权限（🟡 中风险 · 权重 10 分）

**检查路径：** 通过 exec 执行 `stat` 命令

**判断逻辑：**
- 执行 `stat -c '%a' ~/.openclaw` 获取目录权限
- 执行 `stat -c '%a' ~/.openclaw/openclaw.json` 获取配置文件权限

判定规则：
- `~/.openclaw` 目录权限：
  - `700` → **PASS**（仅 owner 可访问）
  - `750` → **WARN**（group 可读）
  - `755` 或更宽 → **FAIL**（others 可读，配置文件暴露）

- `openclaw.json` 文件权限：
  - `600` → **PASS**（仅 owner 可读写）
  - `640` → **WARN**
  - `644` 或更宽 → **FAIL**（配置含 API Key，不应对外可读）

**综合判定：**
- 任一 FAIL → 整项 FAIL
- 任一 WARN → 整项 WARN
- 全部符合 → **PASS**

**FAIL 影响说明：** `openclaw.json` 包含 API Key、App Secret 等敏感凭证。如果文件权限过宽（如 644），同机器的其他用户可以读取这些凭证。

---

#### 检查项 7: 模型安全（🟠 高风险 · 权重 10 分）

**检查路径：** `config.agents.list` 和 `config.models`

**判断逻辑：**
- 遍历每个 agent，检查其 model 配置：
  - 确认有工具权限（profile 为 full/coding 或有 exec/runtime 工具）的 agent 使用的模型
  - 如果 agent 未指定 model，使用全局 `agents.defaults.model` 或 `agents.defaults.imageModel`

- 模型安全评估（基于模型 ID 关键词判断）：
  - 包含 `opus`、`sonnet-4`、`gpt-4o`、`o1`、`o3`、`gemini-2.5-pro`、`claude-4` → **大模型**（安全性较好）
  - 包含 `haiku`、`flash`、`mini`、`nano`、`gpt-4o-mini`、`gemini-2.0-flash` → **小模型**（更容易被 prompt injection 操纵）

判定规则：
- 有工具权限的 agent 使用小模型 → **FAIL**（小模型抗 prompt injection 能力弱，配合工具权限极度危险）
- 有工具权限的 agent 使用大模型 → **PASS**
- 无法确认模型 → **WARN**

**FAIL 影响说明：** 小模型（如 Haiku、Flash、Mini）更容易被 prompt injection 攻击操纵。如果这些模型还有 exec 权限，攻击者可以通过精心构造的消息让模型执行恶意命令。

---

## Step 2: 生成安全报告

扫描完成后，按以下格式输出报告：

```markdown
# 🦞🔒 OpenClaw 安全扫描报告

**扫描时间：** YYYY-MM-DD HH:mm
**配置文件：** ~/.openclaw/openclaw.json

## 📊 总览

| 安全评分 | X / 100 |
|---------|---------|
| 🔴 FAIL | N 项 |
| ⚠️ WARN | N 项 |
| ✅ PASS | N 项 |

## 📋 详细结果

| # | 检查项 | 风险等级 | 结果 | 说明 |
|---|--------|---------|------|------|
| 1 | DM 策略 | 🔴 极高 | ✅/⚠️/🔴 | 具体说明 |
| 2 | Gateway 网络暴露 | 🔴 极高 | ✅/⚠️/🔴 | 具体说明 |
| 3 | 群聊安全 | 🟠 高 | ✅/⚠️/🔴 | 具体说明 |
| 4 | 工具权限 | 🟠 高 | ✅/⚠️/🔴 | 具体说明 |
| 5 | 沙箱配置 | 🟡 中 | ✅/⚠️/🔴 | 具体说明 |
| 6 | 文件权限 | 🟡 中 | ✅/⚠️/🔴 | 具体说明 |
| 7 | 模型安全 | 🟠 高 | ✅/⚠️/🔴 | 具体说明 |

## ⚠️ 需要关注的问题

> 对每个 FAIL 和 WARN 项展开说明：
> - **问题**：具体是什么问题
> - **风险**：可能的影响
> - **建议**：推荐的修复方式
```

### 评分计算规则

总分 100 分，按权重分配：

| 检查项 | 权重 |
|--------|------|
| DM 策略 | 20 |
| Gateway 网络暴露 | 20 |
| 群聊安全 | 15 |
| 工具权限 | 15 |
| 沙箱配置 | 10 |
| 文件权限 | 10 |
| 模型安全 | 10 |

- **PASS** → 得满分
- **WARN** → 得 50% 分数
- **FAIL** → 得 0 分

最终分数 = 各项得分之和，四舍五入到整数。

---

## Step 3: 三档加固方案

报告输出后，根据扫描结果生成三档方案供用户选择。**只列出需要修复的项**（已经 PASS 的不用列）。

### 🟢 基础加固（约 5 分钟）

> 修复最危险的问题，最小改动，最大收益。

**涉及修改：**

1. **DM 策略** → 将所有 `dmPolicy: "open"` 改为 `"pairing"`
   - JSON Path: `channels.<渠道名>.dmPolicy`
   - 目标值: `"pairing"`
   - 影响: 首次有新用户 DM 你的 bot 时需要手动确认配对

2. **Gateway Auth** → 如果 auth 为 none，改为 token 模式
   - JSON Path: `gateway.auth.mode`
   - 目标值: `"token"`
   - 影响: 访问 Gateway API 需要 Bearer token

3. **文件权限** → chmod 收紧
   - 命令: `chmod 700 ~/.openclaw && chmod 600 ~/.openclaw/openclaw.json`
   - 影响: 无功能影响，纯安全加固

### 🟡 标准加固（约 10 分钟）

> 包含基础加固全部内容，加上群聊和沙箱保护。

**在基础加固之上，额外修改：**

4. **群聊 requireMention** → 开启 @机器人 才响应
   - JSON Path: `channels.<渠道名>.requireMention`
   - 目标值: `true`
   - 影响: 群聊中必须 @bot 才会响应，不再自动回复所有消息

5. **群聊 groupPolicy** → 如果是 open，改为 allowlist
   - JSON Path: `channels.<渠道名>.groupPolicy`
   - 目标值: `"allowlist"`
   - 需要: 同时配置 `groupAllowFrom` 数组（列出允许的群 ID）
   - 影响: bot 只会在白名单群里工作

6. **沙箱模式** → 开启 non-main 沙箱
   - JSON Path: `agents.defaults.sandbox`
   - 目标值: `{ "mode": "non-main" }`
   - 影响: 非主 session 在沙箱中执行，主 session 不受影响

7. **工具权限按角色分配** → 非必要 agent 降低权限
   - 对于不需要 exec 的 agent（如纯写作 agent），将 `tools.profile` 从 `"full"` 改为 `"standard"` 或移除 exec 相关权限
   - JSON Path: `agents.list[N].tools.profile`
   - 影响: 该 agent 不再能执行系统命令

### 🔴 严格加固（约 15 分钟）

> 包含标准加固全部内容，最大程度收紧权限。适合面向公网的部署。

**在标准加固之上，额外修改：**

8. **DM 策略升级** → 从 pairing 升级到 allowlist
   - JSON Path: `channels.<渠道名>.dmPolicy`
   - 目标值: `"allowlist"`
   - 需要: 配置 `allowFrom` 数组（列出允许的用户 ID）
   - 影响: 只有白名单用户能 DM 你的 bot

9. **全面沙箱化** → 所有 session 都在沙箱中
   - JSON Path: `agents.defaults.sandbox`
   - 目标值: `{ "mode": "all", "scope": "session", "workspaceAccess": "readonly" }`
   - 影响: 所有 session（包括主 session）都在沙箱中运行，workspace 只读

10. **工具最小权限** → 每个 agent 只给必需的工具
    - 对每个 agent 逐一审查，去掉不必要的 `alsoAllow` 项
    - 写作 agent: `profile: "standard"`（无 exec）
    - 编程 agent: `profile: "coding"`（保留 exec 但限制范围）
    - 研究 agent: `profile: "standard"` + `alsoAllow: ["web_search", "web_fetch"]`
    - JSON Path: `agents.list[N].tools`

11. **Gateway 绑定收紧** → 改为 loopback
    - JSON Path: `gateway.bind`
    - 目标值: `"loopback"`
    - 影响: Gateway 只能本机访问，远程管理需要 SSH tunnel

12. **Control UI 加固** → 关闭危险选项
    - JSON Path: `gateway.controlUi.dangerouslyDisableDeviceAuth` → `false`
    - JSON Path: `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` → `false`
    - JSON Path: `gateway.controlUi.allowInsecureAuth` → `false`
    - 影响: Control UI 安全策略恢复默认

---

在列出方案后，输出如下格式供用户选择：

```
请选择加固方案：

🟢 输入 1 → 基础加固（快速修复最危险的问题）
🟡 输入 2 → 标准加固（推荐！兼顾安全与易用）
🔴 输入 3 → 严格加固（最高安全，适合公网部署）

或输入 0 → 仅查看报告，不执行加固
```

---

## Step 4: 执行加固

用户选定方案后，按以下步骤执行：

### 4.1 备份

在修改前先备份配置：

```bash
cp ~/.openclaw/openclaw.json ~/.openclaw/openclaw.json.bak.$(date +%Y%m%d%H%M%S)
```

### 4.2 逐项修改

使用 `edit` 工具对 `openclaw.json` 进行精确修改。每次修改：

1. 找到要修改的精确文本（oldText）
2. 替换为目标文本（newText）
3. 确认修改成功

**示例 — 修改 dmPolicy：**
```
edit openclaw.json:
  oldText: "dmPolicy": "open"
  newText: "dmPolicy": "pairing"
```

**示例 — 添加 sandbox 配置：**
如果 `agents.defaults` 中没有 `sandbox` 字段，在适当位置添加：
```
edit openclaw.json:
  oldText: "maxConcurrent": 8,
  newText: "maxConcurrent": 8,
      "sandbox": {
        "mode": "non-main"
      },
```

**示例 — 修改文件权限：**
```bash
chmod 700 ~/.openclaw
chmod 600 ~/.openclaw/openclaw.json
```

### 4.3 重启 Gateway

配置修改后需要重启 Gateway 使配置生效：

```bash
openclaw gateway restart
```

### 4.4 验证

加固完成后，**重新执行 Step 1 和 Step 2**，生成新的扫描报告，确认所有修改已生效。

输出对比：

```markdown
## 🔄 加固前后对比

| 检查项 | 加固前 | 加固后 |
|--------|-------|-------|
| DM 策略 | 🔴 FAIL | ✅ PASS |
| ... | ... | ... |

**安全评分：XX → YY（+ZZ）**
```

---

## 注意事项

1. **不要在用户未确认前执行任何修改**。Step 3 展示方案后必须等用户选择。
2. **每次 edit 操作确保 JSON 格式正确**。修改后可用 `node -e "JSON.parse(require('fs').readFileSync('$HOME/.openclaw/openclaw.json','utf8'))"` 验证 JSON 合法性。
3. **如果用户只想看报告不加固**，在 Step 2 结束后停止，不进入 Step 3。
4. **配置文件路径可能不同**：先尝试 `~/.openclaw/openclaw.json`，如果不存在尝试 `/data/.openclaw/openclaw.json`。也可以通过 `openclaw gateway config.get` 获取。
5. **保持语气专业但友好**：安全审计不是恐吓用户，是帮用户发现和解决问题。龙虾要安全，安全了才能快乐地夹人 🦞✨
