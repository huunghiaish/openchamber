# OpenChamber Deployment Guide

**Version:** 1.0 | **Last Updated:** 2026-03-24 | **Platforms:** Web, Docker, Desktop, VS Code, Systemd

---

## Quick Start (All Methods)

| Method | Command | Best For |
|--------|---------|----------|
| **npm global** | `npm install -g @openchamber/web && openchamber` | Quick local dev |
| **Docker Compose** | `docker-compose up -d` | Self-hosted, persistent |
| **macOS App** | Download .dmg from Releases | Native experience |
| **VS Code** | Search "OpenChamber" in Extensions | Editor integration |
| **Systemd** | User service file | VPN/LAN persistent |

---

## Method 1: npm Global Install (Node.js 20+)

### Installation

```bash
# Install globally
npm install -g @openchamber/web

# Verify
openchamber --version
```

### Usage

**Basic Server:**
```bash
openchamber
# Server runs at http://127.0.0.1:3000
# OpenCode spawned automatically at http://127.0.0.1:4095
```

**With Password Protection:**
```bash
openchamber --ui-password my-secure-password
# Access at http://127.0.0.1:3000
# Password required on first login
```

**Custom Port & Host:**
```bash
openchamber --port 8080 --host 127.0.0.1
# Available at http://127.0.0.1:8080

# For LAN access (trusted network only):
openchamber --port 8080 --host 0.0.0.0
# WARNING: Exposes UI to all devices on network
```

**Foreground Mode (systemd/Docker):**
```bash
openchamber --port 3000 --foreground
# Keeps process in foreground (doesn't daemonize)
# Useful with systemd Type=simple or Docker
```

**Connect to External OpenCode:**
```bash
# If OpenCode server already running elsewhere
OPENCODE_PORT=4096 OPENCODE_SKIP_START=true openchamber
# Or custom URL
OPENCODE_HOST=https://myhost:4096 OPENCODE_SKIP_START=true openchamber
```

### CLI Commands

**Tunnel Management:**
```bash
# Quick tunnel (temporary URL)
openchamber tunnel start --provider cloudflare --mode quick --qr

# Managed remote (custom domain)
openchamber tunnel profile add --provider cloudflare --mode managed-remote \
  --name prod-main --hostname app.example.com --token <TOKEN>
openchamber tunnel start --profile prod-main

# Check status
openchamber tunnel status --all

# Stop tunnel (server stays running)
openchamber tunnel stop --port 3000
```

**Server Management:**
```bash
# View logs (follow last 200 lines)
openchamber logs --tail 200 --follow

# Check for updates
openchamber update check

# Install update + restart
openchamber update

# Graceful shutdown
openchamber stop --port 3000
```

---

## Method 2: Docker Compose (Recommended for Self-Hosted)

### Setup

**Prerequisites:**
- Docker & Docker Compose installed
- Port 3000 available (or change in compose file)

**Create `docker-compose.yml`:**
```yaml
version: '3.8'

services:
  openchamber:
    image: oven/bun:1
    working_dir: /app
    ports:
      - "3000:3000"
    environment:
      - UI_PASSWORD=your_secure_password
      - OPENCHAMBER_PORT=3000
      - OPENCHAMBER_HOST=0.0.0.0
      - OPENCODE_SKIP_START=false  # Let container spawn OpenCode
    volumes:
      - ./data/openchamber:/home/openchamber/.config/openchamber
      - ./data/opencode:/home/openchamber/.opencode
      - ./data/ssh:/home/openchamber/.ssh
    command: >
      bash -c "
        npm install -g @openchamber/web &&
        openchamber --port 3000 --host 0.0.0.0 --foreground
      "
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 10s
```

**Create Data Directories:**
```bash
mkdir -p data/openchamber data/opencode/share data/opencode/config data/ssh
chown -R 1000:1000 data/  # openchamber user (UID 1000)
```

**Start Services:**
```bash
docker-compose up -d

# View logs
docker-compose logs -f openchamber

# Stop
docker-compose down
```

### Docker Environment Variables

| Variable | Default | Purpose |
|----------|---------|---------|
| `UI_PASSWORD` | (none) | Password-protect UI |
| `OPENCHAMBER_PORT` | 3000 | Server port (inside container) |
| `OPENCHAMBER_HOST` | 127.0.0.1 | Bind address (0.0.0.0 for multi-device) |
| `OPENCODE_SKIP_START` | false | Use external OpenCode |
| `OPENCODE_HOST` | http://localhost:4095 | External OpenCode URL |
| `OPENCHAMBER_OPENCODE_HOSTNAME` | 127.0.0.1 | Bind address for managed OpenCode |

