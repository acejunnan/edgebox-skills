# Evidence and acceptance

Use this to define completion before applying changes. Evidence is private and
contains no reusable credentials. Mark every item as passed, failed, or not tested,
and identify whether it ran locally, in a simulation/namespace, or on the real host.
Do not count code presence or an old deployment's report as this deployment's evidence.

| Area | Required observation | Insufficient evidence |
| --- | --- | --- |
| Host identity | Authenticated/pinned host keys remain the expected keys on new connections | Accepting the first network key without identity verification |
| Ordinary WAN | Physical default remains appropriate; local DNS and HTTPS work before and after data changes | One old open SSH session |
| Bastion restrictions | Effective operator policy preserved; tunnel public-key only, loopback listener, exact allowed reverse port; forbidden shell/local forwarding/password methods rejected | Merely seeing a Match block in a file |
| Management | A newly authenticated edge SSH session through the bastion; a new VPS operator login | `systemctl is-active` alone |
| Transport | Recent authenticated handshake/endpoint health with expected MTU and keys | Config file or interface existence |
| Real user egress | Request through the actual user-facing client entry reaches an external service; resulting IPv4 matches the edge's independently observed residential WAN exit | WireGuard handshake, VPS's own direct request, `/ready`, or matching a hard-coded old IP |
| DNS and UDP | Client-proxied domain resolution and a bounded valid request for required UDP behavior | Successful TCP website only |
| Fail closed | With data endpoint unavailable, proxied requests fail without switching to the VPS's ordinary direct path | An upstream preference label |
| Management survival | While the data service is stopped/failed, new reverse management login and ordinary WAN/DNS/HTTPS still work | Only restoring them after the test |
| Ownership/idempotency | Reapply is idempotent or explicitly refuses a collision; cleanup leaves unrelated routes/firewall/services intact | Repeated commands that silently delete unknown state |
| Rollback | Deliberately rejected candidate/failed verification restores owned known-good state; watchdog handles installer interruption | A rollback script that was never executed |
| SAFE_MODE/recovery | Owned data state removed, identity and management preserved; reactivation under a new protected transaction succeeds | A status string set to SAFE_MODE |
| Persistence | Observed service/transaction reconciliation after a safe, authorized reboot; uncommitted state not activated | A symlink under multi-user.target |
| WAN change/power loss | Planned actual cable/DHCP/power tests only when physical recovery or a safe equivalent is available | Assuming a namespace test proves field recovery |

Use a low-volume external observation, not a bulk speed test. For CGNAT/multiple
public egresses, explain whether a stable matching IP can be established; do not
force a success by ignoring a mismatch. The IPv4 must come from a parsed observation,
not an address copied from the plan.

## Bounded failure tests

Define each failure action and expected recovery before running it. Stopping only
the owned data-plane service is usually a useful early survival test. Do not stop
physical WAN, sshd, management services, or all firewall state as a shortcut.
Transport or proxy failure must not become a global Internet outage on the edge.

Test an invalid configuration through validation and a failed candidate through
the actual rollback mechanism. If the installer/controller is intentionally stopped,
prove its independent watchdog remains armed and performs cleanup/restoration.
An observed cleanup failure is a failed test even if management happened to remain reachable.

Reboot, power cuts, cable removal and DHCP disruption require the appropriate existing
authorization and a recovery path. If unavailable, list them as pending rather than
manufacturing an outage on an unattended appliance. Core egress, rollback and fresh
management checks remain required before claiming the live deployment is complete.

## Suggested handoff status table

Report system/WAN/management/control/transport/network/Internet-egress independently.
For a missing control backend use “not implemented/not applicable,” not “healthy.”
State actual configuration version, current transport, relay, MTU, bandwidth ceiling,
SAFE_MODE, rollback deadline/status and known-good locations.

Include concise reproducible non-secret evidence and timestamps, any remaining test
gates, and next recovery steps. Keep client links, SSH private material, tokens and
precise deployment inventory out of public reports. A user-authorized public summary
should describe behavior and test scope, with private operational details removed.
