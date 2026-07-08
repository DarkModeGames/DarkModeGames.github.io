# Command-Line Reference

This page documents every command-line argument the {{ product_name }} binary parses, with accepted values, defaults, and runtime effect. Use it when you run a server, automate a build, or boot a [campaign](/manual/getting-started/glossary.html#terms) headless.

## Introduction

{{ product_name }} is a single binary that reads command-line arguments at startup to pick a campaign, choose a network mode, and supply credentials. The launcher build collects the same settings through its UI, but headless and server builds rely entirely on these flags.

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

> Warning: On a launcher build, {{ product_name }} routes straight to the launcher UI and ignores every gameplay argument below. The CLI takes effect only on non-launcher builds, such as a headless server.

## Argument summary

| Argument | Values | Default | Effect |
| --- | --- | --- | --- |
| `--campaign` | campaign name (no `.zip`) | `""` (empty) | Selects the campaign archive to load. |
| `--network` | `none`/`host`/`server`/`client`, `0`–`3`, or `n`/`h`/`s`/`c` | `0` (none) | Chooses the network role. |
| `--uri` | hostname or IP | `localhost` | Server address a client connects to. |
| `--port` | port number | `5000` | UDP port for both server bind and client connect. |
| `--username` | non-empty string | (empty) | Login name. |
| `--secret` | non-empty string | (empty) | Login secret (password). |
| `--dir` | subdirectory name | `""` (empty) | Campaign subdirectory under the binary's directory. |
| `--resolution` | `WIDTHxHEIGHT` | saved option | Window size, e.g. `1920x1080`. Overrides the saved option. |
| `--interface_scale` | number `0.5`–`3.0` | saved option | Scales the on-screen menus and HUD. `1.0` is normal size. |
| `--log-level` | `NONE`/`FATAL`/`ERROR`/`WARN`/`INFO`/`DEBUG`/`TRACE` | `NONE` | Minimum severity written to the log file. |
| `--validate` | directory or `.zip` path | — | Validates a campaign and exits. |
| `--dump-schemas` | output file path (optional) | — | Emits the entity/validation spec as JSON and exits. |
| `--print-paths` | (boolean flag) | — | Prints the resolved data and campaign paths and exits. |
| `--debug-collision-*` | (boolean flags) | off | Draws collision shapes for one or more layers. |

## Campaign selection

### `--campaign=NAME`

Names the campaign archive to load, without the `.zip` extension. The archive sits beside the binary. A campaign packaged as `demo.zip` is loaded with:

```bash
--campaign=demo
```

The default is an empty string, which selects no campaign.

### `--dir=PATH`

Sets a subdirectory, relative to the binary's directory, where {{ product_name }} looks for campaign assets. The default is empty, meaning the binary's own directory is used. The value is appended onto that directory rather than replacing it, so treat `--dir` as a subfolder name, not an absolute override.

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

> Warning: `--network=none` from the command line is currently a no-op. The startup match falls through without loading a campaign, so single-player only works through the launcher today. Only `host`, `server`, and `client` route into a session.

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

The default is `NONE`, meaning no log output until you raise the level. Logs are written to `log.txt` in the binary's directory.

```bash
--log-level=INFO
```

At `INFO`, a successful campaign load reports validation passing and entities being created — the signal you watch for when verifying a campaign boots.

## Diagnostic and tooling flags

These flags run a one-shot task and exit instead of starting a session. They are useful for scripting and for authoring tools.

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

- **No campaign loads.** Check that `--campaign` names an archive that exists beside the binary, and that `--network` is `host`, `server`, or `client` — not `none`.
- **The server will not start.** Another process may hold the port. Try a different `--port`.
- **Login is rejected.** Confirm both `--username` and `--secret` are non-empty and free of the invalid characters listed above.

## See also

- [Install and run](/manual/getting-started/install-and-run.html)
- [Hosting](/manual/multiplayer/hosting.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Troubleshooting](/reference/troubleshooting.html)
- [Glossary](/manual/getting-started/glossary.html)
