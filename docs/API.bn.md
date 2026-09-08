# PayPilot API — v1

> **PayPilot REST API** দিয়ে আপনি শুধু ড্যাশবোর্ডের মেসেজ দেখতে পারবেন না —
> আপনার **ক্লায়েন্টের ওয়েবসাইট থেকেও ট্রানজেকশন আইডি দিয়ে লেনদেন যাচাই** করা যায়।
> English version of this document: [`docs/API.md`](API.md) · বাংলা: [`docs/API.bn.md`](API.bn.md)

- **Base URL:** `https://paypilot-5p9t.onrender.com`
- **Format:** JSON request/response, UTF-8
- **Auth:** `Authorization: Bearer <token>` (license key / web token / owner token)
- **Rate limit:** general API **300 req/min** · device routes **600 req/min** · login **30 req/15 min**

---

## রেসপন্স এনভেলপ

সফল কল:

```json
{ "success": true, "message": "...", "data": { } }
```

ব্যর্থ কল:

```json
{ "success": false, "code": "NOT_FOUND", "message": "Transaction not found" }
```

| HTTP | Code | অর্থ |
|------|------|------|
| 400 | `VALIDATION_ERROR` | ইনপুট ফরম্যাট ভুল |
| 401 | `UNAUTHORIZED` | টোকেন নেই / অবৈধ / মেয়াদোত্তীর্ণ |
| 401 | `TOKEN_REVOKED` | লাইসেন্স রিজেনারেট করা হয়েছে — পুরোনো টোকেন বাতিল |
| 403 | `DISABLED` | মার্চেন্ট অ্যাকাউন্ট সাসপেন্ডেড |
| 401 | `INVALID_CREDENTIALS` | ইউজারনেম/পাসওয়ার্ড ভুল |
| 404 | `NOT_FOUND` | রিসোর্স পাওয়া যায়নি |
| 429 | `RATE_LIMITED` | অনেক বেশি রিকোয়েস্ট |
| 500 | `INTERNAL_ERROR` | সার্ভার ত্রুটি |

---

## ১) ট্রানজেকশন যাচাই (ওয়েবসাইট ইন্টিগ্রেশনের মূল API)

### `POST /api/v1/verify`

একটি **TrxID দিয়ে পেমেন্ট যাচাই** করে। এটি **এককালীন দাবি (one-time claim)** —
সফল যাচাইয়ের পর লেনদেনটি `used` হয়ে যায় এবং **আর কখনো দ্বিতীয়বার যাচাই করা যায় না**
(রিপ্লে/ডুপ্লিকেট অর্ডার ঠেকাতে)। শুধু পড়ার জন্য `GET /api/v1/transactions/{trxId}` ব্যবহার করুন।

**Auth:** মার্চেন্টের লাইসেন্স কী (Bearer)

**রিকোয়েস্ট**

```json
{
  "trx_id": "DI739OTDF3",
  "amount": 100.00,
  "order_id": "ORDER-1024",
  "max_age_minutes": 60
}
```

| ফিল্ড | ধরন | বাধ্যতামূলক | বর্ণনা |
|-------|-----|-------------|--------|
| `trx_id` | string (5–30) | ✅ | bKash SMS-এর TrxID |
| `amount` | number > 0 | ❌ | দিলে সার্ভার টাকার অঙ্ক মিলিয়ে দেখবে (±0.001) |
| `order_id` | string ≤ 100 | ❌ | আপনার অর্ডার আইডি — লেনদেনের সাথে সংরক্ষিত হয় |
| `max_age_minutes` | int 1–1440 | ❌ | ডিফল্ট ৬০ মিনিট; এর পুরোনো লেনদেন `EXPIRED` |

**সফল রেসপন্স `200`**

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

**ব্যর্থতার কোড (HTTP 400):**

| Code | অর্থ |
|------|------|
| `NOT_FOUND` | এই TrxID-এর লেনদেন নেই (এই মার্চেন্টের জন্য) |
| `ALREADY_USED` | আগেই যাচাই হয়েছে — **পুনরায় ব্যবহার নিষেধ** |
| `ATTACK` | সেন্ডার/রিসিভার মিলেনি — সন্দেহজনক বার্তা |
| `EXPIRED` | `max_age_minutes` সময়সীমা পেরিয়েছে |
| `AMOUNT_MISMATCH` | অঙ্ক মেলেনি (মেসেজে প্রত্যাশিত অঙ্ক দেখানো হয়) |

