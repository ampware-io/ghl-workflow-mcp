---
name: ghl-create-workflow
version: 1.0.0
description: Use this skill when the user wants to create a new GoHighLevel workflow from scratch or based on an existing template. Covers plan-first conversation, entity validation, and clone-then-modify.
---

# Create a GoHighLevel Workflow

Use this skill when the user asks to create a new GHL workflow — either from scratch or based on an existing one as a starting point.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

## Step 1: Read the Reference Material First

Before doing anything else, read:

- `ghl://skill/reference` — all valid trigger types, action types, and the JSON schema
- `ghl://skill/guide` — skill overview, wait-timing rules, JSON generation rules
- `ghl://entities/guide` — which fields require real IDs from the GHL location

Skipping these resources is the most common cause of broken workflows — you will invent action types or wait durations that do not exist.

## Step 2: Choose the Starting Point

Two paths:

1. **From scratch.** Ask the user what the workflow should do. Clarify trigger, stop conditions, timing, and branching before writing anything.
2. **Based on an existing workflow.** If the user names a workflow to clone:
   - Call `get_workflow` with `includeActionDetails=true` on the source workflow ID.
   - Explain what the source does in plain English first.
   - Ask what changes they want versus the original.
   - Use `clone_workflow` to duplicate it, then `update_workflow` to apply the changes. Never modify the original.

## Step 3: Plan Conversationally

Build the plan with the user before calling any write tool:

1. Clarify ambiguity — trigger type, timing, stop conditions, branching.
2. Present a numbered plain-English plan using only real triggers and actions from the reference material. Never invent action types.
3. Calculate wait-step durations relative to the **previous step** (not the workflow start). This is the single most common mistake — a 3-email sequence at days 1, 3, 7 uses waits of 1 day, 2 days, 4 days.
4. Ask explicit yes/no confirmation before building.

## Step 4: Entity Validation (MANDATORY)

Before using any ID in an action or trigger, query the matching list tool first:

- Pipeline or stage IDs → `list_pipelines`
- User IDs (assignments, notifications) → `list_users`
- Custom field IDs → `list_custom_fields`
- Custom value IDs (template variables) → `list_custom_values`
- Tag names → `list_tags`

Never guess, approximate, or copy IDs from other workflows. The server validates references on save and returns `entityWarnings` with suggested fixes — read them and apply before publishing.

## Step 5: Build as Draft

- Use the exact `type` values from `ghl://skill/reference`. Never approximate.
- Create with `status: 'draft'`. Never auto-publish without explicit user approval.
- Handle JSON internally — the user never needs to see it.

## Step 6: Show the Result

After creation, present:

- A plain-English summary of what was built (trigger, steps in order, wait timings)
- A direct link in exactly this format:
  `https://app.gohighlevel.com/location/{locationId}/automation/workflows/{workflowId}`
  Replace `{locationId}` and `{workflowId}` with the actual values from the API response.
- Any `entityWarnings` from the create response, with suggested fixes.

Ask if they want to publish, leave as draft, or make further edits.
