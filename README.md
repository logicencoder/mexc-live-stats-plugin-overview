# MEXC Live Stats

![MEXC Live Stats — live trading app at logicencoder.com/mexc-app](assets/live-dashboard.png)

**Live MEXC spot statistics** on Logic Encoder — the interactive product lives at **[logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/)**, not on static article pages. Open the app, pick any USDT pair from the chip grid (or land with **`?coin=SYMBOL`** such as [ETHUSDT](https://logicencoder.com/mexc-app/?coin=ETHUSDT)), and get realtime trade tape, bot-activity scoring, TradingView chart, 24h analytics panels, hourly volume chart, favorites, export, and freeze — all in one WordPress-embedded shell. Shortcode **`[mexc_dashboard]`** drops the same UI on any page; **`symbol="PIUSDT"`** sets the default pair.

## Tech stack

| Layer | Technologies |
|-------|--------------|
| WordPress plugin | PHP, shortcodes, wp-admin dashboards, WordPress AJAX, WordPress REST ingest |
| Public UI | HTML, CSS, vanilla JavaScript, Chart.js, TradingView embed, MessagePack WebSocket client |
| Live backend | Python 3, FastAPI, MEXC protobuf WebSocket ingest, PostgreSQL |
| SEO SSR | Node.js Express — crawler HTML + Schema.org JSON-LD (separate from the live app) |
| Snapshots | Static HTML/JSON + chart PNGs pushed from backend |
| Search | XML sitemap for SEO URLs, IndexNow queue, auto-push on publish |
| Data | PostgreSQL trade history on backend; WordPress options for coin lists and SEO state |
| Hosting | WordPress on shared hosting; Python/Node services on self-hosted Linux servers |

## Shared hosting, heavy work on the server

Logic Encoder publishes the MEXC dashboard on **WordPress shared hosting** — the right layer for shortcodes, sitemaps, IndexNow, and cached snapshot HTML, but the wrong place to absorb a full-exchange trade firehose. From the start the goal was to **keep WordPress thin**: PHP renders the shell, stores coin lists and SEO state, and receives finished payloads. Ingest, aggregation, PostgreSQL, MessagePack fan-out, chart generation, and SSR data bundles run on a **self-hosted Linux server** I operate separately — not inside shared-hosting PHP workers.

That split was a deliberate optimization challenge on tight Hostinger limits. **More than 1,400 USDT spot pairs** run on the live MEXC install today; the same architecture carries **roughly 800 Gate.io pairs** on the sibling Gate stats product. Visitors still get realtime tapes and rolling analytics in the browser; WordPress mostly **displays and indexes** what the backend already computed. Updates keep flowing over WebSocket with REST and transient mirrors as fallback — the site stays current without moving heavy math back onto shared hosting.

The hosting control panel agrees: CPU, memory, PHP workers, disk throughput, IOPS, and concurrent process charts sit well below plan ceilings while both fleets are active — the outcome I was aiming for after offloading work, tuning batch paths, and trimming what PHP has to touch.

![Hostinger shared hosting — CPU and memory usage vs plan limits](assets/hostinger-cpu-memory.jpg)

![Hostinger shared hosting — disk throughput and PHP worker count](assets/hostinger-throughput-workers.jpg)

![Hostinger shared hosting — storage IOPS and max processes](assets/hostinger-iops-processes.jpg)

## Live trading app (`/mexc-app/`)

The dashboard at [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/) is a **single-page trading console**. Every control below is implemented in the plugin JavaScript (`mexcDashboard`); data comes from the self-hosted backend over WebSocket and REST.

### Realtime connection

- **WebSocket** — MessagePack frames carry `mexc_trade` ticks and periodic `mexc_stats` aggregates. Trades for the **active symbol** update the hero price, direction arrow, sparkline, tape rows, and the current hour on the 24h bar chart in the same event loop — no polling for headline price.
- **REST fallback** — `GET /api/stats/memory/{symbol}` loads full 24h stats when you switch pairs or when bootstrap has no cache yet; hero **last price** seeds from `current_price` (last trade in the rolling window) immediately, then WebSocket trades take over.
- **Subscription** — switching chips sends a new `subscribe` action for the active pair; stats for other symbols are ignored so panels never show cross-contaminated data.

### Symbol header and navigation

- **Breadcrumb** — Home › MEXC › current pair name (updates on symbol switch).
- **Title row** — favorite star, **full name — TICKER**, MEXC badge, **Share** button.
- **Share** — copies the current page URL with `?coin=SYMBOL` to clipboard; button flashes confirmation feedback.
- **Hero price** — large USDT last trade; **direction arrow** (↑ buy tint, ↓ sell tint) follows the latest print side.
- **15-minute sparkline** — Chart.js line beside the price, fed from per-symbol price history in memory; **hover tooltip** on the mini chart shows the sampled price at that point.
- **Last updated** — clock time of the last applied tick.

### Favorites

- **Header star** — toggle favorite for the active pair.
- **Per-chip star** on each coin button in the grid — same list.
- **Favorites row** — dedicated chip strip above the full grid when you have favorites; hidden when empty.
- **localStorage** — `mexc_favorites` persists across sessions in the browser; wp-admin exposes **max favorites** (default cap stored in `mexc_max_favorites`).

### Active Coins picker

- **Symbol count** in the panel title — fleet size from backend bootstrap.
- **Search** — filters chips by ticker or full name; **×** clears input.
- **Display mode** — **Ticker**, **Name**, or **Both** on chips; saved in `mexc_coin_display_mode` localStorage.
- **Collapse (−/+)** — hides chip grid, search, and display toggle; state in `mexc_panel_collapse`.
- **Chip grid** — alphabetical USDT pairs; active chip highlighted; one click rebinds all panels and updates the URL bar without reload.

![Symbol header, favorites, and Active Coins grid](assets/per-coin-header-coins.png)

### TradingView chart

Embedded **TradingView** widget (`MEXC:SYMBOL`): timeframe toolbar, indicators, drawings, candlesticks, volume sub-chart. Panel header shows the active symbol. **Collapse** shrinks iframe height to zero and remembers preference. Iframe `src` reloads on every symbol switch.

![TradingView chart — timeframes, indicators, and volume](assets/per-coin-tradingview.png)

### 24h analytics panels

Three side-by-side panels summarize **rolling 24h structure** for the active pair (stats broadcast + REST):

**Current Market Overview** — 24h open, price 24h ago, change vs open and real-time change (each with a **−10% / 0% / +10%** sentiment bar and context label), 24h high and low, **VWAP**, **volatility** on a stable → wild scale.

**Volume Analysis** — total 24h volume in USDT and base asset; **buy vs sell volume** bars; **buy/sell volume ratio** on strong-sell → strong-buy scale; **net flow** in USDT and coins with direction arrow; **whale activity** on retail → whale dominance scale.

**Buy/Sell Analysis** — 24h trade count, buy/sell counts and percentages, **count ratio**, **trading direction** label and bar (all-sell → balanced → all-buy), **buy volume %** gauge.

![Market overview, volume analysis, and buy/sell analysis panels](assets/per-coin-market-panels.png)

### Trade analytics, bot activity, and live tape

**Trade Analytics** — average trade size in USDT and base asset (micro → retail → pro scale); **average interval** between prints (active → slow); **largest trade** size, side, and time; distribution bars for **small**, **medium**, and **large** USDT tiers.

**Bot Activity** — **bot score** on clean → bot scale; **repeat size**, **repeat interval**, **round lot**, and **burst score** each with organic → suspect → likely → confirmed style bars and text labels.

**Real-Time Trades** — live scrolling ledger: time, symbol, price, amount, USDT notional, color-coded **BUY/SELL** with side stripe.

| Control | Behavior |
|---------|----------|
| **Realtime feed** | New rows append at the bottom; if you are at the bottom, the tape **follows the tail**; scroll up to read without auto-jump. |
| **Freeze ❄️** | Pauses live inserts; trades buffer in memory; button shows pending count; unfreeze flushes in order. |
| **Export ↓** | **TXT**, **CSV**, or **JSON** — merges PostgreSQL history (rolling window) with in-memory WebSocket rows, deduplicated; download filename includes symbol and timestamp. |

Only trades matching the **active symbol** update the tape and hero; other symbols still contribute to sparkline history caches.

![Trade analytics, bot activity scoring, and real-time trades tape](assets/per-coin-analytics-tape.png)

### 24h Trading Activity chart

Full-width **hourly bar chart** — green **buy** and red **sell** base-asset volume per clock hour for the active pair.

- **Fixed tooltip strip** above the chart — hour label, buy volume, sell volume (coin + USDT) with color swatches; defaults to the latest hour after load.
- **Hover / scrub** — moving across bars updates the fixed tooltip (Chart.js built-in tooltip disabled for stable mobile layout).
- **Live hour bar** — current hour grows as new trades arrive via `updateCurrentHourBar`.
- **Collapse** — same localStorage persistence as coins grid and TradingView.

![24h hourly trading activity — buy vs sell volume by hour](assets/per-coin-24h-activity.png)

### Symbol switch workflow

Clicking a chip (or loading `?coin=`): clears tape and stats cache, resets sparkline datasets, reloads TradingView, fetches REST stats for immediate panel + hero price fill, re-subscribes WebSocket, and backfills the tape from DB + WS. URL updates via `history.replaceState` so Share stays accurate.

## Monitor Dashboard

wp-admin **Monitor Dashboard** is the operator nerve center when something looks stale on the public site or trade counts drop after a deploy.

### WebSocket throughput and health

Top status bar shows **API** and **SSR** reachability with response times, plus a green **Connected** badge when the admin monitor socket is live.

The **WebSocket throughput** grid tracks messages/sec, download KB/s, peak rate, total downloaded, reconnects, server uptime, min/avg/current latency, and compression ratio. The **3-hour rolling chart** plots download speed and messages/sec per minute.

![WebSocket monitor — throughput, latency, and API health](assets/websocket-monitor.png)

### Subscribed pairs and server metrics

**Subscribed pairs** chip wall — dense symbol tags in alphanumeric order. Summary cards: **Subscribed pairs**, **Broadcast queue**, **PostgreSQL**, **Connected clients**.

![Subscribed pairs and server metrics — MEXC stream, queue, PostgreSQL, clients](assets/server-metrics.png)

### System logs

Dark **system log** console for connection lifecycle: config load, monitor init, WebSocket connect/retry, auth, bulk subscribe, welcome message.

![System logs — WebSocket connect, auth, and symbol subscribe](assets/system-logs.png)

## Coin Manager

wp-admin **Coin Manager** — fleet hygiene across every tracked USDT pair.

### Overview and dead coins

Stat cards: **Tracked coins**, **Live stream**, **Never traded**, **Dead / missing**. **Dead & missing** panel with one-click remove and **Refresh dead list**.

![Coin Manager overview — tracked, live, dead, and missing counts](assets/coin-manager-overview.png)

### Add, reload, and tracked list

**Reload symbols on server**, single add, bulk import, sortable **All tracked coins** table with trades/min, stream status, remove action.

![All tracked coins — trades per minute, stream status, remove action](assets/coin-manager-tracked.png)

### Schema.org validation

**Run validation** — Python API health + Node SSR structure checks (bot section, hourly table, KPI strip, JSON-LD, symbol in document). Expandable **JSON-LD key inventory**.

![Schema.org validation — Python API and Node SSR structure checks](assets/schema-validation.png)

![Schema.org JSON-LD key listing for a validated coin](assets/schema-jsonld.png)

### Activity log

Timestamped operator actions with success/warning/error color coding.

![Coin Manager activity log — add, remove, reload events](assets/coin-manager-activity-log.png)

## Sitemap and IndexNow

**MEXC Sitemap & Indexing** — public sitemap URL check, WordPress hook, static file fallback, IndexNow queue, auto-push on publish, paginated coin table, **Sync now** from backend bootstrap.

![MEXC Sitemap and IndexNow queue status](assets/sitemap-indexing.png)

![IndexNow auto-push, sitemap coin list, and API sync](assets/sitemap-coins-sync.png)

## Search discovery (crawlers only)

Humans use **`/mexc-app/`**. Separate **SEO URLs** under `/mexc/{SYMBOL}/` serve crawler HTML + Schema.org JSON-LD from Node SSR — useful for search, not the interactive product. Static snapshots under `/snapshots/mexc/` backfill when SSR is unavailable.

Private code: [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin) · live data [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend)

Backend overview: [mexc-live-stats-backend-overview](https://github.com/logicencoder/mexc-live-stats-backend-overview)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
