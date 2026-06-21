# Enphase Capability Audit for AEAC V1

Task: `aeac-01-enphase-upstream-audit`  
Reviewed: 2026-06-21, America/New_York  
Audit stance: no live Enphase credentials were used, no live hardware calls were made, and no write endpoint was exercised.

## Decision

Dispatcher implementation is blocked from making live Enphase write calls in V1 unless a later task adds an explicit human-approved capability file proving account type, endpoint access control, site/gateway enrollment, reversibility, idempotency strategy, and dry-run/live separation.

For V1, AEAC may implement telemetry reads and simulator/no-op dispatch only. The only cloud write intent that appears plausibly documentable for a non-VPP product is a high-level battery settings update through `PUT /api/v4/systems/config/{system_id}/battery_settings`, but it still requires a Megawatt or Partner plan access control, owner authorization, system with Encharge/IQ Battery, strict input validation, and an explicit read-before-write/read-after-write guard. It is not safe to implement as an adaptive loop yet.

## Official Sources Reviewed

All sources below are official Enphase sources and were reviewed on 2026-06-21.

| Source | URL | Relevant dates in source | Notes |
| --- | --- | --- | --- |
| Enphase API homepage | `https://developer-v4.enphase.com/` | Page copyright 2026 | Defines Monitoring API, Commissioning API, and VPP API categories. VPP is described as fleet DER monitor/forecast/control across PV, Battery, EVSE, heat pump, HVAC. |
| Quick Start | `https://developer-v4.enphase.com/docs/quickstart.html` | Current page reviewed 2026-06-21 | Documents OAuth 2.0, developer/homeowner authorization flow, partner password-grant style flow, API key plus bearer token requirement, and plan/access-control selection. |
| Monitoring API documentation | `https://developer-v4.enphase.com/docs/monitoring_api` | Current page reviewed 2026-06-21 | Rendered docs point to the official Swagger spec at `/swagger/spec/System_API.json`; each request uses `Authorization: Bearer ...` plus API key header `key`. |
| Monitoring/System Swagger spec | `https://developer-v4.enphase.com/swagger/spec/System_API.json` | Current JSON fetched 2026-06-21 | Exact source for system telemetry, system configuration, battery settings, grid status, storm guard, load control, EV charger monitoring, latest telemetry, and failure responses. |
| Commissioning API documentation | `https://developer-v4.enphase.com/docs/commissioning_api` | Current page reviewed 2026-06-21 | Rendered docs point to the official Partner Swagger spec. |
| Partner/Commissioning Swagger spec | `https://developer-v4.enphase.com/swagger/spec/Partner_API.json` | Current JSON fetched 2026-06-21 | Exact source for activation create/update/delete, activation battery mode CFG/DTG, production mode, meter enable/disable, account/system authorization failures, and rate-limit failure examples. |
| VPP API documentation | `https://developer-v4.enphase.com/docs/vpp_api` | Current page reviewed 2026-06-21 | Rendered docs point to the official VPP spec at `https://vpp.enphaseenergy.com/auth/getApiSpec`; each request uses `Authorization: Bearer ...` plus `x-api-key`. |
| VPP Swagger/OpenAPI spec | `https://vpp.enphaseenergy.com/auth/getApiSpec` | Current JSON fetched 2026-06-21 | Exact source for VPP events, battery event modes, OCPP EVSE configuration, local gateway token generation, program/application enrollment, and event cancellation/update/opt-out semantics. |
| Developer pricing/plans | `https://developer-v4.enphase.com/developer-plans` | Current page reviewed 2026-06-21 | Watt, Kilowatt, Megawatt rate limits and access controls. Megawatt adds System Configurations; developer plans require system owner approval. |
| Installer pricing/plans | `https://developer-v4.enphase.com/installer-plans` | Current page reviewed 2026-06-21 | Partner plan access controls, installer eligibility, Partner rate limits, EV Charger Control, System Configurations, and Commissioning API access. |
| Release Notes | `https://developer-v4.enphase.com/docs/release_notes` | 2022-02-23 through 2025-10-15 | Source for added endpoints and behavior changes, including 2025 EVSE telemetry, 2025 activation battery mode endpoints, 2024 latest telemetry, 2023 live status, 2023/2022 system configuration endpoints, and partner write throttling. |
| 2025 API enhancements news | `https://developer-v4.enphase.com/news/enphase-api-enhancements-2025` | 2025 enhancements page, reviewed 2026-06-21 | Summarizes 2025 EVSE monitoring and battery mode APIs; confirms "greater control" language but not sufficient alone for V1 implementation. |
| API license page linked from official docs | `https://enphase.com/api-license-agreement-v4` | Linked by Quick Start and homepage | Not used for endpoint semantics; downstream implementation must still verify licensing before production-like use. |

