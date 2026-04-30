---
name: ghl-create-workflow
version: 1.0.0
description: Use this skill when the user wants to create a new GoHighLevel workflow from scratch or based on an existing template. Covers plan-first conversation, entity validation, and clone-then-modify.
---

# Create a GoHighLevel Workflow

For tasks spanning multiple workflows (audit several, propose consolidations, edit-with-blast-radius), invoke the `orchestrator` agent instead.

Use this skill when the user asks to create a new GHL workflow — either from scratch or based on an existing one as a starting point.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

## Step 1: Read the Reference Material First

Before doing anything else, read:

- `ghl://skill/reference` — all valid trigger types, action types, and the JSON schema
- `ghl://skill/guide` — skill overview, wait-timing rules, JSON generation rules
- `ghl://entities/guide` — which fields require real IDs from the GHL location

Skipping these resources is the most common cause of broken workflows — you will invent action types or wait durations that do not exist.

## Phase 18: Preflight handshake & shape validation

1. **Read `skill/references/action-shapes.json` first.** Note the `skillAckVersion` field (currently `skill-ack-v1`). You'll pass it as `_skillAck` in your `create_workflow` call. Read it fresh — never hardcode the literal in caller code; if the registry bumps the version (D-15) the cached value goes stale and the call returns `skill_precondition`.
2. **Use only `type` values present in `action-shapes.json`.** The schema rejects unknown types at the SDK boundary before the handler runs (e.g. `update_opportunity_pipeline` is rejected; the real type is `internal_update_opportunity`).
3. **Use the `then`/`else` scaffold for binary if_else.** The verbose three-sibling wire form is still accepted (see `references/wiring.md`) but the scaffold is the simpler default — see `references/actions.md` §Authoring if/else for the shape:

   ```json
   {
     "type": "if_else",
     "name": "Has tag",
     "attributes": { "conditions": [ /* ... */ ] },
     "then": [ /* steps inside the matched branch */ ],
     "else": []
   }
   ```

   Do NOT mix scaffold and verbose forms in one branch. An action with `nodeType: "branch-yes"` / `"branch-no"` / `"condition-node"` placed inside a `then[]` / `else[]` triggers a HARD `malformed-if-else` rejection.

4. **Inspect the response envelope.** `partial: true` with `skippedActions[]` means some steps were salvaged. Read `class` per entry:
   - `unknown-type` (HARD) — type not in registry; the step was dropped. Re-author with a correct type from `action-shapes.json` and re-call `update_workflow`.
   - `malformed-if-else` (HARD) — a scaffold/verbose-form mix or a wrong-shape if_else; rewrite using the scaffold.
   - `malformed-shape-stripped` (SOFT) — known type but malformed attributes; the step was kept with placeholder `defaultAttributes` from the registry. Review and update interactively or via `update_workflow`.
   - `unverified-shape` (in `warnings[]`) — registry has the type but `shapeStatus !== "captured"`; the step was passed through unchanged for advisory only.

   When ANY HARD entry is present, the workflow saves with `status: "draft"` regardless of input (force-draft policy).

## Step 2: Choose the Starting Point

Two paths:

1. **From scratch.** Ask the user what the workflow should do. Clarify trigger, stop conditions, timing, and branching before writing anything.
2. **Based on an existing workflow.** If the user names a workflow to clone:
   - Call `read_workflow_outline` on the source workflow ID for a cheap step map (or `get_workflow` with `includeActionDetails=true` if you need the full JSON).
   - Explain what the source does in plain English first.
   - Ask what changes they want versus the original.
   - Use `clone_workflow` to duplicate it, then `patch_workflow` (with `insert_after` / `update_step` / `delete_step` / `move` ops) for targeted changes — fall back to `update_workflow` only for whole-workflow rewrites or branched-step splice ops. Never modify the original.

   **Worked example — clone-then-patch:**
   > User: "Clone the 'Lead nurture v2' workflow and change the email subject in step 2 to 'Quick check-in'."
   >
   > 1. `clone_workflow(sourceId)` → returns the new clone's ID.
   > 2. `read_workflow_outline(cloneId)` → identifies the email at step `S2`.
   > 3. `read_workflow_step(cloneId, 'S2')` → confirms step type and current attributes.
   > 4. `patch_workflow(cloneId, [{ op: 'update_step', step_id: 'S2', patch: { attributes: { subject: 'Quick check-in' } } }])` → applied; cache refreshes.
   > 5. Confirm with user, then `publish_workflow(cloneId)` if approved.

