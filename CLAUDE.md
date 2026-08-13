# acme-deploy – Projektkontext für Claude Code

Modulares Shell-Script-Framework zum automatischen Deployment von [acme.sh](https://github.com/acmesh-official/acme.sh)-Zertifikaten auf mehrere Dienste und Hosts nach jeder Erneuerung.

## Tech-Stack
- Reines Bash, kein Framework
- Läuft auf dns1, getriggert von acme.sh nach Zertifikatserneuerung (`--reloadcmd`)

## Konzept
`all.sh` ist der Orchestrator: führt jedes `*.sh`-Modul im Verzeichnis (außer `common.sh` und `99-mail.sh`) in alphabetischer Reihenfolge aus, danach `99-mail.sh` für den HTML-Statusreport per E-Mail.

## ⚠️ Wichtig: echte Skripte sind NICHT im Git-Repo
- Nur `*.sh.example`-Templates sind versioniert.
- Die echten `*.sh`-Dateien (mit tatsächlichen Hosts, Pfaden, Zugangsdaten) sind bewusst über `.gitignore` ausgeschlossen.
- **Niemals versehentlich echte `.sh`-Dateien mit Credentials committen** – bei Änderungen an Modulen immer prüfen, ob es sich um die Template- (`.example`) oder die produktive Version handelt.

## Modul-Konvention
- Zweistelliges numerisches Prefix (`01-`, `02-`, ...) steuert Ausführungsreihenfolge
- `98-` und `99-` sind reserviert für spät laufende Module bzw. das Mail-Modul
- Jedes Modul nutzt `log_status "component host" "command"` aus `common.sh` für Logging (Erfolg/Fehler/Skip) → landet als Zeile im E-Mail-Report

## Vorhandene Module
| Skript | Zweck |
|---|---|
| `all.sh` | Orchestrator |
| `common.sh` | `log_status`-Helper |
| `01-nginx.sh` | Lokale Nginx-Instanz |
| `02-unraid.sh` | Unraid-NAS-Hosts via rsync/SSH – unterstützt optionales Online-Window + Catch-up-Cron für nicht immer erreichbare Hosts |
| `03-webmin.sh` | Lokale + remote Webmin-Instanzen |
| `04-wgdashboard.sh` | Remote WGDashboard-Instanz |
| `05-cups.sh` | Lokale CUPS-Instanz |
| `06-npm.sh` | Nginx Proxy Manager via API |
| `07-fritzboxen.sh` | Fritz!Box-Geräte (RSA-Key-Konvertierung für FRITZ!OS) |
| `98-technitium.sh` | Technitium DNS als PFX-Bundle via SSH |
| `99-mail.sh` | HTML-Statusreport bauen und versenden |

## Report
HTML-E-Mail mit Component/Host/Status-Tabelle, Zertifikats-Ablaufdatum, Last-Run- und Next-Scheduled-Run-Timestamps.

## Testen
```bash
./all.sh
```
Manueller Testlauf vor dem Warten auf die nächste echte Erneuerung.