## Local Inputs Reviewed

| Local input | Result |
| --- | --- |
| `energy/README.md` | Energy module currently states read-only observation: "Monitors solar production and battery state-of-charge (SoC) from Enphase systems to optimize household electricity usage." |
| `energy/src/events.py` | Missing in this worktree at the requested path. No local event contract exists yet for AEAC write intents. |
| `docs/superpowers/specs/2026-06-21-aeac-design.md` | Missing in this worktree at the requested path. The specdrift block points instead to `/Users/braydon/projects/experiments/house/docs/superpowers/specs/2026-06-21-implement-the-adaptive-enphase-autonomous-controller-for-the-hou-spec.md`, but this file is not present in the worktree file listing. |
| `.workgraph/planforge/20260621-175434/evidence.json` | Reviewed. It shows the repo is early bootstrap, the broader workgraph created untracked module files outside this task, and prior drift was already yellow for scope/spec drift. |
| Environment variable names | `env | cut -d= -f1 | rg -i 'ENPHASE|ENLIGHTEN|ENVOY|IQ_'` found no Enphase-related local authorization variables. |
| Repo search | No Enphase implementation code or credentials were found, excluding `.env*` files. |

## Account, Auth, and Gateway Requirements

Public Monitoring and System Configuration API:

- Authentication: OAuth 2.0 bearer access token plus application API key header named `key`.
- Developer plans require system-owner authorization before data access.
- Watt plan: 10 hits/minute, 1,000 hits/month; access controls include System Details, Site level Production Monitoring, Site level Consumption Monitoring, EV Charger Monitoring.
- Kilowatt plan: 50 hits/minute, 50,000 hits/month; adds Device level Monitoring and Streaming API.
- Megawatt plan: 100 hits/minute, 300,000 hits/month; adds System Configurations and EV Charger Monitoring. This is the first developer plan that appears to include `/api/v4/systems/config/{system_id}/battery_settings`.
- Common failures: 401 unauthorized, 403 forbidden, 404 not found, 405 method not allowed, 422 unprocessable entity, 429 too many requests, 501 not implemented.

Partner/Commissioning API:

- Authentication: OAuth 2.0 access token plus API key. Quick Start documents Partner token generation using Enphase cloud credentials and the application client credentials.
- Eligibility: Partner plan is available only to registered Enphase installers with at least 10 installations.
- Scope: Partner plan provides all endpoints for installed and maintained systems. Release notes also say authorized subcontractors can access all portal endpoints under Partner if subscribed and authorized.
- Rate limits: 300 hits/minute, 1,500,000 hits/month, plus several 10,000 hits/month buckets. Release notes dated 2022-07-28 add a separate throttle of 200 hits/month for each PUT, POST, and DELETE endpoint under Partner plan.
- Partner writes can be asynchronous task submissions to Envoy/gateway, not immediate state guarantees.

VPP API:

- Authentication: OAuth 2.0 bearer token plus API key header named `x-api-key`.
- Account model: Program/application/enrollment-oriented. Error examples repeatedly require a site to be enrolled in a program belonging to the account and reject invalid program IDs or account/site mismatches.
- Gateway: VPP spec can generate an IQ Gateway local API token for an enrolled site/gateway serial number. The token is documented as valid for 12 hours and generatable any number of times once the site is enrolled.
- VPP is not a homeowner/local-control shortcut. It is a fleet/program API and should be treated as out of scope unless AEAC is explicitly operating under an approved VPP/utility/aggregator program.

Local authorization state:

- This worktree has no declared Enphase API key/token/env var names and no enrolled program metadata.
- Therefore any validation requiring real auth, gateway token generation, or hardware writes is production-like and must be escalated to a human before use.

## Telemetry Surface

Telemetry is read-only and acceptable for AEAC planning if rate-limited and cached. It must not be confused with control.

