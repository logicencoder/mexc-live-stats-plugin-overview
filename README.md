# MEXC Live Stats

![MEXC Live Stats — public trading statistics dashboard](assets/live-dashboard.png)

**Live MEXC spot statistics** on Logic Encoder — trade tape, per-pair analytics, coin search, CSV export, and indexable pages for individual USDT markets. Traders and researchers open [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/) for a full-market grid with live prices and volume; each pair also gets its own SEO URL with charts, bot-activity panels, and structured data for search engines.

The hero screenshot shows the real public layout: a **searchable coin list** on the left (1,400+ USDT pairs), a **detail column** for the selected symbol with KPIs and charts, and a **live trade tape** on the right with freeze and CSV export. Nothing here requires a MEXC account — the backend ingests the public protobuf stream once and fans out to every browser, so readers do not open 1,400 exchange sockets themselves.

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

The main app at **[logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/)** is a full-screen market browser. Shortcode **`[mexc_dashboard]`** drops the same shell into any WordPress page; add `symbol="PIUSDT"` to focus one pair in an embed. URL parameter **`?coin=BTC`** (or `BTCUSDT`) deep-links a symbol on first load.

### Layout and coin list

The left column is the **fleet browser**:

- **Search box** — filter 1,400+ rows by ticker or full name as you type.
- **Display mode toggle** — **ticker only**, **name only**, or **both**; preference persists in localStorage for return visits.
- **Sortable columns** — symbol, last price, direction arrow, trades-per-minute hint; click headers to resort when hunting volatile listings.
- **Row click** — switches the active pair everywhere (tape, charts, KPIs) without full page reload.

Each row updates from the shared WebSocket feed — when a pair prints on MEXC, its row lights up across all connected readers simultaneously.

### Detail column — KPIs and charts

The center panel follows the selected symbol:

- **Headline price** with direction vs last print.
- **24h and session stats** — volume, high/low context, trades-per-minute.
- **15-minute price chart** — short-horizon movement for scalpers watching a fresh listing.
- **24h hourly volume chart** — bar chart with hover tooltips splitting **buy vs sell** volume and USDT notional per hour; spot whether flow is one-sided bot accumulation or two-sided retail.

Charts read from in-memory history buffered per symbol on the client, fed by the backend stream — not recomputed from an empty browser cache on first visit.

### Live trade tape

The right column is the **deal tape**:

- Columns: **time**, **price**, **amount**, **USDT notional**, **side** (color-coded buy/sell).
- **Auto-scroll** follows the tail by default; **freeze** (snowflake button) pauses new rows so you can read a burst — queued prints show a badge until you unfreeze.
- **Export menu** — download visible trades as **CSV** for Excel, Google Sheets, or desk Slack; proper escaping and decimal formatting for downstream analysis.

### Visitor analytics

A lightweight analytics script records page engagement (views, time on page) without blocking the WebSocket thread — operators see traffic in wp-admin; visitors see no extra UI chrome.

The browser is **display-only**: PostgreSQL writes, snapshot generation, and sitemap updates all happen server-side so concurrent readers do not hammer your database.

## Per-coin SEO pages

Every tracked symbol gets a canonical URL at **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** — for example `/mexc/BTCUSDT/` or `/mexc/PIUSDT/`. These are not static marketing stubs; **Node SSR** fills each page with live trading data at request time. Crawlers and social bots receive full HTML; humans get the interactive dashboard when appropriate.

A typical coin page includes:

- **KPI strip** — current price, 24h change, volume, and headline metrics above the fold.
- **24h hourly table** — hour-by-hour price and volume so trends are visible without opening the main app.
- **Bot-activity ring** — visual share of flow that looks automated vs manual over the recent window.
- **Schema.org JSON-LD** — a `Dataset` block with 50+ fields (price, volume, buy/sell split, VWAP, utilization, symbol labels) so Google and AI crawlers index real numbers, not placeholder copy.
- **Chart PNG** — optional social preview image generated by the backend snapshot pipeline.

**Routing**: `/mexc/`, `/mexc-app/`, and legacy query forms redirect to canonical `/mexc/{SYMBOL}/` URLs. When SSR is unavailable, static snapshot HTML under `/snapshots/mexc/` still gives crawlers readable content. The WordPress plugin proxies SSR requests and merges backend pushes — operators never edit PHP templates per coin.

