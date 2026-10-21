# WP Automation by Ritik

A low-cost WhatsApp Automation/CRM backend designed for **Hostinger Node.js hosting**. It uses WhatsApp QR login via Baileys, so it does **not** need Docker, PostgreSQL, Redis, Meta Cloud API, or Chromium.

> ⚠️ QR-login WhatsApp automation is unofficial and may violate WhatsApp terms. Use carefully, avoid spam, and prefer opt-in contacts.

## Features

- WhatsApp QR login dashboard
- Send WhatsApp messages by API or dashboard
- Receive and store recent inbound/outbound messages
- Contact/lead CRM with tags, notes, status, source, value
- Follow-up tracking so important WhatsApp leads do not get buried
- CSV export for leads/contacts
- Webhook integrations for Google Sheets, Zapier or Make
- Simple automation rules:
  - welcome auto-reply
  - keyword auto-replies
  - office-hours fallback
- Broadcast endpoint with rate limiting
- Persistent file-based JSON storage for cheap hosting
- Hostinger-compatible Node.js app structure
- No browser/Chromium dependency

## Quick start locally

```bash
npm install
cp .env.example .env
npm start
```

Open <http://localhost:3000>, scan the QR from WhatsApp → Linked devices.

## Hostinger Node.js deployment

1. Create a Node.js app in Hostinger.
2. Set startup file to:
   ```text
   src/server.js
   ```
3. Connect this GitHub repo.
4. Add environment variable:
   ```text
   API_KEY=your-long-secret-key
   DATA_DIR=/home/YOUR_HOSTINGER_USER/.wp-automation-by-ritik
   ```
5. Run install/build from Hostinger panel if prompted:
   ```bash
   npm install --omit=dev
   ```
6. Restart the Node app.
7. Open your domain and scan the QR.

## API

Pass your API key as either:

```http
x-api-key: your-key
```

or:

```http
Authorization: Bearer your-key
```

### Status

```http
GET /health
GET /api/status
```

### Send message

```http
POST /api/send
{
  "to": "919876543210",
  "message": "Hello from CRM"
}
```

### Contacts / leads

```http
GET /api/contacts
POST /api/contacts
{
  "phone": "919876543210",
  "name": "Amit",
  "tags": ["lead"],
  "status": "new",
  "notes": "Interested in demo"
}
```

### Broadcast

```http
POST /api/broadcast
{
  "recipients": ["919876543210", "919999999999"],
  "message": "Today offer: ...",
  "delayMs": 2500
}
```

## Business positioning from the PDF idea

This project is built for freelancers, tutors, small businesses, service providers, and WhatsApp Business users who want a cheap alternative to monthly CRM SaaS tools.


## PDF feature mapping

This repo is designed around the Hostinger WhatsApp Automation/CRM concept from the PDF:

- **Affordable Node.js hosting**: runs as a simple Node.js app on Hostinger Business hosting.
- **WhatsApp as front end, Node.js as brain**: WhatsApp messages come in through QR login, while Node.js stores memory, leads and automation state.
- **Memory**: stores contacts, recent messages, lead notes, status, source, value and follow-up dates.
- **Automation**: welcome replies, keyword replies, broadcast endpoint and webhook events.
- **Lead organization**: tags, status pipeline, notes, follow-up tracking and CSV export.
- **Integrations**: webhook support for Google Sheets, Zapier or Make so small businesses can sync data without a heavy database.
- **Cheap SaaS alternative**: no Meta Cloud API, no Docker, no PostgreSQL/Redis requirement, no browser/Chromium.

## Extra API endpoints

### Follow-ups

```http
GET /api/followups
POST /api/followups
{
  "phone": "919876543210",
  "nextFollowUpAt": "2026-05-18 11:00",
  "note": "Call for demo"
}
```

### Export contacts CSV

```http
GET /api/contacts.csv
```

### Integrations webhook

```http
PUT /api/integrations
{
  "googleSheetsWebhook": "https://script.google.com/...",
  "zapierWebhook": "https://hooks.zapier.com/...",
  "makeWebhook": "https://hook.eu1.make.com/..."
}
```

Whenever a contact or message is created/updated, the app POSTs an event to the configured webhook.

## Persistent WhatsApp login on Hostinger

WhatsApp QR login credentials are stored in `DATA_DIR`. On Hostinger, keep this outside the Git deployment folder so redeploys do not delete the WhatsApp session.

Recommended env variable:

```text
DATA_DIR=/home/YOUR_HOSTINGER_USER/.wp-automation-by-ritik
```

