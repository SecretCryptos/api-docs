# SecretCryptos API – Full Documentation

The **SecretCryptos API** provides secure, simple, and consistent access to our **Crypto Mixer** and **Crypto Exchange** services. This document focuses on the core endpoints to get you started quickly.

- **Base URL:** `https://api.secretcryptos.com/v1`  
- **Website:** https://secretcryptos.com  
- **Live Docs:** https://secretcryptos.com/api

---

## Authentication

All authenticated requests require an API key.

**Header Example**
```
Authorization: Bearer YOUR_API_KEY
```

- All requests must use **HTTPS**.  
- **Do not** expose your API key in frontend JavaScript. Keep it server-side and proxy responses to the browser.

---

## 🆔 Getting your API Key

1. Go to the **Partner** page: https://secretcryptos.com/partner  
2. Sign in or create an account.  
3. Open the top menu and click the **API** tab.  
4. Copy your **API-KEY** (keep it secret; it works for both Mixer and Exchange).

> ⚠️ Keep your API key secret. Never share it publicly or hard-code it into public repositories.

---

## 🔒 Using your API key safely

**cURL (server-side)**
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.secretcryptos.com/v1/meta/mixer"
```

**PHP proxy**
```php
<?php
$ch = curl_init("https://api.secretcryptos.com/v1/meta/exchange");
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, ["Authorization: Bearer YOUR_API_KEY"]);
echo curl_exec($ch);
```

**Node.js (Express)**
```js
import express from "express";
import fetch from "node-fetch";
const app = express();

app.get("/api/meta/exchange", async (req, res) => {
  const r = await fetch("https://api.secretcryptos.com/v1/meta/exchange", {
    headers: { Authorization: "Bearer " + process.env.SECRETCRYPTOS_API_KEY }
  });
  res.status(r.status).type("application/json").send(await r.text());
});

app.listen(3000);
```

**Python (Flask)**
```py
from flask import Flask, Response
import os, requests
app = Flask(__name__)

@app.get("/api/meta/mixer")
def mixer_meta():
    r = requests.get(
        "https://api.secretcryptos.com/v1/meta/mixer",
        headers={"Authorization": f"Bearer {os.environ['SECRETCRYPTOS_API_KEY']}"}
    )
    return Response(r.content, status=r.status_code, mimetype="application/json")
```

---

## 📡 Ping

Simple health check and version.

**Endpoint**  
`GET /v1/ping`

**Response Example**
```json
{
  "ok": true,
  "service": "SecretCryptos API",
  "version": "1.2.6",
  "ts": 1723800000
}
```

---

## 🗂️ Meta

Root discovery for available meta endpoints.

**Endpoint**  
`GET /v1/meta`

**Response Example**
```json
{
  "ok": true,
  "version": "1.2.6",
  "server_time": { "iso": "2025-08-20T16:04:53+00:00", "unix": 1755705893 },
  "docs": { "html": "https://secretcryptos.com/api" },
  "endpoints": [
    { "name": "Ping", "path": "/v1/ping", "method": "GET", "auth": false, "desc": "Simple health check." },
    { "name": "Meta (Mixer)", "path": "/v1/meta/mixer", "method": "GET", "auth": true, "desc": "Supported coins/networks and mixer rules." },
    { "name": "Meta (Exchange)", "path": "/v1/meta/exchange", "method": "GET", "auth": true, "desc": "Supported trading pairs and exchange limits." },
    { "name": "Create Mixer Order", "path": "/v1/mixer/orders", "method": "POST", "auth": true, "desc": "Create a new mixer order." },
    { "name": "Create Exchange Order", "path": "/v1/exchange/orders", "method": "POST", "auth": true, "desc": "Create a new exchange order." },
    { "name": "Check Order", "path": "/v1/orders/check", "method": "POST", "auth": true, "desc": "Query order status by trackcode." },
    { "name": "Delete Order", "path": "/v1/orders/delete", "method": "POST", "auth": true, "desc": "Delete an order (if allowed)." },
    { "name": "Prices", "path": "/v1/prices", "method": "GET", "auth": true, "desc": "Current exchange rates and prices." },
    { "name": "Validate Signature", "path": "/v1/validate", "method": "POST", "auth": false, "desc": "Validate a digital guarantee letter signature." }
  ]
}
```

---

## 🔄 Meta / Mixer

Per-coin mixer configuration (min/max, service/per-address fees, decimals, confirmations).

**Endpoint**  
`GET /v1/meta/mixer` (Auth required)

**cURL**
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.secretcryptos.com/v1/meta/mixer"
```

