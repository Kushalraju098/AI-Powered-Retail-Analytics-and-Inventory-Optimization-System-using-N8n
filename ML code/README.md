# Notebook

## Retail_Pipeline_Combined_v2.ipynb

The single all-in-one Google Colab notebook for the entire pipeline.

### Cell Summary

| Cell | Purpose | Runtime |
|------|---------|---------|
| 1 | Install packages & imports | ~30 sec |
| 2 | Mount Google Drive + verify CSVs | ~10 sec |
| 3 | Define universal cleaning functions | Instant |
| 4 | Define ML pipeline functions | Instant |
| 5 | Optional manual test (commented out) | — |
| 6 | Start Flask server + ngrok tunnel | ~5 sec |
| 7 | Keep-alive script | Instant |

### Flask Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/ping` | Health check |
| POST | `/run-cleaning` | Trigger cleaning (async, returns in <1s) |
| GET | `/cleaning-status` | Poll cleaning result |
| POST | `/run-pipeline` | Trigger ML pipeline (async, returns in <1s) |
| GET | `/status` | Poll ML result |

### Before committing

⚠️ **Remove your ngrok auth token** from Cell 6 before pushing to GitHub:
```python
# Change this:
pyngrok.set_auth_token('3BHOX1wWLm...')
# To this:
pyngrok.set_auth_token('YOUR_NGROK_AUTH_TOKEN_HERE')
```
