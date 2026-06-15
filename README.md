# MEXC Live Stats

![MEXC Live Stats — per-coin analytics dashboard](assets/live-dashboard.png)

**Live MEXC spot statistics** on Logic Encoder — per-pair analytics, trade tape, bot-activity scoring, and indexable pages for 1,400+ USDT markets. Traders and researchers open [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/) for the full symbol grid, or land directly on **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** for a deep-dive on one pair. Each coin page is a self-contained analytics dashboard: price header, symbol picker, TradingView chart, market panels, trade statistics, bot detection, live tape, and 24h hourly activity — plus Schema.org data for search engines.

The featured screenshot shows a **per-coin page** layout (example pair in the header): breadcrumb navigation, live price with direction sparkline, collapsible **Active Coins** band, embedded chart, three rows of analysis panels, and a full-width hourly volume chart. No MEXC account required — one backend ingest fans out to every reader.

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

## Fleet browser (`/mexc-app/`)

The **market grid** at [logicencoder.com/mexc-app/](https://logicencoder.com/mexc-app/) is the entry point when you want every pair at once. Shortcode **`[mexc_dashboard]`** embeds the same shell; `symbol="PIUSDT"` focuses one pair; `?coin=BTC` deep-links on load.

- **Search** — filter the full USDT list by ticker or name.
- **Display modes** — ticker only, name only, or both (saved in the browser).
- **Live rows** — last price, direction, trades-per-minute hints; click a row to open the per-coin analytics layout.
- **CSV export** and **freeze** on the trade tape where the embed shows it.

One WebSocket connection updates all readers — the browser displays; PostgreSQL and snapshot pipelines write server-side.

## Per-coin analytics page

Canonical URLs: **[/mexc/{SYMBOL}/](https://logicencoder.com/mexc/)** (e.g. `/mexc/BTCUSDT/`). Node SSR hydrates crawlers; humans get the interactive dashboard below. The same panel set powers SEO JSON-LD — rankings in search reflect live fields, not static copy.

### Symbol header and Active Coins

The top band identifies the **active pair** (full name + USDT ticker), **MEXC** badge, and **Share** link to copy the canonical URL. **Favorite** star adds the pair to a quick-access row.

**Price block** — large last price in USDT, direction arrow, inline **15-minute sparkline**, and **last updated** timestamp so you know the feed is alive.

**Active Coins** — symbol count for the fleet, **search** field, **Ticker / Name / Both** toggle, collapse control, **Favorites** row, and an alphabetical **chip grid** to switch pairs without leaving the page. Selecting a chip rebinds every panel below to the new symbol.

![Symbol header, favorites, and Active Coins grid](assets/per-coin-header-coins.png)

### TradingView chart

Full-width **TradingView** embed for the active pair: timeframe buttons (minutes through hours), indicator and drawing toolbar, candlesticks with volume sub-chart, and exchange label in the chart chrome. Collapse the panel when you want more vertical space for the analytics grid. Symbol in the chart header tracks the chip you picked above.

![TradingView chart — timeframes, indicators, and volume](assets/per-coin-tradingview.png)

### Market overview, volume, and buy/sell panels

Three side-by-side panels summarize **24h market structure**:

**Current Market Overview** — 24h open, price 24h ago, change vs open and real-time change (each with a **−10% / 0% / +10%** sentiment bar), 24h high and low, **VWAP**, and **volatility** with a stable → wild scale.

**Volume Analysis** — total 24h volume in USDT and base asset; **buy vs sell volume** bars and values; **buy/sell volume ratio** on a strong-sell → strong-buy scale; **net flow** in USDT and coins with direction arrow; **whale activity** percentage on a retail → whale dominance scale.

**Buy/Sell Analysis** — 24h trade count, buy and sell counts with percentages, **count ratio**, **trading direction** label and bar (all-sell → balanced → all-buy), and **buy volume %** with matching gauge.

![Market overview, volume analysis, and buy/sell analysis panels](assets/per-coin-market-panels.png)

### Trade analytics, bot activity, and live tape

**Trade Analytics** — average trade size in USDT and base asset with a micro → retail → pro scale; **average interval** between prints (active → slow); **largest trade** size, side, and time; distribution bars for **small** (&lt;$10), **medium** ($10–$100), and **large** (&gt;$100) trade tiers.

**Bot Activity** — **bot score** on a clean → bot scale; **repeat size**, **repeat interval**, **round lot**, and **burst score** each with organic → suspect → likely → confirmed style bars; qualitative labels summarize mixed algo/human flow.

**Real-Time Trades** — scrolling ledger: time, symbol, price, amount, USDT notional, color-coded **BUY/SELL**; side stripe on each row; **freeze** and **export** controls in the panel header.

![Trade analytics, bot activity scoring, and real-time trades tape](assets/per-coin-analytics-tape.png)

### 24h hourly activity chart

Full-width **24H Trading Activity** bar chart: each hour shows **buy** (green) and **sell** (red) base-asset volume; hover/tooltip exposes hour label plus buy and sell totals in coin and USDT. Use it to see which hours were one-sided vs two-sided without exporting CSV.

![24h hourly trading activity — buy vs sell volume by hour](assets/per-coin-24h-activity.png)

### SEO and routing

Crawlers receive SSR HTML + **Schema.org JSON-LD** (`Dataset` with price, volume, buy/sell split, VWAP, and dozens of trading fields). `/mexc/`, `/mexc-app/`, and legacy query URLs redirect to canonical `/mexc/{SYMBOL}/`. Static snapshots under `/snapshots/mexc/` backfill when live SSR is down.

## Visitor experience

Public pages use a **MessagePack WebSocket** feed — compact at 1,400-pair scale. Pair switch resubscribes in-page; tape **freeze** pauses auto-scroll; **CSV export** dumps visible rows. Per-symbol price precision comes from exchange metadata on the backend.

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
| **Subscribed pairs** | Tracked vs actively printing counts. Footer: aggregate **trades/min** + **MEXC connection healthy**. |
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
