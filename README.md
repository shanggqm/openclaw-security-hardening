> 🇬🇧 [English](README.en.md)

# OpenClaw Security Hardening

OpenClaw 安全工具集：配置审计 Skill、第三方 Skill 安装审查，以及配套的安全连载文章。

## Skills

### guomeiqing-security-audit — 配置安全审计

对 `openclaw.json` 进行 7 项安全维度审计，输出评分报告，并提供三档加固方案（基础 / 标准 / 严格）。选定方案后自动修改配置并验证。

审计项：

| # | 检查项 | 风险等级 | 说明 |
|---|--------|---------|------|
| 1 | DM 策略 | Critical | 私聊准入控制，`open` 意味着任何人可触达 |
| 2 | Gateway 网络暴露 | Critical | 绑定地址与认证配置 |
| 3 | 群聊安全 | High | requireMention、群白名单 |
| 4 | 工具权限 | High | Agent 工具权限是否过宽 |
| 5 | 沙箱配置 | Medium | sandbox 模式与隔离级别 |
| 6 | 文件权限 | Medium | 配置文件 chmod |
| 7 | 模型安全 | High | 有工具权限的 Agent 是否使用了能力不足的小模型 |

### guomeiqing-safe-install — 第三方 Skill 安全审查

安装社区 Skill 前自动执行 7 项安全审查（外发数据、凭证读取、危险命令、伪装行为、权限提升、隐蔽指令、脚本文件）。审查通过则安装，存在风险则阻断或交由用户确认。

两个 Skill 均为 **prompt-only**（纯 SKILL.md），无脚本依赖，通过 Agent 内置工具（read / edit / exec）完成全部操作。

## 安装

**npm 安装（推荐）：**

```bash
openclaw skills install npm:guomeiqing-security-audit
```

**手动安装：**

```bash
cd ~/.openclaw/skills
mkdir -p guomeiqing-security-audit && cd guomeiqing-security-audit
npm pack guomeiqing-security-audit && tar xzf guomeiqing-security-audit-*.tgz --strip-components=1 package/
rm -f guomeiqing-security-audit-*.tgz
```

安装后重启 Gateway：

```bash
openclaw gateway restart
```

## 使用

对 Agent 说「安全检查」「security check」「安全扫描」等即可触发审计流程。

示例输出：

```
OpenClaw 安全扫描报告

扫描时间：2026-03-22 09:00
配置文件：~/.openclaw/openclaw.json

总览
安全评分：65 / 100
FAIL：1 项 | WARN：3 项 | PASS：3 项

详细结果
1. DM 策略        [Critical]  PASS  — allowlist 已配置
2. Gateway 暴露   [Critical]  WARN  — bind=lan，auth token 已配置
3. 群聊安全        [High]     PASS  — groupPolicy=allowlist
4. 工具权限        [High]     WARN  — 3 个 agent 持有 full profile
5. 沙箱配置        [Medium]   FAIL  — 未配置沙箱
6. 文件权限        [Medium]   WARN  — 目录权限 755，建议 700
7. 模型安全        [High]     PASS  — 有工具权限的 agent 均使用大模型
```

审计完成后可选择加固方案：

| 方案 | 耗时 | 适用场景 |
|------|------|---------|
| 基础 | ~5 min | 修复最高风险项（DM 策略、Gateway Auth、文件权限） |
| 标准 | ~10 min | 基础 + 群聊保护 + 沙箱 + 按角色分配工具权限 |
| 严格 | ~15 min | 标准 + 全面沙箱化 + 最小权限 + Gateway loopback |

## 安全连载文章

「OpenClaw 深度连载 · 安全篇」全 5 篇文章及配图原稿归档于 [`articles/`](articles/) 目录。

| # | 标题 | 主题 |
|---|------|------|
| 1 | [你的龙虾正在裸奔吗？](articles/01-lobster-naked/) | 全局安全概览与 audit 实操 |
| 2 | [安全加固的三层盔甲](articles/02-three-layers-armor/) | 安全心智模型：门禁→权限→围栏 |
| 3 | [你的龙虾会被策反吗？——Prompt Injection 全解](articles/03-prompt-injection/) | 直接 / 间接注入、工具链攻击、供应链投毒 |
| 4 | [你装的 Skill 安全吗？](articles/04-skill-supply-chain/) | Skill 供应链安全、大小模型风险、隐藏指令审计 |
| 5 | [别让龙虾替你泄密——隐私围栏与数据安全](articles/05-privacy-fence/) | 记忆泄露、群聊边界、密钥防护、数据流安全 |

## Roadmap

- [x] guomeiqing-security-audit：一键安全审计 + 三档加固
- [x] guomeiqing-safe-install：第三方 Skill 安全审查
- [x] 安全连载全 5 篇完结
- [ ] 威胁模型文档
- [ ] 权限最小化助手 Skill
- [ ] 沙箱配置向导 Skill
- [ ] 安全配置模板库

## 贡献

详见 [CONTRIBUTING.md](CONTRIBUTING.md)。欢迎提 Issue 和 PR。

## 致谢

- [OpenClaw](https://github.com/openclaw/openclaw)

## License

[MIT](LICENSE)
