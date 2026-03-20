# 🛡️ GigShield

### Dynamic Micro-Insurance for Gig Economy Workers

![React](https://img.shields.io/badge/Frontend-React.js-blue)
![Tailwind](https://img.shields.io/badge/UI-TailwindCSS-38B2AC)
![FastAPI](https://img.shields.io/badge/Backend-FastAPI-green)
![Supabase](https://img.shields.io/badge/Database-Supabase-orange)
![PostgreSQL](https://img.shields.io/badge/DB-PostgreSQL-blue)
![OpenWeather](https://img.shields.io/badge/API-OpenWeatherMap-yellow)
![Stripe](https://img.shields.io/badge/Payments-Stripe-purple)
![Guidewire](https://img.shields.io/badge/Integration-Guidewire-red)
![Status](https://img.shields.io/badge/Build-MVP%20Ready-success)

---

## 🌧️ Inspiration

Every monsoon in Chennai, thousands of delivery partners like **Raj** ride through flooded streets — not by choice, but because missing work means missing rent.

> Why does someone earning ₹18,000/month have less protection than someone earning ₹5 lakh?

Traditional insurance fails gig workers:

* ❌ Flat premiums
* ❌ Manual claims
* ❌ Slow payouts
* ❌ No real-time risk awareness

👉 **GigShield fixes this.**

---

## 💡 What is GigShield?

GigShield is a **real-time, parametric micro-insurance system** designed for gig workers.

It provides:

* 📊 Dynamic risk-based pricing
* ⚡ Instant payouts (no claims required)
* 🛡️ Built-in fraud detection

---

## ⚙️ How It Works

### 🧠 Risk Engine

```
Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)
```

### 💰 Pricing Model

```
Weekly Premium = Base × Risk × Behaviour − Trust Discount
```

👉 Example (Raj):

```
₹150 × 1.3 × 0.9 − 20% = ₹140/week
```

---

### ⚡ Parametric Payout System

No claim needed.

1. Weather API detects heavy rain (>15 mm/hr)
2. System triggers payout automatically
3. Fraud checks run instantly
4. Payment is processed

⏱️ **Total time: < 60 seconds**

---

## 🛡️ Fraud Detection System

### Tier 1 (Real-time)

* IP Geolocation (ipapi.co)
* Browser GPS validation
* Session tracking (Supabase)
* Timezone verification

### Tier 2 (Behavioural)

* Movement pattern consistency
* Network latency analysis
* Route history validation

### Tier 3 (Designed for Scale)

* Fraud ring detection (clustering)
* Social pattern analysis
* Environment cross-verification (news APIs)

---

## 🧱 System Architecture

```
        ┌──────────────────────┐
        │   React Frontend     │
        │ (Worker + Admin UI)  │
        └──────────┬───────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │   FastAPI Backend    │
        │----------------------│
        │ Risk Engine          │
        │ Pricing Engine       │
        │ Fraud Detection      │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │   Supabase (DB)      │
        │ Workers / Claims     │
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │ External APIs        │
        │ Weather / IP / Pay   │
        └──────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer               | Technology              | Purpose                                 |
| ------------------- | ----------------------- | --------------------------------------- |
| **Frontend**        | React.js + Tailwind CSS | Worker dashboard + Admin panel          |
| **Backend**         | FastAPI (Python)        | Core logic: risk, pricing, fraud        |
| **Database**        | Supabase (PostgreSQL)   | Users, policies, claims                 |
| **Auth & Sessions** | Supabase Auth           | Session tracking & device validation    |
| **Weather API**     | OpenWeatherMap          | Real-time environmental data            |
| **IP Validation**   | ipapi.co                | Fraud detection (location verification) |
| **Browser APIs**    | Geolocation + WebRTC    | GPS + anti-spoofing                     |
| **Payments**        | Stripe (Sandbox)        | Premium + payout simulation             |
| **Realtime**        | Supabase Realtime       | Instant notifications                   |
| **Insurance Layer** | Guidewire APIs          | Policy + claims integration             |

---

## 🎯 MVP (What We Actually Built)

We focused on a **real, working system**:

* ✅ Rule-based risk scoring engine
* ✅ Dynamic premium calculation
* ✅ Weather-triggered payout system
* ✅ Basic fraud detection:

  * IP validation
  * GPS consistency
  * Session tracking
* ✅ Live React dashboard
* ✅ Simulated Guidewire integration

---

## 🚀 Why GigShield Wins

### ⚡ Instant Claims → No Friction

Traditional insurance: **days**
GigShield: **seconds**

### 📊 Real-Time Pricing

Premium adapts to:

* Weather
* Work pattern
* Behaviour

### 🛡️ Fraud-Resistant by Design

Multi-signal validation — not just GPS

### 💰 Built for Low-Income Workers

Affordable, weekly, contextual insurance

---

## 🔗 Guidewire Integration

GigShield acts as an **embedded insurance layer on top of Guidewire**:

* PolicyCenter → Policy creation
* BillingCenter → Premium collection
* ClaimCenter → Automated payouts

👉 Demonstrates how Guidewire can power **next-gen gig economy insurance products**

---

## 🏁 Conclusion

GigShield is not just insurance.

It is:

> A real-time financial safety net for millions of gig workers.

---

### 👥 Team

Built for Guidewire Hackathon 🚀
