# 🧃 OWASP Juice Shop DAST Workflow

## Overview

This repository contains a **Dynamic Application Security Testing (DAST)** workflow designed for the OWASP Juice Shop application. The workflow fully automates:

- Running Juice Shop inside Docker  
- Automatically registering & logging in a new user  
- Performing authenticated crawling using Katana  
- Extracting all publicly accessible + authenticated URLs  
- Saving results as artifacts for ZAP or other scanners  
- Acting as a reusable DAST engine for any application  

---

## 1️⃣ Checkout Repository

```yaml
- name: Checkout Repo
  uses: actions/checkout@v3
```

---

## 2️⃣ Start Juice Shop in Docker

```bash
docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
sleep 30
```

- Juice Shop runs at **http://localhost:3000**  
- `sleep 30` ensures the app finishes booting

---

## 3️⃣ Install Katana (JavaScript-aware crawler)

Katana crawls:

- HTML links  
- JavaScript-embedded URLs  
- XHR/fetch requests  
- Angular/React/Vue routes  
- Backend API endpoints  

Install command:

```bash
wget https://github.com/projectdiscovery/katana/releases/download/v1.2.2/katana_1.2.2_linux_amd64.zip
unzip -j katana_1.2.2_linux_amd64.zip katana -d /usr/local/bin
chmod +x /usr/local/bin/katana
```

⚠️ Katana v1.2.x uses:  
`-H "Header: value"` NOT `-header`.

---

## 4️⃣ Auto‑Register & Auto‑Login

The workflow:

1. Generates random email/password
2. Registers via `/api/Users`
3. Logs in via `/rest/user/login`
4. Extracts session JWT → stores in `TOKEN`

Example registration request:

```bash
curl -s -X POST http://localhost:3000/api/Users \
  -H "Content-Type: application/json" \
  -d '{"email":"user123@test.com","password":"Passw0rd!","securityQuestion":{"id":1,"answer":"test"}}'
```

Login request:

```bash
TOKEN=$(curl -s -X POST http://localhost:3000/rest/user/login \
  -H "Content-Type: application/json" \
  -d '{"email":"user123@test.com","password":"Passw0rd!"}' | jq -r '.authentication.token')
```

WHOAMI sanity check:

```bash
curl -i -H "Authorization: Bearer $TOKEN" http://localhost:3000/rest/user/whoami
```

Expected response:

```json
200 OK
{"email":"user123@test.com","role":"customer"}
```

---

## 5️⃣ Katana Crawling (Unauthenticated)

```bash
touch unauthenticated_urls.txt
katana -u http://localhost:3000 -d 7 -jsl -silent -o unauthenticated_urls.txt || true
wc -l unauthenticated_urls.txt
```

---

## 6️⃣ Katana Crawling (Authenticated)

```bash
touch authenticated_urls.txt
katana -u http://localhost:3000 -d 7 -jsl -silent \
  -H "Authorization: Bearer $TOKEN" \
  -o authenticated_urls.txt || true
wc -l authenticated_urls.txt
```

---

## 7️⃣ Extract JS-based URLs

```bash
wget -q -O main.js http://localhost:3000/main.js || true
wget -q -O runtime.js http://localhost:3000/runtime.js || true
wget -q -O polyfills.js http://localhost:3000/polyfills.js || true
wget -q -O vendor.js http://localhost:3000/vendor.js || true

grep -Eo 'https?://[a-zA-Z0-9./?=_-]+' *.js 2>/dev/null > js_urls_unique.txt || true
grep -Eo '/[a-zA-Z0-9./?=_-]+' *.js 2>/dev/null >> js_urls_unique.txt || true

sort -u js_urls_unique.txt -o js_urls_unique.txt
wc -l js_urls_unique.txt
```

---

## 8️⃣ Identify Authenticated-only URLs

```bash
grep -Fxv -f unauthenticated_urls.txt authenticated_urls.txt > auth_only_urls.txt || true
wc -l auth_only_urls.txt
```

---

## 9️⃣ ZAP FULL SCAN (Authenticated)

```yaml
- name: Run ZAP Full Scan Authenticated
  uses: zaproxy/action-full-scan@v0.13.0
  env:
    ZAP_AUTH_HEADER: "Authorization"
    ZAP_AUTH_HEADER_VALUE: "Bearer ${{ env.TOKEN }}"
    ZAP_AUTH_HEADER_SITE: "http://localhost:3000"
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    target: "http://localhost:3000"
    cmd_options: "-j -a -d"
    fail_action: false
    allow_issue_writing: false
    artifact_name: zap_fullscan_auth_report
```

---

## 10️⃣ ZAP API Scan (Authenticated URLs)

```yaml
- name: ZAP API Scan
  uses: zaproxy/action-api-scan@v0.10.0
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    target: http://localhost:3000
    apis: authenticated_urls.txt
    allow_issue_writing: false
    fail_action: false
    artifact_name: zap_apiscan_report
```

---

## 11️⃣ Upload Crawl Artifacts

```yaml
- name: Upload Crawler URL Files
  uses: actions/upload-artifact@v5
  with:
    name: crawl-url-artifacts
    path: |
      unauthenticated_urls.txt
      authenticated_urls.txt
      auth_only_urls.txt
      js_urls_unique.txt
```

---

## 📁 Example Output: authenticated_urls.txt

```
/rest/user/change-password
/rest/products/search
/file-upload
/basket
/order-history
/api/Feedbacks
/api/Users
/rest/admin
/data-export
/address/edit/
```

---

## 🔧 Troubleshooting

- **Katana `-H` flag:** Always use `-H "Header: value"`  
- **401 on WHOAMI:** Check login token/cookie  
- **Few URLs:** Increase depth (`-d 7`) and sleep time (`sleep 30`)  
- **JS URLs missing:** Ensure `-jsl` flag and download main JS bundles

---

## 🚀 Next Steps

- Use `authenticated_urls.txt` → ZAP API scan  
- Seed ZAP Full Scan with discovered URLs  
- Integrate with other CI/CD pipelines  
- Extend workflow to other apps  

---

## 📚 References

- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)  
- [Katana Crawler](https://github.com/projectdiscovery/katana)  
- [OWASP ZAP](https://www.zaproxy.org/)  
- [GitHub Actions](https://docs.github.com/en/actions)
