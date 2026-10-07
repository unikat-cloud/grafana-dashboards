# Grafana Dashboards

Eine kuratierte, bereinigte Sammlung von Grafana-Dashboards für selbst gehostete Dienste.
Jedes Dashboard folgt derselben Hierarchie: eine auf einen Blick erfassbare **Übersicht**-Zeile
ganz oben, dann **Dienst-Metriken**, **Subsysteme** und **Laufzeit-Details** — damit der
wichtigste Zustand ohne Scrollen sichtbar ist.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Grafana](https://img.shields.io/badge/Grafana-9%2B-orange?logo=grafana)](https://grafana.com)
[![Prometheus](https://img.shields.io/badge/Prometheus-required-E6522C?logo=prometheus)](https://prometheus.io)

> **Hinweis:** Diese Dashboards sind keine fertigen Produktiv-Dashboards, die man 1:1
> übernehmen kann. Sie sind aus einem konkreten Setup entstanden und als Ausgangspunkt
> gedacht — Datenquellen, Label-Konventionen (z. B. `job`/`instance`) und
> Exporter-Versionen bitte an die eigene Umgebung anpassen und vor dem Produktiveinsatz
> testen. Nutzung auf eigene Verantwortung.

## Dashboards

| Dashboard | Quelle | Datenquelle | Highlights |
|---|---|---|---|
| [Crowdsec](Crowdsec.json)        | CrowdSec + Nginx Proxy Manager | Prometheus | Version, Log-Pipeline, PAPI, Systemressourcen |
| [GitLab](gitlab.json)            | Omnibus-/Helm-Exporter | Prometheus | 17 Bereiche: Puma, DB, Redis, GC, Sidekiq, SLO |
| [Immich](immich.json)            | immich-exporter / Prozess | Prometheus | CPU/RAM, API-HTTP, Microservices, Job-Warteschlangen |
| [Loki](loki.json)                | Grafana Alloy + journald + Docker | Loki | Log-Volumen, Fehler-Volumen, Live-Logs |
| [n8n](n8n.json)                  | n8n Prometheus-Metriken | Prometheus | Uptime, CPU/RAM, Event Loop, Workflows, Auth |
| [Nextcloud](nextcloud.json)      | nextcloud-exporter | Prometheus | Benutzer, Dateien, Freigaben, Apps |
| [OPNsense](opnsense.json)        | opnsense-exporter + node_exporter | Prometheus | Status, CPU/RAM, Netzwerk, ZFS, System-Interna |
| [Stalwart](stalwart.json)        | stalwart-mail Prometheus-Metriken | Prometheus | SMTP / IMAP / POP3 / HTTP, Sicherheit, Zustellung |
| [Uptime Kuma](uptimekuma.json)   | uptime-kuma-prometheus-exporter | Prometheus | Monitore, Zertifikate, Node.js-Laufzeit |

Alle Dashboards nutzen die Datenquelle als **Input-Variable**
(`${DS_PROMETHEUS}` bzw. `${DS_LOKI}`) und fragen beim Import danach.
OPNsense stellt zusätzlich eine `${job}`-Label-Variable bereit.

## Design-Konventionen

Jedes Dashboard folgt derselben Hierarchie von oben nach unten, damit man zwischen
Diensten wechseln kann, ohne sich neu einzuarbeiten:

1. **Übersicht** — 4–8 Stat-Panels mit Version, Uptime und Gesamtzählern.
   Die Zeile, die man zuerst ansieht.
2. **Primäre Metriken** — Anfragenrate, Antwortzeit und wichtige Zähler als
   Zeitreihen.
3. **Subsysteme** — protokoll- oder komponentenspezifische Aufschlüsselungen
   (SMTP / IMAP, Puma / Sidekiq usw.).
4. **Laufzeit & Interna** — CPU, Speicher, GC, Event Loop, Handles.
5. **Referenz** — Versionen, Zertifikatsstatus, Exporter-Infos.

Die Top-Level-Defaults sind über alle Dashboards hinweg angeglichen:

```json
{
  "editable": true,
  "graphTooltip": 1,
  "timezone": "browser",
  "style": "dark",
  "refresh": "30s",
  "time": { "from": "now-6h", "to": "now" }
}
```

Ausnahmen sind pro Dashboard dokumentiert (z. B. nutzt OPNsense
`liveNow: true` und `refresh: 10s` für Live-Firewall-Monitoring).

## Importieren

1. Grafana öffnen → **Dashboards** → **Neu** → **Importieren**.
2. Die JSON-Datei hochladen (oder ihren Inhalt einfügen).
3. Grafana fragt nach der Datenquelle. Prometheus- oder Loki-Instanz wählen.
   Bei OPNsense wird zusätzlich ggf. das `job`-Label abgefragt.
4. Mit gewünschtem Ordner und Namen speichern.

### Programmatischer Import

```bash
# Grafanas Provisioning-API verwenden
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -d @Crowdsec.json \
  https://grafana.example.com/api/dashboards/import
```

Oder mit [grafana-dashboard-json-exporter](https://github.com/grafana/grizzly):

```bash
grr apply Crowdsec.json
```

## Benötigte Exporter / Metrik-Quellen

| Dashboard   | Exporter / Quelle                                                                                  |
|-------------|----------------------------------------------------------------------------------------------------|
| Crowdsec    | `cs_*`-Metriken von CrowdSec (integriert)                                                          |
| GitLab      | [`gitlab-ci-pipelines-exporter`](https://gitlab.com/gitlab-org/ruby/gems/gitlab-exporter) + `gitlab_*`-Metriken |
| Immich      | `process_*` (node_exporter) + `immich_*` (integrierte API)                                          |
| Loki        | Grafana Alloy (Journal + Docker-Container) → Loki                                                  |
| n8n         | Interner n8n-Prometheus-Endpunkt (`/metrics`)                                                      |
| Nextcloud   | [`nextcloud-exporter`](https://github.com/xperimental/nextcloud-exporter)                          |
| OPNsense    | [`opnsense-exporter`](https://github.com/AthenaMind/opnsense-exporter) + `node_exporter`         |
| Stalwart    | Integrierte `metrics`-Direktive in `config.toml`                                                   |
| Uptime Kuma | [`uptime-kuma-prometheus-exporter`](https://github.com/inputnick/uptime-kuma-prometheus-exporter)  |

## Repository-Struktur

```
.
├── Crowdsec.json       # Grafana-Dashboard-Export
├── gitlab.json
├── immich.json
├── loki.json
├── n8n.json
├── nextcloud.json
├── opnsense.json
├── stalwart.json
├── uptimekuma.json
├── LICENSE
└── README.md
```

## Mitwirken

1. Ein Dashboard in Grafana bearbeiten → **Share** → **Export** →
   „Export for sharing externally“ (entfernt Org-/Folder-IDs).
2. Die JSON committen, pushen und einen Merge Request öffnen.

## Lizenz

[MIT](LICENSE) — der vollständige Text steht in `LICENSE`.

