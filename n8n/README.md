# n8n Workflow

## workflow_export.json

Export of the complete n8n automation workflow.

### How to Import

1. In n8n, go to **Workflows → ⋮ menu → Import from file**
2. Select `workflow_export.json`
3. Update the ngrok base URL in all HTTP Request nodes
4. Connect Gmail and Gemini credentials

### Workflow Stages

```
[Manual Trigger]
      │
      ▼
[POST /run-cleaning]  ──── returns {status: "started"} in <1s
      │
      ▼
[Wait 8 minutes]
      │
      ▼
[GET /cleaning-status]  ──── poll for result
      │
      ▼
[IF status == "success"]
      │
      ▼
[POST /run-pipeline]  ──── returns {status: "started"} in <1s
      │
      ▼
[Wait 20 minutes]
      │
      ▼
[GET /status]  ──── poll for ML result (contains top 5, tier counts, AUC etc.)
      │
      ▼
[Google Gemini]  ──── generate 4 action bullets from ML results
      │
      ▼
[Gmail]  ──── send HTML report with charts + Gemini insights
```

### Why Async Design?

ngrok disconnects any HTTP request taking over 2 minutes (ERR_NGROK_3004).
Both `/run-cleaning` and `/run-pipeline` return immediately, and n8n polls
for completion — this eliminates all timeout errors.
