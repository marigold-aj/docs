# Matching together.ai's Navigation & Landing Page

A comparison of your current Mintlify docs against together.ai's, plus a concrete migration plan. Scope is limited to the two areas you asked about: **navigation/structure** and **landing/home page layout**. Visual branding and content tone are intentionally out of scope here.

---

## 1. Side-by-side comparison

### Navigation & structure

| Dimension | together.ai | One Perfect Slice (current) |
|---|---|---|
| **Top-level rail** | Six horizontal sections in the navbar header: `Docs`, `Guides`, `Demos`, `API`, `CLI`, `Changelog` (plus `Sign in` button) | Three Mintlify `tabs` rendered as a tab bar: `Getting Started`, `Product`, `Developers` (plus `Support` link + `Dashboard` button) |
| **How that rail is built** | Each label is its own Mintlify `tab` (or anchor) — the whole top row swaps the sidebar when you click | Same mechanism (Mintlify `tabs`), but only 3 entries and the split is "audience-y" (getting started / product / dev) |
| **Sidebar grouping inside a tab** | The `Docs` tab contains *multiple* sidebar groups stacked vertically: `GET STARTED`, `INFERENCE`, (likely `TRAINING`, etc.). Group titles are small uppercase labels | Each tab contains only **one** group whose name duplicates the tab name (e.g. tab `Getting Started` → group `Getting Started`). The grouping mechanism is unused |
| **Sidebar density** | ~8–12 items visible per group, light typography, indentation for sub-sections, expand chevrons on items with children (`Dedicated endpoints >`, `Voice >`) | 2–5 items per tab, no nesting, flat |
| **Active-page treatment** | Soft orange pill background behind the active item, orange text | Custom CSS forces a dark sidebar (`#1f1f1f`) with the brown brand color on the active row |
| **Search** | Prominent search bar at the top of the sidebar with `⌘K` hint, plus a sparkle "Ask AI" button next to it | Mintlify default search (top of navbar), no Ask-AI button |
| **Theme toggle** | Sun/moon icon pinned to the bottom-left corner of the sidebar | Mintlify default toggle in the navbar |
| **Top-right CTA** | `Sign in` (outlined button) | `Dashboard` (solid button) |

**The structural punchline:** together.ai uses **fewer** top-level tabs and **more** groups inside each one. You're doing the opposite — using tabs the way they use sidebar groups.

### Landing / home page

| Dimension | together.ai overview page | `index.mdx` (current) |
|---|---|---|
| **Eyebrow** | Small uppercase orange tag (`GET STARTED`) above the H1 | None |
| **Title** | Plain H1 `Overview` (the page is named for its role, not the product) | H1 `One Perfect Slice` (product name is the title) |
| **Subtitle** | One-line tagline directly under H1 (`Run, train, and serve open-source AI models on Together AI.`) | Two-line tagline, centered |
| **Hero layout** | **Two-column**: prose block left, tabbed code sample right (Python / TypeScript / cURL), inside a card with a thin border | **Single column, centered**: tagline first, then a centered `<CodeGroup>` below it |
| **Code tab order** | `Python`, `TypeScript`, `cURL` (Python first; the language is the dominant runtime for their audience) | `cURL`, `Python`, `TypeScript` (cURL first) |
| **Section headings on the home page** | None visible above the fold — the page leads with hero, then jumps into a visual card grid | Four prose H2s (`Get Started`, `Analyze Conversations`, `For Developers`, `Resources`) |
| **Cards** | Visual cards with full-bleed gradient/painterly artwork as the card background. Title sits below or over the artwork | Mintlify icon-cards: small Font Awesome icon + title + 1-line description, no imagery |
| **Card grid density** | A small number of large, eye-catching cards (3-up visible) — feels curated | 4 separate `<Columns>` blocks, ~13 cards total — feels exhaustive |
| **Persistent footer affordance** | Sticky `Ask a question…` input at the bottom of the content column (the AI assistant entry point) | None |
| **Page width** | Default content width, hero card respects the column | `mode: "wide"` is set, so the page is uncapped |