## Visitor experience

Public pages connect over a **WebSocket** feed using **MessagePack** frames — smaller on the wire than JSON at 1,400-pair scale.

| Behavior | Detail |
|----------|--------|
| Live tape | Color-coded sides, timestamps, USDT notional on every row |
| Scroll / freeze | Auto-scroll for monitoring; freeze to inspect a wall of prints |
| Pair switch | Search or click; UI resubscribes without navigation |
| Bot-activity | Ring + hourly panels summarize mechanical vs organic flow |
| Precision | Per-symbol price decimals from exchange metadata |

Latency and compression are tuned for wide subscription lists: one backend process ingests MEXC once and broadcasts to all readers instead of each browser opening its own exchange socket.

## Monitor Dashboard

wp-admin **Monitor Dashboard** is the operator nerve center when something looks stale on the public site or trade counts drop after a deploy. Three panels mirror what you would otherwise SSH in to check.

### WebSocket throughput and health

Top status bar shows **API** and **SSR** reachability with response times, plus a green **Connected** badge when the admin monitor socket is live.

The **WebSocket throughput** grid tracks:

- **Messages/sec** and **download KB/s** — is data actually flowing right now?
- **Peak rate** and **total downloaded** — cumulative volume since server start (hundreds of GB on a busy fleet).
- **Reconnects** and **server uptime** — spot flapping connections or recent restarts.
- **Min / avg / current latency** — end-to-end delay from exchange ingest to your browser (spikes here mean UI lag even if MEXC is fine).
- **Compression ratio** — MessagePack + gzip savings vs raw protobuf size.

The **3-hour rolling chart** plots download speed and messages/sec per minute — correlate a traffic drop with a deploy, a MEXC outage, or a subscription gap. If messages/sec flatlines while MEXC is healthy, scroll to System logs for auth or subscribe failures.

![WebSocket monitor — throughput, latency, and API health](assets/websocket-monitor.png)

### Subscribed pairs and server metrics

The **subscribed pairs** chip wall lists every USDT market on the MEXC stream — typically 1,400+ blue tags in alphanumeric order. Scroll it when you suspect a new listing never subscribed.

Four summary cards below:

| Card | What to watch |
|------|----------------|
| **Subscribed pairs** | Tracked vs actively printing (e.g. 1,460 live of 1,482). Footer: aggregate **trades/min** + **MEXC connection healthy**. |
| **Broadcast queue** | Internal fan-out depth — should stay near zero; growth means clients or Postgres writes lag. |
| **PostgreSQL** | Total trades stored, messages sent to browsers, server start time. |
| **Connected clients** | Unique IPs and WebSocket sessions reading right now. |

When public users report “frozen tape” but trades/min is nonzero, check **connected clients** — you may have a front-end issue, not ingest.

![Subscribed pairs and server metrics — MEXC stream, queue, PostgreSQL, clients](assets/server-metrics.png)

### System logs

Dark **system log** console filtered to connection lifecycle — no per-trade spam. Typical boot sequence:

1. Config loaded from server.
2. Admin monitor initializes (MessagePack mode).
3. WebSocket connect attempt with URL and retry count.
4. Authentication success with masked API key prefix.
5. Bulk subscribe to **all** symbols (1,400+ pairs listed, truncated in UI).
6. Server welcome and **successfully subscribed** confirmation.

Use when the public site shows zero trades/min but MEXC itself is up — you will see whether auth failed, subscribe timed out, or the socket never reconnect after a key rotation.

![System logs — WebSocket connect, auth, and symbol subscribe](assets/system-logs.png)

## Coin Manager

wp-admin **Coin Manager** is fleet hygiene for 1,400+ pairs — add new listings, prune delisted symbols, reload from the server bootstrap, and validate SEO output without leaving WordPress.

### Overview and dead coins

Four stat cards summarize fleet health:

| Card | Meaning |
|------|---------|
| **Tracked coins** | Symbols on the server list; footer shows aggregate trades/min. |
| **Live stream** | Pairs receiving trades now; footer counts idle (no print in 60s). |
| **Never traded** | Added but no historical print yet — common right after listing. |
| **Dead / missing** | Delisted on MEXC, removed from stream, or never subscribed. |

