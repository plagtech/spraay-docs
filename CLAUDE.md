# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Static documentation site for the **Spraay x402 Gateway** — a catalog of pay-per-call HTTP API "primitives" (endpoints) that AI agents and developers invoke, paying USDC per request via the x402 protocol on Base & Solana. Served at `docs.spraay.app` (see `CNAME`) from the Cloudflare Pages project `spraay-docs`; the git remote is `plagtech/spraay-docs`. There is no build step, framework, or package manager — the site is hand-authored HTML/CSS/vanilla JS.

## Structure

- `index.html` — the entire catalog in one ~2700-line file: inline `<style>`, hand-written endpoint cards, and a small inline `<script>` at the bottom. This is where nearly all edits happen.
- `bpa/1.0/index.html` — standalone "Batch Payments for Agents (BPA) 1.0" specification page.
- `bpa/1.0/disbursement-request.schema.json` — JSON Schema (draft 2020-12) for a BPA disbursement request; `$id` is the canonical `docs.spraay.app` URL, so keep it in sync if the path moves.
- `CNAME` — custom domain for GitHub Pages. Do not remove.

## Anatomy of `index.html`

The document is a fixed sidebar + scrolling content column. Three coupled regions must stay consistent:

1. **`<nav class="sidebar">`** — grouped under `sidebar-heading` sections. Each category link is `<a href="#cat-XXX">` with a `side-badge` showing that category's endpoint count.
2. **`docs-hero` stats** (`#overview`) and the `<head>` `<meta>`/`og:` descriptions — carry the aggregate totals (primitives, paid, free, categories).
3. **`category-section` blocks** (`id="cat-XXX"`) — each has a `category-header` with a `cat-count` ("N endpoints") and an `endpoint-grid` of `endpoint-card`s.

### Endpoint card pattern

Every endpoint is one `<div class="endpoint-card" id="..." onclick="toggleCard(this)">` containing:
- a `card-summary` (always visible): method pill (`method-get`/`method-post`), `card-name`, `card-desc`, `card-endpoint` path, `card-price` in USDC, and a `card-status status-live` dot.
- a `card-detail` (revealed on click) that repeats the description in `detail-meta` and shows a `detail-code` request snippet.

The description text is intentionally duplicated between `card-desc` and the `detail-section` — edit both when changing copy. The card `id` is the endpoint path slugified with dashes (e.g. `/api/v1/chat/completions` → `api-v1-chat-completions`); anchor navigation depends on it being unique.

### The inline script (bottom of `index.html`)

Small and dependency-free: `toggleCard` (accordion, one card open at a time), sidebar open/close for mobile, an `IntersectionObserver` that highlights the active sidebar link on scroll, and `copyQuickstart`. The observer watches elements matching `.endpoint-card[id], .docs-hero[id], .quickstart[id], .integrations-section[id], .category-section[id]` — any new anchored section should carry an `id` to participate.

## Keeping counts in sync (the main correctness concern)

The recent commit history is dominated by adding/removing primitives and re-tallying (`157->145 primitives, 139 paid, 38 categories`). When you add, remove, or recategorize an endpoint, update **all** of these so the site stays truthful:

- the hero `hero-stat` totals (primitives / paid / free / categories) in `#overview`
- the `<title>`, `<meta name="description">`, and `og:` description in `<head>`
- the affected category's `cat-count` ("N endpoints")
- the matching sidebar `side-badge` number (and add/remove the sidebar `<a>` and `sidebar-section` if a category is added/removed)

"Free" endpoints are those under the free tier (paths like `/api/v1/free/...`); everything else is paid. Free vs. paid affects the paid/free totals.

## Previewing

No server required — open `index.html` directly in a browser, or serve the folder statically (e.g. `python -m http.server`) if you need correct absolute-path behavior for `/bpa/`. There are no tests, linters, or CI in the repo; verification is visual.

## Deployment

Hosting is the Cloudflare Pages project `spraay-docs` (domains `docs.spraay.app`, `spraay-docs.pages.dev`). It is **not** git-connected: pushing to GitHub does nothing on its own. Deploys are manual, via `npx wrangler pages deploy` (`npx wrangler login` first if `npx wrangler whoami` says you are not authenticated). Only commit/push/deploy when asked.

**Never deploy the repo folder directly.** `wrangler pages deploy` uploads every file in the directory, and this wrangler line (tested on 4.128.0) does not honor `.wranglerignore` for Pages, so a direct deploy publishes this `CLAUDE.md` at `docs.spraay.app/CLAUDE.md`. The standard deploy step is to export the committed tree to a temp copy, drop `CLAUDE.md`, and deploy that copy (Git Bash / POSIX shell):

```sh
DEPLOY=$(mktemp -d) && git archive HEAD | tar -x -C "$DEPLOY" && rm -f "$DEPLOY/CLAUDE.md"   && npx wrangler pages deploy "$DEPLOY" --project-name spraay-docs --branch main --commit-dirty=true   && rm -rf "$DEPLOY"
```

Because the export comes from `HEAD`, commit first; uncommitted edits are not deployed. After deploying, confirm `docs.spraay.app/CLAUDE.md` returns 404 (the edge may serve a cached copy for a short while) and that `/`, `/llms.txt` and `/bpa/1.0/` still serve. Wrangler leaves a `.wrangler/` cache directory in the folder it runs from; delete it rather than committing it.
