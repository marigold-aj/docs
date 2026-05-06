# Implementation plan — IA, file moves, and tickets

Companion to `together-ai-style-plan.md`. That doc explains *what* changes; this doc lays out the new information architecture and breaks the work into ticketed PRs you can pick up one at a time.

---

## 1. User-needs framework

Each tab answers one question for one user. If two pages answer the same question for the same user, they belong in the same tab.

| User intent | Persona | Tab |
|---|---|---|
| "What is OPS, how do I start, how does this concept work?" | App user, sales lead, evaluator, admin, RevOps | **Docs** |
| "How do I plug Claude / an AI agent into OPS?" | Agent power user | **MCP** |
| "What endpoints can I call?" | Developer integrator | **API** |
| "Give me drop-in skills my coding agent can load" | Agent author *(future)* | **Skills** *(reserved)* |

This is the structure together.ai actually uses: one umbrella **Docs** tab whose sidebar holds Getting Started + topic-area groups, with **API** as a separate tab for endpoint reference. The OPS-specific addition is **MCP** as its own tab — together.ai treats MCP as one item in their Get Started group, but for OPS, agent integration is a primary user journey with three pages today and more coming, so it earns the slot.

**Tab order — Docs → MCP → API.** Docs first because every persona starts there. MCP before API is an OPS-specific call: agent-native positioning makes MCP the more strategic surface, and API users will navigate directly to `/api/*` regardless of left-to-right ordering.

**Why not separate Product / Platform / Accounts / Integrations into their own tabs?** Splitting them would put a 1-page "Integrations" tab next to a 4-page "Accounts" tab and a 3-page "Product" tab — sparse, and forces sales leads, admins, and RevOps users to hop between tabs to read related content. Inside one Docs tab, the sidebar groups create the same audience separation without the top-nav clutter.

**Why not 6 tabs like together.ai?** They have ~150 pages, a CLI, runnable demos, and weekly releases. With ~15 MDX pages today (existing + planned), six tabs would mean some tabs hold 1–2 pages — the opposite of the dense sidebar you want.

**Trigger for adding Skills:** ≥2 `.md` skill files exist with at least one tested. Until then, an empty Skills tab reads as abandoned and hurts more than it helps.

---

## 2. Target information architecture

```
┌─────────────────────────────────────────────────────────────────┐
│  Tab: Docs   (this is the / landing page)                        │
├─────────────────────────────────────────────────────────────────┤
│  Group: Getting Started                                          │
│    • Overview                       /                             │
│    • Quickstart                     /quickstart                   │
│                                                                   │
│  Group: Product                                                   │
│    • Slices                         /product/slices               │
│    • Scorecards                     /product/scorecards           │
│    • Posts                          /product/posts                │
│                                                                   │
│  Group: Integrations                                              │
│    • Overview                       /integrations                 │
│    • HubSpot                        /integrations/hubspot         │
│    • [Salesforce, Slack, … future]                                │
│                                                                   │
│  Group: Accounts                                                  │
│    • Organizations                  /accounts/organizations       │
│    • Teams                          /accounts/teams               │
│    • Members & roles                /accounts/members             │
│    • Billing                        /accounts/billing             │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Tab: MCP                                                         │
├─────────────────────────────────────────────────────────────────┤
│  Group: Get started                                               │
│    • Overview                       /mcp/overview                 │
│                                                                   │
│  Group: Setup                                                     │
│    • Claude Code                    /mcp/claude-code              │
│    • Claude Desktop                 /mcp/claude-desktop           │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Tab: API                                                         │
├─────────────────────────────────────────────────────────────────┤
│  Group: Get started                                               │
│    • Introduction                   /api/introduction             │
│                                                                   │
│  Group: API Reference                                             │
│    • Authentication                 /api/authentication           │
│    • Rate Limiting                  /api/rate-limiting            │
│    • Pagination                     /api/pagination               │
│    • Errors                         /api/errors                   │
│                                                                   │
│  Group: Endpoints (auto-generated from openapi.yaml, by tag)      │
│    • Slices                         /api/endpoints/slices/...     │
│    • Runs                           /api/endpoints/runs/...       │
│    • Posts                          /api/endpoints/posts/...      │
│    • [other tag groups as defined in openapi.yaml]                │
│                                                                   │
│  Group: Data Models                                               │
│    • Overview                       /api/data-models              │
│    • [individual model pages added as they're written]            │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│  Tab: Skills (reserved — create when content exists)              │
├─────────────────────────────────────────────────────────────────┤
│  Group: Available skills                                          │
│    • [.md skill files as they're added]   /skills/[slug]          │
└─────────────────────────────────────────────────────────────────┘
```

