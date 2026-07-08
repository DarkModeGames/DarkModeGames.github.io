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

> Note: There is no account database. Any non-empty, valid username and password creates or resumes a save. You decide who may connect by controlling access to the server, not through a registration system the game provides.

## Where files live on disk

All runtime files live **beside the executable**, in the directory that holds the {{ product_name }} game.

| File | Path (relative to the game) | Contents |
|------|-----------------------------|----------|
| Encryption keys | `.rsa/` | The server's login encryption keys. Created automatically; never share. |
| Player saves | `data/` | One save file per player. |

Player saves are **plain JSON**. They are not encrypted. As the operator you can read and edit them, and so can anyone with file access to the server. A save holds the player's character, map, location, resources, and discovery. When a player reconnects, the server loads their existing save. On disconnect, the server saves the player's character automatically.

> Warning: Protect the `.rsa/` and `data/` directories with file permissions. They hold the login encryption keys and the player saves.

## What the game does not provide

{{ product_name }} encrypts player logins and checks campaign versions. It does **not** give you:

- An account database, registration, or password reset.
- Rate limiting, IP banning, or protection against attacks that flood the server.
- Encrypted gameplay. Only the login is encrypted; the rest of the network traffic is not.

You must supply these yourself: a firewall, access controls, monitoring, and a policy for who is allowed to connect. Do not plan a deployment around features the game does not have.

## Maximum clients

The server accepts up to `4095` players. This is a fixed ceiling, not a setting. There is no `--max-clients` option; the session flags are `--uri`, `--port`, `--campaign`, `--network`, `--dir`, `--username`, and `--secret` (see the [CLI reference](/reference/cli.html) for the full list).

> Note: Trial builds disable multiplayer entirely, so trial users cannot host or join networked sessions.

## Deployment scenarios

**Local co-op or testing.** Run one machine in `host` mode. The operator plays locally while others connect over the LAN. No port forwarding is needed for same-network play beyond the host's firewall.

**Dedicated server.** Run `server` mode on a machine that no one plays on. Forward the single UDP server port, restrict file access, and back up the `data/` directory. Keep the `.rsa/` directory off any shared volume.

**Updating a live campaign.** Edit the campaign, rebuild the `.zip`, and restart the server. The version changes, so distribute the new `.zip` to every player before they reconnect; otherwise they hit the campaign mismatch error.

## Verifying a host session

To confirm a campaign loads and a login works on one machine, place your campaign `.zip` beside the game and start a host session with logging on:

```bash
isometry --headless --campaign=demo --network=host --port=5000 --username=p --secret=p --log-level=INFO
```

A successful boot logs `Campaign validation passed` and `Successfully created Entity` lines.

## See also

- [Hosting a campaign](/manual/multiplayer/hosting.html)
- [CLI reference](/reference/cli.html)
- [Campaign structure](/manual/core-concepts/campaign-structure.html)
- [Validation](/manual/core-concepts/validation.html)
