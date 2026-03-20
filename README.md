# 🛡️ GigShield

> **Dynamic Micro-Insurance for Gig Economy Workers**
> *Real-time risk scoring · Personalised weekly premiums · Fraud-resistant auto-payouts*

[![Built at Guidewire Hackathon](https://img.shields.io/badge/Guidewire-Hackathon%202025-1F4E79?style=flat-square)](https://www.guidewire.com)
[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?style=flat-square&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React Native](https://img.shields.io/badge/React%20Native-Expo-61DAFB?style=flat-square&logo=react&logoColor=black)](https://expo.dev)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org)
[![Redis](https://img.shields.io/badge/Redis-7-DC382D?style=flat-square&logo=redis&logoColor=white)](https://redis.io)

---

## 📖 Table of Contents

- [Inspiration](#-inspiration)
- [What It Does](#-what-it-does)
- [Architecture](#-architecture)
- [Risk Scoring Engine](#-risk-scoring-engine)
- [Pricing Engine](#-pricing-engine)
- [Fraud Detection](#-fraud-detection)
- [Tech Stack](#-tech-stack)
- [Getting Started](#-getting-started)
- [Project Structure](#-project-structure)
- [Guidewire Integration](#-guidewire-integration)
- [What's Next](#-whats-next)
- [Team](#-team)

---

## 💡 Inspiration

Every monsoon season in Chennai, thousands of delivery partners like **Raj** keep riding through flooded streets — not because they want to, but because missing a shift means missing rent.

> *Why does a 25-year-old delivery partner earning ₹18,000/month have less financial protection than someone earning ₹5 lakh/month?*

Traditional insurance was never designed for the gig economy. Premiums are flat, forms are manual, and claims take days. Meanwhile, Raj faces a new risk every single day — heavy rain, heatwaves, political rallies, flooded roads. His income is volatile. His exposure is real. His safety net is essentially zero.

**GigShield** closes that gap with micro-insurance that is as dynamic as the work it protects.

---

## ✨ What It Does

GigShield delivers **contextual, weekly micro-insurance** for gig delivery workers — automatically priced, automatically triggered, and fraud-resistant by design.

| Feature | Description |
|---|---|
| 🔍 Auto-Identification | Identifies eligible workers from behavioural + environmental signals — no forms |
| 📊 Real-Time Risk Scoring | Scores each worker across Environmental, Operational, and Behavioural dimensions |
| 💰 Dynamic Pricing | Calculates a personalised weekly premium every renewal cycle |
| ⚡ Auto-Payouts | Verified environmental triggers release payouts in under 60 seconds |
| 🛡️ Fraud Detection | Multi-signal checks: IP, GPS, session fingerprint, timezone, cell tower |
| 🔗 Guidewire Integration | PolicyCenter + BillingCenter + ClaimCenter via REST API |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                       │
│   React Native (Worker App)    React.js (Admin Portal)      │
│              └──────────── Socket.io ────────────┘          │
└─────────────────────┬───────────────────────────────────────┘
                      │ REST / WebSocket
┌─────────────────────▼───────────────────────────────────────┐
│                    API Gateway (Node.js)                      │
│              JWT Auth · Rate Limiting · Routing              │
└────┬──────────────┬──────────────┬──────────────┬───────────┘
     │              │              │              │
┌────▼───┐  ┌───────▼──┐  ┌───────▼──┐  ┌───────▼──────┐
│  Risk  │  │ Pricing  │  │  Fraud   │  │  Policy /    │
│ Engine │  │ Engine   │  │ Service  │  │  Claims API  │
│FastAPI │  │ FastAPI  │  │ FastAPI  │  │  (Guidewire) │
└────┬───┘  └───────┬──┘  └───────┬──┘  └─────────────┘
     │              │              │
┌────▼──────────────▼──────────────▼──────────────────────────┐
│           Data Layer                                         │
│   PostgreSQL (core)  ·  Redis (cache/sessions)               │
│   OpenWeatherMap API  ·  NDMA Flood Zones  ·  Razorpay       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📊 Risk Scoring Engine

Each worker receives a composite risk score every week, combining three independently weighted sub-scores:

$$\text{Risk Score} = w_e \cdot S_{\text{env}} + w_o \cdot S_{\text{ops}} + w_b \cdot S_{\text{behav}}$$

| Weight | Dimension | Default |
|---|---|---|
| $w_e$ | Environmental | 0.4 |
| $w_o$ | Operational | 0.3 |
| $w_b$ | Behavioural | 0.3 |

### Environmental Sub-Score $S_{\text{env}}$
- Rain intensity (mm/hr) — live OpenWeatherMap API
- Flood probability — NDMA zone overlay
- Temperature / heatwave index — IMD alerts
- Air Quality Index — CPCB feed
- Wind speed — storm threshold

### Operational Sub-Score $S_{\text{ops}}$
- Average working hours/day
- Peak-hour dependency (lunch/dinner window)
- Delivery density in active zone
- Order frequency drop during disruptions
- Route type (flood-prone road classification)

### Behavioural Sub-Score $S_{\text{behav}}$
- Movement pattern (continuous vs static)
- Speed variation (GPS delta anomaly detection)
- Stop frequency (traffic signal legitimacy signal)
- Route consistency vs historical fingerprint

The composite score maps to a **risk tier**:

| Score Range | Tier | Risk Multiplier |
|---|---|---|
| 0.0 – 0.3 | Low | 0.8 |
| 0.3 – 0.5 | Medium | 1.0 |
| 0.5 – 0.7 | High | 1.3 |
| 0.7 – 1.0 | Extreme | 1.6 |

> **MVP note:** The demo uses rule-based thresholds. The architecture is designed to hot-swap an XGBoost classifier once labelled incident data is available.

---

## 💰 Pricing Engine

### Core Formula

$$\text{Weekly Premium} = P_{\text{base}} \times R_{\text{multiplier}} \times B_{\text{modifier}} - D_{\text{trust}}$$

$$P_{\text{final}} = \text{clip}\!\left(P_{\text{base}} \times R \times B - D,\; ₹90,\; ₹360\right)$$

| Variable | Description | Range |
|---|---|---|
| $P_{\text{base}}$ | 2–5% of weekly income | ₹90–₹225 (default ₹150) |
| $R_{\text{multiplier}}$ | Risk tier multiplier | {0.8, 1.0, 1.3, 1.6} |
| $B_{\text{modifier}}$ | Behaviour consistency | {0.9, 1.0, 1.2} |
| $D_{\text{trust}}$ | Trust history discount | 0%, 10%, 20% |

### Worked Example — Raj's Premium

| Factor | Value |
|---|---|
| Base Premium | ₹150 |
| Risk: High | × 1.3 |
| Behaviour: Consistent | × 0.9 |
| Sub-total | ₹150 × 1.3 × 0.9 = ₹175.50 |
| Trust Discount: High | − 20% |
| **Final Weekly Premium** | **₹140** |

### Pool Solvency Validation

Before locking multiplier values, we ran a Monte Carlo simulation across 10,000 simulated weather weeks to verify pool solvency:

$$\text{Pool Solvency} = \sum_{i=1}^{N} P_i - \mathbb{E}\!\left[\sum_{j \in \text{claims}} C_j\right] \geq 0$$

The pool remained solvent across a simulated 3-week consecutive flood event for Chennai at all tested worker pool sizes.

---

## 🛡️ Fraud Detection

Fraud detection is built into the **claim trigger** itself — not bolted on after the fact.

### Tier 1 — Implemented in Demo

| Check | Method |
|---|---|
| IP Deduplication | Concurrent claims from same IP blocked via Redis |
| GPS Consistency | Claim origin vs active delivery zone delta validation |
| Session Fingerprint | Device ID, login timestamps, session duration stored per worker |
| Timezone Cross-Check | Phone timezone vs claimed GPS location |
| SIM Validation | Mobile country code vs reported location |

### Tier 2 — Partial Implementation

| Check | Method |
|---|---|
| WebRTC Leak Detection | Local IP captured in-browser; tracked across sessions |
| Cell Tower Triangulation | Nearby tower IDs + signal strength as GPS-independent proxy |

### Tier 3 — Architected (Post-Hackathon)

Fraud ring detection for coordinated mass-claim attacks:

**Trust Score Formula:**

$$T = 1 - \frac{\alpha \cdot f_{\text{flags}} + \beta \cdot f_{\text{gps}} + \gamma \cdot f_{\text{verify}}}{N_{\text{checks}}}$$

Where $\alpha, \beta, \gamma$ are empirically tuned penalty weights. Trust score drives the premium discount:

| $T$ | Trust Level | Discount |
|---|---|---|
| ≥ 0.8 | High | −20% |
| 0.5 – 0.8 | Medium | −10% |
| < 0.5 | Low | 0% |

---

## 🔧 Tech Stack

| Layer | Technology | Status |
|---|---|---|
| Mobile App | React Native (Expo) | ✅ Demo |
| Admin Portal | React.js + Tailwind CSS | ✅ Demo |
| API Gateway | Node.js + Express | ✅ Demo |
| Risk Engine | Python + FastAPI | ✅ Demo |
| Pricing Engine | Python + FastAPI | ✅ Demo |
| Fraud Service | Python + FastAPI | ✅ Tier 1 / 🔄 Tier 2 |
| Real-Time Alerts | Socket.io | ✅ Demo |
| Primary Database | PostgreSQL 16 | ✅ Demo |
| Cache / Sessions | Redis 7 | ✅ Demo |
| Weather Data | OpenWeatherMap API | ✅ Demo |
| Payments | Razorpay SDK | ✅ Demo |
| Guidewire | PolicyCenter + BillingCenter APIs | ✅ Demo |
| ML Scoring | XGBoost (Scikit-learn) | 🔜 Post-hackathon |
| Orchestration | Kubernetes (EKS) | 🔜 Post-hackathon |
| Fraud Ring ML | NetworkX + DBSCAN | 🔜 Post-hackathon |

---

## 🚀 Getting Started

### Prerequisites

- Node.js 20+
- Python 3.11+
- PostgreSQL 16
- Redis 7
- Docker (optional)

### 1. Clone the repo

```bash
git clone https://github.com/your-org/gigshield.git
cd gigshield
```

### 2. Set up environment variables

```bash
cp .env.example .env
# Fill in: OPENWEATHERMAP_API_KEY, RAZORPAY_KEY, GUIDEWIRE_API_URL, DB_URL, REDIS_URL
```

### 3. Start with Docker Compose

```bash
docker compose up --build
```

Or run services individually:

```bash
# Backend API
cd backend && npm install && npm run dev

# Risk / Pricing / Fraud microservices
cd services/risk    && pip install -r requirements.txt && uvicorn main:app --port 8001
cd services/pricing && pip install -r requirements.txt && uvicorn main:app --port 8002
cd services/fraud   && pip install -r requirements.txt && uvicorn main:app --port 8003

# Mobile app
cd mobile && npm install && npx expo start

# Admin portal
cd web && npm install && npm run dev
```

### 4. Seed synthetic data

```bash
cd scripts && python generate_synthetic_data.py --workers 500 --city chennai
```

### 5. Run tests

```bash
# Backend
cd backend && npm test

# Python services
cd services && pytest --cov
```

---

## 📁 Project Structure

```
gigshield/
├── backend/                  # Node.js API gateway
│   ├── src/
│   │   ├── routes/           # REST endpoints
│   │   ├── middleware/        # Auth, rate limiting
│   │   └── integrations/     # Guidewire, Razorpay
│   └── package.json
├── services/
│   ├── risk/                 # FastAPI risk scoring microservice
│   ├── pricing/              # FastAPI pricing engine
│   └── fraud/                # FastAPI fraud detection service
├── mobile/                   # React Native (Expo) worker app
├── web/                      # React.js admin portal
├── scripts/
│   └── generate_synthetic_data.py
├── db/
│   └── schema.sql
├── docker-compose.yml
└── README.md
```

---

## 🔗 Guidewire Integration

GigShield acts as an **embedded intelligence layer** on top of Guidewire InsuranceSuite — extending it into the micro-insurance segment without requiring platform changes.

| Module | GigShield Usage |
|---|---|
| **PolicyCenter** | Auto-creates weekly micro-policies on worker enrollment; manages renewals |
| **BillingCenter** | Routes ₹140/week premium deductions via Razorpay |
| **ClaimCenter** | Files auto-approved payouts; escalates fraud-flagged claims to review queue |

---

## 🔮 What's Next

- [ ] XGBoost risk classifier trained on labelled incident data
- [ ] Cell tower triangulation (Tier 2 fraud) — full implementation
- [ ] Fraud ring detection via social graph + DBSCAN clustering
- [ ] IRDAI micro-insurance sandbox pilot — 100 real workers, Chennai
- [ ] Live Swiggy / Zomato platform API integration
- [ ] Kubernetes (EKS) for production-scale orchestration
- [ ] Expand to Tier 2 cities: Coimbatore, Madurai, Pune
- [ ] White-label risk scoring engine as Guidewire Marketplace component

---

## 👥 Team

Built with ☕ and a lot of weather API calls at **Guidewire Hackathon 2025**.

---

> *GigShield is not just a product — it is a trust infrastructure for India's gig economy, proving that enterprise insurance platforms can power worker-first, real-time, embedded coverage at scale.*
