---
layout: default
title: Tool Reference
nav_order: 3
description: "Complete reference for all 21 Canva MCP tools — parameters, return values, and real-world usage examples."
permalink: /tools/
---

# Tool Reference
{: .no_toc }

All 21 tools available through the Canva MCP integration, organised by category.
{: .fs-5 .fw-300 }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## How to Read This Reference

Each tool entry follows this structure:

- **Purpose** — what the tool does in plain English
- **Parameters** — required (`*`) and optional inputs
- **Returns** — the shape of the structured response
- **Example** — a real prompt and the resulting MCP call

> **Candidate IDs vs Final Designs:** When a tool creates or searches for designs it often returns *candidate* results — temporary previews with short-lived IDs. You must **commit** a candidate to promote it to a permanent design. See the [Best Practices](best-practices.md) page for a full explanation.

---

## Category 1 · Design Creation

### 1. `generate-design`

**Purpose:** Generate one or more new design candidates from a natural-language description.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `prompt` | string | ✅ | Natural-language description of the design |
| `designType` | string | ✅ | e.g. `"presentation"`, `"instagram-post"`, `"logo"` |
| `aspectRatio` | string | | `"16:9"` (default), `"1:1"`, `"9:16"`, `"4:3"` |
| `count` | integer | | Number of candidates to return (1–4, default 2) |
| `brandKit` | boolean | | Apply the account's active brand kit (default `false`) |

**Returns:** Array of candidate objects — each with a `candidateId`, thumbnail URL, and a short expiry timestamp.

**Example:**

```
You: Generate a dark-themed investor pitch for "Endora," a wellness app targeting Gen Z.
     Use 16:9 format and apply our brand kit.

MCP call:
  generate-design(
    prompt      = "Dark-themed investor pitch for Endora wellness app, Gen Z audience",
    designType  = "presentation",
    aspectRatio = "16:9",
    count       = 2,
    brandKit    = true
  )

Response:
  candidates: [
    { candidateId: "a3f9c1", thumbnail: "https://...", expiresAt: "2026-02-20T14:00:00Z" },
    { candidateId: "b71e40", thumbnail: "https://...", expiresAt: "2026-02-20T14:00:00Z" }
  ]
```

---

### 2. `create-design`

**Purpose:** Create a blank design canvas of a specified type and size, ready for manual or automated editing.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designType` | string | ✅ | Template category (see full list in Canva docs) |
| `title` | string | | Human-readable name stored in your Canva library |
| `width` | integer | | Width in pixels (overrides `designType` defaults) |
| `height` | integer | | Height in pixels |

**Returns:** `{ designId, editUrl, thumbnailUrl }`

**Example:**

```
You: Create a blank A4 document called "Endora Brand Guidelines."

MCP call:
  create-design(
    designType = "document",
    title      = "Endora Brand Guidelines",
    width      = 2480,
    height     = 3508
  )
```

---

### 3. `duplicate-design`

**Purpose:** Create an exact copy of an existing design — useful for generating variations without touching the original.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | ID of the source design |
| `title` | string | | Name for the new copy |

**Returns:** `{ newDesignId, editUrl }`

---

### 4. `create-from-template`

**Purpose:** Instantiate a design from one of Canva's 1 M+ templates using a template ID or keyword.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `templateId` | string | | Specific template ID from Canva's library |
| `query` | string | | Keyword search (used if `templateId` is omitted) |
| `designType` | string | | Filter results by design type |
| `title` | string | | Name for the new design |

**Returns:** `{ designId, editUrl, templateName }`

**Example:**

```
You: Start a new pitch deck from a template — something clean and minimal.

MCP call:
  create-from-template(
    query      = "minimal pitch deck",
    designType = "presentation",
    title      = "Endora Pitch v2"
  )
```

---

## Category 2 · Design Discovery & Search

### 5. `search-designs`

**Purpose:** Search your personal Canva library by keyword, design type, or date range.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `query` | string | ✅ | Full-text search query |
| `designType` | string | | Filter by type (e.g. `"presentation"`) |
| `limit` | integer | | Max results (1–50, default 10) |
| `sortBy` | string | | `"relevance"` (default), `"modified"`, `"created"` |

**Returns:** Array of `{ designId, title, thumbnailUrl, lastModified, designType }`

---

### 6. `get-design`

**Purpose:** Fetch full metadata and a high-resolution thumbnail for a single design by its ID.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | The target design's unique identifier |

**Returns:** `{ designId, title, thumbnailUrl, editUrl, pages, width, height, createdAt, modifiedAt }`

---

### 7. `list-designs`

**Purpose:** Paginate through all designs in your library without a search query.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `limit` | integer | | Items per page (default 20, max 50) |
| `cursor` | string | | Pagination cursor from a previous response |
| `designType` | string | | Optional type filter |

**Returns:** `{ designs: [...], nextCursor, totalCount }`

---

## Category 3 · Content Editing

### 8. `set-text`

**Purpose:** Replace the text content of one or more text elements within a design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `replacements` | array | ✅ | Array of `{ elementId, newText }` objects |

**Returns:** `{ updated: number, designId }`

**Example:**

```
You: On slide 1 of the Endora pitch, change the headline to
     "Redefining Wellness for Generation Z."

