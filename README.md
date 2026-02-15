# Pingr API

**Lightweight server monitoring agent written in Go.**

Pingr is a single-binary monitoring agent that runs on your VPS and exposes a clean REST API. Designed to be consumed by the [Pingr mobile app](https://github.com/loutrx/pingr-app), it gives you real-time visibility into your server health — no heavy dashboards, no complex setup.

Part of the [Otterium](https://otterium.com) ecosystem.

## Why Pingr?

Most monitoring tools are built around web dashboards. Pingr takes a different approach: a minimal agent on your server, a mobile app in your pocket. Check your servers while grabbing coffee, get notified instantly when something goes wrong.

- **Single binary** — no runtime, no dependencies, just drop it on your server
- **Zero config** — sane defaults, works out of the box
- **Mobile-first** — built as an API for the Pingr mobile app, not a web UI
- **Lightweight** — minimal resource footprint, won't compete with your actual services
- **Open source** — AGPL-3.0 licensed, community-driven

## Features

### System Metrics
- CPU usage (per-core and aggregate)
- Memory usage (RAM + swap)
- Disk usage (per mount point)
- Network I/O (bandwidth, throughput)
- System uptime and load average

### Docker Monitoring
- Container list with status (running, stopped, unhealthy)
- Per-container resource usage (CPU, memory, network)
- Container health check status
- Container logs (tail)
- Container actions (restart, stop, start)

### Service Health Checks
- HTTP/HTTPS endpoint monitoring
- Custom health check intervals
- Response time tracking
- Status history (up/down timeline)

### Alerting
- Push notifications via Firebase Cloud Messaging (FCM)
- Configurable thresholds (CPU > 90%, disk > 85%, container down, etc.)
- Cooldown periods to avoid notification spam
- Alert history

## Tech Stack

- **Language:** Go 1.22+
- **System metrics:** [gopsutil](https://github.com/shirou/gopsutil)
- **Docker integration:** [Docker Engine SDK](https://pkg.go.dev/github.com/docker/docker/client)
- **HTTP framework:** [Echo](https://echo.labstack.com/) or [Gin](https://gin-gonic.com/)
- **Database:** SQLite (via [modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite)) — metrics history, alert logs
- **Authentication:** API key (simple) or mTLS (advanced)

## Installation

### Quick install (Linux)

```bash
curl -fsSL https://get.pingr.dev | sh
```

### From binary

Download the latest release from [Releases](https://github.com/loutrx/pingr-api/releases) and place it in your PATH.

```bash
chmod +x pingr
sudo mv pingr /usr/local/bin/
```

### From source

```bash
git clone https://github.com/loutrx/pingr-api.git
cd pingr-api
go build -o pingr ./cmd/pingr
```

### Run as a systemd service

```bash
pingr install   # generates and enables a systemd unit
pingr start     # starts the service
```

## Configuration

Pingr works with zero configuration. For custom setups, create a `pingr.yaml`:

```yaml
# Server
port: 9009
host: 0.0.0.0

# Authentication
auth:
  api_key: "your-secret-key"    # auto-generated on first run if not set

# Docker
docker:
  enabled: true
  socket: /var/run/docker.sock

# Metrics retention
storage:
  path: /var/lib/pingr/data.db
  retention: 7d                  # how long to keep metric history

# Health checks
checks:
  - name: "My App"
    url: "https://myapp.example.com/health"
    interval: 60s
  - name: "API"
    url: "https://api.example.com/ping"
    interval: 30s

# Alerts
alerts:
  fcm_token: ""                  # set from mobile app during pairing
  thresholds:
    cpu: 90                      # alert when CPU > 90%
    memory: 85                   # alert when RAM > 85%
    disk: 90                     # alert when disk > 90%
  cooldown: 5m                   # minimum time between repeated alerts
```

## API Reference

All endpoints require the `X-API-Key` header.

### System

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/health` | Agent health check |
| `GET` | `/api/v1/system` | Full system overview (CPU, RAM, disk, network, uptime) |
| `GET` | `/api/v1/system/cpu` | CPU usage details |
| `GET` | `/api/v1/system/memory` | Memory usage details |
| `GET` | `/api/v1/system/disk` | Disk usage per mount |
| `GET` | `/api/v1/system/network` | Network I/O stats |

### Docker

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/containers` | List all containers with status |
| `GET` | `/api/v1/containers/:id` | Container details + resource usage |
| `GET` | `/api/v1/containers/:id/logs` | Container logs (query: `lines`, `since`) |
| `POST` | `/api/v1/containers/:id/restart` | Restart a container |
| `POST` | `/api/v1/containers/:id/stop` | Stop a container |
| `POST` | `/api/v1/containers/:id/start` | Start a container |

### Health Checks

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/checks` | List all configured health checks with current status |
| `GET` | `/api/v1/checks/:name/history` | Status history for a specific check |
| `POST` | `/api/v1/checks` | Add a new health check |
| `DELETE` | `/api/v1/checks/:name` | Remove a health check |

### Alerts

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/alerts` | Alert history |
| `PUT` | `/api/v1/alerts/config` | Update alert thresholds |
| `POST` | `/api/v1/alerts/test` | Send a test notification |

### Pairing

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/pair` | Register mobile app FCM token |

## Example Response

```
GET /api/v1/system
```

```json
{
  "hostname": "vps-prod-01",
  "uptime": 1209600,
  "cpu": {
    "usage_percent": 23.5,
    "cores": 4
  },
  "memory": {
    "total_bytes": 8589934592,
    "used_bytes": 3435973837,
    "usage_percent": 40.0
  },
  "disk": [
    {
      "mount": "/",
      "total_bytes": 53687091200,
      "used_bytes": 21474836480,
      "usage_percent": 40.0
    }
  ],
  "network": {
    "bytes_sent": 1073741824,
    "bytes_recv": 5368709120
  },
  "containers": {
    "running": 5,
    "stopped": 1,
    "unhealthy": 0
  }
}
```

## Project Structure

```
pingr-api/
├── cmd/
│   └── pingr/
│       └── main.go              # entrypoint
├── internal/
│   ├── api/
│   │   ├── router.go            # route definitions
│   │   ├── middleware.go         # auth, logging
│   │   ├── handlers_system.go   # system metrics handlers
│   │   ├── handlers_docker.go   # docker handlers
│   │   ├── handlers_checks.go   # health check handlers
│   │   └── handlers_alerts.go   # alert handlers
│   ├── collector/
│   │   ├── system.go            # gopsutil wrappers
│   │   ├── docker.go            # docker client wrappers
│   │   └── checks.go            # HTTP health checker
│   ├── alerter/
│   │   ├── alerter.go           # alert evaluation engine
│   │   └── fcm.go               # Firebase push notifications
│   ├── storage/
│   │   └── sqlite.go            # SQLite storage layer
│   └── config/
│       └── config.go            # YAML config loader
├── scripts/
│   └── install.sh               # curl installer script
├── pingr.example.yaml
├── go.mod
├── go.sum
├── Dockerfile
├── LICENSE
└── README.md
```

## Deployment with Kamal

Pingr can be deployed alongside your apps using Kamal as an accessory:

```yaml
# In your Kamal deploy.yml
accessories:
  pingr:
    image: ghcr.io/loutrx/pingr-api:latest
    host: 123.45.67.89
    port: 9009
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /var/lib/pingr:/var/lib/pingr
    env:
      PINGR_API_KEY: your-secret-key
```

## Security Considerations

- **Always** use an API key — the agent exposes system info and Docker controls
- Run behind a firewall: only expose port 9009 to your IP or through a VPN/WireGuard tunnel
- Optionally, don't expose the port at all and use Cloudflare Tunnel or Tailscale for access
- The Docker socket is mounted **read-only** by default for metrics; container actions require write access
- API key is stored hashed, never in plaintext in the database

## Roadmap

- [ ] Core system metrics (CPU, RAM, disk, network)
- [ ] Docker container monitoring
- [ ] HTTP health checks
- [ ] Push notifications via FCM
- [ ] Multi-server support (single app, multiple agents)
- [ ] Metrics history with configurable retention
- [ ] WebSocket support for real-time updates in the mobile app
- [ ] Docker Compose awareness (group containers by stack)
- [ ] Optional Prometheus metrics export (`/metrics` endpoint)
- [ ] Plugin system for custom checks
- [ ] ARM64 builds (Raspberry Pi, Oracle Cloud free tier)

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) before submitting a PR.

## License

AGPL-3.0-or-later — see [LICENSE](LICENSE)

---

Built with ☕ by [Otterium](https://otterium.com)
