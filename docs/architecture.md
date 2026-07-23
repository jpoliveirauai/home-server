# Architecture

All YAMS containers share a dedicated bridge network `yams_network`
(`172.60.0.0/24`) with static IPs per service. Rough data flow:

1. **Prowlarr** — indexer aggregator; feeds indexers to Sonarr/Radarr/Lidarr.
2. **Sonarr / Radarr / Lidarr** — decide what to grab (TV/Movies/Music), push jobs to
   the download clients, then import completed media into `movies/` / `tvshows/` /
   `music/`.
3. **qBittorrent / SABnzbd** — actual downloaders. Both are routed through
   **gluetun** (`network_mode: "service:gluetun"`) so all torrent/NZB traffic goes
   through the VPN (NordVPN via OpenVPN); they have no direct network of their own.
4. **slskd + soularr** — Soulseek client + bridge that lets Lidarr source music from
   Soulseek the same way it sources from torrent/NZB indexers.
5. **Bazarr** — fetches subtitles for content Sonarr/Radarr have imported.
6. **Plex** — serves the final media library; runs with `network_mode: host` (not on
   `yams_network`) so DLNA/local-network discovery works.
7. **Portainer** — container management/inspection UI for the whole Docker host.
8. **Watchtower** — auto-updates container images.

Service URLs are listed in [`yams_services.txt`](../yams_services.txt) (LAN IP
`192.168.1.7`).

## Repo layout

- `config/yams/` — the YAMS stack install directory.
  - `docker-compose.yaml` — main stack definition (all *arr apps, Plex, VPN, etc).
  - `docker-compose.custom.yaml` — extension point for adding services to
    `yams_network` without touching the main compose file.
  - `.env` — stack-wide config (not committed — see `.gitignore`).
  - `config/<service>/` — one persistent, stateful config volume per container
    (not committed).
- `config/portainer/docker-compose.yaml` — standalone Portainer instance, separate
  from the one defined inside the YAMS stack, also joined to `yams_network` plus a
  `server-network`.

Media libraries, downloads, secrets, and the standalone `sftpgo/` instance live in
this same home directory but are intentionally out of scope for this repo (see
`.gitignore`) — this repo documents the arr stack only.

## Lifecycle

```
docker compose -f config/yams/docker-compose.yaml up -d
docker compose -f config/yams/docker-compose.yaml down
```

Run from `config/yams/`. Ports are intentionally commented out on several services
(Plex, qBittorrent, SABnzbd) because they route through `gluetun` or `host`
networking instead of publishing directly.
