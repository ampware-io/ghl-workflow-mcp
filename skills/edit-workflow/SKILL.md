---
name: ghl-edit-workflow
version: 1.0.0
description: Use this skill when the user wants to modify an existing GoHighLevel workflow. Explains the current workflow in plain English, confirms changes, then applies via update_workflow.
---

# Edit an Existing GoHighLevel Workflow

For tasks spanning multiple workflows (audit several, propose consolidations, edit-with-blast-radius), invoke the `orchestrator` agent instead.

Use this skill when the user asks to change, adjust, or extend an existing GHL workflow. The flow is: fetch, explain, confirm, apply.

## Launch gate — always ask before opening Chrome

This skill needs Chrome running and connected to GoHighLevel. Before calling `launch_chrome`, call `test_connection` first; if not connected, ask the user in one line ("Chrome needs to launch to reach GoHighLevel — OK?") and wait for an affirmative reply. Never call `launch_chrome` silently. See `ghl://skill/guide` §Launch gate for the full rule.

## Step 1: Read the Reference Material

Before fetching or editing anything, read:

- `ghl://skill/reference` — valid trigger types, action types, JSON schema
- `ghl://skill/guide` — wait-timing rules and workflow conventions
- `ghl://entities/guide` — which fields require real IDs from the GHL location

## Phase 18: Preflight handshake & shape validation

1. **Read `skill/references/action-shapes.json` first.** Note the `skillAckVersion` field (currently `skill-ack-v1`). You'll pass it as `_skillAck` in your `update_workflow` (and `patch_workflow`, `clone_workflow`, `delete_workflow`, `publish_workflow`) calls. Read it fresh — never hardcode in caller code; if the registry bumps the version (D-15) the cached value goes stale and the call returns `skill_precondition`.
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
   - `malformed-shape-stripped` (SOFT) — known type but malformed attributes; the step was kept with placeholder `defaultAttributes` from the registry. Review and patch interactively or via `update_workflow`/`patch_workflow`.
   - `unverified-shape` (in `warnings[]`) — registry has the type but `shapeStatus !== "captured"`; the step was passed through unchanged for advisory only.

   When ANY HARD entry is present, the workflow saves with `status: "draft"` regardless of input (force-draft policy).

## Step 2: Identify the Workflow

If the user did not name a specific workflow:

1. Call `list_workflows` to show what is available.
2. Ask which one they want to edit.

Once you have the workflow ID:

- Call `get_workflow` with `includeActionDetails=true` to pull the current structure.

