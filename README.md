# Fashion E‑Commerce (Django + MongoDB)

Production‑ready e‑commerce web app built with Django. It features a modern UI, user authentication, MongoDB for store data, AJAX UX for addresses/cart, and email + payments (Bakong KHQR and PayPal Sandbox). Deployable on a VPS with Docker or on PaaS.

## 🚀 Deploy to AWS Free Tier

Ready to deploy? See **[START_HERE.md](START_HERE.md)** for a complete beginner-friendly deployment guide with step-by-step checklists.

**Key files:**
- `START_HERE.md` — **Begin here!** Navigation and overview
- `QUICK_START_CHECKLIST.md` — Follow this during deployment (11 phases, all commands)
- `AWS_vs_VPS_GUIDE.md` — Understand why AWS works for your Docker setup
- `README_DEPLOYMENT.md` — Complete deployment summary

Your app is production-ready and works identically on AWS EC2, your current VPS, or any Docker-capable server. **Zero code changes needed!**

## Highlights
- Storefront (SSR templates) with product list/detail, cart, checkout, order pages
- Auth: register, login, profile, address book (AJAX add/edit/delete)
- Orders: create, pay later, pay by Bakong (QR), or PayPal Sandbox
- Emails: order placed, payment received, order cancelled (HTML templates)
- Admin: custom dashboard at `/ecadmin/`
- API layer (DRF) backing the SSR views; MongoDB used for products/orders/addresses

## Tech Stack
- Django 5, Django REST Framework, WhiteNoise
- MongoDB (local or Atlas) via `pymongo`
- Bootstrap UI (Shionhouse‑based) + vanilla JS (AJAX)
- SMTP (Gmail) for transactional emails
- PayPal JS SDK + server capture; Bakong KHQR via `bakong-khqr`

## Key Features (Details)
- Addresses: fully AJAX in profile; checkout address select auto‑fills and persists
- Checkout: requires selecting a saved address; T&C validation; login gating
- Payments:
  - Bakong: dynamic QR generation, MD5 tracking, status polling to complete order
  - PayPal: SDK button and redirect fallback flows; server‑side order capture
- Emails: styled HTML with CTA link back to order detail

## Project Structure
```
ECommerce/
├── ECommerce/          # Django settings, urls, wsgi/asgi
├── main/               # Business logic, MongoDB manager, views, APIs
├── dashboard/          # Custom admin dashboard
├── templates/          # SSR templates (Home, store, auth, emails)
├── static/             # Dev static assets
├── staticfiles/        # Collected static (prod)
├── requirements.txt    # Python deps
├── render.yaml         # Optional Render.com deployment
└── build.sh            # Build script (collectstatic, migrate)
```

## Local Development
1) Setup
```bash
python -m venv venv
venv\Scripts\activate   # Windows
pip install -r requirements.txt
```

2) Environment (.env)
```
SECRET_KEY=your_django_secret
DEBUG=true
ALLOWED_HOSTS=localhost,127.0.0.1

# MongoDB (pick one)
MONGODB_HOST=localhost
MONGODB_PORT=27017
MONGODB_DATABASE=ECommerceSU2
# or Atlas
MONGODB_ATLAS_URI=mongodb+srv://user:pass@cluster/db?retryWrites=true&w=majority

# Email (Gmail SMTP)
EMAIL_BACKEND=django.core.mail.backends.smtp.EmailBackend
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your_gmail@gmail.com
EMAIL_HOST_PASSWORD=your_app_password
DEFAULT_FROM_EMAIL=your_gmail@gmail.com

# PayPal
PAYPAL_CLIENT_ID=your_sandbox_client_id
PAYPAL_CLIENT_SECRET=your_sandbox_secret
PAYPAL_ENV=sandbox
```

3) Run
```bash
python manage.py migrate
python manage.py collectstatic --no-input
python manage.py runserver
```
Visit http://127.0.0.1:8000/

## Payments
- PayPal Sandbox
  - Use a funded Sandbox Buyer (Personal) account in the popup/redirect
  - Server captures the order and sets status/payment_method=PayPal
- Bakong KHQR
  - Requires a valid Bakong developer token and account
  - Generates QR + MD5; polling confirms payment and completes the order

## Email
- Uses Gmail SMTP (App Password required). Emails are sent on:
  - Order placed
  - Payment received
  - Order cancelled
- Templates: `templates/emails/*.html`

## Docker (VPS) Quick Start
Create these files at repo root and deploy with Docker Compose.

Dockerfile (Gunicorn + collectstatic) and docker-compose.yml (web, mongo, nginx) are recommended. See below minimal flow:
```bash
docker compose build
docker compose up -d
```
Then point a free subdomain (e.g., DuckDNS) to your VPS IP, optionally add Caddy/Let’s Encrypt for HTTPS.

## Production Settings Checklist
- DEBUG=False, proper ALLOWED_HOSTS and CSRF_TRUSTED_ORIGINS
- STATIC collected and served (WhiteNoise or Nginx)
- MongoDB Atlas URI set (or Docker Mongo volume)
- SMTP + PayPal env configured

## Troubleshooting (Common)
- PayPal popup shows generic error: ensure Sandbox Buyer (Personal) with funds, disable blockers, totals ≥ 0.01
- Email not received: verify Gmail App Password, DEFAULT_FROM_EMAIL, and server logs (exceptions enabled)
- Address dropdown not filling: event delegation is used; make sure you select a saved address

## License
Educational project. Third‑party assets retain their own licenses.

