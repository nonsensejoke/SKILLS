---
name: add-skill-to-repo
description: 将新创建的 Claude Skill 推送到用户的专属 GitHub Skill 仓库（nonsensejoke/SKILLS），并自动更新 README.md 索引。当用户说"把这个 skill 上传到我的 repo"、"推送到 skill 仓库"、"保存到 GitHub"、"add to repo"、"上传到 nonsensejoke/SKILLS" 等类似意图时立即使用此技能。同时负责在每次推送后维护 README.md 的分类目录表格和更新日志。需要 GitHub MCP 已连接。
---

# 上传 Skill 到专属仓库

## 仓库信息

- **Owner**: `nonsensejoke`
- **Repo**: `SKILLS`
- **Branch**: `main`
- **仓库地址**: https://github.com/nonsensejoke/SKILLS

## 文件夹结构规则

```
SKILLS/
├── README.md                        ← 总索引，每次必须更新
├── social-media/                    ← 分类文件夹
│   └── read-x-post/
│       └── SKILL.md
├── productivity/                    ← 示例：其他分类
│   └── some-skill/
│       └── SKILL.md
└── ...
```

**分类参考**（根据 skill 用途选择，或新建）：

| 分类文件夹 | 适用场景 |
|-----------|----------|
| `social-media` | X/Twitter、微博、Reddit 等社交平台 |
| `productivity` | 任务管理、日历、笔记、效率工具 |
| `writing` | 写作、编辑、翻译、博客发布 |
| `coding` | 代码生成、调试、GitHub 操作 |
| `research` | 搜索、文献、信息整理 |
| `data` | 数据处理、表格、可视化 |
| `media` | 图片、视频、音频处理 |

如果现有分类都不合适，可新建一个语义清晰的英文小写文件夹名。

---

## 工作流程

### 第一步：加载 GitHub 工具

调用 `tool_search`，查询 `"github create file push repository"` 加载 GitHub MCP 工具。

### 第二步：读取仓库现状

```
github:get_file_contents
  owner: "nonsensejoke"
  repo: "SKILLS"
  path: "/"
```

- 若仓库为空（报错 "Git Repository is empty"）→ 跳到第四步，直接创建所有文件
- 若仓库已有内容 → 继续第三步

### 第三步：读取现有 README.md

```
github:get_file_contents
  owner: "nonsensejoke"
  repo: "SKILLS"
  path: "README.md"
```

记录返回的 `sha` 值，更新文件时必须提供。同时解析现有的分类表格和更新日志，以便追加新条目。

### 第四步：确定分类和路径

根据 skill 的用途，选择或新建分类文件夹，确定：
- `skill_category`：如 `social-media`
- `skill_name`：与 skill 的 `name` frontmatter 一致
- `skill_path`：`{skill_category}/{skill_name}/SKILL.md`

### 第五步：推送 skill 文件

使用 `github:push_files` 推送 skill 文件：

```
github:push_files
  owner: "nonsensejoke"
  repo: "SKILLS"
  branch: "main"
  message: "✨ Add {skill_name} skill"
  files:
    - path: "{skill_category}/{skill_name}/SKILL.md"
      content: <skill 的完整 SKILL.md 内容>
```

### 第六步：更新 README.md

用 `github:create_or_update_file` 更新 README（必须带 sha）：

```
github:create_or_update_file
  owner: "nonsensejoke"
  repo: "SKILLS"
  branch: "main"
  path: "README.md"
  sha: <第三步获取的 sha>
  message: "📝 Update README: add {skill_name}"
  content: <更新后的完整 README 内容>
```

**更新规则：**
- 若该分类已存在 → 在对应表格末尾追加一行
- 若该分类不存在 → 在 `## 📂 分类目录` 下新增一个 `###` 小节和表格
- 更新记录追加到表格顶部（最新在最上）

---

## README.md 格式模板

```markdown
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

### ⚙️ {新分类} — {中文说明}

| Skill | 描述 | 依赖 |
|-------|------|------|
| [{skill-name}](./{category}/{skill-name}/SKILL.md) | {描述} | {依赖} |

---

## 📋 更新记录

| 日期 | 操作 | Skill |
|------|------|-------|
| {今天日期} | ✨ 新增 | `{category}/{skill-name}` |
| 2026-05-07 | ✨ 新增 | `social-media/read-x-post` |
```

---

## 注意事项

**关于 README 更新方式：**
- 仓库为空时：用 `push_files` 同时创建 skill 文件和 README（一次 commit）
- 仓库已有 README 时：先 `get_file_contents` 获取 sha → `push_files` 推 skill → `create_or_update_file` 更新 README（带 sha）

**关于 skill 内容来源：**
- 对话中刚创建的 skill → 读取 `/home/claude/{skill-name}/SKILL.md`
- 用户提供 `.skill` 文件 → 解压后读取 SKILL.md
- 用户直接粘贴内容 → 直接使用

**今天日期：** 推送时使用当前真实日期（格式 YYYY-MM-DD）填入更新记录。
