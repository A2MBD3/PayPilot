# PayPilot পেমেন্ট ভেরিফিকেশন API

> **শুধু সার্ভার-টু-সার্ভার।** কাস্টমার TrxID দিলে আপনার **প্রাইভেট ব্যাকএন্ড** থেকে এই API কল করুন।  
> API লাইসেন্স ব্রাউজার, পাবলিক অ্যাপ বা পাবলিক রিপোতে রাখবেন না।  
> English: [`docs/API.md`](API.md)

| | |
|--|--|
| **Base URL** | `https://paypilot-5p9t.onrender.com` |
| **Endpoint** | `POST /api/v1/verify` |
| **Auth** | `Authorization: Bearer <API_LICENSE>` |
| **Content-Type** | `application/json` |
| **রেট লিমিট** | প্রায় ২৪০ রিকোয়েস্ট / মিনিট (প্রতি IP) |

**API license** মার্চেন্ট ড্যাশবোর্ড → **Authorization → API license** থেকে পাবেন।  
নতুন করে জেনারেট করলে পুরনো কী তাৎক্ষণিক বাতিল।

---

## ফ্লো

```
কাস্টমার bKash (পার্সোনাল) দিয়ে পেমেন্ট করে
        ↓
সাইটে TrxID (ঐচ্ছিক amount) দেয়
        ↓
আপনার প্রাইভেট সার্ভার  ──POST /api/v1/verify──►  PayPilot
        ↓
PayPilot পেমেন্ট একবারের জন্য claim করে
        ↓
আপনার সার্ভার অর্ডার ডেলিভার করে
```

- সফল ভেরিফাই = **একবারের claim** (`used` / `claimed`)।
- একই `trx_id` আবার যাচাই করা যাবে না (`ALREADY_USED`)।
- `amount` দিলে SMS-এর অঙ্কের সাথে মিলতে হবে।
- `max_age_minutes` ডিফল্ট **৩৬০** (৬ ঘণ্টা); পুরনো হলে `TOO_OLD` — সাথে `review_required: true` ও পেমেন্টের `data` ফেরত আসে (ম্যানুয়াল রিভিউয়ের জন্য; অটো-এক্সপায়ার হয় না)।

---

## রিকোয়েস্ট

```http
POST /api/v1/verify HTTP/1.1
Host: paypilot-5p9t.onrender.com
Authorization: Bearer <API_LICENSE>
Content-Type: application/json

{
  "trx_id": "DI739OTDF3",
  "amount": 100.00,
  "order_id": "ORDER-1024",
  "max_age_minutes": 60
}
```

| ফিল্ড | টাইপ | আবশ্যক | বর্ণনা |
|-------|------|--------|--------|
| `trx_id` | string (৫–৩০) | **হ্যাঁ** | কাস্টমারের bKash SMS-এর TrxID |
| `amount` | number > 0 | না | দিলে সার্ভারে মিলিয়ে নেয় (±০.০০১) |
| `order_id` | string ≤ ১০০ | না | আপনার অর্ডার রেফারেন্স |
| `max_age_minutes` | int ১–১৪৪০ | না | ডিফল্ট `৩৬০` (৬ ঘণ্টা) |

---

