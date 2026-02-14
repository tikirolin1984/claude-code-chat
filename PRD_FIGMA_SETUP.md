# PRD: Figma Integration Setup Guide

**Author:** Claude
**Date:** 2026-02-14
**Status:** Draft
**Version:** 1.0

---

## 1. Overview

### 1.1 Problem Statement

Setting up a working Figma-to-code pipeline with Claude Code requires installing and configuring multiple tools across three ecosystems (Figma, Anthropic, VS Code). The current setup guide has 11 steps (numbered 2–12) but omits critical prerequisites, authentication flows, configuration commands, and verification checkpoints. Users who follow the guide as-is will hit dead ends.

### 1.2 Objective

Produce a complete, linear, copy-paste-friendly setup guide that takes a user from zero to a verified Figma + Claude Code integration in a single session, with no ambiguity about what to run, where to configure, or how to verify each step.

### 1.3 Target Users

| Persona | Description |
|---------|-------------|
| **Designer-Developer** | Figma user who also codes; wants to generate components from designs |
| **Frontend Developer** | Receives Figma handoffs; wants to extract design tokens and layout data via CLI |
| **VS Code Power User** | Uses `claude-code-chat` extension as their primary Claude interface |

### 1.4 Scope

**In scope:**
- End-to-end setup from OS prerequisites through verified Figma MCP connection
- Both the Remote MCP server and Desktop (local) MCP server paths
- Claude Code CLI and Claude Desktop app configuration
- Optional community MCP servers (Console MCP, Design Systems Assistant, Desktop Bridge)
- Verification and troubleshooting

**Out of scope:**
- Figma plugin development
- Custom MCP server authoring
- CI/CD integration
- Team/org-wide deployment

---

## 2. Architecture Context

```
┌─────────────────────────────────────────────────────────────┐
│                       User's Machine                        │
│                                                             │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   VS Code    │    │ Claude Code  │    │   Claude     │  │
│  │   + claude-  │◄──►│    CLI       │    │   Desktop    │  │
│  │   code-chat  │    │              │    │     App      │  │
│  └──────────────┘    └──────┬───────┘    └──────┬───────┘  │
│                             │                    │          │
│                     ┌───────┴────────────────────┘          │
│                     │  MCP Protocol                         │
│                     ▼                                       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │               MCP Server Layer                        │  │
│  │                                                       │  │
│  │  ┌─────────────┐  ┌──────────────┐  ┌────────────┐  │  │
│  │  │ Figma Remote│  │Figma Desktop │  │ Community  │  │  │
│  │  │ MCP Server  │  │ MCP Server   │  │ MCP Servers│  │  │
│  │  │ (hosted)    │  │ (local:3845) │  │ (optional) │  │  │
│  │  └──────┬──────┘  └──────┬───────┘  └────────────┘  │  │
│  └─────────┼────────────────┼───────────────────────────┘  │
│            │                │                               │
└────────────┼────────────────┼───────────────────────────────┘
             │                │
             ▼                ▼
       ┌───────────┐   ┌───────────┐
       │ Figma API │   │  Figma    │
       │ (REST)    │   │  Desktop  │
       │           │   │   App     │
       └───────────┘   └───────────┘
```

### 2.1 Two Server Paths

| | Remote MCP Server | Desktop MCP Server |
|---|---|---|
| **URL** | `https://mcp.figma.com/mcp` | `http://127.0.0.1:3845/mcp` |
| **Auth** | OAuth (browser popup) | None (local) |
| **Requires Figma desktop app** | No | Yes |
| **Selection-based input** | No | Yes |
| **Variable extraction** | Limited | Full |
| **Works headless / SSH** | Yes | No |

Most users should set up **both**. The remote server works anywhere; the desktop server unlocks full capabilities when Figma is open.

---

## 3. Prerequisites

### 3.1 System Requirements

| Requirement | Minimum | Notes |
|-------------|---------|-------|
| **OS** | macOS 12+, Windows 10+ (with or without WSL), Ubuntu 20.04+ | |
| **Node.js** | v18.0.0+ | Required for Claude Code CLI |
| **npm** | v9+ | Ships with Node.js |
| **Browser** | Any modern browser | For Figma OAuth flow |
| **Disk space** | ~500 MB | Node + Claude Code + MCP servers |

### 3.2 Accounts Required

| Account | Purpose | Plan Requirement |
|---------|---------|-----------------|
| **Anthropic** | Claude Code authentication | API key or Pro/Max subscription |
| **Figma** | Design access + MCP token | Dev or Full seat on paid plan recommended (Starter/View/Collab limited to 6 MCP tool calls/month) |

