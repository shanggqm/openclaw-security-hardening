# 🦞🔒 OpenClaw Security Hardening Skill

一句话让你的龙虾完成安全自检和加固。来自[「OpenClaw 深度连载」安全篇](https://mp.weixin.qq.com/)。

## 这是什么？

一个 prompt-only 的 OpenClaw Skill，安装后对你的龙虾说一句「安全检查」，Agent 就会自动：

1. **🔍 扫描** — 读取 `openclaw.json`，对 7 大安全维度逐项审计
2. **📊 报告** — 输出带评分的安全报告（0-100 分），每项标注 ✅ PASS / ⚠️ WARN / 🔴 FAIL
3. **💡 方案** — 根据扫描结果生成三档加固方案（基础 / 标准 / 严格）
4. **🔧 执行** — 你选定方案后，Agent 自动修改配置并验证

**零脚本、零依赖**，纯靠 Agent 的内置工具（read/edit/exec）完成所有操作。

## 检查项

| # | 检查项 | 风险等级 | 说明 |
|---|--------|---------|------|
| 1 | DM 策略 | 🔴 极高 | 谁能私聊你的 bot？`open` = 全世界 |
| 2 | Gateway 网络暴露 | 🔴 极高 | Gateway 绑定地址和认证配置 |
| 3 | 群聊安全 | 🟠 高 | requireMention、群白名单 |
| 4 | 工具权限 | 🟠 高 | Agent 的工具权限是否过宽 |
| 5 | 沙箱配置 | 🟡 中 | sandbox 模式和隔离级别 |
| 6 | 文件权限 | 🟡 中 | 配置文件的 chmod 权限 |
| 7 | 模型安全 | 🟠 高 | 有工具权限的 Agent 是否用了小模型 |

## 安装

### 方式一：npm 安装（推荐）

```bash
openclaw skills install npm:openclaw-security-hardening
```

### 方式二：一句话让龙虾自己装

直接对你的龙虾说：

> 帮我安装这个安全体检 skill：npm:openclaw-security-hardening ，装完直接帮我做一次安全体检。

### 方式三：手动安装

```bash
# 下载并解压到 skills 目录
cd ~/.openclaw/skills
mkdir -p security-hardening && cd security-hardening
npm pack openclaw-security-hardening && tar xzf openclaw-security-hardening-*.tgz --strip-components=1 package/
rm -f openclaw-security-hardening-*.tgz
```

安装后重启 Gateway：

```bash
openclaw gateway restart
```

验证安装：

```bash
openclaw skills list | grep security
```

## 使用方法

对你的龙虾说以下任意一句话：

- 「安全检查」
- 「security check」
- 「帮我做个安全审计」
- 「harden my setup」
- 「安全扫描」

Agent 会自动开始扫描并输出报告。

## 示例输出

### 扫描报告

```
🦞🔒 OpenClaw 安全扫描报告

扫描时间：2026-03-22 09:00
配置文件：~/.openclaw/openclaw.json

📊 总览

| 安全评分 | 65 / 100 |
|---------|----------|
| 🔴 FAIL | 1 项     |
| ⚠️ WARN | 3 项     |
| ✅ PASS | 3 项     |

📋 详细结果

| # | 检查项          | 风险等级 | 结果 | 说明                              |
|---|----------------|---------|------|-----------------------------------|
| 1 | DM 策略         | 🔴 极高 | ✅   | allowlist，已配置白名单              |
| 2 | Gateway 网络暴露 | 🔴 极高 | ⚠️   | bind=lan，但 auth 已配置 token      |
| 3 | 群聊安全         | 🟠 高  | ✅   | groupPolicy=allowlist，已配置白名单   |
| 4 | 工具权限         | 🟠 高  | ⚠️   | 3 个 agent 有 full profile         |
| 5 | 沙箱配置         | 🟡 中  | 🔴   | 未配置沙箱                          |
| 6 | 文件权限         | 🟡 中  | ⚠️   | 目录权限 755，建议收紧到 700          |
| 7 | 模型安全         | 🟠 高  | ✅   | 所有有工具权限的 agent 使用大模型      |
```

### 加固方案选择

```
请选择加固方案：

🟢 输入 1 → 基础加固（快速修复最危险的问题）
🟡 输入 2 → 标准加固（推荐！兼顾安全与易用）
🔴 输入 3 → 严格加固（最高安全，适合公网部署）

或输入 0 → 仅查看报告，不执行加固
```

## 三档加固方案

| 方案 | 耗时 | 适合场景 |
|------|------|---------|
| 🟢 基础加固 | ~5 分钟 | 修复最危险的问题（DM 策略、Gateway Auth、文件权限） |
| 🟡 标准加固 | ~10 分钟 | 基础 + 群聊保护 + 沙箱 + 工具权限按角色分配 |
| 🔴 严格加固 | ~15 分钟 | 标准 + 全面沙箱化 + 最小权限 + Gateway loopback |

## 技术细节

这是一个 **prompt-only skill**（纯 SKILL.md），不包含任何脚本。所有操作通过 Agent 的内置工具完成：

- `read` — 读取配置文件
- `exec` — 检查文件权限、备份配置、重启 Gateway
- `edit` — 精确修改配置项

Agent 会在修改前自动备份配置文件，加固后重新扫描确认。

## 致谢

本 Skill 的安全检查逻辑来自「OpenClaw 深度连载」安全篇的实战经验。感谢社区的安全反馈。

龙虾要安全，安全了才能快乐地夹人 🦞✨

## License

MIT
