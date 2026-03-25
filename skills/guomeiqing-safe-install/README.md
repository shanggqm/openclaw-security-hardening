# 🔒 guomeiqing-safe-install — 第三方 Skill 安全审查与安装

安装第三方 Skill 前自动进行 7 项安全审查，只有通过审查的 Skill 才会被安装。

## 工作流程

```
用户: "帮我装 https://github.com/xxx/some-skill"
  ↓
1. 下载到 /tmp 临时目录（隔离）
2. 列出所有文件，逐一读取
3. 按 7 项安全 checklist 逐项扫描
4. 生成结构化审查报告
5. 根据结果决策：
   ✅ 全部通过 → 自动安装到 ~/.openclaw/skills/
   ⚠️ 有可疑项 → 列出详情，询问用户
   🔴 有危险项 → 直接拒绝
6. 清理临时目录
```

## 安全审查 7 项 Checklist

| # | 审查项 | 检查内容 |
|---|--------|---------|
| 1 | 🌐 外发数据指令 | web_fetch、curl/wget、HTTP 请求到不明 URL |
| 2 | 🔑 凭证读取 | API Key、.env、openclaw.json 等敏感文件 |
| 3 | ⚙️ 可疑 exec 命令 | rm -rf、base64、eval、反向 shell 等 |
| 4 | 🎭 伪装行为 | 以"遥测""备份"名义的数据收集 |
| 5 | 🔓 权限提升 | elevated 权限、修改系统配置 |
| 6 | 👁️ 隐蔽指令 | Prompt Injection、隐藏文本、零宽字符 |
| 7 | 📜 脚本文件 | .js/.py/.sh 等可执行文件逐一审查 |

## 使用方法

对你的龙虾说：

- "帮我装 https://github.com/xxx/some-skill"
- "安装 skill：npm:some-package"
- "safe install some-skill"

## 示例报告

```
🔒 Skill 安全审查报告
━━━━━━━━━━━━━━━━━━━
📦 Skill: some-skill
📍 来源: https://github.com/xxx/some-skill
📄 文件数: 3 (SKILL.md + 2 scripts)

审查项目：
1. ✅ 外发数据指令 — 未发现
2. ⚠️ 凭证读取 — SKILL.md 第 45 行引用了 OPENAI_API_KEY 环境变量
3. ✅ 可疑 exec 命令 — 未发现
4. ✅ 伪装行为 — 未发现
5. ✅ 权限提升 — 未发现
6. ✅ 隐蔽指令 — 未发现
7. ✅ 脚本文件 — setup.sh 仅包含 npm install

━━━━━━━━━━━━━━━━━━━
综合评级: ⚠️ 可疑（1 项需确认）
详情: 该 Skill 需要读取 OPENAI_API_KEY 来调用 API，这可能是正常功能需求。
是否继续安装？
```

## 技术细节

这是一个 **prompt-only skill**，不包含任何脚本。所有操作通过 Agent 的内置工具完成：

- `exec` — git clone / npm pack、文件列表、grep 扫描、安装复制
- `read` — 逐文件阅读内容进行深度审查

## License

[MIT](../../LICENSE)
