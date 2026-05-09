# 🧠 SKILLS

我的 Claude Skill 收藏库。每个 Skill 是一段让 Claude 掌握特定工作流的指令文件，可直接安装到 Claude.ai 使用。

## 📦 安装方式

下载对应文件夹中的 `.skill` 文件（或直接使用 `SKILL.md`），在 Claude.ai 设置页面中安装。

---

## 📂 分类目录

### 🐦 social-media — 社交媒体

| Skill | 描述 | 依赖 |
|-------|------|------|
| [read-x-post](./social-media/read-x-post/SKILL.md) | 使用 tinyfish 读取 X (Twitter) 帖子并翻译/总结为中文 | tinyfish MCP |

### ⚙️ coding — 开发工具

| Skill | 描述 | 依赖 |
|-------|------|------|
| [add-skill-to-repo](./coding/add-skill-to-repo/SKILL.md) | 将新创建的 Skill 推送到专属 GitHub 仓库并自动更新 README 索引 | GitHub MCP |
| [save-to-snippets](./coding/save-to-snippets/SKILL.md) | 将代码片段、脚本或工具保存到 `nonsensejoke/my_code_snippets` 仓库 | GitHub MCP |

### 🔌 mcp — MCP 配置

| Skill | 描述 | 依赖 |
|-------|------|------|
| [tinyfish-mcp](./mcp/tinyfish-mcp/SKILL.md) | 为 Hermes Agent 配置 TinyFish MCP（网页搜索 + 页面抓取），含完整 OAuth PKCE 认证流程 | python3, firefox |

---

## 📋 更新记录

| 日期 | 操作 | Skill |
|------|------|-------|
| 2026-05-09 | 📝 补充文档 | `coding/save-to-snippets`（已有，首次写入 README） |
| 2026-05-09 | 📝 补充文档 | `mcp/tinyfish-mcp`（已有，首次写入 README） |
| 2026-05-07 | ✨ 新增 | `coding/add-skill-to-repo` |
| 2026-05-07 | ✨ 新增 | `social-media/read-x-post` |

---

> 如需添加新 Skill，请在对应分类子文件夹中创建目录，并更新本 README。
