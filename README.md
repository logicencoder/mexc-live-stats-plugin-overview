# MEXC Live Stats

![MEXC Live Stats — public trading statistics dashboard](assets/live-dashboard.png)

**Live MEXC spot statistics** on Logic Encoder — trade tape, per-pair analytics, coin search, CSV export, and indexable pages for individual USDT markets. Traders and researchers open [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/) for a full-market grid; each pair also gets its own SEO URL with charts, bot-activity panels, and structured data for search engines.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, shortcodes, wp-admin dashboards, WordPress AJAX, WordPress REST ingest |
| Public UI | HTML, CSS, vanilla JavaScript, MessagePack WebSocket client |
| Live backend | Python 3, FastAPI, MEXC protobuf WebSocket ingest, PostgreSQL |
| SEO SSR | Node.js Express — per-symbol HTML + Schema.org JSON-LD |
| Snapshots | Static HTML/JSON + chart PNGs pushed from backend |
| Search | XML sitemap for `/mexc/{SYMBOL}/`, IndexNow queue, auto-push on publish |
| Data | PostgreSQL trade history on backend; WordPress options for coin lists and SEO state |
| Hosting | WordPress on shared hosting; Python/Node services on self-hosted Linux servers |

## Live dashboard

Shortcode **`[mexc_dashboard]`** embeds the main app on any WordPress page — coin grid with search, display modes (ticker / name / both), live trades, charts, and visitor analytics. Optional `symbol="PIUSDT"` focuses one pair in the embed.

## Per-coin SEO pages

Canonical URLs at **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** — Node SSR renders KPI strips, 24h hourly tables, bot-activity rings, and **Schema.org JSON-LD** with dozens of trading fields per pair. Static snapshot HTML and chart PNGs support crawlers and social previews when live SSR is unavailable.

## Visitor experience

Real-time **WebSocket** feed with MessagePack frames — scroll or freeze the trade tape, watch bot-activity indicators, and open pair charts backed by PostgreSQL history on the backend. The browser displays; the server owns all writes.

## Monitor Dashboard

wp-admin **Monitor Dashboard** — throughput, latency, compression ratio, subscribed pair chips, PostgreSQL trade counts, broadcast queue depth, and connected client stats. System logs show connect, auth, and bulk symbol subscribe without trade spam.

<img src="assets/websocket-monitor.png" alt="WebSocket monitor — throughput, latency, and API health" width="860" />

<img src="assets/server-metrics.png" alt="Subscribed pairs and server metrics — MEXC stream, queue, PostgreSQL, clients" width="860" />

<img src="assets/system-logs.png" alt="System logs — WebSocket connect, auth, and symbol subscribe" width="860" />

## Coin Manager

wp-admin **Coin Manager** — fleet hygiene for 1,400+ USDT pairs: tracked vs live-stream counts, dead/missing symbols, single and bulk add, server reload, per-coin trades/min and stream status, and one-click remove.

<img src="assets/coin-manager-overview.png" alt="Coin Manager overview — tracked, live, dead, and missing counts" width="860" />

<img src="assets/coin-manager-tracked.png" alt="All tracked coins — trades per minute, stream status, remove action" width="860" />

Built-in **Schema.org validation** runs Python API stats and Node SSR HTML checks for any symbol — KPI strip, hourly table, ring gauge, JSON-LD presence, and key inventory for SEO QA.

<img src="assets/schema-validation.png" alt="Schema.org validation — Python API and Node SSR structure checks" width="860" />

<img src="assets/schema-jsonld.png" alt="Schema.org JSON-LD key listing for a validated coin" width="860" />

**Activity log** records add, remove, bulk import, reload, and auto-refresh events in a dark console for operator audit.

<img src="assets/coin-manager-activity-log.png" alt="Coin Manager activity log — add, remove, reload events" width="860" />

## Sitemap and IndexNow

**MEXC Sitemap & Indexing** — public XML sitemap for all `/mexc/SYMBOL/` URLs, live HTTP verification, IndexNow batch queue, manual URL submit, and sync from the backend coin bootstrap.

<img src="assets/sitemap-indexing.png" alt="MEXC Sitemap and IndexNow queue status" width="860" />

Auto-push on WordPress publish, coins-in-sitemap table with per-pair URLs, and **Sync now** to refresh the coin list from the Python API.

<img src="assets/sitemap-coins-sync.png" alt="IndexNow auto-push, sitemap coin list, and API sync" width="860" />

Private code: [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin) · live data [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
