# GigShield — Dynamic Micro-Insurance for Gig Economy Workers

---

## Inspiration

Every monsoon season in Chennai, thousands of delivery partners like **Raj** keep riding through flooded streets — not because they want to, but because missing a shift means missing rent.

We kept asking ourselves one question: _why does a 25-year-old delivery partner earning ₹18,000/month have less financial protection than someone earning ₹5 lakh/month?_

Traditional insurance was never designed for the gig economy. Premiums are flat, forms are manual, and claims take weeks. Meanwhile, Raj faces a new risk **every single day** — heavy rain, heatwaves, political rallies, flooded roads. His income is volatile. His exposure is real. His safety net is essentially zero.

That gap is what inspired **GigShield**: a micro-insurance system that is as dynamic as the work it protects.

---

## What it does

GigShield delivers **contextual, weekly micro-insurance** for gig delivery workers — automatically priced, automatically triggered, and fraud-resistant by design.

- 🔍 **Auto-identifies** eligible workers using behavioural and environmental signals — no forms, no manual onboarding
- 📊 **Scores risk in real time** across three dimensions: Environmental, Operational, and Behavioural
- 💰 **Calculates a personalised weekly premium** using the formula:

$$
\text{Weekly Premium} = P_{\text{base}} \times R_{\text{multiplier}} \times B_{\text{modifier}} - D_{\text{trust}}
$$

Where:
- \\( P_{\text{base}} \\) = base premium (2–5% of weekly income, default ₹150)
- \\( R_{\text{multiplier}} \\) = risk tier multiplier ∈ {0.8, 1.0, 1.3, 1.6}
- \\( B_{\text{modifier}} \\) = behaviour modifier ∈ {0.9, 1.0, 1.2}
- \\( D_{\text{trust}} \\) = trust discount up to 20%

For our persona **Raj** (High risk, Consistent worker, High trust):

$$
\text{Premium} = ₹150 \times 1.3 \times 0.9 - 20\% = ₹\mathbf{140/week}
$$

- ⚡ **Triggers automatic payouts** when verified environmental events (e.g. heavy rain) cross a threshold — in under 60 seconds
- 🛡️ **Detects fraud** using a multi-signal system: IP deduplication, GPS consistency, session fingerprinting, timezone verification, and cell tower triangulation
- 🔗 **Integrates with Guidewire** PolicyCenter, BillingCenter, and ClaimCenter via REST APIs

---

## How we built it

We broke the system into three independently deployable microservices, connected by a shared PostgreSQL database and a Redis cache layer.

### Risk Scoring Engine
Built in **Python (FastAPI)**. Each risk dimension produces a normalised sub-score:

$$
\text{Risk Score} = w_e \cdot S_{\text{env}} + w_o \cdot S_{\text{ops}} + w_b \cdot S_{\text{behav}}
$$

Where \\( w_e, w_o, w_b \\) are configurable weights (default: 0.4, 0.3, 0.3). For the hackathon demo, scoring uses **rule-based thresholds** — the architecture is designed to swap in an XGBoost classifier post-hackathon once labelled incident data is available.

### Pricing Engine
Also in **Python (FastAPI)**. The formula above is evaluated server-side on every weekly renewal cycle and on any significant risk score change. Premiums are floored at ₹90 and capped at ₹360 to remain within the worker's affordability band:

$$
P_{\text{final}} = \text{clip}\left(P_{\text{base}} \times R \times B - D,\ ₹90,\ ₹360\right)
$$

### Fraud Detection Service
Tier 1 (fully implemented): IP deduplication, GPS delta validation, session fingerprinting via Redis, and timezone cross-check.

Tier 2 (partial): WebRTC local IP leak capture and cell tower signal correlation.

The **Trust Score** \\( T \in [0, 1] \\) is computed as:

$$
T = 1 - \frac{\text{fraud\_flags} \cdot \alpha + \text{gps\_anomalies} \cdot \beta + \text{failed\_verifications} \cdot \gamma}{N_{\text{total\_checks}}}
$$

Where \\( \alpha, \beta, \gamma \\) are penalty weights tuned empirically on our synthetic dataset.

### Frontend
- **React Native (Expo)** — worker-facing mobile dashboard
- **React.js + Tailwind CSS** — insurer admin portal with risk heatmap and claim queue
- **Socket.io** — real-time payout push notifications

### Data
Swiggy's driver data is not publicly available, so we generated a **synthetic dataset of 500 Chennai delivery partner profiles** using Python's Faker library, calibrated to realistic Chennai delivery density, income ranges, and monsoon seasonality.

