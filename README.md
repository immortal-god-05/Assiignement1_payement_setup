# ShopEase — Premium E-Commerce Store

> A fully functional e-commerce web app with Razorpay payments, Google Sheets as a database, and zero hosting costs.

---

# 🚀 [CLICK HERE TO VIEW LIVE APP → test2905.netlify.app](https://test2905.netlify.app)

> **✅ WORKING HOSTED LINK:** **[https://test2905.netlify.app](https://test2905.netlify.app)**

---

## What This App Does

ShopEase is a complete online store where customers can browse products, add them to a cart, fill in their details, and pay securely via Razorpay. Every successful order is automatically recorded in a Google Sheet. There is no traditional server — the entire backend runs on Google Apps Script.

---

## How It Works — The Full Flow

```
Customer visits index.html
        ↓
Products load from Google Sheets (via Apps Script)
        ↓
Customer adds items to cart → fills name, phone, email, address
        ↓
Frontend asks Apps Script to create a Razorpay Order (server-side)
        ↓
Razorpay payment popup opens → customer pays
        ↓
Razorpay returns payment proof (payment_id + signature)
        ↓
Apps Script verifies the signature using HMAC-SHA256
        ↓
Order saved to Google Sheets → "Order Confirmed" shown to customer
```

---

## Architecture

### Frontend — `index.html`
A single HTML file that runs entirely in the browser. No framework, no build tools, no npm. It handles:

- Loading and displaying products from the sheet
- Cart management (add, remove, change quantity)
- Customer detail form with validation
- Calling the backend to create orders and verify payments
- Showing order confirmation or error screens

### Backend — Google Apps Script (`Code.gs`)
Acts as a serverless API. It exposes two endpoints via HTTP:

| Method | Action | What it does |
|--------|--------|--------------|
| GET | `?action=products` | Reads all rows from the Products sheet and returns them as JSON |
| POST | `action: createOrder` | Calls Razorpay's API server-side to create a payment order |
| POST | `action: verifyOrder` | Verifies the payment signature and saves the order to the Orders sheet |

The Apps Script URL is the only thing connecting the frontend to the backend. The Razorpay secret key **never leaves the Apps Script** — it is never sent to the browser.

### Database — Google Sheets
Two tabs act as the database:

**Products tab** — stores what's for sale:
| Column | Description |
|--------|-------------|
| ID | Unique product ID |
| Name | Product name |
| Price | Price in ₹ (not paise) |
| Description | Short product description |
| Image | Unsplash or any image URL |
| Badge | Optional label: Popular, New, Sale, Hot |

**Orders tab** — auto-filled on every successful purchase:
| Column | Description |
|--------|-------------|
| Order ID | Internal order number (ORD-timestamp) |
| Timestamp | Date and time in IST |
| Customer Name | From checkout form |
| Email | From checkout form |
| Phone | From checkout form |
| Address | Delivery address |
| Product ID / Name | One row per item in the cart |
| Qty / Unit Price / Item Total | Per-item breakdown |
| Grand Total | Full order value |
| Razorpay Payment ID | e.g. `pay_SNwy...` |
| Razorpay Order ID | e.g. `order_SNwy...` |
| Status | `PAID ✓` |

---

## Payment Flow (Razorpay)

Razorpay requires a two-step process for secure payments:

**Step 1 — Order Creation (server-side)**
The frontend asks Apps Script to create an order. Apps Script calls Razorpay's API using the secret key and returns an `order_id`. This ensures the amount cannot be tampered with by the browser.

**Step 2 — Payment (browser)**
The Razorpay checkout popup opens using the `order_id`. The customer pays via UPI, card, netbanking, or wallet.

**Step 3 — Signature Verification (server-side)**
After payment, Razorpay sends back a `payment_id` and a `signature`. Apps Script recomputes the expected signature using:
```
HMAC_SHA256(order_id + "|" + payment_id, razorpay_secret)
```
If it matches → payment is genuine → order is saved to the sheet.

This cryptographic check prevents anyone from faking a successful payment.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| Backend | Google Apps Script (serverless) |
| Database | Google Sheets |
| Payments | Razorpay (test mode / live mode) |
| Fonts | Google Fonts (Cormorant Garamond + Inter) |
| Images | Unsplash (free, no API key needed) |
| Hosting | Any static host (GitHub Pages, Netlify, local file) |

---

## Features

- **Product catalogue** loaded live from Google Sheets
- **Category filters** — Audio, Computing, Accessories, Lighting, Storage
- **Cart drawer** with quantity controls and live subtotal
- **Skeleton loading** placeholders while products fetch
- **Customer form** with validation (name, phone, email, address)
- **Secure Razorpay checkout** with UPI, cards, netbanking, wallets
- **HMAC-SHA256 signature verification** — fraud-proof payment confirmation
- **Auto order logging** to Google Sheets with full details
- **Toast notifications** for cart actions and payment status
- **Fully responsive** — works on mobile, tablet, and desktop
- **Zero backend cost** — Google Apps Script free tier is more than enough

---

## Data Security

- The Razorpay **secret key** lives only in Apps Script — never in the HTML file
- The Razorpay **public key** (`rzp_test_...`) is safe to be in the frontend
- Payment authenticity is verified server-side using cryptographic signatures
- Google Sheets access is restricted by OAuth scopes set in `appsscript.json`

---

## Project Structure

```
Your Computer/
└── index.html        ← Entire frontend (UI + JS logic). Open this in a browser.
```

```
Google (script.google.com) — completely separate, lives in your Google account
├── Code.gs           ← Backend API (products, orders, Razorpay)
└── appsscript.json   ← Permissions and deployment config
```

```
Google Sheets (sheets.google.com) — also separate, linked by Sheet ID
├── Products tab      ← Your product catalogue (edit here to add/remove products)
└── Orders tab        ← Auto-filled every time a customer pays
```

> These three parts are independent. `index.html` is just a file on your computer (or any host).
> Apps Script and Google Sheets are both cloud tools inside your Google account.
> They talk to each other via the Apps Script deployment URL.

---

## Where to Put Each Key / ID

There are **4 values** you need to configure. Here is exactly where each one goes:

---

### 1. `SHEET_ID` → goes in `Code.gs` (Apps Script)
This is the ID of your Google Sheet — found in the sheet's URL:
```
https://docs.google.com/spreadsheets/d/  ➜ SHEET_ID_IS_HERE  ◀  /edit
```
In `Code.gs`, paste it at the very top:
```javascript
var SHEET_ID = "paste-your-sheet-id-here";
```

---

### 2. `RZP_KEY_ID` (Razorpay Public Key) → goes in `Code.gs` (Apps Script)
This is your Razorpay Key ID, starts with `rzp_test_` (test mode) or `rzp_live_` (live mode).
Find it at: **Razorpay Dashboard → Settings → API Keys**
```javascript
var RZP_KEY_ID = "rzp_test_xxxxxxxxxxxx";
```
> Apps Script sends this back to the browser so Razorpay checkout can open. It is safe to expose.

---

### 3. `RZP_SECRET` (Razorpay Secret Key) → goes in `Code.gs` (Apps Script)
This is the secret that pairs with your Key ID. **Never put this in index.html.**
```javascript
var RZP_SECRET = "your-razorpay-secret-here";
```
> This never leaves Apps Script. It is used only server-side to verify payment signatures.

---

### 4. `BACKEND` (Apps Script Deployed URL) → goes in `index.html`
After you deploy your Apps Script as a Web App, you get a URL like:
```
https://script.google.com/macros/s/AKfycb.../exec
```
In `index.html`, find this line near the top of the `<script>` section and paste your URL:
```javascript
var BACKEND = "https://script.google.com/macros/s/AKfycb.../exec";
```
> Every time you make changes to `Code.gs` and redeploy, you get a **new URL**. Remember to update this line in `index.html` each time.

---

### Quick Reference Table

| Value | What it is | Where it goes | Safe to share? |
|-------|-----------|---------------|----------------|
| `SHEET_ID` | Google Sheet ID from the URL | `Code.gs` top | ⚠️ Keep private |
| `RZP_KEY_ID` | Razorpay public key (`rzp_test_...`) | `Code.gs` top | ✅ Yes, it's public |
| `RZP_SECRET` | Razorpay secret key | `Code.gs` top | ❌ Never share |
| `BACKEND` | Apps Script deployed URL | `index.html` | ⚠️ Keep private |

---

## Built By

**IMMORTAL** · Payments secured by Razorpay · Data stored in Google Sheets