| Capability | Official endpoints | Access control | Notes and failure modes |
| --- | --- | --- | --- |
| System summary | `GET /api/v4/systems/{system_id}/summary` | System Details / site monitoring depending on selected plan | Release notes add battery charge/discharge/capacity and EVSE min/max charge/discharge power. Read-only. 401/403/404/429 possible. |
| Latest telemetry | `GET /api/v4/systems/{system_id}/latest_telemetry` | Site Level Consumption Monitoring | Returns last reported PV, consumption, and battery power; release notes and spec add HP/EVSE operational mode. Values become null when a device's last report is older than 7 days. |
| Production telemetry | `GET /api/v4/systems/{system_id}/telemetry/production_meter`, `GET /api/v4/systems/{system_id}/telemetry/production_micro` | Production/device monitoring by plan | Date range restrictions apply; release notes list two-year start limit and max one-week duration for telemetry endpoints. |
| Consumption/import/export telemetry | `GET /api/v4/systems/{system_id}/telemetry/consumption_meter`, `GET /api/v4/systems/{system_id}/energy_import_telemetry`, `GET /api/v4/systems/{system_id}/energy_export_telemetry` | Site Level Consumption Monitoring | Read-only; empty list can be returned for intervals after the last reported interval. |
| Battery telemetry | `GET /api/v4/systems/{system_id}/telemetry/battery`, `GET /api/v4/systems/{system_id}/devices/encharges/{serial_no}/telemetry`, `GET /api/v4/systems/{system_id}/devices/acbs/{serial_no}/telemetry`, `GET /api/v4/systems/{system_id}/battery_lifetime` | Consumption/device monitoring depending on endpoint | Includes battery power/SOC history and last-reported SOC fields. No direct dispatch semantics. |
| Grid, storm, and load status | `GET /api/v4/systems/config/{system_id}/grid_status`, `GET /api/v4/systems/config/{system_id}/storm_guard`, `GET /api/v4/systems/config/{system_id}/load_control` | System Configurations, Megawatt or Partner | Read-only in public System API. Useful as preconditions and safety state, not dispatch. |
| EVSE monitoring | `GET /api/v4/systems/{system_id}/ev_charger/devices`, `GET /api/v4/systems/{system_id}/ev_charger/{serial_no}/telemetry`, `GET /api/v4/systems/{system_id}/{serial_no}/evse_lifetime`, `GET /api/v4/systems/{system_id}/{serial_no}/evse_telemetry` | EV Charger Monitoring; Watt+ includes EV Charger Monitoring, device endpoints may require higher plan depending docs grouping | Read-only. EVSE operational modes include `PLUGGED_OUT`, `IDLE`, `CHARGING`, `FAULTED`; telemetry `wh_consumed` may be positive for EV charge and negative for EV discharge. |
| Live status | Release notes add live status for up to 300 seconds, IQ Gateway version >= 6.0.0 | Streaming/API plan and per-hit pricing | Read-only stream. Default 30 seconds, min 30, max 300. Per-call pricing means fast adaptive loops can create cost and quota risk. |

## Configuration Intent Candidates

These names are high-level AEAC intents, not raw endpoint wrappers. Any live implementation must validate preconditions and use read-before-write/read-after-write.

