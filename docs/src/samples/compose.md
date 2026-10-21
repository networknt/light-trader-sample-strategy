# Stack and deployment design

Use separate offline, browser-integration and paper profiles so the teaching workflow can run
without a broker. The [overview](overview.md) tracks delivery status; S5 will turn this design
into a tested installation runbook.

## Profiles and services

| Profile | Services | Broker boundary |
|---|---|---|
| Offline (default) | Task-owned TimescaleDB, optional Iggy, replay runner and synthetic fixtures | No IB credentials, no broker service or order network access |
| Browser integration (optional) | Light Gateway, chart assets, accepted history/research APIs; retained Python endpoints where needed | Read-only/synthetic until accepted; no bypass around gateway security |
| Paper (explicit opt-in) | Accepted feed-server/trade-server, hashed samples, private storage and operator-provisioned paper IB Gateway | Requires independent qualification, reviewed configuration and supervised activation |

Use one Compose project/network and named volumes dedicated to the sample stack; do not reuse
normal recorders, private databases, broker client IDs or occupied ports. Pin image digests and
module hashes. Run version-matched migrations through the upstream supported migration command;
do not invent tables or assume trade-server performs every service's migration.

The paper template must default strategies to disabled and broker capabilities to unsupported.
No live profile or live-account credentials ship. Account aliases and placeholders belong in
committed examples; secrets belong in ignored, permission-restricted runtime files. Interactive
broker login/2FA is an operator step, never an unattended CI prerequisite.

## Browser security

Light-Fabric `apps/light-gateway` is the planned sole browser HTTPS/WSS entry point. Keep
feed/trade/database/broker listeners private. Reuse gateway login/session, origin/CSRF and routing
contracts; trade-server verifies upstream identity and resource authorization. Do not create a
second example web password or expose the feed's internal service credential to JavaScript.

Follow Light-Trader's chart gateway G0–G6 plan. The gateway routes history, Python indicators,
research jobs and trading APIs independently. Browser chart migration is not a prerequisite for
an offline sample replay, and WSS qualification is not implied by HTTP connectivity.

## Packaging acceptance

S5 must deliver concrete environment templates, pinned image/module manifests, actual configuration
files, health/readiness checks, build/start/stop/log commands and a tested clean-machine runbook.
Validate paths, writable volumes, secret redaction and dependency startup failures. Test offline
replay first; paper activation remains a separate command/profile and explicit approval.

Stopping the stack leaves volumes and broker-held protection intact; it does not mean orders
were cancelled or positions closed. Upgrades require accepted artifact/config compatibility,
state migration/reconciliation and a rollback plan. Do not use an unrestricted pull-and-restart
sequence as a strategy promotion procedure. A WIT version match alone does not guarantee host
compatibility, state compatibility or broker behavior.

See [implementation milestones](implementation.md) and [testing](testing.md) for the acceptance gates.
