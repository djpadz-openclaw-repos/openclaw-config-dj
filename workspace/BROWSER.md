# BROWSER.md — Browser Automation & CDP

## Browser Setup

**OpenClaw browser:** ungoogled-chromium running on Xvnc display `:99`, exposed via CDP on port `18800`

**Headless Playwright in containers/pods:** Use `mcr.microsoft.com/playwright:v1.59.1-noble` — the official Playwright Docker image with all Chromium system dependencies pre-installed. The OpenClaw pod is missing `libnspr4`, `libnss3`, `libatk`, `libgbm`, `libxkbcommon`, etc. and they can't be installed via brew (no sudo). Don't waste time trying to install them — just use the Docker image.

**Networking gotcha:** Docker volumes don't mount properly in this environment (nested containers). Bake test files into the image with a `Dockerfile.e2e`. Use `--network host` and point baseURL at the K8s ClusterIP directly (e.g. `http://10.152.183.195:3000`) — `localhost` port-forwards aren't reachable from inside Docker containers.

**Quick recipe for e2e tests:**
```dockerfile
FROM mcr.microsoft.com/playwright:v1.59.1-noble
WORKDIR /work
RUN npm init -y && npm install --ignore-scripts @playwright/test
COPY playwright.config.ts .
COPY tests/e2e/ tests/e2e/
RUN mkdir -p tests/e2e/screenshots tests/e2e/results
CMD ["npx", "playwright", "test", "--reporter=list"]
```
```bash
docker build -t my-e2e -f Dockerfile.e2e .
docker run --rm --network host my-e2e
# Extract screenshots:
docker run --name e2e-run --network host my-e2e
docker cp e2e-run:/work/tests/e2e/screenshots ./screenshots
docker rm e2e-run
```

- **Never use headless Playwright directly** for sites like OpenTable — bot detection (Akamai/PerimeterX) blocks it with timeouts or HTTP2 errors
- **Correct approach:** `playwright connect_over_cdp("http://127.0.0.1:18800")` — attaches to the real browser with existing sessions/cookies/fingerprint
- **Start if not running:** `DISPLAY=:99 openclaw browser start`
- **Check if running:** `openclaw browser status` or `curl -s http://127.0.0.1:18800/json/version`

## Browser Lifecycle & Semaphore

**CRITICAL:** Any code that uses the CDP browser MUST use the semaphore to prevent orphaned processes.

**Async code pattern:**
```python
from browser_semaphore import browser_context
with browser_context():
    async with async_playwright() as p:
        browser = await p.chromium.connect_over_cdp("http://127.0.0.1:18800")
```

**Sync code pattern:**
```python
from browser_semaphore import browser_acquire, browser_release
browser_acquire()
try:
    # ... work ...
finally:
    browser_release()
```

**Semaphore location:** `/home/openclaw/.openclaw-shared/browser_semaphore.py`
- `browser_acquire()` starts if needed
- `browser_release()` stops when count hits zero
- Use `browser_context()` as a context manager

**Xvnc (:99) should stay running** — it's a persistent display server, never kill it. Only the browser (ungoogled-chromium) should be started/stopped on demand.

## OpenTable Integration

- Session is already logged in as Dj (account `GoldDP`, `dj@heru.net`) — use `skills/opentable/scripts/opentable_book.py`
- **Login detection:** check for `"firstName"` + `"gpid"` in page source — OpenTable embeds profile JSON in every page
- **If session expired:** auth is fully automated — enter `djpadz@padz.net`, poll `padz.net` IMAP (`mail.padz.net:993`, SSL) for the 6-digit code from `opentable.com`. No manual intervention needed.
- **Small/resort-town restaurants** (Big Bear, etc.) often appear on OpenTable but disable online booking. "Party too large" error even for 2 = call directly. See `skills/opentable/references/known-restaurants.md`
- **Always use direct restaurant URL** with `?covers=N&dateTime=YYYY-MM-DDThh:mm:ss` — never search from homepage (defaults metro to Seattle)
- This applies to any site with aggressive bot detection: always go through the CDP-connected real browser
