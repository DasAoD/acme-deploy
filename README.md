# acme-deploy

> **📌 Mirror-Hinweis:** Dieses Repository ist ein automatischer Spiegel.
> Die primäre Entwicklung findet auf **[git.uliana.de/DasAoD/acme-deploy](https://git.uliana.de/DasAoD/acme-deploy)** statt.
> Issues und Pull Requests bitte dort öffnen.

A modular shell script framework for automatically deploying [acme.sh](https://github.com/acmesh-official/acme.sh) certificates to multiple services and hosts after renewal.

After each successful certificate renewal, acme.sh triggers `all.sh`, which runs all deploy scripts in order and sends an HTML status report by email.

---

## Features

- Modular deploy scripts — one per service (Nginx, Technitium, Unraid, Webmin, CUPS, WGDashboard, …)
- Shared `common.sh` with a reusable `log_status` helper
- Automatic status logging per component and host
-@HUML email report with component/host/status table and summary
- Certificate expiry date shown in the report

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
