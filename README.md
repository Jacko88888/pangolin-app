# 🦔 Pangolin for ZimaOS (AdGuard Home)

[![Stars](https://img.shields.io/github/stars/Jacko88888/pangolin-app?style=flat)](https://github.com/Jacko88888/pangolin-app/stargazers)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](#-license)
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)](#-quick-deploy-on-zimaos-cli)
[![ZimaOS](https://img.shields.io/badge/Platform-ZimaOS-purple)](#-install-via-zimaos-ui-optional)

**Network-wide ad & tracker blocking via AdGuard Home, packaged as a one-file ZimaOS app.**  
Fast to deploy. Easy to back up. Safe defaults for ZimaOS.

> **Highlights**
> - Blocks ads/trackers at the DNS layer for your entire network
> - Zero-to-hero in minutes with a single `docker-compose.yml`
> - ZimaOS-friendly ports: UI on **:3000**, DNS on **:53**
> - Config & data live under `/DATA/AppData/pangolin-app/adguard/`

![Wizard step: set port 3000](docs/wizard-port-3000.png)
![Dashboard](docs/dashboard-dark.png)

---

## Table of Contents
- [Included Files](#-included-files)
- [Requirements](#-requirements)
- [Quick Deploy on ZimaOS (CLI)](#-quick-deploy-on-zimaos-cli)
- [Install via ZimaOS UI (optional)](#-install-via-zimaos-ui-optional)
- [Important: ZimaOS Port Note (UI on 80 vs 3000)](#-important-zimaos-port-note-ui-on-80-vs-3000)
- [“401 Unauthorized” on the Dashboard?](#-401-unauthorized-on-the-dashboard)
- [Useful Commands](#-useful-commands)
- [Post-install Recommendations](#-post-install-recommendations)
- [Backup & Restore](#-backup--restore)
- [Uninstall](#-uninstall)
- [Releases](#-releases)
- [Contributing](#-contributing)
- [Security](#-security)
- [License](#-license)

---

## 📦 Included Files

- `docker-compose.yml` — launches AdGuard Home  
- `manifest.json` — describes the app for ZimaOS (shows under Local/Custom)  
- `adguard/conf/AdGuardHome.yaml` *(optional seed)* — app config (you may create/overwrite)  
- `adguard/work/data/` — filter lists, stats, sessions (runtime data)

**Data lives at:**
/DATA/AppData/pangolin-app/adguard/{conf,work/}

yaml
Copy
Edit

---

## ⚙️ Requirements

- ZimaOS with Docker & Docker Compose (default on ZimaOS)
- Free ports: **53/tcp**, **53/udp** (DNS) and **3000/tcp** (Web UI)
- Your ZimaOS IP (e.g. `192.168.0.7`)

**Pre-flight check (safe):**
```bash
ss -lunp | grep -w ':53'   || echo "UDP 53 OK"
ss -ltnp | grep -w ':3000' || echo "TCP 3000 OK"
🚀 Quick Deploy on ZimaOS (CLI)
Installs to /DATA/AppData/pangolin-app and starts the app.

bash
Copy
Edit
# 1) Get the app
cd /DATA/AppData
curl -L -o pangolin-app.tar.gz https://github.com/Jacko88888/pangolin-app/archive/refs/heads/main.tar.gz
mkdir -p pangolin-app
tar -xzf pangolin-app.tar.gz -C pangolin-app --strip-components=1
rm pangolin-app.tar.gz
(Recommended) Pre-seed the UI port to 3000 so the wizard can’t switch it to 80:

bash
Copy
Edit
cat > /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml <<'YAML'
http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
YAML
Start it:

bash
Copy
Edit
docker compose -f /DATA/AppData/pangolin-app/docker-compose.yml up -d
Open the wizard:

arduino
Copy
Edit
http://<ZIMA_IP>:3000/install.html
Wizard tips

Admin Web Interface port: 3000

DNS port: 53

Upstream DNS (examples):

cpp
Copy
Edit
tls://1.1.1.1
tls://1.0.0.1
or

cpp
Copy
Edit
tls://9.9.9.9
tls://149.112.112.112
Create your admin user & password and finish.

Test DNS (from any LAN machine):

bash
Copy
Edit
nslookup openai.com <ZIMA_IP>
nslookup ads.google.com <ZIMA_IP>
🧭 Install via ZimaOS UI (optional)
Because manifest.json is included, ZimaOS may show Pangolin under App Store → Local/Custom.
Installing from the UI still uses this Compose and these data paths.

🔴 Important: ZimaOS Port Note (UI on 80 vs 3000)
AdGuard’s wizard defaults the Admin Web UI to port 80.
On ZimaOS, port 80 is already used by the ZimaOS dashboard.
If you keep 80, clicking Open Dashboard will take you back to ZimaOS instead of AdGuard.

✅ Do this during the wizard (Step 2)
Set Admin Web Interface port to 3000 (keep DNS on 53).

Open AdGuard at: http://<ZIMA_IP>:3000

🛠 If you accidentally chose port 80
bash
Copy
Edit
# 1) Stop only this app (safe)
docker stop pangolin

# 2) Change the AdGuard UI port back to 3000
nano /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml
# find:
#   http:
#     address: 0.0.0.0:80
# change to:
#   http:
#     address: 0.0.0.0:3000

# 3) Start it again
docker start pangolin

# 4) Open the UI
# http://<ZIMA_IP>:3000
🔐 “401 Unauthorized” on the Dashboard?
No admin user exists (or it wasn’t saved). Reset users and rerun the wizard:

bash
Copy
Edit
docker stop pangolin
sed -i 's/^users:.*$/users: []/' /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml
docker start pangolin
# Now visit: http://<ZIMA_IP>:3000/install.html and create a new admin account
🧪 Useful Commands
bash
Copy
Edit
# See the container & port mappings
docker ps --format "{{.Names}}\t{{.Status}}\t{{.Ports}}" | grep pangolin

# View recent logs
docker logs -n 100 pangolin

# Verify ports on the host
ss -ltnp | grep -w ':3000'
ss -lunp | grep -w ':53'
🧰 Post-install Recommendations
Upstream DNS (Settings → DNS settings → Upstream servers):

cpp
Copy
Edit
tls://1.1.1.1
tls://1.0.0.1
or

cpp
Copy
Edit
tls://9.9.9.9
tls://149.112.112.112
Blocklists (Filters → DNS blocklists → Add):

AdGuard DNS filter (official)

OISD Basic

Tip: Test with a single device first — set its DNS to <ZIMA_IP> before updating your router.

💾 Backup & Restore
Backup config only (safe to store/commit):

bash
Copy
Edit
cd /DATA/AppData/pangolin-app
tar -czf ag-config-$(date +%F-%H%M).tgz adguard/conf
Full backup (conf + work data):

bash
Copy
Edit
cd /DATA/AppData/pangolin-app
tar -czf ag-full-$(date +%F-%H%M).tgz adguard
Restore:

bash
Copy
Edit
docker stop pangolin
tar -xzf <your-backup>.tgz -C /DATA/AppData/pangolin-app
docker start pangolin
🧹 Uninstall
bash
Copy
Edit
# Stop/remove the app container only (does not touch other containers)
docker compose -f /DATA/AppData/pangolin-app/docker-compose.yml down

# Optional: remove data to start fresh next time
rm -rf /DATA/AppData/pangolin-app
🔖 Releases
Grab fixed versions from GitHub Releases and install a tagged tarball:

bash
Copy
Edit
# Example: v1.0.0
cd /DATA/AppData
curl -L -o pangolin-app-v1.0.0.tar.gz \
  https://github.com/Jacko88888/pangolin-app/archive/refs/tags/v1.0.0.tar.gz
mkdir -p pangolin-app && tar -xzf pangolin-app-v1.0.0.tar.gz -C pangolin-app --strip-components=1
docker compose -f /DATA/AppData/pangolin-app/docker-compose.yml up -d
🤝 Contributing
Issues and PRs welcome! Open an issue for bugs or feature requests.

🔐 Security
This app is intended for LAN use. Do not expose ports 53/3000 to the internet.
If you discover a security issue, please open a private report via GitHub Security Advisories.

📜 License
MIT

bash
Copy
Edit

### Bonus (compose + seed config)
If you want your repo to ship “production-ready” out of the box, include these files too:

**`docker-compose.yml` (hardened, pinned image + healthcheck):**
```yaml
services:
  pangolin:
    image: adguard/adguardhome:v0.107.65
    container_name: pangolin
    restart: unless-stopped
    ports:
      - "3000:3000/tcp"     # AdGuard UI
      - "53:53/tcp"         # DNS
      - "53:53/udp"
    volumes:
      - /DATA/AppData/pangolin-app/adguard/conf:/opt/adguardhome/conf
      - /DATA/AppData/pangolin-app/adguard/work:/opt/adguardhome/work
    healthcheck:
      test: ["CMD", "wget", "-qO", "-", "http://127.0.0.1:3000/control/status"]
      interval: 30s
      timeout: 5s
      retries: 10
adguard/conf/AdGuardHome.yaml (seed, keeps wizard but locks UI to :3000):

yaml
Copy
Edit
http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
# Intentionally no `users:` — wizard will ask and create admin
.gitignore to keep runtime junk out of git:

gitignore
Copy
Edit
# runtime/state
adguard/work/
adguard/work/*
backup_*.tgz
ag-config-*.tgz
ag-full-*.tgz
