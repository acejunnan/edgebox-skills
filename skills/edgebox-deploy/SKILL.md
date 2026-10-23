---
name: edgebox-deploy
description: Plan, implement, deploy, or recover an EdgeBox residential IPv4 egress appliance with a VPS relay and a Linux edge device, preserving independent remote SSH and automatic rollback. Use for this specific appliance workflow, not generic VPN setup or device flashing.
---

# EdgeBox deployment

Deliver a recoverable residential egress appliance for the user's actual hosts.
The public package is knowledge only: create any implementation, build products,
private configuration, and evidence in a separate user-owned private workspace.
Do not modify an existing application or publish generated code merely because this Skill was loaded.

## Choose the requested mode

- **Preparation / shopping:** use the repository's human guide if available. Prices, vendors, plans, purchase links and experience text are author-maintained; do not supply or replace them unless explicitly asked to research or draft those parts. Missing commercial content does not block technical inspection of existing equipment. Do not purchase anything. The guide is optional context, not a deployment dependency.
- **Preflight / plan:** inspect the actual authorized targets read-only, identify gaps and conflicts, then produce a concrete change-and-recovery plan. No network mutation is authorized by a preflight request.
- **Build / deploy:** load [blueprint](references/blueprint.md), [deployment procedure](references/deployment.md), and [acceptance criteria](references/acceptance.md). Generate an implementation only if needed and authorized, then deploy within the user's confirmed scope.
- **Diagnosis / recovery:** load the deployment and acceptance references, inspect ownership and current access before touching networking, and prioritize restoring management and ordinary WAN. Do not turn diagnosis into a reinstall.

Skill invocation alone is not permission to change a remote network. Existing explicit user authorization for the named targets persists; do not ask again for the same approved reversible steps. Purchases, flashing, removal of unrelated services, credential disclosure, and public publishing are separate scope decisions.

## Respect author-maintained content

Preserve the author's introduction, buying recommendations, prices and experience claims.
Treat marked empty fields as intentionally awaiting the author, not a request to invent content.
When asked to polish, change wording without altering commercial facts or conclusions.
Research current offers only when specifically requested, label verified findings separately,
and do not silently promote them to author-approved recommendations. If an author choice
conflicts with technical prerequisites or recoverability, explain the specific conflict and
seek clarification; do not overwrite the text or waive the required technical safeguards.

## Collect facts, not invented defaults

Use connected sources and local SSH configuration where available before asking for information.
Required facts are the two host addresses, architecture/OS, actual SSH user and port,
available authentication method, host identity, and existing workloads. Honor workspace rules;
if no private output directory is specified, choose a new independent directory and report its location.
Use secure credential entry or local key references; do not ask users to paste private keys into chat.
Record precise remote targets in private state, not this public repository.

Read-only preflight must inspect routes, rules, interfaces, firewall state, forwarding,
SSH/service policy, RAM/disk, kernel transport support, WAN DNS/HTTPS and existing EdgeBox state.
If remote tools are absent, produce local reviewable artifacts and explicitly mark remote deployment as not performed.

## Preserve these invariants

1. The edge initiates cloud connections and works behind NAT/CGNAT/dynamic IPv4. No residential port forwarding, UPnP, DDNS or public SSH exposure.
2. Independent management comes first: unique device-local Ed25519 key, restricted bastion tunnel account, pinned host key, loopback-only assigned reverse port, and independent systemd supervision. Prove a fresh reverse login before data-plane activation.
3. The main default route remains the physical WAN route for local SSH, control HTTPS, DNS and release/package retrieval. Classify only intended forwarded/client data into owned policy routes; never redirect all host traffic into the tunnel.
4. Keep transport credentials and endpoint lifecycle separate from routing, nftables NAT and bandwidth policy. WireGuard is V1, not the architecture.
5. Use isolated, owned nftables state, such as `inet edgebox`. Never `nft flush ruleset`, replace unrelated firewall rules, or add new iptables functionality.
6. Every risky network change follows validate → prepare → apply → verify → commit; otherwise rollback. Save current state and arm a rollback path that survives SSH loss and installer death before applying.
7. SAFE_MODE stops the data plane and removes only its owned routes/rules/firewall/shaping objects, preserving ordinary WAN and independent management. No reboot loops.
8. Client egress fails closed when the edge is unavailable. It must not silently fall back to VPS direct egress.
9. Report configured/effective MTU, bandwidth ceiling and separated health domains. A handshake alone is not Internet-egress health.
10. Keep identity, credentials and deployment metadata distinct. Private credentials remain local where possible; use restrictive file permissions and redacted structured logs.

## Product boundary

Assume working wired Internet at the edge; report failure if the assumption is false.
Do not add hotel authentication, Wi-Fi provisioning, temporary management-AP Internet forwarding,
router replacement, firmware flashing, partition or bootloader changes.
An isolated management AP, existing Web console or updater may be retained if already present;
do not add them to a minimal new deployment without a concrete user requirement.

When the user requests the simple installation experience, create a workstation-local window:
two primary address fields, expandable SSH login settings, explicit verified host identities,
plain progress, visible failures/recovery state, and private client handoff after real verification.
Only-IP installation presupposes already usable login credentials. Do not bypass authentication to meet that appearance.
Keep this installer off public interfaces; authenticate local actions and reject cross-origin requests.

## Deliver and stop at the right boundary

Keep builds and secrets outside this public Skill package. Preserve pre-existing code unless
the user specifically asks to change it. Old `acejunnan/edgebox` code is optional reference,
not a required download, bootstrap, or authority over the user's current task.

Use the acceptance matrix to decide what is actually complete. If live verification fails,
roll back before claiming success. If a required capability or authorization is missing,
complete safe independent work and report the exact remaining gap; do not fabricate tests.
Deliver a private host/service/file inventory, client import method, safe-mode/recovery steps,
known-good state locations, test evidence and untested limits. Never publish the handoff automatically.