## Step 3: Discovery Phase (MANDATORY before any write tool)

Do not call `create_workflow`, `clone_workflow`, or `update_workflow` until every item below has been confirmed with the user. A vague request ("create a follow-up sequence") is not enough — drive it to specifics through questions.

Prefer the `AskUserQuestion` tool over inline prose for any choice with a small option set (trigger type, draft vs publish, folder placement, allowMultiple yes/no). Structured pickers get answered; walls of inline questions get skipped.

**Discovery checklist — must be answered before building:**

1. **Trigger** — exact event, plus filter conditions (which pipeline, which tag, which form, etc.). If the user says "when a contact comes in," ask *from where* — a form, a webhook, a tag add, a pipeline stage move?
2. **Audience scope** — `allowMultiple` (can the same contact enter twice?) and `stopOnResponse` (does an inbound reply halt the sequence?).
3. **Stop conditions** — what should remove a contact mid-flow (tag added, opportunity won/lost, unsubscribe)?
4. **Step list** — every action in order, in plain English, with timing relative to the previous step.
5. **Branching** — any if/else paths, and what each branch does.
6. **Entity references** — for any ID-bearing field (pipeline stage, user assignment, custom field, custom value, tag), confirm the exact name with the user *before* calling the matching `list_*` tool to resolve it.
7. **Naming + placement** — workflow name and target folder.
8. **Draft or publish** — never publish without an explicit "yes, publish."

**Worked example of good discovery:**

> User: "Make a workflow that follows up with new leads."
>
> Skill: "Before I build, a few things to nail down:
> 1. **Trigger** — should this fire on a new contact, a form submit, or when a contact gets a specific tag? (I'd lean form-submit unless you say otherwise.)
> 2. **Cadence** — how many follow-ups, and at what intervals? (e.g., 3 emails at day 0, day 2, day 5)
> 3. **Stop condition** — should it halt if they reply, book a call, or get a `customer` tag?
> 4. **Channel** — email only, or SMS too?
> 5. **Folder + name** — where should it live and what should I call it?"
>
> *(Skill waits for answers before any tool call.)*

**Then, only after discovery is complete:**

- Present a numbered plain-English plan using only real triggers and actions from the reference material. Never invent action types.
- Calculate wait-step durations relative to the **previous step** (not the workflow start). This is the single most common mistake — a 3-email sequence at days 1, 3, 7 uses waits of 1 day, 2 days, 4 days.
- Ask explicit yes/no confirmation before building.

## Step 4: Entity Validation (MANDATORY)

Before using any ID in an action or trigger, query the matching list tool first:

- Pipeline or stage IDs → `list_pipelines`
- User IDs (assignments, notifications) → `list_users`
- Custom field IDs → `list_custom_fields`
- Custom value IDs (template variables) → `list_custom_values`
- Tag names → `list_tags`

Never guess, approximate, or copy IDs from other workflows. The server validates references on save and returns `entityWarnings` with suggested fixes — read them and apply before publishing.

## Step 5: Build as Draft

> **Wiring is preserved verbatim.** The `next`, `parent`, `parentKey`, `sibling`, `nodeType`, and `attributes.targetNodeId` fields you submit ride through unchanged — auto-wiring only fills missing `next` fields on non-goto actions. See `references/wiring.md` for branching, multi-path, and GoTo patterns.

- Use the exact `type` values from `ghl://skill/reference`. Never approximate.
- Create with `status: 'draft'`. Never auto-publish without explicit user approval.
- Handle JSON internally — the user never needs to see it.

## Step 6: Show the Result

After creation, present:

- A plain-English summary of what was built (trigger, steps in order, wait timings)
- A direct link in exactly this format:
  `https://app.gohighlevel.com/location/{locationId}/workflow/{workflowId}`
  Replace `{locationId}` and `{workflowId}` with the actual values from the API response.
- Any `entityWarnings` from the create response, with suggested fixes.

Ask if they want to publish, leave as draft, or make further edits.
