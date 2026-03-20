# 🛡️ GigShield
> **Dynamic Micro-Insurance for Gig Economy Workers**
> *Real-time risk scoring · Personalised weekly premiums · Fraud-resistant auto-payouts*

---

## 💡 Inspiration

Every monsoon season in Chennai, thousands of delivery partners like **Raj** keep riding through flooded streets — not because they want to, but because missing a shift means missing rent.

We kept asking ourselves one question:

> _Why does a 25-year-old delivery partner earning ₹18,000/month have less financial protection than someone earning ₹5 lakh/month?_

Traditional insurance was never designed for the gig economy. Premiums are flat, forms are manual, and claims take weeks. Meanwhile, Raj faces a new risk **every single day** — heavy rain, heatwaves, political rallies, flooded roads. His income is volatile. His exposure is real. His safety net is essentially zero.

That gap is what inspired **GigShield**: a micro-insurance system that is as dynamic as the work it protects.

---

## ✨ What It Does

GigShield delivers **contextual, weekly micro-insurance** for gig delivery workers — automatically priced, automatically triggered, and fraud-resistant by design.

- 🔍 **Auto-identifies** eligible workers using behavioural and environmental signals — no forms, no manual onboarding
- 📊 **Scores risk in real time** across three dimensions: Environmental, Operational, and Behavioural
- 💰 **Calculates a personalised weekly premium** using the formula:

$$\text{Weekly Premium} = P_{\text{base}} \times R_{\text{multiplier}} \times B_{\text{modifier}} - D_{\text{trust}}$$

Where:
- $P_{\text{base}}$ = base premium (2–5% of weekly income, default ₹150)
- $R_{\text{multiplier}}$ = risk tier multiplier ∈ {0.8, 1.0, 1.3, 1.6}
- $B_{\text{modifier}}$ = behaviour modifier ∈ {0.9, 1.0, 1.2}
- $D_{\text{trust}}$ = trust discount up to 20%

For our persona **Raj** (High risk, Consistent worker, High trust history):

$$\text{Premium} = ₹150 \times 1.3 \times 0.9 - 20\% = ₹\textbf{140/week}$$

- ⚡ **Triggers automatic payouts** when verified environmental events cross a threshold — in under 60 seconds
- 🛡️ **Detects fraud** via multi-signal checks: IP deduplication, GPS consistency, session fingerprinting, timezone verification, and cell tower triangulation
- 🔗 **Integrates with Guidewire** PolicyCenter, BillingCenter, and ClaimCenter via REST APIs

---

## 🏗️ How We Plan to Build It

GigShield is designed as three independently deployable microservices connected by a shared PostgreSQL database and Redis cache layer.

### Risk Scoring Engine
Built in **Python (FastAPI)**. Each risk dimension produces a normalised sub-score combined as:

$$\text{Risk Score} = w_e \cdot S_{\text{env}} + w_o \cdot S_{\text{ops}} + w_b \cdot S_{\text{behav}}$$

Where $w_e = 0.4$, $w_o = 0.3$, $w_b = 0.3$ (configurable). The composite score maps to a risk tier:

| Score Range | Tier | Multiplier |
|---|---|---|
| 0.0 – 0.3 | Low | 0.8 |
| 0.3 – 0.5 | Medium | 1.0 |
| 0.5 – 0.7 | High | 1.3 |
| 0.7 – 1.0 | Extreme | 1.6 |

Initial implementation uses **rule-based thresholds**. The architecture is designed to swap in an XGBoost classifier once labelled incident data is available.

### Pricing Engine
**Python (FastAPI)**. Premiums are floored at ₹90 and capped at ₹360 to stay within the worker's affordability range:

$$P_{\text{final}} = \text{clip}\!\left(P_{\text{base}} \times R \times B - D,\ ₹90,\ ₹360\right)$$

Before locking multiplier values, we validated pool solvency using a Monte Carlo simulation across 10,000 simulated weather weeks:

$$\text{Pool Solvency} = \sum_{i=1}^{N} P_i - \mathbb{E}\!\left[\sum_{j \in \text{claims}} C_j\right] \geq 0$$

