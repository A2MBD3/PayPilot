# PayPilot Payment Verification API

> **Server-to-server only.** Call this API from your **private backend** after a customer submits a bKash TrxID.  
> Never put your API license in browser JavaScript, mobile apps you distribute, or public repos.  
> বাংলা: [`docs/API.bn.md`](API.bn.md)

| | |
|--|--|
| **Base URL** | `https://paypilot-5p9t.onrender.com` |
| **Endpoint** | `POST /api/v1/verify` |
| **Auth** | `Authorization: Bearer <API_LICENSE>` |
| **Content-Type** | `application/json` |
| **Rate limit** | ≈240 requests / minute per IP |

Your **API license** is issued in the merchant dashboard under **Authorization → API license**.  
Regenerating it immediately revokes the previous key.

---

## Flow

```
Customer pays via bKash (personal)
        ↓
Customer enters TrxID (and optionally amount) on your site
        ↓
Your private server  ──POST /api/v1/verify──►  PayPilot
        ↓
PayPilot marks the payment claimed (one-time)
        ↓
Your server fulfills the order
```

- Verification is a **one-time claim**. A successful call sets status to `used` / `claimed`.
- The same `trx_id` cannot be verified again (`ALREADY_USED`).
- Optional `amount` must match the SMS amount when provided.
- Optional `max_age_minutes` (default **360** = 6 hours) rejects older payments (`TOO_OLD` — returned with `review_required: true` for manual review, never auto-expired).

---

## Request

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

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `trx_id` | string (5–30) | **Yes** | Transaction ID from the customer’s bKash SMS |
| `amount` | number > 0 | No | If set, must match stored amount (±0.001) |
| `order_id` | string ≤ 100 | No | Your order reference; stored on success |
| `max_age_minutes` | int 1–1440 | No | Max age of the payment; default `360` (6 hours) |

---

## Success response (`200`)

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

| Field | Meaning |
|-------|---------|
| `trx_id` | Confirmed transaction id |
| `amount` | Amount received |
| `sender` | Payer mobile (when parsed) |
| `status` | `used` after claim |
| `category` | `claimed` |
| `app` | Matched wallet label (if any) — informational only; **do not send this field** |
| `order_id` | Echo of your order id |

---

## Error responses

Envelope:

```json
{ "success": false, "code": "NOT_FOUND", "message": "Transaction not found" }
```

| HTTP | `code` | When |
|------|--------|------|
| 400 | `VALIDATION_ERROR` | Missing/invalid body |
| 400 | `NOT_FOUND` | Unknown `trx_id` for this merchant |
| 400 | `AMOUNT_MISMATCH` | `amount` does not match |
| 400 | `ALREADY_USED` | Already verified / claimed |
| 400 | `TOO_OLD` | Older than `max_age_minutes` — returned **with `review_required: true`** and the payment `data` for manual review (see below) |
| 400 | `ATTACK` | Flagged payment (do not fulfill) |
| 401 | `UNAUTHORIZED` | Missing/invalid API license |
| 401 | `TOKEN_REVOKED` | License was regenerated |
| 403 | `DISABLED` | Merchant account suspended |
| 429 | `RATE_LIMITED` | Too many requests |

Only fulfill the order when `success === true` and `code === "VERIFIED"`.

### `TOO_OLD` — manual review payload

Payments older than `max_age_minutes` are **not auto-expired**. The API answers `400 TOO_OLD` with extra fields so your server can queue it for manual review:

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

The payment stays `available` — if your review approves it, verify again with a higher `max_age_minutes` (up to `1440`).

---

## Security checklist

1. Call **only from your backend** (Node, PHP, Laravel, etc.).
2. Store `API_LICENSE` in environment variables / secrets manager.
3. Never expose the license to the browser or a public client.
4. Prefer sending both `trx_id` and `amount`.
5. On `ALREADY_USED`, do not deliver the product again without your own order audit.
6. Treat `ATTACK` / `TOO_OLD` as payment failure.

---

## Code demos

Replace `API_LICENSE`, base URL, and order fields with your values.

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

### Node.js (fetch)

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
  return body.data; // fulfill order
}

// Example
// await verifyPayment({ trxId: 'DI739OTDF3', amount: 100, orderId: 'ORDER-1024' });
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
    if ($raw === false) {
        throw new RuntimeException(curl_error($ch));
    }
    $code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    $body = json_decode($raw, true);
    if (!($body['success'] ?? false) || ($body['code'] ?? '') !== 'VERIFIED') {
        throw new RuntimeException($body['message'] ?? ('HTTP ' . $code));
    }
    return $body['data'];
}

// $data = paypilot_verify('DI739OTDF3', 100.00, 'ORDER-1024');
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
        raise RuntimeError(body.get("message") or body.get("code") or r.status_code)
    return body["data"]

# data = verify_payment("DI739OTDF3", 100.0, "ORDER-1024")
```

### Go

```go
package main

import (
        "bytes"
        "encoding/json"
        "fmt"
        "io"
        "net/http"
        "os"
        "time"
)

