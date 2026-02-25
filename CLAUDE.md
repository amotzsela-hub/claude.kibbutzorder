# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this app is

A PWA (Progressive Web App) for tracking coffee bean orders from repeat customers at a kibbutz store. Customers order and pay later via a payment app. The owner uses this to ensure no payment is missed.

**Target device:** iPhone 15 Pro Max, iOS 26+, added to home screen via Safari.

## Stack

- **No build step.** Pure HTML/CSS/JS in `index.html`. No npm, no framework, no bundler.
- **Storage:** `localStorage` — keys `orders_v1` and `customers_v1`.
- **Offline:** Service worker in `sw.js` caches app assets.
- **PWA manifest:** `manifest.json`

## Running locally

Open `index.html` directly in a browser, or serve with any static server:
```
npx serve .
python -m http.server 8080
```
For service worker to work, must be served over HTTP (not `file://`).

## Deploying (GitHub Pages)

Push to a public GitHub repo → Settings → Pages → Deploy from main branch root. The app URL is then added to iPhone home screen via Safari → Share → Add to Home Screen.

## Architecture

All logic is in the `<script>` tag in `index.html`. Key sections (marked with comments):

| Section | What it does |
|---|---|
| STATE & STORAGE | `S` object + `load/saveO/saveC` |
| RENDER | `render()` → `renderActive()` / `renderHistory()` → `cardHtml()` |
| ORDER ACTIONS | `markReady`, `markPaid`, `deleteOrder`, `editOrder` |
| ORDER MODAL | Open/close, `renderChips()`, `submitOrder()` |
| CUSTOMERS MODAL | Add/remove customers |
| VOICE INPUT | `startVoice` → `parseVoice` (matches customer name, extracts price, rest = item) |
| NOTIFICATIONS | Web Notifications API — fires on new order and on mark-ready |

## Order lifecycle

`ordered` → (Mark Ready) → `ready` → (Mark Paid) → `paid`

## Data shape

```js
// Order
{ id, customerId, customerName, item, price, notes,
  status: 'ordered'|'ready'|'paid',
  createdAt, preparedAt, paidAt }

// Customer
{ id, name }
```

## Voice parsing logic (`parseVoice`)

1. Scan transcript for a known customer name → remove from string
2. Regex-match price (e.g. "130 shekels", "130 nis", or bare number at end)
3. Remaining text → item description field

Uses `navigator.language` so Hebrew names are recognised naturally on an Israeli device.

## User preferences

- Speed and simplicity over features
- ~2 orders/day, all named repeat customers, average ₪130/order
- Offline-first, no server, no account

## Git & deploy

Git identity is set locally (not global) — already configured for this repo.
Deploy workflow: `git add <files> && git commit -m "message" && git push`
Only commit app files: `index.html manifest.json sw.js icon.svg`

**Live URL:** https://amotzsela-hub.github.io/claude.kibbutzorder/
**Repo:** https://github.com/amotzsela-hub/claude.kibbutzorder.git

## UI patterns to keep consistent

- Customer grid uses `.chip` elements; always include a `+ New` chip at the end (calls `addCustomerInline()`)
- After any change to `S.customers`, call `renderChips()` to keep order form in sync
- Dates: use `formatDate(ts)` for absolute dates, `ago(ts)` for relative — active orders show `formatDate(createdAt)`, history shows `formatDate(paidAt)`

## Platform note

iOS 26 is a real version (released after August 2025). Don't second-guess the user about it.
