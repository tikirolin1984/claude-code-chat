# Figma Integration Setup — Missing Steps Analysis

## Original Guide (Steps 2–12)

| # | Step |
|---|------|
| 2 | Install Node |
| 3 | Install Claude Code |
| 4 | Create Figma token |
| 5 | Enable Figma Dev Mode MCP |
| 6 | Configure Figma MCP server |
| 7 | Install Figma Console MCP (local version) |
| 8 | Install Design Systems Assistant MCP |
| 9 | Install Desktop Bridge Plugin |
| 10 | Install Figma MCP server in Claude Desktop |
| 11 | Quit and relaunch Claude Desktop |
| 12 | Enter "check figma status" |

---

## Missing Steps

### A. Prerequisites (before Step 2)

1. **Step 1 is absent.** The guide starts at step 2. A natural step 1 would be: **Install the Figma Desktop App** (required for the local/desktop MCP server). Update it to the latest version.

2. **No IDE / editor installation.** If using this VS Code extension (`claude-code-chat`), VS Code must be installed first.

3. **No `claude-code-chat` extension installation.** If the intended workflow uses this extension as the chat interface, installing it from the VS Code marketplace should be an explicit step.

### B. Authentication & Token Configuration (after Step 4)

4. **Token is created but never stored.** Step 4 says "Create Figma token" but there is no step to **configure the token as an environment variable** (`FIGMA_ACCESS_TOKEN`) or store it in a config file. Community/third-party MCP servers require this. The official remote MCP server uses OAuth instead, which is a different flow entirely.

5. **No Claude Code authentication.** Claude Code itself requires authentication (`claude login` or setting an `ANTHROPIC_API_KEY`). This is never mentioned.

6. **No Figma OAuth flow for the remote MCP server.** If using Figma's official remote server (`https://mcp.figma.com/mcp`), you must complete an OAuth authentication step after adding the server. The guide doesn't mention this.

### C. Figma Desktop App Configuration (around Steps 5–6)

7. **No step to open a Figma Design file.** The desktop MCP server requires you to have a Figma Design file open before enabling the server.

8. **No step to toggle Dev Mode.** You must switch to Dev Mode (`Shift+D`) in the Figma desktop app before the MCP server section becomes visible in the inspect panel.

9. **No step to click "Enable desktop MCP server"** in the Figma inspect panel. This is the actual action that starts the local server at `http://127.0.0.1:3845/mcp`.

### D. MCP Server Registration in Claude Code (between Steps 6 and 10)

10. **No `claude mcp add` commands.** The guide says "configure" and "install" various MCP servers but never shows the actual CLI commands. For example:
    - Remote: `claude mcp add --transport http figma https://mcp.figma.com/mcp`
    - Desktop: `claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp`

11. **No mention of the Claude Code Figma plugin.** Figma provides a streamlined plugin: `claude plugin install figma@claude-plugins-official`. This bundles both remote and desktop server configs plus agent skills. It may replace several manual steps.

12. **No mention of where config files live.** MCP servers are registered in either:
    - Project-level: `.mcp.json` in the project root
    - User-level: `~/.claude/settings.json`
    - Claude Desktop: `~/Library/Application Support/Claude/claude_desktop_config.json` (macOS) or `%APPDATA%\Claude\claude_desktop_config.json` (Windows)

    The guide should specify which file to edit.

### E. Verification & Intermediate Checks

13. **No intermediate verification steps.** The only check is at step 12 ("check figma status"). There should be verification after key milestones:
    - After adding each MCP server: `claude mcp list` to confirm it registered
    - After OAuth: confirm "Authentication successful. Connected to figma."
    - After enabling desktop server: confirm the server is running at `http://127.0.0.1:3845/mcp`
    - `/mcp` command in Claude Code to see connection status of all servers

14. **"check figma status" is ambiguous.** It's unclear whether this is a Claude Code slash command, a prompt to type into the chat, or a CLI command. The standard way is `/mcp` in Claude Code.

### F. Missing Conceptual Clarity

15. **Remote vs. Local server distinction is never explained.** The guide references both but doesn't explain when to use which:
    - **Remote server** (`mcp.figma.com`): No desktop app needed, uses OAuth, works anywhere, limited to REST API capabilities
    - **Local/Desktop server** (`127.0.0.1:3845`): Requires Figma desktop app running, offers full tool access including selection-based input and variable extraction

16. **Claude Code vs. Claude Desktop confusion.** Steps 3 and 10 reference both "Claude Code" (the CLI) and "Claude Desktop" (the GUI app). These are different products with different config files. The guide should clarify which is being used or acknowledge that both are being set up.

17. **No mention of plan/seat requirements.** Figma MCP has rate limits based on your plan:
    - Starter plan / View / Collab seats: up to 6 tool calls per month
    - Dev or Full seat on paid plans: per-minute rate limits (Tier 1 REST API limits)

### G. Post-Setup

18. **No test workflow.** After verifying status, the guide should include a practical test, e.g., "Paste a Figma link and ask Claude to generate a component from it."

19. **No troubleshooting section.** Common issues include:
    - Desktop MCP server not starting (Figma app not updated, Dev Mode not enabled)
    - OAuth failures (browser popup blocked)
    - Token permission scope too narrow
    - Node.js version incompatibility
    - MCP server showing as "disconnected" (restart needed)

---

## Recommended Revised Step Order

For a complete guide, the steps should roughly follow this order:

1. Install the Figma Desktop App (update to latest)
2. Install Node.js (v18+)
3. Install Claude Code (`npm install -g @anthropic-ai/claude-code`)
4. Authenticate Claude Code (`claude login`)
5. *(Optional)* Install VS Code + `claude-code-chat` extension
6. Create a Figma Personal Access Token (for third-party servers)
7. **Choose your server approach: Remote, Desktop, or Both**
8. *If Remote:* Add remote MCP server (`claude mcp add ...`) and complete OAuth
9. *If Desktop:* Open Figma file → toggle Dev Mode → enable desktop MCP server
10. *If Desktop:* Add desktop MCP server to Claude Code or Claude Desktop config
11. Verify MCP connections (`claude mcp list` / `/mcp`)
12. *(Optional)* Install Figma Console MCP (local version)
13. *(Optional)* Install Design Systems Assistant MCP
14. *(Optional)* Install Desktop Bridge Plugin
15. Configure MCP servers in Claude Desktop (`claude_desktop_config.json`) if using Claude Desktop
16. Restart Claude Desktop (if applicable)
17. Final verification: paste a Figma link and ask Claude to describe the design
