# CLAUDE.md — AI Assistant Guide for c1rus/uptime

This file provides context for AI assistants working in this repository.

## Repository Overview

This is an **[Upptime](https://upptime.js.org)** instance — a GitHub-powered, zero-code uptime monitoring system. It monitors **111 websites and services** (primarily Slovakia/Central Europe-based) and publishes a public status page via GitHub Pages.

There is **no custom application code** in this repository. All logic is provided by the `upptime/uptime-monitor` GitHub Action. The repository is entirely **configuration- and data-driven**.

**Owner**: c1rus
**Repo**: c1rus/uptime
**Status page base URL**: `/uptime`
**Upptime version**: v1.41.0

---

## Repository Structure

```
.
├── .upptimerc.yml          # ONLY file you should edit — all configuration lives here
├── .github/
│   └── workflows/          # Auto-generated CI workflows (DO NOT edit manually)
│       ├── uptime.yml          # Runs every 5 min — checks all endpoints
│       ├── response-time.yml   # Runs daily 23:00 UTC — calculates response metrics
│       ├── graphs.yml          # Runs daily 00:00 UTC — generates PNG graphs
│       ├── summary.yml         # Runs daily 00:00 UTC — updates README
│       ├── site.yml            # Runs daily 01:00 UTC — builds + deploys status page
│       ├── setup.yml           # Triggered on .upptimerc.yml changes — full re-setup
│       ├── update-template.yml # Runs daily — keeps workflow files up to date
│       └── updates.yml         # Additional update automation
├── api/                    # Auto-generated — static JSON badge endpoints per site
│   └── {site-name}/
│       ├── response-time-{day,week,month,year}.json
│       ├── response-time.json
│       ├── uptime-{day,week,month,year}.json
│       └── uptime.json
├── graphs/                 # Auto-generated — response time PNG graphs per site
│   └── {site-name}/
│       └── response-time-{day,week,month,year}.png
├── history/                # Auto-generated — YAML status snapshots per site + summary
│   ├── {site-name}.yml         # Per-site status history
│   └── summary.json            # Full aggregated status for all 111 sites (7000+ lines)
├── assets/
│   └── upptime-icon.svg    # Status page logo (teal Upptime icon)
├── LICENSE                 # MIT License
└── README.md               # Auto-generated — DO NOT edit manually
```

---

## The One File to Edit: `.upptimerc.yml`

All configuration changes must be made in `.upptimerc.yml`. **GitHub Actions workflow files are auto-generated and will be overwritten by `update-template` runs.**

### Key Configuration Sections

```yaml
owner: c1rus          # GitHub username
repo: uptime          # GitHub repository name
skipDeleteIssues: true # Don't delete GitHub Issues when sites recover

sites:
  - name: <slug>       # Short identifier (used as directory name in api/, graphs/, history/)
    url: <url>         # Full URL to monitor (HTTP or HTTPS)

status-website:
  baseUrl: /uptime     # GitHub Pages path (no custom domain configured)
  logoUrl: <svg-url>
  name: Upptime
  navbar:
    - title: Status
      href: /
    - title: GitHub
      href: https://github.com/c1rus/uptime
```

### Adding a New Site

Add an entry under `sites:` in `.upptimerc.yml`:

```yaml
- name: my-site        # Must be a unique slug; becomes folder name in api/, graphs/, history/
  url: https://my-site.example.com
```

Pushing this change to `master` triggers the `setup.yml` workflow which runs the full re-initialization pipeline automatically.

### Removing a Site

Remove the entry from `sites:` in `.upptimerc.yml`. Set `skipDeleteIssues: true` (already set) to preserve any open GitHub Issues for the site.

### Advanced Per-Site Options

```yaml
sites:
  - name: my-site
    url: https://my-site.example.com
    check: http              # "http" (default), "tcp-ping", "ws", "ssl"
    method: GET              # HTTP method (default: GET)
    port: 443                # custom port for tcp-ping checks
    expectedStatusCodes:     # accepted HTTP codes (default: [200])
      - 200
      - 201
    maxResponseTime: 5000    # ms threshold before flagging as degraded
    maxRedirects: 10         # max redirects to follow
    headers:                 # custom request headers
      Authorization: "Bearer TOKEN"
    body: '{"key":"value"}' # request body (for POST etc.)
    ipv6: true               # test over IPv6
    type: globalping         # use distributed Globalping network (see below)
    location: "Western Europe" # Globalping probe location filter
```

### New Features in Recent Versions (v1.37.0 – v1.41.0)

**v1.41.0** (September 2025) — *current version*
- **Globalping support**: Monitor sites from distributed global probes instead of GitHub runners. Add `type: globalping` (and optionally `location:`) to any site entry.
- **TCP-ping improved**: Better error handling and debug output to reduce false positives.

**v1.40.0** (April 2025)
- **SSL certificate checker**: Add `check: ssl` to a site entry. The check fails when the certificate expires within 7 days, enabling proactive cert renewal alerts.

**v1.39.0** (February 2025)
- **Gotify notifications**: Push alerts to a self-hosted Gotify server.

**v1.38.0** (August 2024)
- **Custom webhook notifications**: Send outage/recovery events to any HTTP endpoint.
- **GitHub Pages action upgraded** to v4.

**v1.37.0** (June 2024)
- **Multiple Telegram channels**: List multiple Telegram entries in `notifications` to fan-out alerts.

### Notification Configuration

Add a `notifications:` section to `.upptimerc.yml` to enable alerts. Currently **no notifications are configured** in this repo.

```yaml
notifications:
  # Slack
  - type: slack
    channel: C123ABC456
    oauth_token: $SLACK_OAUTH_TOKEN

  # Telegram (multiple channels supported since v1.37.0)
  - type: telegram
    bot_key: $TELEGRAM_BOT_KEY
    chat_id: $CHAT_ID_1
  - type: telegram
    bot_key: $TELEGRAM_BOT_KEY
    chat_id: $CHAT_ID_2

  # Custom webhook (since v1.38.0)
  - type: webhook
    url: https://my-webhook.example.com/upptime

  # Gotify (since v1.39.0)
  - type: gotify
    url: https://gotify.example.com
    token: $GOTIFY_TOKEN
```

Secrets referenced with `$VAR` syntax are read from GitHub repository secrets.

### SSL Certificate Monitoring Example

To monitor SSL expiry for a site (alerts if cert expires within 7 days):

```yaml
sites:
  - name: hint-ssl
    url: https://hint.hollen.sk/
    check: ssl
```

### Globalping Monitoring Example

To monitor from distributed global network probes:

```yaml
sites:
  - name: hint-global
    url: https://hint.hollen.sk/
    type: globalping
    location: "Western Europe"
```

### See Also

- [Upptime configuration docs](https://upptime.js.org/docs/configuration) — full reference
- [uptime-monitor releases](https://github.com/upptime/uptime-monitor/releases) — full changelog

---

## Data Model

### `history/{site-name}.yml`

Per-site status file, auto-updated every 5 minutes:

```yaml
url: https://hint.hollen.sk/
status: up              # "up" or "down"
code: 200               # HTTP response code
responseTime: 878       # in milliseconds
lastUpdated: 2026-02-19T23:08:35.839Z
startTime: 2025-02-02T10:02:07.452Z
generator: Upptime
```

### `history/summary.json`

7000+ line JSON array with aggregated metrics for all 111 sites. Fields per entry:
- `name`, `url`, `icon`
- `status` — current status string
- `uptime`, `uptimeDay`, `uptimeWeek`, `uptimeMonth`, `uptimeYear` — percentage strings
- `time`, `timeDay`, `timeWeek`, `timeMonth`, `timeYear` — average response times (ms)
- `dailyMinutesDown` — per-day outage minutes map

### `api/{site-name}/*.json`

Shields.io-compatible badge JSON:

```json
{ "label": "uptime 24h", "message": "100%", "color": "brightgreen" }
{ "label": "response time 24h", "message": "878 ms", "color": "brightgreen" }
```

---

## GitHub Actions Workflows

### Automation Schedule

| Workflow | Schedule | Trigger | Command |
|---|---|---|---|
| Uptime CI | Every 5 min | cron + dispatch + manual | `update` |
| Response Time CI | Daily 23:00 UTC | cron + dispatch | `response-time` |
| Graphs CI | Daily 00:00 UTC | cron + dispatch | `graphs` |
| Summary CI | Daily 00:00 UTC | cron + dispatch | `readme` |
| Static Site CI | Daily 01:00 UTC | cron + dispatch | `site` + deploy |
| Update Template CI | Daily 00:00 UTC | cron + dispatch | `update-template` |
| Setup CI | On `.upptimerc.yml` push | push + dispatch | full pipeline |

### Authentication

All workflows use `GH_PAT` (GitHub Personal Access Token) stored in repository secrets, falling back to `github.token`. The PAT is required for creating/updating GitHub Issues and pushing data commits.

### Important: Do Not Edit Workflow Files

The header in each workflow file states:

```
# Do not edit this file directly!
# Your changes will be overwritten when the Upptime template updates (by default, weekly)
# Instead, change .upptimerc.yml configuration and the workflows will be generated accordingly.
```

### Deployment

The status page is built by Sapper (Svelte) and deployed to GitHub Pages from `site/status-page/__sapper__/export/`. Deployment uses `peaceiris/actions-gh-pages@v4` with an `Upptime Bot` commit identity.

---

## Monitored Sites

111 sites are monitored, primarily in Slovakia and Central Europe, including:

- **IT/SaaS**: `hint` (hint.hollen.sk), `admin.metriq`, `input.metriq`, `view.metriq`, `master.kabernet.sk`, `kabernet2`, `rackscale`, `asseco-erp`, `qasida`
- **Education/Government**: `zskuliskova`, `zsnevadzova`, `sizp`, `digizrucnosti`, `dubravkaintranet`, `garancnyfond`
- **Real Estate**: `archdom`, `arkadyhof`, `baoffice`, `baiahouse`, `bellavista`, `livinn`, `pbproject`, `lamac`, `lamacan`
- **Retail/E-commerce**: `1sjs`, `priklady.eu`, `pomocky`, `makas`, `vreckovynoz`, `thepop`
- **Environmental/Waste**: `odpadovykalendar`, `mujodpad`, `vylozsmeti`, `myfccservices`, `fcc-group`, `fcc30`, `zistersdorf`
- **Other services**: Restaurants, architecture firms, tourism, healthcare, logistics, media

Each site's slug in `.upptimerc.yml` maps directly to folder names in `api/`, `graphs/`, and `history/`.

---

## Key Conventions

1. **Never manually edit `README.md`** — it is auto-regenerated by the `summary` workflow.
2. **Never manually edit workflow files** — they are auto-regenerated by `update-template`.
3. **All configuration changes go in `.upptimerc.yml` only.**
4. **Auto-generated commits use `[skip ci]`** in their commit message to avoid infinite loops.
5. **Site slugs (names) must be unique** and are used as directory names — avoid special characters.
6. **Data directories (`api/`, `graphs/`, `history/`) are auto-managed** — do not manually create or delete files there.
7. **GitHub Issues are used for incident tracking** — one Issue per outage event, closed on recovery. `skipDeleteIssues: true` prevents historical Issue deletion.

---

## Common Tasks

### Add a monitored site
Edit `.upptimerc.yml`, add entry under `sites:`, commit and push to `master`. Setup CI runs automatically.

### Remove a monitored site
Remove entry from `sites:` in `.upptimerc.yml`, commit and push.

### Trigger a manual uptime check
Go to GitHub Actions → "Uptime CI" → "Run workflow".

### Force a full re-setup
Go to GitHub Actions → "Setup CI" → "Run workflow", or push any change to `.upptimerc.yml`.

### Check current status of all sites
Read `history/summary.json` — it has the full current state of all 111 monitored endpoints.

### Update Upptime version
The `update-template.yml` workflow runs daily and uses `upptime/uptime-monitor@master` — workflow files are auto-regenerated from the latest template automatically. No manual merge from upstream is needed. The pinned version in workflows (`@v1.41.0`) is updated by this process.

### Check if Upptime is up-to-date
All workflow files contain `# 🔼 Upptime @v1.41.0` in their headers — compare this tag to the [latest release](https://github.com/upptime/uptime-monitor/releases) to verify currency.

---

## External Resources

- [Upptime documentation](https://upptime.js.org/docs)
- [Upptime configuration reference](https://upptime.js.org/docs/configuration)
- [Upptime GitHub repository](https://github.com/upptime/upptime)
- [uptime-monitor Action](https://github.com/upptime/uptime-monitor)
