# 🛡️ RAKSHA X
### Dynamic Micro-Insurance for Gig Economy Workers  

![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-5469d4?style=for-the-badge&logo=stripe&logoColor=ffffff)

---

## 👤 Persona

**Name:** Raj  
**Age:** 25  
**Platform:** Swiggy  
**Work:** 10–12 hours/day  
**Location:** Chennai  
**Monthly Income:** ₹18,000/month (~₹4,500/week)

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
Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)



Weights and tier thresholds are learned by a trained XGBoost classifier  
on synthetic worker profiles, capturing non-linear relationships between  
weather severity, worker behaviour, and income loss risk.

---

#### 🌧️ Environmental Risk

- Rain intensity (mm/hr)  
- Flood probability (zone-based)  
- Temperature (heatwaves)  
- Air Quality Index (AQI)  
- Wind speed (storm risk)  

---

#### 📦 Operational Risk

- Average working hours/day  
- Peak-hour dependency (lunch/dinner rush)  
- Delivery density in area  

---

#### 🧠 Behavioural Risk

- Daily login consistency  
- Order acceptance rate  
- Shift completion reliability  
- Historical inactivity patterns  

---

### 3️⃣ Pricing Engine

The computed Risk Score dynamically determines:

- 💰 Premium per day  
- 🛡️ Coverage amount  
- ⚡ Payout trigger thresholds  

> Higher risk → higher premium + higher protection

---

### 4️⃣ Policy Activation

- Short-term, dynamic policies (daily / shift-based)  
- Activated automatically based on risk thresholds  
- No manual onboarding or selection  

---

### 5️⃣ Parametric Payout System

- Triggered automatically when predefined conditions are met  
  *(e.g., rainfall > threshold, flood alert, extreme heat)*  

- ❌ No claim filing  
- ❌ No manual verification  
- ⚡ Instant payout  

---

### 6️⃣ Payment Flow

- Premium collection via Stripe  
- Instant payout to worker account upon trigger  

---

## 🚀 Core Differentiators

- 📊 ML-driven dynamic pricing (XGBoost)  
- ⚡ Zero-claim parametric insurance model  
- 🌧️ Real-time environmental risk integration  
- 🧠 Behaviour-aware fraud resistance  
- 📱 Built for low-income, mobile-first users  

---

## 🧠 System Insight

RAKSHA X is not just insurance — it is a **real-time risk adaptation system**  
designed for volatile income workers in unpredictable environments.

---

## 🔮 Future Scope

- Integration with real delivery platforms (Swiggy, Zomato)  
- Real-world dataset training for improved accuracy  
- Expansion to other gig sectors (ride-sharing, logistics)  
- Mobile-first deployment for field testing  

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
