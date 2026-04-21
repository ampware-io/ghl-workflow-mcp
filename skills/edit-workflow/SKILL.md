---
name: ghl-edit-workflow
description: Use this skill when the user wants to modify an existing GoHighLevel workflow. Explains the current workflow in plain English, confirms changes, then applies via update_workflow.
---

# Edit an Existing GoHighLevel Workflow

Use this skill when the user asks to change, adjust, or extend an existing GHL workflow. The flow is: fetch, explain, confirm, apply.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

## Step 1: Read the Reference Material

Before fetching or editing anything, read:

- `ghl://skill/reference` — valid trigger types, action types, JSON schema
- `ghl://skill/guide` — wait-timing rules and workflow conventions
- `ghl://entities/guide` — which fields require real IDs from the GHL location

## Step 2: Identify the Workflow

If the user did not name a specific workflow:

1. Call `list_workflows` to show what is available.
2. Ask which one they want to edit.

Once you have the workflow ID:

- Call `get_workflow` with `includeActionDetails=true` to pull the current structure.

## Step 3: Explain Before Editing

Before touching anything, explain what the workflow currently does in plain English:

1. What triggers it (event + conditions)
2. Each step in order — what it does and why
3. Wait-step timing — show both the relative wait and the cumulative time from trigger
4. Any branching logic or conditions
5. Workflow settings (`allowMultiple`, `stopOnResponse`, etc.)

This confirms you understand the workflow the same way the user does before changing it.

## Step 4: Confirm the Change Set

Ask what they want to change (if not already stated). Then present a **change summary** before running `update_workflow`:

- What will change (steps added, removed, reordered, modified)
- What stays the same
- Any downstream timing impact — adding a step between existing steps shifts every subsequent cumulative time

Wait for explicit yes/no confirmation. Never apply edits silently.

## Step 5: Apply with Discipline

When calling `update_workflow`:

- Use exact `type` values from the reference — never approximate or invent actions.
- Calculate wait-step durations relative to the **previous step**, not the workflow start.
- The user never needs to see JSON — handle it internally.
- Preserve fields you are not deliberately changing.

## Step 6: Entity Validation (MANDATORY)

Before referencing any ID in a changed or new step, query the matching list tool:

- Pipeline or stage IDs → `list_pipelines`
- User IDs → `list_users`
- Custom field IDs → `list_custom_fields`
- Custom value IDs (template variables) → `list_custom_values`
- Tag names → `list_tags`

The server validates entity references on save and returns `entityWarnings` with suggested fixes — read them and apply corrections before considering the edit done.

## Step 7: Show the Result

After the update, present:

- A plain-English summary of what changed
- A direct link:
  `https://app.gohighlevel.com/v2/location/{locationId}/automation/workflows/{workflowId}`
  Replace `{locationId}` and `{workflowId}` with the actual values.
- Any `entityWarnings` from the response, with recommended corrections.

Ask if they want to publish, keep as draft, or make further edits.