### Cloudflare Tunnel (Optional)

Add tunnel environment to compose:
```yaml
services:
  openchamber:
    # ... existing config
    environment:
      - OPENCHAMBER_TUNNEL_MODE=quick
      - OPENCHAMBER_TUNNEL_PROVIDER=cloudflare
```

For managed-remote:
```yaml
environment:
  - OPENCHAMBER_TUNNEL_MODE=managed-remote
  - OPENCHAMBER_TUNNEL_HOSTNAME=app.example.com
  - OPENCHAMBER_TUNNEL_TOKEN=<cf-token>
```

For managed-local:
```yaml
environment:
  - OPENCHAMBER_TUNNEL_MODE=managed-local
  - OPENCHAMBER_TUNNEL_CONFIG=/home/openchamber/.cloudflared/config.yml
volumes:
  - /path/to/cloudflared/config.yml:/home/openchamber/.cloudflared/config.yml:ro
```

---

## Method 3: Systemd Service (VPN/LAN Access)

Useful for persistent server on VPN (Tailscale, WireGuard) or LAN without Cloudflare.

### Setup OpenCode Service

**File:** `~/.config/systemd/user/opencode.service`

```ini
[Unit]
Description=OpenCode Server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=opencode serve --port 4095
# Set PATH (systemd doesn't source shell profile)
Environment="PATH=/home/linuxbrew/.linuxbrew/bin:/home/YOU/.local/bin:/usr/local/bin:/usr/bin:/bin"
# Required for git over SSH
Environment="SSH_AUTH_SOCK=%t/ssh-agent.socket"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

**Adjust PATH for your environment:**
```bash
# Find your tool paths
which npm          # e.g., /home/YOU/.local/bin/npm
which python3      # e.g., /usr/bin/python3
which git          # e.g., /usr/bin/git

# Update PATH line accordingly
```

### Setup OpenChamber Service

**File:** `~/.config/systemd/user/openchamber.service`

```ini
[Unit]
Description=OpenChamber Web Server
After=opencode.service
Wants=opencode.service

[Service]
Type=simple
ExecStart=openchamber serve --port 3000 --host 0.0.0.0 --ui-password your-password --foreground
# Connect to OpenCode on localhost
Environment="OPENCODE_HOST=http://localhost:4095"
Environment="OPENCODE_SKIP_START=true"
Restart=on-failure
RestartSec=5

[Install]
WantedBy=default.target
```

### Enable & Start

```bash
# Reload systemd user configuration
systemctl --user daemon-reload

# Enable services (start on login)
systemctl --user enable opencode openchamber

# Start services
systemctl --user start opencode openchamber

# Check status
systemctl --user status openchamber
systemctl --user status opencode

# View logs
journalctl --user -u openchamber -f
journalctl --user -u opencode -f

# Stop
systemctl --user stop openchamber opencode
```

**Access:** `http://<your-vpn-ip>:3000` from any device on VPN

---

## Method 4: macOS Desktop App

### Installation

