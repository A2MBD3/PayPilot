<p align="center">
  <img src="docs/icon.png" width="120" alt="PayPilot logo"/>
</p>

<h1 align="center">PayPilot</h1>

<p align="center">
  <b>ওয়েবসাইটের জন্য স্বয়ংক্রিয় পেমেন্ট যাচাই</b><br/>
  bKash ও Nagad পেমেন্ট — আপনার নিজের সার্ভার থেকে একটা API কলেই নিশ্চিত করুন।
</p>

<p align="center">
  <a href="docs/API.bn.md"><img src="https://img.shields.io/badge/API-বাংলা-0ea5e9" alt="API BN"/></a>
  <a href="docs/API.md"><img src="https://img.shields.io/badge/API-English-2563EB" alt="API EN"/></a>
  <img src="https://img.shields.io/badge/license-Proprietary-red" alt="License"/>
</p>

<p align="center">
  <a href="#-কিভাবে-কাজ-করে">কিভাবে কাজ করে</a> ·
  <a href="#-লাইসেন্সিং">লাইসেন্সিং</a> ·
  <a href="docs/API.bn.md">API</a> ·
  <a href="README.md">🇬🇧 English</a>
</p>

---

## PayPilot কী?

PayPilot হলো বাংলাদেশের অনলাইন ব্যবসার জন্য **অটো পেমেন্ট চেক** সিস্টেম।

কাস্টমার মোবাইল ব্যাংকিং দিয়ে টাকা পাঠালে আপনার **প্রাইভেট সার্ভার** PayPilot-কে জিজ্ঞেস করে:

> “এই Transaction ID সত্যি কি এবং আগে ব্যবহার হয়নি তো?”

একটি সুরক্ষিত API কলেই উত্তর পাবেন — হাতে SMS মিলিয়ে দেখার দরকার নেই; অর্ডার অটোমেটিক সম্পন্ন করতে পারবেন।

---

## কিভাবে কাজ করে

```
কাস্টমার পেমেন্ট করে (bKash / Nagad)
        ↓
ওয়েবসাইটে TrxID দেয়
        ↓
আপনার ব্যাকএন্ড  ── POST /api/v1/verify ──►  PayPilot
        ↓
PayPilot একবারের জন্য কনফার্ম করে
        ↓
আপনার সার্ভার প্রোডাক্ট/সার্ভিস ডেলিভার করে
```

- রিকোয়েস্ট **শুধু আপনার সার্ভার** থেকে যাবে — ব্রাউজার/ক্লায়েন্ট সাইড নয়।
- সফল চেক হলে সেই TrxID **একবারই** ব্যবহার হয় (আবার চাইলে `ALREADY_USED`)।
- চাইলে amount মিলিয়ে অতিরিক্ত নিরাপত্তা নিতে পারেন।

বিস্তারিত: **[API ডকুমেন্টেশন](docs/API.bn.md)**

---

## লাইসেন্সিং

PayPilot একটি **লাইসেন্সভিত্তিক** পণ্য।

| লাইসেন্স | কাজ |
|----------|-----|
| **App license** | রিসিভ ফোনে অ্যাপ চালু করে |
| **API token** | আপনার সার্ভার থেকে পেমেন্ট ভেরিফাই করার অনুমতি |

- প্রতি মার্চেন্ট অ্যাকাউন্ট অনুযায়ী ইস্যু হয়  
- নতুন কী বানালে **আগেরটা সাথে সাথে বাতিল**  
- API token শুধু সার্ভারে রাখুন — ফ্রন্টএন্ড বা পাবলিক রিপোতে নয়  

আরও: [License](docs/License.md) · [শর্তাবলি](PayPilot_TC.md)

---

## নিরাপত্তা (সংক্ষেপ)

- শুধু **API token** দিয়ে HTTPS-এ verify  
- একই TrxID দিয়ে দ্বিগুণ ক্লেইম সম্ভব নয়  
- ঐচ্ছিক amount ম্যাচ  

---

## যোগাযোগ

লাইসেন্স ও অ্যাকাউন্টের জন্য যে অপারেটর আপনাকে অ্যাকাউন্ট দিয়েছেন, তার সাথে যোগাযোগ করুন।

---

<p align="center">
  <sub>© PayPilot · Proprietary · ওপেন সোর্স নয়</sub>
</p>