**⚠️ নিরাপত্তা নিয়ম**

- লাইসেন্স কী **কখনো ব্রাউজার/অ্যাপ-ক্লায়েন্টে রাখবেন না** — শুধু আপনার সার্ভার-সাইড কোডে রাখুন।
- যাচাই **আপনার সাইটের পেমেন্ট-কলব্যাক/সার্ভার থেকে** করুন, ইউজারের ব্রাউজার থেকে নয়।
- `amount` সবসময় পাঠান — ভুল অঙ্কের পেমেন্ট স্বয়ংক্রিয়ভাবে বাতিল হবে।

**উদাহরণ**

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
  // order_id ডেলিভারি চালু করুন — একবারই সফল হবে
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

## ২) লেনদেন পড়া (read-only)

### `GET /api/v1/transactions/{trxId}`

এককালীন দাবি ছাড়াই লেনদেনের বর্তমান অবস্থা দেখায় (ফি, স্ট্যাটাস, `order_id`)।

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

নিজের সব লেনদেনের তালিকা (`status`: `available|used|expired|attack`, `limit` ≤ 100, ডিফল্ট ৫০)।

```json
{ "success": true, "data": { "total": 128, "items": [ ... ] } }
```

### `POST /api/v1/sms/receive` *(অ্যাডভান্সড)*

নিজস্ব পার্সার থেকে সরাসরি লেনদেন ইনজেক্ট করতে (সাধারণত দরকার হয় না — অ্যাপ নিজেই পাঠায়)।
ফিল্ড: `trx_id`, `amount`, `sender?`, `fee?`, `balance?`, `received_at` (ISO 8601), `device_id?`, `raw_message?`।

---

## ৩) ড্যাশবোর্ড লগইন ও প্রোফাইল

### `POST /api/v1/auth/login`

মার্চেন্ট/মালিকের ওয়েব লগইন। রেসপন্সে `data.token` (JWT), `data.role` (`owner|user`), `data.user`।

```json
{ "username": "shopuser", "password": "••••••" }
```

### `GET /api/v1/auth/me`

বর্তমান সেশনের প্রোফাইল: `id`, `username`, `name`, `role`, `avatar_url`।

### `POST /api/v1/auth/change-password`

`{ "current_password": "...", "new_password": "..." }` (নতুন পাসওয়ার্ড ন্যূনতম ৬ অক্ষর)।

### `PATCH /api/v1/auth/profile`

মার্চেন্ট নিজের প্রোফাইল আপডেট:

```json
{
  "avatar_url": "https://example.com/logo.png",
  "business_name": "My Shop",
  "name": "Shop Account Name"
}
```

- `avatar_url` = অ্যাপের ড্যাশবোর্ডে দেখা **ব্যবসার লোগো** — অ্যাপ ছবিটি ডাউনলোড করে ক্যাশে রাখে।
- `business_name` = অ্যাপের মার্চেন্ট কার্ডের শিরোনাম।
- খালি স্ট্রিং `""` পাঠালে সেট মুছে যায়।

---

## ৪) অ্যাপ (ডিভাইস) এন্ডপয়েন্ট

এগুলো PayPilot Android অ্যাপ ব্যবহার করে; নিজস্ব ইন্টিগ্রেশনের জন্যও খোলা।

### `POST /api/v1/device/license/verify`

অ্যাপ অ্যাক্টিভেশন + প্রোফাইল সিঙ্ক। রেসপন্সে `valid`, `user_name`, `username`, `email`,
`business_name`, `logo_url` — অ্যাপ এগুলো থেকেই ড্যাশবোর্ড কার্ড ও লোগো দেখায়।

```json
{
  "license_key": "<LICENSE>",
  "device_id": "<stable-uuid>",
  "device_name": "Samsung SM-A156E",
  "app_version": "1.2.2"
}
```

