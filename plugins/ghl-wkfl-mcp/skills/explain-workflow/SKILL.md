---
name: ghl-explain-workflow
version: 1.0.0
description: Use this skill when the user wants a plain-English explanation of what a GoHighLevel workflow does — triggers, steps, wait timings, branching, and settings.
---

# Explain a GoHighLevel Workflow in Plain English

For tasks spanning multiple workflows (audit several, propose consolidations, edit-with-blast-radius), invoke the `orchestrator` agent instead.

Use this skill when the user wants to understand what an existing GHL workflow does. The output is a plain-English explanation — no JSON, no internal ID dumps, no assumptions.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

## Step 1: Read the Reference Material

Before explaining anything, read:

- `ghl://skill/reference` — so you can decode the `type` values on each trigger and action
- `ghl://skill/guide` — wait-timing rules (waits are relative to the previous step, not the workflow start)

Without these you will mislabel action types or misreport timings.

## Step 2: Identify the Workflow

If the user did not name a specific workflow:

1. Call `list_workflows` to show what is available.
2. Ask which one they want explained.

Once you have the workflow ID:

- Call `get_workflow` with `includeActionDetails=true` so you have every action's configuration, not just names.

## Step 3: Explain the Workflow

Produce a plain-English explanation in this shape:

### Summary (one paragraph)

- Workflow name and current status (published / draft)
- Number of steps
- What it does in one sentence

### Trigger

- Trigger type and display name
- Active conditions (filter fields and values)
- Whether it fires on specific records or broadly

### Steps in Order

Number each step. For each one include:

- Step name and action type (use human-readable labels from the reference, not the raw `type` string)
- What it does in plain English — avoid JSON or internal field names unless the user asks
- **Wait timing**: show both the relative wait ("2 days after step 3") and the cumulative time from trigger ("so 5 days after entry")
- Any branching logic (if/else, multipath) with each branch's path

### Settings

- `allowMultiple` — can a contact enter more than once?
- `stopOnResponse` — does the workflow stop if the contact replies?
- Other relevant settings

### Potential Issues (optional but recommended)

Flag anything that looks wrong:

- Wait timings that appear to be calculated from entry rather than from the previous step
- Workflows with no `stopOnResponse` that probably should have it
- References to IDs that cannot be validated from the pulled JSON alone (flag for the user)
- `goto` loops without an exit condition

## Step 4: Format for Readability

- Use a numbered list for the steps in order.
- Use plain English, not JSON.
- Use UK English spelling.
- Do not dump raw UUIDs unless the user explicitly asks.

## Step 5: Offer Follow-Ups

End with an offer to go deeper — edit, audit, clone, or publish — so the user has a natural next step.
