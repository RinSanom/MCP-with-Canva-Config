---
layout: default
title: Setup Guide
nav_order: 2
description: "Install Gemini CLI, register the Canva MCP extension, and complete OAuth authentication."
permalink: /setup/
---

# Setup Guide

{: .no_toc }

Follow these steps to connect Gemini to Canva via MCP in under 10 minutes.
{: .fs-5 .fw-300 }

## Table of Contents

{: .no_toc .text-delta }

1. TOC
   {:toc}

---

## Step 1 — Install Gemini CLI

Gemini CLI is the command-line client that acts as your MCP host. It discovers, registers, and communicates with MCP servers such as Canva's.

### Option A · npm (recommended)

```bash
# Requires Node.js ≥ 18
npm install -g @google/gemini-cli
```

Verify the installation:

```bash
gemini --version
# Expected output: gemini/x.y.z  node/v18+
```

### Option B · Homebrew (macOS / Linux)

```bash
brew install google-gemini-cli
```

### Option C · pip (Python wrapper)

```bash
# Requires Python ≥ 3.10
pip install gemini-cli
```

> **Note:** The npm package is the canonical release. The Homebrew and pip variants are community-maintained wrappers and may lag behind the official version.

### Authenticate with Your Google Account

```bash
gemini auth login
```

A browser window opens. Sign in with the Google account linked to your Gemini API key or Google AI Studio project.

```
✔  Signed in as you@example.com
✔  API key stored in ~/.config/gemini/credentials.json
```

---

## Step 2 — Install the Canva MCP Extension

Gemini CLI uses **extensions** to register MCP servers. The Canva extension bundles the server URL, the OAuth configuration, and the tool manifest.

```bash
gemini extensions install canva
```

The CLI fetches the extension manifest from the Gemini Extensions Registry and installs it:

```
Fetching extension manifest ...  ✔
Validating schema ...            ✔
Installing canva@1.x.x ...       ✔

Extension "canva" installed successfully.
Run `gemini extensions list` to confirm.
```

### Verify the Extension

```bash
gemini extensions list
```

Expected output:

```
Installed extensions:
  canva    1.x.x    MCP server: https://mcp.canva.com/v1    [auth: oauth2]
```

### Behind the Scenes

The `extensions install` command does three things:

1. **Downloads** the extension's `manifest.json` — a self-describing document that lists every available tool, its input schema, and required OAuth scopes.
2. **Registers** the MCP server endpoint in your local Gemini CLI config (`~/.config/gemini/extensions/canva/`).
3. **Schedules** an OAuth authorisation flow the first time a Canva tool is invoked.

---

## Step 3 — OAuth Authentication with Canva

The Canva MCP integration uses **OAuth 2.0 with PKCE** (Proof Key for Code Exchange) — an industry-standard flow that never exposes your Canva credentials to Gemini or to this guide.

### Trigger the Auth Flow

The flow starts automatically the first time you use any Canva tool:

```bash
gemini chat
```

```
You: Search my Canva designs for "Q1 Report"

Gemini: 🔐 Canva requires authorisation before I can access your designs.
        Opening browser for sign-in...
```

Or trigger it explicitly:

```bash
gemini extensions auth canva
```

### Authorisation Steps

```
1. Browser opens  →  https://www.canva.com/oauth/authorize?...
2. Log in with your Canva credentials (if not already signed in).
3. Review the requested permissions:
      ✔  Read your designs
      ✔  Create and edit designs on your behalf
      ✔  Export designs
4. Click "Allow Access."
5. Canva redirects to localhost — Gemini CLI captures the auth code.
6. CLI exchanges the code for an access token + refresh token.
```

### What Permissions Are Requested?

| OAuth Scope            | Purpose                                      |
| ---------------------- | -------------------------------------------- |
| `design:content:read`  | List and search your existing designs        |
| `design:content:write` | Create, duplicate, and edit designs          |
| `design:meta:read`     | Read design metadata (name, size, brand kit) |
| `asset:read`           | Access your uploaded images and media        |
| `asset:write`          | Upload new assets to your Canva library      |
| `export:read`          | Download / export finished designs           |

> **Security note:** Tokens are stored locally in `~/.config/gemini/extensions/canva/token.json` and are never sent to Google's servers. Canva tokens expire after **24 hours**; the CLI refreshes them automatically using the stored refresh token.

### Confirm Successful Authentication

```bash
gemini extensions status canva
```

```
canva extension status:
  Version:        1.x.x
  MCP Server:     https://mcp.canva.com/v1   [reachable ✔]
  Authentication: OAuth2 · token valid · expires in 23h 42m
  Tools available: 21
```

---

## Step 4 — Test the Connection

Run a quick sanity check inside an interactive chat session:

```bash
gemini chat
```

```
You: List my 5 most recent Canva designs.

Gemini: Here are your 5 most recent designs:
  1. "Endora — Investor Pitch"  (Presentation · edited 2 hours ago)
  2. "March Newsletter"         (Email header · edited yesterday)
  3. "Instagram Story Pack"     (Social media · edited 3 days ago)
  4. "Q1 Board Report"          (Document · edited last week)
  5. "Product Launch Banner"    (Banner · edited last week)
```

If you see your designs listed, the integration is working correctly. Proceed to the [Tool Reference](tools.md) to explore everything you can do.

---

## Troubleshooting

### `gemini: command not found`

The npm global `bin` directory is not in your `PATH`. Fix it:

```bash
# Find the npm bin directory
npm bin -g

# Add it to your shell profile (e.g. ~/.zshrc or ~/.bashrc)
export PATH="$(npm bin -g):$PATH"
source ~/.zshrc
```

### Extension installs but 0 tools are listed

The MCP server could not be reached. Check:

1. You are connected to the internet.
2. Your corporate firewall does not block `mcp.canva.com`.
3. Run `gemini extensions refresh canva` to pull the latest manifest.

### OAuth redirect fails (`localhost` refused to connect)

Gemini CLI opens a local HTTP listener on port `9004` by default. If that port is occupied:

```bash
gemini extensions auth canva --port 9005
```

### Token expired / `401 Unauthorised`

```bash
gemini extensions auth canva --refresh
```

This forces a token refresh without re-opening the browser.

---

## Configuration Reference

All Canva extension settings live in `~/.config/gemini/extensions/canva/config.json`:

```jsonc
{
  "server": "https://mcp.canva.com/v1",
  "authPort": 9004, // local OAuth redirect port
  "timeout": 30000, // request timeout in ms
  "maxCandidates": 4, // designs returned per generate call
  "defaultExportFormat": "pdf", // "pdf" | "png" | "jpg" | "mp4"
}
```

Edit this file to override defaults for your workflow.

---

## Next Steps

➡ Continue to the **[Tool Reference](tools.md)** to see all 21 available tools with full examples.
