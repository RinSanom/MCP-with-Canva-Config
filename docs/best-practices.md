---
layout: default
title: Best Practices
nav_order: 4
description: "Candidate IDs vs Final Designs, prompt engineering patterns, rate limits, and advanced tips."
permalink: /best-practices/
---

# Best Practices

{: .no_toc }

Patterns and guardrails for getting the most out of the Canva MCP integration.
{: .fs-5 .fw-300 }

## Table of Contents

{: .no_toc .text-delta }

1. TOC
   {:toc}

---

## Candidate IDs vs Final Designs

This is the single most important concept to understand when working with the Canva MCP integration.

### The Two-Stage Creation Model

When you call `generate-design`, Canva does **not** immediately save a design to your library. Instead, it returns one or more **candidate designs** — lightweight, temporary previews that let you compare options before committing storage.

```
Stage 1 — Generate candidates (temporary)
──────────────────────────────────────────
generate-design(...)
  └─► candidateId: "a3f9c1"  ← short-lived, expires in ~1 hour
  └─► candidateId: "b71e40"  ← short-lived, expires in ~1 hour

Stage 2 — Commit your chosen candidate (permanent)
──────────────────────────────────────────────────
commit-candidate(candidateId: "a3f9c1", title: "My Design")
  └─► designId: "DESa3f9c1abc"  ← permanent, lives in your library
```

### Why This Matters

| Characteristic              | Candidate ID                     | Final Design ID                                             |
| --------------------------- | -------------------------------- | ----------------------------------------------------------- |
| **Format**                  | Short alphanumeric e.g. `a3f9c1` | Longer e.g. `DESa3f9c1abc`                                  |
| **Lifespan**                | ~1 hour (server-side TTL)        | Permanent until deleted                                     |
| **Editable?**               | ❌ No — read-only preview        | ✅ Yes — fully editable                                     |
| **Exportable?**             | ❌ No                            | ✅ Yes                                                      |
| **Shareable?**              | ❌ No                            | ✅ Yes                                                      |
| **Counts against library?** | ❌ No                            | ✅ Yes                                                      |
| **Created by**              | `generate-design`                | `commit-candidate`, `create-design`, `create-from-template` |

### The Golden Rule

> **Never attempt to edit, export, or share a candidate ID.**  
> Always call `commit-candidate` first to promote it to a permanent design.

### Handling Expiry Gracefully

If a candidate has expired before you commit it, Gemini will surface a `CANDIDATE_EXPIRED` error. The correct recovery is to re-run `generate-design` with the same (or refined) prompt:

```
MCP error: CANDIDATE_EXPIRED — candidateId "a3f9c1" is no longer valid.

You: The candidate expired. Regenerate the same pitch deck design.

→ generate-design(prompt="...", ...)  // re-runs with identical params
```

---

## How to Edit a Committed Design

Once a candidate is committed, use the **editing tools** to modify it programmatically:

### Text Edits

```
# First, discover element IDs on a page
→ get-design(designId="DES...")
  └─► pages[0].elements = [
        { id: "h1_s1", type: "text", content: "Old Headline" },
        { id: "body_s1", type: "text", content: "Old body copy" }
      ]

# Then replace by ID
→ set-text(designId="DES...", replacements=[
    { elementId: "h1_s1",   newText: "New Headline" },
    { elementId: "body_s1", newText: "Updated body copy." }
  ])
```

### Image Swaps

```
# Upload new image first
→ upload-asset(url="https://...", name="New Hero Image")
  └─► assetId: "ASTxyz"

# Swap into the design
→ set-image(designId="DES...", elementId="img_hero", assetId="ASTxyz")
```

### Page Operations

Work from the **end of the deck backwards** when deleting pages to avoid index shifting:

```
# ✅ Delete pages 8, then 5 (end → start)
→ delete-page(designId="DES...", pageNumber=8)
→ delete-page(designId="DES...", pageNumber=5)

# ❌ Avoid: deleting page 5 first shifts all subsequent page numbers
```

---

## Prompt Engineering for `generate-design`

### Structure Your Prompts in Three Parts

```
[AUDIENCE + PURPOSE] + [VISUAL STYLE] + [CONTENT REQUIREMENTS]
```

**Example — weak prompt:**

```
Make a presentation for Endora.
```

**Example — strong prompt:**

```
Create a 10-slide investor pitch deck for Endora, a Gen Z wellness app.
Visual style: dark purple (#3B0764) and electric violet (#7C3AED), clean sans-serif fonts,
full-bleed photography with text overlay. Include slides for: problem, solution,
market size ($420B), product demo, business model, traction, team, roadmap, financials, ask.
```