> **Prefer targeted reads.** If you only need a specific step or just the step list, prefer `read_workflow_outline(workflowId)` (cheap step map — IDs/types/names/order/next/parentKey, no attributes) and `read_workflow_step(workflowId, stepId)` (single step's full JSON) over a full `get_workflow`. Both populate the silent cache automatically. Reach for `get_workflow` only when you need the entire workflow JSON (whole-workflow rewrite, full audit, or when no patch op fits the planned change).

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

## Step 4.5: Anti-pattern — handing the work back to the user

**Forbidden response shape:**

> "Given how deeply branched this workflow already is, I'd break it trying to insert these blocks via JSON. Safer for me to draft the variants here, you create the wrappers and drop them in via UI."

That is an abdication, not a recommendation. The skill exists *because* the user does not want to hand-edit JSON in the GHL UI. Pushing the work back is a skill failure.

**The only acceptable reasons to not proceed are:**

- License inactive (`license_status` returned no active license)
- Chrome not connected and user declined `launch_chrome`
- User explicitly said "draft only, do not write"
- A required entity (pipeline, user, custom field) does not exist in the location and cannot be created here

Anything else — including "this workflow is complex," "I haven't seen this schema before," "I might break it," or "this branching is hard to reason about" — is **not** a valid stop condition. Use the recovery procedure below instead.

## Step 4.6: Recovery procedure for risky edits

When the workflow is deeply branched, the schema is unfamiliar, or you genuinely worry about breaking something:

1. **Clone first.** Call `clone_workflow` to make a working copy. Mutate the clone, never the original. Tell the user you're working on a clone and will swap once verified.
2. **Smallest atomic edit.** Make one change at a time. After each `update_workflow`, call `get_workflow` to diff against expectations before the next change.
3. **Unfamiliar schema → go look at one.** If you have not seen a node type or condition shape before (e.g. comparing two custom values), call `list_workflows`, find a workflow that already uses that shape, `get_workflow` it, and copy the schema. Do not punt because the schema is new.
4. **If a single mutation fails repeatedly** with the same `entityWarnings` or schema error after two corrected attempts, *then* surface the specific blocker to the user — name the field, the value you tried, and what the server rejected. That is a debugging conversation, not an abdication.

The bar: you do not exit the loop until either the change is applied, or you've hit one of the four allowed stop conditions above.

## Step 4.7: Targeted edits via `patch_workflow`

For most edits, prefer `patch_workflow` over `update_workflow`. The patch path is server-side pointer-rewired, all-or-nothing, and avoids burning tokens on whole-workflow JSON the model doesn't need.

**Default path:**

- Call `patch_workflow(workflowId, operations)` with explicit ops drawn from this set:

| Op | Shape | Use for |
|----|-------|---------|
| `insert_after` | `{ op: 'insert_after', after_step_id: 'S3', step: { type, name, attributes } }` | Inserting a new step after an existing one |
| `update_step` | `{ op: 'update_step', step_id: 'S3', patch: { attributes: { ... } } }` | Mutating an existing step's fields. Patch is deep-merged. MUST NOT include `next`/`parentKey`/`order`. |
| `delete_step` | `{ op: 'delete_step', step_id: 'S3' }` | Removing a step |
| `move` | `{ op: 'move', step_id: 'S3', after_step_id: 'S5' }` | Reordering a step |

**Fallback to `update_workflow`:** Only for whole-workflow rewrites OR when patch ops are insufficient (e.g., target step is on a branched node and you need a splice). `patch_workflow` returns the SAME response shape as `update_workflow` — entity warnings, workflow URL, refreshed actions list — so the post-write flow (Step 6/7) is identical.

**Worked example — outline → step-read → patch:**

> User: "Change the subject line of the confirmation email to 'Welcome aboard!'"
>
> 1. `read_workflow_outline(workflowId)` → returns step map; identify confirmation email as `S3`.
> 2. `read_workflow_step(workflowId, 'S3')` → returns S3's full JSON; confirm it's an email step and capture current attributes.
> 3. `patch_workflow(workflowId, [{ op: 'update_step', step_id: 'S3', patch: { attributes: { subject: 'Welcome aboard!' } } }])` → server merges the patch, rewires pointers if needed, refreshes the cache.

For multi-step edits (e.g., "insert a 1h wait + email between S3 and S4"), pass an ordered ops array:

```js
[
  { op: 'insert_after', after_step_id: 'S3', step: { type: 'wait', name: '1h delay', attributes: { duration: 3600 } } },
  { op: 'insert_after', after_step_id: 'S3', step: { type: 'send_email', name: 'Confirmation', attributes: { /* ... */ } } }
]
```

If you need a chained insert to reference a just-inserted step's ID and the executor doesn't yet support that, split into two sequential `patch_workflow` calls — outline + step-read between them so the new ID is known.

## Step 5: Apply with Discipline

> **Wiring is preserved verbatim.** The `next`, `parent`, `parentKey`, `sibling`, `nodeType`, and `attributes.targetNodeId` fields you submit ride through unchanged — auto-wiring only fills missing `next` fields on non-goto actions. See `references/wiring.md` for branching, multi-path, and GoTo patterns.

When calling `update_workflow`:

- Use exact `type` values from the reference — never approximate or invent actions.
- Calculate wait-step durations relative to the **previous step**, not the workflow start.
- The user never needs to see JSON — handle it internally.
- Preserve fields you are not deliberately changing.

> **Routing reminder.** `update_workflow` is the fallback path. Default to `patch_workflow` (Step 4.7) for any edit expressible as `insert_after` / `update_step` / `delete_step` / `move` on a non-branched chain. Use `update_workflow` only for whole-workflow rewrites or when patch ops can't express the change (typically a splice through a branched node).

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
  `https://app.gohighlevel.com/location/{locationId}/workflow/{workflowId}`
  Replace `{locationId}` and `{workflowId}` with the actual values.
- Any `entityWarnings` from the response, with recommended corrections.

Ask if they want to publish, keep as draft, or make further edits.
