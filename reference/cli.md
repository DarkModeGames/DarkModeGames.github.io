# Command-Line Reference

This page documents every command-line argument the {{ product_name }} binary parses, with accepted values, defaults, and runtime effect. Use it when you run a server, automate a build, or boot a [campaign](/manual/getting-started/glossary.html#terms) headless.

## Introduction

{{ product_name }} is a single binary that reads command-line arguments at startup to pick a campaign, choose a network mode, and supply credentials. Every build reads these flags, windowed or headless. A custom launcher can collect the same settings in its own UI and pass them to the binary as flags.

This reference covers {{ product_name }}'s own arguments. It also notes the one engine flag you commonly pair with them, `--headless`, which runs the binary without opening a window.

## Prerequisites

- [Install and run](/manual/getting-started/install-and-run.html) — how to obtain and launch the binary.
- The [glossary](/manual/getting-started/glossary.html) for terms such as *campaign*.

## How arguments are parsed

{{ product_name }} parses arguments in two passes. The network mode is read first, during startup, so that the binary knows whether to route into a session. Everything else is read immediately after and stored for the run.

Use the form `--key=value`. A single-dash token such as `-flag` becomes a boolean `true`.

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

Here `--headless` is an engine flag that suppresses the window; the rest are {{ product_name }}'s.

> Tip: Prefer the `--key=value` form. The space-separated `--key value` form mostly works, but `--key=value` is the reliable shape and the one used throughout this manual.

## Argument summary

| Argument | Values | Default | Effect |
| --- | --- | --- | --- |
| `--usage` | (flag) | - | Prints this list of flags and exits (--help and -h are Godot's own engine help). |
| `--campaign` | campaign name (no .zip) | (empty) | Selects the campaign archive to load. |
| `--network` | none/host/server/client, 0-3, or n/h/s/c | none | Chooses the network role; only host, server and client start a session. |
| `--uri` | hostname or IP | localhost | Server address a client connects to. |
| `--port` | port number | 5000 | UDP port for both server bind and client connect. |
| `--username` | non-empty string | (empty) | Login name. |
| `--secret` | non-empty string | (empty) | Login secret (password). |
| `--dir` | absolute path or subdirectory name | (see below) | Folder that holds campaign archives. |
| `--resolution` | WIDTHxHEIGHT | saved option | Window size, e.g. 1920x1080; overrides the saved option. |
| `--interface_scale` | number 0.5-3.0 | saved option | Scales the menus and HUD; 1.0 is normal size. |
| `--log-level` | NONE/FATAL/ERROR/WARN/INFO/DEBUG/TRACE | NONE | Minimum severity written to the log file. |
| `--validate` | campaign directory or .zip path | - | Validates a campaign, prints the result and exits (0 = passed). |
| `--dump-schemas` | output file path (optional) | stdout | Writes the entity/validation spec as JSON and exits. |
| `--regenerate-doc-cache` | output file path (optional) | built-in doc cache | Rebuilds the entity documentation cache and exits. |
| `--print-paths` | (flag) | - | Prints the resolved data and campaign paths and exits. |
| `--app-version` | (flag) | - | Prints the engine version and exits. |
| `--control` | info/restart/shutdown | - | Sends one admin command to a running server, prints the JSON reply and exits. |
| `--admin-port` | port number | game port + 1 | Admin endpoint port (127.0.0.1 only) for a server, or the one --control calls. |
| `--admin-secret` | string | ISOMETRY_ADMIN_SECRET, then the token file | Secret the admin endpoints require. Also read from ISOMETRY_ADMIN_SECRET; without either, a server generates admin_token. |
| `--debug-collision-all` | (flag) | off | Draws the collision shapes of every layer. |
| `--ui-script` | script file path | - | Plays a script of player input and UI checks, prints PASS/FAIL lines and exits 0 or 1 (host only; tickets 284-285). |
| `--debug-collision-wall` | (flag) | off | Draws the wall collision shapes. |
| `--debug-collision-base` | (flag) | off | Draws the base collision shapes. |
| `--debug-collision-hitbox` | (flag) | off | Draws the hitbox collision shapes. |
| `--debug-collision-viewbox` | (flag) | off | Draws the viewbox collision shapes. |
| `--debug-collision-discovery` | (flag) | off | Draws the discovery collision shapes. |
| `--debug-collision-salience` | (flag) | off | Draws the salience collision shapes. |
| `--debug-collision-barrier` | (flag) | off | Draws the barrier collision shapes. |
| `--debug-collision-navigation` | (flag) | off | Draws the navigation collision shapes. |

This table is checked against the engine's flag registry (`ArgParse.FLAGS`) by the test suite, so it lists exactly the flags the engine reads.

## Campaign selection

### `--campaign=NAME`

Names the campaign archive to load, without the `.zip` extension. The archive sits beside the binary. A campaign packaged as `demo.zip` is loaded with:

```bash
--campaign=demo
```

The default is an empty string, which selects no campaign.

### `--dir=PATH`

Sets the folder where {{ product_name }} looks for campaign archives. An absolute path (or a `res://` / `user://` path) is used as-is. Any other value is a subfolder of the binary's directory. With no `--dir`, the engine looks in a `campaigns` folder next to the binary (next to `Isometry.app` on macOS), then in a `campaigns` folder inside the engine's data folder. `--print-paths` lists every folder it checks. An absolute path is the most reliable choice on macOS, where a quarantined app can run from a temporary location.

## Network mode

### `--network=MODE`

Selects the network role for this run. This is the one argument read during startup rather than in the main argument loop.

Accepted values, all case-insensitive:

- `none`, `n`, or `0`
- `host`, `h`, or `1`
- `server`, `s`, or `2`
- `client`, `c`, or `3`

The default is `0` (none). An integer outside `0`–`3`, or an unrecognized name, falls back to `none` and logs a warning.

Each mode behaves as follows at startup:

- **host** — loads or creates the encryption keys, starts the server, then routes into the loading scene once the server is established.
- **server** — identical to `host` at startup: it loads encryption keys and starts the server.
- **client** — routes straight into the loading scene to connect to a server.
- **none** — does nothing. No campaign loads.

> Warning: `--network=none` from the command line is currently a no-op. The startup match falls through without loading a campaign. Only `host`, `server`, and `client` route into a session. To play alone, use `--network=host`.

For the difference between host, server, and client, see [Hosting](/manual/multiplayer/hosting.html) and [Networking and security](/manual/multiplayer/networking-and-security.html).

## Connection settings

### `--uri=ADDRESS`

The server address a client connects to. The default is `localhost`. It has no effect in host or server mode.

```bash
--uri=localhost
```

### `--port=NUMBER`

The UDP port used to bind the server and to connect a client. The default is `5000`. Both ends must agree on the port.

The parser does not check the range; the value passes through to the engine's networking layer. Ports below 1024 may require elevated privileges on some systems.

## Credentials

### `--username=NAME` and `--secret=PASS`

Supply the login name and secret for an authenticated session. The `--secret` flag is the password.

```bash
--username=p --secret=p
```

Both must be non-empty and must contain none of the following invalid characters:

```
<>:"'/\|?*~!@#$%^&()={}[];.`
```

That set includes the single quote (`'`) and the backtick (`` ` ``). A value that is empty or contains any of these characters fails validation. See [Networking and security](/manual/multiplayer/networking-and-security.html) for how credentials authenticate against the server.

## Display settings

These let a custom launcher size the window and scale the interface without opening
the in-game options menu. Each one overrides the matching saved option for the run.

### `--resolution=WIDTHxHEIGHT`

Sets the window size, for example `--resolution=1920x1080`. The window switches to
windowed mode at that size. A malformed value (anything that is not two positive whole
numbers joined by `x`) is ignored and the saved resolution is used instead.

```bash
--resolution=1280x720
```

### `--interface_scale=FACTOR`

Scales the menus and HUD so they read well on different monitors. `1.0` is normal size;
values above enlarge the interface and values below shrink it. The factor is clamped to
the range `0.5`–`3.0`. The isometric world is not affected — only the interface.

```bash
--interface_scale=1.5
```

The same control is available in the in-game options menu as **Interface Scale**.

## Logging

### `--log-level=LEVEL`

Sets the minimum severity written to the log file. The value is uppercased and matched against the level names. Valid names, from least to most verbose:

```
NONE  FATAL  ERROR  WARN  INFO  DEBUG  TRACE
```

The default is `NONE`, meaning no log output until you raise the level. Logs are written to `log.txt` in [the engine's data folder](/manual/getting-started/install-and-run.html#where-files-go).

```bash
--log-level=INFO
```

At `INFO`, a successful campaign load reports validation passing and entities being created — the signal you watch for when verifying a campaign boots.

## Server administration

A running server or host answers admin commands on `127.0.0.1` (the game port + 1, or `--admin-port`). See [Server administration](/manual/multiplayer/server-administration.html) for the replies and the protocol.

### `--ui-script=FILE`

Plays a script of player input against the running game and checks what it shows (tickets 284-285). The input
goes through the same path as a player's keys and mouse (`Input.parse_input_event`), so a script tests the real
game, bindings included. It runs only with `--network=host`: a script drives this process's own player, never a
client on someone else's server. Every check prints `UI-SCRIPT PASS|FAIL <line>: ...`; the run ends with
`UI-SCRIPT DONE: n passed, m failed` and exits 0, or 1 on any failure. Pair it with `--headless` for a test, or with
`--write-movie` to see it.

One command a line; `#` starts a comment; "quoted text" keeps its spaces.

| Command | What it does |
|---|---|
| `ready [seconds]` | Waits until the player is in the world (in play, or in a dialog such as a new character's welcome). |
| `wait <seconds>` / `mark "<text>"` / `quit` | Waits; logs a marker with the frame number; stops. |
| `press <action>` / `hold <action> <seconds>` | A bound action (Keybinds name, e.g. `toggle_quests_view`, `action_3`), as bound now. |
| `key <key>` | A raw key by name (`escape`, `enter`, `tab`, `f1`, ...). |
| `click "<text>"` / `click_node <Name>` | The visible button or reply whose text contains `<text>`; the visible control with that node name. |
| `hover_node <Name>` | Rests the mouse on a control, so its tooltip opens (then `wait` and `expect_text`). |
| `click_actor <actor> [left\|right]` | An actor by Actor key or node name (the nearest to the player). |
| `click_near_player <dx> <dy> [left\|right]` | A point offset from the player, in world pixels. |
| `type "<text>"` | Characters into whatever has focus. |
| `expect_state <STATE>` | The UI state (`GAMEPLAY`, `INTERACTION`, `MENU_QUESTS`, `MENU_CHARACTER`, ...). |
| `expect_visible <Name>` / `expect_hidden <Name>` | A node by name is (not) visible. |
| `expect_text "<text>"` / `expect_no_text "<text>"` | Text in any visible label, rich text or button. |
| `expect <dice> <op> <dice>` | Dice read with the player as caller and target, e.g. `expect $core_gold_resource >= 10`. |
| `expect_target <actor\|none>` / `expect_map <map>` | The player's target (Actor key) and map. |
| `expect_distance <actor> <op> <px>` | Isometric distance from the player to an actor. |

The Interact flag is its HUD button: `click_node InteractButton`, then `click_actor` or `click_near_player`.

### `--admin-secret=SECRET` and `--admin-port=PORT`

On a server: the secret the admin commands require, and the port they listen on. In control mode: the secret to send and the port to call (default: `--port` + 1).

The secret is the first of: `--admin-secret`, the `ISOMETRY_ADMIN_SECRET` environment variable, the `admin_token` file in the data folder. A server with neither flag nor variable writes a random token to that file. Prefer the environment variable: a process's command line is visible to other users (`ps`), its environment is not. A restart passes the secret to the new server in the environment, never on its command line.

### `--control=info|restart|shutdown`

Sends one admin command to a running server, prints its JSON reply on one line and exits: `0` if the server answered ok, `1` if it refused, `2` if nothing answered. Use `--port` (the game port) or `--admin-port`, and `--uri` (default `127.0.0.1`).

```bash
ISOMETRY_ADMIN_SECRET=change-me isometry --headless --control=shutdown --port=5000
```

## Diagnostic and tooling flags

These flags run a one-shot task and exit instead of starting a session. They are useful for scripting and for authoring tools.

### `--usage`

Prints every flag the engine reads, with its accepted values, default and a one-line description, then exits `0` without starting a session. It works in windowed and headless builds and prints to standard output. The list is generated from the engine's flag registry, the same one this page's summary table is checked against.

```bash
isometry --headless --usage
```

> Note: `--help` and `-h` are the Godot engine's own help: Godot answers them before the game starts, listing engine options such as `--headless` and `--verbose`, not {{ product_name }}'s flags. Use `--usage` for the game's flags.

### `--validate=PATH`

Runs the full campaign validator over a campaign directory or `.zip` and exits, without building the world. It prints `Campaign validation passed` and exits `0` on success, or prints a summary of errors and exits `1` on failure.

```bash
isometry --validate=demo.zip
```

A directory is validated without an archive, so asset-path checks are skipped; a `.zip` runs the full pipeline, including asset validation.

### `--dump-schemas[=FILE]`

Emits the machine-readable entity and validation spec (entity schemas, action functions, asset fields, dice fields, and key aliases) as JSON, then exits. With a file path it writes to that file; with no value it prints to standard output.

```bash
isometry --dump-schemas=schema.json
```

### `--app-version`

Prints the engine version (for example `0.400`) and exits. In headless mode the Godot banner line comes first; the version is the last line of output.

```bash
isometry --headless --app-version
```

### `--regenerate-doc-cache[=FILE]`

Rebuilds the entity documentation cache that the docs and tools read, then exits. With no value it writes the engine's built-in cache file.

### `--print-paths`

Prints the resolved application-data directory, the executable's base directory, whether the binary is running translocated, the resolved campaigns directory, and each candidate path the engine checked. It honors `--dir`. Use it to confirm where the binary is looking for campaigns and writing data.

```bash
isometry --print-paths
```

### `--debug-collision-*`

Draws collision shapes as a colored overlay, one color per layer, to show where actors can walk and interact. Each flag is a boolean. The available layers are:

```
wall  base  hitbox  viewbox  discovery  salience  barrier  navigation
```

Enable one layer with `--debug-collision-<layer>`, or enable every layer at once with `--debug-collision-all`.

```bash
isometry --campaign=demo --network=host --username=p --secret=p --debug-collision-hitbox --debug-collision-navigation
```

## Booting a campaign headless

To load a campaign without a window, pair the `--headless` engine flag with {{ product_name }}'s arguments. Place the campaign `.zip` beside the binary, then run:

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

Because `--network=none` loads nothing, use `--network=host` to exercise the full load pipeline from the command line.

## Troubleshooting

When something fails to start, raise `--log-level` to `INFO` or higher and read `log.txt`. A failure to bind the port, a missing campaign archive, or rejected credentials all surface there.

The messages below are guidance, not literal program output:

- **No campaign loads.** Check that `--campaign` names an archive in the campaigns folder (`--print-paths` shows where that is), and that `--network` is `host`, `server`, or `client` — not `none`.
- **The server will not start.** Another process may hold the port. Try a different `--port`.
- **Login is rejected.** Confirm both `--username` and `--secret` are non-empty and free of the invalid characters listed above.

## See also

- [Install and run](/manual/getting-started/install-and-run.html)
- [Hosting](/manual/multiplayer/hosting.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Troubleshooting](/reference/troubleshooting.html)
- [Glossary](/manual/getting-started/glossary.html)