**The landing punchline:** together.ai's home is a *launchpad* — eyebrow, one-line value prop, code you can run, three big visual cards. Yours is a *site map* — a centered hero followed by four tiers of icon-cards covering every page in the docs.

---

## 2. Migration plan

Ordered roughly by impact-per-effort. Each step names the file and the specific keys/components to change.

### Step 1 — Restructure the top tabs (biggest visual shift)

Edit `docs.json` lines 26–85. Collapse to **fewer, broader tabs** and move the audience splits into sidebar groups inside each tab. A together.ai-shaped layout for OPS would look like:

- `Docs` — the everyday reading experience (Get Started, Product, MCP)
- `API Reference` — the OpenAPI-driven section
- `Changelog` — added later (see Step 6)

Concrete shape:

```json
"navigation": {
  "tabs": [
    {
      "tab": "Docs",
      "groups": [
        {
          "group": "Get started",
          "pages": ["index", "quickstart"]
        },
        {
          "group": "Product",
          "pages": [
            "product/slices",
            "product/scorecards",
            "product/posts",
            "product/crm-integration",
            "product/api-keys"
          ]
        },
        {
          "group": "MCP server",
          "pages": [
            "mcp/overview",
            "mcp/claude-code",
            "mcp/claude-desktop"
          ]
        }
      ]
    },
    {
      "tab": "API Reference",
      "groups": [
        {
          "group": "Reference",
          "pages": ["quickstart"],
          "openapi": "api/openapi.yaml"
        }
      ]
    }
  ]
}
```

This single change makes the sidebar look and feel like together.ai's: one tab open, multiple labeled groups stacked inside. Drop the existing `global.anchors` `Developers` entry — together.ai doesn't use anchors that way and once Developer content lives in the `API Reference` tab, the anchor is redundant.

### Step 2 — Add the eyebrow + tighter hero on every section landing page

Together.ai's signature is the small uppercase orange eyebrow above the H1 (`GET STARTED`, `INFERENCE`, etc.). Mintlify's frontmatter doesn't have a built-in eyebrow field, so add it as a small styled element at the top of the page. In `index.mdx` (and at the top of `product/slices.mdx`, `mcp/overview.mdx`, etc.):

```mdx
---
title: "Overview"
description: "Analyze sales conversations with AI. Surface insights, evidence, and scorecards — via the app, API, or MCP."
---

<div className="text-xs font-semibold tracking-widest uppercase mb-2" style={{color: 'var(--primary)'}}>
  Get started
</div>
```

Notes on the change:
- Drop `mode: "wide"` — together.ai uses default content width on the overview, which is what makes the hero card feel tight.
- Rename the H1 from `One Perfect Slice` to `Overview` and let the frontmatter `title` carry it (Mintlify renders frontmatter title as the H1 automatically). The product name belongs in the navbar/logo, not the H1.

### Step 3 — Convert the hero to a two-column layout

Replace the centered hero (`index.mdx` lines 7–48) with a two-column block: prose left, tabbed code right. Mintlify supports this with a `<Columns cols={2}>` wrapping a description on one side and a `<CodeGroup>` on the other. Reorder code tabs to lead with `Python`:

```mdx
<Columns cols={2}>
  <div>
    ### Developer quickstart

    Copy this snippet to start analyzing calls. See the [full quickstart](/quickstart) for more details.
  </div>

  <CodeGroup>
    ```python Python
    import requests

    resp = requests.get(
        "https://app.oneperfectslice.ai/api/public/v1/slices",
        headers={"Authorization": "Bearer sk_your_api_key"}
    )
    ```

    ```typescript TypeScript
    const resp = await fetch(
      "https://app.oneperfectslice.ai/api/public/v1/slices",
      { headers: { Authorization: "Bearer sk_your_api_key" } }
    );
    ```

    ```bash cURL
    curl https://app.oneperfectslice.ai/api/public/v1/slices \
      -H "Authorization: Bearer sk_your_api_key"
    ```
  </CodeGroup>
</Columns>
```

### Step 4 — Replace icon-card walls with a small, visual card grid

This is the biggest landing-page shift. Together.ai's home shows ~3 big cards with full-bleed gradient artwork, not 13 icon-cards across 4 sections.

