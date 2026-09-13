<p align="center">
  <img src="docs/icon.png" width="120" alt="PayPilot logo"/>
</p>

<h1 align="center">PayPilot</h1>

<p align="center">
  <b>Automatic payment verification for your website</b><br/>
  Confirm bKash &amp; Nagad payments in real time — from your own server, with a single API call.
</p>

<p align="center">
  <a href="docs/API.md"><img src="https://img.shields.io/badge/API-docs-2563EB" alt="API docs"/></a>
  <a href="docs/API.bn.md"><img src="https://img.shields.io/badge/API-বাংলা-0ea5e9" alt="API BN"/></a>
  <img src="https://img.shields.io/badge/license-Proprietary-red" alt="License"/>
</p>

<p align="center">
  <a href="#-how-it-works">How it works</a> ·
  <a href="#-licensing">Licensing</a> ·
  <a href="docs/API.md">API</a> ·
  <a href="README.bn.md">🇧🇩 বাংলা</a>
</p>

---

## What is PayPilot?

PayPilot is an **auto payment check** system for online businesses in Bangladesh.

When a customer pays you through mobile banking, your **private server** asks PayPilot:

> “Is this transaction ID real and still unused?”

PayPilot answers in one secure API call — so you can complete the order automatically, without manual SMS checking.

---

## How it works

```
Customer pays (bKash / Nagad)
        ↓
Customer submits TrxID on your website
        ↓
Your backend  ── POST /api/v1/verify ──►  PayPilot
        ↓
PayPilot confirms (one-time claim)
        ↓
Your backend delivers the product / service
```

- Call is made **only from your server** — never from the browser.
- Each successful check **claims** the TrxID once (`ALREADY_USED` if repeated).
- Optional amount match for extra safety.

Full request/response details: **[API documentation](docs/API.md)**

---

## Licensing

PayPilot is a **licensed commercial product**.

| License | Purpose |
|--------|---------|
| **App license** | Activates the companion app on the receiving phone |
| **API token** | Authorizes your server to call payment verification |

- Issued **per merchant account**
- Regenerating a key **immediately revokes** the previous one
- Keep API tokens on the server only — never in frontend code or public repos

See also: [License](docs/License.md) · [Terms](PayPilot_TC.md)

---

## Security in short

- Verify only with your **API token** over HTTPS  
- One-time claim — no double spending of the same TrxID  
- Optional amount check against the real payment  

---

## Contact

For license and access, contact the PayPilot operator who issued your account.

---

<p align="center">
  <sub>© PayPilot · Proprietary software · Not open source</sub>
</p>