## সফল রেসপন্স (`200`)

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
    "app": "bKash",
    "received_at": "2026-09-08T16:32:00.000Z",
    "used_at": "2026-09-08T16:40:12.480Z",
    "order_id": "ORDER-1024"
  }
}
```

`app` শুধু তথ্য — ক্লায়েন্ট থেকে পাঠাবেন না।  
অর্ডার তখনই সম্পন্ন করুন যখন `success === true` এবং `code === "VERIFIED"`।

---

## এরর

```json
{ "success": false, "code": "NOT_FOUND", "message": "Transaction not found" }
```

| HTTP | `code` | অর্থ |
|------|--------|------|
| 400 | `VALIDATION_ERROR` | ভুল/অসম্পূর্ণ ইনপুট |
| 400 | `NOT_FOUND` | এই মার্চেন্টের অধীনে TrxID নেই |
| 400 | `AMOUNT_MISMATCH` | অঙ্ক মিলেনি |
| 400 | `ALREADY_USED` | আগেই claim হয়েছে |
| 400 | `TOO_OLD` | সময়সীমা পেরিয়েছে — সাথে **`review_required: true`** ও পেমেন্টের `data` ফেরত আসে (নিচে দেখুন) |
| 400 | `ATTACK` | সন্দেহজনক — ডেলিভার করবেন না |
| 401 | `UNAUTHORIZED` | API license ভুল/নেই |
| 401 | `TOKEN_REVOKED` | লাইসেন্স regenerate হয়েছে |
| 403 | `DISABLED` | অ্যাকাউন্ট বন্ধ |
| 429 | `RATE_LIMITED` | অতিরিক্ত রিকোয়েস্ট |

---

## `TOO_OLD` — ম্যানুয়াল রিভিউ পেলোড

সময়সীমার বেশি পুরনো পেমেন্ট **অটো-এক্সপায়ার হয় না**। API `400 TOO_OLD` দেয় এবং সাথে অতিরিক্ত ফিল্ড দেয়, যেন আপনার সার্ভার এটিকে ম্যানুয়াল রিভিউয়ের কিউতে রাখতে পারে:

```json
{
  "success": false,
  "code": "TOO_OLD",
  "message": "Payment is older than the allowed window",
  "review_required": true,
  "data": {
    "trx_id": "DI739OTDF3",
    "amount": 100.0,
    "sender": "01722858922",
    "status": "available",
    "app": "bKash",
    "received_at": "2026-09-08T08:32:00.000Z",
    "order_id": null
  }
}
```

পেমেন্ট `available` থাকে — রিভিউতে অনুমোদন হলে বড় `max_age_minutes` (সর্বোচ্চ `১৪৪০`) দিয়ে আবার যাচাই করা যায়।

---

## সিকিউরিটি চেকলিস্ট

1. শুধু **প্রাইভেট সার্ভার** থেকে কল করুন।  
2. `API_LICENSE` এনভায়রনমেন্ট/সিক্রেটে রাখুন।  
3. ব্রাউজার বা পাবলিক ক্লায়েন্টে লাইসেন্স দেবেন না।  
4. সম্ভব হলে `trx_id` + `amount` একসাথে পাঠান।  
5. `ALREADY_USED` এ আবার প্রোডাক্ট দেবেন না (নিজের অর্ডার লগ চেক করুন)।  
6. `ATTACK` / `TOO_OLD` = পেমেন্ট ব্যর্থ।

---

## কোড ডেমো

`PAYPILOT_API_LICENSE` এনভ ভ্যারিয়েবলে API license সেট করুন।

### cURL

```bash
curl -sS -X POST 'https://paypilot-5p9t.onrender.com/api/v1/verify' \
  -H "Authorization: Bearer ${PAYPILOT_API_LICENSE}" \
  -H 'Content-Type: application/json' \
  -d '{
    "trx_id": "DI739OTDF3",
    "amount": 100.00,
    "order_id": "ORDER-1024",
    "max_age_minutes": 60
  }'
```

### Node.js

```js
async function verifyPayment({ trxId, amount, orderId }) {
  const res = await fetch('https://paypilot-5p9t.onrender.com/api/v1/verify', {
    method: 'POST',
    headers: {
      Authorization: `Bearer ${process.env.PAYPILOT_API_LICENSE}`,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      trx_id: trxId,
      amount,
      order_id: orderId,
      max_age_minutes: 60,
    }),
  });
  const body = await res.json();
  if (!body.success || body.code !== 'VERIFIED') {
    throw new Error(body.message || body.code || 'verify failed');
  }
  return body.data;
}
```

### PHP

```php
<?php
function paypilot_verify(string $trxId, ?float $amount = null, ?string $orderId = null): array {
    $payload = ['trx_id' => $trxId, 'max_age_minutes' => 60];
    if ($amount !== null) $payload['amount'] = $amount;
    if ($orderId !== null) $payload['order_id'] = $orderId;

    $ch = curl_init('https://paypilot-5p9t.onrender.com/api/v1/verify');
    curl_setopt_array($ch, [
        CURLOPT_POST => true,
        CURLOPT_HTTPHEADER => [
            'Authorization: Bearer ' . getenv('PAYPILOT_API_LICENSE'),
            'Content-Type: application/json',
        ],
        CURLOPT_POSTFIELDS => json_encode($payload),
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_TIMEOUT => 20,
    ]);
    $raw = curl_exec($ch);
    if ($raw === false) throw new RuntimeException(curl_error($ch));
    $body = json_decode($raw, true);
    if (!($body['success'] ?? false) || ($body['code'] ?? '') !== 'VERIFIED') {
        throw new RuntimeException($body['message'] ?? 'verify failed');
    }
    return $body['data'];
}
```

### Python

```python
import os
import requests

