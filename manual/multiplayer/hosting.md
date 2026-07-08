# Hosting Multiplayer Games

{{ product_name }} has no separate backend, no Docker, and no admin panel. You run the same game binary in a different network mode, set with one command-line flag. This page explains the four modes and the flags that configure a session.

## Introduction

Multiplayer in {{ product_name }} is entirely command-line driven. There is no hosting config file in a [campaign](/manual/getting-started/glossary.html); a campaign is just data, and you choose its network role when you launch it. The role is set by `--network`, which selects one of four modes: none, host, server, or client.

This page covers what each mode does, the flags that start a session, their defaults, and the behavioral differences between hosting and joining. Authentication and the on-disk save format are introduced here only briefly; the full detail lives on [Networking and security](/manual/multiplayer/networking-and-security.html).

## Prerequisites

Before hosting a game, you should be able to install the engine and load a campaign:

- [Install and run](/manual/getting-started/install-and-run.html) — get the binary in place.
- [Campaign structure](/manual/core-concepts/campaign-structure.html) — how a `.zip` campaign is packaged and where it sits relative to the binary.

## Network modes

A *network mode* tells the game whether it is playing, serving, or connecting. There are four modes:

| Mode     | Number | What it does                                                  |
|----------|--------|--------------------------------------------------------------|
| `NONE`   | `0`    | No networking. Currently a no-op (see the limitation below). |
| `HOST`   | `1`    | Starts a server **and** plays locally in the same process.   |
| `SERVER` | `2`    | Starts a server only. Intended to run headless.              |
| `CLIENT` | `3`    | Connects to a running server.                                |

You select a mode by passing `--network`. The flag accepts three forms, so all of these select host mode:

```bash
isometry --network=host
isometry --network=h
isometry --network=1
```

The name is matched case-insensitively, the single letter is the first character of the name (`n`, `h`, `s`, `c`), and the number is the value from the table above. Any value that is not one of these falls back to `NONE` with a warning.

> Warning: `--network=none` (and omitting `--network`, which defaults to `0`) starts nothing. The mode currently does no work: it loads no campaign and starts no server. Use `host` to run and verify a campaign locally. See [Troubleshooting](/reference/troubleshooting.html) for this limitation.

### Host vs. server

Host and server both start a server, in exactly the same way: both open a network listener on the chosen port. The difference is intent, not startup behavior.

- **Host** is for a player who also runs the game. It renders the world and plays locally while serving other clients.
- **Server** is for a dedicated, unattended process. You run it headless so it does no rendering.

{{ product_name }} does not force a server to be headless. Headless is your choice at launch, made with the `--headless` flag:

```bash
isometry --headless --campaign=demo --network=server --port=5000 --username=admin --secret=secret
```

In every mode with a window, the title shows the campaign name followed by the mode name, for example `demo HOST` or `demo SERVER`. A headless server has no window.

## Configuring a session

Beyond `--network`, a handful of flags configure the session. All values arrive as strings from the command line.

| Flag          | Applies to        | Default       | Meaning                                              |
|---------------|-------------------|---------------|------------------------------------------------------|
| `--network`   | all               | `0` (`NONE`)  | The network mode.                                    |
| `--campaign`  | all               | `""` (empty)  | The campaign key, for example `demo`.                |
| `--port`      | host, server, client | `5000`     | The UDP port the server listens on / client dials.  |
| `--uri`       | client            | `localhost`   | The server address a client connects to.            |
| `--username`  | host, server, client | `""` (empty) | The account name used for authentication.           |
| `--secret`    | host, server, client | `""` (empty) | The account password.                               |
| `--dir`       | all               | —             | Override the directory for campaign and save files.  |
| `--log-level` | all               | `NONE`        | How much the engine logs (see below).               |

Flags accept two spellings: `--key=value` (preferred) and `--key value` (space-separated). A bare `-flag` with a single dash is read as the boolean `true`. There is no short-flag aliasing for these long flags.

### Ports

The server opens a port and clients dial the same port. The default is `5000`. A port must be an unused number from `0` to `65535`; ports below `1024` are privileged on most systems and usually require elevated rights.

Pick the same port on both ends. On the server:

```bash
isometry --headless --campaign=demo --network=server --port=7777 --username=admin --secret=secret
```

On a client:

```bash
isometry --campaign=demo --network=client --uri=192.168.1.20 --port=7777 --username=alice --secret=hunter2
```

Note the client uses `--uri`, not `--ip`. The address can be a hostname or an IP; it defaults to `localhost`, which is useful for testing a client and server on the same machine.

> Note: The network uses **UDP** only. If you host across a network, forward the server port as UDP and allow UDP through any firewall. This is standard UDP networking, not an {{ product_name }}-specific setting.

### Usernames and passwords

Every networked session needs a username and password. Both must be non-empty and may not contain invalid characters. Player logins are encrypted automatically.

The username and password together identify a player's saved data. How logins are protected and where saves are stored are covered on [Networking and security](/manual/multiplayer/networking-and-security.html).

```bash
isometry --campaign=demo --network=host --port=5000 --username=alice --secret=hunter2
```

> Warning: In a trial build, authentication always fails, a server caps its client slots at zero, and a client returns without connecting. Use a full build for real multiplayer.

### Maximum clients

A server accepts up to `4095` players. This is a fixed ceiling, not a recommended player count. The practical number of players a campaign supports depends on the campaign and your hardware.

## Logging

By default the engine logs nothing. To see startup and validation output, pass `--log-level` with one of these names, matched case-insensitively:

```
NONE  FATAL  ERROR  WARN  INFO  DEBUG  TRACE
```

`NONE` (the default) silences all output; `TRACE` is the most verbose. Logs are written to a file named `log.txt` in the working directory, not to standard output.

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

## Verify a host locally

To confirm a campaign loads end to end, run it as a host with logging on. Place the campaign `.zip` beside the binary, then launch:

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

The campaign `.zip` must sit in the same directory as the executable. Open `log.txt` and look for two success markers: `Campaign validation passed` and `Successfully created Entity`. If you do not see them, see [Troubleshooting](/reference/troubleshooting.html).

## See also

- [Install and run](/manual/getting-started/install-and-run.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [CLI reference](/reference/cli.html)
- [Troubleshooting](/reference/troubleshooting.html)
