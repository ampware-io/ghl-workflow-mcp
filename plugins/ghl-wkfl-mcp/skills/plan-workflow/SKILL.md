---
name: ghl-plan-workflow
description: Use this skill when the user describes a business process and wants it planned as a GHL workflow — no creation yet, just a numbered plain-English plan with real triggers and actions.
---

# Plan a GoHighLevel Workflow

Use this skill when the user describes a business process and wants it planned — not built yet. The output is a numbered plain-English plan that maps the process to real GHL triggers and actions, with correctly-calculated wait timings.

## Step 1: Read the Reference Material

Before planning anything, read:

- `ghl://skill/reference` — valid trigger types, action types, JSON schema
- `ghl://skill/guide` — wait-timing rules and workflow conventions

You cannot map a process to GHL without knowing which triggers and actions actually exist.

## Step 2: Clarify Ambiguity

Ask about anything unclear before producing a plan:

- **Trigger** — what event starts the workflow? Be specific: "appointment booked" and "appointment status changed" are different triggers.
- **Goal** — what is the desired end state?
- **Branches** — are there different paths (e.g., if they reply vs if they do not)?
- **Timing** — when should each step happen relative to the trigger or previous step?
- **Stop conditions** — should the workflow stop if the contact replies, books, buys, or enters a specific stage?
- **Settings** — should contacts be able to enter multiple times (`allowMultiple`)? Should the workflow stop on response (`stopOnResponse`)?

## Step 3: Map to Real Triggers and Actions

- Identify the correct trigger from the triggers reference. Use the exact `type` value.
- Map each step to a real action from the actions reference. Never invent actions — if something does not exist, say so explicitly.
- Calculate wait-step durations **relative to the previous step**, not the workflow start. Example: emails at days 1, 3, 7 from entry use waits of 1 day, 2 days, 4 days.

## Step 4: Output the Plan

Format the plan as:

```
TRIGGER: [trigger type] — [conditions]
WORKFLOW SETTINGS: allowMultiple: true/false, stopOnResponse: true/false

1. [Action name] — [what it does]
2. Wait X [units] — (cumulative: Y [units] from trigger)
3. [Action name] — [what it does]
4. Wait X [units] — (cumulative: Y [units] from trigger)
...
```

Always show cumulative time from trigger alongside each wait so the user can sanity-check the overall timeline.

## Step 5: Flag Pre-Build Decisions

Call out anything that needs user input before `create_workflow` could run:

- Which calendar for booking links (need calendar ID via `list_calendars` if available)
- Which email template IDs (user must supply or approve)
- Custom field IDs if updating contact fields (need `list_custom_fields`)
- Whether to use `stopOnResponse` (recommended for most follow-up sequences)

## Step 6: Offer the Next Step

Once the plan is finalised, ask: **"Want me to build this workflow now?"**

If they say yes, switch to the `start_create_workflow` flow and follow that skill. If they want to revise, iterate on the plan first.

Never call `create_workflow` from the planning skill itself — planning is read-only.
