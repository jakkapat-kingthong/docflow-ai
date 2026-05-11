# Google Sheets Schema

## Setup

Create a new Google Sheet with 4 tabs named exactly as below.

---

## Tab 1: `All Documents` (main log)

| Column | Header | Notes |
|--------|--------|-------|
| A | Timestamp | Auto-filled by workflow |
| B | Type | slip / invoice / receipt / purchase_order / unknown |
| C | Status | auto / flagged / needs_review |
| D | Confidence | 0.00 – 1.00 |
| E | Amount | Numeric |
| F | Currency | THB |
| G | Sender | Sender name |
| H | Receiver | Receiver name |
| I | Bank | Bank name |
| J | DateTime | Document's own timestamp |
| K | RefNumber | Reference / transaction number |
| L | Description | Additional notes |
| M | ChatID | Telegram chat ID (for reply) |
| N | ExtractionNotes | AI's own uncertainty notes |
| O | RawJSON | Full JSON from Gemini |

---

## Tab 2: `Slips`

Same columns as All Documents, filtered to type=slip.

| A | B | C | D | E | F | G | H | I | J | K | M |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Timestamp | Status | Confidence | Amount | Currency | Sender | Receiver | Bank | DateTime | RefNumber | ChatID | Notes |

---

## Tab 3: `Invoices`

| A | B | C | D | E | F | G | H | J | K | L | M |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Timestamp | Status | Confidence | Amount | Currency | Sender | Receiver | DateTime | RefNumber | Description | ChatID | Notes |

---

## Tab 4: `Flagged` (confidence 60–85%, needs human review)

Same columns as All Documents. This tab accumulates items needing manual verification.
Add a "Reviewed" column (P) manually: ✅ / ❌ after human checks.

---

## Google Sheets Formula Suggestions

Add these to a `Dashboard` tab for quick overview:

```
=COUNTIF(All Documents!C:C,"auto")           → total auto-processed
=COUNTIF(All Documents!C:C,"flagged")        → total flagged
=COUNTIF(All Documents!C:C,"needs_review")   → total needs review
=SUMIF(All Documents!B:B,"slip",All Documents!E:E)   → total slip amount
=AVERAGE(All Documents!D:D)                  → average confidence
```

---

## Sharing

The n8n Google Sheets node uses OAuth2. You need to:
1. Connect Google account in n8n credentials (Settings → Credentials → New → Google Sheets OAuth2)
2. Share the sheet with your Google account (or use service account)
3. Paste the Sheet ID into each Google Sheets node in the workflow