| Intent name | Endpoint basis | Preconditions | Reversibility | Idempotency approach | V1 status |
| --- | --- | --- | --- | --- | --- |
| `battery_profile_set` | `PUT /api/v4/systems/config/{system_id}/battery_settings` | Megawatt or Partner plan with System Configurations; OAuth bearer token; API key; system owner authorization or installer/maintainer/subcontractor access; system has Encharge/IQ Battery; current settings fetched successfully; desired `battery_mode` is exactly one of documented accepted values; `reserve_soc` is not below VLS; `energy_independence` is `enabled` or `disabled`. | Reversible only by saving previous `battery_mode`, `reserve_soc`, and `energy_independence` from GET and later restoring them. Not reversible if external app/user/grid-services event changes settings meanwhile. | Compare desired settings to GET response; skip if already equal; include an idempotency key in local command ledger only, because the Enphase endpoint does not document an idempotency key. Confirm with GET after 200. | Simulator/no-op only until explicit auth and approval are provided. |
| `battery_reserve_soc_set` | Same `PUT /api/v4/systems/config/{system_id}/battery_settings` | Same as above; bounded SOC; must respect VLS and battery shutdown level; must not be used to defeat backup reserve or storm guard. | Reversible by restoring prior `reserve_soc`, subject to concurrent changes and accepted range. | Same read/compare/write/confirm flow. | Simulator/no-op only. |
| `battery_energy_independence_set` | Same `PUT /api/v4/systems/config/{system_id}/battery_settings` | Same as above; only meaningful for Savings/Self-consumption behavior described by Enphase; tariff interactions should be verified first because release notes mention a prior tariff-related failure when updating Savings Mode. | Reversible by restoring prior value, subject to tariff/profile side effects. | Same read/compare/write/confirm flow. | Simulator/no-op only. |
| `battery_charge_discharge_allowed_set` | Partner `PUT /api/v4/activations/{activation_id}/battery_mode` with `CFG_allowed` and `DTG_allowed` | Partner plan; registered installer/maintainer/subcontractor access to installed/maintained activation; battery site, not PV-only; activation ID known; policy/legal review of Charge From Grid and Discharge To Grid; acceptance of Partner write throttles. | Reversible by saving prior booleans from GET and restoring. | GET booleans before PUT; no-op if equal; verify by GET. | Out of scope for homeowner AEAC V1; simulator/no-op only unless the human explicitly supplies installer authorization and policy approval. |
| `battery_vpp_event_schedule` | VPP `POST /api/v1/events` for `function: BATTERY` with modes `Charge_From_PV`, `Charge_From_NetPV`, `Charge_From_PV_Grid`, `Discharge_To_Load`, `Discharge_To_NetLoad`, `Discharge_To_Load_Grid`, or `Idle` | VPP account; approved program; site enrolled in program; event start 1 minute to 7 days in future; duration 5 minutes to 5 hours; mode supported by program; target SOC/rate within program bounds; site/gateway info available; site count below recommended 500. | Reversible only by canceling/ending the event if program allows. Completed events cannot be canceled; cancellation can be disallowed by feature flags. | Use local command ledger and avoid overlapping events; `superseded` only where intended; query event status and command delivery status after creation. | Out of scope for V1 live dispatch; simulator-only. |
| `battery_vpp_event_update` | VPP `PUT /api/v1/events/{event_id}` | Existing scheduled event; update at least 1 minute before start; only `rate_watt` and `target_soc` can be updated. | Partially reversible by another update before lockout. | Track event ID locally; GET before and after update. | Simulator-only. |
| `battery_vpp_event_cancel` | VPP `DELETE /api/v1/events/{event_id}` | Scheduled or in-progress event; program allows upcoming cancellation or ongoing end; event not completed/already canceled. | Not reversible as original event; can only create a new event. | Treat repeated cancel failures such as "already submitted/in_progress/completed" as terminal status, not retry loop. | Simulator-only. |
| `evse_ocpp_register` | VPP `POST /api/v1/systems/{site_id}/devices/evse/{serial_no}/ocpp_configuration` | VPP account; site enrolled; program configured for EVSE Control type OCPP; partner CSMS URL, security profile, and one-time auth key available; human approval for handing charger control to CSMS. | Reversible only through deactivation/reconfiguration flow implied by status schema, not clearly documented as a direct undo endpoint in reviewed docs. | Query `/ocpp_configuration/status`; avoid re-post if ACTIVE and matches desired CSMS; otherwise escalate. | Not an AEAC V1 control intent; simulator/no-op only. |
| `evse_ocpp_reset` | VPP `POST /api/v1/systems/{site_id}/devices/evse/{serial_no}/ocpp_reset` | Same VPP/OCPP preconditions; reset semantics are communication reset, not start/stop charging. | Not a reversible configuration; it is an operational reset. | Use sparingly with cooldown and status polling; never in fast adaptive loop. | Unsupported for V1. |
| `gateway_local_token_generate` | VPP `POST /api/v1/systems/{site_id}/devices/gateway/{serial_no}/local_api_token` | VPP enrolled site and gateway; bearer token plus x-api-key; token valid 12 hours. | Token can expire; generation is repeatable. | Cache only in secret storage; never commit/log token; rotate on expiry. | Not sufficient to implement local writes; audit did not find official local switching endpoints. |

## Unsupported Raw Switching Claims

These must be represented as `unsupported` or `simulator_only` in downstream contracts. They must not result in live Enphase network writes in V1.

