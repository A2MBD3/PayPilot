# PayPilot API — v1

> The PayPilot REST API is not limited to the in-app dashboard feed — a client
> website can **verify a transaction by its TrxID** directly through the API.
> বাংলা ভার্সন: [`docs/API.bn.md`](API.bn.md)

- **Base URL:** `https://paypilot-5p9t.onrender.com`
- **Format:** JSON request/response, UTF-8
- **Auth:** `Authorization: Bearer <token>` (license key / web token / owner token)
- **Rate limits:** general API **300 req/min** · device routes **600 req/min** · login **30 req/15 min**

---

## Response envelope

Success:

```json
{ "success": true, "message": "...", "data": { } }
```

Failure:

```json
{ "success": false, "code": "NOT_FOUND", "message": "Transaction not found" }
```

| HTTP | Code | Meaning |
|------|------|---------|
| 400 | `VALIDATION_ERROR` | Malformed input |
| 401 | `UNAUTHORIZED` | Missing / invalid / expired token |
| 401 | `TOKEN_REVOKED` | License was regenerated — old token is dead |
| 403 | `DISABLED` | Merchant account suspended |
| 401 | `INVALID_CREDENTIALS` | Wrong username/password |
| 404 | `NOT_FOUND` | Resource not found |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |

---

## 1) Verify a transaction (the website-integration endpoint)

### `POST /api/v1/verify`

Verifies a payment by TrxID. This is a **one-time claim** — on success the
transaction becomes `used` and can **never be verified again**, which blocks
replay/duplicate-order abuse. For a read-only lookup use
`GET /api/v1/transactions/{trxId}`.

**Auth:** merchant license key (Bearer)

**Request**

```json
{
  "trx_id": "DI739OTDF3",
  "amount": 100.00,
  "order_id": "ORDER-1024",
  "max_age_minutes": 60
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `trx_id` | string (5–30) | ✅ | TrxID from the bKash payment SMS |
| `amount` | number > 0 | ❌ | When present the server matches the amount (±0.001) |
| `order_id` | string ≤ 100 | ❌ | Your order reference — stored with the transaction |
| `max_age_minutes` | int 1–1440 | ❌ | Default 60 minutes; older transactions return `EXPIRED` |

**Success `200`**

```json
{
  "success": true,
  "code": "VERIFIED",
  "message": "Payment verified successfully",
  "data": {
    "trx_id": "DI739OTDF3",
    "amount": 100.0,
    "sender": "01722858922",
    "status": "used",
    "category": "claimed",
    "app": "My Shop bKash",
    "received_at": "2026-09-08T16:32:00.000Z",
    "used_at": "2026-09-08T16:40:12.480Z",
    "order_id": "ORDER-1024"
  }
}
```

**Failure codes (HTTP 400):**

| Code | Meaning |
|------|---------|
| `NOT_FOUND` | No such transaction for this merchant |
| `ALREADY_USED` | Already claimed once — **replay rejected** |
| `ATTACK` | Sender/receiver mismatch — suspicious message |
| `EXPIRED` | `max_age_minutes` window exceeded |
| `AMOUNT_MISMATCH` | Amount did not match (expected amount in message) |

**⚠️ Security rules**

- Never ship the license key inside browser or mobile-client code — keep it
  **server-side only**.
- Call `verify` from your **payment callback / backend**, never from the end
  user's browser.
- Always send `amount` so a wrong-amount payment is rejected automatically.

**Examples**

cURL:

```bash
curl -X POST https://paypilot-5p9t.onrender.com/api/v1/verify \
  -H "Authorization: Bearer $PAYPILOT_LICENSE" \
  -H "Content-Type: application/json" \
  -d '{"trx_id":"DI739OTDF3","amount":100.00,"order_id":"ORDER-1024"}'
