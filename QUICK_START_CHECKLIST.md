# Quick-Start Checklist: Deploy Your Django App to AWS EC2 (Free Tier)

Use this checklist to follow along step-by-step. Check off each box as you complete it.

---

## Phase 1: AWS Setup (5–10 minutes)

- [ ] **1.1** Go to https://aws.amazon.com/ and click "Create AWS Account"
  - Use your email and create a password.
  - Add billing information (required, but won't charge during free tier).
  - Verify your email.

- [ ] **1.2** Log in to AWS Console: https://console.aws.amazon.com/

- [ ] **1.3** Go to **EC2** (search "EC2" in the top search bar).

- [ ] **1.4** Click **"Launch instances"** (big blue button).

- [ ] **1.5** Name your instance (e.g., "ecommerce-app").

- [ ] **1.6** Choose **Amazon Machine Image (AMI):**
  - Search for "Ubuntu 22.04 LTS"
  - Select it (make sure it says "Eligible for free tier").

- [ ] **1.7** Choose instance type: **t2.micro** (free tier eligible).

- [ ] **1.8** Network settings: click **"Create security group"**
  - Name it: "ecommerce-security-group"
  - Add these inbound rules:
    - **SSH** (port 22): Source = "My IP" (your computer's IP, for security).
    - **HTTP** (port 80): Source = "Anywhere (0.0.0.0/0)".
    - **HTTPS** (port 443): Source = "Anywhere (0.0.0.0/0)".

- [ ] **1.9** Storage: leave default (8 GB) — plenty for your app.

- [ ] **1.10** Key pair:
  - Click **"Create new key pair"**
  - Name it: "my-aws-key"
  - Choose `.pem` format (for Mac/Linux/PowerShell).
  - **Download the file and keep it safe** — you'll need it to SSH.

- [ ] **1.11** Click **"Launch instance"** (blue button).

- [ ] **1.12** Wait for the instance to be in **"Running"** state (shows green checkmark). This takes ~1 minute.

---

## Phase 2: Allocate Elastic IP (2–3 minutes)

- [ ] **2.1** In AWS Console, go to **EC2 > Elastic IPs** (left sidebar).

- [ ] **2.2** Click **"Allocate Elastic IP address"**.

- [ ] **2.3** Click **"Allocate"** (leave defaults).

- [ ] **2.4** Select the Elastic IP you just created, then click **"Associate Elastic IP address"**.

- [ ] **2.5** Choose:
  - Instance: your "ecommerce-app" instance.
  - Private IP: leave default.
  - Click **"Associate"**.

- [ ] **2.6** **Copy the Elastic IP address** (you'll need it for DNS later). Example: `54.123.45.67`

---

## Phase 3: SSH into the Server from Windows (3–5 minutes)

- [ ] **3.1** Open **PowerShell** on your Windows machine.

- [ ] **3.2** Navigate to where you downloaded the key:
  ```powershell
  cd C:\Users\YOUR_USERNAME\Downloads
  ```

- [ ] **3.3** Set key permissions (PowerShell command):
  ```powershell
  chmod 600 .\my-aws-key.pem
  ```
  (If `chmod` doesn't work, right-click the file > Properties > Advanced > uncheck "Read-only".)

- [ ] **3.4** SSH into the server:
  ```powershell
  ssh -i my-aws-key.pem ubuntu@YOUR_ELASTIC_IP
  ```
  Replace `YOUR_ELASTIC_IP` with the IP from Phase 2.6.

- [ ] **3.5** Type **"yes"** when asked "Are you sure you want to continue connecting?"

- [ ] **3.6** You should now see a terminal prompt like:
  ```
  ubuntu@ip-172-xxx-xxx-xxx:~$
  ```
  (You're now inside the server!)

---

## Phase 4: Install Docker on the Server (3–5 minutes)

- [ ] **4.1** Copy and paste this entire block into your terminal:
  ```bash
  sudo apt update
  sudo apt install -y ca-certificates curl gnupg lsb-release
  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt update
  sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin
  sudo usermod -aG docker $USER
  ```

- [ ] **4.2** Log out and back in to apply Docker group permissions:
  ```bash
  exit
  ```
  (PowerShell: SSH back in)
  ```powershell
  ssh -i .\my-aws-key.pem ubuntu@YOUR_ELASTIC_IP
  ```

- [ ] **4.3** Verify Docker is installed:
  ```bash
  docker --version
  docker compose version
  ```
  You should see version numbers. If yes, Docker is ready!

---

## Phase 5: Transfer Your Project to the Server (5–10 minutes)

**Option A: Using Git (recommended if your repo is on GitHub)**

- [ ] **5A.1** On your local machine, push your project to GitHub (if you haven't):
  ```powershell
  cd C:\path\to\your\project
  git add .
  git commit -m "Deploy to AWS"
  git push origin main
  ```

- [ ] **5A.2** On the server (in your SSH terminal), clone the repo:
  ```bash
  cd ~
  git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
  cd YOUR_REPO_NAME
  ```

**Option B: Using scp (if you prefer direct file copy)**

- [ ] **5B.1** On your local PowerShell:
  ```powershell
  scp -i C:\Users\YOUR_USERNAME\Downloads\my-aws-key.pem -r C:\path\to\your\project ubuntu@YOUR_ELASTIC_IP:~/ecommerce_app
  ```

- [ ] **5B.2** On the server:
  ```bash
  cd ~/ecommerce_app
  ```

---

## Phase 6: Place `.env` on the Server (2–3 minutes)

- [ ] **6.1** On your local PowerShell, copy your `.env` to the server:
  ```powershell
  scp -i C:\Users\YOUR_USERNAME\Downloads\my-aws-key.pem C:\path\to\your\.env ubuntu@YOUR_ELASTIC_IP:~/ecommerce_app/.env
  ```

- [ ] **6.2** On the server, verify `.env` is there:
  ```bash
  ls -la .env
  ```
  (You should see the file listed.)

- [ ] **6.3** Secure the `.env` file:
  ```bash
  chmod 600 .env
  ```

---

## Phase 7: Build and Run Docker Compose (10–15 minutes)

- [ ] **7.1** On the server, build the Docker images:
  ```bash
  docker compose build --pull
  ```
  (This downloads the Python base image and installs your requirements. Takes a few minutes.)

- [ ] **7.2** Run Django migrations:
  ```bash
  docker compose run --rm web python manage.py migrate --no-input
  ```

- [ ] **7.3** Collect static files:
  ```bash
  docker compose run --rm web python manage.py collectstatic --no-input
  ```

- [ ] **7.4** Start the services in the background:
  ```bash
  docker compose up -d
  ```

- [ ] **7.5** Check if containers are running:
  ```bash
  docker compose ps
  ```
  You should see `web` and `nginx` with status "Up".

- [ ] **7.6** View logs to check for errors:
  ```bash
  docker compose logs --tail 50
  ```
  (If all is well, you'll see Gunicorn and Nginx starting without errors.)

---

## Phase 8: Test the App (1 minute)

- [ ] **8.1** On your local machine, open a browser and visit:
  ```
  http://YOUR_ELASTIC_IP
  ```
  (Replace with your actual Elastic IP from Phase 2.6.)

- [ ] **8.2** You should see your Django app's homepage!
  - If it doesn't load, check logs: `docker compose logs -f web`.
  - Common issues: `.env` is missing, `MONGODB_ATLAS_URI` has typos, or security group isn't configured.

---

## Phase 9: Point Your Domain to the Server (5 minutes)

- [ ] **9.1** Log into your **Cloudflare account**.

- [ ] **9.2** Go to your domain's **DNS** section.

- [ ] **9.3** Create/edit an **A record**:
  - **Name:** `@` (or `www` if you want just www subdomain).
  - **IPv4 address:** Your Elastic IP (from Phase 2.6).
  - **TTL:** Auto.
  - **Proxy status:** Start with **DNS only** (gray cloud).

- [ ] **9.4** Click **Save**.

- [ ] **9.5** Wait 1–5 minutes for DNS to propagate.

- [ ] **9.6** In your browser, visit your domain:
  ```
  https://yourdomain.com
  ```
  (It should load your app!)

---

## Phase 10: Optional — Enable HTTPS with Cloudflare (5 minutes)

- [ ] **10.1** In Cloudflare, go to **SSL/TLS**.

- [ ] **10.2** Set **SSL/TLS encryption mode** to **"Full"** or **"Full (strict)"** if you want end-to-end encryption.
  - "Full" = Cloudflare encrypts to your server (fine for production).
  - "Full (strict)" = requires a valid certificate on your server (advanced).

- [ ] **10.3** Optionally, set **"Automatic HTTPS Rewrites"** to on (redirects HTTP to HTTPS).

- [ ] **10.4** Test: visit `http://yourdomain.com` — should redirect to `https://`.

---

## Phase 11: Maintenance (ongoing)

- [ ] **11.1** Set up AWS billing alerts:
  - Go to **Billing Dashboard** > **Billing Alerts** > set threshold (e.g., $1/month).

- [ ] **11.2** Every month, check the AWS console to ensure you're in free tier:
  - EC2 > Instances: should show **"t2.micro"** instance running.
  - Elastic IPs: associated (no charge).

- [ ] **11.3** To update your app later:
  ```bash
  git pull origin main
  docker compose build
  docker compose up -d
  ```

- [ ] **11.4** To view logs anytime:
  ```bash
  docker compose logs -f
  ```

- [ ] **11.5** To stop the app (if needed):
  ```bash
  docker compose down
  ```

---

## Troubleshooting Quick Reference

| Problem | Solution |
|---------|----------|
| Can't SSH in | Check security group allows port 22 from your IP. |
| Docker not found | Did you `exit` and SSH back in after `sudo usermod`? |
| `.env` not loading | Check it's in the project root; run `cat .env` to verify. |
| App won't start | Run `docker compose logs -f web` to see errors. |
| Port 80 not responding | Check `docker compose ps` — are containers running? |
| Domain not resolving | Wait a few minutes; DNS takes time. Check Cloudflare DNS settings. |
| MongoDB connection fails | Verify `MONGODB_ATLAS_URI` in `.env` is correct; check Atlas allows your EC2 IP. |

---

## Final Checklist

- [ ] AWS account created ✓
- [ ] EC2 instance running ✓
- [ ] Elastic IP allocated ✓
- [ ] SSH key downloaded and secure ✓
- [ ] Connected to server via SSH ✓
- [ ] Docker installed on server ✓
- [ ] Project files copied to server ✓
- [ ] `.env` on server and secured ✓
- [ ] Docker Compose built and running ✓
- [ ] App loads on `http://Elastic_IP` ✓
- [ ] Domain DNS points to Elastic IP ✓
- [ ] App loads on `https://yourdomain.com` ✓
- [ ] Billing alerts set up ✓

**Congratulations! Your app is live on AWS free tier!**

---

## Next Steps

- Monitor logs: `docker compose logs -f`
- Update code: `git pull && docker compose build && docker compose up -d`
- Scale later if needed (within free tier limits).

