# 🧠 SKILLS

我的 Claude 技能库（Claude Skills Collection）

> 这里收录了我积累的 Claude 自定义技能（`.skill` 文件），按功能分类整理。
> 每个技能都可以安装到 Claude.ai，让 Claude 在特定场景下自动采用最佳工作流。

---

## 📂 目录结构

```
SKILLS/
├── web/          # 网页读取、抓取类技能
├── writing/      # 写作、内容创作类技能
├── tools/        # 工具集成类技能
└── ...           # 更多分类持续添加
```

---

## 📋 技能索引

### 🌐 web — 网页读取 / 抓取

| 技能名 | 描述 | 文件 |
|--------|------|------|
| `read-x-post` | 使用 tinyfish 读取 X（Twitter）帖子，默认全文翻译为中文；说「总结」则输出摘要 | [SKILL.md](./web/read-x-post/SKILL.md) |

---

## 🚀 如何安装技能

1. 下载对应的 `.skill` 文件
2. 前往 [Claude.ai](https://claude.ai) → 设置 → 技能（Skills）
3. 上传 `.skill` 文件即可生效

---

## 📝 更新记录

| 日期 | 变更 |
|------|------|
| 2026-05-07 | 🎉 初始化仓库，添加 `read-x-post` 技能 |
