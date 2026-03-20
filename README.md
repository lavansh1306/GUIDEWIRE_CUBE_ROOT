## Inspiration

Every monsoon season in Chennai, thousands of delivery partners like **Raj** keep riding through flooded streets — not because they want to, but because missing a shift means missing rent.

We kept asking ourselves one question:

> _Why does a 25-year-old delivery partner earning ₹18,000/month have less financial protection than someone earning ₹5 lakh/month?_

Traditional insurance was never designed for the gig economy. Premiums are flat, forms are manual, and claims take days. Meanwhile, Raj faces a new risk **every single day** — heavy rain, heatwaves, political rallies, flooded roads. His income is volatile. His exposure is real. His safety net is essentially zero.

That gap is what inspired **GigShield**: a micro-insurance system that is as dynamic as the work it protects.

## What it does

GigShield delivers **contextual, weekly micro-insurance** for gig delivery workers — automatically priced, automatically triggered, and fraud-resistant by design.

- 🔍 **Auto-identifies** eligible workers using behavioural and environmental signals — no forms, no manual onboarding
- 📊 **Scores risk in real time** across three dimensions: Environmental, Operational, and Behavioural
- 💰 **Calculates a personalised weekly premium** using the formula:

$$\text{Weekly Premium} = P_{\text{base}} \times R_{\text{multiplier}} \times B_{\text{modifier}} - D_{\text{trust}}$$

For our persona **Raj** (High risk, Consistent worker, High trust):

$$\text{Premium} = ₹150 \times 1.3 \times 0.9 - 20\% = ₹\textbf{140/week}$$

- ⚡ **Parametric auto-payout** — when OpenWeatherMap detects rain above the threshold in Raj's zone, the system triggers the payout automatically with **zero action required from the worker.** No claim form. No phone call. No waiting.
- 🛡️ **Detects fraud** via multi-signal checks: IP geolocation (ipapi.co), GPS consistency, session fingerprinting, timezone verification, and cell tower triangulation
- 🔗 **Integrates with Guidewire** PolicyCenter, BillingCenter, and ClaimCenter via REST APIs

## How we built it

### Platform choice — why web over mobile

We chose a **React web app** over a native mobile app for this hackathon build. Delivery partners like Raj already use Swiggy's app daily — adding another app creates friction. A web app runs in any browser, requires no install, and can be embedded directly into existing platforms as a lightweight iframe or webview. It also makes the demo faster to iterate on and easier for judges to access without a device.

### Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | **React.js** | Worker dashboard + admin portal |
| Backend | **FastAPI (Python)** | Risk scoring, pricing, and fraud microservices |
| Database | **Supabase (PostgreSQL)** | Worker profiles, policies, claims, audit logs |
| Weather | **OpenWeatherMap API** | Live rain, temperature, AQI for Chennai |
| IP Geolocation | **ipapi.co** | IP-to-location fraud check |
| Payments | **Stripe** | Weekly premium auto-deduction |
| Real-time | **Supabase Realtime** | Live payout notifications to worker dashboard |
| Guidewire | **InsuranceSuite APIs** | PolicyCenter + BillingCenter integration |

### Parametric Automation Flow

This is the core of GigShield — **the worker never has to file a claim.**

```
1. OpenWeatherMap polls Chennai weather every 15 minutes
2. Rain intensity crosses threshold (e.g. > 15mm/hr)
   → FastAPI trigger service fires automatically
3. Fraud service runs Tier 1 checks in parallel:
   - ipapi.co confirms worker IP is in Chennai zone
   - Supabase session record confirms worker was active today
   - GPS last-known location matches claim zone
4. All checks pass → Supabase updates policy status to "payout triggered"
5. Stripe releases payment to worker wallet
6. Supabase Realtime pushes notification to worker's browser:
   "Heavy rain detected → ₹140 credited to your account"
Total time from trigger to payout: < 60 seconds
```

No worker action. No form. No waiting.

### Risk Scoring Engine (FastAPI)

