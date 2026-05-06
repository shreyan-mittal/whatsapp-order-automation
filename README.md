# WhatsApp Order Automation — Water Distribution Business

An end-to-end automation pipeline that receives WhatsApp orders, extracts structured data using AI, validates products, routes to delivery boys, and logs everything to Google Sheets — with zero manual intervention.

Built for a water distribution business handling 100–200 daily orders across multiple areas of Vadodara, Gujarat.

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
4. If invalid → customer gets an instant WhatsApp reply explaining the error
5. If valid → delivery boy is looked up from a routing sheet based on area and priority
6. A WhatsApp confirmation is sent back to the customer
7. An email notification is sent to the business owner
8. The order is logged to Google Sheets with full metadata

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
        ▼
  Router 1
  ├── [Path 1: valid_order = true]
  │         │
  │         ▼
  │   Google Sheets Routing
  │   (match area → delivery boy by priority)
  │         │
  │         ▼
  │   Google Sheets DeliveryBoys
  │   (get delivery boy details)
  │         │
  │         ▼
  │   Set Variable (build delivery message)
  │         │
  │         ├──────────────┬──────────────┐
  │         ▼              ▼              ▼
  │   WhatsApp reply   Gmail email   Google Sheets
  │   (confirmation)  (notification)  (log order)
  │
  └── [Path 2: valid_order = false]
            │
            ▼
        Router 2
        ├── [not_an_order] → Welcome message → WhatsApp reply
        └── [invalid brand/size] → Rejection message → WhatsApp reply
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Gupshup | WhatsApp Business API — receive & send messages |
| Make.com | Automation orchestration |
| OpenAI GPT-4o-mini | AI-based order extraction & validation |
| Google Sheets | Routing table + delivery boys + order log |
| Gmail | Order notification emails |

---

## Supported Brands & Sizes

| Brand | Sizes |
|-------|-------|
| Bisleri | 20L, 1L, 500ml, 250ml, 200ml, Vedica 1L, Vedica 200ml, Glass Bottle 750ml, Spicy Jeera Soda |
| Bailley | 20L, 1L, 500ml, 250ml |
| Aquafina | 1L, 500ml |
| Wellsun | 1L, 500ml, 200ml |
| Clear | 1L, 500ml, 200ml |
| Aava | 1L, 750ml, 500ml, 250ml, 200ml |
| 7 WTR | 1L, 750ml, 500ml, 250ml |
| Himalayan | All sizes |

---

## Delivery Boy Routing

Orders are routed to delivery boys based on area using a priority system stored in Google Sheets:

| Area | Delivery Boy | Priority |
|------|-------------|----------|
| Subhanpura | Vishal | 1 |
| Alkapuri | Vishal | 1 |
| Alkapuri | Jitu | 2 |
| Gotri | Samadhan | 1 |
| Manjalpur | Jitu | 1 |
| Manjalpur | Jaydeep | 2 |
| ... | ... | ... |

- Priority 1 is always assigned first
- If a delivery boy marks themselves as BUSY, the system automatically falls back to the next priority

---

## Message Handling

| Customer Message | System Response |
|-----------------|----------------|
| Valid order (e.g. Bisleri 20L x2 Akota) | Order confirmed, delivery boy assigned |
| Invalid brand (e.g. Kinley 1L x2 Akota) | Rejection reply with reason |
| Invalid size (e.g. Bisleri 50L x1 Akota) | Rejection reply with reason |
| Greeting (e.g. Hi) | Welcome message with order format instructions |

---

## Google Sheets Schema

Orders are logged with the following columns:

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
├── README.md
├── blueprint/
│   └── Integration_Webhooks_blueprint.json   # Make.com scenario blueprint
└── prompts/
    └── order_extraction_prompt.md            # OpenAI prompt
```

---

## Key Features

- AI-powered parsing — handles natural language orders in any format
- Fuzzy brand matching — handles misspellings like "bislri", "aqaufina"
- Validation layer — rejects invalid brands/sizes before processing
- Smart replies — different WhatsApp responses for valid orders, invalid orders, and greetings
- Priority-based routing — assigns delivery boy by area and priority level
- Full audit trail — every order logged with raw message, AI output and metadata
- IST timestamps — all times stored in India Standard Time
- Instant notifications — WhatsApp reply + email on every order

---

## Setup

To replicate this project:

1. Create a [Gupshup](https://gupshup.io) account and set up a WhatsApp app
2. Import `blueprint/Integration_Webhooks_blueprint.json` into [Make.com](https://make.com)
3. Connect your own: OpenAI API key, Google account, Gupshup API key
4. Set up your Google Sheets with the schema above
5. Add your areas and delivery boys to the Routing and DeliveryBoys sheets
6. Configure your Gupshup webhook URL to point to the Make webhook trigger
7. Activate the scenario

---

## Author

**Shreyan Mittal**
[github.com/shreyan-mittal](https://github.com/shreyan-mittal)
