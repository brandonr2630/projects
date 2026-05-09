# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a monorepo of vanilla HTML/CSS/JS projects for the Terran Resources group of companies. There is no build system, bundler, package manager, or test suite — every project is a flat directory of files deployed directly to GreenGeeks via GitHub Actions.

## Projects

| Directory | Live URL | Repo |
|-----------|----------|------|
| `Terran Group ERP/` | `erp.terranresources.com` | `brandonr2630/meridian-erp` |
| `q2-machines-job-cards/` | `q2m.io/jobs` | `brandonr2630/q2-machines-job-cards` |
| `Q2M WEBSITE/` | `q2m.io` | `brandonr2630/q2m-website` |
| `COC Website/` | `toddsroadcoctt` (GreenGeeks) | `brandonr2630/coc-website` |
| `Terran Resources Website/` | `terranresources.com` | `brandonr2630/terran-resources-website` (no deploy workflow yet) |
| `MyWebPages/` | — | not wired |

## Deployment

Every push to `master` auto-deploys via GitHub Actions → cPanel Git Version Control API (`POST https://chi203.greengeeks.net:2083/execute/VersionControl/update`). The `CPANEL_API_TOKEN` secret is stored in each repo. `terran-resources-website` has no workflow yet.

To trigger a manual redeploy: push any commit to `master` on the relevant repo.

## Meridian ERP (`Terran Group ERP/index.html`)

**The entire app is one file — `index.html` (≈8400 lines of inline CSS + JS). Never edit `index2–5.html`; those are archives.**

### Architecture

- **Backend:** Supabase, called directly via a thin fetch wrapper. All DB access goes through `sb()` → `sbGet/sbPost/sbPatch/sbPatchWhere/sbDelete/sbDeleteWhere`.
- **Auth:** Supabase JWT. Session (access token + refresh token) is persisted to `localStorage` and restored on load by `tryRestoreSession()`.
- **Multi-company:** every data record is scoped to `currentCompany.id`. The company switcher lives in the topbar; `switchCompany(id)` reloads view data.
- **Navigation:** `navigate(view)` sets `currentView` and calls `loadViewData(view)`, which dispatches to the module loader for that view.
- **State:** Global vars declared at `// ── STATE ──` (line ~3373): `currentUser`, `currentCompany`, `companies`, `accounts`, `contacts`, `journalEntries`, `bankAccounts`, etc.

### Roles

`super_admin` > `admin` > `sales` > `user`. Finance modules (AP, Bank, CoA, Journal, Reports) are admin-only. Check `canPost()`, `canVoid()`, `canFinance()`, etc. before rendering controls.

### Module sections (by JS comment header)

Dashboard · Chart of Accounts · Journal Entries · AR · AP · Vendor Payments · Clients · Vendors · ERP Companies · Settings · Bank Accounts · Bank Transactions & Reconciliation · Bills (AP) · Financial Reports · Quotations · Sales Leads · PDF generation (Invoice, Quotation, Delivery Note, Credit Note)

### PDF & exports

PDFs are rendered client-side with a custom multi-page renderer. SheetJS is lazy-loaded from CDN on first Excel export. The report download menu is a shared fixed component.

### Currencies

TTD (`TT$`), USD (`US$`), GYD (`G$`) — formatted via `fmt(amount, currency)`.

## Job Cards PWA (`q2-machines-job-cards/`)

### Architecture

Dual-backend app:
- **Supabase** — primary data store for job records and user auth.
- **Google Apps Script (`Code.gs`)** — web app backed by a Google Spreadsheet, serving config catalogues (Labour, Equipment, Materials, Consumables, QC Checklists) and job number generation. Deployed as "Execute as: Me, Access: Anyone."

The frontend (`index.html`, ≈3100 lines) talks to both backends. Config is fetched from Apps Script on load; job CRUD goes to Supabase.

### Offline

`service-worker.js` caches the app shell under `CACHE_NAME = 'q2-machines-v2'`. When deploying changes that must bust the cache, increment the version string in `service-worker.js`.

### Key modules

Login · Dashboard · Job form (Labour, Equipment, Materials, Consumables, Sub-contractors, QC Checklist, Costing Summary, Drawings/Specs/Reports) · Config Manager · Offline queue · User Management

## Static websites

- **Q2M WEBSITE** — equipment depot listings pulled from `listings.json`; `depot.html` is a separate page.
- **COC Website** — single-page church site; contact form backend TBD.
- **Terran Resources Website** — early scaffold; contact form and deploy not yet wired.
- **MyWebPages** — offline Bible reader; translations stored as large JSON files per version (kjv, asv, lsv, etc.).
