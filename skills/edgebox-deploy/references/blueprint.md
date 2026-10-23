# Implementation blueprint

Use this when generating or evaluating a new implementation. It specifies behavior,
not a fixed set of files to copy. Prefer Go, Debian/Armbian, systemd, nftables and iproute2.
Support ARM64 edge devices and AMD64 infrastructure without making hardware model part of identity.
Avoid Kubernetes, microservices, and containers without a concrete need.

## Planes and traffic

| Plane | Responsibility | Survival requirement |
| --- | --- | --- |
| Control | Configuration, registration, releases, structured health and telemetry; outbound HTTPS when implemented | Ordinary physical WAN; no full backend required in v0.1 |
| Management | Human emergency SSH access via edge-initiated reverse tunnel | Independent service and physical WAN; survives data-plane failure |
| Data | Intended client IP/application traffic through the residential WAN | Owned routing/NAT/shaping; may stop safely without taking other planes down |

Normal user path:

```text
client -> VPS client entry -> transport packet endpoint -> edge classification
       -> owned network policy -> IPv4 NAT -> physical residential WAN
```

VLESS + REALITY through a maintained Xray release is a suitable client entry,
with WireGuard as the first internal transport. Verify upstream current configuration
and release integrity from primary sources before selecting a version.
Do not implement MASQUE, custom QUIC, automatic transport switching or relay optimization for this milestone.
The relay proxy's socket/source selection must prevent accidental fallthrough to VPS direct routing.

The transport contract exposes lifecycle, packet/data endpoint, effective MTU and health.
It may be a kernel interface, TUN or userspace queue. No nftables or policy-routing
logic belongs in a WireGuard or future protocol implementation.

## Owned network resources

Discover actual WAN interface, gateway and DNS configuration. Never assume `eth0`,
an address range, port, table, rule priority, qdisc ownership or an unused interface name.
Propose a small explicit allocation, prove absence of conflicts and persist ownership.
An allocation from an old example is not evidence that a resource is free.

- Keep the main default route intact for locally originated device traffic.
- Use dedicated rules/tables with precise selectors for client/forwarded data.
- Prevent fallthrough from the data table into a VPS direct path when the edge is unavailable.
- Scope nftables forwarding and NAT to the endpoint and admitted source traffic;
  block unintended appliance/LAN/private-destination access from the user data endpoint.
- Account for other firewall base chains: an accept in one owned chain does not override a later drop elsewhere.
  Report a coexistence conflict instead of flushing the host firewall.
- Forward only the required IPv4 traffic. Save and restore any required global forwarding change.
- Bind shaping to owned data traffic; do not throttle the physical management path accidentally.
- Keep configured MTU, effective MTU and protocol overhead together; verify suitable packet sizes.
- Include cleanup after normal stop, process crash, failed apply and startup reconciliation.

## Independent SSH management

Generate a unique Ed25519 tunnel key on each edge. Pin a bastion host public key
verified through a trusted channel. Transport keys, tunnel keys, administrator keys
and release-signing keys have different roles; never reuse one shared private key across devices.

The reverse listener defaults to bastion loopback only. Keep a simple persistent
device-to-port registry with unique assignments. No allocation service is needed.

The unprivileged tunnel service uses `ssh -N -T`, `BatchMode=yes`,
`ExitOnForwardFailure=yes`, bounded connect time, server-alive probes and systemd restart.
It has no dependency on the data-plane service or policy table.

Use a dedicated bastion tunnel account with public-key authentication, no password or
keyboard-interactive login, no agent/X11 forwarding, no local or stream-local forwarding,
no useful shell/session channels, no public GatewayPorts, and an exact key-specific
PermitListen/permitlisten restriction where supported. Check effective sshd policy.
Account locking/PAM differences must be tested on the actual OS; never solve them by
enabling password access or removing the forwarding restrictions.

Do not disable an existing administrator login until a separately provisioned
public-key administrator path has been proven by a fresh session and rollback exists.
An old still-open SSH session is not proof that a reconnect will succeed.

## State, configuration and observability

Maintain explicit lifecycle states such as BOOT, NETWORK_WAIT, CONFIGURING,
CONNECTING, ONLINE, DEGRADED, RECOVERING and SAFE_MODE. Do not create empty packages
or claim an unimplemented registration/control service is live.

Keep stable bootstrap information separate from current, previous-known-good and
factory/default runtime state. Persist a transaction journal and ownership record
before changes; recover interrupted transactions at startup. State files must be
atomic and durable enough for the claimed power-loss behavior. Do not claim power-cut
qualification without a physical test.

Report system, WAN, management, control, transport, routing/NAT and actual Internet
egress health separately. Logs should be structured for journald and identify device,
component, event, configuration version and non-sensitive errors. Collect low-volume
uptime, CPU/RAM/temperature, interface counters, reconnects, relay quality, public IP,
active transport/configuration and SAFE_MODE state. No routine bulk speed tests.

No full control backend, billing, geo-routing, full IPv6, A/B filesystem update or
whole-system OTA is required. Preserve room for those features without implementing them.

## Workstation installer, when requested

The human's normal path is addresses -> verified identities -> authenticated login
-> progress -> real validation -> private client handoff. Keep advanced SSH settings
collapsed where existing local credentials make the two-field path possible.

Prefer existing trusted SSH configuration/agent/key references when available;
otherwise request the minimal secure login input. Do not upload private credentials
to a web service. A local browser UI must bind only to loopback, protect mutating
APIs with an unpredictable session secret and same-origin/Host checks, and avoid
persisting credentials or placing them in URLs/logs.

Re-check the pinned host key on every fresh connection. A changed key is a stopped
connection requiring identity resolution, not an excuse for permissive host checking.
The installer may generate code and deploy via authenticated SSH in a private workspace;
the public Skill repository does not receive the generated code or deployment inventory.
