# 🛒 AI-Powered Retail Analytics & Inventory Optimization

An end-to-end automated retail analytics pipeline that ingests 32M+ transaction records, trains an XGBoost reorder classifier, scores inventory priority, and delivers AI-generated insights via automated Gmail — all triggered by a single n8n button click in under 20 minutes.

---

## 📋 Table of Contents

- [Business Problem](#-business-problem)
- [Key Results](#-key-results)
- [Architecture](#-architecture)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
- [Pipeline Overview](#-pipeline-overview)
- [Results & Charts](#-results--charts)
- [Model Performance](#-model-performance)
- [Future Enhancements](#-future-enhancements)
- [References](#-references)

---

## 🎯 Business Problem

A mid-sized grocery retailer spent **3–4 hours per week** manually exporting CSVs, analysing in Excel, and emailing results — with no predictive capability. This project answers four core business questions **automatically, every week**:

| # | Question | Output |
|---|----------|--------|
| 1 | Which products sell the most? | `top_products_latest.csv` + Power BI chart |
| 2 | Which products will customers buy again? | `reorder_predictions_latest.csv` + email top 5 |
| 3 | When do customers shop the most? | Hourly + daily charts in email + Power BI |
| 4 | How should stores manage inventory? | `inventory_priority_latest.csv` + priority tiers |

---

## 🏆 Key Results

| Metric | Value |
|--------|-------|
| **XGBoost AUC-ROC** | 0.790 |
| **Reorder Recall** | 69.0% (vs 15.9% for Logistic Regression) |
| **Top Product** | Banana — 472,565 orders |
| **Peak Shopping** | 10 AM, Sunday & Monday |
| **High-priority products** | 2 (Banana 0.88, Bag of Organic Bananas 0.79) |
| **Pipeline runtime** | < 20 minutes (end-to-end) |
| **Manual process replaced** | 3–4 hours → 1 button click |

---

## 🏗 Architecture

```
┌─────────────────────────────────────┐
│  Retail Dataset (CSV / Google Drive) │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│        n8n Trigger Node              │  Manual trigger / Schedule (Sunday 6 AM)
│     (Schedule / Webhook)             │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│          Data Fetch Node             │  HTTP POST → Flask/ngrok → Google Colab
│       (Google Drive / ngrok)         │  Async: returns {status:"started"} in <1s
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│     Execute Python Script            │  Universal 8-step cleaning pipeline
│        (Data Cleaning)               │  Auto-discovers all CSVs in Raw_Data/
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│       Feature Engineering            │  6 leakage-free features engineered
│       (Create ML Features)           │  times_purchased, purchase_rate, etc.
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│      Machine Learning Model          │  XGBoost — AUC-ROC 0.790, Recall 69%
│   (Demand / Reorder Prediction)      │  500K stratified sample, scale_pos_weight=9
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│         Prediction Output            │  reorder_predictions_latest.csv
│       (Forecasts & Insights)         │  inventory_priority_latest.csv
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│           Data Storage               │  All outputs saved to Google Drive
│         (CSV / Google Drive)         │  Dated + _latest versions each run
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│       Power BI Dataset Refresh       │  Connects directly to Google Drive CSVs
│                                      │  Live refresh on each pipeline run
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│         Insight Generation           │  Google Gemini API
│                                      │  4 AI-generated action bullets
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│      Notification / Reporting        │  Automated Gmail HTML report
│   (Email / Dashboard)                │  Charts + Gemini insights + top 5 table
└─────────────────────────────────────┘
```

---

## 📁 Project Structure

```
aisle-analytics/
│
├── README.md
├── .gitignore
│
├── notebook/
│   └── Retail_Pipeline_Combined_v2.ipynb   ← Main Colab notebook (all-in-one)
│
├── docs/
│   ├── Final_Report_Complete.docx          ← Full IEEE-format project report
│   └── AI-Powered-Retail-Analytics.pptx   ← Project presentation
│
├── n8n/
│   └── workflow_export.json               ← n8n workflow (import directly)
│
├── powerbi/
│   └── RetailAnalytics_Dashboard.pdf      ← Power BI dashboard export
│
├── outputs/
│   └── charts/
│       ├── top10_products.png
│       ├── hourly_orders.png
│       ├── daily_orders.png
│       ├── feature_importance.png
│       └── model_comparison.png
│
└── .github/
    └── SETUP_GUIDE.md                     ← Step-by-step setup instructions
```

> **Note:** Raw data CSVs (~700 MB) are not included. Download from [Instacart Kaggle Dataset](https://www.kaggle.com/c/instacart-market-basket-analysis) and place in `Retail_Pipeline/Raw_Data/` on your Google Drive.

---

## 🚀 Quick Start

### Prerequisites
- Google Account (Colab + Drive)
- [ngrok](https://ngrok.com) account (free tier)
- [n8n](https://n8n.io) account (cloud or self-hosted)
- Power BI Desktop (free)

### Steps

**1. Set up Google Drive**
```
MyDrive/
└── Retail_Pipeline/
    ├── Raw_Data/        ← Place Instacart CSVs here
    ├── cleaned_data/    ← Auto-created by pipeline
    ├── ML_Output/       ← Auto-created by pipeline
    └── models/          ← Auto-created by pipeline
```

**2. Run the Colab Notebook**
```
1. Open notebook/Retail_Pipeline_Combined_v2.ipynb in Google Colab
2. Run ALL cells top to bottom (Cell 1 → Cell 7)
3. Copy the ngrok URL printed in Cell 6
```

**3. Configure n8n**
```
1. Import n8n/workflow_export.json into your n8n instance
2. Paste the ngrok base URL into all HTTP Request nodes
3. Set up Gmail OAuth credentials in n8n
4. Set up Google Gemini API key in n8n
```

**4. Run the Pipeline**
```
Click the manual trigger in n8n → done in ~20 minutes
```

See [`.github/SETUP_GUIDE.md`](.github/SETUP_GUIDE.md) for the full step-by-step walkthrough.

---

## 🔧 Pipeline Overview

### Data Cleaning (Cell 3)
Universal 8-step pipeline applied automatically to every CSV in `Raw_Data/`:

| Step | Action |
|------|--------|
| 1 | Standardise column names (lowercase + underscores) |
| 2 | Strip whitespace from text columns |
| 3 | Remove fully empty rows |
| 4 | Remove exact duplicate rows |
| 5 | Composite key deduplication (`order_id` + `product_id`) |
| 6 | Range checks (`order_dow` 0–6, `order_hour` 0–23, `reordered` 0/1) |
| 7 | Fill nulls → 0 (except `days_since_prior_order`) |
| 8 | IQR × 1.5 outlier removal on ML feature columns only |

### Feature Engineering (Cell 4)
Leakage-free train/future split: each user's **last order** is the prediction target; all prior orders form training history.

| Feature | Importance | Description |
|---------|-----------|-------------|
| `times_purchased` | **40%** | How many times user ordered this product |
| `purchase_rate` | **37%** | `times_purchased / (total_orders + 1)` |
| `total_orders` | 14% | Total orders placed by this user |
| `product_total_orders` | 4% | Total orders for product across all users |
| `avg_days_between_orders` | 3% | User shopping cadence |
| `avg_order_hour` | 2% | User mean shopping hour |

### Inventory Scoring Formula
```
final_priority = 0.4 × demand_norm + 0.3 × reorder_rate + 0.3 × avg_reorder_prob
```
Products tiered as **High** (0.66–1.0), **Medium** (0.33–0.66), **Low** (0–0.33).

---

## 📊 Results & Charts

### Q1 — Top 10 Best-Selling Products
All top 10 are fresh produce. Banana leads with 472,565 total orders — 1.3× more than third place.

![Top 10 Best-Selling Products](outputs/charts/top10_products.png)

---

### Q3 — When Do Customers Shop?

Orders peak at **10 AM** with ~290,000 orders. Activity remains high through 3 PM before dropping off sharply after 8 PM.

![Orders by Hour of Day](outputs/charts/hourly_orders.png)

**Sunday (~600K) and Monday (~590K)** lead weekly order volumes by a significant margin. Stores should fully stock shelves before 9 AM on Sundays.

![Orders by Day of Week](outputs/charts/daily_orders.png)

---

### Q2 — XGBoost Feature Importance
`times_purchased` (40%) and `purchase_rate` (37%) together account for **77% of model importance**, confirming that individual purchase history is the dominant reorder signal.

![XGBoost Feature Importance](outputs/charts/feature_importance.png)

---

## 📈 Model Performance

XGBoost correctly identifies **69% of actual future reorders**, enabling proactive restocking before stockouts occur. Logistic Regression's misleadingly high 90.4% accuracy comes from predicting non-reorder for almost every case — making it useless for inventory management.

![Model Comparison](outputs/charts/model_comparison.png)

| Metric | Logistic Regression | XGBoost |
|--------|--------------------:|--------:|
| Accuracy | 0.904* | 0.738 |
| AUC-ROC | 0.782 | **0.790** |
| Recall (reorders) | 15.9% | **69.0%** |

---

## 📚 References

1. Hozyfas et al., "Integration of ML and Advanced Computing for Retail Analytics," *Int. J. Business and Economics Insights*, 2022.
2. T. Chen & C. Guestrin, "XGBoost: A Scalable Tree Boosting System," *KDD*, 2016. [arXiv](https://arxiv.org/abs/1603.02754)
3. Microsoft, [Power BI Documentation](https://docs.microsoft.com/power-bi), 2024.
4. n8n, [Workflow Automation Documentation](https://docs.n8n.io), 2024.
5. Google, [Gemini API Documentation](https://ai.google.dev/docs), 2024.
6. Instacart, [Online Grocery Shopping Dataset 2017](https://www.kaggle.com/c/instacart-market-basket-analysis), Kaggle, 2017.
7. P. Geurts et al., "Extremely Randomized Trees," *Machine Learning*, vol. 63, pp. 3–42, 2006.

