<p align="center">
  <img src="docs/icon.png" width="120" alt="PayPilot logo"/>
</p>

<h1 align="center">PayPilot</h1>

<p align="center">
  <b>Real-time payment alert assistant for Bangladeshi merchants</b><br/>
  Every mobile-banking payment confirmation on your shop phone — instantly synced to your PayPilot dashboard.
</p>

<p align="center">
  <a href="https://github.com/A2MBD3/PayPilot/releases/latest"><img src="https://img.shields.io/badge/download-v1.2.3-10B981?logo=github&label=Latest%20release" alt="Download"/></a>
  <img src="https://img.shields.io/badge/Android-8.0%2B-3DDC84?logo=android&logoColor=white" alt="Android 8.0+"/>
  <img src="https://img.shields.io/badge/size-~2.5%20MB-blue" alt="APK size"/>
  <a href="docs/API.md"><img src="https://img.shields.io/badge/docs-API-2563EB" alt="API docs"/></a>
  <img src="https://img.shields.io/badge/license-Proprietary-red" alt="License"/>
</p>

<p align="center">
  <a href="#-features">Features</a> ·
  <a href="#-getting-started">Getting Started</a> ·
  <a href="#-security--privacy">Security</a> ·
  <a href="#-faq">FAQ</a> ·
  <a href="docs/API.md">🔌 API</a> ·
  <a href="#-contact--support">Contact</a> ·
  <a href="README.bn.md">🇧🇩 বাংলা সংস্করণ</a>
</p>

---

## 📖 About

**PayPilot** is a lightweight Android companion app for merchants who accept payments through mobile banking (bKash, Nagad, Rocket and similar services). Payment confirmations arrive as plain SMS, and keeping track of them manually is slow and error-prone. PayPilot solves this: once activated on the shop phone with a valid license, it quietly monitors incoming payment SMS and syncs them to the merchant's PayPilot web dashboard in real time, so the owner can see every transaction the moment it happens.

The app is built to run reliably around the clock. It starts automatically when the phone boots, keeps a minimal always-visible status notification while active, and stores everything locally in encrypted form. If the internet drops, pending messages are queued securely on the device and delivered automatically once the connection returns — no payment is ever lost.

