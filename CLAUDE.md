# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this directory is

This is **not a software project** — it's the home directory of a self-hosted media
server ("YAMS" stack) running on Docker. There is no application source code to build,
lint, or test. Work here is almost always: editing `docker-compose.yaml` / `.env` files,
troubleshooting a container, or managing media library layout.

## Layout

- `config/yams/` — the YAMS stack install directory (`INSTALL_DIRECTORY`).
  - `docker-compose.yaml` — main stack definition (all *arr apps, Plex, VPN, etc).
  - `docker-compose.custom.yaml` — empty extension point for adding services to the
    same `yams_network` without touching the main compose file.
  - `.env` — stack-wide config (PUID/PGID, media/install paths, VPN credentials).
  - `config/<service>/` — one persistent config volume per container (Sonarr, Radarr,
    Lidarr, Prowlarr, Bazarr, qBittorrent, SABnzbd, Plex, gluetun, Portainer, autobrr,
    soularr, slskd).
- `config/.env` — a duplicate of the stack `.env` at the home-directory root.
- `config/portainer/docker-compose.yaml` — standalone Portainer instance (separate from
  the one defined inside the YAMS stack), also joined to `yams_network` plus a
  `server-network`.
- `movies/`, `tvshows/`, `music/`, `photos/`, `books/` — `MEDIA_DIRECTORY`, mounted as
  `/data` into Plex and the *arr apps.
- `blackhole/` — download blackhole/watch folder used by the download clients.
- `sftpgo/` — standalone SFTPGo instance (SFTP file access), with its own `config/`
  (host keys, sqlite db) and `data/`.
- `data/jpoliveirauai/` — misc app data outside the YAMS stack.
- `yams_services.txt` — quick reference of service URLs on the LAN (`192.168.1.7`).
- `.opencode/` — local OpenCode CLI install (unrelated to the media stack).

## Service architecture

All YAMS containers share a dedicated bridge network `yams_network` (`172.60.0.0/24`)
with static IPs per service. Rough data flow:

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

Service URLs are listed in `yams_services.txt` (LAN IP `192.168.1.7`).

## Working with this stack

- Stack lifecycle: `docker compose -f config/yams/docker-compose.yaml up -d` /
  `down` from `config/yams/`. Custom additions go in `docker-compose.custom.yaml`
  rather than editing the generated main file, and note the file's own comment: the
  `services:` key must have no leading/trailing spaces when uncommented, or the
  compose merge fails.
- Per-service settings persist in `config/yams/config/<service>/` — treat these as
  stateful data (databases, `config.xml`, logs), not disposable config.
- `config/yams/.env` and `config/.env` contain live secrets (VPN credentials, and
  `config/yams/config/soularr/config.ini` / other per-service configs contain API
  keys) — never print their contents back in full, commit them anywhere, or paste
  them into logs/messages.
- Ports are intentionally commented out on several services (Plex, qBittorrent,
  SABnzbd) because they route through `gluetun` or `host` networking instead — don't
  "fix" this by uncommenting them without understanding why.
- `.gitignore` scopes this repo to the arr stack: personal media (`movies/`,
  `tvshows/`, `music/`, `photos/`, `books/`), downloads/blackhole/data, live secrets
  (`.env`), stateful per-service config under `config/yams/config/` and
  `config/yams/autobrr/`, the standalone `sftpgo/` instance's keys/db, and personal
  home dotfiles are all ignored. After changing it, run `git status` to confirm only
  arr-stack files (compose files, `README.md`, `docs/`, `yams_services.txt`,
  `CLAUDE.md`) show up as trackable.

## Documentation

- Always update `README.md` when the stack's services, layout, or lifecycle commands
  change — treat it as the source of truth for what this repo documents, not the
  personal media/home-directory contents.
- For anything longer than a README section (architecture details, runbooks,
  migration notes), add a file under `docs/` and link it from `README.md` rather than
  growing the README itself.
