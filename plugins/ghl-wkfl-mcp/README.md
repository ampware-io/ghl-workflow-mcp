# ghl-wkfl-mcp — Claude Code plugin

Thin plugin that wires the `ghl-wkfl-mcp` MCP server into Claude Code. The
server binary itself is distributed separately — download from
https://ampware.io/ghl-wkfl-mcp/install

## Install (two steps)

1. Install the `ghl-wkfl-mcp` binary and put it on your PATH:
   - Download the binary for your platform from
     https://ampware.io/ghl-wkfl-mcp/install
   - `chmod +x ghl-wkfl-mcp-<platform>` and move it onto your PATH (for
     example `/usr/local/bin/ghl-wkfl-mcp`), OR
   - Alternatively: skip this plugin and run
     `./ghl-wkfl-mcp-<platform> setup --install claude-code` to write a
     direct-path config (no PATH setup, no plugin).
2. Install the plugin inside Claude Code:
   ```
   /plugin marketplace add ampware-io/ghl-workflow-mcp
   /plugin install ghl-wkfl-mcp@ghl-workflow-mcp
   ```
3. When prompted for `license_key`, leave blank for free tools or paste your
   license JWT to unlock paid workflow mutations.
4. Run `/reload-plugins` and invoke `test_connection` to verify.

Full docs: https://github.com/ampware-io/ghl-workflow-mcp
Buy a license: https://ampware.io/ghl-wkfl-mcp/buy
