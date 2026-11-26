# Networking and Security

This page explains how {{ product_name }} multiplayer protects player logins, and what you, the operator, must provide on top of what the game offers. Read it before you open a server to other players.

## Introduction

In {{ product_name }} multiplayer, one machine runs the game (the server); players connect to it. Before any player can join, the game does two things:

- It encrypts player logins, so a username and password never cross the network in plain text.
- It checks that everyone is running the same [campaign](/manual/getting-started/glossary.html) version.

Everything beyond those two protections — who is allowed to log in, how the host machine is secured, how the server stays online — is your responsibility. This page covers the protections the game provides, where its files live on disk, and the gaps you must fill yourself.

## Prerequisites

- [Hosting a campaign](/manual/multiplayer/hosting.html) — how to start a server and connect a client.
- [CLI reference](/reference/cli.html) — the command-line flags referenced below.

It also helps to understand [campaign structure](/manual/core-concepts/campaign-structure.html) and [validation](/manual/core-concepts/validation.html), because the campaign version check happens only after a campaign validates.

## Network modes

{{ product_name }} runs in one of four network modes, chosen with the `--network` flag:

| Mode | Value | What it does |
|------|-------|--------------|
| `none` | `0` | Single-process, no networking. Currently a no-op: it loads no campaign. |
| `host` | `1` | Runs a server *and* spawns a local player on the same machine. |
| `server` | `2` | Runs a headless, dedicated server with no local player. |
| `client` | `3` | Connects to a server at a URI and port. |

You can pass the mode as its number, its full name (case-insensitive), or its single letter (`n`, `h`, `s`, `c`). An invalid or out-of-range value falls back to `none`.

The default URI is `localhost` and the default port is `5000`.

`host` and `server` differ only in the local player. Both start listening for clients. A `host` also creates a local character for the operator; a pure `server` stays headless and waits for clients.

> Note: `none` mode loads nothing today. To exercise the load pipeline on a single machine, use `host`.

## The server

The server is the one machine that runs the game; everyone else connects to it as a client. For public access, you forward a single UDP port — the server port — and nothing else.

## How logins are protected

Player logins are encrypted, so a username and password never cross the network in plain text. The encryption keys are created automatically the first time you launch a server and are stored locally on the host machine; you do not set them up or manage them.

