# LinkHarbor for Steam and Source servers

A single-file Python agent that polls a game server over the Steam A2S query protocol and shows it on your [LinkHarbor](https://linkharbor.com) bio page: online or offline, "12 / 50" players, player names, map and version. Joins and leaves become events your followers can get push notifications for.

One agent covers every game that answers A2S queries: Rust, ARK, Valheim, Palworld, Counter-Strike 2, Garry's Mod, Team Fortress 2, Satisfactory, 7 Days to Die, Enshrouded, V Rising, Conan Exiles, DayZ, Squad, Project Zomboid, Killing Floor 2, Insurgency, Left 4 Dead 2 and more.

It runs anywhere with Python 3.10 or newer and network access to the server's query port: on the game server, a VPS or a Raspberry Pi.

Part of the [LinkHarbor integrations](https://github.com/linkharbor/integrations) family, one repo per game.

## Setup

```bash
git clone https://github.com/linkharbor/integration-steam
cd integration-steam
pip install -r requirements.txt

export LH_INGEST_URL="https://dock.at-harbor.com/v1/ingest/123"   # dashboard: Integrations, Custom (API)
export LH_INGEST_SECRET="…"

python3 agent.py --host 203.0.113.10 --port 27015 \
  --name "My Rust Server" --address "203.0.113.10:28015" \
  --url "https://discord.gg/xyz" --interval 60
```

Start with `--once -v`: the agent queries the server once, prints the answer and sends one state. Then run it as a service, see `integration-steam.service` for systemd. The two secrets go into `/etc/integration-steam.env` with `chmod 600`.

## Query ports per game

`--port` is the query port. It is often the game port, but not always.

| Game | Query port (default) | Note |
|---|---|---|
| Rust | game port + 1 (28016) | the player list needs `server.queryport` enabled |
| ARK: Survival Evolved / Ascended | 27015 | game port 7777 |
| Valheim | game port + 1 (2457) | start with `-public 1` |
| Palworld | 27015 | enable `EnableSteamQuery` or `bIsUseAuthSteam` |
| Counter-Strike 2 | 27015 | |
| Garry's Mod, TF2, L4D2, CS:GO | 27015 | |
| Satisfactory | 15777 | |
| 7 Days to Die | game port (26900) | |
| Enshrouded | query port from `enshrouded_server.json` (15637) | |
| V Rising | query port from the settings (9877) | |
| Conan Exiles | 27015 | |
| DayZ | game port + 1 (2303) | |
| Squad | 27165 | |
| Project Zomboid | 16261 | |

If your game is missing, run `--once -v`. If the server answers, it works. Open an issue with the port and we add it to the table.

## Events sent

| Event | When | Template variables |
|---|---|---|
| `server.online` | the first successful query, or the server is back after being unreachable | `{name}`, `{players_online}`, `{players_max}`, `{map}` |
| `server.offline` | the server stopped answering | `{name}` |
| `player.joined` | a new name appears in the player list | `{player}` and the above |
| `player.left` | a name disappears | `{player}` and the above |

The state is sent with every event and every `--heartbeat` minutes (default 10). `--no-player-names` sends counts only.

Suggested push rule (dashboard, Integrations, Push rules, Custom (API), event `player.joined`): batch window 10 minutes, title "🎮 {count} players joined", body "{player_list} are now on {name}", minimum pause 5 minutes.

## Options

```
--host HOST            game server IP or hostname (required)
--port PORT            A2S query port (default 27015)
--name NAME            card title (default: the server's own name)
--address ADDRESS      address shown on the card
--url URL              link behind the card's button
--cta TEXT             button label (default "Join server")
--game TEXT            game label (default from A2S)
--image-url URL        banner image for the large layout
--interval SECONDS     polling interval (default 60, minimum 5)
--heartbeat MINUTES    state-only update interval (default 10, 0 = only on change)
--no-player-names      send counts only
--once                 query once, send, exit
-v                     verbose logging
```

## License

[MIT](LICENSE). © 2026 EMI Solutions OÜ.
