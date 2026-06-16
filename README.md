# 📲 Automated WhatsApp Invoice Reminder — [MGI] Penagihan Due Date

> An n8n automation that connects **Odoo ERP** with **WhatsApp Business API** and **Google Sheets** to automatically remind customers of upcoming invoice due dates — reducing late payments and eliminating manual collection effort.

---

## 🔴 Problem

Customer invoices were frequently paid **late or ignored**, with no systematic follow-up mechanism in place. The collections team had to manually track due dates, contact customers one by one via phone or WhatsApp, and log every interaction by hand.

This created three compounding issues:

- **Cash flow disruption** — late payments delayed operational funds.
- **High manual effort** — the finance team spent hours each week chasing individual invoices.
- **Inconsistent follow-up** — reminders were missed or sent too late, especially during busy periods.

---

## 💡 Solution

Built a fully automated invoice reminder pipeline in **n8n** that triggers exactly **2 days before an invoice due date** and sends a personalized WhatsApp message to each customer — with zero manual intervention.

### How it works

```
Odoo ERP
   │
   │  POST /odoo-penagihan
   ▼
[Webhook] ──► [Get row(s) in sheet] ──► [If1: ID exists?]
                                              │
                              ┌───── true ────┘──── false ────┐
                              ▼                               ▼
                    [Update row in sheet1]          [Append row in sheet]
                     (refresh data, stop)                     │
                                                              ▼
                                                    [Edit Fields]
                                                  Format: Rp amount & date
                                                              │
                                                              ▼
                                                  [If: days_until_due == 2?]
                                                              │
                                                    true ─────┘
                                                              │
                                                              ▼
                                                  [Send template] → customer WA
                                                              │
                                                              ▼
                                                  [Update row in sheet]
                                                  Status: "Terkirim" + timestamp
                                                             
```

### Key technical details

| Component | Detail |
|---|---|
| **Trigger** | HTTP POST Webhook from Odoo ERP (`/odoo-penagihan`) |
| **Data store** | Google Sheets — sheet `Tagih Due Date` |
| **Reminder condition** | `days_until_due == 2` (exactly 2 days before due date) |
| **WA template** | `tagihan_jatuh_tempo\|id` (Bahasa Indonesia) |
| **Template variables** | Customer name, invoice number, amount (IDR formatted), due date (long ID format) |
| **Deduplication** | Upsert logic — new invoices are appended, existing ones are updated (not re-sent) |
| **Audit trail** | Sheet status updated to `Terkirim` with Jakarta-timezone timestamp after send |
| **Internal CC** | A second WA copy sent to the internal monitoring number `6285189889978` |
| **Timezone** | `Asia/Jakarta` |

### Payload from Odoo

```json
{
  "id": "INV-001",
  "invoice_number": "INV/2025/0042",
  "partner_name": "PT Maju Bersama",
  "partner_mobile": "628123456789",
  "amount_total": 5000000,
  "due_date": "2025-07-15",
  "days_until_due": 2
}
```

### WhatsApp message template (formatted output)

```
Halo, PT Maju Bersama

Kami ingin mengingatkan bahwa invoice Anda:

No. Invoice : INV/2025/0042
Nominal     : Rp 55.000.000,-
Jatuh Tempo : 15 Juli 2025

akan jatuh tempo dalam 2 hari. Mohon segera melakukan pembayaran.

Terima kasih.
```

---

## ✅ Result

| Metric | Before | After |
|---|---|---|
| On-time payment rate | ~40% | **>80%** ✅ |
| Manual follow-up effort | High (daily, per invoice) | **Zero** |
| Reminder consistency | Inconsistent | **100% automated** |
| Internal visibility | None | **CC copy per send** |

- **80%+ increase** in on-time customer payments within the first month of deployment.
- Finance team fully freed from manual WhatsApp chasing — collections effort reduced to near zero.
- Every reminder is logged automatically in Google Sheets with timestamp and status, giving management full visibility.
- The internal CC mechanism ensures the finance team is always aware of what was sent and when.

---

## 🛠 Tech Stack

- **n8n** — workflow automation engine
- **Odoo ERP** — invoice data source (via webhook push)
- **WhatsApp Business API** — message delivery
- **Google Sheets** — invoice tracking & audit log

---

## 📁 Files

| File | Description |
|---|---|
| `_MGI__Penagihan_Due_Date.json` | n8n workflow export — import directly into your n8n instance |

---

## 🚀 How to Deploy

1. Import `_MGI__Penagihan_Due_Date.json` into your n8n instance.
2. Connect your **Google Sheets OAuth2** credentials and point to your tracking sheet.
3. Connect your **WhatsApp Business API** credentials.
4. Update the webhook URL in your Odoo server action to match the n8n webhook path (`/odoo-penagihan`).
5. Configure your Odoo scheduled action to push invoices where `days_until_due <= 2`.
6. Activate the workflow — it's ready.

---

*Built with n8n · WhatsApp Business API · Google Sheets · Odoo*
