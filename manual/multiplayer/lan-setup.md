# LAN Setups

This page walks through the usual ways to play together on one local network (home or office Wi-Fi or Ethernet): one player hosts, a spare machine runs a dedicated server, or everything runs on one computer for testing. Internet and cloud hosting are out of scope here.

## Prerequisites

- [Install and run](/manual/getting-started/install-and-run.html), so the engine runs and you know where the `campaigns` folder is.
- [Hosting a session](/manual/multiplayer/hosting.html), for what each `--network` mode means.

## The rules every setup follows

1. **One machine runs the world.** It is started with `--network=host` (it also plays) or `--network=server` (it only serves). Everyone else joins with `--network=client`.
2. **Everyone has the exact same campaign zip.** Each machine puts the same `.zip` in its own `campaigns` folder. If even one byte differs, the client is refused with `Update to the latest campaign version to connect.` Copy the file; don't rebuild it separately on each machine.
3. **Same port on both sides.** The default is `5000`. The traffic is **UDP**.
4. **Clients need the world machine's address.** On a LAN, that is its local IP address, such as `192.168.1.20`. Clients pass it as `--uri`.
5. **Start the world machine first.** A client that dials before the server is listening fails to connect. Start the client again once the server is up.
6. **Every player uses their own username.** The first login reserves a name for its password, and a name can only be logged in once at a time (see [Reserved usernames](/manual/multiplayer/networking-and-security.html#reserved-usernames)).

### Find the world machine's local IP address

Run this on the machine that will host or serve:

| System | Command | Looks like |
|--------|---------|------------|
| macOS | `ipconfig getifaddr en0` (Wi-Fi or the first network port; try `en1` if it prints nothing) | `192.168.1.20` |
| Windows | `ipconfig`, then read "IPv4 Address" | `192.168.1.20` |
| Linux | `hostname -I` (the first address) | `192.168.1.20` |

Use the address that starts with `192.168.`, `10.`, or `172.16.`–`172.31.`. Those are local addresses. Don't use `127.0.0.1`; that only means "this same machine".

### Let the port through the firewall

Only the world machine needs this.

- **macOS:** the first time the engine listens, macOS asks whether to accept incoming connections. Click **Allow**. If you clicked Deny, open System Settings → Network → Firewall → Options and set Isometry to allow incoming connections.
- **Windows:** when Windows Defender Firewall asks, allow access on **Private networks**. Make sure your network is set to Private, not Public.
- **Linux:** if you use `ufw`, run `sudo ufw allow 5000/udp` (with your port).

You do not need router port forwarding for LAN play. That is only for players outside your network.

## Setup 1: One player hosts

The most common setup. One player runs a host and plays; friends on the same network join.

```
  ┌──────────────── your LAN ────────────────┐
  │  Host PC (plays + serves)  ◀── Client A  │
  │  192.168.1.20 : 5000 UDP   ◀── Client B  │
  └──────────────────────────────────────────┘
```

On the host machine:

```bash
isometry --campaign=mygame --network=host --port=5000 --username=alice --secret=alicepass
```

On each other player's machine:

```bash
isometry --campaign=mygame --network=client --uri=192.168.1.20 --port=5000 --username=bob --secret=bobpass
```

Good to know:

- The world only exists while the host is running. When the host quits, everyone is disconnected.
- Player saves are stored on the host, in its data folder. A player who joins next time (to the same host) continues where they left off.
- The host machine does the most work: it renders the game and also runs every monster. Give that job to the strongest computer.

## Setup 2: A dedicated server on a spare machine

Use a machine nobody plays on (an old laptop, a desktop in the corner, a home server). The world stays up when players come and go.

```
  ┌──────────────────── your LAN ─────────────────────┐
  │  Server PC (headless)        ◀── Client A         │
  │  192.168.1.50 : 5000 UDP     ◀── Client B         │
  │                              ◀── Client C         │
  └───────────────────────────────────────────────────┘
```

On the server machine:

```bash
ISOMETRY_ADMIN_SECRET=change-me isometry --headless --campaign=mygame --network=server --port=5000 --username=server --secret=serverpass --log-level=INFO
```

On every player's machine:

```bash
isometry --campaign=mygame --network=client --uri=192.168.1.50 --port=5000 --username=bob --secret=bobpass
```

Good to know:

- `--headless` runs with no window. A server has no player of its own.
- With an admin secret set, you can check or stop the server from that same machine: `ISOMETRY_ADMIN_SECRET=change-me isometry --headless --control=info --port=5000`. `--control=shutdown` saves every player first. The admin endpoint only listens on the server machine itself. See [Server administration](/manual/multiplayer/server-administration.html).
- To keep it running after a reboot, start it from your system's service manager (launchd, systemd, or Task Scheduler). See the [Player Guide's hosting page](/guide/hosting.html) for a systemd example.
- Back up the `data/` folder in the server's data folder; it holds every player's save.

## Setup 3: Server and player on the same machine

Run a dedicated server in the background and join it from a normal game window on the same computer. Other players on the LAN can still join. This is handy if you want the world to stay up while you close and reopen your own game.

Terminal 1:

```bash
isometry --headless --campaign=mygame --network=server --port=5000 --username=server --secret=serverpass
```

Terminal 2 (you):

```bash
isometry --campaign=mygame --network=client --uri=localhost --port=5000 --username=alice --secret=alicepass
```

Other players use the machine's LAN address (`--uri=192.168.1.20`), exactly as in Setup 2.

## Setup 4: Test multiplayer alone

Creators can check co-op features without a second computer. Start a server, then open two client windows with different usernames:

```bash
isometry --headless --campaign=mygame --network=server --port=5000 --username=server --secret=s --log-level=INFO
isometry --campaign=mygame --network=client --uri=localhost --port=5000 --username=tester1 --secret=t1
isometry --campaign=mygame --network=client --uri=localhost --port=5000 --username=tester2 --secret=t2
```

Every process on one machine writes to the same `log.txt`. To keep the logs apart, give each process its own data folder by starting it with a different `HOME`, for example `HOME=/tmp/p1 isometry ...` on macOS or Linux. Delete the test players' saves between rounds so each test starts fresh.

## When a client can't connect

Work down this list:

| Symptom | Likely cause | Fix |
|---------|--------------|-----|
| `Update to the latest campaign version to connect.` | The zips differ. | Copy the server's exact `.zip` to the client. |
| `Username reserved or incorrect secret` | That name was first used with a different password. | Use the original password, or pick a new name. |
| `This username is already logged in` | Someone (maybe you, in another window) is already playing as that name. | Use a different username. |
| The client hangs, then gives up | Wrong address or port, the server isn't running yet, or the firewall is blocking UDP. | Check `--uri` and `--port` against the server. Make sure the server is up. Allow the port through the server's firewall. |
| Works on the host machine with `localhost`, fails from other machines | The firewall, or a network that keeps devices apart. | Allow the port. Guest Wi-Fi and some mesh or "AP isolation" setups block devices from reaching each other. Put everyone on the main network. |
| The log is empty | Logging is off by default. | Add `--log-level=INFO` and look in `log.txt` in the data folder. |

## See also

- [Hosting a session](/manual/multiplayer/hosting.html)
- [Networking and security](/manual/multiplayer/networking-and-security.html)
- [Server administration](/manual/multiplayer/server-administration.html)
- [CLI reference](/reference/cli.html)
