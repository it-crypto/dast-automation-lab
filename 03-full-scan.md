# ZAP Full Scan (Active DAST)

## ⚔️ Overview
ZAP Full Scan performs **active penetration testing**:

- Payload injection
- Attempting XSS
- Attempting SQL injection
- Forced browsing
- Fuzzing inputs
- Full crawler

⚠️ **This should NOT be run against production targets.**

---

## 🛠 Workflow File Used
`.github/workflows/zap-fullscan.yml`

Key features:
- Manual trigger only (workflow_dispatch)
- No auto trigger on push (safety)
- Generates `zap-fullscan-report.html`
- Uploaded as artifact

---

## ▶ How to Run
1. Go to **Actions**
2. Select **ZAP Full Scan**
3. Click **Run workflow**

---

## 📁 Output
- `zap-fullscan-report.html`
A detailed report including:
- High risk alerts
- Attack vectors
- Request/Response payloads
- Steps to reproduce
- Evidence

---

## 🧠 When to Use Full Scan
- Testing QA environments
- Testing staging deployments
- Full DAST during release cycles
- Learning real attack vectors
- Security assessments