**Response (excerpt)**
```json
{
  "ok": true,
  "mixer": {
    "BTC": {
      "name": "Bitcoin",
      "symbol": "₿",
      "service_fee": 0.1,
      "maintenance": 0,
      "per_address_fee": "0.00005000",
      "decimals": 8,
      "confirmations": 1,
      "networks": {
        "btc": { "label": "Bitcoin (BTC)", "min": 0.001, "max": 20 }
      }
    }
  }
}
```

---

## 🔄 Meta / Exchange

Exchange configuration: maintenance, live USD prices, and per-network limits/fees.

**Endpoint**  
`GET /v1/meta/exchange` (Auth required)

**cURL**
```bash
curl -H "Authorization: Bearer YOUR_API_KEY" \
  "https://api.secretcryptos.com/v1/meta/exchange"
```

**Response (excerpt)**
```json
{
  "ok": true,
  "exchange": {
    "maintenance": 0,
    "prices_usd": { "BTC": "117799.51713382", "ETH": "4409.46254479", "USDT": "1.0005594" },
    "outputs": {
      "BTC": {
        "name": "Bitcoin",
        "symbol": "₿",
        "networks": {
          "btc": { "label": "Bitcoin (BTC)", "min_usd": "100", "max_usd": "182520", "service_fee": "0.5" }
        }
      }
    }
  }
}
```

---

## 🌀 MIXER

The **Mixer API** lets you programmatically create privacy-preserving transactions by splitting to one or more output addresses with optional delays.

- Use cases: secure payment intake, revenue sharing, delayed payouts.  
- Get limits & fees via `/v1/meta/mixer` before creating orders.  
- Amounts are in **coin units**. Timestamps are **Unix seconds (GMT-3)**.

### MIXER / Create Order

**Endpoint**  
`POST /v1/mixer/orders`