$$\text{Risk Score} = w_e \cdot S_{\text{env}} + w_o \cdot S_{\text{ops}} + w_b \cdot S_{\text{behav}}$$

Where \\(w_e = 0.4\\), \\(w_o = 0.3\\), \\(w_b = 0.3\\). Maps to Low / Medium / High / Extreme tiers via rule-based thresholds.

### Pricing Engine (FastAPI)

$$P_{\text{final}} = \text{clip}(P_{\text{base}} \times R \times B - D,\ ₹90,\ ₹360)$$

Multiplier values validated via Monte Carlo simulation across 10,000 simulated weather weeks to confirm pool solvency.

### Fraud Detection (FastAPI + Supabase + ipapi.co)

$$T = 1 - \frac{\alpha \cdot f_{\text{flags}} + \beta \cdot f_{\text{gps}} + \gamma \cdot f_{\text{verify}}}{N_{\text{checks}}}$$

- **Tier 1:** ipapi.co IP geolocation, GPS consistency, Supabase session fingerprint, timezone check
- **Tier 2:** Cell tower triangulation, WebRTC leak detection
- **Tier 3 (designed):** Social graph fraud ring detection via NetworkX + DBSCAN

### Data
Swiggy's driver data is not public — we use a **synthetic dataset of 500 Chennai delivery partner profiles** built to replicate real monsoon seasonality and delivery density.

## Challenges we ran into

**Parametric trigger calibration** — setting the right rain threshold matters enormously. Too low and the pool pays out on drizzle. Too high and workers in genuine floods get nothing. We ran Monte Carlo simulations across 10,000 historical Chennai weather weeks to find thresholds that are both fair to workers and keep the pool solvent.

**Fraud without GPS spoofing protection** — parametric payouts are powerful but exploitable. A worker in Bengaluru could claim a Chennai rainstorm. ipapi.co cross-referencing the IP location against the weather zone, combined with Supabase session history, closes this gap without requiring the worker to do anything extra.

**Supabase Realtime for instant UX** — getting sub-second payout notifications to the worker's browser required careful Supabase channel design so the trigger pipeline didn't create a notification backlog under load.

**Honest scoping** — clearly separating what is wired end-to-end from what is designed-but-not-built matters as much as the technical work itself.

## Accomplishments that we're proud of

- **Zero-action parametric payout flow** — from weather trigger to Stripe credit in under 60 seconds, with no worker input at all
- **Actuarially validated pricing** — Monte Carlo solvency testing across 10,000 weather weeks, not just a formula on paper
- **Fraud-resistant by design** — ipapi.co + Supabase session checks make spoofing the parametric trigger genuinely hard without adding friction for legitimate workers
- **Concrete Guidewire integration** — PolicyCenter and BillingCenter mapped to real API workflows, not vague mentions

## What we learned

- **Parametric insurance is only as good as its trigger data.** OpenWeatherMap's granularity at the zone level (not just city level) was critical — city-level averages would miss hyperlocal flooding.
- **Supabase dramatically simplifies the real-time layer.** What would have taken a separate WebSocket server + Redis pub/sub is a few lines of Supabase Realtime configuration.
- **ipapi.co is surprisingly powerful for fraud prevention.** A free-tier IP geolocation check eliminates a large class of cross-region spoofing attacks with zero added UX friction.
- **Scoping is a feature.** A working parametric trigger with honest fraud checks beats a 20-service architecture that exists only on paper.

## What's next for GigShield

- Replace rule-based scoring with a trained **XGBoost classifier** on real incident data
- IRDAI micro-insurance sandbox pilot — 100 real delivery partners in Chennai
- Live Swiggy / Zomato platform API integration
- Expand to Tier 2 cities: Coimbatore, Madurai, Pune
- Full fraud ring detection via social graph clustering + network behaviour ML
- White-label the engine as a **Guidewire Marketplace component** for other insurers

---

🔗 **GitHub Repository:** https://github.com/your-org/gigshield _(link to be updated before submission deadline)_

🎥 **Demo Video:** https://youtu.be/your-demo-link _(link to be updated before submission deadline)_
