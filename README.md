# ⚡ QuickCommerce Operations Optimization

> An end-to-end **operational analytics framework** for a Blinkit/Zepto-style dark-store network — built from scratch with realistic SQL schemas, a linear-programming optimization model, and a Power BI-style executive dashboard.

## 🌐 Live Dashboard

👉 **View the live dashboard:** `https://<your-username>.github.io/<repo-name>/`

(Replace `<your-username>` and `<repo-name>` after deployment)

---

## 🚀 How to Publish on GitHub Pages (First-Time Setup)

### Step 1 — Create a new GitHub repository
1. Go to https://github.com/new
2. Repository name: e.g. `quickcommerce-dashboard` (pick anything)
3. Set it to **Public** (GitHub Pages requires Public on free accounts)
4. **Do NOT** initialize with README/.gitignore (we already have a README)
5. Click **Create repository**

### Step 2 — Upload these files
**Option A: Via GitHub web UI (easiest for first-time users)**
1. On your new empty repo page, click **"uploading an existing file"**
2. Drag-and-drop ALL files and folders from this project into the upload area
   - Make sure `index.html` and `.nojekyll` are at the **root**
3. Scroll down and click **Commit changes**

**Option B: Via Git command line**
```bash
cd QuickCommerce-Operations-Optimization
git init
git add .
git commit -m "Initial commit - QuickCommerce dashboard"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo-name>.git
git push -u origin main
```

### Step 3 — Enable GitHub Pages
1. Go to your repo on GitHub → click **Settings** (top tab)
2. In the left sidebar, click **Pages**
3. Under **"Build and deployment"** → **Source**, select **"Deploy from a branch"**
4. Under **Branch**, choose **`main`** and folder **`/ (root)`**
5. Click **Save**
6. Wait 1–2 minutes ⏳

### Step 4 — Visit your live dashboard
GitHub will show you a green box with your URL:
```
https://<your-username>.github.io/<repo-name>/
```
Click it — your dashboard is live! 🎉

> 💡 **Why `index.html` is at the root:** GitHub Pages automatically serves the `index.html` file at the root of your repository when someone visits your site URL.
>
> 💡 **Why `.nojekyll` is included:** This empty file tells GitHub Pages to skip Jekyll processing and serve your files exactly as they are — important for projects with `_underscore` folders or special characters.

---

