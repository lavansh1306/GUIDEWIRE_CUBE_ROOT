# 🛡️ RAKSHA X

### Insurance for People Working in the Gig Economy

![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)

![TailwindCSS](https://img.shields.io/badge/tailwindcss-%2338B2AC.svg?style=for-the-badge&logo=tailwind-css&logoColor=white)

![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=for-the-badge&logo=fastapi)

![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)

![Postgres](https://img.shields.io/badge/postgres-%23316192.svg?style=for-the-badge&logo=postgresql&logoColor=white)

![Stripe](https://img.shields.io/badge/Stripe-5469d4?style=for-the-badge&logo=stripe&logoColor=ffffff)

![XGBoost](https://img.shields.io/badge/ML-XGBoost-red?style=for-the-badge)

![Guidewire](https://img.shields.io/badge/Integration-Guidewire-orange?style=for-the-badge)

---

## 👤 The Person We Are Helping

**Name:** Raj | **Age:** 25 | **Platform:** Swiggy *Location:** Chennai

**Monthly Income:** ₹18,000 per month (~₹4,500 per week) | **Work:** 10 to 12 hours per day

There are things that can disrupt Rajs work. For example heavy rain, flooding and heatwaves can be a problem. Sometimes things like rallies, elections and local strikes can also cause problems. Raj uses a -range Android phone and is active on Instagram, Telegram and Facebook. RAKSHA X insurance is for people like Raj who work hard every day and need protection when things go wrong not because they made a mistake.

---

## ⚙️ How It Works

### 1️⃣ Finding the Right Workers

Raj does not have to fill out any forms. The RAKSHA X system. Signs up eligible delivery partners automatically. It uses information from Swiggy and other sources to do this. Since we do not have data we made a fake dataset of delivery partner profiles in Chennai. This dataset includes information about the monsoon season. How many deliveries are made. The Risk Score engine decides who is eligible for insurance before a policy is given.

---

### 2️⃣ Calculating the Risk Score

```

Risk Score = (0.4 × Environmental) + (0.3 × Operational) + (0.3 × Behavioural)

```

We use a tool called XGBoost to learn the weights and thresholds for the Risk Score. This tool looks at worker profiles and finds the relationships between weather, worker behavior and the risk of losing income. The Risk Score can be Low, Medium, High or Extreme.

---

#### 🌧️ Environmental Risk

We look at things like:

- How much rain is falling

- If there is a chance of flooding

- How hot it's

- The air quality

- How windy it is

#### 📦 Operational Risk

We look at things, like:

- How many hours Raj works per day

- If Raj works during peak hours like lunch or dinner time when many people are ordering food
