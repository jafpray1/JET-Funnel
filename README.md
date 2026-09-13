# JET Funnel — lead-magnet landing page

Single static page (no build step) that captures name/email/WhatsApp for a
free ebook, posts the lead to JET Dashboard (`POST /api/leads`, which appends
a row to the "Leads" Google Sheet), and then hands the visitor a WhatsApp
deep link with a **prefilled message** instead of ever messaging them first
(spam-flag risk on a fresh WhatsApp number — see JET Dashboard's ADR 0019).

Deploys to Vercel as-is (static, zero config).

## Before going live, edit `index.html`

- **Copy**: title, benefits, and fine print near the top of `<body>` are all
  placeholders — swap in the real ebook offer.
- **`WA_NUMBER`** (in the `<script>` block near the bottom): the dedicated
  marketing WhatsApp number (international format, digits only, e.g.
  `6281234567890`). This must be a number wired into JET Dashboard's Chatwoot
  inbox for the WA-activation watcher to see the reply (see ADR 0019 there).
- **`WA_PREFILLED_TEXT`**: must contain one of the trigger phrases configured
  in JET Dashboard's `/setup` → Auto-funnel drip card, or the watcher will
  never recognise it.
- **`LEADS_ENDPOINT`**: already points at `https://jet.japri.biz.id/api/leads`
  — only change this if the dashboard moves domains.

## How a submission flows

1. Visitor submits the form → `POST /api/leads` on JET Dashboard.
2. That appends a row to the sheet's `Leads` tab (`Waktu`, `Nama`, `Email`,
   `WhatsApp`, `Sumber`, `Halaman`).
3. JET Dashboard's existing drip cron (every 15 min) emails the ebook link
   per the H+0/H+1/H+3 templates set in `/setup`.
4. The visitor is shown a `wa.me` link with the prefilled trigger phrase —
   if and when they send it, JET Dashboard's WA-activation watcher (every
   5 min) turns the WhatsApp follow-up on for that lead and replies
   immediately.