**Design choices and why:**

- **`index.mdx` is the Docs tab's first page at `/`.** Click the logo, land on Overview, eyebrow says `Getting Started`. Exact together.ai pattern.
- **Quickstart stays in Docs → Getting Started.** It's a "do your first thing" doc — general onboarding content that happens to use the API.
- **Authentication moves to `API → API Reference`** (Sybill / Stripe pattern). Developers expect to find auth next to rate limiting and pagination, not in a general "getting started" group. From Docs Getting Started, the Quickstart links over to it.
- **API tab gains a real Reference group.** Authentication, Rate Limiting, Pagination, Errors — the standard set of topics every serious API reference covers. Mintlify's auto-generation handles the per-endpoint pages from the OpenAPI spec; this group covers everything *around* those endpoints.
- **API Keys becomes Authentication** (`/api/authentication`). The existing `product/api-keys.mdx` content is 90% right; needs a one-paragraph reframe at the top.
- **Endpoints group is auto-generated by OpenAPI tag.** Mintlify groups endpoints by their OpenAPI `tags`. Make sure `api/openapi.yaml` tags endpoints meaningfully (e.g. `Slices`, `Runs`, `Posts`) — the sidebar mirrors those tags one-to-one.
- **Data Models is editorial supplement, not auto-generation.** The OpenAPI spec defines schemas; the Data Models page explains how the entities relate to each other in plain English (a slice produces runs, runs produce posts, posts can be scorecards, etc.). Higher-level than schema-by-schema docs.
- **CRM integration becomes a HubSpot page** under `Integrations`. It's not a concept (slice/scorecard/post are concepts) and the file name should reflect the actual tool, not the generic category. A future Salesforce page sits next to it.
- **Each non-leaf section gets an `Overview` page** where it adds value. `Integrations` has one (lists available integrations); `Accounts` doesn't strictly need one because the four child pages are self-explanatory.
- **Account groups four pages even though only some exist today.** Designing the slots now keeps Docs from getting reshuffled the day each new page lands. Pages can ship one at a time; the IA absorbs them without churn.
- **MCP graduates from a single item to a top-level tab.** Together.ai treats their MCP server as one item because it's peripheral to a much larger product. For OPS, agent integration is a primary user journey, so it earns the tab.

---

## 3. File operations

This restructure is much smaller than the previous plan because most files stay in place.

### Moves (2)

| From | To |
|---|---|
| `product/crm-integration.mdx` | `integrations/hubspot.mdx` |
| `product/api-keys.mdx` | `api/authentication.mdx` |

### Stays in place (9)

- `index.mdx` (root) — renders at `/`, lives in Docs → Getting Started → Overview
- `quickstart.mdx` (root) — Docs → Getting Started → Quickstart
- `product/slices.mdx` — Docs → Product → Slices
- `product/scorecards.mdx` — Docs → Product → Scorecards
- `product/posts.mdx` — Docs → Product → Posts
- `mcp/overview.mdx`, `mcp/claude-code.mdx`, `mcp/claude-desktop.mdx`
- `api/openapi.yaml`

### Net new files

**Ship now (6):**
- `api/introduction.mdx` — what the API is, base URL, response envelope, link to Authentication and Quickstart. ~30 lines.
- `api/rate-limiting.mdx` — rate limits, quota, retry-after handling. Even if OPS doesn't currently rate-limit, ship a page that says so explicitly so developers know.
- `api/pagination.mdx` — cursor or offset pattern used by list endpoints, page size limits, examples.
- `api/errors.mdx` — error envelope shape, list of common error codes and meanings.
- `api/data-models.mdx` — editorial overview of the Slice / Run / Post / Scorecard / Evidence relationships.
- `integrations/index.mdx` — Integrations group landing (1 paragraph + table of available integrations).