> ⚠️ **PayPilot is a licensed product.** The app activates only with a valid license key issued per device. See [Licensing](#-licensing) for details.

## ✨ Features

| | Feature | Description |
|---|---|---|
| 📲 | **Real-time payment monitoring** | Payment confirmation SMS are captured the moment they arrive and synced to your dashboard within seconds. |
| 🔄 | **Never lose a payment** | No internet? Messages are kept in an encrypted on-device queue and delivered automatically when you're back online. |
| 🔐 | **Encrypted local storage** | Queued messages, counters and credentials are encrypted with hardware-backed keys (AES-256-GCM, Android Keystore). |
| 🛡️ | **Hardened connection** | All server communication runs over HTTPS with certificate pinning — man-in-the-middle tools cannot intercept your data. |
| 🔁 | **Auto-start & keep-alive** | Starts on boot, restarts if the system stops it, and guides you through battery-optimization exemption so it keeps running. |
| 📶 | **Dual-SIM support** | Every connected wallet gets its own card (name, logo, number); its SIM slot is auto-detected from the wallet number — or pick a slot manually per wallet. Both SIMs forward simultaneously. |
| 💳 | **Wallet cards** | All wallets linked to your account (even the same number on multiple wallets) show as separate cards with their logo, receiver number and SIM slot. |
| 🔔 | **Smart notifications** | A minimal, always-collapsed status notification shows that monitoring is running; you get an immediate high-priority alert if the connection fails. |
| 🧭 | **Clean merchant dashboard** | See your business logo, business name, account info, permission status and service state at a glance — with full dark mode. |
| ⬆️ | **In-app updates** | The app checks for new versions and notifies you when an update is available — always download only from this official page. |
| 🪶 | **Feather-light** | The whole app is ~2.5 MB and uses a few MB of data per month at most. |

## 🚀 Getting Started

1. **Request a license** — PayPilot activates per device. See [docs/License.md](docs/License.md) for the license details and application steps, or [contact the developer](#-contact--support) directly.
2. **Download the APK** — grab the latest `PayPilot-v1.2.3.apk` from the [Releases page](https://github.com/A2MBD3/PayPilot/releases/latest). Only ever download PayPilot from this official repository.
   > 📦 From v1.2.2 the app uses the package `com.a2mbd3.paypilot` — it installs as a separate app next to any older version; sign in with the same license key, then uninstall the old app.
3. **Install** — open the APK and allow "Install unknown apps" for your browser/file manager when Android asks (this is standard for apps distributed outside Google Play).
4. **Accept the Terms** — on first launch, read and accept the Terms & Conditions to continue.
5. **Activate** — paste your license key (there's a paste button — or shake the phone to clear the field), and the app verifies your license over a secure connection.
6. **Grant permissions** — approve SMS and notification permissions and follow the built-in guide to disable battery optimization for reliable background operation.
7. **Start monitoring** — tap **Start** and you're live. Your dashboard now updates in real time. 🎉

## ⚙️ System Requirements

| Requirement | Details |
|---|---|
| Operating system | Android 8.0 (Oreo) or newer |
| SIM | Active SIM that receives your mobile-banking payment SMS |
| Internet | Any stable mobile data or Wi-Fi connection |
| License | One valid PayPilot license per device |
| APK size | ~2.5 MB |

## 🔒 Security & Privacy

PayPilot is designed around a simple principle: **only payment confirmations, only for the licensed merchant.**

- **What the app reads** — while monitoring is active, PayPilot processes incoming payment-confirmation SMS on the licensed device. It does not read, store or send your personal messages, contacts, photos or files.
- **How data is protected on the device** — queued messages, counters and your license are stored encrypted (AES-256-GCM) using hardware-backed Android Keystore keys. Even if someone gets the phone, the data stays scrambled without the device.
- **How data travels** — all communication uses HTTPS with certificate pinning, so the connection cannot be silently intercepted or redirected.
- **What is never done** — PayPilot does not sell your data, does not show ads, and does not collect anything beyond what is needed to run the payment monitoring service.
- **Signed releases** — every official APK is digitally signed. If you download from this repository's Releases, you're getting the genuine app.

## ❓ FAQ

<details>
<summary><b>Do I need a license to use PayPilot?</b></summary>
Yes. PayPilot is a commercial, license-protected product. Each device needs its own valid license key. Contact the developer to get one.
</details>

<details>
<summary><b>Why does Android warn about "unknown apps" when installing?</b></summary>
PayPilot is distributed directly through GitHub rather than the Play Store, so Android shows the standard warning for any app installed outside an app store. It's normal — just allow the install for the app you're downloading with. Always make sure you download the APK only from this official page.
</details>

<details>
<summary><b>Why is there a permanent notification?</b></summary>
Android requires a visible notification for any app that keeps a background service running. PayPilot keeps it as small and quiet as possible — it simply tells you that the monitoring service is active and the server is reachable. If the connection ever fails, you'll get a separate, high-priority alert.
</details>

<details>
<summary><b>Will it drain my battery or data?</b></summary>
No. The service is deliberately lightweight — a tiny status check every few seconds and message sync only when payments arrive. Typical data usage is a few megabytes per month.
</details>

<details>
<summary><b>What happens if the internet goes down?</b></summary>
Payments received while offline are stored in an encrypted queue on the phone and automatically delivered when the connection returns. Nothing is lost.
</details>

<details>
<summary><b>Does it work with dual-SIM phones?</b></summary>
Yes. The app auto-detects the SIM that receives your payment SMS, and you can manually choose the SIM slot from the dashboard at any time.
</details>

<details>
<summary><b>Which Android versions are supported?</b></summary>
Android 8.0 (Oreo) and above — including the latest Android releases, with modern notification permissions handled automatically.
</details>

## 🔌 Payment verification API

Merchants verify bKash payments **from their private server** with a one-time claim API (`POST /api/v1/verify`). Use the **API license** from the dashboard Authorization tab — never embed it in a browser or public client.

📖 Full guide + code samples (Node, PHP, Python, Go, Java, cURL): **[docs/API.md](docs/API.md)** · [বাংলা](docs/API.bn.md)

```bash
curl -X POST https://paypilot-5p9t.onrender.com/api/v1/verify \
  -H "Authorization: Bearer $PAYPILOT_API_LICENSE" \
  -H "Content-Type: application/json" \
  -d '{"trx_id":"DI739OTDF3","amount":100.00,"order_id":"ORDER-1024"}'
```

## 📥 Downloads

| Version | Date | Download |
|---|---|---|
| **v1.2.3** (latest) | 2026-09-08 | [PayPilot-v1.2.3.apk](https://github.com/A2MBD3/PayPilot/releases/download/v1.2.3/PayPilot-v1.2.3.apk) — [release notes](https://github.com/A2MBD3/PayPilot/releases/tag/v1.2.3) |
| v1.2.2 | 2026-09-08 | package rename — superseded by v1.2.3 |
| v1.2.1 | 2026-09-08 | crash-fix release — superseded |
| v1.2.0 | 2026-09-08 | superseded — had a startup crash, please use v1.2.3 |

> 🔐 APK SHA-256 (v1.2.3): `08ca20123b6f3bd782ee552b8df1622e251d88d4f6f0f57f384cdf32747141d6`

> ⚠️ **v1.2.2 note:** the app package is now `com.a2mbd3.paypilot`, so that update installs as a **separate app** — sign in with your existing license key and then uninstall the old `com.teamcrx.paypilot` app. Updating from **v1.0.x**? Uninstall the old version first (the certificate changed at v1.2.0).

<details>
<summary><b>Update history</b></summary>

See the full [Changelog.md](Changelog.md) — the same file the app shows on its update screen.

</details>

## 📞 Contact & Support

Need a license, help with setup, or have an issue? Reach out any time:

| | Channel | Link |
|---|---|---|
| 👤 | **Owner** | Abdullah Al Mamun — *Team CRX* |
| 📧 | **Email** | [aam.abdullah1@hotmail.com](mailto:aam.abdullah1@hotmail.com) |
| 🌐 | **Portfolio** | [a2mbd3.pages.dev](https://a2mbd3.pages.dev) |
| 💻 | **GitHub** | [github.com/A2MBD3](https://github.com/A2MBD3) |
| 📘 | **Facebook** | [facebook.com/a2mbd3](https://www.facebook.com/a2mbd3) |
| 📸 | **Instagram** | [instagram.com/a2mbd3](https://instagram.com/a2mbd3) |
| ✈️ | **Telegram** | [t.me/a2mbd3](https://t.me/a2mbd3) |
| 📍 | **Location** | Barishal, Bangladesh |

## 📄 Licensing

PayPilot is **proprietary software**, not open source.

- The application, its design, branding and all source materials are © Abdullah Al Mamun (Team CRX). All rights reserved.
- Using the app requires a valid, per-device license key issued by the owner. The full license terms and conditions are shown inside the app on first launch.
- Redistribution, reverse engineering, modification or unauthorized commercial use of the app or its APK is not permitted.
- To purchase a license or discuss partnership, contact **aam.abdullah1@hotmail.com** or any of the channels above.

---

<p align="center">
  Made with care in Bangladesh 🇧🇩 by <a href="https://a2mbd3.pages.dev">Abdullah Al Mamun</a>
</p>
