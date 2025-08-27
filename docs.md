# SecretCryptos API – Full Documentation

The **SecretCryptos API** provides secure, simple, and consistent access to our **Crypto Mixer** and **Crypto Exchange** services.  
This document describes the main endpoints and usage examples.

- Base URL: `https://api.secretcryptos.com/v1`  
- Website: [https://secretcryptos.com](https://secretcryptos.com)  
- Live Docs: [https://secretcryptos.com/api](https://secretcryptos.com/api)

---

## 🔑 Authentication
All authenticated requests require an API key.

**Header Example:**
Authorization: Bearer YOUR_API_KEY

- All requests must use **HTTPS**.  
- **Do not** expose your API key in frontend JavaScript. Always keep it server-side.

---

## 🆔 Getting your API Key
1. Go to the [SecretCryptos Partner page](https://secretcryptos.com/partner).  
2. Sign in or create an account.  
3. Navigate to the **API** tab.  
4. Copy your **API Key** (keep it secret).

⚠️ Keep your API key secret. Never share it publicly or hard-code it into public repositories.

---

## 📡 Ping
Simple health check and version.

**Endpoint:**  
GET /v1/ping

**Response Example:**
{
  "ok": true,
  "service": "SecretCryptos API",
  "version": "1.2.6",
  "ts": 1723800000
}

---

## 🗂️ Meta
Root discovery for available meta endpoints.

**Endpoint:**  
GET /v1/meta

**Response Example:**
{
  "ok": true,
  "version": "1.2.6",
  "server_time": {
    "iso": "2025-08-20T16:04:53+00:00",
    "unix": 1755705893
  },
  "docs": {
    "html": "https://secretcryptos.com/api"
  }
}

---

## 🔄 Meta / Mixer
Get per-coin mixer configuration (limits, fees, confirmations).

**Endpoint:**  
GET /v1/meta/mixer

Auth required: Authorization: Bearer YOUR_API_KEY

---

## 🔄 Meta / Exchange
Get supported trading pairs and exchange limits.

**Endpoint:**  
GET /v1/meta/exchange

Auth required: Authorization: Bearer YOUR_API_KEY

---

## 🌀 Mixer – Create Order
Create a new mixer order with one or more outputs.

**Endpoint:**  
POST /v1/mixer/orders

**Headers:**  
Authorization: Bearer YOUR_API_KEY  
Content-Type: application/json

**Body Example:**
{
  "action": "create_order",
  "addresses": [
    {
      "address": "35iMHbUZeTssxBodiHwEEkb32jpBfVueEL",
      "percent": "84.93",
      "delay": "0"
    },
    {
      "address": "1P279UBChDFPAky8S4DcKGaaxKEMYBK9MM",
      "percent": "15.07",
      "delay": "120"
    }
  ],
  "amount": "10.00000000",
  "crypto": "btc",
  "network": "btc",
  "partner": "YOUR_PARTNER_KEY",
  "service_fee": 0.45,
  "qrcode": 1
}

---

## 🔁 Exchange – Create Order
Swap coins between supported networks.

**Endpoint:**  
POST /v1/exchange/orders

**Headers:**  
Authorization: Bearer YOUR_API_KEY  
Content-Type: application/json

**Body Example:**
{
  "from_coin": "eth",
  "from_network": "eth",
  "to_coin": "doge",
  "to_network": "doge",
  "to_address": "DLPaeuaJi2JLUcvYHD4ddLxadwnGaVSt4p",
  "amount": "1.00000000",
  "service_fee": 0.6,
  "partner": "YOUR_PARTNER_KEY",
  "qrcode": 1
}

---

## 💱 Prices
Get the latest market prices (USD).

**Endpoint:**  
GET /v1/prices?symbols=BTC,ETH,USDT  
Authorization: Bearer YOUR_API_KEY

**Response Example:**
{
  "ok": true,
  "base": "USD",
  "ts": 1755600000,
  "prices": {
    "BTC": "117799.51",
    "ETH": "4409.46",
    "USDT": "1.00"
  }
}

---

## 📦 Orders
Check or delete orders.

### ✅ Check Status
POST /v1/orders/check

### ❌ Delete Order
POST /v1/orders/delete

Headers:  
Authorization: Bearer YOUR_API_KEY  
Content-Type: application/json

---

## 🔏 Validate Signature
Validate a digitally signed Letter of Guarantee.

**Endpoint:**  
POST /v1/validate  
Content-Type: application/json

**Body Example:**
{
  "signature": "-----BEGIN DIGITAL SIGNATURE----- ... -----END DIGITAL SIGNATURE-----"
}

---

## 📊 Rate Limits
- Default: 1000 requests/day per API key  
- Higher tiers available upon request
