# 🧃 OWASP Juice Shop DAST Workflow

## Overview

This repository contains a **Dynamic Application Security Testing (DAST)** workflow designed for the OWASP Juice Shop application.

The workflow fully automates:

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
sleep 20
```

- Juice Shop runs at **http://localhost:3000**
- `sleep 20` ensures the app finishes booting

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
`-H "Header: value"`  
NOT `-header`.

---

## 4️⃣ Auto‑Register & Auto‑Login

The workflow:

1. Generates random email/password
2. Registers via `/api/Users`
3. Logs in via `/rest/user/login`
4. Extracts session cookie → stores in `AUTH_COOKIE`

Example registration response:

```json
{
  "status": "success",
  "data": {
    "role": "customer",
    "id": 23,
    "email": "user123@test.com",
    "isActive": true
  }
}
```

Cookie saved in:

```
AUTH_COOKIE
```

---

## 5️⃣ Authenticated Crawling with Katana

```bash
katana \
  -u http://localhost:3000 \
  -d 5 \
  -jsl \
  -H "Cookie: $AUTH_COOKIE" \
  -silent \
  -o discovered_urls.txt
```

### Flags explained:

| Flag | Description |
|------|-------------|
| `-u` | Base URL |
| `-d 5` | Crawl depth |
| `-jsl` | JavaScript link parsing |
| `-H` | HTTP header for cookie |
| `-o` | Output file |

### URLs found include:

- Public pages → `/login`, `/register`
- Auth pages → `/profile`, `/wallet`
- Angular routes
- Challenge routes
- API endpoints → `/api/Users`, `/rest/products`

---

## 6️⃣ Upload URL List Artifact

```yaml
- name: Upload discovered URLs
  uses: actions/upload-artifact@v5
  with:
    name: discovered_urls
    path: discovered_urls.txt
```

This provides input for:

- ZAP Full Scan  
- Fuzzers  
- Manual testing  
- BurpSuite crawling seed  

---

## 📁 Example Output: discovered_urls.txt

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

## 🧠 Why API URLs Appear

Juice Shop is a **single-page Angular app**.  
All actions call APIs.

Katana extracts:

- JS strings
- fetch() calls
- xhr.open()
- Angular service URLs
- Dynamic client-side routing

Thus you see:

```
/api/Users
/api/Products
/rest/user/login
/rest/saveLoginIp
```

This is correct.

---

## 🔧 Troubleshooting

### ❗ Error: `flag provided but not defined: -header`

Fix: use `-H` instead of `-header`.

Correct form:

```bash
-H "Cookie: $AUTH_COOKIE"
```

---

### ❗ Cookie Not Working

Test:

```bash
curl -i -H "Cookie: $AUTH_COOKIE" http://localhost:3000/rest/user/whoami
```

Expected:

```
200 OK
{"user": "..."}
```

If 401 → login failed.

---

### ❗ Few URLs Found

Fixes:

- Increase crawl depth  
  ```bash
  -d 7
  ```
- Add more wait time  
  ```bash
  sleep 30
  ```
- Ensure cookie valid
- Ensure JS parsing enabled

---

## 🚀 Next Steps

- Use discovered_urls.txt → seed ZAP Full Scan  
- Add fuzzing  
- Add browser-driven injection attempts  
- Convert workflow to a general-purpose DAST tool  
- Make login paths dynamic for other apps  

---

## 📚 References

- Juice Shop → https://owasp.org/www-project-juice-shop/  
- Katana → https://github.com/projectdiscovery/katana  
- ZAP → https://www.zaproxy.org/  
- GitHub Actions → https://docs.github.com/en/actions  

---
```

