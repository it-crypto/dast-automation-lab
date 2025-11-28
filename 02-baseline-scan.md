# ZAP Baseline Scan (Passive DAST)

## 🔍 Overview
ZAP Baseline Scan is a **passive-only scan**.
This means:

- No payload injection
- No active attacks
- Safe for production
- Fast (1–3 minutes)

It crawls the target website and applies **passive rules**, such as:
- Missing security headers
- Cookie flags (Secure, HttpOnly)
- Mixed-content checks
- Basic HTML/JS issues

---

## 🛠 Workflow File Used
`.github/workflows/zap-baseline.yml`

Key elements:
- Trigger on push to `main`
- Manual trigger (workflow_dispatch)
- Generates `zap-baseline-report.html`
- Uploads as artifact

---

## ▶ How to Run
1. Go to **Actions**
2. Select **ZAP Baseline Scan**
3. Click **Run workflow**

---

## 📁 Output
A report will be available under:

**Artifacts → zap-baseline-report → zap-baseline-report.html**

You can download and open it locally.

---

## 🧠 When to Use Baseline Scan
- Regular health check scans
- Quick testing in CI/CD
- Production-safe testing
- Monitoring websites for missing headers 
- Pre-commit checks for PR validation
