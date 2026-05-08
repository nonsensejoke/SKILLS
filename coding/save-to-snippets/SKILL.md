---
name: save-to-snippets
description: 将代码片段、脚本或工具保存到用户的代码仓库 nonsensejoke/my_code_snippets。当用户说"把这个脚本存起来"、"保存到我的 snippet repo"、"上传到我的代码仓库"、"save this to my snippet repo"、"存到 my_code_snippets"，或者在完成一个脚本/工具后说"帮我存一下"时，立即使用此技能。需要 GitHub MCP 已连接。
---

# 保存代码到 Snippet Repo

## 仓库信息

- **Owner**: `nonsensejoke`
- **Repo**: `my_code_snippets`
- **Branch**: `main`
- **地址**: https://github.com/nonsensejoke/my_code_snippets

## 现有目录结构（持续更新）

```
my_code_snippets/
├── amber_md/        # AMBER 分子动力学相关
├── linux/           # Linux 命令/系统工具
├── pdb_seq_alignment/  # PDB 序列比对
└── pymol/           # PyMOL 相关脚本
```

---

## 工作流程

### 第一步：加载 GitHub 工具

调用 `tool_search`，查询 `"github push files repository"` 加载工具。

### 第二步：确定目标目录

根据代码内容选择已有目录，或新建一个语义清晰的英文小写目录名。

| 内容类型 | 推荐目录 |
|---------|----------|
| AMBER MD / GROMACS 等分子动力学 | `amber_md/` |
| Linux shell / 系统工具 | `linux/` |
| PDB / 序列处理 | `pdb_seq_alignment/` |
| PyMOL 脚本 | `pymol/` |
| 其他结构生物学工具 | 新建合适目录，如 `structure_tools/` |
| Python 通用工具 | 新建 `python/` 或更具体的名字 |

如果不确定，简要询问用户，或根据上下文自行判断并告知用户你的选择。

### 第三步：确定文件名

- 优先沿用用户在对话中已命名的文件名（如 `cif2pdb.sh`）
- 若无明确名称，根据功能生成一个简洁的英文文件名（如 `prep_topology.sh`、`batch_rmsd.py`）
- 文件扩展名与语言对应：bash → `.sh`，Python → `.py`，PyMOL → `.pml`

### 第四步：推送文件

使用 `github:push_files`：

```
github:push_files
  owner: "nonsensejoke"
  repo: "my_code_snippets"
  branch: "main"
  message: "add {filename}"
  files:
    - path: "{directory}/{filename}"
      content: <完整代码内容>
```

### 第五步：确认并回报

推送成功后，输出：
```
✓ 已保存到 {directory}/{filename}
https://github.com/nonsensejoke/my_code_snippets/blob/main/{directory}/{filename}
```

---

## 注意事项

- **代码来源**：从对话中直接获取代码内容，完整保留，不做任何修改
- **新建目录**：GitHub 没有空目录概念，直接写 `new_dir/file.sh` 即可，目录会自动创建
- **多文件**：如果需要同时保存多个文件，在同一个 `push_files` 调用中用 `files` 数组一次提交
