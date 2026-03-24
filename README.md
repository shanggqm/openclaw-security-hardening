# 🦞🔒 OpenClaw Security — 安全加固全家桶

让你的龙虾从裸奔到全副武装。

**Skill + 文章 + 教程 + 资源**，一个仓库搞定 OpenClaw 安全加固的一切。

---

## 📚 安全连载文章

来自「OpenClaw 深度连载 · 安全篇」公众号系列，从零建立安全认知。

| # | 标题 | 一句话简介 | 状态 |
|---|------|----------|------|
| 1 | [你的龙虾正在裸奔吗？](articles/01-lobster-naked/) | 全局安全概览：跑一次 audit，看看你的龙虾到底有多危险 | ✅ 已发布 |
| 2 | [安全加固的三层盔甲](articles/02-three-layers-armor/) | 安全心智模型：门禁→权限→围栏，三层防护框架 | ✅ 已发布 |
| 3 | 守好门：只让对的人跟你的龙虾说话 | 实战配置 allowlist、pairing、requireMention | 📝 计划中 |
| 4 | TBD | TBD | 📝 计划中 |
| 5 | TBD | TBD | 📝 计划中 |

---

## 🛠️ Skills

开箱即用的安全加固 Skill，装上就能让龙虾自己做安全检查。

| Skill | 说明 | 安装方式 |
|-------|------|---------|
| [security-audit](skills/security-audit/) | 一句话安全体检 + 三档加固方案 | `openclaw skills install npm:openclaw-security-hardening` |

> 💡 更多 Skill 陆续开发中（权限最小化助手、沙箱配置向导等）

### 快速安装

```bash
openclaw skills install npm:openclaw-security-hardening
```

安装后对你的龙虾说：

> 「安全检查」

Agent 会自动扫描配置、输出评分报告、给出三档加固方案。详见 [Skill 文档](skills/security-audit/)。

---

## 📖 教程 & 文档

| 文档 | 说明 | 状态 |
|------|------|------|
| [威胁模型速查](docs/threat-model.md) | OpenClaw 面临的安全威胁全景 | 🚧 编写中 |

> 更多教程和最佳实践文档持续更新中。

---

## 📦 资源

安全加固相关的配置模板、检查清单等资源。

> 🚧 Coming soon — 欢迎 PR 贡献你的安全配置模板！

---

## 🗺️ Roadmap

- [x] security-audit Skill：一键安全体检 + 三档加固
- [x] 安全连载文章（第 1-2 篇）
- [ ] 安全连载文章（第 3-5 篇）
- [ ] 威胁模型文档
- [ ] 权限最小化助手 Skill
- [ ] 沙箱配置向导 Skill
- [ ] 安全配置模板库
- [ ] 安全加固 checklist（可打印版）

---

## 🤝 贡献

欢迎任何形式的贡献：

- **发现安全问题？** 提 Issue
- **有加固经验？** 写文章 PR 到 `articles/`
- **想做新 Skill？** 放到 `skills/` 下，附 SKILL.md + README

---

## 📊 安全检查项总览

| # | 检查项 | 风险等级 | 说明 |
|---|--------|---------|------|
| 1 | DM 策略 | 🔴 极高 | 谁能私聊你的 bot？`open` = 全世界 |
| 2 | Gateway 网络暴露 | 🔴 极高 | Gateway 绑定地址和认证配置 |
| 3 | 群聊安全 | 🟠 高 | requireMention、群白名单 |
| 4 | 工具权限 | 🟠 高 | Agent 的工具权限是否过宽 |
| 5 | 沙箱配置 | 🟡 中 | sandbox 模式和隔离级别 |
| 6 | 文件权限 | 🟡 中 | 配置文件的 chmod 权限 |
| 7 | 模型安全 | 🟠 高 | 有工具权限的 Agent 是否用了小模型 |

---

## 致谢

- [OpenClaw](https://github.com/openclaw/openclaw) — 让 AI Agent 触手可及
- 「OpenClaw 深度连载」安全篇的读者们
- 社区的安全反馈和贡献

龙虾要安全，安全了才能快乐地夹人 🦞✨

## License

[MIT](LICENSE)