Mintlify's `<Card>` accepts an `img` prop that puts an image in the card header. There's no built-in gradient prop, so the play is to commit gradient PNG/JPGs into `/images/cards/` and reference them. On `index.mdx` keep **one** card grid of three, drop the rest:

```mdx
<Columns cols={3}>
  <Card title="Quickstart" img="/images/cards/quickstart.png" href="/quickstart">
    Get an API key and run your first slice in 5 minutes.
  </Card>
  <Card title="Run a slice via API" img="/images/cards/api.png" href="/quickstart#3-run-a-slice">
    Start async analysis, poll for results, and retrieve evidence.
  </Card>
  <Card title="MCP server" img="/images/cards/mcp.png" href="/mcp/overview">
    Connect OPS to Claude Code, Claude Desktop, and other agents.
  </Card>
</Columns>
```

Generate the three gradient images using your brand palette (`#773624` brown, `#F08C80` salmon, plus a secondary teal `#10a690` you already use for `GET` badges). Anything that doesn't earn its place on the home page moves into the relevant section's overview page.

### Step 5 — Turn on Ask AI (the bottom search bar)

Together.ai's sticky `Ask a question…` input at the bottom is Mintlify's native AI assistant. It's a docs.json toggle, not custom code. Add at the top level of `docs.json`:

```json
"contextual": {
  "options": ["copy", "view", "chatgpt", "claude"]
}
```

…and verify the AI chat / Assistant feature is enabled in your Mintlify dashboard (it's a paid-plan feature on most tiers — confirm before relying on it). The widget renders automatically once enabled. The sparkle icon next to the search bar in their screenshot is the same feature surfaced in a second spot.

### Step 6 — (Optional, post-MVP) Add a Changelog tab

Together.ai treats Changelog as a top-level tab. If you have release notes worth surfacing, add a `changelog/` folder and a fourth tab. Mintlify also has a built-in changelog component pattern that mirrors what they show.

### Step 7 — Sidebar polish to match the orange-pill active state

Your `custom.css` currently paints the sidebar dark (`#1f1f1f`) with white text. Together.ai uses the platform default (light sidebar in light mode) and only highlights the *active item* with a soft orange tint. To match:

In `custom.css` lines 29–55, remove the global `#sidebar` background overrides. Keep only the active-row treatment:

```css
#sidebar a[data-active="true"],
#sidebar .active {
  background: rgba(240, 140, 128, 0.15); /* salmon @ 15% */
  color: #773624;
  border-radius: 6px;
}
.dark #sidebar a[data-active="true"] {
  background: rgba(240, 140, 128, 0.20);
  color: #f08c80;
}
```

Also drop the `#navbar` dark-bg override (lines 57–82) — it fights Mintlify's clean navbar.

---

## 3. Suggested order of operations

1. Step 1 (nav restructure) on a branch — preview locally, this changes the whole feel.
2. Steps 2 + 3 (eyebrow, two-column hero) on `index.mdx`.
3. Step 7 (sidebar CSS cleanup) — the dark sidebar will start fighting the new layout.
4. Step 4 (visual card grid) — needs the three artwork files; the slowest step because of the assets.
5. Step 5 (Ask AI) once the structure is settled.
6. Step 6 (Changelog) when there's content to put there.

## 4. What I deliberately did not propose

- Renaming brand colors. Your existing `#773624` / `#F08C80` palette already lives in the same family as together.ai's orange — the issue is *layout and surface treatment*, not hue.
- Switching fonts away from Yantramanav. That's a brand choice, not a together.ai-match question.
- Replicating the bottom-left light/dark pill exactly — Mintlify's default placement in the navbar is fine and the move would be cosmetic.

---

Sources for the Mintlify capability checks underlying this plan:
- [Mintlify — Navigation](https://www.mintlify.com/docs/organize/navigation)
- [Mintlify — Tabs, Anchors, Dropdowns](https://mintlify.com/docs/navigation/divisions)
- [Mintlify — Cards component](https://mintlify.com/docs/content/components/cards)
- [Mintlify — Refactoring mint.json into docs.json](https://www.mintlify.com/blog/refactoring-mint-json-into-docs-json)
