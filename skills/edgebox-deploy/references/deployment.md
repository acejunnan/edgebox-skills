# Deployment and recovery procedure

Use for authorized live deployment or recovery. Adapt commands to the actual OS;
this reference deliberately contains no turnkey root script or assumed addresses.

## A. Establish scope and inspect read-only

Identify the exact VPS, edge device, workspace and allowed operations. If the current
request only asks for preparation, stop after inspection and a reviewable plan.
If those targets and changes are already authorized, do not ask redundant permission.
Do not infer purchase, public publishing, firmware or unrelated-service removal authority.

Before trusting a target, verify its SSH host identity through existing known_hosts,
cloud/local console or another authenticated channel. A network keyscan alone is not verification.
Do not send a login password to an unverified host.

Inspect:

- actual distribution, architecture, init system and kernel support;
- login privilege and existing sudo capability; do not silently grant broad sudo;
- interfaces/addresses, main default routes, DNS, other tables and rules;
- full firewall state read-only, forwarding state and data-path conflicts;
- SSH configuration, effective account restrictions, management service and fresh access;
- port listeners, cloud firewall information available to the user, existing applications;
- storage, memory, thermal/power symptoms and existing device identity/credentials;
- low-volume DNS and HTTPS reachability from each host.

Read existing config only as needed; redact secret values from observations and logs.
When an existing installation is found, inspect and choose an explicit migration or
repair path. Never treat it as an empty host, overwrite identities, or delete state to bypass checks.
If physical WAN is not working, report that unmet prerequisite instead of adding hotel
authentication or rebuilding the user's router configuration.

## B. Produce a concrete plan and implementation

Before remote mutation, describe target hosts, owned resources, exact service/file
locations, existing services that might conflict, expected routing, credentials to
generate, verification probes, rollback trigger and recovery command.

Generate code only inside the private workspace when necessary. Inspect any existing
implementation and its tests before reuse. Cross-compile on a suitable workstation
when that avoids memory/disk pressure on a small edge device. Verify OS/architecture,
download provenance/checksum and dependencies before installation. Do not assume
an old binary/version from an example is current or safe.

Complete local tests of validation, owned-resource cleanup, idempotency/conflict refusal,
interrupted transactions and management survival. Test Linux behavior in an isolated
namespace/VM when available; record that it is not the actual remote host.

## C. Prepare independent recovery before changing networking

Store root-readable backups and an inventory of the working routes, rules, firewall,
SSH and affected service/configuration state. Capture hashes for files expected to remain untouched.
Keep an existing working management session and a separate fresh-login method available.

Install an independent rollback mechanism before apply. It must survive installer
exit, SSH disconnect and failed candidate restart; use an appropriate systemd unit/timer
or equivalent durable watchdog outside the process being changed. Check that it is
armed, can execute, targets only owned resources, and will restore required global flags.
For boot-time activation, ensure an uncommitted/crashed transaction is reconciled after reboot.

Choose a bounded deadline appropriate to the operation; report it to the user.
Do not cancel a rollback merely because a service reports active. If the user loses
access, the rollback must still proceed without a chat message or the installing AI.
Rollback must not flush unrelated firewall rules or restore an obsolete whole-host
snapshot over legitimate DHCP changes.

## D. Establish management first

Generate the edge-local tunnel key and verify/pin bastion identity. Assign an unused
loopback reverse port and record it per device. Prepare account-scoped bastion restrictions.
Before reloading sshd, validate syntax and effective policies for both the tunnel
account and the existing operator account; retain a timed reversal of SSH changes.

Start the independent edge management service. From the operator path, create a
new SSH session through the bastion's loopback listener to the edge, verifying the
original edge host key and actual administrator authentication. Also re-test a new
VPS administrator session. Only then commit management changes.
Verified management remains available if any later data-plane step fails.

## E. Apply the minimal data plane transaction

Validate desired config -> prepare immutable candidate and journal -> arm recovery
-> apply the smallest owned changes -> verify -> commit; otherwise clean up/restore.

Generate transport private keys separately on the two hosts and exchange only public keys.
Apply the endpoint lifecycle separately from policy routing, nftables NAT and shaping.
Classify only the intended traffic, preserve ordinary WAN routing, and configure fail-closed
client egress on the relay. Do not globally redirect local device traffic.

Keep uncommitted state distinguishable from current-known-good state. Do not enable an
unsafe boot path before validation. On a two-host failure, attempt scoped rollback on
both hosts; remote independent watchdogs must cover hosts the installer can no longer reach.
Do not claim distributed atomicity just because two sequential commands succeeded.

## F. Verify and commit

Use [acceptance criteria](acceptance.md). Prove a fresh reverse login, ordinary DNS/HTTPS,
unchanged physical-WAN management path and a real client request that exits through the edge.
Inspect IPv6/client split-routing behavior so direct terminal traffic is not confused with proxy traffic.
Test required UDP functionality separately; a TCP-only probe is not UDP validation.

Commit and enable restart/boot behavior only after the required checks pass.
Cancel each host's rollback only after its committed configuration and recovery path
are verified. Keep a private record of any partial commit and resolve it explicitly.

## G. Failure and SAFE_MODE

On failed verification, immediately initiate the prepared rollback; do not keep the
candidate active while asking the user what to do. Preserve independent management,
normal WAN, DNS and control/release HTTPS. Clean only recorded owned routes/rules,
nftables and shaping objects; stop the data transport and report degraded/safe mode.

If access is lost, wait for the already armed rollback deadline and test a fresh
connection, rather than sending repeated destructive commands or rebooting blindly.
If the mechanism fails or ownership is uncertain, report exact evidence and use the
available cloud/local console or human recovery contact. Do not fabricate restored access.

Recovery from SAFE_MODE is another validate/prepare/apply/verify/commit transaction:
resolve the cause, preserve known-good state, arm a new rollback, apply the minimum
change, prove new management and egress sessions, then commit. Keep credentials and identity.

## H. Private handoff

Give the user actual host/identity and port assignments, installed files and services,
current/previous/factory state locations, last transaction and rollback status, concise
status/safe-mode/recovery commands, private client import details and remaining gates.
Report observed versus desired state; do not present a stored node profile as an active tunnel.
Do not upload this handoff or generated implementation to the public Skill repository.
