# 🍔 QuickBite Data Analysis Project

> **A data analytics case study** investigating QuickBite's business crisis (Jun–Sep 2025) using the **Google Data Analytics framework** — Ask, Prepare, Process, Analyze, Share, Act.

## 🌐 Live Dashboard
[📊 View Interactive Power BI Dashboard](https://app.powerbi.com/links/5DRncCm1-4?ctid=d9a9308d-26d9-43d6-9800-dd669921a312&pbi_source=linkShare)
---

## 📌 Project Overview

QuickBite, a food delivery platform, experienced a significant operational and business crisis between **June and September 2025**. This project conducts a structured end-to-end data analysis to:

- Quantify the **impact of the crisis** on orders, revenue, cancellations, and customer loyalty
- Identify **root causes** and **affected segments** (cities, restaurants, customers)
- Derive **actionable recommendations** to recover and rebuild trust

This project follows the **6-phase Google Data Analytics process** as a guiding framework.

---

## 🗂️ Project Structure

```
QuickBite_data_analysis/
│
├── data/
│   ├── raw/                        # Original datasets (orders, customers, reviews)
│   └── processed/                  # Cleaned and transformed CSV files
│
├── Analysis/
│   ├── 01_QuickBite_data_analysis.ipynb     # Phase 3 — Process and Phase 4 — Primary Analysis (Q1–Q10)
│   └── 02_QuickBite_Dashboard.pbix    # Power BI dashboard file  Phase 4 — Secondary Analysis
│
├── docs/
│   └── Primary_and_Secondary_Analysis.pdf  # Analysis brief (Codebasics)
│
└── README.md
```

---

## 📊 Analysis Framework — Google Data Analytics (6 Phases)

### Phase 1 — ASK 🎯
**Define the business problem and key questions.**

The business is experiencing a crisis. Key stakeholder questions include:

**Primary Analysis (Based on Available Data):**
1. **Monthly Orders** — How severe is the order decline from pre-crisis (Jan–May 2025) to crisis (Jun–Sep 2025)?
2. **City-Level Decline** — Which top 5 city groups saw the highest percentage drop in orders?
3. **Restaurant Impact** — Among restaurants with ≥50 pre-crisis orders, which top 10 saw the largest percentage decline?
4. **Cancellation Analysis** — What is the cancellation rate trend, and which cities are most affected?
5. **Delivery SLA** — Did average delivery time worsen significantly during the crisis?
6. **Ratings Fluctuation** — Which months saw the sharpest drop in average customer ratings?
7. **Sentiment Insights** — What are the most frequent negative keywords in crisis-period reviews?
8. **Revenue Impact** — What is the estimated revenue loss (subtotal, discount, delivery fee)?
9. **Loyalty Impact** — How many loyal customers (5+ pre-crisis orders) stopped ordering during the crisis?
10. **Customer Lifetime Decline** — Which top 5% high-value customers showed the largest drop? What patterns do they share?

**Secondary Analysis (Requires Additional Research):**
1. How does QuickBite's crisis compare to competitors (Swiggy, Zomato) in the same period?
2. What external factors (ad prices, seasonal effects) contributed to CAC tripling?
3. Which strategies (cashbacks, partnerships, food safety audits) could rebuild trust most effectively?
4. Which restaurant types (cloud kitchens vs dine-in, small vs large brands) are most at risk of churning?
5. Which lapsed customers show the highest probability of returning with the right incentives?

**Extra Detail Questions:**
- Which Tier-1/Tier-2 cities show the highest risk of long-term demand loss?
- Did customers shift to lower-value "survival orders" during the crisis?
- Do spikes in negative reviews align with the delivery outage period?

---

### Phase 2 — PREPARE 📁
**Collect and understand the data.**

| Dataset | Description | Period Covered |
|---|---|---|
| `orders.csv` | Order-level data (subtotal, discount, delivery fee, status) | Jan 2025 – Sep 2025 |
| `customers.csv` | Customer profiles and segment info | — |
| `restaurants.csv` | Restaurant metadata (type, city, category) | — |
| `reviews.csv` | Customer review text and ratings | Jan 2025 – Sep 2025 |
| `delivery_logs.csv` | Delivery timestamps and SLA data | Jan 2025 – Sep 2025 |

**Data credibility checks (ROCCC):**
- **Reliable** — Internal platform data
- **Original** — First-party source
- **Comprehensive** — Covers all orders, customers, and reviews
- **Current** — Includes crisis period (Jun–Sep 2025)
- **Cited** — Provided by Codebasics.io

---

### Phase 3 — PROCESS 🔧
**Clean, transform, and validate the data.**

Tools used: `Python (pandas)` · `Power Query (Power BI)`

Key cleaning steps:
- Remove duplicate order records
- Handle missing values in ratings and review text
- Standardize city names and restaurant categories
- Create a `phase` column: `pre_crisis` (Jan–May 2025) vs `crisis` (Jun–Sep 2025)
- Parse and tokenize review text for sentiment analysis
- Validate delivery timestamps for SLA calculations

```python
import pandas as pd

df = pd.read_csv("data/raw/orders.csv", parse_dates=["order_date"])

# Remove duplicates
df.drop_duplicates(inplace=True)

# Add crisis phase label
df["phase"] = df["order_date"].apply(
    lambda x: "pre_crisis" if x < pd.Timestamp("2025-06-01") else "crisis"
)

# Fill missing ratings with median
df["rating"].fillna(df["rating"].median(), inplace=True)

df.to_csv("data/processed/orders_clean.csv", index=False)
print(df["phase"].value_counts())
```

---

### Phase 4 — ANALYZE 🔍
**Perform analysis to answer the key questions.**

#### Q1 — Monthly Order Decline

```python
monthly = df.groupby(df["order_date"].dt.to_period("M"))["order_id"].count().reset_index()
monthly.columns = ["month", "total_orders"]
monthly.plot(x="month", y="total_orders", title="Monthly Orders — Pre-Crisis vs Crisis")
```

#### Q2 — Top 5 Cities with Highest Order Decline

```python
city_phase = df.groupby(["city", "phase"])["order_id"].count().unstack()
city_phase["pct_decline"] = (
    (city_phase["pre_crisis"] - city_phase["crisis"]) / city_phase["pre_crisis"] * 100
)
top5_cities = city_phase.nlargest(5, "pct_decline")[["pct_decline"]]
print(top5_cities)
```

#### Q4 — Cancellation Rate by Phase & City

```python
df["is_cancelled"] = df["status"] == "Cancelled"
cancel_rate = (
    df.groupby(["city", "phase"])["is_cancelled"]
    .mean()
    .mul(100)
    .round(2)
    .reset_index()
)
cancel_rate.columns = ["city", "phase", "cancellation_rate_%"]
print(cancel_rate.sort_values("cancellation_rate_%", ascending=False).head(10))
```

#### Q7 — Sentiment Word Cloud (Power BI)

- Load `reviews.csv` into Power BI via **Get Data**
- Filter to `phase = crisis` using Power Query
- Add the **Word Cloud** visual from AppSource
- Set the **Values** field to `review_text`
- Configure stop words to filter out common filler words
- Publish the visual to the dashboard

#### Q8 — Revenue Impact

```python
df["revenue"] = df["subtotal"] - df["discount"] + df["delivery_fee"]
revenue_by_phase = df.groupby("phase")["revenue"].sum()
revenue_loss = revenue_by_phase["pre_crisis"] - revenue_by_phase["crisis"]
print(f"Estimated Revenue Loss: ₹{revenue_loss:,.0f}")
```

#### Q9 — Loyalty Impact

```python
# Loyal customers: 5+ pre-crisis orders
pre = df[df["phase"] == "pre_crisis"]
crisis = df[df["phase"] == "crisis"]

loyal = pre.groupby("customer_id").filter(lambda x: len(x) >= 5)["customer_id"].unique()
crisis_customers = crisis["customer_id"].unique()

churned = set(loyal) - set(crisis_customers)
print(f"Loyal customers who churned: {len(churned)}")

# How many had avg rating > 4.5
avg_ratings = pre[pre["customer_id"].isin(churned)].groupby("customer_id")["rating"].mean()
high_raters_churned = (avg_ratings > 4.5).sum()
print(f"High-rated churned customers: {high_raters_churned}")
```

---

### Phase 5 — SHARE 📢
**Communicate findings through visualizations and dashboards.**

**Tool:** Power BI (`dashboard/QuickBite_Dashboard.pbix`)

The processed CSVs from `data/processed/` are loaded into Power BI as the data source.

| Dashboard Page | Visuals Used |
|---|---|
| **Executive Summary** | KPI cards — Total Orders, Revenue, Cancellation Rate, Avg Rating |
| **Order Trends** | Line chart with pre-crisis vs crisis phase shading |
| **Geographic Analysis** | Map + bar chart of city-level order decline |
| **Restaurant Performance** | Horizontal bar chart — Top 10 hardest-hit restaurants |
| **Delivery & SLA** | Line chart — avg delivery time; Gauge — SLA compliance % |
| **Customer Sentiment** | Word Cloud visual on crisis review text |
| **Customer Loyalty** | Funnel chart — loyal customer churn; Donut — retention rate |
| **Revenue Impact** | Waterfall chart — subtotal vs discounts vs delivery fees |

---

### Phase 6 — ACT ✅
**Recommend actions based on findings.**

| Finding | Recommended Action |
|---|---|
| Sharp order decline Jun–Sep | Investigate root cause — delivery outage, PR crisis, competitor promotions |
| Top cities with highest decline | Launch city-specific recovery campaigns (cashbacks, free delivery) |
| High cancellation rate | Improve restaurant readiness and delivery partner capacity |
| Delivery SLA worsening | Renegotiate SLA with delivery partners; introduce real-time tracking |
| Negative sentiment spike | Run food safety audits; address hygiene/quality complaints publicly |
| Loyal customers churned | Re-engagement campaigns with personalized offers for high-rating churners |
| Revenue loss estimate | Prioritize recovery in high-GMV cities/restaurants first |

---

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| **Python (pandas, matplotlib, seaborn)** | Data cleaning, EDA, and analysis |
| **wordcloud / collections** | Sentiment keyword extraction |
| **Power BI** | Interactive dashboard & Word Cloud visualization |
| **Power Query** | Data transformation inside Power BI |
| **Google Data Analytics Framework** | Structured 6-phase methodology |

---

## ⚙️ Requirements

```txt
pandas
matplotlib
seaborn
wordcloud
jupyter
```

Install all dependencies:
```bash
pip install -r requirements.txt
```

---

## 🚀 How to Reproduce This Analysis

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-username/QuickBite_data_analysis.git
   cd QuickBite_data_analysis
   ```

2. **Install Python dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Run notebooks in order**
   ```bash
   jupyter notebook notebooks/01_data_cleaning.ipynb
   jupyter notebook notebooks/02_primary_analysis.ipynb
   jupyter notebook notebooks/03_secondary_analysis.ipynb
   ```

4. **Open the Power BI dashboard**
   - Open `dashboard/QuickBite_Dashboard.pbix` in Power BI Desktop
   - Update the data source path to point to your `data/processed/` folder
   - Click **Refresh** to load the latest data

---

## 📚 References & Credits

- **Data and case study** provided by [Codebasics.io](https://codebasics.io)
- **Methodology:** [Google Data Analytics Certificate](https://grow.google/certificates/data-analytics/) (Coursera)
- **Tools Docs:** [Power BI Docs](https://learn.microsoft.com/en-us/power-bi/) · [pandas Docs](https://pandas.pydata.org/docs/) · [wordcloud Docs](https://amueller.github.io/word_cloud/)

---

*Made with ❤️ using Python · Power BI · Google Data Analytics framework*
