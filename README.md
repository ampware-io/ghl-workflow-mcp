# GHL Workflow MCP

**Let AI assistants build, edit, and audit your GoHighLevel workflows — using the browser you already have signed in.**

No official API, no OAuth dance, no SaaS subscription. The MCP server connects to your local Chrome session and lets Claude (Desktop, Code, or Cursor) read, create, modify, and clone workflows through the same endpoints GHL's workflow builder uses internally.

Version: `v2.0.0-beta.1` · [Latest release](https://github.com/ampware-io/ghl-workflow-mcp/releases/tag/v2.0.0-beta.1)

---

_Screenshots and a short demo GIF are in the works — check back after the next release. In the meantime, the install flow is documented end-to-end below with exact commands and expected output._

---

## Easy install — Claude Desktop (start here)

No terminal required. Download the installer, run it once, restart Desktop.

- **Windows:** Download `ghl-wkfl-mcp-windows-x64.exe` from [the latest release](https://github.com/ampware-io/ghl-workflow-mcp/releases/tag/v2.0.0-beta.1). Double-click it. A small window asks which clients to set up — tick "Claude Desktop" and confirm.
- **macOS (Apple Silicon):** Download `ghl-wkfl-mcp-darwin-arm64-installer.zip` from [the latest release](https://github.com/ampware-io/ghl-workflow-mcp/releases/tag/v2.0.0-beta.1). Unzip it, then double-click `Install.command`. If macOS warns about an "unidentified developer", right-click `Install.command` → Open → Allow.
- **macOS (Intel):** Same steps as above but use `ghl-wkfl-mcp-darwin-x64-installer.zip`.

After the installer finishes, **fully quit and restart** Claude Desktop (a reload is not enough to pick up a new MCP server). The installer also extracts the 5 workflow skills into `~/.claude/skills/ghl-wkfl/` automatically.

<details>
<summary>Linux install</summary>

```
chmod +x ghl-wkfl-mcp-linux-x64
./ghl-wkfl-mcp-linux-x64
```

Run the binary once — it auto-detects the Claude Desktop / Claude Code / Cursor config on your system and writes the right entries.

</details>

## Claude Code

Claude Code picks up the MCP server and skills in two steps. The marketplace install is the recommended path.

**Step 1 — put the binary on PATH as `ghl-wkfl-mcp`:**

```
# macOS / Linux example
chmod +x ghl-wkfl-mcp-darwin-arm64
sudo mv ghl-wkfl-mcp-darwin-arm64 /usr/local/bin/ghl-wkfl-mcp
ghl-wkfl-mcp --version    # sanity check
```

On Windows, rename `ghl-wkfl-mcp-windows-x64.exe` to `ghl-wkfl-mcp.exe` and place it on `%PATH%` (e.g. `%LOCALAPPDATA%\Programs\ghl-wkfl-mcp\`). If adding to PATH is not an option, use the easy installer above — it writes absolute paths and does not need PATH.

**Step 2 — install the plugin from the marketplace:**

```
/plugin marketplace add ampware-io/ghl-workflow-mcp
/plugin install ghl-wkfl-mcp@ghl-workflow-mcp
```

Claude Code prompts for `license_key` during install — leave blank to use the free tools, or paste your JWT to unlock the 6 paid tools. You can also activate later via the `license_activate` MCP tool.

Run `/reload-plugins` (or restart Claude Code), then invoke `test_connection` to confirm the server is live.

## Cursor

1. Download the platform binary from [Releases](https://github.com/ampware-io/ghl-workflow-mcp/releases/tag/v2.0.0-beta.1).
2. Run the installer once (same binary as Claude Desktop): `./ghl-wkfl-mcp setup --install cursor`.
3. Restart Cursor.
4. Open Cursor's MCP panel — you should see `ghl-wkfl-mcp` listed. Skills auto-extract to `~/.claude/skills/ghl-wkfl/`.

The installer writes the Cursor MCP config at the OS-appropriate location (macOS: `~/Library/Application Support/Cursor/User/globalStorage/mcp.json`; Windows: `%APPDATA%\Cursor\User\globalStorage\mcp.json`). If you need to change it later, re-run `ghl-wkfl-mcp setup --install cursor`.

## Skills

Skills are Claude's first-class guidance format — short, structured markdown files that teach Claude how to handle a specific task. GHL Workflow MCP ships five skills that tell Claude exactly how to plan, build, edit, explain, and audit GoHighLevel workflows against a live GHL account.

| Skill | Say something like... |
|-------|------------------------|
| `ghl-create-workflow` | "Create a GHL workflow that tags new leads from Form X." |
| `ghl-plan-workflow` | "Help me plan a workflow that reminds customers about abandoned carts." |
| `ghl-edit-workflow` | "Update the trigger on my Welcome workflow to include Form Y." |
| `ghl-explain-workflow` | "What does workflow `abc-123` actually do?" |
| `ghl-audit-workflow` | "Check this workflow for broken references before I publish." |

The installer extracts these skills to `~/.claude/skills/ghl-wkfl/` automatically. Claude picks the right skill based on what you ask — no manual selection needed.

Manual install paths (raw curl, git clone, drag-and-drop `.skill` archive) are documented at [skills/README.md](skills/README.md).

## Free vs Paid

**Free forever** (21 tools — reads, audit, folder / tag listings, connection tools, license management):

- Inspect any workflow (`list_workflows`, `get_workflow`)
- Audit a workflow for broken references (`audit_workflow`)
- List tags, users, pipelines, custom fields, custom values, business fields, associations
- Manage folders (`list_folders`, `create_folder`, `delete_folder`, `rename_folder`, `move_workflow`)
- Connection tools (`launch_chrome`, `reconnect`, `test_connection`)
- License tools (`license_activate`, `license_deactivate`, `license_status`)

**Paid** (6 workflow-mutation tools):

- `create_workflow` — build a new workflow from JSON
- `update_workflow` — modify an existing workflow
- `delete_workflow` — delete a workflow
- `publish_workflow` — publish (activate) or unpublish a workflow
- `clone_workflow` — duplicate a workflow in the same location
- `create_tag` — create a new GHL tag

One-time purchase, no subscription:

**→ [Buy a license — https://ampware.io/ghl-wkfl-mcp/buy](https://ampware.io/ghl-wkfl-mcp/buy)**

After purchase you receive a JWT by email. Activate in any of three ways:

1. Call the `license_activate` MCP tool from your AI assistant with the JWT.
2. Paste into your Claude Code plugin's `license_key` userConfig field at install time (routed via `env.GHL_MCP_LICENSE_KEY` at launch).
3. Set the `GHL_MCP_LICENSE_KEY` environment variable before launching Claude.

For headless / scripted setups, the CLI supports: `ghl-wkfl-mcp setup --license <jwt>`.

## Tools reference

27 tools total. Paid tools are marked **Paid** — all others are free.

### Read tools (12, all free)

| Tool | Free / Paid | What it does |
|------|-------------|--------------|
| `list_workflows` | Free | List every workflow in the current GHL location |
| `get_workflow` | Free | Fetch the full structure of one workflow (triggers, actions, branches) |
| `audit_workflow` | Free | Scan a workflow for broken entity references (pipelines, stages, users, tags, custom fields) |
| `list_folders` | Free | List workflow folders |
| `list_tags` | Free | List all tags in the current location |
| `list_users` | Free | List users / team members |
| `list_pipelines` | Free | List opportunity pipelines and their stages |
| `list_custom_fields` | Free | List custom fields on contacts/opportunities |
| `list_custom_values` | Free | List custom values (template strings) |
| `list_associations` | Free | List custom object associations |
| `list_business_fields` | Free | List business-object fields |
| `test_connection` | Free | Verify Chrome is reachable and GHL is signed in |

### Workflow-mutation tools (6, paid)

| Tool | Free / Paid | What it does |
|------|-------------|--------------|
| `create_workflow` | **Paid** | Build a new workflow from JSON |
| `update_workflow` | **Paid** | Modify an existing workflow's triggers / actions |
| `delete_workflow` | **Paid** | Delete a workflow |
| `publish_workflow` | **Paid** | Publish (activate) or unpublish a workflow |
| `clone_workflow` | **Paid** | Duplicate a workflow in the same location |
| `create_tag` | **Paid** | Create a new GHL tag |

### Folder + organization tools (4, free)

| Tool | Free / Paid | What it does |
|------|-------------|--------------|
| `create_folder` | Free | Create a new workflow folder |
| `delete_folder` | Free | Delete an empty folder |
| `rename_folder` | Free | Rename a folder |
| `move_workflow` | Free | Move a workflow into a different folder |

### Connection tools (2 unique, free)

| Tool | Free / Paid | What it does |
|------|-------------|--------------|
| `launch_chrome` | Free | Launch Chrome with remote debugging on port 9222 |
| `reconnect` | Free | Re-establish CDP connection after Chrome restart |

(`test_connection` is also listed under Read tools above.)

### License tools (3, free)

| Tool | Free / Paid | What it does |
|------|-------------|--------------|
| `license_activate` | Free | Install a license JWT (persisted to a 0o600 file or read from `GHL_MCP_LICENSE_KEY`) |
| `license_deactivate` | Free | Remove the stored license |
| `license_status` | Free | Show active license info (sanitized — no full JWT) |

## Advanced paths

- **CLI setup:** `./ghl-wkfl-mcp setup --install all` writes MCP config for every detected client in one go (non-interactive, safe to re-run). Individual targets: `--install claude-desktop`, `--install claude-code`, `--install cursor`.
- **Re-extract skills only:** `./ghl-wkfl-mcp setup --install-skills` refreshes `~/.claude/skills/ghl-wkfl/` from the binary-embedded SKILL.md files (useful after a manual delete).
- **Manual plugin install:** Download `ghl-wkfl-mcp.plugin.zip` from [Releases](https://github.com/ampware-io/ghl-workflow-mcp/releases/tag/v2.0.0-beta.1), unzip, and drop the `ghl-wkfl-mcp/` folder into `~/.claude/plugins/`. Then restart Claude Code or Desktop.
- **Build from source:** Source is maintained in a private repository; releases are published here with pre-built binaries. Contact [ampware.io](https://ampware.io) for build access.

## Usage

After install, launch Chrome with remote debugging and sign in to GHL:

```
google-chrome --remote-debugging-port=9222
```

(Mac users: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222`. Windows users: `"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222`.)

Then ask Claude something like _"List my GHL workflows"_ or _"Create a workflow that tags new leads from Form X."_ Claude picks the right skill automatically — no `/slash` command or tool selection required.

## License

The MCP server itself is free; the 6 workflow-mutation tools require a license JWT.

**Purchase a license: [https://ampware.io/ghl-wkfl-mcp/buy](https://ampware.io/ghl-wkfl-mcp/buy)** — one-time purchase, no subscription.

---

Built by [ampware.io](https://ampware.io). Releases and plugin distribution: [github.com/ampware-io/ghl-workflow-mcp](https://github.com/ampware-io/ghl-workflow-mcp).
