# PrintOS

An all-in-one, self-hosted business operations platform originally built for **The Print Collective** (3D printing). What started as a shop-management tool for orders, inventory, and invoicing has grown into a full command center that also runs a customer-facing storefront, a crypto/prediction-market trading bot, an AI-driven Facebook marketing pilot, a home network monitor, two browser games, and a few personal/day-job utilities — all under one login with a customizable, drag-and-drop sidebar.

Single-process **Flask** backend, vanilla **JS/HTML/CSS** frontend (no build step), flat **JSON files** for storage. Built to run on a Raspberry Pi / small home server.

---

## Table of contents

- [Core shop management](#core-shop-management)
- [Customer-facing storefront](#customer-facing-storefront)
- [Finance](#finance)
- [Extras](#extras)
  - [BTC / prediction-market trading bot](#btc--prediction-market-trading-bot)
  - [Facebook Autopilot](#facebook-autopilot)
  - [Network Monitor](#network-monitor)
  - [Lots of Loot](#lots-of-loot)
  - [Shadowfall (ARPG)](#shadowfall-arpg)
  - [Call Notes](#call-notes)
  - [Paydays](#paydays)
- [System & platform features](#system--platform-features)
- [Tech stack](#tech-stack)
- [Project structure](#project-structure)
- [License](#license)

---

## Core shop management

The "Daily" and "Work" modules that run the print shop day to day:

| Page | What it does |
|---|---|
| **Dashboard** | At-a-glance revenue, pending/printing/complete order counts, low-stock alerts, and recent orders. |
| **Orders** | Full order lifecycle (Pending → Printing → Complete), linked to customers, inventory, and filament spools. |
| **Customers** | Customer records with linked order, ticket, invoice, and work-order history. |
| **Inventory** | Product/part catalog with stock levels, min-stock thresholds, images, and sell prices (feeds the Shop). |
| **Calendar** | Drag-to-schedule print jobs against start/end dates. |
| **Kanban** | Visual board for moving jobs between stages. |
| **Quotes** | Public quote-request intake form → internal review → accept/decline, with reference-image uploads. |
| **Tickets** | Support ticket queue that can be converted directly into a work order. |
| **Work Orders** | Scoped service jobs with status tracking, separate from print orders. |
| **Hardware** | Tracks owned equipment/printers. |
| **Services** | Catalog of billable services (e.g., the remote PC cleanup/optimization offering). |
| **Filament** | Spool inventory by material/color with usage deduction per print. |
| **Calculator** | Print-cost calculator using configurable electricity rate, printer hourly cost, labor rate, and markup. |
| **Invoices** | Generate, view, and mark invoices paid; customer-facing pay links with a payment-session/webhook flow. |
| **Reports** | Rollup reporting across orders/revenue/customers. |

## Customer-facing storefront

- **`/shop`** — Public storefront pulling from Inventory (any item with a sell price). Fully brandable (hero title/subtitle, feature badges) from Settings, and can be toggled off entirely (`shop_disabled` state).
- **`/shop/<id>`** — Individual product pages.
- **Quote/order requests** submitted from the shop flow straight into the internal Quotes queue.
- **`/track`** — Public order-tracking page via a generated tracking code.
- **`/camera`** + **`/camera_public`** — Live print-cam view (public and internal variants).
- **Visitor analytics** — every shop visit, product view, and order request is logged (browser, platform, country, referrer) via a built-in activity logger, visible in an internal activity feed with stats.

## Finance

- **Expenses** — logged expenses with category stats.
- **Taxes** — mileage log, 1099 contractor tracking, and configurable tax settings.
- **Paydays** — see [below](#paydays).
- **Reports** — revenue/customer/order rollups.

## Extras

### BTC / prediction-market trading bot

A `/btc` dashboard controlling an automated trading bot (`btc_bot/`) that:
- Pulls price/indicator data and runs a defined strategy (`strategy.py`, `indicators.py`, `data_feed.py`) to generate buy/sell signals.
- Trades on **Kalshi** event contracts and **Polymarket** prediction markets (separate runners for each, with start/stop controls and a dry-run mode).
- Tracks wins/losses, session P&L, bet history, and account balance live in the UI.
- Exports bet logs and a model-training dataset to Excel.

### Facebook Autopilot

An AI content pipeline for a Facebook Page, with its own page (`/fb-autopost`):
- Generates on-brand post copy from rotating content "pillars," previews it, and can generate ad-style image variants before posting.
- Posts immediately or on a scheduled cron.
- Syncs engagement and fetches comments, with AI-drafted replies queued for approval before sending.
- A "groups" queue for cross-posting into Facebook groups, each with its own review/approval step.
- Token management (status/refresh/bootstrap) and a configurable alerts/digest system that can test-fire and preview a run.
- A separate "extras" layer that can source PrintOS shop photos as post candidates.

### Network Monitor

A `/network` dashboard for home-network health:
- Continuous ping/jitter/packet-loss tracking with configurable interval.
- On-demand traceroute and speed tests (background job, non-blocking).
- Outage detection/log with uptime tracking, plus daily/weekly bandwidth trend rollups.
- Raw interface throughput pulled straight from `/proc/net/dev`.

### Lots of Loot

A lightweight browser **extraction-shooter loot economy** (`/lootgame`) — currency + a stash of items by rarity/value, with extract/sell/sell-all actions.

### Shadowfall (ARPG)

An original, Diablo-2-inspired **action RPG** (`/arpg`) with its own account system (separate from the main PrintOS login):
- Multiple original classes (e.g. Warbringer, Stormcaller, Huntress, Sentinel, Deathspeaker, Shadowblade, Wildshaper), each with tiered skill trees (attacks, buffs, auras).
- Character creation/management, an in-game shop, and **bot profiles** per character (automated/idle play configurations).
- Custom generated art assets for items, monster portraits, and sprites.

### Call Notes

A standalone note/script-template tool (`/callnotes`) built for day-job customer-service calls (Extra Space Storage), deliberately **not** behind the main login so it can be shared with coworkers:
- Reusable note templates by category/scenario, with a lightweight usage log (no customer data).
- Ships with a **browser bookmarklet** (`notebox_fill.js`) that injects a floating panel into the internal ESS tool and auto-fills the note title/body/category fields from a selected template.

### Paydays

An owner-only personal income tracker (`/paydays`) for juggling multiple jobs/pay schedules:
- Multiple income sources with weekly/biweekly/monthly/semimonthly cadences and a running balance.
- Generates a personal **iCalendar (.ics) feed** (with a regenerate-able secret token) so upcoming paydays show up automatically in any calendar app.

## System & platform features

- **Auth & onboarding** — first-run `/setup` creates the owner account; after that, new staff join via single-use invite links (`/invite/<token>`).
- **Role-based permissions** — an `owner` role with full access, plus granular per-page permissions grantable to staff accounts (e.g., a tech can see Tickets/Work Orders without seeing Settings, Staff, or Paydays, which are owner-only).
- **Customizable navigation** — the entire sidebar is drag-and-drop: users can reorder items, rename/reorganize the default sections (Daily, Work, Inventory, Finance, Extras, System), and hide pages they don't use, persisted per deployment.
- **Demo mode** — a toggle that swaps live data for a bundled demo dataset and blocks all writes, so the whole app can be shown off publicly without touching real business data.
- **Activity log** — a unified event log across shop visits, admin actions, invite/permission changes, and more, with stats.
- **Backup endpoint** — on-demand data export.
- **Branding/settings** — business name, tagline, contact info, social links, invoice notes/footer, and full shop hero/feature copy, all editable from Settings and injected into every template.
- **File uploads** — product images, quote reference images, etc. (size-capped, extension-restricted).

## Tech stack

- **Backend:** Python / Flask, session-based auth, JSON-file persistence (`data.json`, `users.json`, plus feature-specific stores)
- **Frontend:** Server-rendered Jinja templates + vanilla JS (`static/js/`), no frontend build pipeline
- **Bot/automation:** standalone Python modules (`btc_bot/`) run as background threads/processes, called from Flask routes
- **Key dependencies:** Flask, Werkzeug, Requests, `cryptography`, `openpyxl` (Excel export), `py-clob-client` (Polymarket)

## Project structure

```
app.py                  # Main Flask app — core shop/business routes
arpg_routes.py          # Shadowfall ARPG
callnotes_routes.py     # Call Notes tool
lootgame_routes.py      # Lots of Loot
network_routes.py       # Network Monitor
bookmarklet_routes.py   # Call Notes bookmarklet delivery
printos_blueprint.py    # Shared blueprint helpers
btc_bot/                # Trading bot: strategy, indicators, data feed, Kalshi/Polymarket clients
templates/               # Jinja templates, one+ per page/feature
static/
  ├─ css/               # Stylesheet
  ├─ js/                # Per-feature JS (arpg.js, lootgame.js, callnotes.js, notebox_fill.js, app.js)
  └─ img/arpg/           # Generated ARPG art (items, monsters, sprites, portraits)
data.json / users.json / paydays.json  # Primary data stores
demo_data.json           # Dataset used in Demo Mode
```

## License

Proprietary — all rights reserved. See [`LICENSE.txt`](./LICENSE.txt). Personal-use only; no redistribution, modification, or commercial use without explicit permission.