MCP call:
  set-text(
    designId     = "a3f9c1_committed",
    replacements = [{ elementId: "title_01", newText: "Redefining Wellness for Generation Z." }]
  )
```

---

### 9. `set-image`

**Purpose:** Swap an image element in a design with a new image (by URL or Canva asset ID).

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `elementId` | string | ✅ | ID of the image element to replace |
| `imageUrl` | string | | Public URL of the replacement image |
| `assetId` | string | | Canva asset ID (mutually exclusive with `imageUrl`) |

---

### 10. `apply-brand`

**Purpose:** Apply the account's active brand kit (colours, fonts, logo) to all eligible elements in a design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `brandKitId` | string | | Specific brand kit (defaults to the account's primary kit) |
| `scope` | string | | `"all"` (default), `"colors"`, `"fonts"`, `"logos"` |

---

### 11. `resize-design`

**Purpose:** Resize an existing design to new dimensions, with optional content rescaling.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `width` | integer | ✅ | New width in pixels |
| `height` | integer | ✅ | New height in pixels |
| `scaleContent` | boolean | | Proportionally scale all elements (default `true`) |

---

### 12. `add-page`

**Purpose:** Append a new blank page (slide) to a multi-page design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `position` | integer | | Insert at position (1-indexed; appends to end if omitted) |
| `copyFromPage` | integer | | Copy layout from this existing page number |

---

### 13. `delete-page`

**Purpose:** Remove a page from a multi-page design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `pageNumber` | integer | ✅ | 1-indexed page to delete |

---

### 14. `reorder-pages`

**Purpose:** Change the order of pages within a multi-page design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `pageOrder` | array | ✅ | New page sequence as an array of 1-indexed page numbers |

---

## Category 4 · Asset Management

### 15. `upload-asset`

**Purpose:** Upload an image, video, or audio file from a URL into your Canva asset library.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `url` | string | ✅ | Publicly accessible URL of the file to upload |
| `name` | string | | Display name in the Canva library |
| `tags` | array | | Array of string tags for searchability |

**Returns:** `{ assetId, name, url, mimeType, fileSize }`

---

### 16. `list-assets`

**Purpose:** List assets (images, videos, audio) from your Canva library.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `type` | string | | `"image"`, `"video"`, `"audio"` (omit for all) |
| `query` | string | | Keyword search within asset names/tags |
| `limit` | integer | | Max results (default 20) |

---

### 17. `delete-asset`

**Purpose:** Permanently delete an asset from your Canva library.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `assetId` | string | ✅ | ID of the asset to delete |

> ⚠️ **This action is irreversible.** Any designs using this asset will display a placeholder.

---

## Category 5 · Export & Sharing

### 18. `export-design`

**Purpose:** Export a finished design to a downloadable file in the requested format.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `format` | string | ✅ | `"pdf"`, `"png"`, `"jpg"`, `"svg"`, `"mp4"`, `"gif"`, `"pptx"` |
| `pages` | array | | Page numbers to export (omit for all pages) |
| `quality` | string | | `"standard"` or `"high"` (default `"high"`) |

**Returns:** `{ downloadUrl, expiresAt, fileSize, format }`

**Example:**

```
You: Export the Endora pitch deck as a PDF.

MCP call:
  export-design(
    designId = "a3f9c1_committed",
    format   = "pdf",
    quality  = "high"
  )

Response:
  { downloadUrl: "https://export.canva.com/...", expiresAt: "2026-02-21T00:00:00Z", fileSize: "4.2 MB" }
```

---

### 19. `get-share-link`

**Purpose:** Generate a shareable Canva link for viewing or editing a design.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | Target design ID |
| `access` | string | ✅ | `"view"` or `"edit"` |
| `expiresIn` | string | | Duration string e.g. `"7d"`, `"24h"` (default: no expiry) |

**Returns:** `{ shareUrl, access, expiresAt }`

---

## Category 6 · Workflow & Automation

### 20. `commit-candidate`

**Purpose:** Promote a **candidate design** (a temporary preview) into a permanent design saved to your Canva library.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `candidateId` | string | ✅ | The short-lived candidate ID from `generate-design` |
| `title` | string | | Name for the committed design |

**Returns:** `{ designId, editUrl, thumbnailUrl }`

> 🔑 **This is a critical step.** Candidate IDs expire in under 1 hour. You must call `commit-candidate` before editing or exporting. See [Best Practices](best-practices.md#candidate-ids-vs-final-designs).

**Example:**

```
You: I like the first option. Save it.