If you do not set it, the app automatically uses:

```text
$HOME/.wp-automation-by-ritik
```

## Advanced CRM features

The dashboard now includes multiple business sections:

- **Overview**: WhatsApp status, QR, quick send, analytics, recent messages, due follow-ups.
- **Leads**: add/edit contacts with name, status, tags, source, value, notes, next follow-up.
- **Pipeline**: kanban-style lead status board: new, contacted, qualified, won, lost.
- **Campaigns**: send broadcasts by tag/status/all contacts, use templates, delay messages, and store campaign history.
- **Templates**: reusable WhatsApp message templates with variables like `{{name}}` and `{{phone}}`.
- **Settings**: Google Sheets/Zapier/Make webhook integration and task reminders.

### Analytics

```http
GET /api/analytics
```

### Templates

```http
GET /api/templates
POST /api/templates
{
  "name": "Demo Follow-up",
  "text": "Hi {{name}}, are you available for a demo today?"
}
```

### Campaigns

```http
POST /api/campaigns
{
  "name": "Offer campaign",
  "tag": "lead",
  "templateId": "offer",
  "delayMs": 3000
}
```

### Tasks

```http
GET /api/tasks
POST /api/tasks
{
  "title": "Call lead",
  "phone": "919876543210",
  "dueAt": "2026-05-18 11:00"
}
```

## Business type presets

This repo is designed so other people can deploy it and immediately choose their business type. The app then loads matching auto-replies, templates, tags and lead flow.

Included presets:

- Restaurant / Cafe
- Doctor / Clinic
- Tutor / Coaching
- Salon / Spa
- Real Estate / Property
- E-commerce / Online Store
- Freelancer / Service Provider

Dashboard path:

```text
Settings → Business Preset → Apply Business Preset
```

API:

```http
GET /api/presets
POST /api/presets/apply
{
  "preset": "restaurant",
  "keepExisting": true
}
```

Each preset includes welcome messages, keyword replies, and reusable templates. Messages support variables:

```text
{{name}}, {{phone}}, {{business}}
```

## Google Sheets Auto-Sync Link (no Apps Script, no OAuth)

For non-technical users, this is the recommended integration. They only copy one formula from the dashboard and paste it into Google Sheets cell A1. Google Sheets automatically creates the table columns and pulls updated CRM rows.

Dashboard:

```text
Settings → Google Sheets - Easy Link → Generate Sheet Links
```

Then copy the formula into Google Sheets cell A1:

```text
=IMPORTDATA("https://your-app.com/sheets/contacts.csv?token=YOUR_TOKEN")
```

Messages feed:

```text
=IMPORTDATA("https://your-app.com/sheets/messages.csv?token=YOUR_TOKEN")
```

Optional env variable:

```text
SHEET_TOKEN=separate-secret-for-sheets
```

If `SHEET_TOKEN` is not set, the app uses `API_KEY` for the sheet links.

## Professional conversation flows

The bot includes SaaS-style step-by-step flows, not only simple keyword replies.

Examples:

- Customer: `book hotel` / `book table` → bot asks name, date, time, guests, phone → saves booking lead.
- Customer: `appointment` → bot asks patient name, age, concern, date, time → saves clinic lead.
- Customer: `demo` → bot asks student name, class, subject, time → saves tutor lead.
- Customer: `site visit` → bot asks location, budget, date, phone → saves property lead.
- Customer: `order` → bot asks product, quantity, address, phone → saves order lead.

API:

```http
GET /api/flows
```

### What users do

1. Open dashboard → **Settings**.
2. Click **Generate Google Sheet Formula**.
3. Open a new Google Sheet.
4. Paste the Contacts formula into cell `A1`.
5. Optional: create a second tab and paste the Messages formula into cell `A1`.

Google Sheets will show table headers automatically and refresh the data periodically. No Google Cloud setup, no OAuth, no Apps Script.

> Note: Google controls `IMPORTDATA` refresh timing. It is simple and automatic, but not instant real-time.

## Bookings / Appointments table

When a professional conversation flow completes, the app creates a clean booking record.

Dashboard:

```text
Bookings → view/confirm/cancel/export bookings
```

Examples that create records:

- `book hotel`
- `book table`
- `appointment`
- `demo`
- `site visit`
- `order`

API:

```http
GET /api/bookings
PUT /api/bookings/:id
GET /api/bookings.csv
```

Google Sheets Easy Link also includes a bookings formula:

```text
=IMPORTDATA("https://your-app.com/sheets/bookings.csv?token=YOUR_TOKEN")
```