func verifyPayment(trxID string, amount *float64, orderID string) (map[string]any, error) {
        payload := map[string]any{
                "trx_id":          trxID,
                "max_age_minutes": 60,
        }
        if amount != nil {
                payload["amount"] = *amount
        }
        if orderID != "" {
                payload["order_id"] = orderID
        }
        b, _ := json.Marshal(payload)
        req, err := http.NewRequest(http.MethodPost, "https://paypilot-5p9t.onrender.com/api/v1/verify", bytes.NewReader(b))
        if err != nil {
                return nil, err
        }
        req.Header.Set("Authorization", "Bearer "+os.Getenv("PAYPILOT_API_LICENSE"))
        req.Header.Set("Content-Type", "application/json")

        client := &http.Client{Timeout: 20 * time.Second}
        res, err := client.Do(req)
        if err != nil {
                return nil, err
        }
        defer res.Body.Close()
        raw, _ := io.ReadAll(res.Body)
        var body map[string]any
        if err := json.Unmarshal(raw, &body); err != nil {
                return nil, err
        }
        if body["success"] != true || body["code"] != "VERIFIED" {
                return nil, fmt.Errorf("%v", body["message"])
        }
        data, _ := body["data"].(map[string]any)
        return data, nil
}
```

### Java (HttpClient)

```java
import java.net.URI;
import java.net.http.*;
import java.time.Duration;

public class PayPilotVerify {
  public static String verify(String trxId, double amount, String orderId) throws Exception {
    String json = """
      {"trx_id":"%s","amount":%s,"order_id":"%s","max_age_minutes":60}
      """.formatted(trxId, amount, orderId);

    HttpRequest req = HttpRequest.newBuilder()
        .uri(URI.create("https://paypilot-5p9t.onrender.com/api/v1/verify"))
        .timeout(Duration.ofSeconds(20))
        .header("Authorization", "Bearer " + System.getenv("PAYPILOT_API_LICENSE"))
        .header("Content-Type", "application/json")
        .POST(HttpRequest.BodyPublishers.ofString(json))
        .build();

    HttpResponse<String> res = HttpClient.newHttpClient()
        .send(req, HttpResponse.BodyHandlers.ofString());

    if (res.statusCode() != 200 || !res.body().contains("\"VERIFIED\"")) {
      throw new IllegalStateException(res.body());
    }
    return res.body();
  }
}
```

---

## Getting your API license

1. Sign in to the PayPilot merchant dashboard.  
2. Open **Authorization**.  
3. Copy **API license** (not the mobile App license).  
4. Set it as `PAYPILOT_API_LICENSE` on your server only.

Support / license issues: use the contact channels on the main [README](../README.md).


---

## Device & wallet status (optional)

### `POST /api/v1/status/devices` · `GET /api/v1/status/devices`

Call from your **private server** with the **API Token** to learn:

- which **wallets** the merchant has active
- which **device** last received a payment for each wallet
- how long ago that device last pinged the server (`device_seconds_ago`)

PayPilot does **not** decide whether you should accept an order. Use `device_seconds_ago` / `device_last_seen_at` with your own threshold.

**Auth:** `Authorization: Bearer <API_TOKEN>`  
(API Token only — the app **License** is rejected.)

**Body (POST) or query (GET)**

| Field | Description |
|-------|-------------|
| `wallets` | Optional array of wallet id / provider / number |
| `wallet` | Optional single value (POST) or `?wallet=bkash` (GET) |

If omitted, **all active wallets** and **all known devices** are returned. If set, only matching wallets (and devices that appear on those wallets) are emphasized in the wallet list.

### How wallet → device is chosen

For each wallet, the server looks at payment history and picks the **device that most recently received money** on that wallet (`device_id` from the latest matching transaction). That device’s last heartbeat is then attached to the wallet.

- No payment yet on a wallet → `device_id` / `device_seconds_ago` are `null`
- Device removed from records but still referenced → `device_id` may be set with `device_seconds_ago: null`

### Example

```bash
curl -sS -X POST 'https://paypilot-5p9t.onrender.com/api/v1/status/devices' \
  -H "Authorization: Bearer $PAYPILOT_API_TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{"wallets":["bkash"]}'
```

### Success response

```json
{
  "success": true,
  "data": {
    "business_name": "My Shop",
    "merchant_name": "Abdullah",
    "any_online": true,
    "devices": [
      {
        "device_id": "abc-123",
        "device_name": "Pixel",
        "app_version": "1.2.4",
        "last_seen_at": "2026-09-10T06:00:00.000Z",
        "seconds_ago": 4,
        "online": true,
        "wallets": [
          {
            "id": "...",
            "name": "bKash",
            "provider": "bkash",
            "wallet_number": "01...",
            "last_payment_at": "2026-09-10T05:55:00.000Z"
          }
        ]
      }
    ],
    "wallets": [
      {
        "id": "...",
        "name": "bKash",
        "provider": "bkash",
        "wallet_number": "01...",
        "status": "active",
        "device_id": "abc-123",
        "device_name": "Pixel",
        "device_last_seen_at": "2026-09-10T06:00:00.000Z",
        "device_seconds_ago": 4,
        "device_online": true,
        "last_payment_at": "2026-09-10T05:55:00.000Z"
      }
    ],
    "wallets_total_active": 1,
    "checked_at": "2026-09-10T06:00:05.000Z"
  }
}
```

### Field notes

| Field | Meaning |
|-------|---------|
| `wallets[].device_id` | Device that last received a payment for this wallet |
| `wallets[].device_seconds_ago` | Seconds since that device’s last server ping (`null` if unknown) |
| `wallets[].device_online` | Convenience flag: last ping within **2 minutes** |
| `wallets[].last_payment_at` | Time of the payment used to bind wallet → device |
| `devices[].seconds_ago` | Seconds since this device last pinged |
| `devices[].wallets` | Wallets whose last payment arrived on this device |
| `any_online` | At least one device pinged within 2 minutes |

**Removed:** `can_accept_payments` — your server chooses the offline threshold (e.g. reject if `device_seconds_ago > 120`).

**Related (verify):** payments older than **6 hours** return `TOO_OLD` with `review_required` (not auto-expired). Default `max_age_minutes` on verify is **360**.