**OpenAPI tag check (do at the same time as `api/introduction.mdx`):** open `api/openapi.yaml` and confirm every endpoint has a `tags` field with a sensible group name (e.g. `Slices`, `Runs`, `Posts`, `Scorecards`, `Evidence`). Mintlify groups endpoints in the sidebar by these tags. Untagged endpoints land in a generic group; mistagged endpoints scatter across the sidebar.

**Ship as content lands (4 + N):**
- `accounts/organizations.mdx`
- `accounts/teams.mdx`
- `accounts/members.mdx` *("Members & roles")*
- `accounts/billing.mdx`
- additional `integrations/[slug].mdx` per new integration
- additional `api/data-models/[model].mdx` if you decide to split Data Models into one page per model
- `skills/*.mdx` when Skills tab launches

The IA in §2 references these slots; only list a page in `docs.json` once its MDX file exists — Mintlify will render broken sidebar links for missing files.

### Redirects (2)

```json
"redirects": [
  { "source": "/product/crm-integration", "destination": "/integrations/hubspot" },
  { "source": "/product/api-keys",        "destination": "/api/authentication" }
]
```

`/quickstart`, `/product/slices`, `/product/scorecards`, `/product/posts`, and all `/mcp/*` URLs don't change — no redirects needed.

### Internal links to update

- `index.mdx` — every `<Card href="…">` URL (full rewrite in T3 anyway)
- `product/posts.mdx` line ~28 — `[Scorecards](/product/scorecards)` stays correct; no change needed
- Anywhere referencing `/product/crm-integration` or `/product/api-keys` — update to new paths

---

## 4. Tickets

Eight tickets, sequenced. Each is sized to land in one PR.

### T1 — File moves and directory creation

**Why first:** every later ticket assumes the new paths exist.

**Changes:**
- Create `integrations/`, `accounts/` directories.
- Execute the 2 moves in §3.
- Update internal cross-links inside the moved files for paths that just changed:
  - `product/crm-integration.mdx` → `integrations/hubspot.mdx` — content references aren't path-sensitive, but check
  - `product/api-keys.mdx` → `authentication.mdx` — same

**Acceptance:** `git status` shows clean renames; `mintlify dev` boots without 404s.

---

### T2 — Rewrite `docs.json` to the new IA

**Replace the `navigation` block** (lines 26–85) with:

```json
"navigation": {
  "tabs": [
    {
      "tab": "Docs",
      "groups": [
        {
          "group": "Getting Started",
          "pages": [
            "index",
            "quickstart"
          ]
        },
        {
          "group": "Product",
          "pages": [
            "product/slices",
            "product/scorecards",
            "product/posts"
          ]
        },
        {
          "group": "Integrations",
          "pages": [
            "integrations/index",
            "integrations/hubspot"
          ]
        },
        {
          "group": "Accounts",
          "pages": []
        }
      ]
    },
    {
      "tab": "MCP",
      "groups": [
        {
          "group": "Get started",
          "pages": ["mcp/overview"]
        },
        {
          "group": "Setup",
          "pages": [
            "mcp/claude-code",
            "mcp/claude-desktop"
          ]
        }
      ]
    },
    {
      "tab": "API",
      "groups": [
        {
          "group": "Get started",
          "pages": ["api/introduction"]
        },
        {
          "group": "API Reference",
          "pages": [
            "api/authentication",
            "api/rate-limiting",
            "api/pagination",
            "api/errors"
          ]
        },
        {
          "group": "Endpoints",
          "openapi": "api/openapi.yaml"
        },
        {
          "group": "Data Models",
          "pages": ["api/data-models"]
        }
      ]
    }
  ]
}
```

**A note on the empty `Accounts` group:** Mintlify will hide an empty group from the sidebar. Add page slugs as their MDX files land in subsequent tickets. Don't list pages that haven't been written.

**Also in `docs.json`:**
- Delete `global.anchors` (lines 76–84) — the new structure makes it redundant.
- Add the `redirects` array from §3.
- Confirm `api.baseUrl` (currently `https://app.oneperfectslice.ai`) is still right.

**Acceptance:** sidebar shows `Docs` / `MCP` / `API` tabs in that order; Docs sidebar shows the four groups (Accounts hidden until pages exist); both redirects from §3 return the new URL.

---

### T3 — Rebuild `index.mdx` as the Docs landing

**Full rewrite** (the file is 120 lines; cleaner than diffing):

