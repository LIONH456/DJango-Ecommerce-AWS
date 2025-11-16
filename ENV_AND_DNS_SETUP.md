# .env Setup & Cloudflare DNS — Beginner's Guide

This document explains how to prepare your `.env` file for production and set up Cloudflare DNS to point to your AWS server.

---

## Part 1: Understanding Your `.env` File

Your `.env` file contains **secrets and configuration** that should never be committed to Git.

### What's in your `.env`?

```
SECRET_KEY=rnq2sk!qy7ei3@8jhxjdw=km@h$(4s)o%+3au$#a#5+bbb@id^
DEBUG=True
MONGODB_ATLAS_URI=mongodb+srv://lionh456:LVixKN2699pd2wpO@ecommercesu2.xcvh6ig.mongodb.net/...
ALLOWED_HOSTS=su12ecommerce.lionh456.uk,www.su12ecommerce.lionh456.uk,157.10.73.77,localhost,127.0.0.1
CSRF_TRUSTED_ORIGINS=https://su12ecommerce.lionh456.uk,https://www.su12ecommerce.lionh456.uk
USE_HTTPS=True
```

### What each means:

| Variable | What it does | Example | Keep secret? |
|----------|-------------|---------|-------------|
| `SECRET_KEY` | Django's encryption key for sessions/tokens | Long random string | **YES** |
| `DEBUG` | `True` = show errors (dev), `False` = hide errors (production) | `False` in production | No |
| `MONGODB_ATLAS_URI` | Connection string to your MongoDB cluster | `mongodb+srv://username:password@...` | **YES** |
| `ALLOWED_HOSTS` | Domains that can access your app | `yourdomain.com,www.yourdomain.com` | No |
| `CSRF_TRUSTED_ORIGINS` | Domains allowed to make POST requests | `https://yourdomain.com` | No |
| `USE_HTTPS` | Force HTTPS | `True` | No |
| `EMAIL_HOST_PASSWORD` | Gmail app password for sending emails | Your Gmail app password | **YES** |
| `PAYPAL_CLIENT_SECRET` | PayPal secret key | From PayPal dashboard | **YES** |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token | From BotFather | **YES** |
| `BAKONG_TOKEN` | Bakong payment token | Your Bakong account | **YES** |

---

## Part 2: Preparing `.env` for Production (AWS)

When you deploy to AWS, make these changes to your `.env`:

### 1. Change `DEBUG` to `False` (critical for production)

**Before (development):**
```
DEBUG=True
```

**After (production):**
```
DEBUG=False
```

Why? `DEBUG=True` shows sensitive error messages to anyone who visits your site. **Never use `True` in production.**

### 2. Update `ALLOWED_HOSTS` to include your domain and Elastic IP

**Before (local testing):**
```
ALLOWED_HOSTS=localhost,127.0.0.1,157.10.73.77
```

**After (AWS with domain):**
```
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com,54.123.45.67
```

Replace:
- `yourdomain.com` with your actual domain.
- `54.123.45.67` with your **Elastic IP** from AWS.

### 3. Update `CSRF_TRUSTED_ORIGINS` to match your domain

**Before:**
```
CSRF_TRUSTED_ORIGINS=https://localhost:8000,http://127.0.0.1
```

**After:**
```
CSRF_TRUSTED_ORIGINS=https://yourdomain.com,https://www.yourdomain.com
```

### 4. Keep `USE_HTTPS=True` (important for Cloudflare)

```
USE_HTTPS=True
```

This ensures Django knows it's behind HTTPS (via Cloudflare proxy).

### 5. All secrets stay the same

Your `MONGODB_ATLAS_URI`, email passwords, and API tokens don't change. Just make sure they're:
- Not shared publicly.
- Secured with `chmod 600 .env` on the server.

---

## Example Production `.env` (for AWS)

