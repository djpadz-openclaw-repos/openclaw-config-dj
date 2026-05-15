# MEDIA.md — Media Server Stack

All credentials in 1Password → `Arr apps` item (OpenClaw vault). Overseerr key: use raw base64 value, NOT decoded.

## Services

- **Sonarr:** `http://dindjarin.tail1916d.ts.net:8989`
- **Radarr:** `http://dindjarin.tail1916d.ts.net:7878`
- **Lidarr:** `http://dindjarin.tail1916d.ts.net:8686` — **standup comedy only** (music is in Apple Music)
- **Bazarr:** `http://dindjarin.tail1916d.ts.net:6767`
- **Overseerr:** `https://overseerr.sandiego.padz.net`
- **Tautulli:** `http://dindjarin.tail1916d.ts.net:8181`
- **Jackett:** `http://dindjarin.tail1916d.ts.net:9117`
- **NZBGet:** `http://dindjarin.tail1916d.ts.net:6789`
- **Deluge:** `http://dindjarin.tail1916d.ts.net:8112`
- **Plex server:** dindjarin (same host)

## Users & Access

- **John Beck** = Overseerr user id 5, Tautulli user id 600993302 (`sandiegob`) — friend with server access who requests a lot of current/new movies

## Movie Pruning Policy

Keep: classics (traditional + cult), complete series sets, Chaplin collection, Muppets, Mel Brooks, Steve Martin, Star Trek complete set (all films), Star Wars, Looney Tunes

Remove: low-rated standalones nobody's watched

John's unwatched requests: stay until he watches them + 2 week grace period

## Automation

**Sonarr notifier:** `media-automation/notify_sonarr.py` — runs every 15 min via cron job, Telegrams when new episodes imported

Repo: https://github.com/djpadz-openclaw-gh/media-automation

Cron job ID: `5f710a8e-d441-4219-9c72-651113b7e0b0` (Sonarr notifier)
