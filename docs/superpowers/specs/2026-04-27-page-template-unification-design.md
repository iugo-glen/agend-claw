# Page template unification — design

**Date:** 2026-04-27
**Status:** Approved by user, dispatching orchestrator immediately.
**Repo:** `agend-orchestrate` (pure static HTML, no build system).

## Goal

Make `index.html` (the homepage) the canonical chrome for every page on the site. Strip the foreign chrome (blog/changelog/sign-in/sign-up + `dashboard.agend.dev` branding + "Agend Dashboard" identity + "OpenClaw Gateway" body copy) inherited from a separate Next.js export. Fix every remaining "Claw" → "Orchestrate" reference. Do this without introducing a build system.

## Decisions

| Decision | Choice |
|---|---|
| Reuse strategy | Copy-paste with documented `_partials/` canon. No build step. |
| Top nav | `For Associations · AI Agents · Autonomous AI · Just Shipped · See Demo` |
| Body content | Preserve faithfully on every page; only chrome gets swapped |
| Tailwind | Keep CDN (matches current homepage) |
| Mobile nav | Add hamburger (homepage currently hides nav below `sm`) |
| Marker convention | Wrap chrome in `<!-- HEADER START/END -->` and `<!-- FOOTER START/END -->` for future bulk-edit ergonomics |
| Active nav state | `<body data-page="X">` + tiny inline script toggles `text-indigo-600` on matching `[data-nav="X"]` |
| Page-1 body | Untouched. Only nav, footer, and Claw refs change. |

## Architecture

7 agents total: 1 orchestrator + 6 specialists across 3 phases.

### Phase 1 — Build canonical template (sequential, blocks Phase 2)
**1× design-foundation specialist.** Produces:
- `_partials/header.html` — full sticky nav with 5 links + mobile hamburger
- `_partials/footer.html` — copyright footer (matches current)
- `_partials/template.html` — full skeleton with `{{TITLE}}` / `{{DESCRIPTION}}` / `{{PAGE_KEY}}` / `{{PAGE_CONTENT}}` placeholders + shared `<style>` block (gradient-text, animate-fade-up, .nav-link, fadeUp keyframes) + inline script for nav active state and mobile menu toggle
- `_partials/README.md` — canon source-of-truth, propagation recipe, marker convention

**Phase gate:** orchestrator validates: 5 nav links present, no "Claw"/"dashboard.agend.dev"/"Agend Dashboard"/"OpenClaw"/"blog"/"changelog"/"sign-in"/"sign-up" anywhere in `_partials/`, mobile menu present, marker comments present.

### Phase 2 — Apply template to all 12 pages (parallel fan-out)
**4× component-development specialists.** Each handles 3 pages.

| Specialist | Pages |
|---|---|
| α | `index.html` (homepage) → `ai-agents/shipped/index.html` → `ai-agents/index.html` (hub) |
| β | `ai-agents/associations` → `autonomous` → `executive-director` |
| γ | `ai-agents/membership-manager` → `events-coordinator` → `cpd-manager` |
| δ | `ai-agents/communications-manager` → `compliance-officer` → `finance-admin` |

α dispatches first; β/γ/δ dispatch shortly after so they can read α's first finished page (homepage) as a worked reference.

**Per-page recipe:**
1. Read existing file (single-line minified for Category B; multi-line for Category A).
2. Locate body content boundaries — for Category B, extract inner HTML of existing `<main>` (or content between existing header and footer).
3. Sanitise extracted body — strip `dashboard.agend.dev` URLs, "Agend Dashboard" text, "OpenClaw Gateway" → "Orchestrate Gateway", any `/blog`, `/changelog`, `/sign-in`, `/sign-up` links.
4. Recover page-specific `<title>` and `<meta name="description">` — rewrite to drop dashboard-branded references.
5. Compose new file = `template.html` skeleton + extracted body + correct `data-page` value.
6. **Self-check:** count `<section>`, `<h1>`, `<h2>` tags before vs after — must match. Report diff in completion summary.
7. For Category A pages (homepage + shipped): leave body untouched, only swap nav/footer markers + fix Claw refs in mailto subjects.

### Phase 3 — Verification (sequential, single agent)
**1× UI auditor.** Walks every page and produces a per-page PASS/FAIL report:
- Identical nav HTML between markers across all 12 pages
- Identical footer HTML between markers across all 12 pages
- No banned strings: "Claw", "OpenClaw", "dashboard.agend.dev", "Agend Dashboard", `/blog`, `/changelog`, `/sign-in`, `/sign-up`
- All mailto subjects use "Agend Orchestrate"
- Section counts in body match pre-rewrite values (sampled from each Category B page)
- `data-page` attribute present and correct on every page

## Acceptance criteria

1. Every page has byte-identical content between `<!-- HEADER START -->` and `<!-- HEADER END -->`.
2. Every page has byte-identical content between `<!-- FOOTER START -->` and `<!-- FOOTER END -->`.
3. `grep -ri "claw\|openclaw\|dashboard.agend.dev\|agend dashboard"` over `*.html` returns 0 hits (excluding the spec file itself).
4. `grep -ri "/blog\|/changelog\|/sign-in\|/sign-up"` over `*.html` returns 0 hits in nav/footer regions.
5. Phase 3 report shows PASS for every page.
6. Original body content of every persona page is preserved (section counts match).

## Risks & mitigations

| Risk | Mitigation |
|---|---|
| Content extraction loses sections from minified Next.js | Section-count self-check per specialist + Phase 3 audit |
| Page-specific CSS that won't survive vanilla Tailwind | Each specialist reports any required custom CSS in completion summary; orchestrator merges into shared `<style>` block if needed |
| `mailto:` subjects in body content (not just nav CTAs) miss the rebrand | Sanitisation pass includes a regex for `mailto:.*Claw` |
| Push to main affects shared state | User explicitly authorised this in the same instruction |
| Worktree drift if orchestrator is interrupted mid-Phase-2 | Phase 1 is atomic; if Phase 2 fails partway, individual page files are independent and the canonical chrome already exists, so retry is safe |

## Out of scope

- No expanded footer (YAGNI; user didn't ask)
- No new pages
- No content rewriting on persona pages
- No build system, package.json, or tooling
- No redirects
- No upstream Next.js source changes (the user can either keep editing exports directly or update upstream separately)
