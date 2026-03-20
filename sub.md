Persona

Name: Raj
Age: 25
Platform: Swiggy
Work: 10–12 hours/day
Location: Chennai
Monthly Income: ₹18,000/month (~₹4,500/week)

Environmental disruptions: Heavy rain, flooding, heatwaves
Social disruptions: Political rallies, elections, local strikes
Tech behavior: Mid-range Android, active on Instagram, Telegram, Facebook
Risk context: Unstable income, potential for system abuse, no existing safety net

Our insurance targets consistent workers like Raj — those who show up daily
and deserve protection when the environment fails them, not when they do.

---

Workflow

1. Worker Identification

Raj does not fill any form manually. The system identifies and
onboards eligible delivery partners automatically using behavioural
and environmental signals from synthetic Swiggy data (real driver
data is not publicly available — synthetic dataset of Chennai
delivery partner profiles built to replicate monsoon seasonality
and delivery density).

Eligibility is determined by the Risk Score engine before a policy
is issued.

---

2. Risk Score

Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)

Weights and tier thresholds are learned by a trained XGBoost classifier
on synthetic worker profiles, capturing non-linear relationships between
weather severity, worker behaviour, and income loss risk. The formula
above reflects the learned weight distribution.

Environmental Risk Score
a. Rain intensity (mm/hr)
b. Flood probability (zone-based)
c. Temperature (heatwaves)
d. Air Quality Index (AQI)
e. Wind speed (storm risk)

Operational Risk Score
a. Average working hours/day
b. Peak-hour dependency (lunch/dinner rush)
c. Delivery density in area
d. Order frequency drop during disruptions
e. Route type (high flood-prone roads)

Behavioural Risk Score (Synthetic dataset — Phase 1)
a. Movement pattern (continuous vs static)
b. Speed variation (real vs fake travel)
c. Stop frequency (traffic signals, rest stops)
d. Route consistency

Score maps to tiers:
Low / Medium / High / Extreme

---

3. Weekly Pricing Model

Weekly Premium = Base Premium × Risk Multiplier × Behaviour Modifier − Trust Discount

Base Premium is set at 2–5% of weekly income:
Raj earns ₹18,000/month ≈ ₹4,500/week → Base Premium = ₹150

Risk Multiplier
Low        → 0.8
Medium     → 1.0
High       → 1.3
Extreme    → 1.6

These multipliers are outputs of a regression model trained on
historical Chennai weather data and synthetic worker profiles —
not fixed lookup values. A worker in a flood-prone zone during
northeast monsoon season receives a dynamically higher multiplier
than the same profile in May.

Behaviour Modifier
Consistent worker  → 0.9
Normal             → 1.0
Irregular          → 1.2

Trust Discount
Parameters considered:
- History of claims
- Claim success rate
- Suspicious behaviour flags
- GPS anomaly history
- Micro-task verification success

High trust   → −20%
Medium trust → −10%
Low trust    →   0%

Raj's Case:
Base = ₹150 | Risk = High (1.3) | Behaviour = Consistent (0.9) | Trust = High (−20%)
₹150 × 1.3 × 0.9 = ₹175.5 → after trust discount → ₹140/week

Premium range: ₹90 (floor) to ₹360 (ceiling)
Validated via Monte Carlo simulation across 10,000 synthetic
Chennai weather weeks to confirm pool solvency.

---

4. Parametric Automation Flow

Primary flow is fully automatic — the worker never needs to
file a claim. A manual claim option exists for edge cases where
the automatic trigger did not fire but the worker was genuinely
affected (e.g. localised flooding not captured by OpenWeatherMap's
zone average). Manual claims go through the same Tier 1 + Tier 2
fraud checks before payout is approved.

Step 1: OpenWeatherMap polls Chennai weather every 15 minutes
Step 2: Rain intensity crosses threshold (>15mm/hr)
         → FastAPI trigger service fires automatically
Step 3: Fraud checks run in parallel (Tier 1):
         - ipapi.co confirms worker IP is in Chennai zone
         - Supabase session record confirms worker was active today
         - Browser geolocation matches claimed weather zone
         - Random Forest fraud classifier scores the claim
         - Isolation Forest flags statistical anomalies in
           claim pattern
Step 4: All checks pass
         → Guidewire PolicyCenter updates policy status to
           "payout triggered" via REST API
         → Guidewire ClaimCenter creates and closes the
           claim record automatically
         → Guidewire BillingCenter initiates payment
           instruction to Stripe
Step 5: Stripe releases payment to worker wallet
Step 6: Supabase Realtime pushes notification to worker's browser:
         "Heavy rain detected → ₹140 credited to your account"

Total time from trigger to payout: under 60 seconds

---

5. Fraud Detection Mechanism

Detection is split into three tiers by complexity.

Tier 1 — Automatic (runs on every claim trigger)

a. IP Geolocation (ipapi.co)
   - Worker's IP resolved to real-world city and zone
   - Cross-referenced against claimed weather zone
   - Mismatch → claim flagged instantly