---

## 4. Functional Requirements

### 4.1 Setup Steps — Complete Specification

Each step below is a discrete, verifiable action. Steps are grouped into phases. Within each phase, steps are sequential. Phases themselves are sequential.

---

#### Phase 1: Core Tool Installation

**Step 1 — Install the Figma Desktop App**

| Field | Value |
|-------|-------|
| Action | Download and install Figma desktop app; update to latest version |
| Reference | https://www.figma.com/downloads/ |
| Verification | Open Figma app; confirm version is current via Figma menu > About |
| Required for | Desktop MCP server (Phase 3B) |
| Skippable if | Only using the remote MCP server |

**Step 2 — Install Node.js (v18+)**

| Field | Value |
|-------|-------|
| Action | Install Node.js LTS from official installer or via a version manager (`nvm`, `fnm`) |
| Reference | https://nodejs.org/en/ |
| Verification | `node --version` returns `v18.x.x` or higher |
| Verification | `npm --version` returns `9.x.x` or higher |

**Step 3 — Install Claude Code CLI**

| Field | Value |
|-------|-------|
| Action | `npm install -g @anthropic-ai/claude-code` |
| Reference | https://docs.anthropic.com/en/docs/claude-code |
| Verification | `claude --version` returns a version string |

**Step 4 — Authenticate Claude Code**

| Field | Value |
|-------|-------|
| Action | Run `claude login` and complete the browser-based authentication, **or** set the `ANTHROPIC_API_KEY` environment variable |
| Verification | `claude` opens an interactive session without auth errors |

**Step 5 — Install VS Code + claude-code-chat extension (optional)**