**Headers**
```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

**Validation Rules**
- `amount` must be within coin *min/max* from **Meta / Mixer**.
- `addresses`: 1–10 outputs.
- `percent`
  - Up to 2 decimals (e.g., `10`, `10.5`, `10.50`).
  - Per address `1.00`–`100.00`; total must be exactly `100.00`.
  - If single output, it must be exactly `100.00`.
- `delay`: minutes (e.g., `120`) or label (`"2h 0m"`); **max 48h** (`2880` minutes).
- `service_fee` (optional): `0.10`–`5.00` (%).
- Address format must match the network; `destination_tag/memo` required for some networks (e.g., **XRP**, **TON**).

**Body Example (BTC, 2 outputs)**
```json
{
  "action": "create_order",
  "addresses": [
    { "address": "35iMHbUZeT...", "percent": "84.93", "delay": "0" },
    { "address": "1P279UBChD...", "percent": "15.07", "delay": "120" }
  ],
  "amount": "10.00000000",
  "crypto": "btc",
  "network": "btc",
  "partner": "YOUR_PARTNER_KEY",
  "service_fee": 0.45,
  "qrcode": 1
}
```

**Body Example (XRP with tag)**
```json
{
  "action": "create_order",
  "addresses": [
    { "address": "raGXwk3P9y...", "percent": "100", "delay": "0", "destination_tag": "435757008" }
  ],
  "amount": "10.00000000",
  "crypto": "btc",
  "network": "btc",
  "partner": "YOUR_PARTNER_KEY",
  "service_fee": 0.45,
  "qrcode": 1
}
```

**How to use the QR Code**
When `qrcode=1`, the response includes a Base64 data URL under `deposit.qr_code`. Embed directly:
```html
<img src="data:image/png;base64,..." alt="Scan to pay" />
```

**Response (excerpt)**
```json
{
  "ok": true,
  "trackcode": "6A5FB3BA8A150EC9",
  "deposit": {
    "address": "31wXuLH5AK...",
    "network": "btc",
    "deposit_amount": "10.00000000",
    "min_amount": 0.001,
    "max_amount": 20,
    "expires_at": 1755614593,
    "qr_code": "data:image/png;base64,iVBORw0..."
  },
  "fees": {
    "service_fee_percent": 0.45,
    "service_fee_value": "0.04500000",
    "fee_per_output": "0.00005000",
    "fee_outputs_total": "0.00010000",
    "fee_total": "0.04510000"
  },
  "outputs": [
    { "id": 1, "address": "35iMHbUZe...", "share_percent": 84.93, "delay_minutes": 0, "amount": "8.45469657" },
    { "id": 2, "address": "1P279UBCh...", "share_percent": 15.07, "delay_minutes": 120, "amount": "1.50020343" }
  ],
  "signature_text": "..."
}
```

**PHP Example**
```php
<?php
$url = "https://api.secretcryptos.com/v1/mixer/orders";
$headers = [
  "Authorization: Bearer YOUR_API_KEY",
  "Content-Type: application/json"
];
$data = [
  "action" => "create_order",
  "addresses" => [
    ["address"=>"35iMHbUZeT...","percent"=>"84.93","delay"=>"0"],
    ["address"=>"1P279UBChD...","percent"=>"15.07","delay"=>"120"]
  ],
  "amount" => "10.00000000",
  "crypto" => "btc",
  "network" => "btc",
  "partner" => "YOUR_PARTNER_KEY",
  "service_fee" => 0.45
];
$ch = curl_init($url);
curl_setopt_array($ch, [
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_HTTPHEADER => $headers,
  CURLOPT_POST => true,
  CURLOPT_POSTFIELDS => json_encode($data, JSON_UNESCAPED_UNICODE|JSON_UNESCAPED_SLASHES)
]);
echo curl_exec($ch);
curl_close($ch);
```

**Common Errors**
```json
{ "ok": false, "code": "BAD_REQUEST", "message": "Sum of percents must be exactly 100.00" }
```

---

## 🔁 EXCHANGE

Swap a **from_coin/from_network** deposit into a single **to_coin/to_network** destination.

- Get pair limits & default fee via `/v1/meta/exchange`.  
- Amounts are in **coin units**; limits are **USD-based**.

### EXCHANGE / Create Order

**Endpoint**  
`POST /v1/exchange/orders`

**Headers**
```
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json
```

**Validation Rules**
- `amount × from_price(USD)` must be within `min_usd/max_usd` for the pair.
- `to_address` format must match `to_network`.
- `destination_tag/memo` may be required (e.g., **XRP**, **TON**).
- `service_fee` (optional, %): up to 2 decimals. Global clamp: **min 0.50%**, **max 3.00%**.  
  If omitted → effective fee = `max(site_default, 0.50%)`.

**Body Example**
```json
{
  "from_coin": "eth",
  "from_network": "eth",
  "to_coin": "doge",
  "to_network": "doge",
  "to_address": "DLPaeuaJi2...",
  "amount": "1.00000000",
  "service_fee": 0.6,
  "partner": "YOUR_PARTNER_KEY",
  "qrcode": 1
}
```

**Body Example (XRP / TON)**
```json
{
  "from_coin": "usdt",
  "from_network": "trx",
  "to_coin": "xrp",
  "to_network": "xrp",
  "to_address": "rLWyHZwAhs...",
  "destination_tag": "123456",
  "amount": "250"
}
---
{
  "from_coin": "btc",
  "from_network": "btc",
  "to_coin": "ton",
  "to_network": "ton",
  "to_address": "UQDTQmCrng...",
  "memo": "MYMEMO123",
  "amount": "0.015"
}
```

**Response (excerpt)**
```json
{
  "ok": true,
  "type": "exchange",
  "trackcode": "D391FC08747E7B04",
  "pair": { "from": "ETH_eth", "to": "DOGE_doge" },
  "deposit": {
    "address": "0x7ed2bf65...",
    "network": "eth",
    "deposit_amount": "1.00000000",
    "min_usd": 100,
    "max_usd": 88565,
    "qr_code": "data:image/png;base64,..."
  },
  "quote": {
    "from": { "coin":"ETH","network":"eth","price_usd":4300.8836,"amount":"1.00000000","amount_usd":"4300.88362842" },
    "to":   { "coin":"DOGE","network":"doge","price_usd":0.22014648,"estimated_receive":"19419.24455241" },
    "fee_percent": 0.6,
    "final_usd": "4275.07832665"
  },
  "receive": {
    "address":"DLPaeuaJi2...",
    "coin":"DOGE","network":"doge",
    "percent":100,"delay_minutes":10,"delay_label":"0h 10m",
    "amount":"19419.24455241"
  },
  "signature_text":"..."
}
```

**PHP Example**
```php
<?php
$url = "https://api.secretcryptos.com/v1/exchange/orders";
$headers = [
  "Authorization: Bearer YOUR_API_KEY",
  "Content-Type: application/json"
];
$payload = [
  "from_coin"=>"eth","from_network"=>"eth",
  "to_coin"=>"doge","to_network"=>"doge",
  "to_address"=>"DLPaeuaJi2...",
  "amount"=>"1.00000000",
  "service_fee"=>0.6,
  "partner"=>"YOUR_PARTNER_KEY",
  "qrcode"=>1
];
$ch = curl_init($url);
curl_setopt_array($ch, [
  CURLOPT_RETURNTRANSFER=>true,
  CURLOPT_HTTPHEADER=>$headers,
  CURLOPT_POST=>true,
  CURLOPT_POSTFIELDS=>json_encode($payload, JSON_UNESCAPED_UNICODE|JSON_UNESCAPED_SLASHES),
]);
echo curl_exec($ch);
curl_close($ch);
```

**QR Code**  
When `qrcode=1`, embed `deposit.qr_code` as an `<img>` to let users scan the deposit address.

---

## 💱 PRICES / Get Market Rates

Retrieve the latest market prices in **USD**.

**Endpoint**  
`GET /v1/prices?symbols=BTC,ETH,DOGE` (Auth required)

**Query**
- `symbols` *(optional)*: comma-separated tickers. If omitted, all supported coins are returned.

**Example Response**
```json
{
  "ok": true,
  "base": "USD",
  "ts": 1755600000,
  "prices": {
    "BTC": "117799.51713382",
    "ETH": "4409.46254479",
    "USDT": "1.0005594"
  }
}
```

**PHP (only BTC, ETH)**
```php
<?php
$url = "https://api.secretcryptos.com/v1/prices?symbols=" . urlencode("BTC,ETH");
$headers = ["Authorization: Bearer <API_KEY>"];
$ch = curl_init($url);
curl_setopt_array($ch, [CURLOPT_RETURNTRANSFER=>true, CURLOPT_HTTPHEADER=>$headers, CURLOPT_TIMEOUT=>20]);
$result = curl_exec($ch);
echo $result === false ? curl_error($ch) : $result;
curl_close($ch);
```

**PHP (all symbols)**
```php
<?php
$url = "https://api.secretcryptos.com/v1/prices";
$headers = ["Authorization: Bearer <API_KEY>"];
$ch = curl_init($url);
curl_setopt_array($ch, [CURLOPT_RETURNTRANSFER=>true, CURLOPT_HTTPHEADER=>$headers, CURLOPT_TIMEOUT=>20]);
$result = curl_exec($ch);
header("Content-Type: application/json; charset=utf-8");
echo $result === false ? json_encode(["error"=>curl_error($ch)]) : $result;
curl_close($ch);
```

**Errors**
- `BAD_REQUEST` (invalid `symbols`)  
- `UNAUTHORIZED/FORBIDDEN` (missing/invalid key)  
- `SERVER_ERROR`

---

## 📦 ORDERS

### ✅ ORDERS / Check Status (Mixer & Exchange)

Use a single endpoint to check status of both **mixer** and **exchange** orders.

**Endpoint**  
`POST /v1/orders/check`

**Body**
- `trackcode` *(required)*: 16-char uppercase code  
- `type` *(optional)*: `mixer` | `exchange`  
- `outputs` *(optional)*: `none` | `summary` | `full` (default `summary`)  
- `mask_addresses` *(optional)*: `true` (default) | `false`

**Response Notes**
- `deposit.confirm_status`: `0` none, `1` received/pending, `2` fully confirmed  
- `deposit.funding`: `waiting_balance`, `received_balance`, `remaining_need`, `is_fully_funded`, `can_start`  
- `outputs`: depends on mode; `full` returns per-output details and `delay_seconds`

**Business Rules**
- Order **does not start** unless fully funded (`is_fully_funded=false`).  
- When it becomes valid (confirmed + fully funded), outputs are timestamped once and affiliate earnings are credited.

**PHP Example**
```php
<?php
$url = "https://api.secretcryptos.com/v1/orders/check";
$headers = ["Authorization: Bearer <API_KEY>","Content-Type: application/json"];
$data = [
  "trackcode" => "554FEC10054743FD",
  "type" => "mixer",
  "outputs" => "full",
  "mask_addresses" => false
];
$ch = curl_init($url);
curl_setopt_array($ch, [CURLOPT_RETURNTRANSFER=>true, CURLOPT_HTTPHEADER=>$headers, CURLOPT_POST=>true, CURLOPT_POSTFIELDS=>json_encode($data)]);
echo curl_exec($ch);
curl_close($ch);
```

**Example (Fully funded, excerpt)**
```json
{
  "ok": true,
  "type": "mixer",
  "trackcode": "6A5FB3BA8A150EC9",
  "deposit": {
    "confirm_status": 2,
    "coin": "btc",
    "network": "btc",
    "delete_in_sec": 167812,
    "funding": {
      "waiting_balance": "10.00000000",
      "received_balance": "10.00000000",
      "remaining_need": "0.00000000",
      "is_fully_funded": true,
      "can_start": true
    }
  },
  "outputs": [
    { "index": 1, "address": "35iMHbUZe...", "state": 2, "confirm": 1, "amount": "8.45469657" },
    { "index": 2, "address": "1P279UBCh...", "state": 1, "confirm": 0, "amount": "1.50020343" }
  ]
}
```

**Example (Underfunded, excerpt)**
```json
{
  "ok": true,
  "type": "exchange",
  "trackcode": "A1B2C3D4E5F6A7B8",
  "deposit": {
    "confirm_status": 2,
    "coin": "usdt",
    "network": "erc20",
    "delete_in_sec": 14321,
    "funding": {
      "waiting_balance": "500.00000000",
      "received_balance": "420.00000000",
      "remaining_need": "80.00000000",
      "is_fully_funded": false,
      "can_start": false
    }
  },
  "status_reason": "INSUFFICIENT_FUNDS",
  "outputs": { "count": 3, "sent_count": 0, "sent_total": "0.00000000" }
}
```

**Errors**
- `BAD_REQUEST` (missing/invalid `trackcode`)  
- `NOT_FOUND` (order not found)  
- `UNAUTHORIZED/FORBIDDEN` (invalid/disabled key)  
- `TOO_MANY_REQUESTS` (daily limit)  
- `SERVER_ERROR`

---

### ❌ ORDERS / Delete

Delete an existing order so it can no longer be accessed through the API.

**Endpoint**  
`POST /v1/orders/delete`

**Body**
- `trackcode` *(required)*  
- `type` *(optional)*: `mixer` | `exchange` (if omitted, auto-detect)

**Success**
```json
{ "ok": true, "trackcode": "D391FC08747E7B04", "type": "exchange", "deleted": true }
```

**Not Found**
```json
{ "ok": false, "code": "NOT_FOUND", "message": "Order not found or already deleted" }
```

**PHP Example**
```php
<?php
$url = "https://api.secretcryptos.com/v1/orders/delete";
$headers = ["Authorization: Bearer <API_KEY>", "Content-Type: application/json"];
$data = ["trackcode" => "D391FC08747E7B04"];
$ch = curl_init($url);
curl_setopt_array($ch, [CURLOPT_RETURNTRANSFER=>true, CURLOPT_HTTPHEADER=>$headers, CURLOPT_POST=>true, CURLOPT_POSTFIELDS=>json_encode($data)]);
echo curl_exec($ch);
curl_close($ch);
```

---

## 🔏 VALIDATE / Signature

Validate a digitally signed payload (Letter of Guarantee). The API returns a verified subset of related order details.

**Endpoint**  
`POST /v1/validate`

**Body**
- `signature` *(required)*: digital signature block (raw Base64 or with BEGIN/END lines)

**PHP Example**
```php
<?php
$url = "https://api.secretcryptos.com/v1/validate";
$signatureBlock = <<<SIG
-----BEGIN DIGITAL SIGNATURE-----
eyJ0cmFjayI6ICJEMzkxRkMwODc0N0U3QjA0IiwgInR5cGUiOiAibWl4ZXIiLCAiZXh0cmEiOiAiLi4uIn0=
-----END DIGITAL SIGNATURE-----
SIG;
$headers = ["Authorization: Bearer <API_KEY>", "Content-Type: application/json", "Accept: application/json"];
$payload = ["signature" => $signatureBlock];
$ch = curl_init($url);
curl_setopt_array($ch, [
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_HTTPHEADER => $headers,
  CURLOPT_POST => true,
  CURLOPT_POSTFIELDS => json_encode($payload, JSON_UNESCAPED_UNICODE|JSON_UNESCAPED_SLASHES),
  CURLOPT_TIMEOUT => 20,
  CURLOPT_SSL_VERIFYPEER => true,
  CURLOPT_SSL_VERIFYHOST => 2
]);
echo curl_exec($ch);
curl_close($ch);
```

**Success (example)**
```json
{
  "ok": true,
  "message": "The signature has been validated successfully.",
  "type": "mixer",
  "route": "deposit",
  "trackcode": "6A5FB3BA8A150EC9",
  "order": {
    "deposit_address": "31wXuLH5AK...",
    "service_fee": "0.45",
    "coin": "btc",
    "network": "btc",
    "waiting_balance": "10.00000000",
    "created_at": "1755698732"
  },
  "outputs": [
    { "coin": "btc", "network": "btc", "address": "35iMHbUZe...", "share_percent": "84.93", "delay_minutes": 0, "delay_label": "0h 0m", "amount": "8.45469657" },
    { "coin": "btc", "network": "btc", "address": "1P279UBCh...", "share_percent": "15.07", "delay_minutes": 120, "delay_label": "2h 0m", "amount": "1.50020343" }
  ],
  "outputs_count": 2
}
```

**Error Examples**
```json
{ "ok": false, "error": "BAD_REQUEST", "message": "Digital signature is required" }
---
{ "ok": false, "error": "INVALID_SIGNATURE", "message": "The provided data is not valid Base64 or is too short" }
---
{ "ok": false, "error": "DECRYPTION_FAILED", "message": "Decryption failed. Ensure that the provided signature is valid and retry." }
---
{ "ok": false, "error": "MISSING_TRACKCODE", "message": "Trackcode is required in the signature payload." }
---
{ "ok": false, "error": "NOT_FOUND", "message": "The requested order details could not be found." }
```

**Notes**
- Numeric precision: `waiting_balance` & `amount` → 8 decimals (string), `share_percent` & `service_fee` → 2 decimals.  
- Only recent orders (within **48h**) are eligible.

---

## 📊 Rate Limits

**Default:** 1000 requests / day / API key  
Higher tiers are available upon request.

---