def verify_payment(trx_id: str, amount: float | None = None, order_id: str | None = None) -> dict:
    payload = {"trx_id": trx_id, "max_age_minutes": 60}
    if amount is not None:
        payload["amount"] = amount
    if order_id is not None:
        payload["order_id"] = order_id
    r = requests.post(
        "https://paypilot-5p9t.onrender.com/api/v1/verify",
        headers={
            "Authorization": f"Bearer {os.environ['PAYPILOT_API_LICENSE']}",
            "Content-Type": "application/json",
        },
        json=payload,
        timeout=20,
    )
    body = r.json()
    if not body.get("success") or body.get("code") != "VERIFIED":
        raise RuntimeError(body.get("message") or body.get("code"))
    return body["data"]
```

### Go

```go
// Same pattern as English docs/API.md — POST JSON with Bearer API license
// Endpoint: https://paypilot-5p9t.onrender.com/api/v1/verify
```

সম্পূর্ণ Go/Java উদাহরণ: [docs/API.md](API.md)।

---

## API license কোথায় পাবেন

1. মার্চেন্ট ড্যাশবোর্ডে লগইন  
2. **Authorization** ট্যাব  
3. **API license** কপি করুন (মোবাইল App license নয়)  
4. শুধু সার্ভার এনভে রাখুন: `PAYPILOT_API_LICENSE`

যোগাযোগ: মূল [README.bn.md](../README.bn.md)।


---

## ডিভাইস স্ট্যাটাস চেক (ঐচ্ছিক)

### `POST /api/v1/status/devices` · `GET /api/v1/status/devices`

অর্ডার নেওয়ার আগে মার্চেন্টের Android ডিভাইস অনলাইন কিনা (**API license** দিয়ে প্রাইভেট সার্ভার থেকে) দেখুন।

**Auth:** `Authorization: Bearer <API_LICENSE>`

ঐচ্ছিক `wallets` / `wallet` — নির্দিষ্ট ওয়ালেট (id / provider / নম্বর)। না দিলে সব ডিভাইস।

```bash
curl -sS -X POST 'https://paypilot-5p9t.onrender.com/api/v1/status/devices' \
  -H "Authorization: Bearer $PAYPILOT_API_LICENSE" \
  -H 'Content-Type: application/json' \
  -d '{"wallets":["bkash"]}'
```

`online` = শেষ পিং **২ মিনিটের** মধ্যে। ৬ ঘণ্টার বেশি পুরনো পেমেন্টে `TOO_OLD` (ম্যানুয়াল রিভিউ; অটো-এক্সপায়ার নয়) (`verify`-এ `max_age_minutes` ডিফল্ট **৩৬০**)।

**সফল রেসপন্সের গঠন**

```json
{
  "success": true,
  "data": {
    "business_name": "আমার দোকান",
    "merchant_name": "আবদুল্লাহ",
    "any_online": true,
    "can_accept_payments": true,
    "devices": [
      {
        "device_id": "...",
        "device_name": "Pixel",
        "app_version": "1.2.4",
        "last_seen_at": "2026-09-09T04:00:00.000Z",
        "seconds_ago": 4,
        "online": true
      }
    ],
    "wallets": [
      {
        "id": "...",
        "name": "bKash",
        "provider": "bkash",
        "wallet_number": "01...",
        "status": "active"
      }
    ],
    "wallets_total_active": 1,
    "checked_at": "2026-09-09T04:00:05.000Z"
  }
}
```
