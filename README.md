# devpulse

A daily dev digest CLI that runs every morning and opens a local web dashboard.

## What it shows

- **Hacker News** — top stories filtered by dev topics (AI, LLM, CLI, security, devtools, etc.)
- **GitHub Trending** — hot repos today for your languages (Python, TypeScript, Go, Rust, JS)
- **Security Breaches & Hacks** — latest breach/hack news from The Hacker News, BleepingComputer, Krebs on Security, and SecurityWeek — sorted newest first

## Setup

### 1. Install the CLI

```bash
cp devpulse devpulse-server ~/bin/
chmod +x ~/bin/devpulse ~/bin/devpulse-server
mkdir -p ~/.devpulse
```

### 2. Web server (nginx via Docker)

```bash
docker run -d \
  --name devpulse-web \
  --restart unless-stopped \
  -p 8080:80 \
  -v ~/.devpulse:/usr/share/nginx/html:ro \
  nginx:alpine
```

Dashboard available at **http://localhost:8080**

### 3. Refresh server (systemd user service)

```bash
mkdir -p ~/.config/systemd/user
cp systemd/devpulse-server.service ~/.config/systemd/user/
systemctl --user daemon-reload
systemctl --user enable --now devpulse-server
```

### 4. Run on shell startup (once per day)

Add to `~/.bashrc`:

```bash
_devpulse_stamp="${XDG_CACHE_HOME:-$HOME/.cache}/devpulse_last_run"
if [[ "$(cat "$_devpulse_stamp" 2>/dev/null)" != "$(date +%Y-%m-%d)" ]]; then
    devpulse && date +%Y-%m-%d > "$_devpulse_stamp"
fi
unset _devpulse_stamp
```

## Usage

```bash
devpulse                          # full digest + open browser
devpulse --no-browser             # terminal only
devpulse --lang go,rust           # filter GitHub trending by language
devpulse --topics security,wasm   # filter HN by topic
devpulse --no-hn                  # skip Hacker News
devpulse --no-gh                  # skip GitHub trending
```

## Files

| File | Description |
|------|-------------|
| `devpulse` | Main CLI script |
| `devpulse-server` | Refresh server (port 8081) — triggered by the Refresh button in the UI |
| `web/index.html` | Landing page with clock, quick links, and devpulse link |
| `systemd/devpulse-server.service` | systemd user service for the refresh server |
