# MEXC Live Trading Statistics — WordPress Plugin (Public Overview)

How the **private WordPress plugin** presents the MEXC live stats product on LogicEncoder.com.  
No PHP source, API keys, or hosting credentials in this repository.

**Live URL:** [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/)  
**Private implementation:** [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin)

![MEXC Live Trading Statistics Dashboard](./screenshot.png)

---

## Purpose

WordPress is where visitors land. The plugin embeds a **multi-coin realtime dashboard** via shortcode, connects the browser to the private FastAPI backend, and **receives** pre-computed SEO snapshots so search engines index static HTML with Schema.org data—not an empty React/Vanilla shell.

---

## System position

```
Visitor opens page with [mexc_dashboard]
        │
        ├──► Browser ◄──WebSocket/REST──► FastAPI backend (private)
        │
        └──► Crawler ◄──static HTML────► /snapshots/*.html (written by plugin)
                      ▲
                      │ POST JSON (scheduled)
               FastAPI snapshot job
```

Backend overview: [mexc-live-stats-backend-overview](https://github.com/logicencoder/mexc-live-stats-backend-overview).

---

## Live dashboard shortcode

### What

The `[mexc_dashboard]` shortcode renders the full trading UI: searchable coin grid, per-symbol stats cards, live trade list, display modes (ticker / name / both), and mobile layout fixes (controls under search on narrow screens). Assets load only on pages that contain the shortcode (or block embedding it).

### Why it exists

Editors need a one-line embed without custom theme templates. Conditional loading avoids running heavy JS site-wide.

### Who benefits

- **Visitors** — single-page experience for hundreds of MEXC USDT pairs.  
- **Site owners** — drop-in block/shortcode, no child-theme fork.  
- **Operators** — same UI as production with admin connection settings.

### How it fits

Frontend JavaScript speaks MessagePack to backend `/ws` and uses REST bootstrap endpoints configured in plugin settings (URLs are private deploy details).

---

## Backend connection (read path)

### What

The plugin exposes AJAX endpoints so the browser can obtain the WebSocket API key, server monitoring stats, coin full names, and health panels. Admin screens poll snapshot disk status and sitemap generation results.

### Why it exists

Secrets must not be hard-coded in public JS files. WordPress stores keys in options; AJAX delivers them to authenticated sessions or controlled nopriv handlers as implemented.

### Who benefits

- **Visitors** — seamless connect/subscribe flow.  
- **Operators** — see whether backend queues are backing up before users complain.

### How it fits

Read-only toward the backend; the plugin does not ingest MEXC directly.

---

## Snapshot receive and HTML publish

### What

`POST /wp-json/mexc/v1/snapshot` accepts JSON from the Python server: symbol, snapshot type (generic vs exchange-specific), Schema.org bundle, stats object, slug, optional base64 chart image. The plugin renders HTML, writes files under the web root:

- `/snapshots/{slug}.html` — generic price pages  
- `/snapshots/mexc/{slug}.html` — MEXC-branded pages  
- `/snapshots/charts/*.png` — 24h charts  

### Why it exists

WordPress PHP is the stable write target on Hostinger; Python on SOL does the heavy schema and chart work. Splitting compute (SOL) and publish (WP) matches each platform’s strength.

### Who benefits

- **Search engines** — fetch complete documents.  
- **AI crawlers** — read JSON-LD with explicit field names from backend.  
- **Visitors** sharing snapshot URLs — fast TTFB on static files.

### How it fits

Requires matching API key in plugin settings and backend environment—never published in this overview.

---

## Schema.org and FAQ structured data

### What

Rendered snapshot pages embed JSON-LD `Dataset` (and related types) prepared by the backend, plus FAQ blocks where configured. Publisher attribution points to Logic Encoder; citations reference canonical snapshot URLs.

### Why it exists

Rich results and AI summaries reward pages that declare **what** each metric means (not opaque numeric keys).

### Who benefits

- **SEO team** — long-tail price queries.  
- **Researchers** — reproducible cited datasets.

### How it fits

`render_snapshot_html()` in private code merges backend `schema_data` with theme-aligned CSS.

---

## Snapshot sitemap and IndexNow

### What

The plugin can scan snapshot directories, build an XML sitemap of all published coin pages, append the sitemap URL to `robots.txt`, and ping search engines (including IndexNow when a key is configured). Manual “generate” and “ping” buttons exist in admin.

### Why it exists

Hundreds of new URLs will not be discovered quickly without a sitemap. Automation after each snapshot batch keeps index fresh.

### Who benefits

- **Site owner** — indexing without manual Search Console uploads per coin.  
- **Visitors** — find pages via search sooner after new listings go live.

### How it fits

Toggleable via settings (`enable_snapshot_sitemap`, `auto_update_sitemap`, `ping_search_engines`).

---

## SEO meta on snapshot routes

### What

For snapshot URL patterns, the plugin injects canonical links and Open Graph tags using coin full names from MEXC metadata helpers.

### Why it exists

Social shares and search consolidation need correct canonicals between generic and exchange-specific variants.

### Who benefits

- **Social traffic** — correct titles in previews.  
- **SEO** — reduced duplicate-content ambiguity.

### How it fits

Runs on `wp_head` early priority when URI matches snapshot paths.

---

## Admin operator surfaces

### What

- **Settings** — WebSocket URL, API URL, snapshot key, which snapshot types to accept, chart inclusion.  
- **Snapshot dashboard** — file counts, disk usage, directory writability, refresh status AJAX.  
- **Sitemap status** — last generation time, URL counts.  
- **Notices** — success/failure after manual sitemap/ping.

### Why it exists

Operators on shared hosting need visibility without SSH. Counts prove the Python job is still writing files.

### Who benefits

- **Maintainers** — detect 403/permission failures immediately.  
- **Non-developers** — trigger sitemap regen after bulk symbol add.

### How it fits

Requires `manage_options`; no public admin AJAX for destructive actions.

---

## REST settings endpoint for backend

### What

`GET /wp-json/mexc/v1/settings` returns which snapshot types and sitemap options are enabled so the Python job can skip disabled work.

### Why it exists

Avoid generating exchange-specific pages when the site owner disabled them—saves SOL CPU and disk.

### Who benefits

- **Operators** — one place (WP) controls behavior for both sides.  
- **Backend** — respects policy without redeploy.

### How it fits

Public read; no secrets in response body.

---

## Visitor analytics hook

### What

Optional lightweight visitor tracking script (configured in private code) logs dashboard usage events for internal metrics.

### Why it exists

Understand which coins drive engagement without third-party trackers on every page.

### Who benefits

- **Product owner** — prioritize symbol list curation.  
- **Privacy-conscious visitors** — no ad network requirement (implementation-specific).

### How it fits

Loaded from footer script hook on dashboard pages only.

---

## Security model (public summary)

- Snapshot writes require API key header.  
- Admin functions require WordPress administrator capability.  
- Static snapshot files should be served with cache headers appropriate for periodic refresh (handled via directory rules in private code).  
- No exchange API secrets in the plugin—MEXC connectivity is entirely on the backend.

---

## Recent UI iteration (documented behavior)

- Related-coins widget removed from main grid to reduce clutter.  
- Display mode selector kept (Ticker / Name / Both).  
- Mobile: display controls moved below search; spacing fixes for flex layout on small screens.

---

## Who should read this repo

| Audience | Use it to… |
|----------|----------------|
| **Recruiters** | Understand WordPress + realtime backend integration scope |
| **PHP/WordPress engineers** | See REST ingest + static SEO split |
| **SEO specialists** | Map dual snapshot URLs and sitemap flow |
| **Collaborators** | Extend UI without touching MEXC ingest |

---

## Related repositories

| Repo | Role |
|------|------|
| [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin) | Private PHP source |
| [mexc-live-stats-backend-overview](https://github.com/logicencoder/mexc-live-stats-backend-overview) | Ingest + analytics + snapshot generation |
| [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend) | Private backend |

---

## Disclosure

This repository is documentation and screenshots only. Implementation details, hooks, and exact option names are in the private plugin repo and `ARCHITECTURE.md` there.

---

## Author note

The plugin is deliberately **thin on trading logic** and **thick on publishing and UX**—the backend owns the market feed; WordPress owns what Google indexes.
