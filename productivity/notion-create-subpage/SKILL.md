---
name: notion-create-subpage
description: Creates a new Notion page as a child of a specified parent page. Use this skill whenever the user wants to create a subpage, child page, or nested page inside an existing Notion page — e.g. "在 xxx 下创建一个新页面 yyy", "create a page under xxx", "add a subpage to xxx", "在 xxx 页面的层级下新建 yyy". The Notion MCP's parent_id parameter does not work for hierarchy placement; this skill uses a reliable create-then-move two-step approach instead.
compatibility:
  tools:
    - Notion:notion-search
    - Notion:notion-create-pages
    - Notion:notion-move-pages
    - Notion:notion-fetch
---

# Notion: Create Subpage Under a Parent

## Background

The Notion MCP tool `notion-create-pages` accepts a `parent_id` parameter, but it **does not actually place the new page under that parent** — the page is silently created at the workspace root level. The reliable workaround is a two-step approach: create first, then move.

## Workflow

### Step 1 — Resolve the parent page ID

If the user provided a Notion URL or page ID directly, use it. Otherwise, search by name:

```
Notion:notion-search  query="<parent page name>"
```

- Pick the best match from results (check title and type).
- Extract the `id` field — this is the **parent_id** to use in steps 2 and 3.
- If multiple plausible matches exist, confirm with the user before proceeding.

### Step 2 — Create the new page

```
Notion:notion-create-pages
  pages: [{ "properties": { "title": "<new page name>" } }]
  parent_id: <parent_id>   # NOTE: this param has no effect on placement; pass it anyway
```

- Note the returned `id` of the new page — this is the **new_page_id**.

### Step 3 — Move the new page to the correct parent

```
Notion:notion-move-pages
  page_or_database_ids: ["<new_page_id>"]
  new_parent: { "type": "page_id", "page_id": "<parent_id>" }
```

This is the step that **actually** places the page in the correct hierarchy.

### Step 4 — Verify (recommended)

Fetch the parent page and confirm the new page appears in its `<content>` block as a `<page>` entry:

```
Notion:notion-fetch  id="<parent_id>"
```

Look for a line like:
```
<page url="https://www.notion.so/...">new page name</page>
```

at the bottom of the content. If it appears, the subpage was created successfully.

## Output to user

Report:
- ✅ New page name
- 📄 Parent page name
- 🔗 Link to the new page (`url` from Step 2 result)

## Edge cases

- **Parent not found**: Tell the user no matching page was found, ask them to confirm the page name or provide a URL.
- **Move fails**: If `notion-move-pages` returns an error, report it and suggest the user manually drag the page in Notion.
- **Duplicate name**: Proceed — Notion allows pages with duplicate titles. Mention it to the user if detected.
- **User provides a URL**: Extract the page ID from the URL path (the 32-char hex string, with or without dashes).