Because a username and password are joined with a period internally, neither may contain one (see [Valid credentials](#valid-credentials)).

## Campaign version check

The game checks that each connecting player is running the same campaign version as the server. If their campaign differs in any way, they are refused. This check happens before the login is checked.

When a player's campaign does not match the server's, they cannot join and see this error:

```
Update to the latest campaign version to connect.
```

To fix a mismatch, give your players the server's exact campaign `.zip`.

> Warning: Editing any campaign file changes its version. After you update a server's campaign, every player must receive the updated `.zip` or they will be rejected.

<a id="valid-credentials"></a>
## Valid credentials

A username and password each must be non-empty and may not contain any of the rejected characters. There is no length requirement.

The rejected characters are:

```
<>:"'/\|?*~!@#$%^&()={}[];.`
```

> Note: There is no registration step. The first login of a username claims it; after that only the same password gets in (see [Reserved usernames](#reserved-usernames)). You still decide who may connect by controlling access to the server.

<a id="reserved-usernames"></a>
## Reserved usernames

A server reserves each username for the password that first used it. Usernames are compared without regard to case or surrounding spaces, so `Kilthunox` and ` kilthunox ` are the same name.

- **A new name** is reserved for the password it logs in with.
- **The same name with the same password** logs in and resumes its save.
- **The same name with a different password** is refused. The player sees this message and a Quit button:

  ```
  Username reserved or incorrect secret
  ```

- **A name that is already playing** cannot log in a second time. The new login is refused with the message below, and the player already in the game is not disturbed:

  ```
  This username is already logged in
  ```

The host's own player follows the same rules. A dedicated server has no player of its own and reserves nothing.

The reservations are kept per campaign in `data/usernames.<campaign>.json`, beside the saves. Each entry holds a random salt and a salted, repeatedly hashed form of the password; the password itself is never written to disk or to the log.

**Upgrading a server that already has saves.** Older saves have no reservation. A name is reserved the first time someone logs in with it after the upgrade. If two old saves share a username with different passwords, whichever logs in first keeps the name; the other is refused from then on, and its save file is left as it was. To give the name back, stop the server, remove that name's entry from `data/usernames.<campaign>.json`, and start the server again.

## What an arriving player sees

A player who logs in or changes map receives every actor on that map, then, in one batch, each actor's current resources: health, loot counts, flags and the rest. Health bars on focus plates and `{% raw %}{{@x}}{% endraw %}` dialog text show the values as they are now, not as they were when the actor first appeared. After that, each change is sent as it happens. Measures are worked out on the player's own machine from those resources.

The batch carries only the values that differ from each Resource's `default`, so it stays small: a town of about 230 actors is about 7 KB.

## Where files live on disk

All runtime files live in [the engine's data folder](/manual/getting-started/install-and-run.html#where-files-go): `~/Library/Application Support/isometry/` on macOS, `~/.local/share/isometry/` on Linux, and `%APPDATA%\isometry\` on Windows. Run `isometry --print-paths` to see the exact folder.

| File | Path (inside the data folder) | Contents |
|------|-----------------------------|----------|
| Encryption keys | `.rsa/` | The server's login encryption keys. Created automatically; never share. |
| Log | `log.txt` | The engine log (see `--log-level`). |
| Player saves | `data/` | One save file per player. |
| Reserved usernames | `data/usernames.<campaign>.json` | Each username's salt and hashed password. |

Player saves are **plain JSON**. They are not encrypted. As the operator you can read and edit them, and so can anyone with file access to the server. A save holds the player's character, map, location, resources, and discovery. When a player reconnects, the server loads their existing save. On disconnect, the server saves the player's character automatically.

> Warning: Protect the `.rsa/` and `data/` directories with file permissions. They hold the login encryption keys and the player saves.

## What the game does not provide

{{ product_name }} encrypts player logins and checks campaign versions. It does **not** give you:

- Registration, password changes, or password reset (usernames are reserved on first use; see [Reserved usernames](#reserved-usernames)).
- Rate limiting, IP banning, or protection against attacks that flood the server.
- Encrypted gameplay. Only the login is encrypted; the rest of the network traffic is not.

You must supply these yourself: a firewall, access controls, monitoring, and a policy for who is allowed to connect. Do not plan a deployment around features the game does not have.

## Maximum clients

The server accepts up to `4095` players. This is a fixed ceiling, not a setting. There is no `--max-clients` option; the session flags are `--uri`, `--port`, `--campaign`, `--network`, `--dir`, `--username`, and `--secret` (see the [CLI reference](/reference/cli.html) for the full list).

> Note: Trial builds disable multiplayer entirely, so trial users cannot host or join networked sessions.

## Deployment scenarios

**Local co-op or testing.** Run one machine in `host` mode. The operator plays locally while others connect over the LAN. No port forwarding is needed for same-network play beyond the host's firewall. See [LAN setups](/manual/multiplayer/lan-setup.html) for step-by-step commands.

**Dedicated server.** Run `server` mode on a machine that no one plays on. Forward the single UDP server port, restrict file access, and back up the `data/` directory. Keep the `.rsa/` directory off any shared volume.

**Updating a live campaign.** Edit the campaign, rebuild the `.zip`, and restart the server. The version changes, so distribute the new `.zip` to every player before they reconnect; otherwise they hit the campaign mismatch error.

## Verifying a host session

To confirm a campaign loads and a login works on one machine, put your campaign `.zip` in the `campaigns` folder and start a host session with logging on:

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

A successful boot logs `Campaign validation passed` and `Successfully created Entity` lines.

## See also

- [Hosting a campaign](/manual/multiplayer/hosting.html)
- [CLI reference](/reference/cli.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Validation](/manual/core-concepts/validation.html)