```
# Django
SECRET_KEY=rnq2sk!qy7ei3@8jhxjdw=km@h$(4s)o%+3au$#a#5+bbb@id^
DEBUG=False

# Database
MONGODB_ATLAS_URI=mongodb+srv://lionh456:LVixKN2699pd2wpO@ecommercesu2.xcvh6ig.mongodb.net/?retryWrites=true&w=majority&appName=EcommerceSU2
MONGODB_DATABASE=ECommerceSU2
MONGODB_COLLECTION=users

# Domain & HTTPS
ALLOWED_HOSTS=su12ecommerce.lionh456.uk,www.su12ecommerce.lionh456.uk,54.123.45.67
CSRF_TRUSTED_ORIGINS=https://su12ecommerce.lionh456.uk,https://www.su12ecommerce.lionh456.uk
USE_HTTPS=True

# Email (keep these secrets!)
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_HOST_USER=junhongho25@gmail.com
EMAIL_HOST_PASSWORD=upam kbcl oyjv sqcp
EMAIL_USE_TLS=True
DEFAULT_FROM_EMAIL=junhongho25@gmail.com

# Payment APIs (keep these secrets!)
PAYPAL_CLIENT_ID=AUfbZvPXNm__BzoyZU1Ox2etveau7AJRPAIHaj9zfhC3AAj-hw-9ONMUftYN4RVV2LECbLM4o9pbluCn
PAYPAL_CLIENT_SECRET=EIuJ_MdUsuczaZp__wl42yD6_5ec1Bs9oETwHX51FRTyDgjzT6eIR-OnMPcvBZ6DxkJE8K7uksU6beI_
PAYPAL_ENV=sandbox

# Bakong (keep these secrets!)
BAKONG_TOKEN=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
BAKONG_ACCOUNT=junhong_ho@aclb
BAKONG_MERCHANT_NAME=JunHong Ho
BAKONG_MERCHANT_CITY=Phnom Penh
BAKONG_PHONE=85512891258
BAKONG_STORE_LABEL=SU12-Ecommerce
BAKONG_TERMINAL_LABEL=JH

# Telegram Bot (keep these secrets!)
TELEGRAM_BOT_TOKEN=8457819519:AAHf1TVYk1MSMLXwblS_hgVOMuCODbbhkSc
TELEGRAM_CHAT_ID=977184900
```

---

## Part 3: Setting up Cloudflare DNS (step-by-step)

### What is DNS?
DNS (Domain Name System) translates domain names to IP addresses.
- You type `yourdomain.com` in the browser.
- DNS looks up the IP address (e.g., `54.123.45.67`).
- Browser connects to that IP.

### Step 1: Log into Cloudflare

1. Go to https://dash.cloudflare.com/
2. Log in with your email and password.
3. Select your domain.

### Step 2: Go to DNS Settings

1. Click the **DNS** tab (left sidebar).
2. You should see a list of DNS records (A records, MX records, etc.).

### Step 3: Add or Edit an A Record

An **A record** maps a domain name to an IPv4 address.

**Click "Add record"** and fill in:

| Field | Value | Example |
|-------|-------|---------|
| **Type** | `A` (IPv4 address) | Dropdown |
| **Name** | `@` for apex (yourdomain.com) or `www` for www.yourdomain.com | `@` |
| **IPv4 address** | Your AWS Elastic IP | `54.123.45.67` |
| **TTL** | Time-to-live (how long to cache) | `Auto` (Cloudflare decides) |
| **Proxy status** | DNS only (gray cloud) or Proxied (orange cloud) | Start with **DNS only** |

### Visual Example (Cloudflare DNS Dashboard)

```
DNS Records:
┌────────────────────────────────────────┐
│ Type │ Name │ Value          │ Status  │
├──────┼──────┼────────────────┼─────────┤
│  A   │  @   │ 54.123.45.67   │ ✓ DNS  │
│  A   │ www  │ 54.123.45.67   │ ✓ DNS  │
└────────────────────────────────────────┘
```

### Step 4: Save and Wait for DNS Propagation

1. Click **Save**.
2. DNS takes **1–5 minutes** to propagate globally (sometimes longer).
3. You can check if it's live with: `nslookup yourdomain.com` (PowerShell).

### Step 5: Test in Browser

```
https://yourdomain.com
```

If DNS is set up correctly, your browser will resolve `yourdomain.com` to your Elastic IP, and your app will load!

---

## Part 4: Cloudflare SSL/TLS Setup (Recommended)

Once your domain is pointing to your server, set up HTTPS encryption.

### Step 1: Go to SSL/TLS Settings

1. In Cloudflare, click **SSL/TLS** (left sidebar).
2. Select **"Overview"**.

### Step 2: Choose an Encryption Mode

