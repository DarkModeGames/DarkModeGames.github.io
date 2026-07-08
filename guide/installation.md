# Installation & Setup

## Downloading Isometry

{{ product_name }} is a single executable — the engine *is* the product.
Download it from itch.io:

<div class="cta">
<a class="primary" href="https://dark-mode-games.itch.io/isometry">Download on itch.io</a>
</div>

The download includes builds for **Linux**, **Windows**, and **macOS**. Pick the
build for your platform and unpack it into a folder of your choosing.

On macOS, if the app won't open, run this once:

```bash
xattr -dr com.apple.quarantine Isometry.app
```

## Installing Campaigns

Campaign files are ZIP archives containing YAML or JSON entity definitions and
assets. Place a campaign `.zip` in one of these locations:

- The same folder as the {{ product_name }} executable
- A `campaigns/` subfolder next to the executable
- A custom path you point at with `--dir=/path/to/campaigns`

## Using the Launcher

Launching {{ product_name }} without network flags opens the launcher — a
graphical interface for starting games.

**Choosing a username & password.** Your in-game username is displayed to other
players. Your password is encrypted on the host's machine, and a new
username/password combination is created automatically the first time you connect
to a host. Be mindful of where your password is stored: don't reuse a password
from another personal account, and **write your username and password down** —
the host has no way to recover them for you.

1. Open {{ product_name }}.
2. Enter your **username** and **password**.
3. Select a **campaign** from the dropdown.
4. Enter the **URI** and **port** (defaults: `localhost:5000`).
5. Click **HOST** to start a server, or **JOIN** to connect to one.

## Command-Line Launch

For advanced use, {{ product_name }} can launch directly from the command line.
The examples below use the Linux executable name; substitute the executable for
your platform.

```bash
# Single player (host mode, localhost)
./isometry --campaign=demo --network=host --port=5000 \
  --username=player1 --secret=secret

# Join a multiplayer server
./isometry --campaign=demo --network=client \
  --uri=server.example.com --port=5000 \
  --username=player1 --secret=secret

# Run a dedicated server
./isometry --campaign=demo --network=server --port=5000 \
  --username=server --secret=serverpass
```

### CLI Arguments Reference

| Argument | Description | Default |
|----------|-------------|---------|
| `--campaign=NAME` | Campaign name (without `.zip`) | Required |
| `--network=MODE` | `host`, `server`, `client`, or `none` | Required |
| `--uri=HOST` | Server address | `localhost` |
| `--port=PORT` | Server port (1024–65535) | `5000` |
| `--dir=PATH` | Campaign directory path | auto-detected |
| `--username=NAME` | Player display name | Required |
| `--secret=PASS` | Authentication password | Required |
| `--log-level=LEVEL` | `trace`, `debug`, `info`, `warn`, `error` | `info` |
| `--delay=MS` | Connection delay in milliseconds | `0` |

You can also validate a campaign without launching a session:

```bash
# Check a campaign for errors (exit 0 = valid, 1 = invalid)
./isometry --validate=/path/to/campaign.zip
```

## Where to next

- [Playing](/guide/playing.html) — controls, joining a campaign, and keybinds.
- [Hosting](/guide/hosting.html) — run a host or dedicated server for others.
