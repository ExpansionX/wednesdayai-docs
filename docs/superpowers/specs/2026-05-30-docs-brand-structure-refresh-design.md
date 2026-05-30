# WednesdayAI Docs — Brand Refresh, Structure Fix & Content Enrichment

_Design spec — 2026-05-30_

## Problem

The live docs site (https://docs.wednesdayai.dev) has two visible deficiencies the user reported:

1. **No User / Admin / Developer sections appear.** The audience split is invisible — the site collapses to a single flat "Getting started" group in the sidebar.
2. **Content looks basic**, especially compared with the much richer `docs/` folder in `WednesdayAI-core` (743 markdown files vs 42 docs-site pages).

### Root cause (diagnosed)

**Missing tabs:** the repo configures Mintlify via `mint.json` — the **deprecated schema**. The current Mintlify platform reads **`docs.json`** (new nested schema). The legacy file's top-level `tabs` / `anchors` / `navigation` arrays are silently ignored for tab rendering, so the three audience tabs never appear even though they are declared. The most recent repo commit ("correct schema format for Mintlify legacy validator") attempted to fix this _within_ the legacy schema — which cannot work; the file format itself is the problem.

The content files already exist and are already split correctly:

```text
admin/        15 pages
users/         8 pages
developers/   14 pages
reference/     4 pages   (api, cli, config, hooks-catalogue)
```

They are simply not being surfaced.

**Basic visuals:** default Mintlify `prism` theme, DM Sans, a flat indigo, no custom CSS, and a minimal placeholder logo. None of the WednesdayAI brand design system is applied.

## Design source

The WednesdayAI Design System handoff bundle (Claude Design export, 2026-05-29). Key foundations used here, from `colors_and_type.css` and the marketing `site.css`:

- **Palette:** Indigo `#6366f1` (primary/CTA/links), `#818cf8` light, `#5457e5`/`#4f46e5` darker; Violet `#8b5cf6`/`#a78bfa` (brick-gradient companion); **Pink `#f472b6` reserved as an accent** for "make it yours" moments — not the global primary. Slate neutrals `#0f172a → #f8fafc`. Emerald `#34d399` success, Amber `#fbbf24` warning.
- **Type:** Space Grotesk (display/headings, tight tracking), Inter (body/UI), JetBrains Mono (code).
- **Mark:** modular-brick "W" — 4 indigo/violet bricks + 1 pink accent brick. Wendy mascot exists but is not needed for docs chrome.
- **Surfaces & feel:** flat slate backgrounds (no marketing gradients in dense UI), 1px low-contrast borders that brighten on hover, subtle card lift (`translateY(-1px)`), 2-ring indigo focus, soft layered shadows, `prefers-reduced-motion` honoured. Corner radii 6/8/12/16px.
- **Tone:** direct, "you/we", Australian English (colour, behaviour, centre), sentence case, emoji sparingly (🧱 signature, ✅ comparisons). Brand name **WednesdayAI** (one word). "Hard fork of OpenClaw at v2026.3.2." Never "LEGO".

## Scope (confirmed with user)

- **PR 1 — Structure + Brand** (ships first, fixes both visible complaints):
  - Migrate `mint.json` → `docs.json` (new nested schema) so Administrators / Users / Developers tabs + Reference render.
  - Apply brand theme via `docs.json` fields + a custom `style.css` (config + CSS depth, no bespoke landing page in this pass).
- **PR 2 — Content enrichment** (separate, after an approved audit):
  - Audit-first gap report, then enrich tab-by-tab (admin → users → developers → reference) from the core `docs/` folder.

A bespoke branded landing/home page is explicitly **out of scope** for now (user chose "config + custom CSS", not "push CSS hard").

## PR 1 — Detailed design

### 1. `docs.json` migration

Replace `mint.json` with `docs.json` (`$schema: https://mintlify.com/docs.json`). Translate the structure:

- Top-level `navigation.tabs[]` — one tab per audience:
  - **Administrators** (`admin/*`) — groups: Getting started, Gateway configuration, Channels, Security, Maintenance.
  - **Users** (`users/*`) — groups: Getting started, Talking to your AI, Built-in tools, Help.
  - **Developers** (`developers/*`) — groups: Getting started, Extension points, AI providers, SDK reference, Contributing.
  - **Reference** (`reference/*`) — promoted to a top-level tab (config, cli, hooks-catalogue, api). _(Decision: a top tab rather than a global anchor — it is first-class reference content, not an external link.)_
- Each group keeps its existing page list verbatim (same file paths). `admin/channels/index` etc. preserved.
- `navbar.links` (GitHub, Discord) and the `Get started` primary button carried over to the `navbar` object.
- Changelog + GitHub external links → `navigation.global.anchors`.
- `search.prompt`, `feedback`, `seo` carried over to their new homes.

The 42 `.mdx` files are not moved or renamed; only the config changes.

### 2. Brand theme

**`docs.json` fields:**

- `colors`: `primary #6366f1`, `light #818cf8`, `dark #4f46e5`; `background.dark #0f172a` (brand slate — already correct, kept).
- `fonts`: heading = **Space Grotesk**, body = **Inter** (replacing DM Sans). Mintlify loads Google Fonts by family name.
- `logo` / `favicon`: regenerated brick-W assets (below).
- Theme: keep a light/dark capable theme; verify the dark mode reads as purple-slate, not black.

**Assets (`assets/`):** regenerate `logo-light.svg`, `logo-dark.svg`, `favicon.svg` as the modular-brick "W" (4 indigo/violet bricks + 1 pink accent brick), matching the design-system mark. Wordmark in Space Grotesk. The existing SVGs are close but will be refined to the canonical brick layout and brand hexes.

**`style.css`** (referenced from `docs.json`), within Mintlify's CSS hooks:

- Heading tracking (Space Grotesk, `-0.02em`/`-0.035em` at large sizes); eyebrow/label treatment.
- Card / card-group hover: border brightens + 1px lift + shadow step.
- Focus: 2-ring indigo `box-shadow`.
- Code blocks: brand-tuned (min-dark/min-light base, indigo selection).
- Callout/note tint using accent-subtle alphas; success/warning semantic colours.
- A restrained brick/indigo radial accent on the hero/header area only — no marketing gradients across body content.
- Pink used sparingly as an accent (e.g. primary-button glow / active states), never as the dominant colour.
- All motion wrapped in `@media (prefers-reduced-motion: no-preference)`.

### 3. Verification

- Run Mintlify local dev (`mint dev` / `mintlify dev`).
- Playwright: confirm the four tabs render across the top, each with its own sidebar groups; confirm brand colours/fonts/logo applied in light and dark; screenshot before/after.
- Open **PR 1** from `feat/docs-json-brand-refresh`.

## PR 2 — Content enrichment (outline only)

1. **Gap audit report** — per page: current depth, specific richer core `docs/` sources, proposed additions. User approves.
2. **Enrich in passes** — admin → users → developers → reference, each independently reviewable. Australian English + brand tone. Accuracy verified against core source; no invented behaviour.
3. Dev log + CHANGELOG per the core repo's PR documentation norms (adapted for the docs repo).

## Risks / decisions

- **Mintlify CSS ceiling.** Themes are config + custom CSS only; arbitrary component overrides aren't supported. The marketing-site look can be _echoed_, not pixel-matched. Accepted.
- **Fonts.** Space Grotesk / Inter load from Google Fonts by family — no TTFs vendored. Matches the design system's own approach.
- **Reference as tab vs anchor.** Chosen as a top tab. Easy to flip to a global anchor if it crowds the navbar.
- **No content moves in PR 1** — pure config/theme, so PR 1 is low-risk and fast to review.
