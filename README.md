# GHL Workflow MCP — Claude Code plugin

**Manage GoHighLevel workflows from Claude Code** — read, create, audit, edit, organise, and clone — using your existing Chrome session. No GHL API key, no OAuth, no SaaS subscription.

> This repo hosts the **Claude Code plugin marketplace listing only**. The MCP server binary is distributed separately and must be installed first. For Claude Desktop, Cursor, or a standalone CLI install, see **https://ampware.io/ghl-wkfl-mcp/install**.

---

## Install (Claude Code)

### Step 1 — install the binary

Download the platform binary from **https://ampware.io/ghl-wkfl-mcp/install** and put it on PATH as `ghl-wkfl-mcp`:

**macOS / Linux:**

```
chmod +x ghl-wkfl-mcp-<platform>
sudo mv ghl-wkfl-mcp-<platform> /usr/local/bin/ghl-wkfl-mcp
ghl-wkfl-mcp --version    # sanity check
```

**Windows:** rename `ghl-wkfl-mcp-windows-x64.exe` to `ghl-wkfl-mcp.exe` and place it on `%PATH%` (e.g. `%LOCALAPPDATA%\Programs\ghl-wkfl-mcp\`).

The plugin's `.mcp.json` looks the binary up by bare name, so if `ghl-wkfl-mcp` (or `ghl-wkfl-mcp.exe` on Windows) isn't resolvable on PATH the plugin will install but the MCP server won't connect.

### Step 2 — install the plugin

```
/plugin marketplace add ampware-io/ghl-workflow-mcp
/plugin install ghl-wkfl-mcp@ghl-workflow-mcp
```

Claude Code prompts for `license_key` — leave blank to use the free tools, or paste your license JWT to unlock workflow mutations. You can also activate later via the `license_activate` MCP tool.

### Step 3 — verify

```
/reload-plugins
```

Then ask Claude to call `test_connection`. If Chrome isn't running with remote debugging on, the tool will tell you what to do.

---

## What's included

- **MCP server** — 27 tools (21 free, 6 paid)
- **Workflow skills** — short markdown files Claude uses to handle GHL tasks. Auto-extracted to `~/.claude/skills/ghl-wkfl/` by the plugin. Current skills: `ghl-audit-workflow`, `ghl-create-workflow`, `ghl-edit-workflow`, `ghl-explain-workflow`, `ghl-plan-workflow`. Claude picks the right skill based on what you ask.

## Free vs Paid

**Free forever** (21 tools): inspect any workflow (`list_workflows`, `get_workflow`, `audit_workflow`), browse entities (`list_pipelines`, `list_users`, `list_custom_fields`, `list_custom_values`, `list_tags`, `list_business_fields`, `list_associations`, `list_folders`), manage folders (`create_folder`, `delete_folder`, `rename_folder`, `move_workflow`), connect to Chrome (`launch_chrome`, `reconnect`, `test_connection`), manage your license (`license_activate`, `license_deactivate`, `license_status`).

**Paid** (6 workflow-mutation tools): `create_workflow`, `update_workflow`, `delete_workflow`, `publish_workflow`, `clone_workflow`, `create_tag`.

**[Buy a license — ampware.io/ghl-wkfl-mcp/buy](https://ampware.io/ghl-wkfl-mcp/buy)** — one-time purchase, no subscription.

After purchase, you receive a JWT by email. Activate it any of four ways:

1. Paste into the plugin's `license_key` field at install time.
2. Call the `license_activate` MCP tool from Claude Code with the JWT.
3. Set the `GHL_MCP_LICENSE_KEY` environment variable before launching Claude Code.
4. CLI: `ghl-wkfl-mcp setup --license <jwt>` for headless / scripted setups.

## Usage

Launch Chrome with remote debugging on, then ask Claude what you want:

```
google-chrome --remote-debugging-port=9222
```

(macOS: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222`. Windows: `"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222`.)

Sign in to GHL in that Chrome window, then ask Claude things like:

- _"List my GHL workflows."_
- _"Create a workflow that tags new leads from Form X."_
- _"Audit workflow `abc-123` for broken references before I publish."_
- _"Explain what workflow `xyz-456` does in plain English."_

Claude picks the right skill automatically. No `/slash` commands or tool selection required.

## Other clients

This plugin is for **Claude Code only**. For Claude Desktop, Cursor, or a standalone CLI install, the binary itself handles setup:

- **Claude Desktop:** download the binary + skills zip from [ampware.io/ghl-wkfl-mcp/install](https://ampware.io/ghl-wkfl-mcp/install), run the installer, then upload each `.skill` file via Claude Desktop's *Settings → Capabilities → Skills → Upload*.
- **Cursor:** download the binary, run `ghl-wkfl-mcp setup --install cursor`, restart Cursor.
- **Standalone:** `ghl-wkfl-mcp setup --install all` writes MCP config for every detected client.

## Manual plugin install

If `/plugin marketplace add` doesn't work for any reason (offline install, custom config), download `ghl-wkfl-mcp.plugin.zip` from the install page, unzip it, and drop the `ghl-wkfl-mcp/` folder into `~/.claude/plugins/`. Restart Claude Code.

## Support

- **Issues / questions:** [github.com/ampware-io/ghl-workflow-mcp/issues](https://github.com/ampware-io/ghl-workflow-mcp/issues)
- **Source:** maintained in a private repository; releases distributed via this repo's plugin marketplace listing and the public install page.
- **Built by [ampware.io](https://ampware.io)**.