```mdx
---
title: "Overview"
description: "Run AI-powered analysis on your sales conversations. Surface structured insights, evidence, and scorecards — via the app, API, or MCP."
---

<div className="text-xs font-semibold tracking-widest uppercase mb-2" style={{color: 'var(--primary)'}}>
  Getting started
</div>

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
    slices = resp.json()["data"]["slices"]
    ```

    ```typescript TypeScript
    const resp = await fetch(
      "https://app.oneperfectslice.ai/api/public/v1/slices",
      { headers: { Authorization: "Bearer sk_your_api_key" } }
    );
    const { data } = await resp.json();
    ```

    ```bash cURL
    curl https://app.oneperfectslice.ai/api/public/v1/slices \
      -H "Authorization: Bearer sk_your_api_key"
    ```
  </CodeGroup>
</Columns>

<Columns cols={3}>
  <Card title="Product" img="/images/cards/product.png" href="/product/slices">
    Learn how slices, scorecards, and posts work.
  </Card>
  <Card title="MCP" img="/images/cards/mcp.png" href="/mcp/overview">
    Connect OPS to Claude Code, Claude Desktop, and other agents.
  </Card>
  <Card title="API" img="/images/cards/api.png" href="/quickstart">
    Call OPS from your code — get an API key and run your first request.
  </Card>
</Columns>
```

**Notes:**
- Three cards instead of four because there are three top tabs. The card grid mirrors the tab bar.
- `mode: "wide"` is gone; default content width matches together.ai's hero card proportions.
- The 13-card site map from the old `index.mdx` is fully replaced.
- **The API card points at `/quickstart`** — that page exists today and is the natural API onramp. After T4 ships `/api/introduction`, decide whether to repoint the card or keep it on `/quickstart` (Quickstart is arguably more useful for new users than an Introduction page).

**Acceptance:** home renders eyebrow + side-by-side hero + 3 visual cards; no h2 wall; clicking each card lands on a real page (`/product/slices`, `/mcp/overview`, `/quickstart`).

---

### T4 — Reframe `api/authentication.mdx` and create `api/introduction.mdx`

**`api/authentication.mdx`** (the moved `product/api-keys.mdx`): add a 1-paragraph opener at the top reframing it as the auth doc, not a product feature:

```mdx
---
title: "Authentication"
description: "Authenticate API requests with team-scoped Bearer tokens."
---

Every request to the OPS API must include a Bearer token in the `Authorization` header. Tokens are created in the dashboard and scoped to a single team.

[existing API Keys content from line 8 onward continues here]
```

**`api/introduction.mdx`** (new):

```mdx
---
title: "Introduction"
description: "Programmatic access to One Perfect Slice — slices, runs, posts, and evidence."
---

The OPS Public API is REST-style and returns JSON. Use it to start slice runs, poll for results, search posts, and fetch evidence.

**Base URL:** `https://app.oneperfectslice.ai/api/public/v1`

All requests require a Bearer token in the `Authorization` header — see [Authentication](/api/authentication).

## Response shape

Every endpoint returns a JSON envelope:

\`\`\`json
{
  "data": { ... },
  "error": null
}
\`\`\`

On error, `data` is `null` and `error` contains a machine-readable code and human message — see [Errors](/api/errors).

## Quickstart

For a step-by-step walkthrough — getting an API key and making your first call — see the [Quickstart](/quickstart).
```

**Acceptance:** `/api/authentication` renders the reframed page; `/api/introduction` renders as the API tab landing; `/product/api-keys` redirects to `/api/authentication`.

---

### T4b — Scaffold Rate Limiting, Pagination, Errors, Data Models

**Why a sub-ticket:** these four pages are net-new, OPS-specific reference content. T4a (Authentication + Introduction) has dependencies (the moved file); T4b is independent and can be picked up by anyone with API knowledge.

For each page, ship a real first draft with whatever you actually do today. If a topic doesn't apply yet (e.g., no rate limits in place), say so explicitly — developers need to know whether to plan for it.

**`api/rate-limiting.mdx`** — describe what limits exist, what status code is returned (`429` is standard), what header carries the retry hint (`Retry-After`), and how a client should back off. If OPS has no rate limits today, write:

```mdx
The OPS API does not currently enforce rate limits. We may introduce them in the future; clients should be prepared to handle `429 Too Many Requests` and respect the `Retry-After` header when issued.
```

