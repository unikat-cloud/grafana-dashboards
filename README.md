# Grafana Dashboards

A curated, sanitized set of Grafana dashboards for self-hosted services.
Every dashboard follows the same hierarchy: a glanceable **Overview** row at
the top, then **Service Metrics**, **Subsystems** and **Runtime Details** —
so the most important state is visible without scrolling.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Grafana](https://img.shields.io/badge/Grafana-9%2B-orange?logo=grafana)](https://grafana.com)
[![Prometheus](https://img.shields.io/badge/Prometheus-required-E6522C?logo=prometheus)](https://prometheus.io)

## Dashboards

| Dashboard | Source | Datasource | Highlights |
|---|---|---|---|
| [Crowdsec](Crowdsec.json)        | CrowdSec + Nginx Proxy Manager | Prometheus | Version, log pipeline, PAPI, system resources |
| [GitLab](gitlab.json)            | Omnibus / Helm chart exporters  | Prometheus | 17 sections: Puma, DB, Redis, GC, Sidekiq, SLO |
| [Immich](immich.json)            | immich-exporter / process      | Prometheus | CPU/Mem, API HTTP, microservices, job queues |
| [Loki](loki.json)                | Grafana Alloy + journald + Docker | Loki | Log volume, error volume, live logs |
| [n8n](n8n.json)                  | n8n Prometheus metrics          | Prometheus | Uptime, CPU/Mem, Event Loop, Workflows, Auth |
| [Nextcloud](nextcloud.json)      | nextcloud-exporter              | Prometheus | Users, files, shares, apps |
| [OPNsense](opnsense.json)        | opnsense-exporter + node_exporter | Prometheus | Status, CPU/RAM, network, ZFS, system internals |
| [Stalwart](stalwart.json)        | stalwart-mail Prometheus metrics | Prometheus | SMTP / IMAP / POP3 / HTTP, security, delivery |
| [Uptime Kuma](uptimekuma.json)   | uptime-kuma-prometheus-exporter | Prometheus | Monitors, certificates, Node.js runtime |

All dashboards expose their data source as an **input variable**
(`${DS_PROMETHEUS}` or `${DS_LOKI}`) and prompt the importer to pick one.
OPNsense additionally exposes a `${job}` label variable.

## Design conventions

Every dashboard in this repo follows the same top-to-bottom hierarchy so
operators can switch between services without re-learning the layout:

1. **Overview** — 4–8 stat panels with version, uptime, total counters.
   This is the row you look at first.
2. **Primary metrics** — request rate, response time, key counters as
   time series.
3. **Subsystems** — protocol- or component-specific breakdowns
   (SMTP / IMAP, Puma / Sidekiq, etc.).
4. **Runtime & internals** — CPU, memory, GC, event loop, handles.
5. **Reference** — versions, certificate status, exporter info.

Top-level defaults are aligned across all dashboards:

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

Exceptions are documented per dashboard (e.g. OPNsense uses
`liveNow: true` and `refresh: 10s` for live firewall monitoring).

## Importing

1. Open Grafana → **Dashboards** → **New** → **Import**.
2. Upload the JSON file (or paste its contents).
3. Grafana will prompt for the data source. Pick your Prometheus or Loki
   instance. For OPNsense you may also be prompted for the `job` label.
4. Save with the desired folder and name.

### Programmatic import

```bash
# Use Grafana's provisioning API
curl -X POST -H "Content-Type: application/json" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -d @Crowdsec.json \
  https://grafana.example.com/api/dashboards/import
```

Or with [grafana-dashboard-json-exporter](https://github.com/grafana/grizzly):

```bash
grr apply Crowdsec.json
```

## Required exporters / metrics sources

| Dashboard   | Exporter / source                                                                                  |
|-------------|----------------------------------------------------------------------------------------------------|
| Crowdsec    | `cs_*` metrics from crowdsec (built-in)                                                            |
| GitLab      | [`gitlab-ci-pipelines-exporter`](https://gitlab.com/gitlab-org/ruby/gems/gitlab-exporter) + `gitlab_*` metrics |
| Immich      | `process_*` (node_exporter) + `immich_*` (built-in API)                                             |
| Loki        | Grafana Alloy (journal + Docker containers) → Loki                                                 |
| n8n         | n8n internal Prometheus endpoint (`/metrics`)                                                      |
| Nextcloud   | [`nextcloud-exporter`](https://github.com/xperimental/nextcloud-exporter)                          |
| OPNsense    | [`opnsense-exporter`](https://github.com/AthenaMind/opnsense-exporter) + `node_exporter`         |
| Stalwart    | Stalwart built-in `metrics` directive in `config.toml`                                             |
| Uptime Kuma | [`uptime-kuma-prometheus-exporter`](https://github.com/inputnick/uptime-kuma-prometheus-exporter)  |

## Repository structure

```
.
├── Crowdsec.json       # Grafana dashboard export
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

## Contributing

1. Edit a dashboard in Grafana → **Share** → **Export** → **Export for
   sharing externally** (this strips org/folder IDs).
2. Commit the JSON, push, open a merge request.

## License

[MIT](LICENSE) — see `LICENSE` for the full text.
