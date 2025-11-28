# ZAP API Scan

## 📘 Overview
ZAP API scan is designed for:
- REST APIs
- OpenAPI/Swagger JSON
- API gateway testing
- Backend services

It does not crawl webpages.
Instead, it reads an OpenAPI (Swagger) file and tests:
- Auth endpoints
- Input validation
- Error handling
- Injection surfaces

---

## 🛠 Workflow File Used
`.github/workflows/zap-api-scan.yml`

Key features:
- Manual trigger
- Uses the ZAP API scan action
- Takes a Swagger/OpenAPI spec
- Generates `zap-api-report.html`

---

## ▶ How to Run
1. Go to **Actions**
2. Select **ZAP API Scan**
3. Run workflow manually

---

## 📁 Output
- `zap-api-report.html`

This includes:
- API endpoints tested
- Injection attempts
- Vulnerability summary
- Evidence from request/response

---

## 🧠 When to Use API Scan
- Testing backend microservices
- Testing API gateways
- Security scanning in CI
- Learning how APIs expose risks
- Validating new API endpoints
