# `_partials/` — canonical chrome source-of-truth

This directory holds the **canon** for site-wide chrome (header + footer) on the static
Agend Orchestrate site. There is no build step. Pages are committed as final HTML and
the chrome is propagated by hand whenever it changes.

## Files

| File | Purpose |
|---|---|
| `header.html` | Sticky top nav with 5 desktop links + mobile hamburger. Wrapped in `<!-- HEADER START -->`/`<!-- HEADER END -->` markers. |
| `footer.html` | Copyright footer. Wrapped in `<!-- FOOTER START -->`/`<!-- FOOTER END -->` markers. |
| `template.html` | Full page skeleton with `{{TITLE}}`, `{{DESCRIPTION}}`, `{{PAGE_KEY}}`, `{{PAGE_CONTENT}}` placeholders. Bake new pages from this. |
| `README.md` | This file. |

## Marker convention

Every live page MUST contain:

```html
<!-- HEADER START -->
…canonical nav…
<!-- HEADER END -->
```

and

```html
<!-- FOOTER START -->
…canonical footer…
<!-- FOOTER END -->
```

These markers exist so a bulk-update script (or a human with `sed`) can replace the chrome
across all 12 pages atomically without re-parsing each page's body content.

## Active-nav state

The shared inline script reads `<body data-page="X">` and adds
`text-indigo-600 font-semibold` to any `[data-nav="X"]` link. Page keys in use:

| Page | `data-page` value |
|---|---|
| `/index.html` | `home` |
| `/ai-agents/index.html` | `ai-agents` |
| `/ai-agents/shipped/index.html` | `shipped` |
| `/ai-agents/associations/index.html` | `associations` |
| `/ai-agents/autonomous/index.html` | `autonomous` |
| `/ai-agents/executive-director/index.html` | `executive-director` |
| `/ai-agents/membership-manager/index.html` | `membership-manager` |
| `/ai-agents/events-coordinator/index.html` | `events-coordinator` |
| `/ai-agents/cpd-manager/index.html` | `cpd-manager` |
| `/ai-agents/communications-manager/index.html` | `communications-manager` |
| `/ai-agents/compliance-officer/index.html` | `compliance-officer` |
| `/ai-agents/finance-admin/index.html` | `finance-admin` |

## Propagation recipe (when header or footer changes)

1. Edit the canonical file (`_partials/header.html` or `_partials/footer.html`).
2. For every page in the inventory above, replace the block between the START/END
   markers with the new canonical block.
3. Run the banned-string scan:

   ```bash
   grep -ri "claw\|openclaw\|dashboard.agend.dev\|agend dashboard" \
     --include='*.html' . | grep -v docs/superpowers
   ```

   It must return zero hits.
4. Spot-check the homepage, hub, and one persona page in a browser.

## Banned strings

These must never appear anywhere except inside `docs/superpowers/specs/`:

- `Claw` (use `Orchestrate`)
- `OpenClaw` (use `Orchestrate`)
- `dashboard.agend.dev` (use `orchestrate.agend.com.au` or relative paths)
- `Agend Dashboard` (use `Agend Orchestrate`)
- `/blog`, `/changelog`, `/sign-in`, `/sign-up` in nav/footer regions

Any `mailto:` subjects must use `Agend%20Orchestrate`, never `Agend%20Claw`.
