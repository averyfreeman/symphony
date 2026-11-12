# Feature Specification: Multi-App-Server Dashboard

**Feature Directory**: specs/001-multi-app-server-dashboard

**Implementation Branch**: Not created during specification

**Created**: 2026-11-11

**Status**: Draft

**Input**: Configure Symphony to connect to and switch between local and remote Codex app-server daemons, with a minimal UI for selecting the Symphony host network and target host IP.

## Brownfield Context (informative)

- **Existing behavior**: Symphony currently starts app-server processes for worker runs and exposes one orchestrator snapshot in its Phoenix dashboard. The orchestrator records worker_host on running, retrying, and blocked issue entries.
- **Existing contract**: Root SPEC.md sections 13.3–13.7 describe runtime snapshots and the optional observability surface; Appendix A describes SSH workers and the current worker host pool.
- **Compatibility constraints**: The new daemon connections are observability-only. Existing issue polling, assignment, dispatch, retries, and the /api/v1/state response retain their current behavior.
- **Network inventory**: OCI currently reports enp0s6, zt6ntidzpt, tailscale0, and lo. “Tailscale” is displayed as tailscale0; “Localhost” is displayed as lo. tailscaled is a service name, not a network interface.

## User Scenarios & Testing

### User Story 1 - Connect to local and remote app-server hosts (Priority: P1)

As a Symphony operator, I want to see a local app-server and add remote app-server hosts so that the dashboard can observe the daemons available to this Symphony instance.

**Why this priority**: Host connections are the prerequisite for multi-host metrics.

**Independent Test**: Start with a local daemon and one reachable remote daemon; confirm both appear with independent connection status.

**Acceptance Scenarios**:

1. **Given** the Symphony host has a running local app-server daemon, **When** the dashboard opens, **Then** the local host appears through localhost/lo without an SSH connection.
2. **Given** the local daemon is unavailable, **When** the dashboard opens, **Then** the local host is marked unavailable and the rest of the dashboard remains usable.
3. **Given** a remote daemon is reachable, **When** the operator selects a Symphony-host interface, enters the target IP, and connects, **Then** Symphony establishes an SSH-proxied connection and reports the host's status.
4. **Given** the target IP is invalid, the selected interface has no usable address, or SSH/app-server authentication fails, **When** the operator connects, **Then** the UI shows an actionable error without saving credentials or blocking other hosts.
5. **Given** the remote-host form is opened for the first time, **When** it is displayed, **Then** it is prefilled with Moltbuntu at 100.110.240.112 and Tailscale (tailscale0), and no remote connection starts until the operator selects Connect.
6. **Given** an interface is unavailable or cannot route to the target, **When** the operator attempts to use it, **Then** the UI identifies the invalid route; localhost (lo) remains available only for local connections.
7. **Given** a remote profile has been saved, **When** Symphony restarts, **Then** its metadata is restored and Symphony attempts to reconnect without asking for credentials.
8. **Given** the configuration form is open on OCI, **When** the interface options are shown, **Then** available devices include enp0s6, zt6ntidzpt, Tailscale (tailscale0), and Localhost (lo), while tailscaled is not presented as a device.
9. **Given** a profile already exists for the same target IP and interface, **When** the operator submits a duplicate, **Then** the duplicate is rejected and the existing profile remains intact.
10. **Given** the configuration controls are used, **When** a request reaches Symphony, **Then** it remains on the existing private Caddy/Tailscale route and retains browser CSRF protection.
11. **Given** a daemon does not provide a metric, **When** its host view renders, **Then** that metric is marked unavailable rather than shown as zero or inferred from another source.

### User Story 2 - Select which host's metrics are displayed (Priority: P1)

As an operator, I want a dashboard host selector separate from connection setup so that I can switch the displayed metrics without changing how Symphony assigns issue work.

**Why this priority**: The selected host must be unambiguous while connection setup remains independent.

**Independent Test**: Connect a local daemon and a remote daemon with distinct fixture metrics; switch between each host and All Hosts and verify the displayed data source.

**Acceptance Scenarios**:

