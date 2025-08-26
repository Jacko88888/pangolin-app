0) Target repo layout
pangolin-app/
├─ README.md
├─ docker-compose.yml
├─ manifest.json
├─ .gitignore
├─ docs/
│  ├─ wizard-port-3000.png         # (upload screenshot)
│  └─ dashboard-dark.png           # (upload screenshot)
└─ adguard/
   ├─ conf/
   │  └─ AdGuardHome.yaml          # seed config (keeps wizard, locks UI to :3000)
   └─ work/
      └─ .gitkeep                  # keeps folder in git; runtime data stays out

1) README.md (full, polished)

Replace the entire file with this:

# 🦔 Pangolin App for ZimaOS (AdGuard Home)

A pre-packaged **AdGuard Home** container setup as a **ZimaOS** app.

AdGuard Home is a network-wide DNS blocker that removes ads & trackers at the DNS layer. Pangolin bundles a ready-to-run Docker Compose and ZimaOS manifest so you can deploy it in minutes.

![Wizard step: set port 3000](docs/wizard-port-3000.png)
![Dashboard](docs/dashboard-dark.png)

---

## 📦 Included Files

- `docker-compose.yml` — launches AdGuard Home  
- `manifest.json` — describes the app for ZimaOS (shows under Local/Custom)  
- `adguard/conf/AdGuardHome.yaml` *(optional seed)* — app config (you may create/overwrite)  
- `adguard/work/data/` — filter lists, stats, sessions (runtime data)

Data lives at:



/DATA/AppData/pangolin-app/adguard/{conf,work/}


---

## ⚙️ Requirements

- ZimaOS with Docker & Docker Compose (default on ZimaOS)
- Free ports: **53/tcp**, **53/udp** (DNS) and **3000/tcp** (Web UI recommended)
- Local IP of your ZimaOS box (e.g. `192.168.0.7`)