**`api/pagination.mdx`** — pick one pattern (cursor-based or offset-based), document the request parameters (`limit`, `cursor` or `page`/`page_size`), document the response fields (`next_cursor`, `has_more`, or `total`/`page_count`), and show one full request/response example. Pagination is one of those things that bites every API consumer; clarity here pays back fast.

**`api/errors.mdx`** — list the error envelope shape (already defined in `api/introduction.mdx`), then a table of common error codes:

| Code | HTTP | Meaning |
|---|---|---|
| `unauthorized` | 401 | Missing or invalid Bearer token |
| `forbidden` | 403 | Token valid but lacks access to the requested resource |
| `not_found` | 404 | Resource doesn't exist or isn't visible to this token |
| `validation_error` | 400 | Request body or parameters failed validation |
| `internal_error` | 500 | Server-side issue; please retry |

**`api/data-models.mdx`** — editorial overview, not schema reference. Mintlify already renders schemas from the OpenAPI spec; this page covers relationships:

```mdx
---
title: "Data Models"
description: "How OPS entities relate to each other."
---

OPS organizes data around five entity types:

- **Slice** — a saved analysis configuration (template + filters + scoring rules).
- **Run** — one execution of a slice over a set of evidence at a point in time.
- **Post** — output produced by a run; can be a Summary or a Scorecard.
- **Scorecard** — a structured kind of post, with element-by-element scores.
- **Evidence** — the inputs (calls, transcripts, CRM snapshots) a run reasons over.

A slice produces runs; a run produces posts; a scorecard is one type of post. Evidence is what runs operate on.

For the field-level schema of each model, see the relevant endpoint in [Endpoints](/api/openapi).
```

**Acceptance:** all four pages render in `API → API Reference` and `API → Data Models` groups; cross-links between them resolve.

---

### T5 — Create `integrations/index.mdx` and reframe `integrations/hubspot.mdx`

**`integrations/index.mdx`** (new):

```mdx
---
title: "Integrations"
description: "Connect OPS to your CRM, communication tools, and data sources."
---

OPS integrations sync external context (deals, accounts, contacts) so slices and scorecards can reason about them.

| Integration | Status | Setup |
|---|---|---|
| HubSpot | Available | [Connect](/integrations/hubspot) |
| Salesforce | Coming soon | — |
```

**`integrations/hubspot.mdx`** (renamed from `product/crm-integration.mdx`): the page used to be titled "CRM Integration" and described the abstract concept. As a HubSpot-specific page, lead with HubSpot directly. Suggested top:

```mdx
---
title: "HubSpot"
description: "Connect HubSpot to enrich OPS with deal, company, and contact context."
icon: "plug"
---

Connecting HubSpot lets OPS sync three entity types — deals, companies, and contacts — so slices and scorecards can filter and reason about your pipeline.

[existing CRM Integration content from line 8 onward continues here, with mentions of "CRM" replaced by "HubSpot" where they're talking about HubSpot specifically]
```

**Acceptance:** `/integrations` renders the index; `/integrations/hubspot` renders the reframed HubSpot page; `/product/crm-integration` redirects to `/integrations/hubspot`; the page no longer references "CRM" generically.

---

### T6 — Generate the three gradient hero cards

**Why a separate ticket:** unblocks T3's visual fidelity but T3 will function (with broken `img` paths showing alt text) until this lands.

**Deliverables:** three PNGs at `/images/cards/`, target dimensions 800×400 (2:1).

- `product.png` — teal-to-salmon (`#10a690` → `#F08C80`) — for the Product card
- `mcp.png` — brown-to-deep (`#773624` → `#261B15`) — for the MCP card
- `api.png` — warm salmon-to-brown (`#F08C80` → `#773624`) — for the API card

Tools: Figma export, or any AI image generator with prompt "soft painterly gradient, [color1] to [color2], abstract, no text, 800×400". Together.ai's cards are deliberately abstract — no icons baked in — so the title sits cleanly below the artwork.

**Acceptance:** three images committed; home page renders them; no broken-image alt text.

---

### T6b — Design dependency checks (typography + Mintlify version)

**Why a sub-ticket:** two small decisions that affect the visual match but aren't tied to T6's image work.

