# Lead Ledger

A shared, browser-based ledger for managing Meta (Facebook & Instagram) lead-generation data — import, dedupe, analyse, and export, with built-in AI Q&A. Built for **Health Total by Anjali Mukerjee**.

Lead Ledger turns raw lead exports into a clean, searchable, team-shared record with dashboards, flexible filtering, Excel export, and a natural-language assistant. It runs as a single-file React app inside Claude, using persistent shared storage so everyone with the link sees and works on the same data.

---

## Contents

- [Why it exists](#why-it-exists)
- [Features](#features)
- [How it works](#how-it-works)
- [Importing leads](#importing-leads)
- [Google Sheet sync](#google-sheet-sync)
- [Ask AI](#ask-ai)
- [Privacy](#privacy)
- [Tech stack](#tech-stack)
- [Running the app](#running-the-app)
- [Field mapping](#field-mapping)
- [Limitations](#limitations)
- [License](#license)

---

## Why it exists

Meta's lead-gen exports are awkward to work with: inconsistent column names, mixed date formats, duplicates across overlapping downloads, and no shared view for a team. Lead Ledger fixes that. You drop in a file exactly as Meta gives it and get a tidy, deduplicated, analysable record that the whole team can read and act on from one link.

## Features

- **Automatic import** — Upload a Meta export ZIP as-is, a plain CSV/Excel file, or paste CSV text. ZIPs are unpacked entirely in the browser (no server), and column headers are auto-mapped to standard fields.
- **Smart date handling** — Detects day-first vs. month-first numeric dates, Excel serial numbers, and ISO timestamps, then normalises everything to a single consistent timeline.
- **Deduplication** — Leads are matched by phone, then email, then name + date, so re-uploading an overlapping file only adds what's genuinely new.
- **Dashboard** — Headline stats (total leads, this month, busiest day, top campaign) plus charts for leads-per-day, by campaign, by city, and month-by-month trend.
- **Leads view** — Full-text search; filters for month, campaign, and date range; multiple sort orders; and one-click Excel export of exactly the filtered set.
- **Google Sheet sync** — Pull the newest rows from a Zapier-fed Google Sheet on demand or automatically every five minutes.
- **Ask AI** — Question your leads in plain language and get specific, numbers-backed answers with suggested marketing actions.
- **Shared team ledger** — Persistent shared storage means everyone with the link sees and edits the same records.

## How it works

The app is a single React component. On load it reads its index and lead batches from persistent shared storage and renders four tabs:

| Tab | What it does |
|-----|--------------|
| **Dashboard** | Aggregate stats and charts across all leads |
| **Leads** | Searchable, filterable, sortable list with Excel export |
| **Import** | File/ZIP/paste import and Google Sheet sync |
| **Ask AI** | Natural-language analysis of aggregated data |

Imports are stored as discrete **batches** so any single import can be deleted later without disturbing the rest. A lightweight index tracks each batch's label, lead count, and import date.

## Importing leads

Three ways to get leads in, all on the **Import** tab:

1. **ZIP** — Upload the Meta export ZIP exactly as downloaded. Every CSV/Excel file inside is read and merged automatically; unreadable or password-protected entries are skipped with a note.
2. **CSV / Excel** — Upload a single `.csv`, `.tsv`, `.xlsx`, or `.xls` file.
3. **Paste** — Paste raw CSV text (with a header row) directly.

Across all three, headers are normalised and mapped to standard fields, dates are parsed, and duplicates are skipped automatically. After an import you'll see a summary: how many leads were saved, the date span covered, how many duplicates were skipped, and how many rows had no readable date.

## Google Sheet sync

If Zapier collects your Meta leads into a Google Sheet, the ledger can pull from it directly:

1. Enter the **exact** sheet name on the Import tab.
2. Click **Sync now**, or tick **Auto-sync** to refresh every five minutes while the app is open.

Sync reads the most recent rows of the sheet (core fields only) and adds anything new, skipping duplicates. Whoever runs the sync needs **Google Drive connected** to their Claude account (Settings → Connectors) and access to the sheet. For full history or custom form-answer columns, upload the sheet's Excel export instead.

## Ask AI

The **Ask AI** tab answers questions about your saved leads in plain language — "Which campaign brought the most leads?", "Compare this month with last month", "Which city brings the most leads?". Responses are specific and number-driven, and often include practical marketing suggestions.

## Privacy

Contact details never leave the ledger. The AI assistant receives **only aggregated statistics and a de-identified sample** (date, city, campaign, ad set) — never names, phone numbers, or emails. The Google Sheet sync handles contact fields only to store them locally in the ledger; it does not send them to the model.

## Tech stack

- **React** — UI (single-file component)
- **Recharts** — dashboard charts
- **PapaParse** — CSV parsing
- **SheetJS (xlsx)** — Excel read/write
- **lucide-react** — icons
- **Claude artifact runtime** — `window.storage` for persistence and the Anthropic Messages API for AI features and Google Drive sync

ZIP extraction uses the browser's native `DecompressionStream`, so no archive library is bundled.

## Running the app

This app is built to run as a **Claude artifact**, which provides the two services it depends on:

- `window.storage` — shared, persistent key-value storage
- The Anthropic Messages API (and Google Drive MCP) — used for Ask AI and Sheet sync

Open `health-total-lead-ledger.jsx` inside Claude to run it with full functionality. Outside that environment, persistent storage and AI features won't be available; the app detects this and falls back to session-only, in-memory data with a warning banner.

> **Note:** This is the source for the artifact rather than a standalone web app. To embed it elsewhere, you'd need to supply your own equivalents of `window.storage` and the API endpoints.

## Field mapping

Incoming column headers are normalised (lowercased, stripped of non-alphanumerics) and mapped to a standard set of fields. A range of common Meta and spreadsheet header variants are recognised, including:

| Standard field | Recognised headers (examples) |
|----------------|-------------------------------|
| `created_time` | created time, date, timestamp, submitted at, lead date… |
| `name` | full name, lead name (also built from first + last name) |
| `phone` | phone, mobile, contact number, whatsapp number… |
| `email` | email, email address, email id |
| `city` | city, location, town/city |
| `campaign` | campaign name, campaign |
| `adset` / `ad` | ad set name, ad name |
| `form` | form name, form |
| `platform` | platform |

Any unrecognised columns are preserved per-lead under an `extra` field, so nothing is lost — they show in the lead detail view and in Excel exports.

## Limitations

- Designed for the Claude artifact environment; full persistence and AI require it.
- Each individual import must fit within the storage value limit (very large files should be split).
- Google Sheet sync pulls the most recent rows (core fields), not the entire history — use Excel export for that.




---

Built for Health Total by Anjali Mukerjee.