| Claim | Reason |
| --- | --- |
| Raw solar on/off as an adaptive optimizer action | Partner API has `POST /api/v4/activations/{activation_id}/ops/production_mode` with `mode: on/off`, but this is a commissioning/installer operation that sends a task to Envoy, requires Partner access, and can materially affect generation. It is not an optimizer-safe homeowner control. |
| Raw microinverter or PV curtailment setpoint | No public System API endpoint reviewed provides a homeowner-safe PV curtailment setpoint. Production mode is too coarse and partner-gated. |
| Raw battery charge/discharge power from ordinary developer plan | Public System API battery settings update changes profile/reserve/energy-independence only. VPP events can set `rate_watt`, but only inside approved VPP programs with enrollment and event constraints. |
| Hold battery, lock discharge, or arbitrary battery sleep | No reviewed official public endpoint directly exposes a "hold" or "sleep" primitive. It may be approximated by documented profiles/reserve or VPP `Idle` events only under their constraints, but not as raw switching. |
| EVSE start/stop/amps through public developer API | Official System API has EV Charger Monitoring only. VPP EVSE Control is OCPP onboarding/reset/status for partner CSMS, not a homeowner direct charger start/stop endpoint in the reviewed docs. |
| Gateway-local writes | VPP can generate a local API token, but the reviewed official docs do not define local write endpoints for battery, solar, or EVSE switching. |
| Fast closed-loop live writes | Rate limits, monthly partner write throttles, per-hit live status pricing, asynchronous task semantics, and non-idempotent event endpoints make fast adaptive write loops unsafe without a separate control-plane design. |

## Failure Modes and Safety Notes

- 401/403: token, API key, plan, account, site, program, or owner/installer authorization mismatch. Downstream code must treat these as hard capability failures, not retryable dispatch failures.
- 404: system, activation, event, meter, EVSE, gateway, or program not found. Treat as configuration error.
- 405/501: method unavailable or not implemented. Treat as unsupported capability.
- 422: valid-looking request rejected by system state, such as no Encharge, PV-only site, grid status unavailable, stage too low, no active Envoys, event state not eligible, or values out of allowed bounds. Treat as unsafe/no-op and surface explicitly.
- 429: quota/rate limit exceeded. Adaptive loops must back off and degrade to telemetry-only.
- Asynchronous writes: production mode and meter control examples say tasks are sent and require waiting. A 200 does not prove the physical state changed.
- Concurrent writers: Enphase App, installer tools, grid services, VPP events, tariffs, storm guard, and utility programs may override or conflict with AEAC settings.
- Storm guard/grid outage: if storm guard is active or grid status is off-grid/unknown, AEAC should not reduce backup posture or send battery optimization writes.
- Tariff side effects: release notes mention a resolved battery_settings/Savings Mode tariff failure; any live implementation must snapshot tariff/profile context and use low-frequency human-approved changes only.
- Privacy/security: access tokens, refresh tokens, API keys, local gateway tokens, site IDs, serials, and homeowner tokens must be treated as secrets or sensitive operational identifiers.

## Dispatcher Gate for Downstream Tasks

Downstream dispatcher tests and implementation must enforce this matrix:

| Intent category | V1 live network status | Required behavior |
| --- | --- | --- |
| Telemetry polling | Allowed if credentials are explicitly configured and only GET endpoints are used | Rate-limit, cache, redact identifiers in logs, and degrade to stale/unknown state on auth or quota failure. |
| Public battery profile/reserve settings | Blocked by default | Expose as simulator/no-op unless an approved capability file enables `battery_profile_set`, `battery_reserve_soc_set`, and/or `battery_energy_independence_set` with all preconditions. |
| Partner activation battery CFG/DTG | Blocked | Simulator/no-op only without explicit installer/Partner authorization and human policy approval. |
| Partner production mode and meter control | Blocked | Unsupported for AEAC optimizer; do not map to energy-saving intents. |
| VPP battery events | Blocked | Simulator-only unless AEAC is explicitly running as an enrolled VPP/utility program participant. |
| VPP EVSE OCPP configuration/reset | Blocked | Unsupported for household optimizer V1; do not implement direct charger control from these endpoints. |
| Gateway-local control | Blocked | No official local write endpoint was audited; local token generation alone is insufficient. |

Minimum contract for any future live write enablement:

1. A human-approved capability file names the exact account plan, access controls, system/activation scope, enrolled program if applicable, and allowed endpoint list.
2. Every live intent has `read_current`, `validate_preconditions`, `compare_desired`, `write_once`, `read_after`, `rollback_plan`, and `audit_log_redaction`.
3. Every write has a local idempotency record and a cooldown; no high-frequency adaptive loop may call write endpoints.
4. Every write path has simulator parity and must be disabled by default.
5. Every production credential or token use requires explicit escalation and must never be exercised by an agent without human approval.

## Verification

- Official docs were accessed without account credentials.
- No live write or credential use occurred.
- Local authorization constraints were checked by environment-variable name search only; no secret files were opened.
- Required output path exists and is non-empty after this audit.