### Useful Prompt Modifiers

| Modifier                                  | Effect                                  |
| ----------------------------------------- | --------------------------------------- |
| `"Use our brand kit"`                     | Triggers `brandKit: true` automatically |
| `"Give me 4 options"`                     | Sets `count: 4`                         |
| `"16:9 format"` / `"square"`              | Sets appropriate `aspectRatio`          |
| `"Based on the [template name] template"` | Routes to `create-from-template`        |
| `"Minimal"` / `"Bold"` / `"Corporate"`    | Influences visual tone                  |

### Iterating on Designs

Instead of editing a committed design when the output is far from what you want, it is often faster to regenerate with a refined prompt and a higher `count`:

```
# First pass — 2 candidates
→ generate-design(prompt="dark wellness pitch", count=2)
  "Neither looks right — too light, not enough purple."

# Second pass — 4 candidates, refined prompt
→ generate-design(
    prompt = "Dark, moody wellness pitch deck. Deep purple dominant colour,
              electric violet accents. NO white backgrounds.",
    count  = 4
  )
```

---

## Working with Brand Kits

When `brandKit: true` is passed to `generate-design` or `apply-brand` is called, Canva applies the **primary brand kit** associated with the logged-in account. To apply a specific brand kit:

```
→ apply-brand(designId="DES...", brandKitId="BK_ENDORA_2026", scope="colors")
```

Retrieve available brand kit IDs from your Canva account settings, or ask Gemini:

```
You: What brand kits do I have in Canva?

→ (Gemini uses search-designs + internal metadata to list available kits)
```

---

## Rate Limits & Quotas

| Limit                                     | Value   | Notes                                     |
| ----------------------------------------- | ------- | ----------------------------------------- |
| **Requests per minute**                   | 60      | Applies to all tools combined             |
| **`generate-design` per hour**            | 20      | AI generation is resource-intensive       |
| **Candidates per `generate-design` call** | 4 max   | Set with `count` parameter                |
| **Asset upload size**                     | 100 MB  | Per file                                  |
| **Export downloads**                      | 100/day | Download URL expires in 24 hours          |
| **Concurrent sessions**                   | 1       | Only one active OAuth session per account |

When you hit a rate limit, Gemini will surface a `RATE_LIMIT_EXCEEDED` error with a `retryAfter` timestamp. Wait for that period before retrying.

---

## Security Best Practices

### Never Share Your Token File

The OAuth token file at `~/.config/gemini/extensions/canva/token.json` contains your Canva refresh token. Treat it like a password:

```bash
# Correct permissions (user-only read/write)
chmod 600 ~/.config/gemini/extensions/canva/token.json
```

### Revoke Access When Not Needed

```bash
gemini extensions revoke canva
```

This invalidates the refresh token server-side. You will need to re-authenticate before using Canva tools again.

### Audit Tool Calls

Enable verbose logging to keep a record of every MCP call:

```bash
gemini chat --log-level verbose --log-file ~/gemini-canva-audit.log
```

---

## Common Mistakes & How to Avoid Them

| ❌ Mistake                                  | ✅ Fix                                             |
| ------------------------------------------- | -------------------------------------------------- |
| Calling `set-text` on a `candidateId`       | Always `commit-candidate` first                    |
| Deleting pages from the front of the deck   | Delete from the back first, or use `reorder-pages` |
| Passing an expired candidate to any tool    | Re-run `generate-design` with the same prompt      |
| Uploading an asset but forgetting to use it | Call `set-image` after `upload-asset`              |
| Exporting before all edits are done         | Sequence: edit → verify → export                   |
| Using `delete-design` on the wrong ID       | Use `get-design` to confirm ID and title first     |

---

## Cheat Sheet

```
📐 CREATE
  generate-design  →  commit-candidate  →  edit  →  export

🔍 FIND
  search-designs / list-designs  →  get-design (for full metadata)

✏️  EDIT  (always on committed designId, never candidateId)
  set-text · set-image · apply-brand · resize-design
  add-page · delete-page · reorder-pages

📦 ASSETS
  upload-asset  →  set-image (reference assetId in designs)

📤 SHARE
  export-design (pdf/png/pptx/mp4)
  get-share-link (view or edit, optional expiry)

🗑️  CLEAN UP (irreversible!)
  delete-page · delete-asset · delete-design
```

---

## Further Reading

- [MCP Specification (modelcontextprotocol.io)](https://modelcontextprotocol.io)
- [Canva Developer Portal](https://www.canva.com/developers/)
- [Gemini CLI Documentation](https://ai.google.dev/gemini-api/docs/cli)
- [Back to Tool Reference](tools.md)
