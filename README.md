# MEXC Live Stats

![MEXC Live Stats — public trading statistics dashboard](assets/live-dashboard.png)

Public **MEXC spot trading statistics** on logicencoder.com — live trade tape, per-symbol analytics, coin search, CSV export, and indexable SEO pages for individual USDT pairs.

Private plugin: [logicencoder/mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin). Data pipeline: [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend) — [backend overview](https://github.com/logicencoder/mexc-live-stats-backend-overview).

## Live dashboard

**[logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/)**

Shortcode **`[mexc_dashboard]`** — coin grid with search, display modes (ticker / name / both), live trades, charts, visitor analytics script.

## Per-coin SEO pages

- Canonical SSR: **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** (Node SSR via backend)
- Static snapshot HTML: `/snapshots/` and `/snapshots/mexc/` (backend POST pipeline)
- Chart PNGs: `/snapshots/charts/` for social previews

## Visitor experience

Real-time WebSocket from **`wss://ws.logicencoder.com/ws`** (msgpack frames). Trade tape with scroll/freeze, bot-activity panels, pair charts backed by PostgreSQL history on the backend.

## WordPress admin

Settings: WebSocket URL, REST API URL, snapshot toggles, sitemap / IndexNow integration. Snapshot dashboard shows file counts and disk usage. Manual sitemap generate and search-engine ping actions.

Built-in **WebSocket monitor** — throughput, latency, subscribed pairs, PostgreSQL trade counts, broadcast queue, and connection logs for operators debugging the live feed.

<img src="assets/websocket-monitor.png" alt="WebSocket monitor — throughput, latency, and API health" width="860" />

<img src="assets/server-metrics.png" alt="Subscribed pairs and server metrics — MEXC stream, queue, PostgreSQL, clients" width="860" />

<img src="assets/system-logs.png" alt="System logs — WebSocket connect, auth, and symbol subscribe" width="860" />

**REST (`mexc/v1`):**

| Route | Role |
|-------|------|
| `POST /snapshot` | Backend pushes SEO JSON + optional chart PNG |
| `GET /settings` | Snapshot config for backend worker |
| `GET /monitoring` | Admin health proxy |

**AJAX:** API key, server stats, monitoring, snapshot status, sitemap status, coin full name (admin + nopriv where wired).

## Data flow

MEXC protobuf WebSocket → Python ingest → PostgreSQL → fan-out to `/ws`, REST stats, and snapshot POST to WordPress. Distinct from **`mexc_trading_app`** (local order-placement console).

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
