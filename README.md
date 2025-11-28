# DAST Automation Lab

This repository demonstrates automated Dynamic Application Security Testing (DAST) using **OWASP ZAP** and **GitHub Actions**.

## 📚 What’s Inside
- 🔹 ZAP Baseline Scan (passive scan)
- 🔹 ZAP Full Scan (active DAST)
- 🔹 ZAP API Scan (OpenAPI-based)

All scans generate HTML reports automatically.

## 📂 Repository Structure
.github/workflows/
├── zap-baseline.yml
├── zap-fullscan.yml
└── zap-api-scan.yml

docs/
├── 01-introduction.md
├── 02-baseline-scan.md
├── 03-full-scan.md
└── 04-api-scan.md


## ▶ How to Run Scans
Go to **Actions** → select a workflow → **Run workflow**.

## 🧠 Why This Project?
This repo helps you learn:
- CI/CD security automation
- DAST scanning techniques
- Difference between passive, active, and API scans
- GitHub Actions automation
- ZAP report interpretation