**Pre-flight check (safe):**
```bash
ss -lunp | grep -w ':53'   || echo "UDP 53 OK"
ss -ltnp | grep -w ':3000' || echo "TCP 3000 OK"

🚀 Quick Deploy on ZimaOS (CLI)

Installs to /DATA/AppData/pangolin-app and starts the app.

# 1) Get the app
cd /DATA/AppData
curl -L -o pangolin-app.tar.gz https://github.com/Jacko88888/pangolin-app/archive/refs/heads/main.tar.gz
mkdir -p pangolin-app
tar -xzf pangolin-app.tar.gz -C pangolin-app --strip-components=1
rm pangolin-app.tar.gz


(Recommended) Pre-seed the UI port to 3000 so the wizard can’t switch it to 80:

cat > /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml <<'YAML'
http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
YAML


Start it:

docker compose -f /DATA/AppData/pangolin-app/docker-compose.yml up -d


Open the wizard:

http://<ZIMA_IP>:3000/install.html


Wizard tips

Admin Web Interface port: 3000

DNS port: 53

Upstream DNS (examples):

tls://1.1.1.1
tls://1.0.0.1


or

tls://9.9.9.9
tls://149.112.112.112


Create your admin user & password and finish.

Test DNS (from any LAN machine):

nslookup openai.com <ZIMA_IP>
nslookup ads.google.com <ZIMA_IP>

🧭 Install via ZimaOS UI (optional)

Because manifest.json is included, ZimaOS may show pangolin-app under App Store → Local/Custom.
Installing from the UI still uses this Compose and these data paths.

🔴 Important: ZimaOS Port Note (UI on 80 vs 3000)

AdGuard’s wizard defaults the Admin Web UI to port 80.
On ZimaOS, port 80 is already used by the ZimaOS dashboard.
If you keep 80, clicking Open Dashboard will take you back to ZimaOS instead of AdGuard.

✅ Do this during the wizard (Step 2)

Set Admin Web Interface port to 3000 (keep DNS on 53).

Open AdGuard at: http://<ZIMA_IP>:3000

🛠 If you accidentally chose port 80
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

docker stop pangolin
sed -i 's/^users:.*$/users: []/' /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml
docker start pangolin
# Now visit: http://<ZIMA_IP>:3000/install.html and create a new admin account

🧪 Useful commands
# See the container & port mappings
docker ps --format "{{.Names}}\t{{.Status}}\t{{.Ports}}" | grep pangolin

# View recent logs
docker logs -n 100 pangolin

# Verify ports on the host
ss -ltnp | grep -w ':3000'
ss -lunp | grep -w ':53'

🧰 Post-install recommendations

Upstream DNS (Settings → DNS settings → Upstream servers):

tls://1.1.1.1
tls://1.0.0.1


or

tls://9.9.9.9
tls://149.112.112.112


Blocklists (Filters → DNS blocklists → Add):

AdGuard DNS filter (official)

OISD Basic

Tip: Test with a single device first — set its DNS to <ZIMA_IP> before updating your router.

💾 Backup & Restore

Backup config only (safe to store/commit):

cd /DATA/AppData/pangolin-app
tar -czf ag-config-$(date +%F-%H%M).tgz adguard/conf


Full backup (conf + work data):

cd /DATA/AppData/pangolin-app
tar -czf ag-full-$(date +%F-%H%M).tgz adguard


Restore:

docker stop pangolin
tar -xzf <your-backup>.tgz -C /DATA/AppData/pangolin-app
docker start pangolin

🧹 Uninstall
# Stop/remove the app container only (does not touch other containers)
docker compose -f /DATA/AppData/pangolin-app/docker-compose.yml down

# Optional: remove data to start fresh next time
rm -rf /DATA/AppData/pangolin-app

🔒 Security Notes

Intended for LAN use. Do not expose ports 53/3000 to the public internet.

Use a strong admin password.

Back up adguard/conf/ periodically.

🙏 Credits

AdGuard Home

Pangolin packaging by @Jacko88888

📜 License

MIT (unless you choose another license)

🧱 Advanced (optional)
1) Pin image & add healthcheck (compose hardening)

Pinned versions prevent surprise upgrades; the healthcheck gives nicer status.

# docker-compose.yml (suggested hardening)
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

2) Seed config that locks UI to :3000 but still runs the wizard

Commit this file as adguard/conf/AdGuardHome.yaml:

http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
# Intentionally no `users:` — wizard will ask and create admin

3) Make the app tile open the right URL (manifest tip)

If your manifest supports a web link, point it to port 3000 so the tile opens AdGuard directly (use the key your schema expects):

{
  "title": "Pangolin (AdGuard Home)",
  "index": "http://{host}:3000"
}


or

{
  "title": "Pangolin (AdGuard Home)",
  "webUI": "http://{host}:3000"
}

4) Optional release for fixed installs

Create a GitHub release and link a versioned tarball:

# Install a fixed version (example v1.0.0)
cd /DATA/AppData
curl -L -o pangolin-app-v1.0.0.tar.gz https://github.com/Jacko88888/pangolin-app/archive/refs/tags/v1.0.0.tar.gz
mkdir -p pangolin-app && tar -xzf pangolin-app-v1.0.0.tar.gz -C pangolin-app --strip-components=1
docker compose -f pangolin-app/docker-compose.yml up -d


---

## 2) docker-compose.yml (hardened)

> Replace your compose with this:

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

3) adguard/conf/AdGuardHome.yaml (seed)

Create this file (or overwrite it). It locks UI to :3000 but still runs the wizard.

http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
# Intentionally no `users:` — wizard will ask and create admin

4) manifest.json (point tile to :3000)

Add the field your schema supports (keep everything else you already have):

{
  "title": "Pangolin (AdGuard Home)",
  "index": "http://{host}:3000"
}


or

{
  "title": "Pangolin (AdGuard Home)",
  "webUI": "http://{host}:3000"
}

5) .gitignore (keeps runtime data out of git)

Create at repo root:

# runtime/state
adguard/work/
adguard/work/*
backup_*.tgz
ag-config-*.tgz
ag-full-*.tgz
