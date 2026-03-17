# acme-deploy

A modular shell script framework for automatically deploying [acme.sh](https://github.com/acmesh-official/acme.sh) certificates to multiple services and hosts after renewal.

After each successful certificate renewal, acme.sh triggers `all.sh`, which runs all deploy scripts in order and sends an HTML status report by email.

---

## Features

- Modular deploy scripts — one per service (Nginx, Technitium, Unraid, Webmin, CUPS, WGDashboard, …)
- Shared `common.sh` with a reusable `log_status` helper
- Automatic status logging per component and host
- HTML email report with component/host/status table and summary
- Certificate expiry date shown in the report
- Easy to extend with new services

---

## Structure

```
local-deploy/
├── common.sh.example          # Shared helper: log_status()
├── all.sh.example             # Orchestrator — runs all scripts in order
├── 01-nginx.sh.example        # Deploy to local Nginx
├── 02-unraid.sh.example       # Deploy to Unraid NAS servers (rsync/SSH)
├── 03-webmin.sh.example       # Deploy to local and remote Webmin
├── 04-wgdashboard.sh.example  # Deploy to remote WGDashboard (Nginx)
├── 05-cups.sh.example         # Deploy to local CUPS print server
├── 98-technitium.sh.example   # Deploy to Technitium DNS (PFX via SSH)
└── 99-mail.sh.example         # Send HTML deployment report by email
```

> The actual `.sh` files (containing your personal paths, hostnames and credentials)
> are excluded via `.gitignore`. Copy each `.sh.example` to `.sh` and adjust the placeholders.

Add your own scripts following the naming convention `XX-service.sh`. They are picked up automatically by `all.sh`.

---

## Setup

### 1. Clone the repository and copy the example scripts

```bash
git clone https://github.com/DasAoD/acme-deploy /path/to/acme.sh/local-deploy
cd /path/to/acme.sh/local-deploy

# Copy examples to actual scripts
for f in *.sh.example; do cp "$f" "${f%.example}"; done
chmod +x *.sh
```

### 2. Adjust placeholders

Each script contains clearly marked placeholders:

```bash
DOMAIN_DIR="/path/to/acme/domain"  # <-- adjust
HOST="server1.example.com"          # <-- adjust
```

Replace them with your actual paths and hostnames.

### 3. Create the log directory

```bash
mkdir -p /log/certuser
```

### 4. Register all.sh as acme.sh reload hook

Add to your domain config (`~/.acme.sh/your.domain/your.domain.conf`):

```bash
Le_ReloadCmd='/path/to/acme.sh/local-deploy/all.sh'
```

Or set it via acme.sh:

```bash
acme.sh --install-cert -d your.domain \
  --reloadcmd '/path/to/acme.sh/local-deploy/all.sh'
```

### 5. Configure email (99-mail.sh)

Make sure a working mail transport is available (`msmtp`, `ssmtp`, `postfix`, etc.).
Adjust `MAILTO`, `FROM` and `CERTFILE` in `99-mail.sh`.

---

## Log Format

Each deploy script writes one line per host to the status log:

```
component hostname<TAB>✅ Erfolgreich
component hostname<TAB>❌ Fehler
```

Example:
```
nginx   server1.example.com     ✅ Erfolgreich
unraid  nas1.example.com        ✅ Erfolgreich
unraid  nas2.example.com        ❌ Fehler
webmin  server1.example.com     ✅ Erfolgreich
```

`99-mail.sh` reads this file and generates the HTML report.

---

## Manual Run

```bash
/path/to/acme.sh/local-deploy/all.sh
```

With debug output:

```bash
bash -x /path/to/acme.sh/local-deploy/all.sh 2>&1 | tee /tmp/deploy-debug.log
```

---

## Supported Services

| Script | Service | Method |
|--------|---------|--------|
| `01-nginx.sh` | Nginx (local) | File copy + reload |
| `02-unraid.sh` | Unraid NAS | Bundle PEM + rsync/SSH |
| `03-webmin.sh` | Webmin (local + remote) | Bundle PEM + rsync/SSH |
| `04-wgdashboard.sh` | WGDashboard (remote Nginx) | rsync/SSH |
| `05-cups.sh` | CUPS (local) | File copy + restart |
| `98-technitium.sh` | Technitium DNS | PFX via SSH + openssl |

---

## License

[MIT](LICENSE)
