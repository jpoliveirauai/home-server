# home-server

Repository documenting the *arr stack ("YAMS") running on my self-hosted media
server, managed with Docker and [Portainer](https://www.portainer.io/). This repo
tracks the stack definitions only — personal media libraries and secrets are
excluded (see [`.gitignore`](.gitignore)).

## Stack

Managed via `docker compose`, with [Portainer](https://www.portainer.io/) for
day-to-day container management/inspection.

* Prowlarr — indexer aggregator
* Sonarr / Radarr / Lidarr — TV / Movies / Music automation
* qBittorrent / SABnzbd — torrent / NZB download clients (routed through a VPN via
  gluetun)
* slskd + soularr — Soulseek source for Lidarr
* Bazarr — subtitles
* Plex — media server
* Portainer — container management UI
* Watchtower — automatic image updates

See [`docs/architecture.md`](docs/architecture.md) for the full service data flow,
repo layout, and stack lifecycle commands.

## Docs

* [`docs/architecture.md`](docs/architecture.md) — service architecture, repo
  layout, lifecycle commands
* [`yams_services.txt`](yams_services.txt) — quick reference of service URLs on the
  LAN