### Guidewire Integration
We used Guidewire's **PolicyCenter** and **BillingCenter** REST APIs to:
1. Auto-create weekly micro-policies on worker enrollment
2. Route premium deductions through Razorpay → BillingCenter
3. File and close claims via ClaimCenter on verified payout events

---

## Challenges we ran into

**1. Real data doesn't exist — synthetic data has to be honest**
Swiggy and Zomato don't publish driver-level GPS or income data. Building a convincing synthetic dataset that behaves like real Chennai delivery patterns (monsoon seasonality, peak lunch/dinner density, flood-prone road clusters) took far longer than expected.

**2. Fraud detection without being invasive**
The line between _fraud prevention_ and _surveillance_ is thin, especially for low-income workers who are the most vulnerable to being falsely flagged. Every signal we added to the fraud layer had to be justified as proportionate. We deliberately left camera-based physics checks as a future feature — forcing that on a worker mid-claim felt wrong for the MVP.

**3. Pricing that is fair AND solvent**
A premium of ₹140/week sounds small, but calibrating the multipliers so that:
- The product is affordable for workers
- The pool remains solvent across a worst-case monsoon event cluster
- The Trust Discount doesn't create adverse selection

...required building a simple **Monte Carlo simulation** to stress-test the premium pool across 10,000 simulated weather weeks before locking the multiplier values.

$$
\text{Pool Solvency} = \sum_{i=1}^{N} P_i - \mathbb{E}\left[\sum_{j \in \text{claims}} C_j\right] \geq 0
$$

**4. Scoping honestly**
Early versions of the document made it look like everything from Kubernetes orchestration to TensorFlow Lite on-device ML was fully live. It wasn't. Deciding to clearly label components `[DEMO]`, `[PARTIAL]`, and `[FUTURE]` was a deliberate choice — we think credibility matters more than appearing impressive on paper.

---

## Accomplishments that we're proud of

- ✅ **End-to-end working demo**: Worker onboarding → risk score → ₹140 premium → rain trigger → payout in under 60 seconds — all on synthetic data but fully connected
- ✅ **Guidewire integration**: Live PolicyCenter and BillingCenter API calls within the demo flow — not mocked
- ✅ **Honest architecture**: Clean separation between what's built and what's designed, so judges can evaluate both the demo and the vision independently
- ✅ **The pricing formula actually works**: Monte Carlo validation showed the premium pool stays solvent even in a simulated 3-week consecutive flood event for Chennai
- ✅ **Fraud detection that respects the user**: Tier 1 checks are invisible to legitimate workers — they only surface for anomalous behaviour

---

## What we learned

- **Micro-insurance is a systems problem, not just a product problem.** Pricing, fraud, UX, and claims processing are deeply interdependent. Getting the pricing wrong breaks fraud incentives. Getting fraud wrong breaks trust. Getting UX wrong breaks adoption.

- **Guidewire's platform is more flexible than it looks from the outside.** The PolicyCenter and BillingCenter APIs are well-documented and composable — we were able to model a ₹140/week micro-policy using the same primitives designed for ₹1L annual premiums.

- **Synthetic data forces you to understand the real thing deeply.** You can't generate realistic Chennai delivery data without first understanding monsoon flood zones, peak order windows, and road network topology. The data generation process became one of our best research exercises.

- **Scoping is a feature.** The most important engineering decision we made was deciding what _not_ to build for the demo. A credible, working MVP beats an overengineered deck every time.

---

## What's next for GigShield

**Short-term (0–3 months)**
- Replace rule-based risk scoring with a trained **XGBoost classifier** once labelled incident data is available from insurer partnerships
- Implement Tier 2 fraud checks: cell tower triangulation and WebRTC leak detection
- Pilot with a small cohort of real delivery partners in Chennai (targeting 100 workers via IRDAI micro-insurance sandbox)

**Medium-term (3–12 months)**
- Integrate with Swiggy / Zomato platform APIs directly for live worker data ingestion
- Expand to **Tier 2 cities**: Coimbatore, Madurai, Pune — where flood and heat risk is high but insurance penetration is near zero
- Deploy **Kubernetes (EKS)** orchestration to handle 10,000+ concurrent worker sessions
- Add fraud ring detection: social graph clustering via NetworkX + DBSCAN

**Long-term**
- Extend the GigShield framework to **urban cab drivers, construction day labourers, and domestic workers** — any workforce that is platform-adjacent, income-volatile, and risk-exposed
- White-label the risk scoring + pricing engine as a **Guidewire Marketplace component** for other insurers to deploy micro-insurance products on top of their existing InsuranceSuite implementation

> _GigShield is not just a product — it is a trust infrastructure for India's gig economy, proving that enterprise insurance platforms can power worker-first, real-time, embedded coverage at scale._