MCP call:
  commit-candidate(
    candidateId = "a3f9c1",
    title       = "Endora Investor Pitch — Final"
  )

Response:
  { designId: "a3f9c1_committed", editUrl: "https://www.canva.com/design/a3f9c1_committed/edit" }
```

---

### 21. `delete-design`

**Purpose:** Permanently delete a design from your Canva library.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `designId` | string | ✅ | ID of the design to delete |

**Returns:** `{ deleted: true, designId }`

> ⚠️ **This action is irreversible** and cannot be undone via the API.

---

## Full Tool Summary Table

| # | Tool | Category | Creates Content? | Destructive? |
|---|---|---|---|---|
| 1 | `generate-design` | Creation | ✅ | |
| 2 | `create-design` | Creation | ✅ | |
| 3 | `duplicate-design` | Creation | ✅ | |
| 4 | `create-from-template` | Creation | ✅ | |
| 5 | `search-designs` | Discovery | | |
| 6 | `get-design` | Discovery | | |
| 7 | `list-designs` | Discovery | | |
| 8 | `set-text` | Editing | | |
| 9 | `set-image` | Editing | | |
| 10 | `apply-brand` | Editing | | |
| 11 | `resize-design` | Editing | | |
| 12 | `add-page` | Editing | | |
| 13 | `delete-page` | Editing | | ⚠️ |
| 14 | `reorder-pages` | Editing | | |
| 15 | `upload-asset` | Assets | ✅ | |
| 16 | `list-assets` | Assets | | |
| 17 | `delete-asset` | Assets | | ⚠️ |
| 18 | `export-design` | Export | | |
| 19 | `get-share-link` | Sharing | | |
| 20 | `commit-candidate` | Workflow | ✅ | |
| 21 | `delete-design` | Workflow | | ⚠️ |

---

## Real-World Walkthrough: The Endora Pitch Deck

This end-to-end example shows how multiple tools combine to produce a finished asset from a single conversation.

```
# 1. Generate candidates
You: Create a 16:9 investor pitch for Endora, a Gen Z wellness app.
     Dark purple palette, 10 slides, apply our brand kit.

→ generate-design(prompt="...", designType="presentation", count=2, brandKit=true)
  candidateId: "a3f9c1" (option 1)  |  candidateId: "b71e40" (option 2)

# 2. Review thumbnails, choose option 1
You: The first one looks great. Save it as "Endora Pitch — Feb 2026."

→ commit-candidate(candidateId="a3f9c1", title="Endora Pitch — Feb 2026")
  designId: "DESa3f9c1abc"

# 3. Update the title slide text
You: Set the headline on slide 1 to "Redefining Wellness for Gen Z."
     Set the subheadline to "Series A · $5M Raise · Q2 2026."

→ set-text(designId="DESa3f9c1abc", replacements=[
    { elementId: "h1_s1", newText: "Redefining Wellness for Gen Z." },
    { elementId: "h2_s1", newText: "Series A · $5M Raise · Q2 2026." }
  ])

# 4. Upload a product screenshot and swap it into slide 3
You: Use this screenshot for the product demo slide.
     [user provides URL: https://endora.app/assets/screenshot-v2.png]

→ upload-asset(url="https://endora.app/assets/screenshot-v2.png", name="Endora App Screenshot v2")
  assetId: "AST9924bc"

→ set-image(designId="DESa3f9c1abc", elementId="img_s3_main", assetId="AST9924bc")

# 5. Add a final "Thank You" slide
You: Add a closing thank-you slide at the end.

→ add-page(designId="DESa3f9c1abc", copyFromPage=10)
→ set-text(designId="DESa3f9c1abc", replacements=[
    { elementId: "h1_s11", newText: "Thank You" },
    { elementId: "h2_s11", newText: "hello@endora.app  ·  endora.app" }
  ])

# 6. Export as PDF
You: Export the final deck as a high-quality PDF.

→ export-design(designId="DESa3f9c1abc", format="pdf", quality="high")
  downloadUrl: "https://export.canva.com/DESa3f9c1abc/endora-pitch-feb-2026.pdf"
  fileSize: "6.8 MB"

# 7. Share a view-only link with the investor
You: Generate a view-only link that expires in 7 days.

→ get-share-link(designId="DESa3f9c1abc", access="view", expiresIn="7d")
  shareUrl: "https://www.canva.com/view/DESa3f9c1abc?token=xyz"
```

---

## Next Steps

➡ Read **[Best Practices](best-practices.md)** to learn about Candidate IDs, rate limits, and advanced prompt patterns.