```

Node.js:

```js
const res = await fetch('https://paypilot-5p9t.onrender.com/api/v1/verify', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.PAYPILOT_LICENSE}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ trx_id, amount: 100.0, order_id }),
});
const { success, code, data } = await res.json();
if (success && code === 'VERIFIED') {
  // one-time claim succeeded — deliver the order
}
```

PHP:

```php
$ch = curl_init('https://paypilot-5p9t.onrender.com/api/v1/verify');
curl_setopt_array($ch, [
  CURLOPT_POST => true,
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_HTTPHEADER => [
    'Authorization: Bearer ' . getenv('PAYPILOT_LICENSE'),
    'Content-Type: application/json',
  ],
  CURLOPT_POSTFIELDS => json_encode([
    'trx_id' => $trxId, 'amount' => 100.00, 'order_id' => $orderId,
  ]),
]);
$result = json_decode(curl_exec($ch), true);
```

---

## 2) Read transactions (read-only)

### `GET /api/v1/transactions/{trxId}`

Current state of a transaction without claiming it (fee, status, `order_id`).

```json
{
  "success": true,
  "data": {
    "trx_id": "DI739OTDF3",
    "amount": 100.0,
    "sender": "01722858922",
    "fee": 0.0,
    "status": "available",
    "received_at": "2026-09-08T16:32:00.000Z",
    "used_at": null,
    "order_id": null
  }
}
```

### `GET /api/v1/transactions?status=&limit=&offset=`

List your transactions (`status`: `available|used|expired|attack`, `limit` ≤ 100, default 50).

```json
{ "success": true, "data": { "total": 128, "items": [ ... ] } }
```

### `POST /api/v1/sms/receive` *(advanced)*

Inject a transaction from your own parser (normally unnecessary — the Android
app posts raw SMS itself). Fields: `trx_id`, `amount`, `sender?`, `fee?`,
`balance?`, `received_at` (ISO 8601), `device_id?`, `raw_message?`.

---

## 3) Dashboard login & profile

### `POST /api/v1/auth/login`

Merchant/owner web login. Response: `data.token` (JWT), `data.role`
(`owner|user`), `data.user`.

```json
{ "username": "shopuser", "password": "••••••" }
```

### `GET /api/v1/auth/me`

Current session profile: `id`, `username`, `name`, `role`, `avatar_url`.

### `POST /api/v1/auth/change-password`

`{ "current_password": "...", "new_password": "..." }` (min 6 characters).

### `PATCH /api/v1/auth/profile`

Merchant self-service profile update:

```json
{
  "avatar_url": "https://example.com/logo.png",
  "business_name": "My Shop",
  "name": "Shop Account Name"
}
```

- `avatar_url` = the **business logo shown inside the Android app** (the app
  downloads and caches the image).
- `business_name` = the title of the merchant card inside the app.
- Send `""` to clear a field.

---

## 4) Device (Android app) endpoints

Used by the official app; open to custom integrations too.

### `POST /api/v1/device/license/verify`

App activation + profile sync. Response includes `valid`, `user_name`,
`username`, `email`, `business_name`, `logo_url` — these drive the app's
merchant card and logo.

```json
{
  "license_key": "<LICENSE>",
  "device_id": "<stable-uuid>",
  "device_name": "Samsung SM-A156E",
  "app_version": "1.2.2"
}
```

### `POST /api/v1/device/ping` / `GET /api/v1/device/ping?k=&d=`

Heartbeat (every 5 seconds while monitoring). Compact response:
`{ "ok": 1, "s": 1 }`.

### `POST /api/v1/device/sms`

Upload an incoming SMS — the server parses bKash messages itself, matches the
receiver SIM against the merchant's wallets, stores the transaction and assigns
a category (`pending | claimed | otp | unwanted | attack`). Fields:
`device_id`, `sms_id`, `address?`, `body`, `received_at`, `sim_slot?`,
`receiver_number?`.

---

## 5) Merchant portal (own data)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/api/v1/portal/stats` | Transaction / available / used / SMS / device counters |
| GET | `/api/v1/portal/notifications?category=&app_id=` | Own SMS feed |
| GET | `/api/v1/portal/devices` | Own devices (online = heartbeat within 2 minutes) |
| GET | `/api/v1/portal/apps` | List wallets (receiver SIMs) |
| POST | `/api/v1/portal/apps` | Add wallet — `{ name, provider: "bkash"|"nagad", wallet_number, allowed_senders? }` |
| PATCH | `/api/v1/portal/apps/{id}` | Edit wallet / `status: active\|disabled` |
| DELETE | `/api/v1/portal/apps/{id}` | Delete wallet |

> `wallet_number` must be the number of the SIM that receives the payment SMS.
> `allowed_senders` accepts a comma-separated payer whitelist (empty = any
> payer); payments from unknown senders are categorised as `attack`.

---

## 6) Owner endpoints

These power the **official owner web dashboard** and require an owner token —
not intended for third-party use. In short:

- `GET /api/v1/admin/stats` · `GET|POST|PATCH|DELETE /api/v1/admin/users…`
- `GET /api/v1/admin/users/{id}/license` · `POST /api/v1/admin/users/{id}/token` (regenerate license)
- `POST /api/v1/admin/users/{id}/apps` · `PATCH|DELETE /api/v1/admin/apps/{id}` (merchant wallet management)
- `GET /api/v1/admin/transactions` · `GET /api/v1/admin/notifications` · `GET /api/v1/admin/devices` · `GET /api/v1/admin/audit`

---

## Getting a license key

1. The PayPilot owner creates your merchant account — you receive a
   **username + password** (web dashboard) and a **license key** (app + API).
2. Activate the Android app with the license key and/or configure it on your
   website's server.
3. If the key ever leaks, contact the owner immediately — regenerating it
   revokes the old key instantly.

**Contact:** [github.com/A2MBD3](https://github.com/A2MBD3) · [a2mbd3.pages.dev](https://a2mbd3.pages.dev)
