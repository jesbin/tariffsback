# TariffBack

**Free, browser-based IEEPA tariff refund calculator for finance and trade compliance teams.**

Live at: [tariffback.ai](https://tariffback.ai) · Built by [CreditPulse](https://creditpulse.com)

---

## What it does

TariffBack helps importers identify and prioritize IEEPA tariff refund opportunities across their entry portfolio. Upload your ACE ES-003 export and instantly see:

- Total IEEPA duty exposure across all entries
- Which entries qualify for a **Post Summary Correction (PSC)** — the fastest refund path
- Which entries require a **Protest** (19 USC § 1514) — and how many days remain before the deadline expires
- Which entries have passed their protest window and may only be recoverable via **Court of International Trade (CIT)**
- Estimated refund amounts including accrued interest (5% per annum)

All processing happens in the browser. No data is sent to any server.

---

## Refund paths

TariffBack models three legally distinct refund paths based on liquidation status:

| Status | Path | Deadline |
|--------|------|----------|
| Not yet liquidated | **PSC** (Post Summary Correction) | File before liquidation |
| Liquidated ≤ 90 days ago | **Protest — Urgent** | Liquidation date + 180 days |
| Liquidated 91–180 days ago | **Protest — Watch** | Liquidation date + 180 days |
| Liquidated > 180 days ago | **CIT Risk** | Protest window expired |

- **Estimated liquidation** = entry date + 314 days (when CBP liquidation date is absent)
- **PSC – Act Soon** = unliquidated entry with estimated liquidation ≤ 30 days away
- **Interest** = IEEPA duty × 5% × years elapsed since entry date

---

## Covered HTS codes

TariffBack only calculates refunds for lines under active IEEPA tariff codes:

- `9903.01.10` – `9903.01.45` — Fentanyl / China tariffs
- `9903.02.01` – `9903.02.10` — Reciprocal tariffs

Other duty types (Section 301, antidumping, etc.) are excluded from refund calculations.

---

## Supported file format

### ACE ES-003 — Entry Summary Line Tariff Details

TariffBack is built for the CBP Automated Commercial Environment (ACE) ES-003 report.

**How to export from ACE:**
1. Log in to ACE Secure Data Portal → **Reports**
2. Select **Entry Summary** → **ES-003 Entry Summary Line Tariff Details**
3. Set date range: **Feb 4, 2025 – present** (IEEPA tariffs began Feb 4, 2025)
4. Export as **CSV** or **Excel (.xlsx)**

The parser uses a dual strategy:
- **Strategy A** — reads the `IEEPA Duty Amount (USD)` column directly (standard ACE export)
- **Strategy B** — filters by HTS prefix `9903.01` / `9903.02` and sums `Total Entered Value` (broker exports)

A sample file is included: [`ACE_ES-003_Sample.csv`](./ACE_ES-003_Sample.csv)

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| Frontend | Single HTML file, no build step |
| CSV parsing | [PapaParse](https://www.papaparse.com/) (CDN) |
| Excel parsing | [SheetJS](https://sheetjs.com/) (CDN) |
| Hosting | Cloudflare Workers (static assets) |
| Lead capture | n8n webhook → HubSpot + Slack |

No frameworks, no bundler, no database. The entire app is `index.html`.

---

## Deployment

The site is deployed as a Cloudflare Worker with static assets via `wrangler.jsonc`.

```bash
# Deploy to Cloudflare Workers
npx wrangler deploy
```

Config (`wrangler.jsonc`):
```json
{
  "name": "tariffsback",
  "assets": { "directory": "." },
  "compatibility_date": "2025-09-27"
}
```

The production Worker runs at `https://tariffsback2.jordan-77f.workers.dev` (mapped to `tariffback.ai`).

---

## Local development

No build step required — open `index.html` directly in a browser, or serve it with any static file server:

```bash
npx serve .
# or
python3 -m http.server 8080
```

---

## Lead capture flow

When a user books an assessment or requests results by email:

1. Form data is `POST`ed to an n8n webhook
2. n8n creates a HubSpot contact + deal
3. n8n sends a Slack notification to the CreditPulse team

The booking link goes to: `https://calendar.app.google/dEPUjzLs3PC5fX5N9`

---

## File structure

```
tariffsback/
├── index.html              # Entire application (HTML + CSS + JS)
├── ACE_ES-003_Sample.csv   # Sample data for testing the parser
├── wrangler.jsonc          # Cloudflare Workers deploy config
└── README.md
```

---

## Legal disclaimer

Results are estimates only and do not constitute legal or customs advice. IEEPA refund eligibility depends on liquidation status, protest deadlines, and ongoing CBP guidance. Contact a licensed customs broker for official claim filing.

---

Built by [CreditPulse](https://creditpulse.com) · [Book a free assessment](https://calendar.app.google/dEPUjzLs3PC5fX5N9)
