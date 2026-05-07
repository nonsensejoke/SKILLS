---
name: read-x-post
description: 使用 tinyfish 读取 X（Twitter）帖子或长文内容，并默认将全文翻译为中文。当用户提供 x.com 或 twitter.com 链接，或者说"帮我看看这条推文/帖子"、"这篇文章在讲什么"、"翻译这条 X 帖子"等类似意图时，立即使用此技能。如果用户说"总结一下"或类似意思，则提供摘要而非全文翻译。需要 tinyfish (agent-tinyfish-ai) MCP 已连接。
---

# 读取 X (Twitter) 帖子

## 概述

X（Twitter）的页面依赖 JavaScript 渲染，普通 fetch 工具（如 web_fetch、fetch_content）无法获取内容。必须使用 `run_web_automation` 来完整渲染页面并提取帖子内容。

## 工作流程

### 第一步：加载 tinyfish 工具

调用 `tool_search`，查询 `"run web automation"` 或 `"fetch content"` 来加载 tinyfish 工具定义。

### 第二步：先尝试 fetch_content（可选快速路径）

```
agent-tinyfish-ai:fetch_content
  urls: ["<X帖子URL>"]
  format: "markdown"
  image_links: false
  include_html_head: false
  links: false
```

- 如果返回有效内容（不是 "JavaScript is disabled" 错误）→ 直接使用
- 如果返回 JavaScript 错误 → 立即跳到第三步

> 注意：X 几乎总是会触发 JS 错误，所以也可以直接跳过这步，直接用 run_web_automation。

### 第三步：使用 run_web_automation（主要方法）

```
agent-tinyfish-ai:run_web_automation
  url: "<X帖子URL>"
  session_id: "<随机 UUID v4>"
  goal: "Extract the full text content of this tweet/post, including the main post text, any replies, and any linked article content visible on the page."
  capture_config: { "screenshots": true }
```

**关键点：**
- `session_id` 每次必须是全新的随机 UUID v4，绝不能复用
- goal 用英文描述效果更好
- 该工具会完整渲染页面，能绕过 JS 限制

### 第四步：输出处理

根据用户意图选择输出方式：

**默认行为 → 全文翻译为中文**
- 将帖子正文、标题、作者信息完整翻译成中文
- 保留原文中的链接、列表结构
- 格式清晰，分段呈现

**用户说"总结"或类似意思 → 提供摘要**
- 用中文简明扼要地概括核心观点
- 列出关键信息（作者、主题、结论、链接等）
- 不需要逐字翻译

## 示例触发语

- "https://x.com/xxx/status/xxx 这篇在讲什么"
- "帮我翻译这条推文：[链接]"
- "这个 X 帖子说了什么"
- "总结一下这篇 Twitter 文章"

## 常见问题

| 问题 | 原因 | 解决方法 |
|------|------|----------|
| fetch_content 返回"JavaScript disabled" | X 需要 JS 渲染 | 改用 run_web_automation |
| web_fetch 被 robots.txt 拒绝 | X 禁止爬虫 | 同上 |
| run_web_automation 超时 | 网络慢或页面复杂 | 检查 list_runs，不要重复调用 |
| 帖子内容为空 | 需要登录才能查看 | 告知用户该内容需要登录，无法获取 |
