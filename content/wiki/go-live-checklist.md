---
title: "Go-live load test & deployment checklist"
draft: false
---

> 🎯 **Goal:** Ensure the system is stable, scalable, and production-ready before launching.

---

## 🧱 I. CODE & FUNCTIONAL

- [ ] Code reviewed and approved by at least one senior developer
- [ ] Unit test coverage ≥ 80%
- [ ] Edge cases handled properly
- [ ] No N+1 queries (GraphQL/ORM optimized)
- [ ] No leftover `console.log` or `debugger` in code
- [ ] No secrets or tokens hardcoded

---

## 🚦 II. CI/CD & DEPLOYMENT

- [ ] CI pipeline set up (build → test → deploy)
- [ ] Rollback plan in place (script or quick revert strategy)
- [ ] Staging/UAT environment available and tested
- [ ] `.env` variables configured correctly (not copied from local)
- [ ] Proper branch/tag naming (e.g. `release/v1.0.0`)

---

## 🔒 III. SECURITY

- [ ] Auth applied on all sensitive routes (JWT, OAuth, API keys)
- [ ] Role & permission system tested
- [ ] No publicly exposed sensitive endpoints (`/admin`, `/debug`)
- [ ] Checked for XSS, SQLi, path traversal vulnerabilities
- [ ] Rate limiting/throttling in place for public APIs
- [ ] Security headers enabled (`helmet`, CORS, etc.)

---

## 📈 IV. PERFORMANCE & SCALING

- [ ] Load tested for target RPS (e.g. ≥ 10k req/s)
- [ ] Concurrent users simulated (e.g. 1k logins)
- [ ] DB indexed properly (`EXPLAIN ANALYZE` used)
- [ ] Redis, CDN, or in-memory caching applied
- [ ] File/media offloaded to external storage (e.g. S3)
- [ ] API only returns necessary data (REST/GraphQL select)

---

## 🧠 V. MONITORING & ALERTING

- [ ] Logging integrated (Winston, Pino, ELK, Datadog, etc.)
- [ ] Logs include traceId, userId, route context
- [ ] Metrics collection: Prometheus / Grafana / OpenTelemetry
- [ ] Alert configured for:
  - High error rate (5xx)
  - CPU > 80%
  - Memory full
  - Queue backlog increasing

---

## ⚙️ VI. INFRASTRUCTURE

- [ ] Load balancer configured and stable
- [ ] Auto-restart and healthcheck ready (PM2, Docker, k8s readiness)
- [ ] Database backup and restore tested
- [ ] Horizontal scaling/pod autoscaling set up if using k8s
- [ ] `.env.production` encrypted or managed via Vault

---

## 📋 VII. OTHER

- [ ] Changelog written (`CHANGELOG.md`)
- [ ] Release announcement sent to relevant teams
- [ ] Post-deploy checklist created (Smoke test UI/API)
- [ ] QA checklist completed (login, checkout, error, etc.)
- [ ] Graceful fallback UI for error handling
- [ ] `/status`, `/health`, `/metrics`, `/version` endpoints ready

---

## ✅ READY TO GO LIVE?

| Goal                           | Ready? |
|--------------------------------|--------|
| Code clean and stable          | ✅      |
| Load test passed               | ✅      |
| Monitoring in place            | ✅      |
| Rollback plan confirmed        | ✅      |
| Stakeholders notified          | ✅      |

---

## 🔁 POST GO-LIVE ACTIONS

- [ ] Real-time system monitoring active
- [ ] User feedback monitored
- [ ] Latency/error rate observed
- [ ] 24h go-live report or postmortem (if needed)