1. **Given** at least one daemon profile exists, **When** the operator selects Local, a remote host, or All Hosts, **Then** the dashboard displays the corresponding host metrics.
2. **Given** Symphony issue entries are associated with a selected worker host, **When** that daemon is selected, **Then** only the associated running, retrying, and blocked issue entries appear with the daemon metrics.
3. **Given** a worker is configured under an SSH alias, **When** the operator associates that worker identity with the matching host profile, **Then** its issue entries appear only in that host's dashboard view.
4. **Given** a daemon has no matching Symphony worker-host association, **When** it is selected, **Then** daemon metrics remain visible and the issue section clearly reports that no Symphony issue data is mapped.
5. **Given** the dashboard uses All Hosts, **When** it renders, **Then** existing aggregate orchestrator metrics remain available and host-specific daemon data is labeled by host.
6. **Given** Symphony has global cumulative totals without host attribution, **When** a single host is selected, **Then** the dashboard does not present those global totals as host-specific values.
7. **Given** a host is selected, **When** metrics refresh, **Then** only read-only observations occur and no issue work is dispatched or rerouted.
8. **Given** an existing client requests GET /api/v1/state, **When** host selection is added, **Then** the response retains its current shape and aggregate meaning.
9. **Given** a daemon returns thread metrics, **When** those metrics are displayed, **Then** prompt and message contents are not shown or persisted.
10. **Given** multiple daemons use the same Codex account, **When** rate limits are displayed, **Then** the dashboard labels them as account-scoped rather than separate per-host quotas.

### User Story 3 - Recover from interrupted connections (Priority: P2)

As an operator, I want connection failures and stale data to remain visible so that an offline daemon does not make the dashboard appear healthy or hide metrics from other hosts.

**Why this priority**: Remote hosts and tailnet routes can become temporarily unavailable.

**Independent Test**: Disconnect a configured remote host and verify its stale/offline state while the local host remains selectable.

**Acceptance Scenarios**:

1. **Given** a configured host becomes unreachable, **When** the connection fails, **Then** the host is marked offline or stale with its last successful update time.
2. **Given** a configured host becomes reachable again, **When** Symphony reconnects, **Then** the host returns to connected status and fresh metrics replace stale data.
3. **Given** the operator removes a host profile, **When** removal completes, **Then** Symphony disconnects its metrics reader without stopping the remote daemon or its active work.
4. **Given** one host is offline, **When** another host is selected or refreshed, **Then** the other host remains responsive and its data is available.
5. **Given** the selected profile is removed, **When** removal completes, **Then** the selector returns to All Hosts.

### Edge Cases

- The OS interface list changes after startup, or an interface is down or has no address compatible with the target IP.
- A remote profile uses the loopback interface with a non-loopback target.
- A host is removed while selected; the dashboard returns to All Hosts.
- A duplicate profile is submitted for the same target IP and source interface.
- A remote daemon does not provide rate-limit data or has no readable thread metrics.
- A worker host is configured with an SSH alias rather than the target IP.
- A host becomes unavailable during a dashboard refresh.

## Scope

### In Scope

- Discover current Symphony-host network interfaces dynamically and allow selection for remote SSH egress.
- Detect a local daemon through the local control socket and present it as the Symphony host.
- Add, connect, remove, and persist remote daemon profiles; prefill Moltbuntu at 100.110.240.112 with Tailscale (tailscale0).
- Use the existing OpenSSH identity/configuration and an SSH proxy to the remote app-server control socket; do not expose a remote WebSocket listener.
- Provide a separate host selector for Local, each configured host, and All Hosts.
- Display per-daemon thread/activity and token metrics, rate limits when available, connection health, and related Symphony issue entries when a worker-host association exists.
- Keep the existing private Caddy/Tailscale dashboard access boundary.

### Out of Scope

- Selecting, balancing, or routing new issue work across connected daemon profiles. Record this as a future pooling feature; current worker.ssh_hosts behavior remains independent.
- Starting, stopping, resuming, editing, archiving, or deleting remote threads.
- Exposing conversation prompts or message contents in the dashboard.
- Adding public REST endpoints or changing the existing /api/v1/state response for this version.

## Requirements

### Functional Requirements