### `POST /api/v1/device/ping` / `GET /api/v1/device/ping?k=&d=`

হার্টবিট (৫ সেকেন্ড পরপর)। কমপ্যাক্ট রেসপন্স: `{ "ok": 1, "s": 1 }`।

### `POST /api/v1/device/sms`

ইনকামিং SMS আপলোড — সার্ভার নিজেই bKash বার্তা পার্স করে, ওয়ালেট-নম্বর (রিসিভার সিম) মিলিয়ে
লেনদেন তৈরি করে এবং ক্যাটাগরি দেয় (`pending | claimed | otp | unwanted | attack`)।
ফিল্ড: `device_id`, `sms_id`, `address?`, `body`, `received_at`, `sim_slot?`, `receiver_number?`।

---

## ৫) মার্চেন্ট পোর্টাল (নিজের ডেটা)

| Method | Path | বর্ণনা |
|--------|------|--------|
| GET | `/api/v1/portal/stats` | লেনদেন/available/used/SMS/ডিভাইস কাউন্টার |
| GET | `/api/v1/portal/notifications?category=&app_id=` | নিজের SMS ফিড |
| GET | `/api/v1/portal/devices` | নিজের অ্যাক্টিভ ডিভাইস (online = ২ মিনিটে হার্টবিট) |
| GET | `/api/v1/portal/apps` | ওয়ালেট (receiver SIM) তালিকা |
| POST | `/api/v1/portal/apps` | ওয়ালেট যোগ — `{ name, provider: "bkash"|"nagad", wallet_number, allowed_senders? }` |
| PATCH | `/api/v1/portal/apps/{id}` | ওয়ালেট এডিট / `status: active\|disabled` |
| DELETE | `/api/v1/portal/apps/{id}` | ওয়ালেট ডিলিট |

> `wallet_number` অবশ্যই সেই সিমের নম্বর হতে হবে যেখানে পেমেন্ট SMS আসে।
> `allowed_senders`-এ কমা দিয়ে পেয়ার-নম্বর দিলে শুধু সেই পেয়ারদের পেমেন্ট গৃহীত হবে
> (খালি = যেকোনো পেয়ার)। হোয়াইটলিস্টের বাইরের পেমেন্ট `attack` ক্যাটাগরিতে যায়।

---

## ৬) মালিক (Owner) এন্ডপয়েন্ট

এগুলো **অফিসিয়াল ওয়েব ড্যাশবোর্ডের জন্য** — owner টোকেন লাগে। থার্ড-পার্টি ব্যবহারের
জন্য নয়; সম্পূর্ণ তালিকা প্রয়োজনে সাপোর্টে চাইুন। সংক্ষেপে:

- `GET /api/v1/admin/stats` · `GET|POST|PATCH|DELETE /api/v1/admin/users…`
- `GET /api/v1/admin/users/{id}/license` · `POST /api/v1/admin/users/{id}/token` (লাইসেন্স রিজেনারেট)
- `POST /api/v1/admin/users/{id}/apps` · `PATCH|DELETE /api/v1/admin/apps/{id}` (মার্চেন্টের ওয়ালেট ম্যানেজমেন্ট)
- `GET /api/v1/admin/transactions` · `GET /api/v1/admin/notifications` · `GET /api/v1/admin/devices` · `GET /api/v1/admin/audit`

---

## লাইসেন্স কী কীভাবে পাবেন

1. PayPilot মালিক আপনার জন্য মার্চেন্ট অ্যাকাউন্ট তৈরি করবে — পাবেন **username + password** (ওয়েব লগইন) এবং **লাইসেন্স কী** (অ্যাপ + API)।
2. লাইসেন্স কী দিয়ে Android অ্যাপ অ্যাক্টিভেট করুন এবং/অথবা আপনার ওয়েবসাইটের সার্ভারে কনফিগ করুন।
3. কী লিক হলে সাথে সাথে মালিককে জানান — রিজেনারেট করলে পুরোনো কী সাথে সাথে বাতিল হয়ে যায়।

**যোগাযোগ:** [github.com/A2MBD3](https://github.com/A2MBD3) · [a2mbd3.pages.dev](https://a2mbd3.pages.dev)