**From GitHub Releases:**
1. Visit [github.com/btriapitsyn/openchamber/releases](https://github.com/btriapitsyn/openchamber/releases)
2. Download `.dmg` (e.g., `openchamber_1.9.1_aarch64.dmg`)
3. Double-click to mount
4. Drag app to Applications folder

### Usage

- **Launch:** Cmd+Space → "OpenChamber"
- **Multi-window:** File → New Window
- **Preferences:** OpenChamber → Settings (or Cmd+,)
- **Update:** OpenChamber → Check for Updates

### Configuration

**Settings Location:** `~/.openchamber/`

| File | Purpose |
|------|---------|
| `settings.json` | Theme, window geometry, preferences |
| `desktop-hosts.json` | Remote host list (SSH) |
| `desktop-local-port.txt` | Sidecar port cache |

### SSH Remote Instances

**Via UI:**
1. Settings → Hosts
2. Add Host
3. Fill: hostname, user, port, key path
4. Click Connect
5. App shows live logs + status

**Via CLI (alternative):**
```bash
# View current config
cat ~/.openchamber/desktop-hosts.json

# Edit manually
nano ~/.openchamber/desktop-hosts.json
```

---

## Method 5: VS Code Extension

### Installation

**From Marketplace:**
1. Open VS Code Extensions (Cmd+Shift+X)
2. Search "OpenChamber"
3. Click Install

**Manual Install (from .vsix):**
```bash
# Download .vsix from GitHub Releases
code --install-extension openchamber-1.9.1.vsix
```

### First Run

- Extension spawns local web server (port 3001)
- Opens Chat sidebar + Agent Manager panel
- Connects to OpenCode (spawned or external)

### Configuration

**Command Palette (Cmd+Shift+P):**
- `OpenChamber: Show Chat` — Open/focus chat panel
- `OpenChamber: New Session` — Create session
- `OpenChamber: Settings` — Open extension settings

**Settings (VS Code → Extensions → OpenChamber):**
- `openchamber.port` — Web server port (default 3001)
- `openchamber.theme` — Theme mode (inherit from VS Code)
- `opencode.port` — OpenCode server port (default 4095)
- `opencode.skipStart` — Use external OpenCode

---

## Environment Variables Reference

### Server Control

```bash
# Listening address
OPENCHAMBER_HOST=127.0.0.1              # Default (localhost only)
OPENCHAMBER_HOST=0.0.0.0                # Listen on all interfaces (VPN/LAN)

# Port
OPENCHAMBER_PORT=3000                   # Default (CLI --port overrides)

# UI protection
UI_PASSWORD=secret                       # Password-protect UI

# Foreground mode (systemd/Docker)
# (use --foreground CLI flag instead of env var)
```

### OpenCode Integration

```bash
# Connect to running OpenCode server
OPENCODE_PORT=4096                      # Default 4095
OPENCODE_HOST=https://myhost:4096       # Full URL override
OPENCODE_SKIP_START=true                # Don't spawn OpenCode

# Managed OpenCode hostname
OPENCHAMBER_OPENCODE_HOSTNAME=0.0.0.0  # Bind address for spawned OpenCode
```

### Tunnel Configuration

```bash
# Quick tunnel (temporary)
OPENCHAMBER_TUNNEL_MODE=quick
OPENCHAMBER_TUNNEL_PROVIDER=cloudflare

# Managed-remote (custom domain)
OPENCHAMBER_TUNNEL_MODE=managed-remote
OPENCHAMBER_TUNNEL_HOSTNAME=app.example.com
OPENCHAMBER_TUNNEL_TOKEN=<cloudflare-token>

# Managed-local (corporate)
OPENCHAMBER_TUNNEL_MODE=managed-local
OPENCHAMBER_TUNNEL_CONFIG=/path/to/config.yml
```

### Development

```bash
# Disable PWA in development
OPENCHAMBER_DISABLE_PWA_DEV=1

# Enable React profiling
VITE_ENABLE_REACT_SCAN=1
```

---

## Networking & Security

### Default Behavior

- **Binding:** `127.0.0.1:3000` (localhost only)
- **OpenCode:** `127.0.0.1:4095` (localhost only)
- **Tunnel:** Cloudflare quick mode (temporary)
- **UI Auth:** None (set `UI_PASSWORD` for protection)

**Security Model:**
- No authentication by default → assume trusted network
- UI password optional → use for remote/shared access
- Tunnel tokens → one-time use, session-based
- Path validation → prevent directory traversal
- SSH keys → user-configured, never auto-discovered

### VPN / LAN Access

**Tailscale + Systemd (Recommended):**
1. Install Tailscale on dev machine
2. Create systemd services (see Method 3)
3. Access from any device on Tailscale network:
   ```bash
   http://dev-machine-tailscale-ip:3000
   ```

**Manual LAN:**
1. Find your LAN IP: `ifconfig | grep inet`
2. Run: `openchamber --host 0.0.0.0`
3. Access from other device: `http://192.168.1.100:3000`
4. **WARNING:** Exposes to all LAN devices; use `UI_PASSWORD`

### Cloudflare Tunnel (Recommended)

**Quick Setup:**
```bash
openchamber tunnel start --provider cloudflare --mode quick --qr
# Scan QR with phone to access instantly
# URL auto-expires when stopped
```

**Persistent Domain:**
```bash
# Get Cloudflare token: https://dash.cloudflare.com/profile/api-tokens
openchamber tunnel profile add \
  --provider cloudflare \
  --mode managed-remote \
  --hostname app.example.com \
  --token <TOKEN>

openchamber tunnel start --profile myprofile
# https://app.example.com accessible anytime
```

---

## Database & Persistence

### Data Locations

**Web/Desktop (localStorage + ~/.config/):**
```
~/.config/openchamber/
├── settings.json              # User preferences
├── tunnel-cli-state.json      # Tunnel configuration & state
├── themes/                    # Custom theme JSON files
├── desktop-hosts.json         # Remote SSH hosts (desktop)
└── desktop-local-port.txt     # Sidecar port cache (desktop)

~/.opencode/                   # OpenCode auth & config
├── auth.json                  # OAuth tokens (mode 600)
└── config.json
```

**VS Code Extension:**
```
~/.vscode/extensions/openchamber-*/
└── (extension-specific storage)
```

**Docker Compose:**
```
./data/
├── openchamber/               # Mounted to ~/.config/openchamber
├── opencode/share/            # OpenCode sessions
├── opencode/config/           # OpenCode config
└── ssh/                       # SSH keys (for git)
```

### Backup

**Important Files to Backup:**
```bash
# Settings
~/.config/openchamber/settings.json
~/.config/openchamber/themes/*.json

# Auth tokens
~/.opencode/auth.json

# SSH keys (if using git over SSH)
~/.ssh/id_ed25519
~/.ssh/id_rsa
```

**Backup Script:**
```bash
#!/bin/bash
BACKUP_DIR=~/backups/openchamber-$(date +%Y%m%d)
mkdir -p "$BACKUP_DIR"

cp -r ~/.config/openchamber "$BACKUP_DIR/"
cp -r ~/.opencode "$BACKUP_DIR/"
cp -r ~/.ssh "$BACKUP_DIR/"

echo "Backup complete: $BACKUP_DIR"
```

---

## Troubleshooting

### Server Won't Start

**Check port in use:**
```bash
# macOS/Linux
lsof -i :3000
# Kill process
kill -9 <PID>

# Then retry
openchamber --port 3000
```

**Check OpenCode:**
```bash
# Is OpenCode binary available?
which opencode
opencode --version

# Try explicit spawn
OPENCODE_PORT=4096 openchamber
```

### Tunnel Not Working

**Test Cloudflare:**
```bash
# Manual test
cloudflared tunnel --url http://127.0.0.1:3000

# Check status
openchamber tunnel status --all
```

**View tunnel logs:**
```bash
# Docker
docker-compose logs openchamber | grep tunnel

# Systemd
journalctl --user -u openchamber | grep tunnel
```

### Terminal Not Responding

**Restart PTY:**
```bash
# Terminal → Disconnect
# Terminal → Create new session
```

**Check PTY availability:**
```bash
# node-pty or bun-pty should be installed
npm list node-pty
npm list bun-pty
```

### Git Operations Failing

**SSH keys not found:**
```bash
# Ensure SSH agent is running
ssh-add -l

# Add keys
ssh-add ~/.ssh/id_ed25519

# For systemd: set SSH_AUTH_SOCK env var (see systemd setup)
```

**GitHub auth issues:**
```bash
# Clear auth token
rm ~/.opencode/auth.json

# Re-authenticate
# Settings → GitHub → Authenticate
```

---

## Performance Tuning

### Memory & CPU

**Reduce bundle size (web):**
- Lazy load views (chat/git/settings)
- Use virtualized lists for long messages

**Optimize terminal:**
- Disable syntax highlighting for very large outputs
- Use `--max-buffer` if needed

**Monitor resources:**
```bash
# macOS
top -o rsize | head -20

# Linux
htop
```

### Network

**For slow connections:**
- Enable compression (built-in with Express)
- Reduce chunk sizes in Vite config
- Use Cloudflare tunnel (handles optimization)

**Bandwidth limits:**
- Max upload: 50 MB (configurable in server)
- Max request timeout: 4 minutes
- Streams (SSE, WebSocket) preferred over polling

---

## Updates & Upgrades

### Web Package

```bash
# Check for updates
npm outdated -g @openchamber/web

# Upgrade
npm install -g @openchamber/web@latest

# Using CLI
openchamber update check
openchamber update
```

### Docker

```bash
# Pull latest image
docker-compose pull

# Rebuild & restart
docker-compose up -d --build
```

### Desktop App

**Auto-update:**
- Automatically checks GitHub releases on startup
- Prompts user to install if newer version available
- Restarts app with new version

**Manual:**
1. Visit [Releases](https://github.com/btriapitsyn/openchamber/releases)
2. Download `.dmg`
3. Drag new app to Applications folder

### VS Code Extension

Auto-updates via VS Code Marketplace (check in Extensions panel).

---

## Security Best Practices

1. **Always use UI password** if exposing beyond localhost
2. **Use tunnel tokens** for remote access (not just password)
3. **Rotate SSH keys** regularly
4. **Never commit** `~/.opencode/auth.json` or `.env` files
5. **Backup auth tokens** separately (encrypted)
6. **Use HTTPS** for tunnel access (Cloudflare provides this)
7. **Monitor logs** for unauthorized access attempts
8. **Keep OpenCode updated** (security patches)

---

**Maintained by:** OpenChamber Team | **License:** MIT
