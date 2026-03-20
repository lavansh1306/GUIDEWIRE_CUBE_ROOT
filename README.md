# 🛡️ RAKSHA X
### Dynamic Micro-Insurance for Gig Economy Workers  

![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-5469d4?style=for-the-badge&logo=stripe&logoColor=ffffff)
![XGBoost](https://img.shields.io/badge/ML-XGBoost-red?style=for-the-badge)
![Guidewire](https://img.shields.io/badge/Integration-Guidewire-orange?style=for-the-badge)

---

## 👤 Persona

**Name:** Raj | **Age:** 25 | **Platform:** Swiggy | **Location:** Chennai
**Monthly Income:** ₹18,000/month (~₹4,500/week) | **Work:** 10–12 hours/day

**Environmental disruptions:** Heavy rain, flooding, heatwaves
**Social disruptions:** Political rallies, elections, local strikes
**Tech behavior:** Mid-range Android, active on Instagram, Telegram, Facebook
**Risk context:** Unstable income, potential for system abuse, no existing safety net

> Our insurance targets consistent workers like Raj — those who show up daily
> and deserve protection when the environment fails them, not when they do.

---

## ⚙️ Workflow

### 1️⃣ Worker Identification

Raj does not fill any form manually. The system identifies and
onboards eligible delivery partners automatically using behavioural
and environmental signals from synthetic Swiggy data.

Real driver data is not publicly available — a synthetic dataset of Chennai
delivery partner profiles was built to replicate monsoon seasonality
and delivery density.

Eligibility is determined by the Risk Score engine before a policy is issued.

---

### 2️⃣ Risk Score
```
Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)
```

Weights and tier thresholds are learned by a trained **XGBoost classifier**
on synthetic worker profiles, capturing non-linear relationships between
weather severity, worker behaviour, and income loss risk.

Maps to: **Low / Medium / High / Extreme**

---

#### 🌧️ Environmental Risk
- Rain intensity (mm/hr)
- Flood probability (zone-based)
- Temperature (heatwaves)
- Air Quality Index (AQI)
- Wind speed (storm risk)

#### 📦 Operational Risk
- Average working hours/day
- Peak-hour dependency (lunch/dinner rush)
- Delivery density in area
- Order frequency drop during disruptions
- Route type (high flood-prone roads)

#### 🧠 Behavioural Risk
- Movement pattern (continuous vs static)
- Speed variation (real vs fake travel)
- Stop frequency (traffic signals, rest stops)
- Route consistency

---

### 3️⃣ Weekly Pricing Model
```
Weekly Premium = Base Premium × Risk Multiplier × Behaviour Modifier − Trust Discount
```

Base Premium = 2–5% of weekly income → Raj: ₹4,500/week → **Base = ₹150**

| Risk Level | Multiplier | Behaviour | Modifier |
|---|---|---|---|
| Low | 0.8 | Consistent | 0.9 |
| Medium | 1.0 | Normal | 1.0 |
| High | 1.3 | Irregular | 1.2 |
| Extreme | 1.6 | | |

| Trust Level | Discount | Parameters |
|---|---|---|
| High | −20% | Clean claim history, no flags |
| Medium | −10% | Minor anomalies |
| Low | 0% | Suspicious behaviour detected |

**Raj's case:**
```
₹150 × 1.3 (High) × 0.9 (Consistent) − 20% (High Trust) = ₹140/week
```

Premium range: **₹90 (floor) to ₹360 (ceiling)**
Validated via Monte Carlo simulation across 10,000 synthetic Chennai
weather weeks to confirm pool solvency.

> Multipliers are outputs of a regression model trained on historical
> Chennai weather data — not fixed lookup values.

---

### 4️⃣ Parametric Automation Flow

Primary flow is **fully automatic** — the worker never needs to file a claim.
A manual claim option exists for edge cases where the trigger did not fire
but the worker was genuinely affected. Manual claims go through identical
fraud checks before approval.
```text
Step 1: OpenWeatherMap polls Chennai weather every 15 minutes
Step 2: Rain intensity crosses threshold (>15mm/hr)
         → FastAPI trigger service fires automatically
Step 3: Fraud checks run in parallel (Tier 1):
         - ipapi.co confirms worker IP is in Chennai zone
         - Supabase session confirms worker was active today
         - Browser geolocation matches claimed weather zone
         - Random Forest fraud classifier scores the claim
         - Isolation Forest flags statistical anomalies
Step 4: All checks pass
         → Guidewire PolicyCenter updates policy status to
           "payout triggered" via REST API
         → Guidewire ClaimCenter creates and closes the
           claim record automatically
         → Guidewire BillingCenter initiates payment
           instruction to Stripe
Step 5: Stripe releases payment to worker wallet
Step 6: Supabase Realtime pushes notification to browser:
         "Heavy rain detected → ₹140 credited to your account"
```

⏱️ **Total time from trigger to payout: under 60 seconds**

---

### 5️⃣ Fraud Detection

Detection runs across three tiers by complexity.

#### Tier 1 — Automatic (every claim)

- **IP Geolocation (ipapi.co)** — IP resolved to real-world zone, cross-referenced against claimed weather zone
- **Browser GPS + OpenWeatherMap** — GPS coordinates validated against actual weather at that location
- **Session Fingerprinting (Supabase)** — device ID, timestamps, session duration, timezone, IP consistency
- **WebRTC Leak Detection** — browser local IP tracked across sessions, inconsistency flags claim
- **Location Change Flag** — any location change in 7 days before claim logged as risk signal
- **Random Forest Classifier** — trained on synthetic fraud/legitimate profiles. Inputs: IP zone match, GPS deviation, session duration, timezone consistency, WebRTC IP match, claim timing vs weather. Output: fraud probability 0–1

#### Tier 2 — Behavioural (flagged claims only)

- **Isolation Forest** — unsupervised ML flags statistical outliers in claim volume, timing, location clustering
- **Location Fingerprint Analysis** — deviation from established delivery zones and routes raises fraud score
- **Network Behaviour Analysis** — latency drift, jitter, packet timing. Real workers in bad weather show consistent degradation — spoofers do not

#### Tier 3 — Fraud Ring Detection (Phase 3)

- DBSCAN clustering: 50+ claims in 30 minutes from 500m GPS radius but IPs from different neighbourhoods → entire cluster flagged
- Local news API cross-check verifies ground conditions match the claim
- Social graph analysis: same IP ranges, identical behaviour timing across accounts

---

### 6️⃣ UX Flow

**Worker Dashboard**
```
┌──────────────────────────────────┐
│  RAKSHA X                        │
│  Hey Raj, you are protected ✓    │
│                                  │
│  This week's premium: ₹140       │
│  Auto-deducted every Sunday      │
│                                  │
│  Chennai · Rain: 18mm/hr ⚠       │
│  Above threshold                 │
│                                  │
│  → Payout triggered automatically│
│  ₹140 credited to your account   │
│                                  │
│  [File Manual Claim]             │
└──────────────────────────────────┘
```

**Admin / Insurer Dashboard**
- Active policies and premium pool balance
- Live weather trigger log
- Fraud flag queue with tier classification
- Payout history and loss ratio
- Worker risk score breakdown

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | React.js + Tailwind CSS | Worker dashboard + admin portal |
| **Backend** | FastAPI (Python) | Risk scoring, pricing, fraud microservices |
| **ML** | scikit-learn / XGBoost | Risk classifier, regression pricing, RF fraud scorer, IF anomaly detection |
| **Database** | Supabase (PostgreSQL) | Worker profiles, policies, claims, audit logs |
| **Auth** | Supabase Auth | Session tracking + device validation |
| **Weather** | OpenWeatherMap API | Live rain, temperature, AQI for Chennai |
| **IP Validation** | ipapi.co | IP-to-location fraud check |
| **Browser APIs** | Geolocation + WebRTC | GPS validation + anti-spoofing |
| **Payments** | Stripe (Sandbox) | Premium deduction + payout simulation |
| **Realtime** | Supabase Realtime | Instant payout notifications |
| **Insurance** | Guidewire InsuranceSuite | PolicyCenter → status update · ClaimCenter → auto create/close · BillingCenter → payment to Stripe |

💰 **Entire demo stack cost: ₹0** — all services on free/sandbox tiers.

---

## 🏗️ System Architecture
```text
┌─────────────────────────────────────────────────────────────┐
│                     FRONTEND (React.js)                     │
│         Worker Dashboard  │  Admin / Insurer Portal         │
└───────────────────────────┬─────────────────────────────────┘
                            │ REST API Calls
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                     BACKEND (FastAPI)                       │
│                                                             │
│   ┌─────────────────┐  ┌──────────────┐  ┌──────────────┐  │
│   │   Risk Engine   │  │   Pricing    │  │    Fraud     │  │
│   │   (XGBoost)     │  │   Engine     │  │  Detection   │  │
│   │                 │  │ (Regression) │  │  (RF + IF)   │  │
│   └────────┬────────┘  └──────┬───────┘  └──────┬───────┘  │
│            └─────────────────┼──────────────────┘          │
│                              │                              │
│   ┌───────────────────────────▼─────────────────────────┐  │
│   │              Parametric Trigger Service              │  │
│   │   Monitors → Validates → Scores → Initiates Payout  │  │
│   └───────────────────────────┬─────────────────────────┘  │
└───────────────────────────────┼─────────────────────────────┘
                                │
          ┌─────────────────────┼──────────────────────┐
          │                     │                      │
          ▼                     ▼                      ▼
┌─────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│  EXTERNAL APIs  │  │    SUPABASE      │  │    GUIDEWIRE     │
│                 │  │                  │  │                  │
│ OpenWeatherMap  │  │ Worker Profiles  │  │ PolicyCenter     │
│ Rain / AQI /    │  │ Policies         │  │ → status update  │
│ Temperature     │  │ Claims           │  │                  │
│                 │  │ Audit Logs       │  │ ClaimCenter      │
│ ipapi.co        │  │ Session Data     │  │ → auto create    │
│ IP Geolocation  │  │                  │  │   & close claim  │
│                 │  │ Realtime         │  │                  │
│ WebRTC          │  │ → push notif     │  │ BillingCenter    │
│ Local IP leak   │  │   to browser     │  │ → payment inst.  │
└─────────────────┘  └──────────────────┘  └────────┬─────────┘
                                                     │
                                                     ▼
                                          ┌──────────────────┐
                                          │  STRIPE SANDBOX  │
                                          │                  │
                                          │ Premium          │
                                          │ Auto-deduction   │
                                          │                  │
                                          │ Instant Payout   │
                                          │ to Worker Wallet │
                                          └──────────────────┘
```

### ⚡ Flow Summary
```text
Worker Session
      │
      ▼
Risk Scored (XGBoost) → Premium Set (Regression) → Policy Issued
      │
      ▼
Weather Event Detected (OpenWeatherMap)
      │
      ▼
Fraud Scored (Random Forest + Isolation Forest + 6-signal checks)
      │
      ├── PASS → Guidewire → Stripe → Payout → Worker notified
      │
      └── FAIL → Flagged → Tier 2 review → Manual assessment
```

---

## 🚀 Core Differentiators

- 📊 **ML-driven dynamic pricing** — XGBoost + regression, not flat premiums
- ⚡ **Zero-claim parametric payout** — under 60 seconds, no worker action
- 🛡️ **6-signal ML fraud detection** — spoofers have to beat all 6 at once
- 🔗 **Full Guidewire integration** — PolicyCenter, ClaimCenter, BillingCenter
- 💰 **Built for low-income workers** — weekly pricing, ₹90 floor premium


---

### 👥 Team
Built for Guidewire DEVTrails 2026 🚀

---

🔗 **GitHub Repository:** _(link to be updated before submission deadline)_
🎥 **Demo Video:** _(link to be updated before submission deadline)_
