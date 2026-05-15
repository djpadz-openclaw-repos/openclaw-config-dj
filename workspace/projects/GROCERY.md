# GROCERY.md — Grocery App

Built 2026-03-22, deployed to k8s.

## URLs & Access

- **URL:** https://grocery.oc.ctb.padz.net — k8s namespace `grocery`
- **Auth token:** `~/.openclaw-dj/projects/.grocery-token` (30-day JWT, refresh with `node grocery-app/refresh-token.js`)

## User & Configuration

- **Primary user:** `djpadz` (user ID 2), linked to Apple ID
- **Default list ID:** 2 ("Groceries")
- **Stores:** 1=Pavilions Hillcrest, 2=Ralph's Mission Center

## Telegram Integration

- **Handle grocery-related Telegram messages directly** — parse intent, call API, respond in plain English
- Full API reference: `grocery-app/TELEGRAM_INTEGRATION.md`

## Deployment

Rebuild+deploy:
```bash
cd client && npm run build && cd .. && rsync -a --exclude='client/node_modules' --exclude='*.db' --exclude='.git' grocery-app/ ~/grocery-build/ && docker build -t localhost:32000/grocery:latest ~/grocery-build/ && docker push localhost:32000/grocery:latest && /snap/bin/microk8s.kubectl rollout restart deployment/grocery -n grocery
```

## Sign In with Apple

- Nonce MUST be passed to Apple JS SDK, but do NOT pass raw nonce to `verifyIdToken` (double-hash bug).