- **FR-001**: The configuration UI MUST list usable interfaces discovered on the Symphony host, including enp0s6, zt6ntidzpt, tailscale0, and lo when present.
- **FR-002**: The interface selector MUST identify the Tailscale device as tailscale0 and loopback as localhost (lo); it MUST NOT offer tailscaled as an interface unless the OS reports it as a network device.
- **FR-003**: The remote-host form MUST accept a valid IPv4 or IPv6 literal and a selected Symphony-host interface. Its initial values MUST identify Moltbuntu at 100.110.240.112 and select tailscale0; first connection MUST require an explicit Connect action.
- **FR-004**: Selecting lo MUST be limited to local/loopback connections. Remote connections MUST use an address assigned to the selected interface as their source route.
- **FR-005**: The local app-server MUST be discovered through the Symphony host's local control socket when available.
- **FR-006**: Remote connections MUST use the Symphony process's existing non-interactive OpenSSH identity/configuration to proxy the daemon control socket. The UI MUST NOT request or store SSH passwords, private keys, Codex tokens, or other credentials.
- **FR-007**: Host profiles MUST persist across Symphony restarts in the app-owned registry at ~/.config/symphony/app_servers.json, with restrictive file permissions. The registry stores connection metadata only.
- **FR-008**: The UI MUST show per-host connection status, last successful update, and actionable failure state. A failed host MUST NOT prevent selecting or viewing another host.
- **FR-009**: The dashboard MUST provide a separate selector with Local, each configured host, and All Hosts. All Hosts is the default selection.
- **FR-010**: A selected host view MUST show that daemon's available thread/activity and token metrics, account-scoped rate limits when available, and last-update status. Missing metrics MUST be identified as unavailable, not represented as zero.
- **FR-011**: Symphony issue rows MUST be filtered by the selected profile's worker-host association. Global orchestrator totals MUST NOT be attributed to one host unless the source provides that attribution.
- **FR-012**: The dashboard MUST clearly indicate when a daemon has no mapped Symphony issue data and MUST keep daemon metrics separate from orchestrator issue metrics.
- **FR-013**: Remote app-server access MUST use read-only operations. Connecting, selecting, disconnecting, or removing a profile MUST NOT start, stop, or mutate a thread or turn, and MUST NOT dispatch or reroute issue work.
- **FR-014**: Host profiles and errors MUST NOT expose or persist prompt contents, message contents, authentication secrets, or private keys.
- **FR-015**: The existing aggregate dashboard and GET /api/v1/state response MUST remain backward-compatible.
- **FR-016**: A remote profile MUST be associable with an existing Symphony worker-host identity when that identity cannot be matched from the target IP alone.
- **FR-017**: The system MUST reject a duplicate profile for the same target IP and source interface with a clear validation message.
- **FR-018**: Account rate-limit data MUST be labeled as account-scoped and MUST NOT be presented as an independent quota for each host.

### Operational and Privacy Requirements

- **OR-001**: A disconnected host MUST remain selectable with an explicit offline/stale status and last successful update time.
- **OR-002**: Connection and refresh work MUST NOT block LiveView rendering or the other host connections.
- **PR-001**: Connection controls MUST remain behind the existing private dashboard access path and retain browser CSRF protection.

### Key Entities

- **Host Profile**: A local or remote app-server host with a stable identity, display name, target IP when remote, selected source interface, optional worker-host association, connection status, and last successful update.
- **Host Metrics Snapshot**: Read-only thread/activity, token, and rate-limit data labeled with its host and observation time.
- **Symphony Issue Snapshot**: Existing running, retrying, and blocked entries from the single orchestrator, filtered only when a worker-host association is known.

## Success Criteria

### Measurable Outcomes

- **SC-001**: An operator can identify the local daemon and configure Moltbuntu using the required network selector and target IP without editing workflow files.
- **SC-002**: Selecting Local, Moltbuntu, or All Hosts displays data only from the selected source or sources and labels each source.
- **SC-003**: An unavailable host is visibly offline or stale while other host views continue to work.
- **SC-004**: Local and remote issue states are never confused with daemon metrics or attributed using global totals without host evidence.
- **SC-005**: Connecting or removing a host produces no issue dispatch or remote thread/turn mutation.
- **SC-006**: Existing aggregate dashboard and JSON observability clients continue to work without response-shape changes.

## Assumptions

- The Symphony host runs Linux and can enumerate its interfaces and assigned addresses. Current OCI interfaces include enp0s6, zt6ntidzpt, tailscale0, and lo.
- The local app-server daemon is already managed on the Symphony host and its control socket is accessible to the Symphony process user.
- Remote hosts already have remote control enabled and can be reached from Symphony using non-interactive OpenSSH authentication. The UI reports authentication failures instead of collecting credentials.
- The dashboard continues to be served through the existing private Caddy/Tailscale path.
- A profile is associated with Symphony issue state only when its worker-host identity can be matched; otherwise only daemon metrics are shown.
- Future issue pooling across these daemon profiles is tracked separately from this feature.
