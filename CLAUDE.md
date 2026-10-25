# acme-deploy – Projektkontext für Claude Code

- Versioniert sind nur die `*.sh.example`-Templates. Die echten `*.sh` (Hosts, Pfade, Zugangsdaten) stehen bewusst in `.gitignore`: nie committen, bei Modul-Änderungen prüfen, ob Template oder Produktiv-Version gemeint ist.
- Module: zweistelliges Prefix steuert die Reihenfolge (`98-`/`99-` reserviert für spät laufende Module und das Mail-Modul). Logging über `log_status "component host" "command"` aus `common.sh`.
