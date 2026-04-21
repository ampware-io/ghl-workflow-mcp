---
name: ghl-audit-workflow
description: Use this skill when the user wants to audit a GoHighLevel workflow for broken entity references (invalid pipeline/stage/user/custom-field/custom-value IDs) before publishing or editing.
---

# Audit a GoHighLevel Workflow for Broken Entity References

Use this skill when the user wants to validate an existing workflow — checking that every ID it references (pipelines, stages, users, custom fields, custom values, tags) still exists in the current GHL location. Audit is read-only; it never changes the workflow.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

Typical triggers for running an audit:

- Before publishing a draft workflow
- Before editing a workflow that has been inactive for a while
- After importing or cloning a workflow from another location
- When a workflow is firing but producing unexpected results and you suspect stale IDs

## Step 1: Read the Reference Material

Before auditing, read:

- `ghl://skill/reference` — to understand which action and trigger types reference which entities
- `ghl://entities/guide` — which fields require real IDs and how to validate them

This lets you interpret the audit output correctly.

## Step 2: Identify the Workflow

If the user did not name a specific workflow:

1. Call `list_workflows` to show what is available.
2. Ask which one they want audited.

Once you have the workflow ID:

- Call `get_workflow` with `includeActionDetails=true` to pull the full workflow (the audit needs every action's configuration, not just step names).

## Step 3: Run the Audit

Call the `audit_workflow` tool with the workflow ID. It returns two content blocks:

1. A Markdown report summarising valid vs invalid references
2. A JSON report with the same information in structured form (use this for precise field-by-field fixes)

## Step 4: Explain the Results

Summarise the audit in plain English:

### Valid References

- How many references were checked
- Which types were all-valid (pipelines, users, custom fields, etc.)

### Broken References

For each invalid reference, explain:

- **Where it lives** — which step (by name and step number), and which field
- **What it was trying to reference** — e.g., "pipeline ID `abc-123`"
- **Why it is broken** — the pipeline was deleted, the stage was renamed, the user was removed, etc.
- **Suggested fix** — the audit returns `entityWarnings` with candidate replacements when a close match is found. Present these clearly.

### Recommended Next Steps

For each broken reference, point the user at the list tool that produces valid replacements:

- Broken pipeline or stage → `list_pipelines`
- Broken user assignment → `list_users`
- Broken custom field → `list_custom_fields`
- Broken custom value → `list_custom_values`
- Broken tag → `list_tags`

## Step 5: Do Not Auto-Fix

Audit is diagnostic. Do not call `update_workflow` or `publish_workflow` as part of the audit flow — the user must decide which replacements to apply. Offer to switch to the `edit-workflow` skill next if they want help applying the fixes.

## Step 6: If the Audit Is Clean

Say so explicitly: "No broken entity references found." Mention how many of each entity type were validated so the user knows the audit actually ran against real data.