b. Browser Geolocation + OpenWeatherMap Cross-check
   - Browser reports GPS coordinates via navigator.geolocation
   - FastAPI queries OpenWeatherMap for weather at those exact
     coordinates
   - If claimed location shows clear skies → claim rejected
   - If GPS zone and IP zone don't match → claim flagged

c. Session Fingerprinting (Supabase)
   - Device ID
   - Session login timestamps
   - Session duration
   - Activity patterns
   - Last known GPS location
   - Timezone verification
   - IP address (multiple users cannot share the same IP)

d. WebRTC Leak Detection
   - Browser exposes local IP (192.168.x.x) via WebRTC
   - Local IP tracked across sessions
   - Inconsistency between WebRTC IP and claimed location
     → claim flagged

e. Suspicious Location Change Flag
   - Any location change in the 7 days before a claim attempt
     is logged and scored as a risk signal

f. Random Forest Fraud Classifier
   - Trained on synthetic fraud/legitimate claim profiles
   - Input features: IP zone match, GPS deviation score,
     session duration, timezone consistency, WebRTC IP match,
     claim timing vs weather event
   - Output: fraud probability score 0–1
   - Claims above threshold → escalated to Tier 2

Tier 2 — Behavioural (runs on flagged claims)

a. Isolation Forest Anomaly Detection
   - Unsupervised ML — no labelled fraud examples needed
   - Flags statistical outliers in claim volume, timing,
     and location clustering automatically

b. Location Fingerprint Analysis
   - Past delivery locations
   - Order completion zones
   - Movement routes
   - Time patterns
   - Deviation from established fingerprint → elevated fraud score

c. Network Behaviour Analysis
   - FastAPI sends 3–5 rapid requests to the client
   - Checks: latency drift, jitter, packet timing
   - Checks: IP consistency, latency variation, routing changes
   - Real workers in bad weather show consistent network
     degradation patterns — spoofers do not

Tier 3 — Fraud Ring Detection (designed, Phase 3 build)

Target: coordinated groups spoofing the same weather zone

Social graph checks:
   - Are multiple users claiming from suspiciously similar
     GPS coordinates?
   - Same IP ranges across claimants?
   - Identical behaviour timing across accounts?

Environment cross-check:
   - Local news API verifies whether reported disruption
     matches real ground conditions in that zone

Clustering detection:
   - 50+ claims within 30 minutes from a 500m GPS radius
     but IPs resolving to different neighbourhoods
     → entire cluster flagged as coordinated fraud ring
   - DBSCAN clustering via NetworkX (Phase 3)

Physics challenge (optional, mobile web only):
   - For highest-risk flagged users only
   - Camera snapshot of surroundings
   - Gyroscope + accelerometer liveness check
   - Not part of standard claim flow

---

6. UX Flow

Platform: React Web Application
Accessible via any browser — no app install required.
Can be embedded into existing platforms as an iframe or webview.
Chosen over native mobile to reduce friction for workers already
using Swiggy's app daily, and to speed up demo iteration.

Worker View (Raj's Dashboard)

+----------------------------------+
|  GigShield                       |
|  Hey Raj, you are protected ✓    |
|                                  |
|  This week's premium: ₹140       |
|  Auto-deducted every Sunday      |
|                                  |
|  Current conditions: Chennai     |
|  Rain: 18mm/hr ⚠ Above threshold |
|                                  |
|  → Payout triggered automatically|
|  ₹140 credited to your account   |
|                                  |
|  [File Manual Claim]             |
+----------------------------------+

Primary flow: automatic, no worker action needed.
Manual claim option available for edge cases only.
Manual claims go through identical fraud checks before approval.

Admin View (Insurer Dashboard)

- Active policies and premium pool balance
- Live weather trigger log
- Fraud flag queue with tier classification
- Payout history and loss ratio
- Worker risk score breakdown

---

7. Tech Stack

Layer            Technology              Purpose
---------------------------------------------------------
Frontend         React.js                Worker dashboard +
                                         admin portal
Backend          FastAPI (Python)        Risk scoring, pricing,
                                         fraud microservices
ML               scikit-learn /          XGBoost risk classifier,
                 XGBoost                 regression pricing model,
                                         Random Forest fraud
                                         scorer, Isolation Forest
                                         anomaly detection
Database         Supabase (PostgreSQL)   Worker profiles,
                                         policies, claims,
                                         audit logs
Weather          OpenWeatherMap API      Live rain, temperature,
                                         AQI for Chennai
IP Geolocation   ipapi.co               IP-to-location
                                         fraud check
Payments         Stripe (sandbox)        Weekly premium
                                         auto-deduction
Real-time        Supabase Realtime       Live payout
                                         notifications
Guidewire        InsuranceSuite APIs     PolicyCenter — policy
                                         status updates on
                                         payout trigger
                                         ClaimCenter — auto
                                         claim creation and
                                         closure
                                         BillingCenter —
                                         payment instruction
                                         to Stripe
Entire demo stack cost: ₹0
All services run on free/sandbox tiers.

Entire demo stack cost: ₹0
All services run on free/sandbox tiers.
