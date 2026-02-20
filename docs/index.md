---
layout: default
title: Home
nav_order: 1
description: "Canva MCP Integration Guide — connect Gemini (and other LLMs) to Canva's design platform via the Model Context Protocol."
permalink: /
---

# Canva MCP Integration Guide

{: .fs-9 }

Supercharge your design workflow by letting AI models talk directly to Canva.
{: .fs-6 .fw-300 }

[Get Started →](setup.md){: .btn .btn-primary .fs-5 .mb-4 .mb-md-0 .mr-2 }
[Browse Tools →](tools.md){: .btn .fs-5 .mb-4 .mb-md-0 }

---

## What Is the Model Context Protocol (MCP)?

The **Model Context Protocol (MCP)** is an open standard developed to give large language models (LLMs) — such as Google Gemini, Anthropic Claude, and OpenAI GPT — a secure, structured way to access external tools, APIs, and data sources at runtime.

Think of MCP as a **universal adapter layer** that sits between an AI model and the outside world:

```
┌───────────────────┐        MCP           ┌──────────────────────┐
│   LLM / AI Agent  │ ◄─────────────────►  │  External Tool / API │
│  (e.g. Gemini)    │   structured calls   │  (e.g. Canva, GitHub)│
└───────────────────┘                      └──────────────────────┘

```

![Alt text](assets/image.png)

### Why MCP Matters

| Concern             | Without MCP                     | With MCP                             |
| ------------------- | ------------------------------- | ------------------------------------ |
| **Security**        | API keys embedded in prompts    | OAuth-scoped tokens, no key exposure |
| **Standardisation** | One custom integration per tool | One protocol, many tools             |
| **Discoverability** | Model must guess capabilities   | Tools self-describe via manifests    |
| **Auditability**    | Opaque tool calls               | Structured, loggable requests        |

MCP follows a **client → server → tool** pattern:

1. The **MCP Client** (e.g. Gemini CLI) discovers what tools are available.
2. The **MCP Server** (e.g. Canva's server) validates the request and enforces permissions.
3. The **Tool** (e.g. `generate-design`) executes the action and returns structured data.

---

## What Is the Canva MCP Integration?

Canva provides an official **MCP server** that exposes its design platform to any MCP-compatible AI model. Once authenticated, your AI assistant can:

- 🎨 **Create** branded presentations, social posts, videos, and more — all from natural language.
- 🔍 **Search** your existing Canva designs by keyword, type, or date.
- ✏️ **Edit** live designs: swap text, update colours, resize elements.
- 📤 **Export** finished assets in multiple formats directly from the conversation.

### How It Works End-to-End

```
You (natural language prompt)
      │
      ▼
Gemini CLI  ──[MCP]──►  Canva MCP Server  ──►  Canva Platform
      ▲                                               │
      └──────────── structured response ◄────────────┘
```

---

## Documentation Map

| Page                                    | Contents                                                       |
| --------------------------------------- | -------------------------------------------------------------- |
| **[Setup Guide](setup.md)**             | Install Gemini CLI · Register the Canva extension · OAuth flow |
| **[Tool Reference](tools.md)**          | All 21 tools with parameters, examples & return values         |
| **[Best Practices](best-practices.md)** | Candidate IDs vs Final Designs · Prompt patterns · Rate limits |

---

## Prerequisites at a Glance

- A **Google account** with access to [Google AI Studio](https://aistudio.google.com) or Gemini Advanced
- A **Canva account** (Free, Pro, or Teams)
- **Node.js ≥ 18** or **Python ≥ 3.10** installed locally
- **Gemini CLI** (installation covered in [Setup](setup.md))

---

## Quick Example

Once set up, you can generate a complete branded presentation in a single sentence:

```
You: Create a 10-slide investor pitch for "Endora" — a wellness app.
     Use a dark purple colour palette and include a market size slide.

Gemini + Canva MCP:
  ✔ generate-design  →  created candidate a3f9c1 (Presentation, 16:9)
  ✔ set-text         →  "Endora" applied to title slide
  ✔ apply-brand      →  colour palette #3B0764 / #7C3AED applied
  ✔ duplicate-slide  →  10 slides scaffolded
  ✔ export-design    →  PDF exported → ~/Downloads/endora-pitch.pdf
```

---

## Licence & Contributing

This documentation is released under the [MIT Licence](https://opensource.org/licenses/MIT).  
Spotted an error? Open a pull request or [file an issue](https://github.com/your-username/mcp_config/issues).
