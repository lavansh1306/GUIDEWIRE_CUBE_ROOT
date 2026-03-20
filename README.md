# 🛡️ GigShield

### Dynamic Micro-Insurance for Gig Economy Workers


![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)!![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-5469d4?style=for-the-badge&logo=stripe&logoColor=ffffff)
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

## 👤 Persona

**Name:** Raj | **Age:** 25 | **Platform:** Swiggy | **Location:** Chennai
**Income:** ₹18,000/month (~₹4,500/week)

| Disruption Type | Examples |
|---|---|
| Environmental | Heavy rain, flooding, heatwaves |
| Social | Political rallies, elections, local strikes |

Raj works 10–12 hours/day on a mid-range Android. His income is volatile,
his exposure is real, and his safety net is essentially zero.
GigShield targets consistent workers like Raj — those who show up daily
and deserve protection when the environment fails them, not when they do.

---

## 💡 What is GigShield?

GigShield is a **real-time, parametric micro-insurance system** for gig
delivery workers — automatically priced by ML, automatically triggered,
and fraud-resistant by design.

* 📊 **Dynamic ML-powered risk pricing** — XGBoost classifier trained on
  synthetic Chennai delivery partner profiles
* ⚡ **Instant parametric payouts** — no claim form, no phone call, no waiting
* 🛡️ **Multi-signal ML fraud detection** — Random Forest scorer +
  Isolation Forest anomaly detection
* 🔗 **Guidewire integration** — PolicyCenter, ClaimCenter, BillingCenter

---

## ⚙️ How It Works

### 1. Worker Identification

Raj does not fill any form. The system automatically identifies and
onboards eligible delivery partners using behavioural and environmental
signals from a synthetic dataset of Chennai delivery partner profiles
built to replicate monsoon seasonality and delivery density.

---

### 2. 🧠 Risk Scoring Engine (FastAPI + XGBoost)
```
Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)
```

**Environmental:** Rain intensity, flood probability, temperature, AQI, wind speed

**Operational:** Avg working hours, peak-hour dependency, delivery density,
order frequency drop, route type

**Behavioural:** Movement pattern, speed variation, stop frequency,
route consistency

Weights and tier thresholds are learned by a trained **XGBoost classifier**
on synthetic worker profiles, capturing non-linear relationships between
weather severity, worker behaviour, and income loss risk.

Maps to: **Low / Medium / High / Extreme**

---

### 3. 💰 Pricing Engine (FastAPI + Regression Model)
```
Weekly Premium = Base × Risk Multiplier × Behaviour Modifier − Trust Discount
```

| Risk Level | Multiplier | Behaviour | Modifier |
|---|---|---|---|
| Low | 0.8 | Consistent | 0.9 |
| Medium | 1.0 | Normal | 1.0 |
| High | 1.3 | Irregular | 1.2 |
| Extreme | 1.6 | | |

| Trust Level | Discount |
|---|---|
| High | −20% |
| Medium | −10% |
| Low | 0% |

Trust parameters: claim history, claim success rate, suspicious behaviour
flags, GPS anomaly history, micro-task verification success.

Multipliers are outputs of a **regression model** trained on historical
Chennai weather data — not fixed lookup values. A worker in a flood-prone
zone during northeast monsoon gets a dynamically higher multiplier than
the same profile in May.

**Raj's case:**
```
Base ₹150 × High (1.3) × Consistent (0.9) − High Trust (−20%) = ₹140/week
```

Premium range: ₹90 (floor) to ₹360 (ceiling)
Validated via Monte Carlo simulation across 10,000 synthetic Chennai
weather weeks to confirm pool solvency.

---

### 4. ⚡ Parametric Automation Flow