![Tech Stack](https://img.shields.io/badge/SQL-PostgreSQL-blue) ![Excel](https://img.shields.io/badge/Excel-Solver%20LP-green) ![Python](https://img.shields.io/badge/Python-Plotly%20%2B%20PuLP-orange) ![Status](https://img.shields.io/badge/status-portfolio--ready-success)

---

## 🎯 Business Problem

Quick-commerce platforms (Blinkit, Zepto, Instamart) promise a **15-minute delivery SLA**. When a dark store violates this promise, every late order triggers:

- **Refund risk** (~15% of delayed orders request refunds → cost = 15% × order value)
- **Coupon compensation** (₹50 per delivery > 20 minutes — matches Blinkit's actual policy)
- **Customer churn** that compounds over time

**Goal:** Use SQL → Excel Solver → Power BI to identify failing stores, **mathematically optimize** rider allocation, and **quantify the rupee impact** of the change.

---

## 🏗️ Project Architecture

```
        ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐
        │  PostgreSQL      │    │  Excel Solver    │    │  Power BI / HTML │
        │  ──────────────  │ →  │  ──────────────  │ →  │  Dashboard       │
        │  3 tables        │    │  LP optimizer    │    │  3 pages         │
        │  100 orders      │    │  CBC engine      │    │  KPIs + charts   │
        │  40 riders       │    │                  │    │                  │
        └──────────────────┘    └──────────────────┘    └──────────────────┘
                ↓                       ↓                        ↓
          5 analytical            Optimal rider           Live operational
              queries              allocation             health monitoring
```

---

## 📂 Repository Structure

```
QuickCommerce-Operations-Optimization/
├── README.md                          ← You are here
├── sql/
│   ├── 01_schema_and_seed.sql        ← 3 tables, 100 orders, 40 riders
│   └── 02_analytical_queries.sql     ← 5 CTE / Window-function queries
├── excel/
│   └── QuickCommerce_Optimization.xlsx ← Solver workbook (LP-ready)
├── dashboard/
│   ├── dashboard.html                 ← Interactive Power-BI replica (Plotly)
│   └── QuickCommerce_Dashboard_Portfolio.pdf  ← 4-page PDF version
├── outputs/                           ← CSV exports of all 5 queries
└── docs/
    └── interviewer_walkthrough.md     ← Verbal explanation script
```

---

## 🔑 Key Findings

| Metric                                | Value           |
|---------------------------------------|-----------------|
| Stores violating 15-min SLA           | **9 of 10**     |
| Worst performer                       | DS002 Morar — **90 % delay rate** |
| Best performer                        | DS003 Thatipur — **10 % delay rate** |
| Daily late cost (current)             | **₹1,75,975**   |
| Daily late cost after LP optimization | **₹54,800**     |
| **Monthly savings**                   | **₹36.3 Lakh**  |
| **Annual savings**                    | **₹4.4 Crore**  |

---

## 🧮 The Linear Programming Model

**Decision variable:** `xᵢ` = riders allocated to dark store *i* (integer ≥ 1)

**Objective:** Minimize total daily cost
> minimize Σᵢ (payoutᵢ × 10 × xᵢ)  +  ₹200 × Σᵢ unmet_demandᵢ

**Constraints:**
1. `Σ xᵢ ≤ Total rider pool` (active + bench)
2. `10 × xᵢ + slackᵢ ≥ daily_demandᵢ` (meet demand)
3. `10 × xᵢ ≤ max_capacityᵢ` (store throughput limit)
4. `xᵢ ∈ ℤ⁺` (riders are whole humans)

Implemented twice for traceability:
- **Excel Solver** (Simplex LP / `Simplex_LP` engine) — `excel/QuickCommerce_Optimization.xlsx`
- **PuLP + CBC** (open-source LP solver) — `build_excel_solver.py`

Both produce **identical** optimal allocations.

---

## 🧠 SQL Techniques Demonstrated

| Query | Technique                                                                 |
|-------|---------------------------------------------------------------------------|
| Q1    | CTE chain + conditional `COUNT` for SLA percentage                        |
| Q2    | Multi-table `JOIN` + `NULLIF` to prevent division-by-zero                 |
| Q3    | Three **window functions** in one query: `RANK() OVER`, `AVG() OVER`, running-`SUM() OVER` with frame clause |
| Q4    | Embedded **business logic**: 15 % refund risk + ₹50 coupon = Blinkit's real policy |
| Q5    | Composite weighted KPI score (50 % punctuality + 30 % rider quality + 20 % cost efficiency) |

---

## ▶️ How to Reproduce This Project

```bash
# 1. Create the database
psql -f sql/01_schema_and_seed.sql

# 2. Run the analytical queries
psql -f sql/02_analytical_queries.sql

# 3. Open the Excel workbook
#    File → excel/QuickCommerce_Optimization.xlsx
#    Go to sheet "2. Solver Setup" → Data → Solver → Solve
#    Compare with sheet "3. Optimal Solution"

# 4. Open the dashboard
#    Open dashboard/dashboard.html in any browser
#    OR open the 4-page PDF: dashboard/QuickCommerce_Dashboard_Portfolio.pdf
```

---

## 🗣️ Interviewer Walkthrough

> *"To prove my analytical capabilities, I built an end-to-end operational optimization framework for a quick-commerce model. I didn't use a clean online dataset — I wrote the SQL schemas from scratch to simulate realistic operational bottlenecks like dark-store capacity limits, rider attendance gaps, and SLA penalties.*
>
> *I then translated the SQL findings into Excel Solver to run a linear-programming optimization model — minimizing a weighted objective of payout cost plus late-delivery penalty, constrained by rider pool, store throughput, and integer rider counts. The LP says we should move 6 riders to Morar and Hazira, reducing daily late-cost from ₹1.76 L to ₹55 K — a 69 % drop, or ₹4.4 crore annually.*
>
> *Finally, I visualized the operational and financial impact in a Power BI-style 3-page dashboard. You can see the full database architecture, all five analytical queries, the Solver workbook, and the interactive dashboard live in this repository right now."*

---

## 🛠️ Tech Stack

- **Database:** PostgreSQL 14+ (also SQLite & BigQuery compatible)
- **Optimization:** Microsoft Excel Solver (Simplex LP) + PuLP + CBC solver
- **Visualization:** Power BI Desktop (PBIX) / Plotly (HTML) / Matplotlib (PDF)
- **Language:** Python 3.13 (Pandas, openpyxl, PuLP, Plotly)

---

## 📈 Roadmap / Next Steps

- [ ] Add a **Power BI .pbix** file (Windows-only) for native Power BI users
- [ ] Extend to **multi-day demand forecasting** (Prophet / XGBoost)
- [ ] Add **real-time alerting** via Slack/Email when a store crosses the 40 % delay threshold
- [ ] Deploy interactive dashboard to **GitHub Pages** for a shareable live link

---

## 👤 Author

Portfolio project demonstrating SQL + Optimization + BI skills for analyst / data-science roles in **e-commerce, operations, and supply chain**.

*Built end-to-end · No off-the-shelf datasets used · Every line of code is original.*
