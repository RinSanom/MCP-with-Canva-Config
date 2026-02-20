# Canva MCP Integration Guide

> **Documentation site** for integrating Canva with AI models (Gemini, Claude, GPT) via the **Model Context Protocol (MCP)**.

[![GitHub Pages](https://img.shields.io/badge/docs-GitHub%20Pages-blue?logo=github)](https://your-username.github.io/mcp_config)
[![Licence: MIT](https://img.shields.io/badge/Licence-MIT-green.svg)](https://opensource.org/licenses/MIT)

---

## What's Inside

| File | Purpose |
|---|---|
| [`docs/index.md`](docs/index.md) | Introduction — what MCP is and how the Canva integration works |
| [`docs/setup.md`](docs/setup.md) | Installation guide — Gemini CLI, extension install, OAuth flow |
| [`docs/tools.md`](docs/tools.md) | Full reference for all 21 MCP tools with examples |
| [`docs/best-practices.md`](docs/best-practices.md) | Candidate IDs vs Final Designs, prompt patterns, security |
| [`_config.yml`](_config.yml) | Jekyll/GitHub Pages site configuration |

---

## Quick Start

```bash
# 1. Install Gemini CLI
npm install -g @google/gemini-cli

# 2. Authenticate with Google
gemini auth login

# 3. Register the Canva MCP extension
gemini extensions install canva

# 4. Start chatting — the OAuth flow triggers on first Canva tool use
gemini chat
```

See the **[full Setup Guide](docs/setup.md)** for detailed steps and troubleshooting.

---

## Hosting on GitHub Pages

1. Push this repository to GitHub.
2. Go to **Settings → Pages → Source** and select `main` branch, `/ (root)`.
3. GitHub Pages will build and publish the Jekyll site automatically.
4. Update `url` and `baseurl` in [`_config.yml`](_config.yml) to match your repository.

---

## Project Structure

```
mcp_config/
├── _config.yml              # Jekyll site configuration
├── README.md                # This file (GitHub repo overview)
└── docs/
    ├── assets/              # Images, logos, diagrams
    ├── index.md             # Home page
    ├── setup.md             # Installation guide
    ├── tools.md             # Tool reference (21 tools)
    └── best-practices.md    # Tips, patterns, security
```

---

## Licence

MIT © 2026 Your Organisation
