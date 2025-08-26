🦔 Pangolin App for ZimaOS (AdGuard Home)

A pre-packaged AdGuard Home container setup as a ZimaOS app.

AdGuard Home is a network-wide DNS blocker that removes ads & trackers at the DNS layer. Pangolin bundles a ready-to-run Docker Compose and ZimaOS manifest so you can deploy it in minutes.

📦 Included Files

docker-compose.yml — launches AdGuard Home

manifest.json — describes the app for ZimaOS (shows under Local/Custom)

adguard/conf/AdGuardHome.yaml (optional seed) — app config (you may create/overwrite)

adguard/work/data/ — filter lists, stats, sessions (runtime data)

Data is stored under:
/DATA/AppData/pangolin-app/adguard/{conf,work/}

⚙️ Requirements

ZimaOS with Docker & Docker Compose (default on ZimaOS)

Free ports: 53/tcp, 53/udp (DNS) and 3000/tcp (Web UI recommended)

Local IP of your ZimaOS box (e.g. 192.168.0.7)

Quick pre-flight (safe):

root@ZimaOS:~# ss -lunp | grep -w ':53'   || echo "UDP 53 OK"
root@ZimaOS:~# ss -ltnp | grep -w ':3000' || echo "TCP 3000 OK"

🚀 Quick Deploy on ZimaOS (CLI)

This installs to /DATA/AppData/pangolin-app and starts the app.

# 1) Get the app
root@ZimaOS:~# cd /DATA/AppData
root@ZimaOS:/DATA/AppData# curl -L -o pangolin-app.tar.gz https://github.com/Jacko88888/pangolin-app/archive/refs/heads/main.tar.gz
root@ZimaOS:/DATA/AppData# mkdir -p pangolin-app
root@ZimaOS:/DATA/AppData# tar -xzf pangolin-app.tar.gz -C pangolin-app --strip-components=1
root@ZimaOS:/DATA/AppData# rm pangolin-app.tar.gz


(Recommended) Pre-seed the UI port to 3000 so the wizard can’t switch it to 80:

root@ZimaOS:/DATA/AppData# cat > pangolin-app/adguard/conf/AdGuardHome.yaml <<'YAML'
http:
  address: 0.0.0.0:3000
dns:
  bind_hosts:
    - 0.0.0.0
  port: 53
YAML


Start it:

root@ZimaOS:/DATA/AppData# docker compose -f pangolin-app/docker-compose.yml up -d


Open the wizard:

http://<ZIMA_IP>:3000/install.html


Wizard tips:

Admin Web Interface port: 3000

DNS port: 53

Upstream DNS: e.g.

tls://1.1.1.1
tls://1.0.0.1


or

tls://9.9.9.9
tls://149.112.112.112


Create your admin user + password and finish.

Test DNS (from any LAN machine):

nslookup openai.com <ZIMA_IP>
nslookup ads.google.com <ZIMA_IP>

🧭 Install via ZimaOS UI (optional)

Because manifest.json is included, ZimaOS may show pangolin-app under App Store → Local/Custom.
You can install from the UI; it still uses the same Compose and data paths.

🔴 Important: ZimaOS Port Note (UI on 80 vs 3000)

AdGuard Home’s setup wizard defaults the Admin Web UI to port 80.
On ZimaOS, port 80 is already used by the ZimaOS dashboard.
If you keep 80, clicking Open Dashboard will take you back to ZimaOS instead of AdGuard.

✅ Do this during the wizard (Step 2)

Admin Web Interface port: set to 3000 (keep DNS on 53).

Then open AdGuard at: http://<ZIMA_IP>:3000

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

🔐 “401 Unauthorized” on the dashboard?

That means no admin user exists (or it wasn’t saved). Reset users and rerun the wizard:

docker stop pangolin
sed -i 's/^users:.*$/users: []/' /DATA/AppData/pangolin-app/adguard/conf/AdGuardHome.yaml
docker start pangolin
# Visit http://<ZIMA_IP>:3000/install.html and create a new admin account

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

Point a single device to test before changing your router: set its DNS to <ZIMA_IP>.

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

🔒 Security notes

This setup is meant for LAN use. Do not expose ports 53/3000 to the public internet.

Always set a strong admin password.

Back up your adguard/conf/ periodically.

🙏 Credits

AdGuard Home

Pangolin packaging by @Jacko88888

📜 License

MIT (unless you choose another license for this repo)