| Field | Value |
|-------|-------|
| Action | Install VS Code (https://code.visualstudio.com/), then install the `claude-code-chat` extension from the marketplace |
| Verification | `Ctrl+Shift+C` (or `Cmd+Shift+C` on Mac) opens the Claude Code Chat panel |
| Skippable if | Using Claude Code CLI directly in the terminal or using Claude Desktop only |

---

#### Phase 2: Figma Authentication

**Step 6 — Create a Figma Personal Access Token**

| Field | Value |
|-------|-------|
| Action | In Figma, go to Account Settings > Personal Access Tokens > Generate new token. Give it a descriptive name (e.g., `claude-mcp`). Copy the token immediately — it is shown only once. |
| Reference | https://help.figma.com/hc/en-us/articles/8085703771159 |
| Scopes | Minimum: `file:read`. Recommended: `file:read`, `dev_resources:read` |
| Verification | Token string is saved securely (password manager, env var, etc.) |

**Step 7 — Store the Figma token**

| Field | Value |
|-------|-------|
| Action | Export the token as an environment variable so MCP servers can read it. Add to your shell profile (`~/.zshrc`, `~/.bashrc`, or equivalent): `export FIGMA_ACCESS_TOKEN="your-token-here"` Then reload the shell: `source ~/.zshrc` |
| Verification | `echo $FIGMA_ACCESS_TOKEN` prints the token |
| Notes | The official Figma remote MCP server uses OAuth instead of this token. This token is needed by community/third-party MCP servers (Console MCP, Design Systems Assistant, etc.) |

---

#### Phase 3A: Remote MCP Server (no desktop app required)

**Step 8 — Add the Figma remote MCP server to Claude Code**

| Field | Value |
|-------|-------|
| Action | `claude mcp add --transport http figma-remote https://mcp.figma.com/mcp` |
| Verification | `claude mcp list` shows `figma-remote` |

**Step 9 — Complete OAuth authentication**

| Field | Value |
|-------|-------|
| Action | Start a Claude Code session (`claude`). The first time a Figma tool is invoked, a browser window opens for OAuth. Select "Authenticate", click "Allow Access". Return to the terminal. |
| Verification | Terminal shows "Authentication successful. Connected to figma." |
| Verification | `/mcp` command in Claude Code shows `figma-remote` as connected |

---

#### Phase 3B: Desktop MCP Server (full feature access)

**Step 10 — Open a Figma Design file and enable Dev Mode**

| Field | Value |
|-------|-------|
| Action | Open the Figma desktop app. Open or create a Design file. Toggle Dev Mode using `Shift+D` or the toolbar toggle at the bottom. |
| Verification | The inspect panel on the right shows Dev Mode tools |

**Step 11 — Enable the desktop MCP server**

| Field | Value |
|-------|-------|
| Action | In the Dev Mode inspect panel, locate the "MCP server" section. Click "Enable desktop MCP server". |
| Verification | The panel confirms the server is running |
| Verification | `curl -s http://127.0.0.1:3845/mcp` returns a response (not "connection refused") |

**Step 12 — Register the desktop MCP server in Claude Code**

| Field | Value |
|-------|-------|
| Action | `claude mcp add --transport http figma-desktop http://127.0.0.1:3845/mcp` |
| Verification | `claude mcp list` shows `figma-desktop` |
| Verification | `/mcp` command in Claude Code shows `figma-desktop` as connected |

---

#### Phase 4: Optional Community MCP Servers

These extend the integration with additional capabilities. Each is independent.

**Step 13 — Install Figma Console MCP (local version)**

| Field | Value |
|-------|-------|
| Action | Clone the repo and follow its README for installation. Register with `claude mcp add`. |
| Reference | GitHub repository (see original guide step 7) |
| Requires | `FIGMA_ACCESS_TOKEN` environment variable |
| Verification | `claude mcp list` shows the server |

**Step 14 — Install Design Systems Assistant MCP**

| Field | Value |
|-------|-------|
| Action | Clone the repo and follow its README for installation. Register with `claude mcp add`. |
| Reference | GitHub repository (see original guide step 8) |
| Requires | `FIGMA_ACCESS_TOKEN` environment variable |
| Verification | `claude mcp list` shows the server |

**Step 15 — Install Desktop Bridge Plugin**

| Field | Value |
|-------|-------|
| Action | Install the Figma plugin from the community plugins page or the referenced repo. |
| Reference | GitHub repository (see original guide step 9) |
| Verification | Plugin appears in Figma's plugin menu |

---

#### Phase 5: Claude Desktop App Configuration (if applicable)

**Step 16 — Identify the Claude Desktop config file**

| OS | Path |
|----|------|
| macOS | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Windows | `%APPDATA%\Claude\claude_desktop_config.json` |
| Linux | `~/.config/Claude/claude_desktop_config.json` |

**Step 17 — Add MCP servers to Claude Desktop config**

| Field | Value |
|-------|-------|
| Action | Open the config file and add the MCP server entries. Example: |

```json
{
  "mcpServers": {
    "figma-remote": {
      "url": "https://mcp.figma.com/mcp"
    },
    "figma-desktop": {
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

| Verification | File is valid JSON (no trailing commas, proper nesting) |

**Step 18 — Quit and relaunch Claude Desktop**

| Field | Value |
|-------|-------|
| Action | Fully quit Claude Desktop (not just close the window — use Cmd+Q / Alt+F4 or the system tray). Relaunch the app. |
| Verification | Claude Desktop opens without errors |

---

#### Phase 6: Final Verification

**Step 19 — Verify all MCP connections**

| Field | Value |
|-------|-------|
| Action (Claude Code CLI) | Run `claude mcp list` and confirm all expected servers are listed |
| Action (Claude Code CLI) | Start a session and run `/mcp` to see connection status |
| Action (Claude Desktop) | Open settings and check the MCP server status panel |
| Expected result | All configured servers show as "connected" |

**Step 20 — Run a functional test**

| Field | Value |
|-------|-------|
| Action | In Claude Code or Claude Desktop, send a prompt with a real Figma file URL: `"Here is my design: https://www.figma.com/design/FILE_KEY/FILE_NAME. Describe the layout and generate a React component for the main frame."` |
| Expected result | Claude fetches design metadata from Figma, describes the layout, and generates code |
| Failure modes | If Claude says it can't access Figma or doesn't recognize the MCP tools, see Troubleshooting |

---

## 5. Non-Functional Requirements

### 5.1 Performance

| Metric | Target |
|--------|--------|
| Total setup time (experienced user) | < 15 minutes |
| Total setup time (first-time user) | < 30 minutes |
| MCP server response latency | < 3 seconds per tool call |

### 5.2 Reliability

- The desktop MCP server must remain running as long as Figma desktop is open with Dev Mode active.
- The remote MCP server OAuth token should persist across Claude Code sessions (re-auth should not be needed each time).
- If the desktop MCP server stops (Figma closed), Claude Code should gracefully fall back to the remote server if both are configured.

### 5.3 Security

- Figma Personal Access Tokens must never be committed to version control.
- `.env` files containing tokens must be in `.gitignore`.
- OAuth tokens are managed by the Figma remote server and do not touch the local filesystem.
- The desktop MCP server binds to `127.0.0.1` only (not exposed to the network).

---

## 6. Configuration File Reference

### 6.1 Where MCP servers are registered

| Scope | File | Use when |
|-------|------|----------|
| **Project** | `.mcp.json` in repo root | You want the config checked into version control for the team |
| **User (Claude Code)** | `~/.claude/settings.json` | You want the servers available in all projects |
| **Claude Desktop** | See Phase 5 paths above | You're using the Claude Desktop GUI app |

### 6.2 Example `.mcp.json` (project-level)

```json
{
  "mcpServers": {
    "figma-remote": {
      "url": "https://mcp.figma.com/mcp"
    },
    "figma-desktop": {
      "url": "http://127.0.0.1:3845/mcp"
    }
  }
}
```

### 6.3 Environment Variables

| Variable | Required by | Purpose |
|----------|-------------|---------|
| `FIGMA_ACCESS_TOKEN` | Community MCP servers | Figma REST API authentication |
| `ANTHROPIC_API_KEY` | Claude Code (if not using `claude login`) | Anthropic API authentication |

---

## 7. Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `claude mcp list` shows no servers | Servers were never registered | Run the `claude mcp add` commands from Phase 3 |
| `/mcp` shows server as "disconnected" | Server process not running | For desktop: ensure Figma is open with Dev Mode enabled. For remote: re-run OAuth |
| OAuth popup doesn't appear | Browser popup blocked | Allow popups for the Figma auth domain |
| "Authentication failed" on remote server | Expired or revoked OAuth token | Remove and re-add the remote server: `claude mcp remove figma-remote` then re-add |
| `curl http://127.0.0.1:3845/mcp` → "connection refused" | Figma desktop MCP server not enabled | Open Figma file > Dev Mode > enable desktop MCP server |
| Community MCP server errors | `FIGMA_ACCESS_TOKEN` not set | Verify with `echo $FIGMA_ACCESS_TOKEN`; reload shell if recently added |
| "6 tool calls per month" limit hit | Figma Starter plan or View/Collab seat | Upgrade to a Dev or Full seat on a paid Figma plan |
| Claude Desktop doesn't see MCP servers | Config file not saved or not valid JSON | Validate JSON; fully quit and relaunch Claude Desktop |
| Node.js version errors | Node < 18 | `node --version`; upgrade via `nvm install --lts` |

---

## 8. Mapping: Original Guide to PRD

| Original Step | PRD Step(s) | What was missing |
|---------------|-------------|-----------------|
| *(none)* | 1 | Install Figma Desktop App |
| 2. Install Node | 2 | Verification command |
| 3. Install Claude Code | 3, 4 | Authentication was missing entirely |
| *(none)* | 5 | VS Code + extension installation |
| 4. Create Figma token | 6, 7 | Token was created but never stored/configured |
| 5. Enable Figma Dev Mode MCP | 10, 11 | No sub-steps for opening file, toggling Dev Mode, clicking enable |
| 6. Configure Figma MCP server | 8, 9, 12 | No actual commands; remote vs. desktop not distinguished |
| 7. Install Figma Console MCP | 13 | No verification step |
| 8. Install Design Systems Assistant MCP | 14 | No verification step |
| 9. Install Desktop Bridge Plugin | 15 | No verification step |
| 10. Install Figma MCP in Claude Desktop | 16, 17 | Config file path and JSON content not provided |
| 11. Quit and relaunch Claude Desktop | 18 | Minor — needs "fully quit" clarification |
| 12. Enter "check figma status" | 19, 20 | Ambiguous; replaced with concrete verification commands and a functional test |

---

## 9. Success Criteria

| Criteria | Measurement |
|----------|-------------|
| A user following the PRD from step 1 can reach a verified connection | Manual QA walkthrough on macOS + Windows |
| No step requires information not provided in a previous step | Review by someone unfamiliar with the tools |
| Every step has a concrete verification action | Audit: each step row contains a "Verification" field |
| Troubleshooting table covers the top 8 failure modes | Cross-reference with Figma community forum + GitHub issues |

---

## 10. Open Questions

| # | Question | Impact | Status |
|---|----------|--------|--------|
| 1 | Which community MCP server repos (steps 7–9 in original guide) are being used? The original guide truncated the GitHub URLs. | Cannot write exact install commands for Phase 4 | Blocked — need full URLs |
| 2 | Should the guide recommend Remote, Desktop, or Both as the default path? | Affects step ordering and "skippable" annotations | Suggest: Both, with Remote as the minimum viable path |
| 3 | Is the `claude plugin install figma@claude-plugins-official` plugin a viable replacement for manual `claude mcp add` commands? | Could simplify Phase 3 to a single command | Needs testing |
| 4 | Does the team want this guide embedded in the README, as a separate doc, or as an in-extension walkthrough? | Affects format and delivery | Pending decision |
