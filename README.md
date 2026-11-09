# Automation & QA Developer Assessment — Ronit Gulia

## Repository Deliverables
1. **`Task1_QA_Report_Ronit_Gulia.pdf`**: Full QA & Debug audit of `https://demo.realworld.io` covering 6 production-blocking issues (concurrency, JWT storage security, network error boundaries, input trimming, auth UX, and accessibility) plus a deep-dive Root-Cause Analysis on form idempotency.
2. **`Task2_Workflow_Ronit_Gulia.json`**: Automated "Morning Brief" n8n workflow with multi-API enrichment, transformation, threshold routing, credential store usage, and non-blocking error fallbacks.
3. **`Bonus_UptimeMonitor_Ronit_Gulia.json`**: 5-minute automated health monitor for `https://demo.realworld.io` featuring 3x retry logic, millisecond latency tracking, 24h rolling uptime calculation, and non-200 incident alerting.
4. **`screenshots/`**: Visual verification of workflow canvases and live execution payloads.

---

## Task 2: n8n API Integration Architecture

### 1. APIs Used & Why
* **Primary API (`GET https://api.github.com/search/repositories`)**: Fetches trending TypeScript automation repositories. Chosen because engineering teams benefit from daily visibility into active open-source automation tooling, and GitHub returns rich metadata without complex setup.
* **Enrichment API (`GET https://api.github.com/repos/{owner}/{repo}/commits?per_page=1`)**: Enriches each top repository with its latest commit message, author, and timestamp so the digest shows real-time maintenance velocity rather than static star counts alone.

### 2. Data Transformation Logic
The **Code (`2. Filter & Keep Top 5 Repos`)** node:
1. Filters out `archived` and `disabled` repositories to remove dead projects.
2. Sorts remaining repositories in descending order by `stargazers_count`.
3. Slices the top 5 repositories and maps them into a clean schema (`rank`, `full_name`, `stars`, `forks`, `open_issues`, `html_url`).
4. A second transformation merges the upstream repository metrics with the enriched commit metadata and routes items through an **`IF` node (`stars > 1000`)** into either a **High-Traction Digest** or a **Rising Watchlist Digest**.

### 3. Error Handling & Security Design
* **Zero Silent Failures**: Every external `HTTP Request` node has `onError: continueErrorOutput` enabled, creating a dedicated error branch wired to the **`Fallback Error Handler & Logger`** node. If GitHub rate-limits (`403/429`) or times out, the workflow captures the error context, logs an actionable fallback payload, and exits cleanly.
* **Credentials Store**: All API authentication headers (`httpHeaderAuth`) are referenced via n8n's built-in Credentials Store—zero secrets are hardcoded in node parameters.

### 4. Happy-Path Webhook Test
You can trigger the workflow directly via `curl`:
```bash
curl -i -X GET http://localhost:5678/webhook-test/morning-brief