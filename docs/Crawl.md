🧃 Juice Shop DAST Workflow Documentation
Overview

This repository contains a Dynamic Application Security Testing (DAST) workflow for the OWASP Juice Shop
 application.

The goal is to:

Automatically spin up Juice Shop in a Docker container

Discover all reachable URLs (both public and authenticated)

Prepare these URLs for vulnerability scanning using tools like ZAP

Build a reusable framework for testing other web applications

Workflow Description

1️⃣ Checkout Repository

The workflow starts by checking out the repository to the GitHub Actions runner using:

uses: actions/checkout@v4


This ensures any scripts or configuration files are available to the runner.

2️⃣ Start Juice Shop in Docker

Juice Shop is run inside a Docker container:

docker run -d -p 3000:3000 --name juice-shop bkimminich/juice-shop
sleep 20


Why Docker? Provides an isolated environment.

Why wait? Juice Shop needs time to fully start before crawling.

3️⃣ Install Katana Crawler

We use Katana
 — a modern, JS-aware crawler that can:

Parse HTML and JavaScript files

Detect API endpoints

Follow dynamic routes in single-page applications (SPAs)

Installation:

wget https://github.com/projectdiscovery/katana/releases/download/v1.2.2/katana_1.2.2_linux_amd64.zip
unzip -j katana_1.2.2_linux_amd64.zip katana -d /usr/local/bin
chmod +x /usr/local/bin/katana

4️⃣ Auto-Register & Login

To maximize URL discovery, the workflow creates a new user dynamically and logs in:

Generate random email and password

Register via POST /api/Users

Login via POST /rest/user/login

Capture the authentication cookie

Why this step?

Enables crawling of authenticated-only endpoints

Avoids using admin credentials (preserves them for brute force testing)

Ensures the workflow is reusable for any web application with registration

5️⃣ Authenticated Crawling

Katana is run with the captured authentication cookie:

katana -u http://localhost:3000 -d 5 -jsl -header "Cookie: $AUTH_COOKIE" -silent -o discovered_urls.txt


-d 5 → crawl depth

-jsl → parse JavaScript links

-header → include authentication cookie

Output saved to discovered_urls.txt

What it discovers:

Front-end pages (e.g., /login, /basket, /score-board)

REST API endpoints (e.g., /api/Users, /rest/products)

Protected routes (e.g., /profile, /wallet)

This ensures near-complete coverage of the application’s attack surface.

6️⃣ Upload Discovered URLs

The output is uploaded as a GitHub artifact:

uses: actions/upload-artifact@v5
with:
  name: discovered_urls
  path: discovered_urls.txt


Ensures all discovered URLs are preserved for further analysis (e.g., feeding ZAP)

Benefits of This Approach

Reusable: Can be adapted to any SPA with login/registration

Comprehensive: Covers both public and authenticated URLs

Automated: Runs fully in GitHub Actions

Safe: Does not expose admin credentials

Next Steps

Feed discovered_urls.txt into ZAP for full DAST scanning

Enable active scanning, fuzzing, and attack-mode on key endpoints

Analyze and report vulnerabilities, including SQLi, XSS, IDOR, and logic flaws

References

https://owasp.org/www-project-juice-shop/

https://github.com/projectdiscovery/katana

https://www.zaproxy.org/docs/desktop/start/
