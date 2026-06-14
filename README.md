# MEXC Live Stats

Public **MEXC spot trading statistics** dashboard on logicencoder.com — live trade tape, per-symbol 24h stats, charts, coin search, and indexable SEO pages for individual pairs.

Private plugin: [logicencoder/mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin). Data pipeline: [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend) — [backend overview](https://github.com/logicencoder/mexc-live-stats-backend-overview).

## Live dashboard

**[logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/)**

Shortcode **`[mexc_dashboard]`** — coin grid with search, display modes (ticker / name / both), live trades, analytics panels, CSV export.

## Per-coin SEO pages

- SSR routes: **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** (Node canonical)
- Static snapshot HTML under `/snapshots/` and `/snapshots/mexc/` (backend POST from live stats)
- Chart PNGs at `/snapshots/charts/` for social and search previews

## Visitor experience

Real-time WebSocket feed from `wss://ws.logicencoder.com/ws` — scroll or freeze trade tape, bot-activity indicators, pair-level charts backed by PostgreSQL history on the backend.

## WordPress admin

Settings: WebSocket URL, API URL, snapshot toggles, sitemap / IndexNow. Snapshot dashboard (file counts, disk usage), manual sitemap generate and search-engine ping. REST endpoints for backend snapshot push and monitoring proxy.

## Data flow

MEXC protobuf WebSocket → Python ingest → PostgreSQL → fan-out to browser clients and snapshot pipeline → WordPress static SEO files.

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