The pool remained solvent across a simulated 3-week consecutive flood scenario for Chennai.

### Fraud Detection Service
**Python (FastAPI)** with Redis for session state. Worker trust score:

$$T = 1 - \frac{\alpha \cdot f_{\text{flags}} + \beta \cdot f_{\text{gps}} + \gamma \cdot f_{\text{verify}}}{N_{\text{checks}}}$$

Fraud checks are layered in tiers:

**Tier 1 — Core checks:**
- IP address deduplication
- GPS consistency validation
- Session fingerprinting (device ID, timestamps, activity patterns)
- Timezone cross-verification

**Tier 2 — Advanced checks:**
- Cell tower triangulation (signal strength as GPS-independent proxy)
- WebRTC local IP leak detection
- Location fingerprinting against historical delivery zones

**Tier 3 — Fraud ring detection:**
- Social graph clustering (NetworkX + DBSCAN) to detect coordinated mass-claim attacks
- Network behaviour analysis: static spoofers produce unnaturally stable latency fingerprints

### Frontend
- **React Native (Expo)** — worker-facing mobile dashboard
- **React.js + Tailwind CSS** — insurer admin portal
- **Socket.io** — real-time payout push notifications

### Data
Since Swiggy's driver data is not publicly available, the demo will use a **synthetic dataset of 500 Chennai delivery partner profiles** generated to replicate realistic income ranges, delivery density, and monsoon seasonality.

---

## ⚡ Challenges We Anticipate

**Pricing that is fair AND solvent** — Calibrating multipliers so the product is affordable for low-income workers while keeping the premium pool solvent across worst-case weather clusters is a genuine actuarial problem, not just a product design one.

**Fraud detection without being invasive** — The line between fraud prevention and surveillance is thin, especially for workers who are most vulnerable to being falsely flagged. Every signal needs to be proportionate.

**No real data** — Building a convincing synthetic dataset that behaves like actual Chennai delivery patterns requires deeply understanding monsoon flood zones, peak order windows, and road topology before writing a single line of data generation code.

**Honest scoping** — Clearly separating what is built for the demo from what is architecturally designed but not yet wired is as important as the technical work itself.

---

## 🏆 Accomplishments We're Proud Of

- **The pricing formula is actuarially validated** — Monte Carlo solvency testing across 10,000 weather weeks, not just a formula on paper
- **Fraud detection is proportionate** — Tier 1 checks are invisible to legitimate workers and only surface on anomalous behaviour
- **Honest MVP scope** — We clearly separate demo components from future architecture, so the submission is credible rather than just impressive-looking
- **Guidewire integration is concrete** — PolicyCenter, BillingCenter, and ClaimCenter mapped to specific workflows, not vague API mentions

---

## 📚 What We Learned

- **Micro-insurance is a systems problem, not just a product problem.** Pricing, fraud, UX, and claims are deeply interdependent — getting one wrong breaks the others.
- **Guidewire's platform is more composable than it looks.** The same API primitives designed for ₹1L annual premiums can model a ₹140/week micro-policy.
- **Synthetic data forces you to understand the real thing.** You can't generate realistic Chennai delivery data without first understanding the city's flood zones, peak order windows, and road network.
- **Scoping is a feature.** A credible, honest MVP beats an overengineered claim every time.

---

## 🔮 What's Next for GigShield

- Replace rule-based scoring with a trained **XGBoost classifier** on real incident data
- IRDAI micro-insurance sandbox pilot — 100 real delivery partners in Chennai
- Live integration with Swiggy / Zomato platform APIs
- Expand to Tier 2 cities: Coimbatore, Madurai, Pune
- Full fraud ring detection: social graph clustering + network behaviour ML
- White-label the risk + pricing engine as a **Guidewire Marketplace component** for other insurers

---

> _GigShield is not just a product — it is a trust infrastructure for India's gig economy, proving that enterprise insurance platforms can power worker-first, real-time, embedded coverage at scale._
