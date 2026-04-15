# WhatsApp Order Automation — Water Distribution Business

An end-to-end automation pipeline that receives WhatsApp orders, extracts structured data using AI, validates products, routes to delivery boys, and logs everything to Google Sheets — with zero manual intervention.

Built for a water distribution business handling 100–200 daily orders across multiple areas.

---

## How it works

A customer sends a WhatsApp message like:
```
Bisleri 20L x2 Akota
```

Within seconds:
1. The order is received via a Gupshup webhook
2. GPT-4o-mini extracts brand, size, quantity and area
3. The order is validated against allowed brands and sizes
4. A delivery boy is looked up from a routing sheet based on area
5. A WhatsApp confirmation is sent back to the customer
6. An email notification is sent to the business owner
7. The order is logged to Google Sheets with full metadata

---

## Architecture

```
Customer WhatsApp
        │
        ▼
  Gupshup (v2 webhook)
        │
        ▼
  Make.com Webhook (trigger)
        │
        ▼
  OpenAI GPT-4o-mini
  (extract: brand, size, qty, area, valid_order, exception_reason)
        │
        ▼
  JSON Parse
        │
   [Filter: valid_order = true]
        │
        ├──────────────────────────────────────┐
        ▼                                      ▼
  Google Sheets                         [Invalid orders]
  Routing lookup                         stop here
  (match area → delivery boy)
        │
        ▼
  Set Variable
  (build delivery message)
        │
        ├─────────────────┬──────────────────┐
        ▼                 ▼                  ▼
  WhatsApp reply     Gmail email       Google Sheets
  (Gupshup HTTP)    (notification)     (log order row)
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Gupshup | WhatsApp Business API — receive & send messages |
| Make.com | Automation orchestration |
| OpenAI GPT-4o-mini | AI-based order extraction & validation |
| Google Sheets | Routing table + order log database |
| Gmail | Order notification emails |

---

## AI Prompt — Order Extraction

The GPT-4o-mini prompt validates orders against allowed brands and sizes, returning structured JSON:

```json
{
  "brand": "Bisleri",
  "size": "20L",
  "quantity": 2,
  "area": "Akota",
  "valid_order": true,
  "exception_reason": ""
}
```

Invalid orders (wrong brand, wrong size, missing info) are flagged with `valid_order: false` and an `exception_reason` — and stopped before reaching the database.

---

## Supported Brands & Sizes

| Brand | Sizes |
|-------|-------|
| Bisleri | 20L, 1L, 500ml, 200ml |
| Bailley | 20L, 1L, 500ml, 200ml |
| Clear | TBD |
| Aquafina | TBD |
| Wellsun | TBD |
| Aava | TBD |
| 7 WTR | TBD |
| Himalayan | TBD |

---

## Google Sheets Schema

Orders are logged to a sheet with the following columns:

| Column | Field |
|--------|-------|
| A | order_id (timestamp) |
| B | order_time (IST) |
| C | customer_phone |
| D | customer_name |
| E | address_full |
| F | area |
| G | delivery_date |
| H | delivery_slot |
| I | items_json |
| J | items_summary |
| K | payment_mode |
| L | payment_status |
| M | amount_total |
| N | razorpay_paymentlink_id |
| O | assigned_delivery_boy |
| P | delivery_status |
| Q | delivered_time |
| R | conversation_stage |
| S | ai_confidence |
| T | exception_flag |
| U | exception_reason |
| V | raw_message |

---

## Files in this repo

```
├── README.md                          # This file
├── blueprint/
│   └── Integration_Webhooks_blueprint.json   # Make.com scenario blueprint
└── prompts/
    └── order_extraction_prompt.md     # OpenAI prompt used in the pipeline
```

---

## Key Features

- **AI-powered parsing** — handles natural language orders in any format
- **Validation layer** — rejects invalid brands/sizes before processing
- **Area-based routing** — automatically assigns delivery boy based on customer area
- **Full audit trail** — every order logged with raw message, AI output and metadata
- **IST timestamps** — all times stored in India Standard Time
- **Instant notifications** — WhatsApp reply + email on every order

---

## Setup

To replicate this project:

1. Create a [Gupshup](https://gupshup.io) account and set up a WhatsApp app
2. Import the `blueprint/Integration_Webhooks_blueprint.json` into [Make.com](https://make.com)
3. Connect your own: OpenAI API key, Google account, Gupshup API key
4. Set up your Google Sheets with the schema above
5. Configure your routing sheet with areas and delivery boy assignments
6. Add your Gupshup webhook URL pointing to the Make webhook trigger
7. Activate the scenario

---

## Author

**Shreyan Mittal**  
[github.com/shreyan-mittal](https://github.com/shreyan-mittal)
