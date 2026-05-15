# INFRASTRUCTURE.md — Network, k8s, System Config

## Memory Directory Symlinks (Permanent Architecture Fix)

**Problem:** Multi-profile workspace structure has session-memory hook writing daily files to `~/.openclaw/workspace-<profile>/memory/`, but dreaming looks in `~/.openclaw-<profile>/memory/`.

**Solution:** Symlinks bridge the gap:
- `~/.openclaw-dj/memory` → `~/.openclaw/workspace-dj/memory`
- `~/.openclaw-sherra/memory` → `~/.openclaw/workspace-sherra/memory`

**Why this works:**
- Preserves multi-profile workspace structure (profile state separate from workspace)
- Allows dreaming to find daily memory files without code changes
- Transparent to all tools (symlinks are invisible)
- Survives reboots (persistent filesystem links)
- No path configuration needed

**Maintenance:** If adding new profiles, create the corresponding symlink:
```bash
ln -s /home/openclaw/.openclaw/workspace-<profile>/memory /home/openclaw/.openclaw-<profile>/memory
```

**Setup date:** 2026-04-10

## Infrastructure Work (2026-04-06)

**mem0 path fixes:**
- `context-writer.ts`, `reader.ts`, `db.ts` all had hardcoded paths pointing to `~/.openclaw` instead of `~/.openclaw-dj`
- Fixed all three to point to the correct workspace
- Copied existing `mem0-context.md` to the correct location
- Now mem0 will write distilled memories to the right place and I'll actually read them at startup

**Cleanup:**
- Removed `~/.openclaw` directory entirely — it was a ghost/stub from the original profile before the split
- `.openclaw-dj` is now the one true profile
- Verified systemd services are all correct and will auto-start on reboot
- Lingering is enabled, so user services start on boot without login

## Kiro Gateway (Current State)

**Last Updated:** 2026-04-26

- **Current deployment:** `kiro-gateway` namespace, single-account setup, ClusterIP on port 9000
- **Previous:** `kiro-multi` namespace (multi-account with AWS Builder ID + GitHub OAuth) — decommissioned ~2026-04-25
- **Cron job:** `refresh-aws-builder-id-token.py` updated to gracefully skip when `kiro-multi` doesn't exist
- **If restoring multi-account:** recreate `kiro-multi` namespace with `kiro-gateway-config` secret containing `ACCOUNT_1_REFRESH_TOKEN`

## Sudo Access

- `openclaw` user has passwordless sudo for at least apt, apt-get, snap, dpkg (added 2026-03-26 via `/etc/sudoers.d/openclaw`) — but the list is incomplete.
- **Before running an unknown command under sudo, run `sudo -l` first** — it shows exactly what's allowed. Don't guess.

## Network

- **Tailnet:** `tail1916d.ts.net`
- **This server (oc):** `oc.tail1916d.ts.net` (100.73.183.75)
- **VNC:** `vnc://oc.tail1916d.ts.net:5999`, password `openclaw` (Xvnc on DISPLAY :99)

## Shared Calendar (Dj & Sherra)

- Config and calendar IDs in `/home/openclaw/.openclaw-shared/gcal-config.json`
- Re-auth: deploy `/tmp/gcal-auth/` to k8s as `gcal-auth.oc.ctb.padz.net`, visit URL, grab token from `/token` endpoint
- Google Places API key referenced via 1Password SecretRef in `openclaw.json` (exec provider → `onepassword`)

## Temporary Services / microk8s

- For any temporary HTTP/HTTPS service (OAuth callbacks, webhooks, test endpoints), deploy to microk8s and use any subdomain under `oc.ctb.padz.net` — wildcard DNS points to this server (85.239.236.177)
- Use `cert-manager.io/cluster-issuer: letsencrypt-prod` annotation for automatic TLS (same pattern as grocery app)
- Tear down with `microk8s kubectl delete namespace <name>` when done
- Example: `smartcar.oc.ctb.padz.net`, `callback.oc.ctb.padz.net`, etc.
