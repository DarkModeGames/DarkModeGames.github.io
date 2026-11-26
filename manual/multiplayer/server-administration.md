# Server Administration

A running server (`--network=server` or `--network=host`) answers three admin commands: **info**, **restart** and **shutdown**. A launcher or a script uses them to see who is playing, and to stop or restart the server without losing anyone's progress.

## Introduction

Stopping a server by killing its process loses every connected player's unsaved progress: players are saved when they disconnect or when the window is closed, and a killed process does neither. The admin commands save every connected player first, warn them, and then stop or restart the server cleanly.

The commands are sent with the game binary itself in a one-shot *control mode* (`--control=…`), or by any program that can open a TCP connection.

## Prerequisites

- [Hosting](/manual/multiplayer/hosting.html): how to start a server or host.
- The [command-line reference](/reference/cli.html) for every flag.

## Starting a server with admin access

The admin endpoint starts with the server. It listens on **127.0.0.1 only** (the same machine), on the game port + 1, or on `--admin-port`.

Give it a secret, preferably in the `ISOMETRY_ADMIN_SECRET` environment variable, which other users can't read from the process list the way they can read command-line flags:

```bash
ISOMETRY_ADMIN_SECRET=change-me isometry --headless --campaign=demo --network=server --port=5000
```

The server takes the first of these it finds:

1. `--admin-secret=…` on the command line;
2. the `ISOMETRY_ADMIN_SECRET` environment variable;
3. the `admin_token` file in its data folder (the folder `--print-paths` shows as `app_data`). If there is none, the server generates a random token and writes it there, readable only by the user who started it.

A control call looks for its secret in the same order, so a call made by the same user on the same machine finds the token file automatically.

## The commands

Run the binary in control mode. It connects, sends one command, prints the JSON reply on one line, and exits.

```bash
export ISOMETRY_ADMIN_SECRET=change-me
isometry --headless --control=info     --port=5000
isometry --headless --control=restart  --port=5000
isometry --headless --control=shutdown --port=5000
```

`--port` is the server's *game* port (the admin port is worked out from it), or pass `--admin-port` directly. `--uri` defaults to `127.0.0.1`.

Exit status: `0` when the server answered `"ok": true`, `1` when it refused (for example a wrong secret), `2` when no server answered.

### info

```json
{"ok":true,"result":{"status":"running","version":"0.400","campaign":"demo","mode":"server","uptime_s":312,"port":5000,"admin_port":5001,"maps":["town_map","moor_map"],"players":[{"peer_id":148613813,"username":"brann","map":"moor_map"}]}}
```

`players` lists every connected player. A dedicated server has no player of its own; a host lists its own player as peer `1`.

`status` is `booting` while the server is still loading the campaign, spawning its actors and building its maps, and `running` after that. The endpoint answers from the moment the server starts, even while loading keeps the game busy; until loading finishes the other fields may be incomplete (no maps or players yet).

### shutdown

While the server is booting, shutdown and restart answer `{"ok":false,"error":"booting"}`: nobody can be connected yet, so stopping the process loses nothing.

Saves every connected player (the same save as a disconnect), tells the clients "The server is shutting down.", answers with the saved usernames, then exits with code `0`.

```json
{"ok":true,"result":{"action":"shutdown","saved":["brann"]}}
```

### restart

Saves and warns like shutdown ("The server is restarting. Reconnect in a moment."), closes its ports, and starts a fresh server process with the same flags and the same admin secret. The secret is passed to the new process in `ISOMETRY_ADMIN_SECRET`, never on its command line, even if the old server was given `--admin-secret`. The reply carries the new process id; the old process exits with code `0`. Players reconnect to the new server.

```json
{"ok":true,"result":{"action":"restart","pid":21997,"saved":["brann"]}}
```

A launcher that tracks the server process should switch to the new `pid`. Engine-only flags the engine does not pass on to the game (for example `--fixed-fps`) are not carried over; `--headless` is.

## Calling the endpoint without the game binary

The protocol is one line of JSON each way over TCP to `127.0.0.1:<admin port>`:

```
→ {"secret":"change-me","command":"info"}
← {"ok":true,"result":{...}}
```

A request ends at its newline. A request sent without one is answered as soon as it is a complete JSON object, so a client may also close its sending side right after the request (as `nc` does) and still get the reply.

Errors come back as `{"ok":false,"error":"unauthorized"}`, `"unknown command"`, `"bad request"` or `"booting"`. The server closes the connection after replying.

## Security

- The endpoint binds 127.0.0.1 only; it can't be reached from another machine.
- Every command needs the secret, compared in constant time. Players never see it.
- Prefer `ISOMETRY_ADMIN_SECRET` to `--admin-secret`: other users on the machine can see a process's command line. Keep the secret out of shared logs and screenshots. The generated `admin_token` file is readable only by its owner.

## See also

- [Hosting](/manual/multiplayer/hosting.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Command-line reference](/reference/cli.html)