Primary flow is **fully automatic** — the worker never needs to file a
claim. A manual claim option exists for edge cases where the automatic
trigger did not fire but the worker was genuinely affected (e.g. localised
flooding not captured by OpenWeatherMap's zone average). Manual claims
go through the same Tier 1 + Tier 2 fraud checks before payout is approved.
```
1. OpenWeatherMap polls Chennai weather every 15 minutes
2. Rain intensity crosses threshold (>15mm/hr)
   → FastAPI trigger service fires automatically
3. Fraud service runs Tier 1 checks in parallel:
   - ipapi.co confirms worker IP is in Chennai zone
   - Supabase session record confirms worker was active today
   - Browser geolocation matches claimed weather zone
   - Random Forest fraud classifier scores the claim
   - Isolation Forest flags statistical anomalies in claim pattern
4. All checks pass
   → Guidewire PolicyCenter updates policy status to
     "payout triggered" via REST API
   → Guidewire ClaimCenter creates and closes the
     claim record automatically
   → Guidewire BillingCenter initiates payment
     instruction to Stripe
5. Stripe releases payment to worker wallet
6. Supabase Realtime pushes notification to worker's browser:
   "Heavy rain detected → ₹140 credited to your account"
```

⏱️ **Total time from trigger to payout: under 60 seconds**

---

## 🛡️ Fraud Detection System

### Tier 1 — Automatic (runs on every claim)

* **IP Geolocation (ipapi.co)** — worker IP resolved to real-world zone,
  cross-referenced against claimed weather zone
* **Browser GPS + OpenWeatherMap cross-check** — GPS coordinates
  validated against actual weather at that location
* **Session Fingerprinting (Supabase)** — device ID, login timestamps,
  session duration, activity patterns, timezone, IP consistency
* **WebRTC Leak Detection** — browser local IP tracked across sessions,
  inconsistency flags claim
* **Suspicious Location Change Flag** — any location change in 7 days
  before claim attempt logged as risk signal
* **Random Forest Fraud Classifier** — trained on synthetic fraud/
  legitimate claim profiles. Input features: IP zone match, GPS deviation,
  session duration, timezone consistency, WebRTC IP match, claim timing
  vs weather event. Output: fraud probability score 0–1

### Tier 2 — Behavioural (runs on flagged claims)

* **Isolation Forest Anomaly Detection** — unsupervised ML flags
  statistical outliers in claim volume, timing, and location clustering
* **Location Fingerprint Analysis** — deviation from established delivery
  zones, routes, and time patterns raises fraud score
* **Network Behaviour Analysis** — FastAPI sends 3–5 rapid requests,
  checks latency drift, jitter, packet timing. Real workers in bad weather
  show consistent network degradation — spoofers do not

### Tier 3 — Fraud Ring Detection (designed, Phase 3)

* Social graph checks: similar GPS clusters, same IP ranges, identical
  behaviour timing across accounts
* Environment cross-check: local news API verifies ground conditions
  match the claim
* DBSCAN clustering: 50+ claims within 30 minutes from a 500m GPS
  radius but IPs from different neighbourhoods → entire cluster flagged
* Physics challenge (optional, mobile web only): camera snapshot +
  gyroscope liveness check for highest-risk flagged users only

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
        │ XGBoost Risk Engine  │
        │ Regression Pricing   │
        │ RF + IF Fraud Layer  │
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
        │ Guidewire Suite      │
        └──────────────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | React.js + Tailwind CSS | Worker dashboard + Admin panel |
| **Backend** | FastAPI (Python) | Core logic: risk, pricing, fraud |
| **ML** | scikit-learn / XGBoost | Fraud classifier, premium regression, anomaly detection |
| **Database** | Supabase (PostgreSQL) | Workers, policies, claims, audit logs |
| **Auth & Sessions** | Supabase Auth | Session tracking + device validation |
| **Weather API** | OpenWeatherMap | Real-time environmental data |
| **IP Validation** | ipapi.co | Fraud detection — location verification |
| **Browser APIs** | Geolocation + WebRTC | GPS validation + anti-spoofing |
| **Payments** | Stripe (Sandbox) | Premium deduction + payout simulation |
| **Realtime** | Supabase Realtime | Instant payout notifications |
| **Insurance Layer** | Guidewire InsuranceSuite APIs | PolicyCenter — policy status updates. ClaimCenter — auto claim creation and closure. BillingCenter — payment instruction to Stripe |

💰 **Entire demo stack cost: ₹0** — all services on free/sandbox tiers.

---

## 🎯 What We Built

* ✅ XGBoost risk scoring engine
* ✅ Regression-based dynamic premium calculation
* ✅ Weather-triggered parametric payout flow
* ✅ Random Forest fraud classifier (Tier 1)
* ✅ Isolation Forest anomaly detection (Tier 2)
* ✅ IP validation + GPS consistency + session fingerprinting
* ✅ WebRTC leak detection
* ✅ Live React dashboard — worker + admin views
* ✅ Guidewire PolicyCenter + ClaimCenter + BillingCenter integration
* ✅ Manual claim fallback with identical fraud checks

---

## 🔗 Guidewire Integration

GigShield sits as an **embedded insurance layer on top of Guidewire**:

* **PolicyCenter** → updates policy status to "payout triggered" on
  weather event
* **ClaimCenter** → auto-creates and closes claim record
* **BillingCenter** → initiates payment instruction to Stripe

Demonstrates how Guidewire can power **next-gen gig economy insurance.**

---

## 🚀 Why GigShield Wins

| Traditional Insurance | GigShield |
|---|---|
| Flat premiums | ML-dynamic weekly pricing |
| Manual claim forms | Zero-action parametric payout |
| Days to process | Under 60 seconds |
| GPS-only fraud check | 6-signal ML fraud scoring |
| No gig worker focus | Built exclusively for delivery partners |

---

## 📊 ML Summary

| Model | Type | Purpose |
|---|---|---|
| XGBoost Classifier | Supervised | Risk tier classification |
| Regression Model | Supervised | Dynamic premium calculation |
| Random Forest | Supervised | Per-claim fraud scoring |
| Isolation Forest | Unsupervised | Anomaly detection on claim patterns |

All models trained on synthetic Chennai delivery partner dataset
engineered to replicate monsoon seasonality and delivery density.

---

## 🏁 Conclusion

GigShield is not just insurance.

> A real-time ML-powered financial safety net for millions of gig workers
> who currently have none.

---

### 👥 Team

Built for Guidewire DEVTrails 2026 🚀

---

🔗 **GitHub Repository:** https://github.com/your-org/gigshield
_(link to be updated before submission deadline)_

🎥 **Demo Video:** https://youtu.be/your-demo-link
_(link to be updated before submission deadline)_
