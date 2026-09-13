# acme-deploy

> **📌 Mirror-Hinweis:** Dieses Repository ist ein automatischer Spiegel.
> Die primäre Entwicklung findet auf **[git.uliana.de/DasAoD/acme-deploy](https://git.uliana.de/DasAoD/acme-deploy)** statt.
> Issues und Pull Requests bitte dort öffnen.

A modular shell script framework for automatically deploying [acme.sh](https://github.com/acmesh-official/acme.sh) certificates to multiple services and hosts after renewal.

After each successful certificate renewal, acme.sh triggers `all.sh`, which runs all deploy scripts in order and sends an HTML status report by email.

---

## Features

- Modular deploy scripts — one per service (Nginx, Technitium, Unraid, Webmin, CUPS, WGDashboard, Nginx Proxy Manager, Fritz!Box, …)
- Shared `common.sh` with reusable `log_status`, `host_reachable`, and pending-queue helpers
- Automatic status logging per component and host
- HTML email report with component/host/status table and summary
- Certificate expiry date shown in the report, plus last-run and next-scheduled-run timestamps
- Optional online-window support for hosts that aren't always reachable (skip + catch-up instead of a false failure)
- Generic unreachable-host catch-up: any module can skip an unreachable host instead of failing, queue it, and have `catchup.sh` (run on its own cron) redeploy to just that host as soon as it's back online — with its own short confirmation email

---

## Requirements

- [acme.sh](https://github.com/acmesh-official/acme.sh) already installed and managing at least one certificate
- `bash`
- A local MTA/`mail` command (e.g. `mailutils`, `bsd-mailx`, or similar) for the status report
- Depending on which modules you use: `curl`, `openssl`, `rsync`/`ssh`, `python3` (used by the NPM module for JSON parsing)

---

## Quickstart

1. Clone this repo into a working directory of your choice, e.g. next to your acme.sh installation.
2. For every module you want to use, copy the example to a real file and edit it:
   ```bash
   cp 01-nginx.sh.example 01-nginx.sh
   chmod +x 01-nginx.sh
   ```
   Replace the placeholder paths/domains (`/path/to/acme.sh   # <-- adjust`, `example.com`, …) with your actual values.
3. Do the same for `common.sh.example` → `common.sh` and `all.sh.example` → `all.sh`.
4. Remove or ignore any `.sh.example` you don't need — `all.sh` only picks up files that actually exist.
5. Register `all.sh` as the reload command for your certificate:
   ```bash
   acme.sh --install-cert -d example.com --reloadcmd "/path/to/local-deploy/all.sh"
   ```
6. Trigger a manual run to verify everything works before waiting for the next real renewal:
   ```bash
   ./all.sh
   ```
7. Optional: enable the unreachable-host catch-up by copying `catchup.sh.example` → `catchup.sh`, adjusting `MAILTO`/`FROM`, and scheduling it via cron (every 15 minutes is a good default):
   ```bash
   */15 * * * * root /path/to/local-deploy/catchup.sh >> /var/log/acme-deploy-catchup.log 2>&1
   ```

The real `*.sh` files are meant to be excluded from version control (see `.gitignore`) since they contain your actual hosts, paths, and credentials — only the `.sh.example` templates are tracked.

---

## Writing your own module

`all.sh` runs every `*.sh` file in the directory except `common.sh`, `99-mail.sh`, and `catchup.sh`, in alphabetical order — that's the entire convention. A minimal module looks like this:

```bash
#!/bin/bash
source "$(dirname "$0")/common.sh"

log_status "mycomponent myhost.example.com" "
  cp /path/to/acme.sh/example.com/fullchain.cer /somewhere &&
  systemctl reload myservice
"
```

`log_status` runs the given command, logs success/failure (or, if you set it up, a skip state) per component/host to the shared status log, and that's what ends up as a row in the email report. Use a two-digit numeric prefix (`01-`, `02-`, …) to control execution order; `98-` and `99-` are reserved for late-running and mail-sending modules respectively.

To make a host in your module catch-up-aware, guard its deploy with `host_reachable` and `queue_pending`/`dequeue_pending`, and honor `DEPLOY_FORCE_HOST` so `catchup.sh` can target just that one host:

```bash
hosts=("myhost.example.com")
if [[ -n "${DEPLOY_FORCE_HOST:-}" ]]; then
  hosts=("$DEPLOY_FORCE_HOST")
fi

for host in "${hosts[@]}"; do
  if ! host_reachable "$host" 22; then
    echo -e "mycomponent $host\t⏭ Übersprungen (nicht erreichbar)" >> /path/to/deploy-status.log
    queue_pending "05-mymodule.sh" "$host" 22
    continue
  fi

  log_status "mycomponent $host" "..."
  dequeue_pending "05-mymodule.sh" "$host"
done
```

---

## Scripts

| Script | Purpose |
|---|---|
| `all.sh` | Orchestrator — runs every module in order, then `99-mail.sh` |
| `common.sh` | Shared `log_status`, `host_reachable`, and pending-queue (`queue_pending`/`dequeue_pending`) helpers |
| `01-nginx.sh` | Deploy to a local Nginx instance |
| `02-unraid.sh` | Deploy to Unraid NAS host(s) via rsync/SSH; supports an optional online-window + catch-up cron, and the generic unreachable-host catch-up |
| `03-webmin.sh` | Deploy to local + remote Webmin instances; remote hosts support the generic unreachable-host catch-up |
| `04-wgdashboard.sh` | Deploy to a remote WGDashboard instance |
| `05-cups.sh` | Deploy to a local CUPS instance |
| `06-npm.sh` | Deploy to Nginx Proxy Manager via its API |
| `07-fritzboxen.sh` | Deploy to one or more Fritz!Box devices (converts the key to the RSA format FRITZ!OS requires) |
| `98-technitium.sh` | Deploy to Technitium DNS as a PFX bundle via SSH; remote hosts support the generic unreachable-host catch-up |
| `99-mail.sh` | Builds and sends the HTML status report |
| `catchup.sh` | Standalone, cron-driven: rechecks hosts queued as unreachable and redeploys to just those hosts once they're back online, with its own short confirmation email |

---

## Mitwirkende

Dieses Projekt wurde in Zusammenarbeit mit [Claude](https://claude.ai) (Sonnet 4.6) von [Anthropic](https://anthropic.com) entwickelt und iterativ ausgebaut.  
Der überwiegende Teil des Codes, der Architektur und der Dokumentation wurde durch KI generiert und gemeinsam verfeinert.

| Rolle | Person / Tool |
|---|---|
| Projektidee, Anforderungen & Tests | [DasAoD](https://git.uliana.de/DasAoD) |
| Code, Architektur, Dokumentation | [Claude](https://git.uliana.de/Claude) (Anthropic) |

## License

[MIT](LICENSE)
