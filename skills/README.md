# GHL Workflow Skills

> Skills that teach Claude (Desktop, Code, Cursor) how to build, edit, explain, and audit GoHighLevel workflows. These files are consumed by the [GHL Workflow MCP server](..) — install the MCP first, then pick an install path below for the skills.

Version: `v2.0.0-beta.1`

## The five skills

| Skill | When to invoke | Source |
|-------|----------------|--------|
| `ghl-create-workflow` | Build a new GHL workflow, from scratch or cloned from an existing one | [create-workflow/SKILL.md](create-workflow/SKILL.md) |
| `ghl-plan-workflow` | Plan a workflow conversationally before any write tool is called | [plan-workflow/SKILL.md](plan-workflow/SKILL.md) |
| `ghl-edit-workflow` | Modify an existing workflow (triggers, actions, timing, branches) | [edit-workflow/SKILL.md](edit-workflow/SKILL.md) |
| `ghl-explain-workflow` | Read a workflow and describe what it does in plain English | [explain-workflow/SKILL.md](explain-workflow/SKILL.md) |
| `ghl-audit-workflow` | Scan a workflow for broken entity references (missing tags, deleted users, etc.) | [audit-workflow/SKILL.md](audit-workflow/SKILL.md) |

Each `SKILL.md` has YAML frontmatter (`name:`, `description:`) per Claude's skill spec. Claude Desktop also supports `.skill` archive format — those are mirrored below.

## Three install paths

Pick the one that matches your comfort level.

### 1. Easy: run the installer

If you installed the [GHL Workflow MCP binary](..), the installer already extracted these skills to `~/.claude/skills/ghl-wkfl/`. No action needed — skip to [Usage](..#usage).

Want to re-extract after a manual delete?

```
./ghl-wkfl-mcp setup --install-skills
```

This refreshes `~/.claude/skills/ghl-wkfl/` from the binary-embedded SKILL.md files. Useful after a manual delete or when upgrading to a newer binary.

### 2. Technical: git clone this repo

```
git clone https://github.com/ampware-io/ghl-workflow-mcp.git
mkdir -p ~/.claude/skills/ghl-wkfl
cp -r ghl-workflow-mcp/skills/create-workflow \
      ghl-workflow-mcp/skills/plan-workflow \
      ghl-workflow-mcp/skills/edit-workflow \
      ghl-workflow-mcp/skills/explain-workflow \
      ghl-workflow-mcp/skills/audit-workflow \
      ~/.claude/skills/ghl-wkfl/
```

Restart your Claude client.

### 3. Surgical: curl one skill at a time

```
# Example for create-workflow — repeat for each skill you want
mkdir -p ~/.claude/skills/ghl-wkfl/create-workflow
curl -fsSL \
  https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/create-workflow/SKILL.md \
  -o ~/.claude/skills/ghl-wkfl/create-workflow/SKILL.md
```

Available raw URLs:

- `https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/create-workflow/SKILL.md`
- `https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/plan-workflow/SKILL.md`
- `https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/edit-workflow/SKILL.md`
- `https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/explain-workflow/SKILL.md`
- `https://raw.githubusercontent.com/ampware-io/ghl-workflow-mcp/main/skills/audit-workflow/SKILL.md`

### 4. Claude Desktop skill-drag (non-technical)

Claude Desktop supports importing `.skill` archives via its Skills panel. Download any of the five archives below, then drag-and-drop into Desktop's Skills UI:

- `ghl-wkfl-create-workflow.skill`
- `ghl-wkfl-plan-workflow.skill`
- `ghl-wkfl-edit-workflow.skill`
- `ghl-wkfl-explain-workflow.skill`
- `ghl-wkfl-audit-workflow.skill`

Desktop's Skills UI rejects multi-skill archives — that's why we ship five separate single-skill archives instead of one aggregate. Each archive holds a single SKILL.md; drag each one in turn.

## Learn more

- [Main README](..#readme) — MCP server install, Free vs Paid, feature matrix
- [Purchase a license](https://ampware.io/ghl-wkfl-mcp/buy) — unlock the 6 paid workflow-mutation tools (`create_workflow`, `update_workflow`, `delete_workflow`, `publish_workflow`, `clone_workflow`, `create_tag`)
