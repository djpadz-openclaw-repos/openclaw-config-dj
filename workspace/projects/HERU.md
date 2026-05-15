# HERU.md — Heru Portal & Infrastructure

## Product Context

Heru Portal is a **medical testing platform for eye examinations** — handles DICOM imaging, patient data, appointment scheduling, multi-tenant (account-based) architecture.

- Dj developed a **42-table MySQL schema** for the portal; has deep knowledge of this schema
- "Account 89" = Dev Hospital (test account with 809 patients) — used for development/testing

## Technician Workstations

- Run **React Native Windows (UWP)** apps
- Known issue: crashes over Remote Desktop (DirectX/Direct2D rendering)

## Kiosk Devices

- **ARM64 Windows 11 tablets** (Surface Pro X and others)
- Standard Assigned Access / Shell Launcher don't work on ARM64 — custom PowerShell-based kiosk setup via Intune required
- Chrome kiosk URL: `https://www.seeheru.com` (production) and `https://portal.dev.devqa.env.heru.net` (dev)

## EHR Integration

- App: `EHRIntegrationAppNative.sln` — React Native Windows Win32 (not UWP)
- Build: `npx react-native run-windows --release`

## Azure Infrastructure

- Uses Azure extensively: AKS, Azure Service Bus, Azure Static Web Apps + Function Apps, Azure Front Door, Azure DNS, MySQL Flexible Server, blob storage
- Azure DNS zone: `dev.devqa.env.heru.net` in resource group `dns` — DNSSEC managed by Azure (not exportable)
- Hub-and-spoke VNet topology; MySQL Flexible Server uses delegated subnet + private DNS zones
- Datadog used for monitoring; exclusion filters for CosmosDB wrapper upload spam and debug logs; anomaly detection via metric monitors (log monitors don't support anomaly detection directly)
- Datadog Sentinel Playbook Operator role assigned via Azure Service Groups (preview feature — no RBAC inheritance, must assign per-resource)

## Intune / Device Management

- Manages devices with **Microsoft Intune + Autopilot**; deploys PowerShell platform scripts for kiosk config, Chrome policy, Arc onboarding
- Chrome policy via Intune: block-all (`*`) + allowlist `http://*.heru.net/*` and `https://*.heru.net/*` pattern
- ARM64 kiosk limitation: Assigned Access and Shell Launcher have gaps on ARM64; workaround is PowerShell script creating scheduled task at login
- Has used `slmgr` for Windows license management (deactivation/reuse of product keys)
- YubiKey with multiple accounts registered — Windows 11 login selects account by user selection first, then key

## Video Processing Pipeline (2026-03-20)

Working on video processing pipeline (Azure Service Bus queues).

- **Video queue** (`dev-video-service`) message schema: `{ appt_id, test_id, results_container, raw_container, results_base_filename }`
- **Report queue** (`dev-report`) message schema: `{ blob_filename: "{appt_id}/{test_id}/{results_base_filename}.json" }`
- **Flow:** upstream → video service → report server → dev-results blob storage
- **Report server:** on AKS, Datadog logging still being wired up (sparse logs)
- **Datadog query for report server:** `source:vfreportserver kube_namespace:dev kube_cluster_name:heru-eastus-dev-akscluster subscription_id:1900826d-6330-431f-b525-e4c4eef4b893`
- **Result types:** both_eom, both_pupil, both_cover, both_pretest-pro
- **Note:** `blob_filename` is correct key (not `filename` — that was a past typo in dead letters)