The **Dead & missing** panel lists red chips for symbols no longer on the exchange. One click **removes** them so sitemap and SSR do not 404 forever. **Refresh dead list** pulls the latest diff from the backend after a MEXC maintenance window.

![Coin Manager overview — tracked, live, dead, and missing counts](assets/coin-manager-overview.png)

### Add, reload, and tracked list

**Reload symbols on server** syncs the WordPress coin list with the Python bootstrap — run after a backend deploy or MEXC listing batch.

- **Single add** — type `BTCUSDT`, press Add coin.
- **Bulk import** — paste one symbol per line, **Add all** for dozens of new listings.

The **All tracked coins** table is the sortable fleet roster: symbol, **trades/min**, **stream** status (live vs idle), **ever traded** flag, and **Remove** per row. **Refresh now** updates stream stats without reloading the page — confirm a new listing went live within seconds of MEXC enabling the pair.

![All tracked coins — trades per minute, stream status, remove action](assets/coin-manager-tracked.png)

### Schema.org validation

Before you push a new coin to production SEO, pick a symbol and hit **Run validation**. Two layers:

**Python API — server stats**: connected clients, messages sent, latency (min / max / avg), compression ratio, subscription room counts. Confirms the data pipe is warm before you judge HTML.

**Node SSR — HTML structure checks**:

- Bot Activity section present
- 24h hourly table present
- KPI strip present
- Ring gauge (bot) present
- Schema.org JSON-LD block present
- Symbol string in document
- HTML byte size (sanity check against empty fallback pages)

The expandable **JSON-LD key inventory** lists every Schema.org field the SSR emitted — `current_price_usdt`, 24h high/low, buy/sell volume split, VWAP, percentage changes, and dozens more. Operators confirm Google Rich Results will see a real `Dataset`, not a stub page.

![Schema.org validation — Python API and Node SSR structure checks](assets/schema-validation.png)

![Schema.org JSON-LD key listing for a validated coin](assets/schema-jsonld.png)

### Activity log

Timestamped **activity log** records operator actions: coin add/remove, bulk import results, server reload (with POST status), auto-refresh coin-count changes, and success/warning/error color coding. When someone asks “who removed BUTTHOLEUSDT yesterday?” the answer is in this console.

![Coin Manager activity log — add, remove, reload events](assets/coin-manager-activity-log.png)

## Sitemap and IndexNow

Search discovery is first-class — 1,400+ coin URLs would be impossible to submit by hand.

### Sitemap status and IndexNow queue

**MEXC Sitemap & Indexing** dashboard shows:

- **Public sitemap URL** with live HTTP check — confirms WordPress serves a valid XML urlset.
- **WordPress hook** registered and **static file** present in the plugin for fallback.
- **Total URLs** in sitemap (~1,479) and cache timestamp.
- Buttons to **view XML**, open **Google Search Console**, and **save static sitemap file** after bulk coin changes.

**IndexNow queue** panel tracks batch submissions to Bing/Yandex:

- Pending URLs, last batch time/status, sent vs remaining counts.
- **Last push new coins** — how many fresh `/mexc/SYMBOL/` URLs went out after the last listing batch.

![MEXC Sitemap and IndexNow queue status](assets/sitemap-indexing.png)

### Auto-push, coin table, and API sync

- **Auto push on publish** — when a WordPress post or page goes live, its permalink hits IndexNow automatically (status and last URL shown).
- **Coins in sitemap** — paginated table of symbol + `/mexc/SYMBOL/` path — catch typos before Google does.
- **Sync now** — pulls the authoritative coin list from the Python API bootstrap and rebuilds sitemap entries. Run after Coin Manager bulk import or backend listing update.

Typical workflow after MEXC lists new pairs: bulk add in Coin Manager → **Reload symbols on server** → **Sync now** on sitemap → watch IndexNow queue drain → spot-check one symbol in Schema validation.

![IndexNow auto-push, sitemap coin list, and API sync](assets/sitemap-coins-sync.png)

Private code: [mexc-live-stats-plugin](https://github.com/logicencoder/mexc-live-stats-plugin) · live data [mexc-live-stats-backend](https://github.com/logicencoder/mexc-live-stats-backend)

See [REPOS.md](REPOS.md).

---

**Made by [Logic Encoder](https://logicencoder.com)** · [GitHub](https://github.com/logicencoder) · [Contact](https://logicencoder.com/contact/)
