# 贡献指南

感谢你对 OpenClaw Security 的关注！欢迎任何形式的贡献。

## 如何贡献

### 🐛 报告 Bug

1. 先搜索 [已有 Issue](https://github.com/shanggqm/openclaw-security-hardening/issues)，避免重复
2. 创建新 Issue，描述清楚：
   - 你做了什么
   - 你期望发生什么
   - 实际发生了什么
   - 你的环境（OpenClaw 版本、OS 等）

### 💡 功能建议

1. 创建 Issue，标签选 `feature`
2. 说清楚：
   - 你想解决什么问题
   - 你设想的方案是什么
   - 有没有替代方案

### 📝 贡献文章

安全连载文章存放在 `articles/` 目录下：

1. Fork 本仓库
2. 创建分支：`git checkout -b docs/add-article-xxx`
3. 在 `articles/` 下创建新目录，包含 `README.md`（文章正文）和 `images/`（配图）
4. 提交 PR

### 🔧 贡献 Skill

新的安全相关 Skill 放在 `skills/` 目录下：

1. Fork 本仓库
2. 创建分支：`git checkout -b feat/new-skill-name`
3. 在 `skills/` 下创建新目录，至少包含：
   - `SKILL.md` — Skill 定义
   - `README.md` — 使用说明
   - `package.json` — 如果需要通过 npm 发布
4. 提交 PR

## 开发规范

### Commit Message

使用 [Conventional Commits](https://www.conventionalcommits.org/)：

```
feat(skill): add prompt injection detector
fix(audit): correct score calculation
docs(article): add article #3
```

### 分支命名

```
feat/<描述>      # 新功能
fix/<描述>       # 修复
docs/<描述>      # 文档
refactor/<描述>  # 重构
```

### PR 要求

- 标题使用 Conventional Commits 格式
- 描述包含：What / Why / How
- 一个 PR 做一件事，不要混合无关改动

## 行为准则

- 友善、尊重、包容
- 就事论事，不对人
- 欢迎新手，耐心解答

## License

贡献的代码将使用本项目的 [MIT License](LICENSE)。