**Typography decision** — Yantramanav (current) is humanist and slightly decorative; together.ai uses a geometric sans (Inter-like) which reads cleaner. Two options:
- **Keep Yantramanav.** Your brand voice; structural match to together.ai still comes through. Recommended unless brand says otherwise.
- **Switch to Inter.** Edit `docs.json` `font.headings` and `font.body` to point at Inter from Google Fonts. Closer literal match, less differentiation.

Document the decision somewhere — a one-line note in `docs.json` or `custom.css` is enough. T2 doesn't change either way; the decision just informs whether to swap font URLs in T2.

**Mintlify version check** — together.ai's theme toggle sits at the bottom-left of the sidebar (and the search bar sits at the top of the sidebar instead of in the navbar). Both are Mintlify defaults in recent versions. Run `npx mintlify@latest version` to check, and upgrade if you're behind. Don't write custom CSS to reposition these — let the framework handle it.

**Acceptance:** font decision recorded; Mintlify CLI on a recent version.

---

### T7 — Sidebar / navbar CSS cleanup

**File:** `custom.css`

**Remove** lines 29–82 (the `#sidebar` and `#navbar` blocks that force dark backgrounds). Together.ai uses Mintlify's defaults for these surfaces and only customizes the active-row treatment.

**Replace the active-row CSS with:**

```css
#sidebar a[data-active="true"],
#sidebar .active {
  background: rgba(240, 140, 128, 0.15);
  color: #773624;
  border-radius: 6px;
}

.dark #sidebar a[data-active="true"],
.dark #sidebar .active {
  background: rgba(240, 140, 128, 0.20);
  color: #f08c80;
}
```

**Keep** everything else in `custom.css` (CSS variables, link colors, code block treatment, callouts, tables, method badges).

**Acceptance:** sidebar matches Mintlify default in both modes; active page shows soft salmon pill.

---

### T8 — Enable Ask AI + internal cross-link sweep

**Combined ticket because both are small.**

**Ask AI:** confirm your Mintlify plan includes the AI assistant. Add to `docs.json`:

```json
"contextual": {
  "options": ["copy", "view", "chatgpt", "claude"]
}
```

Flip on the Assistant feature in the Mintlify dashboard. The bottom sticky search input from the together.ai screenshot is this feature.

**Link sweep:** grep across all `.mdx` and update each hit:

- `/product/crm-integration` → `/integrations/hubspot`
- `/product/api-keys` → `/api/authentication`
- `/api-reference` references — none should exist; flag if found

**Acceptance:**
- Ask AI sparkle appears next to search; bottom prompt input renders.
- `grep -rn "/product/crm-integration\|/product/api-keys" .` returns zero results.
- All internal links resolve to 200s in `mintlify dev`.

---

## 5. Sequencing

**Strategy: design first, IA second.** The goal is to make the site visually match together.ai before reorganizing content. T7 alone — killing the dark sidebar/navbar overrides — produces the biggest single visual shift. T3's landing rebuild can ship before the IA changes because every URL it references (`/product/slices`, `/mcp/overview`, `/quickstart`) exists today and survives the IA change unchanged.

**Critical path:** PR 1 → PR 2 lands the full visual match. PR 3 onward is structural reorganization that's invisible to anyone not browsing the sidebar.

**PR batching:**

- **PR 1 (Style refresh):** T7 + T6b
  - T7: CSS cleanup — light sidebar, salmon active pill, drop dark navbar override
  - T6b: typography decision (Yantramanav vs Inter) + Mintlify CLI version verification
  - **After this PR:** sidebar matches together.ai's surface treatment; navbar is light; active row is the salmon pill.

- **PR 2 (Landing):** T3 + T6
  - T6: generate three gradient PNGs at `/images/cards/`
  - T3: rewrite `index.mdx` with eyebrow, two-column hero, three-card grid
  - Card destinations: `/product/slices`, `/mcp/overview`, `/quickstart` (all exist today and persist through later IA changes)
  - **After this PR:** the home page matches together.ai's launchpad pattern. The site visually matches the reference.

- **PR 3 (IA foundation):** T1 + T2
  - T1: 2 file moves (`product/crm-integration.mdx` → `integrations/hubspot.mdx`, `product/api-keys.mdx` → `api/authentication.mdx`)
  - T2: rewrite `docs.json` to the 3-tab structure with redirects
  - **After this PR:** content is in the right tabs/groups. Old URLs redirect.

