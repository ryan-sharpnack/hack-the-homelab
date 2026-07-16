# IHA-RB-SIEM-LOKI-GRAFANA-DEPLOYMENT

**Category:** SIEM
**Applies to:** SIEM01 (10.50.10.20)
**Last updated:** 2026-07-15 (Day 10)

## Purpose

Deploy a self-hosted Loki + Grafana log aggregation stack on SIEM01 via Docker Compose, with Windows Event Log collection from both domain controllers via Grafana Alloy. Replaces the originally-planned Wazuh deployment (see risk register IHA-011).

## Prerequisites

- SIEM01 patched and rebooted clean before deployment (avoids taking a stale baseline)
- SIEM01 sized at 4GB RAM (restores original Phase 1 budget; see Doc 04)
- Docker and Docker Compose installed on SIEM01
- Outbound HTTP/HTTPS/DNS firewall rules in place on FW01 LAN tab (package/image pulls)

## Docker Compose deployment

`docker-compose.yml`:

\`\`\`yaml
services:
  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"
    volumes:
      - loki-data:/loki
      - ./loki-config.yaml:/etc/loki/local-config.yaml:ro
    restart: unless-stopped

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    volumes:
      - grafana-data:/var/lib/grafana
    depends_on:
      - loki
    restart: unless-stopped

volumes:
  loki-data:
  grafana-data:
\`\`\`

`restart: unless-stopped` is required on both services — validated under a real host reboot on Day 10; without it, a host restart leaves the stack down until someone notices.

Retention (`loki-config.yaml`, 30-day/720h active retention): pull the running container's actual default config first (`docker exec <loki-container> cat /etc/loki/local-config.yaml`) rather than assuming a schema, then add:

\`\`\`yaml
limits_config:
  retention_period: 720h

compactor:
  working_directory: /loki/compactor
  retention_enabled: true
  delete_request_store: filesystem
\`\`\`

Bring up: `docker compose up -d`

## Verification

\`\`\`bash
docker compose config      # validate YAML before applying — catches indentation errors before they cost a cycle
docker ps -a               # confirm both containers show "Up"
\`\`\`

## Grafana data source configuration

1. `http://<SIEM01-IP>:3000` (port 3000, not 443 — this is not a Wazuh-style HTTPS dashboard)
2. Connections → Data sources → Add data source → Loki
3. URL: `http://loki:3100` (Docker Compose service name, not `localhost` or the host IP)
4. No authentication, no TLS — connection never leaves the Compose network
5. Save & test — expect a green success message

## Alloy agent setup (both domain controllers)

Identical `config.alloy` on DC01-IRONBRIDGE and DC01-HAWTHORNE — `instance = constants.hostname` makes each host independently distinguishable without needing separate configs:

\`\`\`
loki.write "endpoint" {
  endpoint {
    url = "http://<SIEM01-IP>:3100/loki/api/v1/push"
  }
}

loki.source.windowsevent "security" {
  eventlog_name          = "Security"
  use_incoming_timestamp = true
  labels = {
    job      = "windows_eventlog",
    instance = constants.hostname,
  }
  forward_to = [loki.write.endpoint.receiver]
}

loki.source.windowsevent "system" {
  eventlog_name          = "System"
  use_incoming_timestamp = true
  labels = {
    job      = "windows_eventlog",
    instance = constants.hostname,
  }
  forward_to = [loki.write.endpoint.receiver]
}

loki.source.windowsevent "application" {
  eventlog_name          = "Application"
  use_incoming_timestamp = true
  labels = {
    job      = "windows_eventlog",
    instance = constants.hostname,
  }
  forward_to = [loki.write.endpoint.receiver]
}
\`\`\`

Note the full push path (`/loki/api/v1/push`) — different from the bare `http://loki:3100` used in the Grafana data source; Alloy doesn't append it automatically. Use the real SIEM01 IP here, not the `loki` container hostname — the DCs aren't on that Docker network.

Install: MSI installer, run as Administrator, default path. Config file: `C:\Program Files\GrafanaLabs\Alloy\config.alloy`. Restart the **Alloy** Windows service via `services.msc` after any config change.

Verify in Grafana Explore: `{instance="DC01-IRONBRIDGE"}` and `{instance="DC01-HAWTHORNE"}`, separately — confirms per-host ingestion rather than a blended, ambiguous result.

## Known gotchas

- **YAML indentation:** every sibling key in a mapping must sit at identical indentation. A one-space drift produces a cryptic `did not find expected key` parser error. Run `docker compose config` before `up -d` to catch this before it costs a troubleshooting cycle.
- **`config.alloy` typos are silent killers:** a single mistyped attribute name (e.g., `use_incoming_timestamp` mistyped as `use_incoming>timestamp`) fails the entire config file, not just that block. If the Alloy service won't start, check the config file before assuming a network/firewall issue.
- **`apt-daily`/`unattended-upgrades` race on fresh Ubuntu VMs:** on a freshly provisioned Ubuntu guest, running an install script (Docker, etc.) immediately after first boot can race against the automatic `apt-daily` update timers, causing intermittent, hard-to-reproduce package-lock failures. Run this immediately after first boot, before any install script:
  \`\`\`bash
  sudo systemctl stop apt-daily.service apt-daily-upgrade.service apt-daily.timer apt-daily-upgrade.timer
  \`\`\`
  Encountered on SIEM01; applies to **any** freshly-provisioned Ubuntu VM — apply this on WS01/APP01 in Phase 3 before running their install scripts, not just here.