| Mode | What it does | When to use |
|------|-------------|------------|
| **Off** | No encryption (bad, don't use) | Never |
| **Flexible** | Cloudflare → your server is NOT encrypted (not recommended) | Legacy only |
| **Full** | Cloudflare → your server uses self-signed cert (fine for production) | **Recommended for beginners** |
| **Full (Strict)** | Cloudflare → your server uses valid cert (requires extra setup) | Advanced |

**For beginners:** Choose **"Full"** (it's already set up by default).

### Step 3: Enable Auto HTTPS Rewrites (Optional)

1. In SSL/TLS, go to **"Edge Certificates"**.
2. Enable **"Always Use HTTPS"** (redirects `http://` to `https://`).
3. Enable **"Automatic HTTPS Rewrites"** (fixes mixed content).

---

## Part 5: Diagram — How It All Connects

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  Your Visitor                                                   │
│  Browser: https://yourdomain.com                                │
│  ↓                                                              │
│  (1) DNS lookup: yourdomain.com → 54.123.45.67                 │
│      (Cloudflare handles this)                                  │
│  ↓                                                              │
│  (2) Browser connects to 54.123.45.67 (your AWS Elastic IP)     │
│      Cloudflare proxy (optional, if "orange cloud")             │
│  ↓                                                              │
│  (3) AWS Security Group allows port 443 (HTTPS) traffic         │
│  ↓                                                              │
│  (4) Nginx (container) receives HTTPS request                   │
│      Nginx listens on port 80 (Cloudflare proxy handles TLS)    │
│  ↓                                                              │
│  (5) Nginx routes to Django (Gunicorn on port 8000)             │
│  ↓                                                              │
│  (6) Django app processes request (checks ALLOWED_HOSTS)        │
│  ↓                                                              │
│  (7) Response sent back through the chain                       │
│  ↓                                                              │
│  Your Visitor sees your site! ✓                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 6: Troubleshooting

### Domain doesn't resolve

**Symptom:** `This site can't be reached` or `connection refused`

**Causes & fixes:**
- DNS not propagated yet → **wait 5–10 minutes** and refresh.
- Wrong Elastic IP in Cloudflare → **copy-paste exact IP from AWS**.
- Security group doesn't allow port 80/443 → **check AWS security group settings**.
- App not running → **SSH into server, run `docker compose ps`, check if containers are up**.

### Domain resolves but shows error page

**Symptom:** Site loads but shows 502 Bad Gateway or connection error

**Causes & fixes:**
- `ALLOWED_HOSTS` doesn't include your domain → **update `.env` and restart**.
- MongoDB can't connect → **check Atlas allows your EC2 IP in network access settings**.
- Django app crashed → **check logs: `docker compose logs -f web`**.

### Cloudflare shows "SSL certificate error"

**Symptom:** Browser shows red SSL warning

**Causes & fixes:**
- SSL/TLS mode is "Flexible" → **change to "Full"** in Cloudflare.
- Using `http://` instead of `https://` → **enable "Always Use HTTPS"** in Cloudflare Edge Certificates.

---

## Part 7: Security Best Practices

1. **Keep `.env` private:**
   - Don't commit to Git (`.gitignore` handles this).
   - On server: `chmod 600 .env` (only owner can read).

2. **Use strong secrets:**
   - `SECRET_KEY`: should be a random 50+ character string.
   - Change if accidentally shared.

3. **Monitor MongoDB Atlas access:**
   - Go to Atlas > Network Access.
   - Restrict to your EC2's security group or IP (or allow anywhere with caution).

4. **Use Cloudflare security:**
   - Enable DDoS protection (free).
   - Set up firewall rules if needed.
   - Review traffic in Analytics.

5. **SSL/TLS best practice:**
   - Always use `https://` in production.
   - Use "Full" or "Full (Strict)" mode in Cloudflare.

---

## Checklist: `.env` & DNS Setup

- [ ] Changed `DEBUG=False` in `.env`.
- [ ] Updated `ALLOWED_HOSTS` to include your domain and Elastic IP.
- [ ] Updated `CSRF_TRUSTED_ORIGINS` to match your domain.
- [ ] `USE_HTTPS=True` is set.
- [ ] `.env` copied to server and secured with `chmod 600`.
- [ ] Cloudflare A record points to Elastic IP (DNS only or proxied).
- [ ] Domain resolves to your Elastic IP (`nslookup yourdomain.com`).
- [ ] Site loads on `https://yourdomain.com`.
- [ ] SSL/TLS mode set to "Full" or "Full (Strict)".
- [ ] "Always Use HTTPS" enabled in Cloudflare.

**Done! Your site is secure and live.** 🎉

