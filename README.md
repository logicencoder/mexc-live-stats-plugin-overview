# MEXC Exchange - Live Trading Statistics WordPress Plugin (Overview)

Public overview repository for the private WordPress plugin implementation.

> This repository documents capabilities and integration architecture only. Private implementation code is intentionally not published.

## Positioning

This plugin is the live WordPress delivery layer for `https://logicencoder.com/mexc-app/`.
It renders the operator dashboard UI, coordinates data fetch from the private backend, and receives snapshot payloads for SEO-indexable content pages.

- Current delivery mode: live WordPress plugin + private FastAPI backend
- Live URL: `https://logicencoder.com/mexc-app/`
- Source exposure mode: capabilities and integration architecture only

## UI Snapshot

![MEXC Live Trading Statistics Dashboard](./screenshot.png)

## Product Summary

The plugin solves one practical frontend problem: provide a responsive, operator-friendly trading statistics interface inside WordPress without exposing backend internals.

## Tech Stack Used

- **Plugin runtime**: PHP + WordPress plugin APIs
- **Frontend delivery**: shortcode-rendered dashboard (`[mexc_dashboard]`) + JavaScript runtime
- **Integration**: WordPress AJAX + REST routes + backend API/WebSocket consumption
- **SEO publishing**: snapshot ingest + sitemap and metadata management

## High-Level Responsibilities

- Render live dashboard UI through shortcode page embedding
- Fetch and display backend statistics in near real time
- Expose snapshot ingest REST endpoint (`POST /wp-json/mexc/v1/snapshot`)
- Persist snapshot pages and support sitemap update flows
- Apply SEO/canonical metadata for snapshot content
- Provide admin controls for snapshot status/settings

## Integration Flow

1. User opens the WordPress page containing `[mexc_dashboard]`
2. Plugin frontend runtime requests stats from backend endpoints
3. Backend delivers realtime/statistical payloads to plugin UI
4. Backend posts periodic snapshot payloads to plugin REST endpoint
5. Plugin stores snapshot content and updates sitemap/indexing workflow

## Current UI Iteration Changes

- Removed related coins widget from the live dashboard layout
- Kept display mode controls (`Ticker`, `Name`, `Both`) for coin selector labeling
- Fixed mobile layout by placing display mode controls under search input
- Reduced search-to-controls spacing issue caused by flex growth on mobile

## Who This Overview Is For

### Recruiters

Use this repo to evaluate practical WordPress + realtime integration delivery (not template-only plugin work).

### System Engineers

Use this repo to review frontend/backend boundary design, snapshot ingestion flow, and plugin-side operational responsibilities.

### Collaborators

Use this repo to understand extension points for UI modules, endpoint wiring, and snapshot/SEO behavior.

### Potential Employers

Use this repo as evidence of production-minded PHP plugin work integrated with async backend systems.

## Security & Disclosure Policy

- Private repository retains implementation internals.
- Public overview excludes endpoint secrets, infrastructure wiring, and private admin logic.
- This repository is intentionally non-sensitive and documentation-first.

## Related Private Repositories

- `logicencoder/mexc-live-stats-plugin`
- `logicencoder/mexc-live-stats-backend` (integration counterpart)

## Author note

Built with a practical, self-taught workflow: deliver the UX that operators need, keep boundaries clear between frontend and backend, and avoid hype in public documentation.