- **PR 4 (API depth):** T4 + T4b
  - T4: reframe `api/authentication.mdx`, create `api/introduction.mdx`
  - T4b: scaffold `rate-limiting.mdx`, `pagination.mdx`, `errors.mdx`, `data-models.mdx`
  - Decide: repoint the landing's API card from `/quickstart` to `/api/introduction`?
  - **After this PR:** API tab matches the Sybill / Stripe pattern.

- **PR 5 (Integrations):** T5
  - Create `integrations/index.mdx`, reframe `integrations/hubspot.mdx`
  - **After this PR:** Integrations group has a coherent landing.

- **PR 6 (Polish):** T8
  - Enable Ask AI in `docs.json` + Mintlify dashboard
  - Internal cross-link sweep
  - **After this PR:** sticky "Ask a question…" bar renders; zero broken internal links.

Each PR is independently reviewable and won't leave the docs in a broken state.

---

## 6. Account pages — write-as-you-go

After T2, the `Accounts` group exists in `docs.json` but is empty. As you write each page, add its slug to the `Accounts.pages` array:

```json
{
  "group": "Accounts",
  "pages": [
    "accounts/organizations",   // add when written
    "accounts/teams",           // add when written
    "accounts/members",         // add when written
    "accounts/billing"          // add when written
  ]
}
```

Suggested write order (most-frequent question first):
1. `accounts/members.mdx` — "how do I invite someone / what permissions exist"
2. `accounts/organizations.mdx` — "what's the org → team relationship"
3. `accounts/teams.mdx` — "how do I create / manage teams"
4. `accounts/billing.mdx` — "how does billing work, who can see it"

Each page is a small, isolated PR. No further IA decisions needed.

---

## 7. Skills tab — what to design now

You don't ship Skills today, but a small amount of forward-design now keeps the future tab from feeling like a bolt-on:

- **Trigger to ship:** ≥2 `.md` skill files exist, with at least one tested.
- **Tab name:** `Skills` (singular tab, plural content).
- **Tab position:** rightmost — after API.
- **URL prefix:** `/skills/[slug]`.
- **Sidebar shape (when ready):**
  - Group `Get started` — page `Overview` (what these are, how to install)
  - Group `Available skills` — one page per skill
- **Page template per skill:** front-matter title + description + a copy-paste install snippet (e.g., `claude code skill install ops/run-slice`) + the skill's purpose, tools used, example invocation.
- **No tab visible until the trigger is met.** Adding the tab is a 4-line `docs.json` change once you have content.

---

## 8. Open questions to resolve before T1

1. **Hero code snippet** — the `GET /slices` example is the simplest call but not necessarily the most representative. Together.ai chose `chat.completions.create` because it's their flagship endpoint. The OPS equivalent might be starting a slice run (`POST /slices/{id}/runs`) rather than listing them. Worth a 5-minute think before T3.
2. **Footer links** — once `Dashboard`, `Support`, `OpenAPI Spec` leave the home page (T3 drops them), do you want them in the footer? `docs.json`'s `footer` block supports this; the navbar `primary` button already covers Dashboard.
3. **Concepts ordering** — current order is `slices → scorecards → posts`. A new reader might do better with `slices → posts → scorecards` (slice produces posts; scorecards are a kind of post). Defer to your read of the user.
4. **Group ordering inside Docs** — currently `Getting Started → Product → Integrations → Accounts`. Some teams put Accounts before Integrations on the theory that you set up your org before connecting tools. Either works; flag if you want to swap.
5. **OpenAPI tags** — does `api/openapi.yaml` tag every endpoint with a sensible group name (e.g. `Slices`, `Runs`, `Posts`, `Scorecards`, `Evidence`)? Mintlify's `Endpoints` group renders one sub-group per tag. Untagged endpoints will collapse into a generic catch-all. Worth opening the YAML and confirming before T2.
6. **Pagination pattern** — does the OPS API use cursor-based or offset-based pagination today? `api/pagination.mdx` (T4b) needs the actual answer. If neither, decide which the docs should describe and align the API.
7. **Rate limits** — does OPS rate-limit at all today? `api/rate-limiting.mdx` (T4b) needs an answer. The page is short either way; the question is whether to describe limits or describe their absence.

When you're ready, I can start on T1 + T2.
