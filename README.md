# DocFlow AI 🧾

> **AI-powered Thai business document pipeline**  
> Telegram → Gemini Flash 2.0 → Confidence routing → Google Sheets  
> Zero manual data entry. Self-aware AI that knows when it's not sure.

---

## What makes this different

Most n8n document bots: `send image → OCR → write to sheet`

DocFlow AI adds **Confidence-Aware Routing**:

| Confidence | Action |
|---|---|
| < 60% | Ask user to resend a clearer image |
| 60–85% | Log to Sheets + flag for human review |
| > 85% | Auto-log instantly, reply with extracted data |

This means **zero silent errors** — the system escalates rather than silently writing garbage data.

---

## Demo

```
User: [sends slip image]
Bot:  ✅ บันทึกแล้วอัตโนมัติ
      📄 ประเภท: slip
      💰 ยอด: 1,500 THB
      👤 จาก: นาย ก สมชาย
      🏦 ธนาคาร: กสิกรไทย
      🔢 เลขอ้างอิง: 67ABC12345
      🎯 confidence: 94%
```

---

## Document types supported

- `slip` — bank transfer slip (สลิปโอนเงิน)
- `invoice` — tax invoice / bill (ใบแจ้งหนี้ / ใบกำกับภาษี)
- `receipt` — payment receipt (ใบเสร็จรับเงิน)
- `purchase_order` — purchase order (ใบสั่งซื้อ)

---

## Architecture

```
Telegram Bot
    ↓
n8n (self-hosted via Docker)
    ↓
Gemini Flash 2.0
    classify · extract · confidence score
    ↓
Confidence Router
    ├── Low  < 60%  → ask user to resend
    ├── Med  60–85% → Sheets (flagged) + notify
    └── High > 85%  → Sheets (auto) + confirm reply
```

---

## Cost

| Component | Cost |
|---|---|
| Gemini Flash 2.0 | Free tier: 1,500 req/day |
| n8n self-hosted | $0 (your machine) |
| ngrok tunnel | Free tier: 1 agent |
| Google Sheets | Free |
| **Total** | **$0/month** for typical demo/small business use |

Paid tier estimate: 10,000 docs/day ≈ $2-3/month (Gemini Flash pricing).

---

## Prerequisites

- Docker + Docker Compose
- Python 3.x (optional, for local testing)
- Telegram account
- Google account
- ngrok account (free)

---

## Setup (estimated 45 minutes)

### Step 1 — Clone and configure (5 min)

```bash
git clone https://github.com/YOUR_USERNAME/docflow-ai
cd docflow-ai
cp .env.example .env
```

Edit `.env` with your values (see comments inside).

### Step 2 — Create Telegram Bot (5 min)

1. Open Telegram → search `@BotFather`
2. Send `/newbot` → follow prompts
3. Copy the token → paste into `.env` as `TELEGRAM_TOKEN`

### Step 3 — Get Gemini API Key (3 min)

1. Go to https://aistudio.google.com/app/apikey
2. Create API key (free, no credit card needed)
3. Paste into `.env` as `GEMINI_API_KEY`

### Step 4 — Start n8n (5 min)

```bash
docker compose up -d
```

Open http://localhost:5678 → login with credentials from `.env`

### Step 5 — Set up ngrok tunnel (5 min)

```bash
# Install ngrok: https://ngrok.com/download
ngrok http 5678
```

Copy the `https://xxxx.ngrok-free.app` URL → paste into `.env` as `NGROK_URL`

Restart n8n so it picks up the webhook URL:
```bash
docker compose restart
```

### Step 6 — Import workflow (2 min)

1. In n8n: click **+** → **Import from file**
2. Select `workflow/docflow-ai.json`
3. Click **Import**

### Step 7 — Configure credentials in n8n (10 min)

**Telegram credential:**
1. Settings → Credentials → New → Telegram API
2. Paste your `TELEGRAM_TOKEN`
3. Save → assign to **Telegram Trigger** node

**Google Sheets credential:**
1. Settings → Credentials → New → Google Sheets OAuth2 API
2. Follow OAuth flow → authorize your Google account
3. Save → assign to both **Sheets — Flagged** and **Sheets — Auto** nodes

### Step 8 — Set up Google Sheets (5 min)

1. Create a new Google Sheet
2. Create a tab named exactly: `All Documents`
3. Add headers in Row 1 (see `docs/sheets-schema.md` for full schema)
4. Copy the Sheet ID from the URL → paste into `.env` as `SHEET_ID`

   Sheet URL: `docs.google.com/spreadsheets/d/{SHEET_ID}/edit`

### Step 9 — Activate and test (5 min)

1. In n8n: toggle **Active** on the workflow
2. Open Telegram → find your bot → send a slip image
3. Verify data appears in Google Sheets

---

## Adjusting confidence thresholds

In `.env`:
```
LOW_THRESHOLD=0.60   # below this → ask to resend
HIGH_THRESHOLD=0.85  # above this → auto-log
```

Restart n8n after changes:
```bash
docker compose restart
```

---

## Google Sheets column headers (Row 1)

Paste exactly into `All Documents` tab:

```
Timestamp | Type | Status | Confidence | Amount | Currency | Sender | Receiver | Bank | DateTime | RefNumber | Description | ChatID | ExtractionNotes | RawJSON
```

---

## Project structure

```
docflow-ai/
├── docker-compose.yml       # n8n container setup
├── .env.example             # environment variables template
├── workflow/
│   └── docflow-ai.json      # n8n importable workflow (14 nodes)
├── docs/
│   ├── gemini-prompt.md     # extraction prompt + engineering notes
│   └── sheets-schema.md     # Google Sheets column spec
└── README.md
```

---

## Planned features (V2)

- [ ] LINE OA trigger (in addition to Telegram)
- [ ] Duplicate detection by ref_number
- [ ] Daily summary cron → Telegram notification
- [ ] Google Looker Studio dashboard connected to Sheets
- [ ] Confidence calibration based on correction history

---

## Tech stack

| Layer | Technology |
|---|---|
| Orchestration | n8n (self-hosted) |
| AI / OCR | Google Gemini Flash 2.0 |
| Storage | Google Sheets |
| Trigger | Telegram Bot API |
| Tunnel | ngrok |
| Container | Docker Compose |

---

## Author

**Jakkapat Kingthong (เต้)**  
AI Engineer · Google Student Ambassador  
[github.com/jakkapat-kingthong](https://github.com/jakkapat-kingthong) · [linkedin.com/in/jakkapat-kingthong-35345039a](https://linkedin.com/in/jakkapat-kingthong-35345039a)
