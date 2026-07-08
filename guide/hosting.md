# Hosting & Networking

## Network Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `host` | Server + local client | Solo or small groups |
| `server` | Dedicated server only | Always-on multiplayer |
| `client` | Connect to a server | Join an existing game |
| `none` | No networking | Offline testing |

## Host Mode

Host mode starts a server and connects a local player automatically. This is the
simplest way to play with friends.

```bash
./isometry --campaign=mygame --network=host \
  --port=5000 --username=host_player --secret=secret
```

Other players connect as clients to your IP address.

## Dedicated Server

For always-on multiplayer, run a dedicated server:

```bash
./isometry --campaign=mygame --network=server \
  --port=5000 --username=server --secret=adminpass
```

### Server Deployment

#### Systemd Service

```ini
[Unit]
Description=Isometry Game Server
After=network.target

[Service]
Type=simple
User=isometry
ExecStart=/opt/isometry/isometry \
  --campaign=mygame --network=server --port=5000 \
  --username=server --secret=secret
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Port Configuration

- Default port: **5000** (UDP)
- Port range: 1024–65535
- Forward UDP traffic on your router/firewall

### Security

- Encryption keys are generated on first server launch and stored locally.
- Client passwords are encrypted during transmission.
- The engine checks everyone is on the same campaign version.
- The maximum client count is configurable (default: 4095).

## Where to next

- [Playing](/guide/playing.html) — controls and gameplay mechanics.
- [Installation](/guide/installation.html) — download and set up
  {{ product_name }}.
