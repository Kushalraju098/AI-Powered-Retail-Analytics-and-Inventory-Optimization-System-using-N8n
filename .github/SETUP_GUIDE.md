# 🛠 Step-by-Step Setup Guide

This guide walks you through setting up the full Aisle Analytics pipeline from scratch.

---

## Prerequisites

| Tool | Version | Purpose |
|------|---------|---------|
| Google Account | Any | Colab + Drive + Gmail |
| ngrok | Free tier | Expose Colab Flask server |
| n8n | Cloud or self-hosted | Workflow orchestration |
| Power BI Desktop | Free | Dashboard |
| Python | 3.10+ | Auto-installed in Colab |

---

## Step 1 — Download the Dataset

1. Go to [Kaggle Instacart Dataset](https://www.kaggle.com/c/instacart-market-basket-analysis/data)
2. Download and unzip all CSV files
3. You need: `orders.csv`, `order_products__prior.csv` (rename to `order_products.csv`), `products.csv`, `aisles.csv`, `departments.csv`

---

## Step 2 — Set Up Google Drive

Create this exact folder structure in your Google Drive:

```
MyDrive/
└── Retail_Pipeline/
    └── Raw_Data/
        ├── orders.csv
        ├── order_products.csv
        ├── products.csv
        ├── aisles.csv
        └── departments.csv
```

The pipeline auto-creates `cleaned_data/`, `ML_Output/`, and `models/` on first run.

---

## Step 3 — Set Up ngrok

1. Sign up at [ngrok.com](https://ngrok.com) (free)
2. Go to **Your Authtoken** in the dashboard
3. Copy your token
4. In Cell 6 of the notebook, replace the placeholder:
   ```python
   pyngrok.set_auth_token('YOUR_TOKEN_HERE')
   ```

**Optional but recommended:** Create a static domain in ngrok dashboard so the URL never changes between sessions. Then in Cell 6:
```python
tunnel = pyngrok.connect(5000, domain='your-domain.ngrok-free.app')
```

---

## Step 4 — Run the Colab Notebook

1. Open `notebook/Retail_Pipeline_Combined_v2.ipynb` in [Google Colab](https://colab.research.google.com)
2. Go to **Runtime → Change runtime type → T4 GPU** (optional but faster)
3. Run cells **in order, top to bottom**:
   - Cell 1: Installs packages
   - Cell 2: Mounts Drive, shows your CSV files
   - Cell 3: Defines cleaning functions
   - Cell 4: Defines ML functions
   - Cell 5: Skip (manual test only)
   - Cell 6: Starts Flask + ngrok — **copy the printed URL**
   - Cell 7: Keep-alive script — **leave this running**

> ⚠️ Keep the Colab tab open. Cell 6 must remain running for n8n to reach the endpoints.

---

## Step 5 — Configure n8n

### Import the Workflow
1. In n8n, go to **Workflows → Import from file**
2. Select `n8n/workflow_export.json`

### Set the ngrok URL
In every HTTP Request node, update the URL to your ngrok base URL:

| Node | Method | Path |
|------|--------|------|
| Trigger Cleaning | POST | `{YOUR_URL}/run-cleaning` |
| Poll Cleaning | GET | `{YOUR_URL}/cleaning-status` |
| Trigger ML | POST | `{YOUR_URL}/run-pipeline` |
| Poll ML | GET | `{YOUR_URL}/status` |

### Set Up Credentials
- **Gmail:** Connect via OAuth 2.0 in n8n credentials
- **Google Gemini:** Add your API key from [Google AI Studio](https://aistudio.google.com/app/apikey)

### Workflow Timing
The workflow is designed around these wait times:
```
POST /run-cleaning → Wait 8 mins → GET /cleaning-status
                                          ↓ (if success)
POST /run-pipeline → Wait 20 mins → GET /status
                                          ↓ (if success)
Gemini AI → Gmail delivery
```

---

## Step 6 — Set Up Power BI

1. Open `powerbi/RetailAnalytics_Dashboard.pbix` in Power BI Desktop
2. Go to **Transform Data → Data Source Settings**
3. Update the Google Drive paths to point to your `ML_Output/` folder:
   - `reorder_predictions_latest.csv`
   - `inventory_priority_latest.csv`
   - `top_products_latest.csv`
4. Click **Refresh**

---

## Step 7 — Run the Full Pipeline

1. In n8n, click the **Manual Trigger** button
2. Watch the workflow execute step by step
3. After ~20 minutes, check your Gmail for the automated report

---

## Troubleshooting

### ERR_NGROK_3004 (timeout)
The Flask endpoints return `{status: "started"}` immediately — they should never timeout. If you see this, check that Cell 6 is still running.

### Colab RAM crash
The pipeline uses dtype downcasting to reduce `order_products.csv` from ~2 GB to ~800 MB. If you still crash, try: **Runtime → Disconnect and delete runtime → Re-run all cells**.

### n8n shows "running" forever on /cleaning-status
The cleaning step takes 7–10 minutes for the large CSVs. The 8-minute wait node is intentional. If it consistently fails, increase the Wait node to 12 minutes.

### Gmail images not showing
Gmail blocks base64 inline images. The pipeline uses Google Drive public URL links instead. Make sure `ML_Output/` charts are set to **"Anyone with the link can view"** in Google Drive.

### Gemini rate limit
The free tier supports high concurrent volume. If you hit limits, add a 5-second delay node before the Gemini call in n8n.

---

## Optional: Schedule Weekly Automation

To run automatically every Sunday at 6 AM:
1. In n8n, replace the Manual Trigger with a **Schedule Trigger**
2. Set: Cron expression `0 6 * * 0`
3. Make sure your Colab session is running (or deploy Flask to Cloud Run — see Future Enhancements in README)
