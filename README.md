# DVWA-Juiceshop-Homelab
## Web App Security Home Lab — DVWA & OWASP Juice Shop

A personal home lab for practicing web application security concepts using two intentionally vulnerable applications: **Damn Vulnerable Web Application (DVWA)** and **OWASP Juice Shop**, both running locally via Docker on Kali Linux.

## Overview

This repository documents the setup, configuration, and ongoing practice of common web vulnerability classes (SQL injection, XSS, broken authentication, command injection, etc.) in a safe, isolated, legal environment.

## Environment

- **Host OS:** Kali Linux (VirtualBox VM)
- **Containerization:** Docker
- **Target Apps:**
  - [DVWA](https://github.com/digininja/DVWA) — `http://localhost:8080`
  - [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) — `http://localhost:3000`

## Setup

### Prerequisites
- Docker installed and running
- User added to the `docker` group (to run without `sudo`)

### 1. Install Docker
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

### 2. Run DVWA
```bash
docker run -d --name dvwa -p 8080:80 vulnerables/web-dvwa
```
- Visit `http://localhost:8080`
- Login: `admin` / `password`
- Click **Create / Reset Database** on first run

### 3. Run Juice Shop
```bash
docker run -d --name juiceshop -p 3000:3000 bkimminich/juice-shop
```
- Visit `http://localhost:3000`

### 4. Verify containers are running
```bash
docker ps
```

## Managing the Lab
```bash
docker stop dvwa juiceshop      # stop both
docker start dvwa juiceshop     # resume both
docker rm -f dvwa juiceshop     # remove and reset from scratch
```

## Progress / Writeups

| Challenge / Vulnerability | App | Status | Notes |
|---|---|---|---|
| Score Board (hidden page) | Juice Shop | ✅ Solved | Found via direct URL `#/score-board` |
| SQL Injection (low) | DVWA | ⬜ Not started | |
| Command Injection (low) | DVWA | ⬜ Not started | |
| XSS (Reflected) | DVWA | ⬜ Not started | |
| Broken Authentication | Juice Shop | ⬜ Not started | |

## Disclaimer

Both DVWA and OWASP Juice Shop are **intentionally vulnerable applications** created for educational and training purposes. All testing in this repository was performed against local, isolated instances running in a personal home lab — never against production systems or systems the author does not own or have explicit permission to test.

## Resources
- [DVWA GitHub](https://github.com/digininja/DVWA)
- [OWASP Juice Shop GitHub](https://github.com/juice-shop/juice-shop)
- [Pwning OWASP Juice Shop (official companion guide)](https://pwning.owasp-juice.shop/)
