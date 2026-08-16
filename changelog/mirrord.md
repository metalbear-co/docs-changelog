---
title: mirrord Changelog
date: 2022-02-01T00:00:00.000Z
lastmod: 2026-08-16T00:00:00.000Z
draft: false
images: []
weight: 100
toc: true
tags:
  - oss
  - team
  - enterprise
description: >-
  The release changelog for mirrord.
---

## 3.247.0 - 2026-08-12


### Added

- Browsers can join a session from a share link, with no extension.
- Multi-cluster preview replicas.


### Changed

- `mirrord ui` now shows the state, e.g. idling or active, of preview sessions.

## 3.246.0 - 2026-08-10


### Added

- The agent now replaces the `Cache-Control` header of HTTP responses that went
  through it with `no-cache, no-store, must-revalidate`, so that browsers and
  caching proxies don't cache responses served while mirrord redirects a
  target. Set the new `agent.override_cache_control` config option to `false`
  to turn this off.


### Changed

- mirrord can now use `target.path.labels` to target every matching pod in a
  namespace (requires operator), allowing one local session to intercept
  traffic across multiple workloads that share the configured labels.


### Fixed

- Fixed concurrency issues in mirrord-agent logic for outgoing connections.

## 3.245.0 - 2026-08-07


### Added

- Added Windows crash diagnostics for `mirrord exec`, producing a crash record,
  memory dump, and report for native faults and external kills.
- Added a `--key` filter to `mirrord session ls`, letting you list only the
  active local and in-cluster sessions started with a given session `key`.
- Added support for specifying a kube context in `mirrord up`. In order of
  precedence, it can be set:

  1. with the `--context` argument when running `mirrord up` (highest
  precedence)
  2. with the `context` field under a service in the configuration file
  3. with the `common.context` field in the configuration file

  If none of these are set, the default behaviour remains the same.
- Configuration templating now exposes a `git_branch` variable holding the
  current git branch, so a
  config can derive values from it, for example giving each branch its own
  session key. Outside a git
  checkout the variable stays undefined, so pair it with the `default` filter
  when the same config
  also has to work there.


### Changed

- Raised the default CPU limit on agent pods from `100m` to `1` core, so agents
  are not throttled under heavier traffic. Set `agent.resources` to override.
- Set `TCP_NODELAY` on the agent's connection to its clients, so messages sent
  to a session are not held back by Nagle's algorithm.
- Updated the `kube` fork to 4.2.0. `TCP_NODELAY` is now set on connections to
  the Kubernetes API
  server, so requests are not held back by Nagle's algorithm.
- `/etc/ssl/certs` is now read from the remote target by default, so the local
  process trusts the same
  certificate authorities as the target when talking to services in the
  cluster. Add the path to
  `feature.fs.local` to restore the previous behaviour.


### Fixed

- Fixed Windows `pitm` reusing a stale layer DLL after the mirrord binary was
  upgraded by giving its extracted layer the existing per-build unique
  filename.
- Fixed Windows applications reporting the wrong error when a requested local
  port was unavailable.
- Fixed the Windows layer's Java debugger-port auto-detection.
- Fixed the Windows mirrord JetBrains extension leaking application processes
  when a Debug session is terminated.
- Fixed the config wizard generating invalid config when path or header filter
  is set.
- Renewing an expired client certificate no longer requests a CI credential
  from the operator.
  Users whose stored certificate had expired failed to start a session with
  `Enterprise license is
  required for generating mirrord CI api key` unless the operator ran on an
  Enterprise license.
- Send the user-provided session key when creating a copy target.
- The `chaos edit` command no longer returns a "422 Unprocessable Entity"
  error.
- `mirrord up` now splits Kafka topics automatically.

## 3.244.1 - 2026-08-02

## 3.244.0 - 2026-08-02


### Added

- Added `profile` field for selecting administrator-defined db branch
  configuration profiles.
- Added an `sslmode` connection parameter for CockroachDB branching, for use
  when the connection is configured with individual parameters instead of a
  URL.

## 3.243.0 - 2026-07-31


### Added

- Added `auto_queue_splitting` option for copy target. Set to `true` by
  `mirrord up` command
  to inform the operator that parameters of unsupported queue kinds shall be
  dismissed rather than
  rejected.


### Changed

- Connections accepted by the layer now have `TCP_NODELAY` set to reduce
  latency.
- `experimental.guard_std_fds` is now enabled by default in OSS (still off by
  default in mfT).


### Fixed

- Stolen HTTP requests no longer wait out a retry backoff on a connection the
  local application has already closed. Connections were cached for reuse
  without checking whether they were still open, so a request that drew a
  closed one from the cache failed its first send attempt and waited 50
  milliseconds before making the connection it could have made immediately.
- `getaddrinfo` calls that pass `AI_NUMERICHOST` are no longer resolved
  remotely. That flag asks whether the given string is already a numeric
  address rather than for a name lookup, and must fail when it is not, so
  resolving it reported every hostname as a literal address.

## 3.242.0 - 2026-07-30


### Changed

- Enabled IPv6 support by default; set `feature.network.ipv6: false` to
  disable.
- Enabled `TCP_NODELAY` on every socket used to relay mirrord traffic - layer
  to internal proxy, internal proxy to external proxy and to the agent, agent
  outgoing and passthrough connections, redirected incoming connections, and
  the local sockets used for intercepted and port-forwarded traffic. Relayed
  data was already framed by the app on the other end of the hop, so Nagle's
  algorithm only added latency to small writes.


### Fixed

- Added the `experimental.guard_std_fds` config (off by default), which keeps
  the layer's internal proxy connection from breaking when the process starts
  with a closed standard fd. The connection socket could be assigned fd 0-2 and
  get reconfigured by the app's runtime (e.g. `libuv` setting `O_NONBLOCK` on
  `stdin`), killing the process with "Resource temporarily unavailable." This
  broke Next.js with Turbopack, which spawns its worker processes with `stdin`
  closed.
- Database branch failures now surface the failure reason instead of a bare
  timeout.
- `mirrord operator status` now more clearly labels machine session counts.

## 3.241.0 - 2026-07-29


### Added

- Added a `replace` service mode to `mirrord up`. A service run in `replace`
  mode copies
  the target workload and scales the original down to zero.
- Added templating support with tera to `mirrord up`.
- Added the `mirrord chaos` command for managing chaos rules. The available
  subcommands are `list`,
  `add`, `edit` and `delete`. Silently starts the local UI if needed. New rules
  can be provided as a
  file or to `stdin`, and output can be JSON format or pretty printed.


### Changed

- The "Generic" DB-branching type is now capitalized in the status output,
  again.


### Fixed

- Fixed `mirrord up` services with `run.type: container` running with an empty
  default config: the spawned `mirrord container` child resolved its config
  from scratch instead of using the resolved config passed by `mirrord up`,
  silently dropping the target, `feature.env.override`, remote environment, and
  HTTP filters.
- Fixed corrupted permissions on files created through `openat64` or
  `openat$NOCANCEL` on a path that
  mirrord handles locally. Both hooks dropped the variadic `mode` argument
  before bypassing to libc,
  so libc read whatever value happened to occupy that argument slot.

## 3.240.0 - 2026-07-27


### Added

- `mirrord up` can now run a subset of services (`mirrord up service-a
  service-b`), and services can be marked `skip: true`.


### Fixed

- Fixed `mirrord exec` on Windows failing to launch targets whose executable
  path contains a space (e.g. Python installed under `C:\Program Files`), by
  quoting the child command line instead of naively space-joining the
  arguments.
- Fixed a `layer <-> intproxy` protocol framing bug that broke sessions where
  an app opened many concurrent outgoing connections.
- Windows layer now falls back to a random local port when the requested bind
  port is busy, matching the unix layer's behavior.

## 3.239.0 - 2026-07-27


### Changed

- Update config doc about `agent.external_ip_fix`.


### Fixed

- Fixed `mirrord ui` returning 404 for every page on Windows: the Windows
  release build skipped the frontend build, so released binaries embedded no UI
  assets.
- Fixed an issue where chaos UI not refreshing existing rule definition if
  the rule isn't updated by using the UI.
- Fixed the broken [Tera](https://keats.github.io/tera/) template engine link
  in the configuration docs.

## 3.238.0 - 2026-07-24


### Added

- Added container and image-native Flyway DB branch migration flavors.


### Changed

- The mirrord ui dark mode now uses neutral dark gray surfaces with the brand
  purple reserved for accents, improving contrast over the previous
  purple-on-purple scheme.


### Fixed

- Fixed 410 session errors when reusing an existing copy target.

## 3.237.0 - 2026-07-22


### Changed

- Job and CronJob targets no longer require enabling the `copy_target` feature
  manually. Since these targets have no long-running pod to attach to, mirrord
  now enables copy target for them automatically and tells you it did so,
  instead of failing config verification.


### Fixed

- Made the mirrord logo on the config wizard homepage render identically in
  both themes, on its own periwinkle chip, instead of only getting the chip in
  dark mode.

## 3.236.1 - 2026-07-21


### Fixed

- Fixed a bug in `mirrord-auth` causing seat counting client key pair being
  re-generated when running
  a burst of mirrord sessions concurrently.

## 3.236.0 - 2026-07-21


### Added

- Add CockroachDB database branching support.
- Add IRSA support for AWS RDS branching.
- Add support for specifying a custom image per db branch via the `image` field
  in `feature.db_branches` (all db types).
- If the operator advertises the `DiagnosticsPing` feature, mirrord uses the
  new ping endpoint for
  diagnosing local-to-cluster latency.


### Changed

- Preview sessions that fail to start are no longer immediately deleted, to
  allow further inspection of why they failed.

## 3.235.0 - 2026-07-20


### Added

- Add MariaDB database branching support.
- Support mirrord chaos testing rule management via `mirrord ui` web
  application.


### Fixed

- Terminate the local process when the agent connection is lost and cannot be
  recovered, instead of silently entering failover and leaving it running
  disconnected as a zombie that keeps holding its ports.

## 3.234.0 - 2026-07-17


### Added

- Added idle mode for preview environments (`feature.preview.idle`).
- Added the `MIRRORD_AGENT_STEALER_FLUSH_CONNECTIONS_CONNTRACK` agent
  environment variable. When set to `false`, the connection flush performed
  when stealing starts skips the `conntrack -D` command and relies only on `ss
  -K`, avoiding a burst of dropped redirected connections (and accompanying
  `ENOENT` errors) on busy ports. Defaults to `true`, preserving the previous
  behaviour.
- `mirrord up` now infers a service's target from its key in `mirrord-up.yaml`
  when `target.path` is omitted, prompting you to pick one when nothing
  matches.


### Changed

- Fix bug in `AddressFilter::Name` that allowed port filters to match hostnames
  ports incorrectly.
- Fix error `File not found` being shown when running `mirrord ui stop` with no
  mirrord UI running.
- Renamed `feature.preview.idle.timeout_secs` to `sleep_after_secs`.
- The environment variable used to control the logging level for the mirrord
  CLI and layer has been changed from `RUST_LOG` to `MIRRORD_LOG`.


### Fixed

- Fixed `mirrord operator session kill --id` rejecting the names of
  multi-cluster sessions, which carry an `mc-` prefix.
- Fixed local redis database branch startup so readiness no longer depends on
  `redis-cli` being installed on the host.

## 3.233.0 - 2026-07-14


### Added

- Detect missing queue splitting configuration.
- Inject `mirrord-key` into all forwarded messages.
- Preview environments can now selectively filter which labels are copied from
  the target through the `feature.preview.labels.{include,exclude}` options,
  analogous to our existing `feature.env.{include,exclude}` options.
- Support CronJob and Job database branching targets.
- `mirrord preview start` now overrides existing sessions with the same key and
  target by default, essentially making `--force` the default behavior.
  Consequently, that argument is now unnecessary and deprecated.


### Changed

- Copy target with an HTTP filter now tailors its warning to the connected
  operator's behaviour
  instead of always claiming that unmatched requests are discarded.
- List active sessions through the operator's active-sessions API.

## 3.232.0 - 2026-07-12


### Removed

- Removed the experimental `go_cgo_stack_switch` flag. The Go 1.25+ cgo
  stack-switch fix is now covered only by the `go_asmcgocall` experimental
  flag.
  `go_asmcgocall` is now enabled by default for OSS users.


### Added

- Add a retryable agent error variant.
- Add support for generic db branching.
- Added `container.host_gateway_detection`, enabled by default, which makes
  mirrord container detects
  and connects through host gateway.
- Added `feature.preview.secret_mounts` to mount files into a preview pod from
  a Kubernetes Secret, so sensitive files can be access-controlled via RBAC
  separately from the session.
- Added a context and namespace selector to the `mirrord ui`, backed by a new
  context/namespace-aware `/api/v2` API. You can now view cluster sessions per
  kube context and filter them by namespace, independently in each browser tab.
  Local sessions always show, and each one displays the context and namespace
  it runs against. The previous `/api/*` routes stay in place for backward
  compatibility.
- Added support for `jq_filter` in Kafka queue splitting. The jq program runs
  on a JSON representation of each Kafka message (topic, partition, offset,
  timestamp, key, payload, and headers), and messages for which it outputs
  `true` are routed to the local application. Requires a mirrord Operator
  version that supports jq filters for Kafka.
- Allow sharing preview environments.


### Changed

- API routes for managing chaos rules are now under `/api/chaos/rules` instead
  of `/chaos/rules`.
- Add queue splitting config support to `mirrord up`.
- Improve error message when no `mirrord-up.yaml` config file is found as part
  of `mirrord up`, suggesting `mirrord up init`.
- Requests for chaos rules with a latency effect now specify `"read_ms"` and
  `"write_ms"` instead of
  `"delay_ms"`. They are applied in the read and write directions respectively
  and cannot both be 0.
- The `mirrord ui` session monitor and the `mirrord wizard` config wizard are
  now a single web app
  served by `mirrord ui`. `mirrord wizard` opens the same app on the config
  wizard tab, and both
  features share one server, one theme, and a light/dark toggle.


### Fixed

- Requesting a database branch whose engine is disabled on the mirrord operator
  now fails with a
  clear "not enabled on the mirrord operator" message, instead of an opaque
  `404 page not found` or a
  hang waiting for the branch to become ready.
- The `mirrord ui` session monitor no longer shows a duplicate logo inside the
  merged UI, and its
  kube context, namespace, and account controls now share the shell's top bar
  instead of a separate
  row. The logo is also no longer inverted in dark mode.
- The `mirrord ui` session monitor no longer shows an operator error when the
  operator's status
  service is momentarily unavailable (e.g. during a pod restart). It now keeps
  the last-known team
  sessions on screen and displays a "Reconnecting to operator…" hint instead.
- The config wizard's mirrord logo is now legible in dark mode, shown on a
  light backdrop instead of blending into the dark card.

## 3.231.0 - 2026-07-08


### Added

- Queue splitting support for mirror mode (`queue_mode: mirror`), delivering
  matched messages to both the session and the deployed application.


### Changed

- Adds handling for chaos errors for ongoing connections.


### Fixed

- When running `mirrord container` command from a WSL instance, docker bridge
  between WSL instances
  may not be ready by the time mirrord's proxy container sidecar connects its
  proxy process, e.g.
  Docker Desktop WSL to another WSL. Added connection retry mechanism that will
  make the end-to-end
  flow reliable.
- operator status for SQS now shows jq-filter-only sessions

## 3.230.0 - 2026-07-07


### Added

- Add support for ClickHouse.
- Add support for Google Spanner.
- Added `mirrord ui` API endpoints to list available kube contexts and to list
  namespaces for the current or a specified context.
- Implement Flyway migrations.
- mirrord config files can now be loaded from `stdin` by passing `-` to the
  `--config-file` or `-f` arguments.


### Changed

- Add support for ClickHouse.
- Preview sessions will no longer fail to spawn when targeting workloads with
  very long names.
- The `mirrord ui` command now starts the local UI server as a separate process
  that runs in the background on your machine.
  To stop the server, run `mirrord ui stop`.


### Fixed

- Fixed a phantom session in `mirrord ui` showing an empty target and "NaNs"
  uptime that could not be removed and crashed the page when clicked. The
  WebSocket `session_added` payload now carries the same flat session shape as
  `GET /api/sessions`.
- Fixed passthrough requests failing when the target app listens on the pod's
  external IP instead of loopback. The new `agent.external_ip_fix` option,
  enabled by default in the open source CLI, fixes this.
- Fixed the database branching config docs repeating the shared branch fields
  (`connection`, `id`, `name`, `ttl_secs`, `ttl_mins`, `creation_timeout_secs`,
  `version`, `migrations`) once per engine, which produced duplicate `-1`/`-2`
  anchors on the config options page. They are now documented once.
- Relaxed rules used when searching the prebuilt SIP-utils bundle, fixing SIP
  issues on some macOS versions.

## 3.229.0 - 2026-07-05


### Changed

- Due to the Windows MSI installer limit of 255 for the major and minor version
  components,
  updated mirrord's MSI installer versioning scheme so that a CLI version of
  `3.256.1` is
  mapped to an MSI installer version of `1.3.2561`.
- Properly upstream errors from the session monitor to the UI server.

## 3.228.0 - 2026-07-03


### Added

- Added Google Secret Manager as a source for database branching connection
  parameters, so the branch data copy can fetch secrets (such as the source
  database password) at runtime without putting them in the mirrord config, a
  Kubernetes Secret, or the pod spec. Set `env_var_name` to also expose the
  resolved value to your local process for a local or preview override.


### Changed

- Updated `kube-rs` to 4.0. The Kubernetes client now falls back to the
  operating system's native trust store through the rustls platform verifier
  when a kubeconfig sets no CA, rather than to a bundled set of root
  certificates.
- Adjust go hook implementation that's behind the `go_asmcgocall` experimental
  flag.
- Changed experimental `go_cgo_stack_switch` go hook implementation to resolve
  g TLS offset dynamically.

## 3.227.0 - 2026-07-02


### Added

- Added `mirrord queues` (alias `mirrord qs`) to browse the status of active
  queue-splitting sessions, including each session's phase, target, the queues
  resolved from the target, and which target pods are patched and ready.
- Added a list form for the `split_queues` config so the same queue id can be
  used more than once across different broker types.
- Added support for DynamoDB database branches, including `empty` and `all`
  copy modes and IAM-based authentication for reading source tables.


### Fixed

- Fixed a use-after-free in the macOS `getifaddrs` hook (used when
  `hide_ipv6_interfaces` is enabled) that could make applications see garbage
  interface addresses and fail DNS resolution.

## 3.226.0 - 2026-07-01


### Added

- Added an experimental `go_asmcgocall` flag that makes the Go 1.25+ `syscall`
  hook on `x86-64` reuse the Go runtime's own `asmcgocall` stack-switching
  routine (as the `arm64` hook already does), fixing crashes seen in some Go
  programs that use `cgo`.


### Changed

- Added per-connection client-to-agent write back pressure to intercepted
  outgoing connections.

## 3.225.0 - 2026-06-30


### Added

- Added `feature.db_branches[].connection_settings` for PostgreSQL database
  branches. mirrord sends these PostgreSQL session settings on every source
  connection it opens while building the branch, so they take effect for the
  schema dump and the data copies. This lets a branch copy read tables guarded
  by a Row-Level Security policy that depends on a session variable.


### Changed

- Don't suggest a bug report on agent-reported errors.

## 3.224.0 - 2026-06-29


### Changed

- Improves the error message when `mirrord container` fails to read the
  intproxy address, or when
  the TLS PEM file cannot be accessed.
- Marked `experimental.latency` configuration as deprecated. To perform chaos
  testing, use the
  mirrord chaos feature instead.
- Updated the chaos outgoing latency implementation so that busy connection's
  latency
  will not accumulate. Latency chaos rules will now add the specified latency
  amount
  more accurately.


### Fixed

- Fixed the issue that chaos latency is not applied to the read direction of
  outgoing connections.
- Queue splitting and other operator features now work through ingress proxies
  (e.g. GKE Connect Gateway) that reject the encoded JSON in the connect URL
  query string, by sending the connect parameters in a header when the operator
  supports it.

## 3.223.0 - 2026-06-25


### Added

- mirrord now support running on clusters managed by
  [`vcluster`](https://www.vcluster.com).


### Changed

- Stripped instrument and logging in layer functions that Golang hooks make
  call to in order to
  reduce stack consumption.
- Tweaked the Go 1.25+ syscall hook's stack switch to mimic the runtime's
  `asmcgocall` contract,
  avoiding mutation of goroutine scheduler state that may cause intermittent
  crashes.


### Fixed

- Fixed `mirrord ui` not syncing its token to the browser extension when the
  page loads without a `?token=` query string, such as from the `Open mirrord
  ui` button or after a reload. The page now reads the token from a new
  authenticated `/api/token` endpoint and forwards it to the extension.

## 3.222.0 - 2026-06-24


### Added

- Adds the pre-mvp implementation of mirrord chaos testing. With this feature,
  you can artificially add latency and connection errors for outgoing traffic
  through an API that runs as part of `mirrord ui`.


### Changed

- - added `--no-browser` flag to prevent the browser opening automatically
  - UI now respects `BROWSER` env var for selecting which browser to open with
  - UI auth token can be set as `x-auth-token` header
  - `mirrord exec` now includes session ID in progress printout

## 3.221.1 - 2026-06-23


### Fixed

- Fixed a segfault in the macOS DNS configuration hook
  (`dns_configuration_free`) that crashed short-lived Node workers (e.g.
  Next.js/Turbopack) on macOS 26 when remote DNS config could not be built.
- Stopped the layer from reporting `EOPNOTSUPP` ("Operation not supported on
  socket") socket errors as hard layer errors, which flooded logs and could
  kill processes such as Turbopack workers.

## 3.221.0 - 2026-06-22


### Changed

- The bundled Apple utilities are now re-extracted to `~/.mirrord/binaries`
  only when their version changes, tracked via `~/.mirrord/binaries_version`.


### Fixed

- Fixed Next.js development servers crashing under mirrord when Turbopack
  compiles CSS. Turbopack runs its pooled Node workers over a loopback TCP
  connection back to the parent process. To keep that connection local, mirrord
  now detects Next.js processes and enables the outgoing `ignore_localhost`
  option, so the worker handshake succeeds.

## 3.220.0 - 2026-06-19


### Added

- Add `copy.dump_args` for PostgreSQL and MySQL database branches, allowing
  users to override the arguments passed to `pg_dump` and `mysqldump`.


### Changed

- Migrating `RabbitMQ` queue splitting to the unified
  `operator-queue-splitting` crate and `CRDs`.

## 3.219.0 - 2026-06-18


### Added

- Added `mirrord subscribe` command that streams operator interception events
  for a session key as JSON.


### Fixed

- Fixed a bug in SIP-patch flow using the bundled coreutils.

## 3.218.0 - 2026-06-16


### Added

- Running `mirrord ui` while one is already running now detects the existing
  instance and prints its URL with the reused token instead of failing to start
  a second server.


### Changed

- Split the `mirrord up init` save step into separate save/filename prompts,
  and re-ask for a filename instead of aborting when declining to overwrite.
- `mirrord up init` no longer writes commented-out template lines for omitted
  defaults.


### Fixed

- Fixed incorrect deprecation warnings emitted by mirrord when
  `experimental.sip_utils` config is used.
- Reject duplicate incoming port mappings instead of silently dropping one of
  them.

## 3.217.1 - 2026-06-14


### Fixed

- Fixed the Windows computer-name hooks `GetComputerNameW`,
  `GetComputerNameExW`, and `gethostname` to respect the caller's buffer and
  the standard size-probe contract. A long remote pod `hostname` no longer
  makes .NET's `Environment.MachineName` throw, which broke clients such as
  `StackExchange.Redis` with a `RedisConnectionException`. It also no longer
  breaks outgoing `TLS`: `SChannel` queries the `NetBIOS` name with a fixed
  buffer during the handshake, and a too-long name made
  `AcquireCredentialsHandle` fail with `SEC_E_SECPKG_NOT_FOUND`, breaking
  `HTTPS` and `gRPC` to external services such as `GCP`.

## 3.217.0 - 2026-06-14


### Added

- Add support for `Redis Pub/Sub`.
- Add support for redis with `location: remote`.
- Added support for splitting [Temporal](https://temporal.io/) task queues.
- Added the number of concurrent Preview Environment session to the output of
  `mirrord operator status`.
- The `--key` argument for `mirrord exec`, `mirrord preview`, and `mirrord up`
  can now be provided with the `MIRRORD_KEY` environment variable.


### Changed

- A new command argument `--resolved` is added to the `mirrord verify-config`
  subcommand
  to display a fully resolved mirrord config including user inputs and all
  default
  values being used.
- Renamed the mirrord UI identity label to "Running as" (previously "Signed in
  as") to reflect that it shows the active cluster identity queried by the CLI
  rather than an authenticated session.
- `feature.network.dns.filter` is no longer marked as unstable in config docs
  or CLI warnings.


### Fixed

- Fixed concurrent DNS resolution intermittently failing under mirrord on
  Windows. A burst of in-flight async `GetAddrInfoExW` queries, such as a .NET
  `HttpClient` issues under load, could exhaust the managed-handle registry's
  single-lock retry budget and drop a query, which surfaced to the app as a
  `WSAHOST_NOT_FOUND` `SocketException`. The registry now shards its lock
  across independent shards, so concurrent registration no longer serializes on
  one lock.
- Fixed concurrent file I/O contending the Windows `IOCP` file-binding map
  under mirrord. Many parallel open/close operations serialized on a single
  `RwLock`, so `bind` and `unbind` could give up under contention and drop a
  file's async-read completion binding, or leak its entry on close. The binding
  now lives in the file's managed-handle record, which shards its lock across
  independent locks and drops the binding atomically when the handle closes.
- Preview environments now hash the session key before placing it in the
  `PreviewSession` resource's label. In practice this means keys containing
  characters like `/`, which are invalid for Kubernetes labels, are accepted.

## 3.216.0 - 2026-06-07


### Added

- Added `-u` flag to `up` that opens the UI in parallel to running `mirrord
  up`.
- Added `internal_proxy.ping_interval` to configure how often intproxy checks
  the agent connection.


### Fixed

- Database branches with a user-specified `id` are now shared across workloads.
  Two sessions targeting different deployments that point at the same database
  and use the same branch `id` now reuse a single branch instead of each
  creating its own.

## 3.215.1 - 2026-06-04

## 3.215.0 - 2026-06-04


### Added

- `mirrord up` now reports a single aggregate analytics event per invocation
  describing how the multi-service configuration is shaped, alongside the
  existing per-session events. Honors the `telemetry` opt-out.

## 3.214.0 - 2026-06-03


### Added

- Preview environments can now control the number of pod replicas in a session
  through the new `feature.preview.replicas` option.
- `mirrord ui` now works on Windows, using a named pipe restricted to the
  current user instead of the mode 0o600 unix socket.


### Changed

- Refreshed experimental feature configuration default values.
  `hook_rename`, `dns_permission_error_fatal`, `force_hook_connect` are
  removed.
  `non_blocking_tcp_connect` and `sip_utils` are now `true` by default and
  marked as deprecated.
  `disable_reuseaddr` is set to `true` by default for OSS users.


### Fixed

- Fix so if db-branching resource name is generated will not exceed the 63
  character limit of k8s.

## 3.213.0 - 2026-05-28


### Added

- Add support for Unix sockets of type `SOCK_SEQPACKET`.
- Added the `s3Event` flag to the operator queue registry CRD.
- Windows: added support for asynchronous file reads (fixes
  `File.ReadAllTextAsync` etc).
- mirrord now automatically reads the target container's volume mounts
  (ConfigMaps, Secrets, PVCs) from the remote pod and serves those paths
  read-only from the remote, so the local process transparently sees its
  mounted files. Controlled by the new `feature.magic.auto_mount` flag, enabled
  by default.


### Changed

- Wizard target selection now lets users choose the target container.


### Fixed

- Agent now sets up ip6tables rules whenever the IPv6 stack is usable
  (including IPv4-only clusters that still have an IPv6 loopback), so traffic
  to `[::1]` is no longer silently excluded from steal/mirror.
- Fixed mirrored/stolen HTTP traffic failing to reach a local server bound to a
  wildcard address such as `0.0.0.0` on Windows: the intproxy HTTP gateway now
  normalizes the forward address to loopback before connecting, matching the
  raw-TCP path.
- Fixed the issue where foreground CI session prints stdout and stderr even
  though output directory
  is set in mirrord CI configuration.
- Implemented support for Windows async DNS resolution, and solved scenarios
  where DNS resolution would wrongly pass through local.

## 3.212.0 - 2026-05-22


### Added

- Added operator-session listings to the local `mirrord ui`: a Team tab that
  shows cluster-wide sessions, and a 3-step Connect operator wizard for the
  no-operator state.
- Preview environments are now resilient to pod crashes and evictions.
- Preview environments now support mounting user-supplied files in the preview
  pod via a new `spec.config_mounts` field on `PreviewSession`.


### Changed

- Preview sessions are now deleted automatically when their TTL expires, and
  failed sessions are cleaned up automatically after the retention window
  configured by `operator.preview.cleanupAfterMins` in the chart configuration.
  Alongside this change, `mirrord preview status` now only shows active
  sessions by default, and a new `--failed` flag lets you inspect failed
  sessions that haven't been cleaned up yet.
- Rewrite README intro to name both halves of the developer + AI coding agent
  feedback loop. Update `metalbear.co` links to `metalbear.com`.


### Fixed

- Fixed an issue where remote DNS resolution would always fail for host names
  containing an underscore.
- Fixed broken pipe error in install script.
- Fixed panic on Windows when resolving config with `feature.magic.aws`
  enabled. The auto-generated `~/.aws` path mapping now uses the same
  home-directory transform as the `layer-win` filter logic (drive letter
  stripped, slashes flipped, regex-escaped), so it no longer produces an
  invalid regex from a Windows `USERPROFILE` (`HOME`) like `C:\Users\foo`.
- Host names listed in `feature.network.outgoing.filter` are now automatically
  mirrored into `feature.network.dns.filter` so that DNS resolution for each
  host happens on the same side (local app or remote pod) the connection is
  routed to.
- Reject configurations that put the same environment variable in both
  `feature.env.override` and `feature.db_branches[].connection`.
- `mirrord operator status` and `mirrord session list` no longer show
  preview-env entries. Previews are still surfaced in the local `mirrord ui`
  and browser extension via `MirrordOperator.status.sessions`, but the CLI's
  session-management surfaces hide them so the displayed ids, ports, and
  queue-splitting state always reflect real exec sessions. Long-term
  unification of previews and exec sessions is tracked separately.

## 3.211.0 - 2026-05-19


### Added

- Add `iam` auth for `mysql`.
- Add support for `Azure Servicebus` queue splitting.
- Add support for unified queue splitting `azure` + `gcp` for preview env.
- Added `mirrord up init`, an interactive wizard that generates a skeleton
  `mirrord-up.yaml`.
- HTTP protocol detection on redirected connections now applies a read timeout
  (default `2s`, configurable via `agent.http_detection_timeout` /
  `MIRRORD_AGENT_HTTP_DETECTION_TIMEOUT`) instead of waiting indefinitely for
  the client's first byte. This unblocks server-first protocols such as SMTP
  that previously stalled in detection.

## 3.210.0 - 2026-05-13


### Added

- Added `feature.preview.ttl_secs` and `feature.db_branches[].ttl_mins` so each
  feature accepts the TTL in either unit. Setting both `ttl_secs` and
  `ttl_mins` on the same item is rejected.
- Added clarifications for Windows-specific `fs` mapping and `fs` filter quirks
- Document jq filtering support for GCP Pub/Sub in the configuration docs.


### Changed

- In mirrord-up, renamed `mode` to `default_mode` and added a CLI argument to
  set it.


### Fixed

- Agent now falls back to an IPv4 client listener when setting up the
  dual-stack IPv6 one fails (e.g. on clusters with IPv6 disabled).
- Fixed `getaddrinfo` truncating IPv6 addresses resolved through the agent.
- Fixed outgoing connections to path-addressed Unix sockets failing with a "not
  found" error when the agent runs as an ephemeral container.
- The agent no longer logs a spurious warning when a DNS server returns an
  error response code, and layer no longer logs bogus DNS errors with error
  level.

## 3.209.2 - 2026-05-09


### Fixed

- Fixed remote Unix socket connections failing for some applications, including
  when the socket lives in a directory mounted into the target pod.
- `mirrord ui` no longer drops operator sessions whose `user` field doesn't
  match the `username/k8s_username@hostname` format. Falls back to the raw user
  string for both fields, so synthetic owners like preview environments surface
  in the session list.

## 3.209.1 - 2026-05-01


### Fixed

- Add missing `file` utility in `appleutils`

## 3.209.0 - 2026-04-30


### Added

- Support for single cluster sessions in Multi Cluster.


### Changed

- Bump bundled `appleutils`

## 3.208.0 - 2026-04-30


### Added

- Added support for GCP Pub Sub.


### Changed

- `mirrord up` now rejects unknown fields in `mirrord-up.yaml` to catch typos
  in configuration.
- `mirrord up` now reports a clear error when `mirrord-up.yaml` is missing,
  with help text on how to specify a custom path.


### Fixed

- Fixed Windows DNS hook error reporting so DNS lookup failures now return the
  correct WinSock error, for example `WSAHOST_NOT_FOUND` code `11001`, instead
  of surfacing unrelated errors like `ERROR_INVALID_HANDLE` with `os error 6`.
- Fixed a Node/libuv crash when mirrord's Unix `getaddrinfo` hook returned a
  non-`EAI_*` error after remote DNS found no records.
- Fixed example in `agent.image_pull_secrets` configuration docs to use `name`
  instead of `secret-key`.
- Fixed local `Redis` containers/processes not being cleaned up.
- mirrord now expands templates inside the root `key` field, so you can derive
  session keys from expressions like `{{ get_env(...) }}` in the config file.

## 3.207.0 - 2026-04-28


### Added

- `mirrord ui` now auto-configures the mirrord browser extension once the user
  opens its Web UI in Chrome, so users no longer have to copy a
  `chrome-extension://...` configure URL by hand. The session UI binds on
  `127.0.0.1` instead of `[::1]` so the page origin matches what the
  extension's `externally_connectable` manifest entry accepts.

## 3.206.1 - 2026-04-24

## 3.206.0 - 2026-04-24


### Added

- `mirrord ui` now polls `MirrordOperator.status.sessions` and exposes the
  result through a `/api/operator-sessions` HTTP endpoint, letting the browser
  extension list existing sessions grouped by their `key`. Adds `key` and
  `http_filter` to `Session` so external clients can identify which sessions to
  join and derive the matching header to inject.


### Fixed

- Fix issue where username can't be determined breaking operator use

## 3.205.0 - 2026-04-24


### Added

- Added `mirrord up`, a tool for spawning and managing multiple concurrent
  mirrord sessions from a single `mirrord-up.yaml` file.
- Support for composite environment variables (with `value_pattern` regex) and
  multi source connection parameters in db branching.


### Changed

- Agent now automatically decides when to enable IPv6 based on IPs assigned to
  available interfaces.


### Fixed

- DB branch credentials from a literal value or Secret reference now override
  the local env vars correctly, even when the target pod does not set them.

## 3.204.1 - 2026-04-20


### Fixed

- mirrord go run on macOS regression fixed

## 3.204.0 - 2026-04-20


### Added

- Added `pitm` subcommand for Windows as a necessity to support JetBrains IDE
  extensions.
- Support for plain value credentials in db branching.


### Fixed

- tailwind wrong config didn't bundle `metalbear-ui`.

## 3.203.1 - 2026-04-17


### Fixed

- Changed the release workflow to build Linux and Windows CLI artifacts through
  `cargo xtask build-cli` instead of handwritten cargo build steps.
  Fixes regression with wrong artifact on Linux causing mirrord to not work

## 3.203.0 - 2026-04-16


### Added

- Add a Settings dialog to the session monitor UI (accessed via the gear icon
  in the header) with a toggle for anonymous usage analytics. The preference
  persists in the browser's local storage. When turned off, the client calls
  `posthog.opt_out_capturing` and stops any active session recording
  immediately; the session's own `config.telemetry = false` still vetoes
  telemetry regardless of the UI toggle.


### Fixed

- Fix `mirrord ui` serving 404 in release builds: enable `corepack` on the
  macOS CLI release jobs so the xtask actually builds the monitor frontend.
  Also permit the PostHog telemetry host in the UI `Content-Security-Policy`,
  and mask all text and inputs in session replays so they do not capture
  customer-sensitive content surfaced by the UI.

## 3.202.0 - 2026-04-16


### Added

- Add -n and -f flags to session commands
- Add token-based authentication, CORS, Host header validation, and security
  response headers to the session monitor API to protect against cross-origin
  attacks on localhost.
- Added a web frontend for the local session monitor API under
  `monitor-frontend/`.


### Changed

- Fix the squashed mirrord logo in the session monitor header and align the
  monitor's typography, color tokens, and header layout with the
  operator-dashboard's design system so both apps share the same look.


### Fixed

- Fix an issue where mirrord would wait indefinitely if the agent image could
  not be pulled (e.g. ImagePullBackOff, ErrImagePull). The CLI now surfaces a
  clear error instead of hanging.
- Adds agent support for multiple mirrord sessions that have the same target
  pod, but they're targeting different containers of this pod. Previously the
  second agent iptables' would take over the first agent's, now iptables' chain
  names are dynamic, and thus avoid this problem.
- Fix a bug where unknown fields in some nested mirrord config sections were
  accepted during deserialization instead of being rejected.
- Fix the root layer process being absent from the local UI processes list.
  Also update the `mirrord ui` help text from "Launch the session monitor UI"
  to "Launch the mirrord local UI".

## 3.201.0 - 2026-04-10


### Added

- Added `mirrord diagnose license` for troubleshooting mirrord for Teams
  license usage.
- Added local and remote session management commands under `mirrord session`.
- Implemented `mirrord attach` on Windows to support IDE extensions.


### Changed

- Preview environments now have a default HTTP header filter, `baggage:
  *.mirrord-session={{key}}.*`, which will be used if no filter is configured.
- Preview environments now ignore the config option
  `feature.network.incoming.http_filter.ports` to prevent accidentally stealing
  traffic without a filter. This means that all HTTP filters now
  unconditionally apply to all intercepted ports.

## 3.200.0 - 2026-04-09


### Changed

- Add Ruby and sh files to default local filter
- Moved agent container command line arguments from `command` to `args` in
  order to enable matching with GKE Autopilot WorkloadAllowlist.
- Identify local redis database branch instances by db branch id instead of by
  port.

### Fixed

- Fix preview environments failure when using it in CI contexts where a mirrord
  user isn't configured.
- Preview env resolve target backwards compatible fix.

## 3.199.0 - 2026-04-06


### Added

- Emit a warning when agent settings are configured but ignored because the
  mirrord operator is managing the session.


### Changed

- Adjusted error message for client failure to connect to operator


### Fixed

- fixed regression of mirrord container introduced in 3.196.0
- mirrord container on colima now works without disabling TLS

## 3.198.0 - 2026-04-02


### Added

- Added `mirrord ui` command that launches a local web dashboard for monitoring
  all active mirrord sessions in real time. Each session now exposes a
  per-session HTTP API on a Unix socket for health checks, session info, event
  streaming (SSE), and session termination.
- Preview env support for Multi Cluster.
- When DB branches are enabled, intproxy now automatically creates portforwards
  to DB pods. Active portforwards can be viewed with `mirrord db-branches
  connections`.


### Fixed

- Changed the progress printout to specify `(cli version X.Y.Z)` to avoid
  confusing output in operator commands.
- Fixed an issue where new versions of the mirrord CLI were sometimes not able
  to use old versions of the mirrord Operator (`Unable to decode to pem
  body...` error).


## 3.197.1 - 2026-03-31


### Fixed

- Fixed windows python `socketpair` regression. affecting versions
  3.190.0-3.196.0


## 3.197.0 - 2026-03-31


### Added

- Added RabbitMQ queue splitting support.


### Changed

- Check for existing key (preview env) and warn the user.
- `mirrord ci start --foreground` command behaves exactly like `mirrord exec`.

## 3.196.0 - 2026-03-28


### Added

- Add `mirrord ci container` command: starts a mirrord for CI session using a
  container, similar to mirrord container.
- Preview environments now support DB branching.
- Preview environments now support `feature.network.incoming.ignore_ports`.
- Preview environments now support environment variable manipulation
  (`feature.env`).
- Preview environments now support queue splitting.
- Support for Copy Target for Multi Cluster.


### Fixed

- Fixed regression using `mirrord.exe exec` without an already existing
  `MIRRORD_LAYER_FILE` environment variable.


## 3.195.0 - 2026-03-26


### Changed

- Apple binary bundle is now included in mirrord CLI binary.
- Bumped dependencies
- Run `mirrord ci start` instead of `mirrord exec` when the user ran `mirrord
  exec` but the environment variable `MIRRORD_CI_API_KEY` is set.
- Using pre-built Apple utility binaries is on by default for OSS users.


### Fixed

- Fixed a misleading "invalid state" error shown when the target node has
  reached its pod limit: mirrord now explains the real cause and suggests
  freeing pod capacity or using ephemeral containers.
- SIP no longer needlessly patches binaries that allow DYLD side loading like
  Node but are runtime-protected


## 3.194.0 - 2026-03-19


### Added

- Revamped the interactive configuration wizard frontend for generating `mirrord.json`
  files, with cluster-aware target selection, traffic mode configuration, and
  JSON export.
- Added Windows installation via Chocolatey (`choco install --pre mirrord`)
- Added header filtering using jaq for more complex queries, removed safejaq
  crate
- Experimental flag 'experimental.sip_utils' that uses our own built binaries
  instead of patching
- Preview environment's usage is now collected through our telemetry system


### Changed

- HTTP filter docs now instruct users to use W3C baggage/tracestate
- Modified non-blocking TCP connect mechanism to eliminate delays.
- `mirrord preview start` no longer replaces an existing session automatically
  when giving it the same image registry, image repository, target and key. To
  replace an existing session use the new `--force` argument, which will
  replace it regardless of whether the images "match" or not.


### Fixed

- Fixed the example in the documentation for outgoing traffic.
- Preview env will now respect `MIRRORD_PROGRESS_MODE` when printing info.


## 1.193.0 - 2026-03-05


### Changed

- Make db branching type to be optional, now it always checks for env and
  env_from.
- Updated the `experimental` section of the mirrord config:
  1. Marked `hook_rename` as deprecated.
  2. Marked `dns_permission_error_fatal` as deprecated.
  3. Marked `force_hook_connect` as deprecated.

## 3.192.1 - 2026-03-04

No significant changes.


## 3.192.0 - 2026-03-02


### Added

- You can now update the configuration and/or image of an existing preview
  environment session by re-running `preview start` with the same key, target,
  image registry and image repository.
- `mirrord preview stop` now resolves the given target's container before
  comparing it to the preview sessions, allowing you to specify, for example,
  `deployment/foo`, instead of having to spell out the full target, including
  container.
- The `feature.preview.ttl_mins`/`--ttl` setting for preview environments now
  accepts the `"infinite"` string value, which makes the session live
  indefinitely until being manually stopped.


## 3.191.0 - 2026-02-26


### Added

- SQS splitting now supports filtering by jq programs. A jq program runs
  against a json of each message and if it returns true, the messaged is
  filtered.
- `mirrord dump` no longer requires `--ports` to be specified manually - they
  will be auto-detected if absent.


### Changed

- Updated mirrord for Teams links across CLI, error messages, and agent to
  point directly to app.metalbear.com instead of docs pages, providing a
  shorter path to trial signup.
- Use nodeSelector when possible for agent creation. Improve
  capacity/scheduling issues.

  When using nodeName directly (old, fallback way) we bypass kube scheduler,
  meaning we can't preempt existing pods.
  By using nodeSelector, we still use kube scheduler and with the right
  priority class for the agent (default in operator chart now) we always get
  scheduled.
  User might not have "get" access on node, but operator always has so that's
  why we have a fallback.


## 3.190.0 - 2026-02-23


### Added

- Add multi cluster session to the CLI status.
- The `mirrord preview status` command will now show the remaining TTL of each
  preview environment session.


### Changed

- Update reqwest dependency


### Fixed

- Fixed a Windows file-read regression where remote reads could return EOF too
  early when reading beyond the first chunk of data.


## 3.189.0 - 2026-02-19


### Added

- Added feature.magic.aws — allows using the AWS CLI within mirrord with the remote pod's identity by default.
- Inject mirrord session key headers into incoming requests.
- mirrord now supports "Preview Environments" — a new type of mirrord session that runs directly in the cluster and can be shared with other users.

### Fixed

- Show all branches in the db-branching status command when no branch name is specified.
- `mirrord dump` no longer hangs if the required `--ports` flag is not specified.
  It also no longer reports an incorrect error when no target is specified.

## 3.188.2 - 2026-02-13


### Fixed

- Rolled back the change for loading into arm64e, fixing many bugs on macOS.


## 3.188.1 - 2026-02-12

Re-releasing 3.188.0 for pipeline mistake.

## 3.188.0 - 2026-02-12


### Added

- Expose feature.env.load_from_process as env MIRRORD_ENV_LOAD_FROM_PROCESS


### Fixed

- Don't require emulation for macOS arm64e
- The db-branches status command would fail when one of the branching features
  (MySQL/PostgreSQL) was not enabled on the cluster.


## 3.187.0 - 2026-02-09


### Added

- Add new functionality to support multi cluster sessions.


## 3.186.0 - 2026-02-06


### Added

- SIP - add option to re-sign binary using codesign for santa usecases if santa
  detected.
- Add new flag for debugging deadlock in mirrord layer - controlled by env
  MIRRORD_NODEADLOCK.

### Fixed

- Fixed socket regressions.

## 3.185.1 - 2026-02-04


### Fixed

- Fix double listen making port unsubscribed. Happens in Rails.

## 3.185.0 - 2026-02-03


### Removed

- Removed operator setup command, use helm chart instead.


### Added

- .NET 10 switched to using $NOCANCEL variants of the libc functions.
  A full list of functions affected can be found under
  [this issue](https://github.com/dotnet/runtime/issues/117299).
  For each function in the list that `mirrord` already hooks a detour,
  `mirrord` now hooks the $NOCANCEL variant as well.


### Changed

- Add concurrent ci sessions count to `mirrord operator status`.


### Fixed

- Fixed CI start command output directory on macOS.
- Layer will now reuse intproxy connections across `exec` calls. This fixes
  remote resource tracking in the intproxy.


## 3.184.0 - 2026-01-27


### Added

- "traceparent" and "baggage" fields can now be added to the mirrord
  configuration and propagated in requests to the
  operator. Follows OTel/W3C context propagation conventions.
- Added `agent.disable_mesh_sidecar_injection` config option that controls
  relevant annotations are added to agent pods that disable service mesh
  sidecar injection.


### Fixed

- Fix issue with subscriptions being removed when multiprocessing is used.
- Fixed an issue where mirrord was not able to deliver incoming traffic due to
  firewall rules on the local machine.
- Fixed bug on Windows where `fstat` would fail on Node.js read file operation.


## 3.183.0 - 2026-01-23


### Added

- Add bypassed requests metric.
- Added BSD `connectx(2)` detour.
- Added `mirrord kubeconfig fix`, which finds non-absolute paths in kubeconfig
  `exec` fields and interactively replaces them with absolute paths to make
  them `$PATH`-independent.
- Db Branching support for MongoDB.
- Experimental configuration to add artificial latency to outgoing connections


### Changed

- Improved error messages produced in case of mirrord-agent pod failure.
- make /Program Files always local, helps when debugging Python in WSL


### Fixed

- Agent will no longer ignore trailing slashes when handling file open requests
- Fixed bogus error logs printed by the agent on failed reverse DNS lookups.
- Link to tree-sitter version that doesn't break release build.


## 3.182.0 - 2026-01-16


### Added

- Add support for db branches IAM.


### Changed

- `mirrord` now sets agent priority class for targeted agent pod as well.


### Fixed

- Agent will now terminate the intproxy side of HTTP connections that were
  closed by the remote HTTP client.
- Fixed a regression caused by panicking when closing a user socket that is
  already dropped.


## 3.181.1 - 2026-01-14


### Fixed

- `mirrord` now unsets env from process even if the process is skipped on
  macOS.
- `getsockname`/`getpeername` will now correctly report IPv4-mapped-IPv6 addresses when necessary. 
  When the client receives an IPv6 connection on an IPv4 server socket, `127.0.0.1` will be reported instead.


## 3.181.0 - 2026-01-12


### Removed

- Removed traffic mirroring implementation based on packet sniffing, along with
  related configuration options.


### Added

- Control agent repository/tag via env vars
- Added new operator feature that allows bypassing user credential verification
  when running mirrord ci start.


### Changed

- Allows `mirrord ci start` to be run multiple times (now you can start
  multiple mirrord ci sessions), and `mirrord ci stop` will stop every session.
- Unified `http_filter.ports` and `incoming.ports` behavior and made it more
  intuitive. Now, ports will be stolen unfiltered/not stolen when
  `http_filter.ports`/`incoming.ports` is set AND the port in question is not
  in the list.


### Fixed

- Connections redirected by the agent will now correctly terminate when the
  target pod is killed
- Ports will now be unsubscribed only when `close.2` has been called on all
  handles referring to the matching socket.


## 3.180.0 - 2026-01-06


### Added

- Add support local redis in db branches.
- Added an experimental configuration for logging all Apple variables before
  process start.
- Support for limiting outgoing connections from mirrord sessions using a
  kubernetes policy has been added to the operator.


### Changed

- Changed `experimental.force_hook_connect` default value to `true`.
- Changed changelog instructions
- Make file not found logs more detailed


### Fixed

- Fixed file not found Windows build regression
- iptables cleanup would fail if some rules were already deleted.


## 3.179.0 - 2025-12-24


### Added

- Agents can be told to cleanup leftover iptable rules on startup instead of
  erroring out.


### Changed

- Fixed two issues found in DB branching:
  - resolve env when target type is rollout
  - support envFrom as connection source
- Print browser extension configuration URL if Chrome couldn't launch
- When targeting an empty deployment with the operator, mirrord now checks if
  the deployment is managed by a rollout with the same name. If this is the
  case, the user is instructed to target that rollout instead.


### Fixed

- mirrord will now fail (panic) if SIP patching encounters the `"Too many open
  files"` error. To fix this, the user can
  try increasing the file limit using `ulimit`.


## 3.178.0 - 2025-12-20


### Added

- Added the `mirrord wizard` command, which starts the onboarding wizard in
  your browser.
  You can use the wizard to learn about mirrord basics and create a config file
  from existing boilerplate.


### Fixed

- fix openapi v3 spec of Target

## 3.177.0 - 2025-12-18

### Changed

- Added /var/db/timezone/zoneinfo to local to improve performance.
- Refreshed configuration documentation.
- Switched `mirrord port-forward` to evaluate connections eagerly. Using it
  with MySQL is now possible.

## 3.176.0 - 2025-12-07


### Added

- Added support for node.js'
  [`fs.copyFile`](https://nodejs.org/api/fs.html#fspromisescopyfilesrc-dest-mode)
  function.
- Added clang in build tools list, so mirrord only patches it and hooks exec
  and spawn.
- Added security context configuration option for agent pod.

## 3.175.0 - 2025-12-03


### Added

- Add pg branching.
- Added dlopen detour to support cgo library being loaded dynamically on Linux.
  The feature is behind an experimental feature flag.
- Added support for filtering incoming HTTP requests by running JSONPath
  queries on the body and matching the results against a regex.


### Changed

- Change tracing to not emit ansi format logs in the terminal
- Made `mirrord ci stop` command idempotent.
- Marked mirrord config settings related to agent's mirroring as deprecated.
  Mirroring implementation based on raw sockets will be removed in the future.
- Updated the `experimental` section of the mirrord config:
  1. Removed deprecated `readlink` setting.
  2. Removed deprecated `readonly_file_buffer` setting, which had been moved to
  `feature.fs`.
  3. Removed `vfork_emulation` setting. vfork emulation is now always enabled.
  4. `hook_rename` is now enabled by default.
  5. `dns_permission_error_fatal` is now enabled by default.


### Fixed

- Fixed a rare layer crash in `exec`
- Fixed an issue where mirrord-agent could fail with an OOM error when serving
  heavy TCP traffic.
- Fixed main branch Windows build errors and warnings
- mirrord-console now compiles again


## 3.174.0 - 2025-11-20


### Added

- Added HM Courts & Tribunal Service to ADOPTERS.md.
- Added a new option under `mirrord ci start` to run the binary and wait for it
  to complete.


### Changed

- Allow an overlap in the configuration for `incoming.ports` and
  `incoming.http_filter.ports`.


### Fixed

- Fixed minor edge case where intproxy might hit code that's supposed to be
  unreachable and panic
- Fixed a DNS regression that happens when user's DNS library bind their
  outgoing
  socket to unspecified address before send UDP message to the nameserver.

## 3.173.2 - 2025-11-18


### Changed

- Exclude `GOPATH` and `GOMODCACHE` from env fetched

## 3.173.1 - 2025-11-16

No significant changes.


## 3.173.0 - 2025-11-16


### Changed

- Update `experimental.non_blocking_tcp_connect` config default to `false`.


### Fixed

- Fixed the security advisories of the config's template engine by updating it
  to the newest version


## 3.172.0 - 2025-11-06


### Removed

- Reverted arm64e support in mirrord-sip.


### Changed

- When possible, mirrord now binds application socket to the requested address
  (instead of always to localhost or unspecified).
- Reverted independent send handles for intproxy tasks.
- The `fs.not_found` filesystem filter is now checked *before* bypassing
  relative files, meaning you can now ignore relative files instead of always
  reading them locally
- `incoming.listen_ports` mappings now always apply, regardless of whether the
  port is stolen or not.
- mirrord CLI progress messages now always include the binary version.

## 3.171.0 - 2025-11-04


### Added

- Added ADOPTERS.md
- Added configurable creation timeout to the db branching config
- mirrord can now run arm64e binaries, both when inside a universal binary and
  standalone. In practice, this means running arm64e+x86_64 universal binaries
  no longer has to go through the Rosetta emulation layer.


## 3.170.0 - 2025-10-29


### Added

- Added copy mode configuration for MySQL branching.


### Changed

- `experimental.non_blocking_tcp_connect` now defaults to `true` in OSS.


### Fixed

- Fixed a bogus error reported by `mirrord ci stop` when the application has
  already exited.


## 3.169.0 - 2025-10-27


### Added

- `mirrord ci start` now pipes application's stdout and stderr to files in a
  temporary directory (`/tmp/mirrord/{binary-name}-{timestamp}-{random}` by
  default). This can be configured with the `ci.output_dir` option in your
  mirrord config.


## 3.168.1 - 2025-10-24


### Changed

- Operator status not showing data for SQS split queues.
- When it's root, CWD will no longer be in the default-local list for file
  operations.


## 3.168.0 - 2025-10-23


### Added

- CLI for db branches to show and destroy


### Changed

- Changed semantics of `internal_proxy.log_destination` and
  `external_proxy.log_destination` fields.
  Now mirrord auto detects whether the path leads to the desired log files, or
  directories where the log files should be created.


### Fixed

- fix rare case of fork and deadlock


## 3.167.0 - 2025-10-17

### Added

- Add Kafka splitting info to the operator status.
- mirrord now makes outgoing connect attempts concurrently.

### Fixed

- Fixed a bug in the `mirrord port-forward` command where half closed streams
  were not properly handled, causing "missing" traffic.
- Fixed issues with outgoing connections to abstract UNIX sockets.
- `mirrord ci` subcommand is now visible in the `--help` output.

## 3.166.0 - 2025-10-15


### Added

- Added new config option `experimental.non_blocking_tcp_connect`. The setting
  enables a new way of handling outgoing TCP connections,
  which should improve performance of asynchronous applications.
- Added 2 new commands: `mirrord ci start` which runs mirrord in a ci runner,
  and `mirrord ci stop` which stops the mirrord-for-ci.
- add MIRRORD_DONT_LOAD to crash mirrord if exists when mirrord loads to enable
  users running destructive actions with mirrord


### Changed

- Moving operator tests from mirrord to operator.


### Fixed

- Improved error messages originating from parsing configuration files.
- Fix JSONSchema to reflect that target.path can be string


## 3.165.0 - 2025-10-09

### Changed

- intproxy compiles on Windows.
- All configuration snippets for agent are now self-contained.

### Fixed

- Fixed the check that disallows operator-only targets when the operator is not
  in use.

## 3.164.0 - 2025-10-06


### Added

- Added a new command to generate mirrord CI API key for operator users.


### Changed

- Improves the agent handling of background tasks by having them run in a
  separate tokio runtime. Also adds a timeout for waiting on these tasks to
  finish (timeout of 5 seconds).
- Removed the outdated operator version check.


### Fixed

- Disallow nested `mirrord exec` and `mirrord container` executions.
- Fixed an issue where mirrord sessions would sometimes fail after reconnecting
  to the operator.


## 3.163.0 - 2025-09-19


### Added

- Improved UX when spawning the agent pod fails or takes a long time
- Added a configuration option to automatically inject `Mirrord-Agent` header
  into responses to HTTP requests served by the mirrord-agent.
  The header indicates whether the request was handled by the local
  or the remote app. Header can be enabled with `agent.inject_headers`.


### Fixed

- Fixed compatibility issues with Go 1.25.


## 3.162.0 - 2025-09-16


### Added

- Mirroring incoming TCP traffic now supports using an HTTP filter.
- mirrord now retries failed Kubernetes API requests made during startup. This
  should improve stability when facing transient errors, like cluster
  connectivity issues.

  The retry policy can be adjusted in startup_retry mirrord config section.


### Changed

- Agent->layer message logs will no longer contain environment variables.
- Update TELEMETRY.md to include the machine_id metric in the OSS.
- Updated so if the PingPong will not attempt to request restart if not
  connected to operator.


### Fixed

- Intproxy now no longer runs as a child of the user process so it doesn't get
  reaped by wait/waitall calls.


## 3.161.0 - 2025-09-04


### Added

- Added config for specifying mirrord-tls.pem path when using mirrord container
  command.


### Changed

- Changed MySQL database branch's default TTL to 5 minutes.


### Fixed

- Fixed an issue where mirrord was not handling gRPC error responses properly,
  resulting in errors like "server closed connection with RST_STREAM without
  sending trailers".
- Fixed iptables backend detection on kernel 6.12+.


## 3.160.0 - 2025-09-02


### Added

- Added hook for the `rename` function, enabled with the
  `experimental.hook_rename` setting.
- Introduced `MysqlBranchDatabase` CRD.
- `workloadRestartTimeout` field for `MirrordWorkloadQueueRegistry`, that
  controls the timeout for the target workload to restart on the first SQS
  session start of a target.


## 3.159.2 - 2025-09-01


### Changed

- A mirrord session can now be terminated early if a remote DNS lookup fails
  with `permission denied` error.
  This error indicates that the Kubernetes cluster is hardened and the
  mirrord-agent might not be fully functional.
  The behavior is controlled with the `experimental.dns_permission_error_fatal`
  setting, and enabled by default in OSS.


### Fixed

- Fixed a compatibility issue with openapiv2 for operator CRDs


## 3.159.1 - 2025-08-27

### Changed

- Default "/run/" and "/selinux" to local.
- SimpleProgress implementation now handles prints nicely instead of debug
  print.
- Changed some Debug implementations to not print HTTP body.

### Fixed

- Removed conntrack-related bogus warnings from mirrord-agent logs.

## 3.159.0 - 2025-08-24


### Added

- Add method filter to HTTP filters.


### Changed

- Proper error message when the operator cannot be retrieved, and it's
  required.
- Improve copy target schema docs
- In SIP-patched binaries, set the identifier to start with the original
  filename, like the codesign binary does.
- `agent.passthrough_mirroring` now defaults to `true` in both OSS and mfT.
- change vfork emulation to be enabled by default, fixes different issues
  around the use of it
- mirrord container: change to always use native arch for intproxy, improving
  performance significantly

## 3.158.0 - 2025-08-20


### Added

- Added rtt metrics (prometheus) for checking mirrord and operator latency.


### Changed

- mirrord-agent and mirrord-intproxy now include the crate version in the
  produced HTTP 502 responses.


### Fixed

- Fixed a bug in automatic iptables backend detection in the mirrord agent.


## 3.157.2 - 2025-08-14

### Added

- Added periodic dump of connected processes for diagnostic purposes.
- Added the logfile location to the `intproxy` command-line so it can be seen in
  the process output.

## 3.157.1 - 2025-08-13


### Fixed

- Fixed an issue where the mirrord-agent would sometimes hang and linger in the
  cluster.


## 3.157.0 - 2025-08-08

### Added

- Added a new configuration option to ignore specified containers and init containers
  when copying the session target.

## 3.156.0 - 2025-08-06


### Added

- On Linux, mirrord now checks whether the target binary is dynamically linked.
  If it's not, a warning is issued.


### Fixed

- Fixed incorrect parsing of PATH-like `kubeconfig` config values.
- Fixed an issue where mirrord agent was not able to pass through gRPC requests
  without scheme or authority in the URI.


## 3.155.0 - 2025-08-05


### Added

- When picking a target container automatically, mirrord now takes the
  `kubectl.kubernetes.io/default-container` annotation into consideration.


### Changed

- mirrord policy CRDs now allow for blocking scale down when using the copy target feature.


### Fixed

- Fixed a bug where mirrord-layer was encountering issues due to `vfork` in the
  user application. The fix is hidden behind the `experimental.vfork_emulation`
  mirrord config flag.
- Fixed issues with default mirrord CLI progress mode and interactive cluster auth plugins.


## 3.154.1 - 2025-08-04


### Fixed

- Fixed an issue where `mirrord dump` command was not receiving HTTP traffic
  from an agent configured to use passthrough mirroring.
- Fixed an issue with how mirrord-intproxy handles operator connection errors.


## 3.154.0 - 2025-07-30


### Added

- Add machine_id to UserData so it can be used in analytics.
- Extended `MirrordKafkaClientConfig` CRD with a kind of a SASL OAUTH token
  provider to use.


### Fixed

- Added "Ready!" message when `mirrord port-forward` command finishes setup.
- Fixed a bug in `kube` crate which did not show messages printed out from
  interactive cluster auth.
- Fixed progress message printed when mirrord automatically adds probe ports to
  `feature.network.incoming.http_filter.ports`.
- Reverted changes to SIP log levels that might cause bad instructions under
  the hood.


## 3.153.0 - 2025-07-28


### Added

- Added the `mirrord newsletter` command, which opens the sign-up page in the
  browser.


### Changed

- Passthrough mirroring is now enabled by default, unless mirrord for Teams is
  used.


### Fixed

- Fixed a bug where the SIP patching process was discarding too many open files error
  during layer injection.
- Fixed a typo in Istio Ambient warning message.

## 3.152.1 - 2025-07-22


### Fixed

- Agent communication port now uses the `SO_REUSEADDR` flag, fixing cases where agent
  port is reused in a fast consecutive manner and fails.
- Fixed a bug where mirrord-agents were lingering after all client connections
  were gone.


## 3.152.0 - 2025-07-18


### Added

- Added the config option `experimental.sip_log_destination` to write basic SIP logs to a file.

## 3.151.0 - 2025-07-16


### Added

- Added a new traffic mirroring implementation, in which the connections are
  redirected using iptables. The new implementation can be enabled in the
  agent configuration with the `agent.passthrough_mirroring` flag.

## 3.150.0 - 2025-07-14


### Changed

- Changed semantics of the `agent.nftables` configuration field.

  When the field is not set, the agent will pick between `iptables-legacy` and
  `iptables-nft` at runtime,
  based on kernel support and existing mesh rules.

  When the field is set, the agent will always use either `iptables-legacy` or
  `iptables-nft`.
- The agent now eagerly detects HTTP in incoming connections.


## 3.149.0 - 2025-07-11


### Added

- mirrord-intproxy now propagates critical errors to the mirrord-layer,
  solving the issue where the user application was terminating with a very
  generic
  `Proxy error, connectivity issue or a bug` error message.
- Added MirrordClusterSession CRD initial implementation, currently hidden
  behind "experimental" feature for mirrord-operator crate.
- Added browser extension configuration and initiation.


### Changed

- Change the SIP patch dir to be nested in the same folder as the extracted
  layer (`TMPDIR/mirrord`).
- Do not check for pod running or if the deployment has replicas when using
  copy-target.
- Clearer error when iptables are dirty.


### Fixed

- Fixed an issue where mirrord was failing to spawn an ephemeral agent due to
  `resourceVersion` conflict.


## 3.148.0 - 2025-07-01


### Added

- Added a warning that notifies about the possibility of losing
  requests unmatched requests when both HTTP filter and copy target
  are used.
- Added E2BIG error handling in the mirrord CLI. mirrord CLI
  now shows a more informative error.


### Changed

- Made the documentation on outgoing traffic configuration clearer.


### Fixed

- Fixed an issue where the copy target feature was failing with a request
  timeout.


## 3.147.0 - 2025-06-26


### Changed

- mirrord exec config_file with no extension, e.g. heredoc, now assumed to be
  of json format
- mirrord now extracts layer to temp_dir()/mirrord to allow easier whitelisting
  with Carbon Black
- Expand current profile config doc to address the new namespaced profile
  feature.


## 3.146.0 - 2025-06-24


### Added

- Added a new mirrord config `agent.priority_class` field for specifying a priority class name
  for targetless agent pods.
- Added a new `mirrord dump -p <PORT> -t <TARGET>` command. The command allows for getting dump
  of target's incoming traffic.


### Changed

- Changed `container.override_host_ip` config to use Docker's internal address by
  default when running `mirrord container docker` (also changes
  `external_proxy.host_ip` to 0.0.0.0).
- Removed Discord links.

## 3.145.0 - 2025-06-17


### Added

- Introduced namespaced mirrord profile.
- Pass the user's git branch to the operator to allow integration with mirrord
  Jira app.


### Fixed

- Fix duplicated http_filter_ports and not having the default 80,8080.
- Optimized mirrord-agent's memory usage when mirroring incoming traffic.


## 3.144.0 - 2025-06-10


### Added

- Add user-agent to version check
- Added an option to exclude agent's communication port from sidecar proxy if
  the target is in a service mesh.


### Changed

- Automatically add health probe ports to http_filter ports (if a filter is
  set).


### Fixed

- Fixed big packets crashing mirrord agent - skip and warn, allow user to
  override max packet size


## 3.143.0 - 2025-05-29


### Added

- Added `fallbackName` to the `MirrordWorkloadQueueRegistry` CRD.
- Added support for SQS splitting without copy target
  if the feature is enabled in the operator.


### Changed

- Updated `mirrord.dev` links in docs.
- When `skip_sip` is not set in user's configuration, use a default list that
  currently only contains `git`.


### Fixed

- Fixed an issue where copy target sessions could fail to reuse an existing
  copy.


## 3.142.2 - 2025-05-22


### Added

- Added a link to Slack to error messages and documentation.


### Changed

- When mirrord sends an error response to a stolen request, the request body
  now indicates whether the error comes from the agent or from the internal
  proxy.


### Fixed

- Added `experimental.ignore_system_proxy_config` flag to disable any system
  proxy from affecting the running app.


## 3.142.1 - 2025-05-16


### Changed

- mirrord-agent is now less strict when parsing intercepted HTTP/1 requests.


### Fixed

- Fixed the logic for dirty iptables detection and cleanup in the
  mirrord-agent.

## 3.142.0 - 2025-05-12


### Added

- Add new `container.override_host_ip` config key to override what address will
  be used as host addr inside the container.
- Add the `-t` argument to the `mirrord ls` command to list targets of a
  specific type. Also allow target types to be read from env.
- Added support for skipping custom build tools via environment variable and
  `skip_build_tools` in config.


### Changed

- Renamed `MirorrdProfile` to `MirrordClusterProfile` and support both.
- Increase timeout of outgoing.rs tests to (hopefully) reduce flakiness.
- In iptables guard, skip iptables cleanup if the child agent process has
  already done so.
- Improved HTTP detection logic in the mirrord agent.
- Add watch permission for mirrord policies (so the operator can use a
  reflector and cache policies).


## 3.141.0 - 2025-04-28


### Added

- Added the option to skip sip patching.
- Extended the `MirrordKafkaTopicsConsumer` CRD with a `split_ttl` field.


### Changed

- Increased the maximum allowed size of `config.feature.fs.readonly_file_buffer`
  to 15 MB. Added a warning when using size over 1 MB.


### Fixed

- Tied SIP patch files to the version of mirrord binary, so that fixes to patching
  logic will create new files.
- Fixed docs on how to specify multiple binaries.
- Fixed logic for detecting whether the operator supports Kafka splitting
  without copying the target.


## 3.140.0 - 2025-04-22


### Added

- New config value to allow override of listen ip for external proxy
  (`external_proxy.host_ip`).


### Changed

- The IP table chain names used by the agent are no longer randomized. The
  agent detects if another agent is already
  running or if a previous cleanup failed.
- Corrected agent pod creation progress message.


### Fixed

- Fix issue with port-forward feature that prevented more than one open
  connection per socket addr.
- Use ss for killing existing connections to allow stealing to begin


## 3.139.1 - 2025-04-17


### Fixed

- Fixed an issue where mirrord-agent was being OOM killed due to a traffic
  redirection loop.


## 3.139.0 - 2025-04-16


### Added

- Added config file option for `mirrord operator session` commands (`-f`).


### Changed

- Increased number of OS threads used by the mirrord agent.

## 3.138.0 - 2025-04-15


### Added

- Added `applies_to_copy_targets` to the mirrord Policy specs, so we can enable
  policies when using copied targets.
- Added a sanity e2e test for the copy target feature.
- Added CLI support for Kafka splitting without copying the target.
- The agent now issues warnings to clients that do not meet minimum
  protocol version requirement for stealing an HTTP request.


### Changed

- Improved mermaid diagrams readability (changed colors and lines).
- Improved mirrord progress message from spawning the agent without an
  operator.
- By default, the `DOTNET_STARTUP_HOOKS` env var will not be fetched from the
  target.
- Updated contributing guidance.
- Updated dev container base image and JSON configuration file.


### Fixed

- mirrord now correctly detects whether the configuration allows for stealing
  health checks from the targeted pod.
- Fixed `mirrord_agent_dns_request_count` Prometheus metric.


## 3.137.0 - 2025-03-26


### Added

- mirrord prolicies now allow for enforcing usage of mirrord profiles.


### Changed

- Moved the `readonly_file_buffer` configuration option from experimental to
  `config.feature.fs`.
- Allow ping-pong an extra timeout period if intproxy receives other messages
  from the agent in last period.


### Fixed

- Added a limit on the size of `config.feature.fs.readonly_file_buffer` to a maximum of 1 MB
  to avoid EIO errors.
- Fixed a bug related to stealing IPv6 traffic (resolving original destination
  of a stolen connection).
- Fixed an issue where mirrord was preventing the local application from making
  gRPC connections to sidecar containers.


## 3.136.0 - 2025-03-21


### Added

- mirrord CLI now prints the path to the internal proxy logfile.
- Added support for new CRD - MirrordProfile. mirrord profiles allow for
  storing mirrord config templates in the cluster and applying them to the
  mirrord config at runtime.


### Fixed

- Regression in running some SIP-protected binaries with mirrord.
- Regression in running SIP-protected binaries that have entitelements.


## 3.135.0 - 2025-03-18


### Added

- mirrord now issues a warning when the user's config allows for
  stealing health checks from the target.


### Changed

- `502 Bad Gateway` responses returned by the mirrord-agent now contain the
  source error.


### Fixed

- Added missing `PodTemplate` permissions to the `ClusterRole` produced by
  `mirrord operator setup`.
- Fixed a bug where mirrord was producing a malformed `credentials` file.
- Fixed a bug where mirrord was unable to target Argo Rollouts with both
  `workloadRef`s and `selector`s.


## 3.134.2 - 2025-03-06


### Added

- If a stolen HTTP request matches filters of multiple users,
  the users who don't get the request are now notified with a log message.


### Changed

- Improved the `mirrord_agent_http_request_in_progress_count` metric.


### Fixed

- Fixed `unlink` and `unlinkat` logic for files vs. directories.
- Fixed an bug in TCP mirroring feature.
- Fixed an error where mirrord would sometimes fail with `NotImplemented` error
  due to latency on agent/operator connection.
- Fixed an issue where mirrord was unable to perform an HTTP/1 upgrade over a
  local TLS connection.
- Improved remote DNS errors returned to the client application from the
  mirrord-agent.


## 3.134.1 - 2025-02-28


### Fixed

- Fixed mirrord failing to load when running emulated in an x86 shell by using
  code shim in builds for arm64 *and* x86.
- Fix go -> execve sip use cases, i.e air or go reloaders
- Fixed mirrord Operator's `ClusterRole` generated by the `mirrord operator
  setup` command.


## 3.134.0 - 2025-02-24


### Added

- Added support for stealing HTTPS requests with a filter (requires mirrord
  Operator).
- Added Nix installation instructions to the README.


### Fixed

- Fixed an issue where stealing a remote port was preventing the application
  from making TCP connections with the same destination port number.
- Fixed the order of path checks/operations in file ops handlers in the mirrord
  layer.
- Fixed an issue where mirrord was sometimes unable to steal traffic from more
  than one port with an HTTP filter.


## 3.133.1 - 2025-02-19


### Fixed

- Added a reconnection mechanism when using mirrord operator.
- Fixed issues with rollout targets without a `selector` field present.
- Look for the correct pid that matches the targets container_id (by searching
  /proc/pid/cgroup).
- Prevent reading a remote directory from producing an 'unexpected response'
  error and crashing.
- Fixed a remote DNS regression introduced when `hickory-resolver` and
  `hickory-proto` versions were bumped.
- mirrord CLI now correctly emits logs when enabled with `RUST_LOG` environment
  variable.


## 3.133.0 - 2025-02-17


### Added

- Added an option to configure timeout for idle local HTTP connections
  (`experimental.idle_local_http_connection_timeout`).


### Changed

- Improved the warning produced when the user specifies agent namespace for a
  targetless run.


### Fixed

- Correct statfs data for Go.
- Updated `hickory-resolver` and `hickory-proto` to `0.25.0-alpha.5` and `rand`
  from `0.8` to `0.9`.
- Respect ignored paths and path mapping in statfs hook.
- Some FS libc calls could be carried out remotely instead of locally in some
  cases.
- `mirrord ls` command now does not list unnecessary target types when called
  from a plugin/extension.
- Fixed wrong link for ipv6 config docs.


## 3.132.1 - 2025-02-06


### Fixed

- Fixed operator connect URL produced by the CLI when a target container is
  specified.

## 3.132.0 - 2025-02-06


### Removed

- Removed faulty `statfs` hook for Go applications.


### Added

- Added Kubernetes ReplicaSet as a new target type (requires mirrord Operator).


### Changed

- Namespace for `targetless` runs is now specified with the
  `target.namespace` config field (or the `MIRRORD_TARGET_NAMESPACE`
  environment variable).
  `agent.namespace` field is ignored in targetless runs.


## 3.131.2 - 2025-01-29


### Fixed

- Removed an error log on a `notimplemented` response from the `mirrord-intproxy`,
  might fix go crash.


## 3.131.1 - 2025-01-28


### Changed

- mirrord commands now accept the `-f`/`--config-file` argument without the value as well.
  When no value is provided, `./.mirrord/mirrord.json` is assumed.


### Fixed

- Added ping pong subtask to mirrord-extproxy to keep agent connection alive while it is
  up.
- `agent.privileged` no longer affects targetless agents' pods.

## 3.131.0 - 2025-01-27


### Added

- `statfs` support
- Support for in-cluster DNS resolution of IPv6 addresses.
- Prometheus metrics to the mirrord-agent.
- Kubernetes Service as a new type of mirrord target (requires mirrord
  operator).


### Fixed

- Misleading doc for `.target.namespace` config.
- Agent now correctly clears incoming port subscriptions of disconnected
  clients.
- mirrord no longer uses the default `{"operator": "Exists"}` tolerations when
  spawning targetless agent pods.

## 3.130.0 - 2025-01-21


### Added

- Added support for `rmdir`, `unlink` and `unlinkat`.


### Changed

- Updated `configuration.md` and improved `.feature.env.mapping` doc.


### Fixed

- Stopped mirrord entering a crash loop when trying to load into some processes
  like VSCode's `watchdog.js` when the user config contained a call to
  `get_env()`, which occurred due to missing env - the config is now only
  rendered once and set into an env var.
- Fixed an issue where HTTP requests stolen with a filter would hang with a
  single-threaded local HTTP server.
  Improved handling of incoming connections on the local machine (e.g
  introduces reuse of local HTTP connections).


## 3.129.0 - 2025-01-14


### Added

- Support for stealing incoming connections that are over IPv6.
- mirrord policy to control file ops from the operator.
- mirrord policy to restrict fetching remote environment variables.


### Changed

- Updated how intproxy is outputting logfile when using container mode, now logs
  will be written on host machine.
- Changed log level for debugger ports detection.
- Readonly file buffering is not enabled by default to improve performance
- Extended docs for HTTP filter in the mirrord config.


### Fixed

- Fixed panic when Go >=1.23.3 verifies pidfd support on Linux.
- Fix misleading agent IO operation error that always mentioned getaddrinfo.
- Fixed a bug where port mirroring block (due to active mirrord policies) would
  terminate the mirrord session.


## 3.128.0 - 2024-12-19


### Added

- Added to mirrord config a new experimental field
  `.experimental.readonly_file_buffer`. If set to a value greater than 0,
  mirrord will fetch remote readonly files in chunks of at least this size (in bytes).
  This is to improve performance with applications that make many small reads
  from remote files.
- Added `mirrord container-ext` command that should be used by extension and
  works similarly to `mirrord ext` but for containers.
- Added runAsNonRoot and RO file system to operator deployment
- Added custom resource definition for cluster-wide mirrord policy -
  `MirrordClusterPolicy`.
- Added mapping option for env vars config, allowing the user to map multiple env
  vars to another value based on regexes.
- Added mkdir support


### Fixed

- Added debugger port detection type for the node `--inspect`, `--inspect-wait`
  and `--inspect-brk` flags.
- Fixed `mirrord operator setup` - added missing `/tmp` volume to the operator
  deployment.


## 3.127.0 - 2024-12-10


### Added

- `MirrordPolicy` can now block traffic mirroring (requires operator support).


### Changed

- Updated dependencies.


### Fixed

- Fixed link to operator docs.


## 3.126.0 - 2024-12-06


### Added

- Added SQS splitting state to mirrord operator status reporting (requires operator support).


### Changed

- Hidden files and directories in `$HOME` directory are now read locally by
  default.


### Fixed

- Can now run cs-installed sbt. We now only need to be able to parse the first
  line of a script, so we now support scripts like that sbt, which starts with
  a normal shebang but then has text in a weird encoding, or maybe non-textual
  data.
- Prevent reverse port forwarding from ending unexpectedly due to
  unexpected connection end.
- Added a sleep and await on it after websocket connection to drive IO runtime
  and prevent websocket closing without handshake.


## 3.125.2 - 2024-11-29


### Fixed

- Manually call `docker start <sidecar_id>` if after our sidecar `run` command
  the container hasn't started yet and is in "created" status.


## 3.125.1 - 2024-11-27


### Fixed

- Added retry of HTTP requests (intproxy) on hyper's `IncompleteMessage` error.


## 3.125.0 - 2024-11-21


### Added

- Added a configuration option that allows for specifying an env file for
  mirrord execution.
- Added notice that fs mapping does not apply to relative paths.

### Changed

- Ignore paths that start with the current dir path, instead of any path that
  contains the current dir path. Also, ignore only paths that end with the
  current exe's path, not all that contain it.
- Print a warning to the user when `-p` is provided as part of `mirrord container`
  run command, as it may cause issues because of our usage of
  container type network mode.


### Fixed

- Change `getifaddrs` hook to allocate memory for a new list instead of modifying
  list returned from libc call.
- Read current dir, current exe, and temp dir locally, also when they contain
  characters with a meaning for regexes, like e.g. parentheses.


## 3.124.2 - 2024-11-08


### Fixed

- Fix agent crash on sniffer failure
- Fix file mapping doesn't affect xstat

## 3.124.1 - 2024-11-07


### Changed

- Bump dependencies


### Fixed

- Fix crash when listing interfaces caused by enabling the new hook by default

## 3.124.0 - 2024-11-06


### Changed

- hide ipv6 interfaces by default


### Fixed

- Make sure agent doesn't send `Close` message when Sniffer fails to load.


## 3.123.0 - 2024-11-05


### Changed

- log better errors of local file creation and add option to use alternative
  way
- add .class to be always local


### Fixed

- use /dev/null by default


## 3.122.1 - 2024-10-30


### Changed

- Bump rust version to 2024-10-11 on macOS [match Linux]


### Fixed

- Add arm64 layer to macOS fat binary.

## 3.122.0 - 2024-10-30


### Added

- Added serviceAccount option to agent config


### Changed

- Allow targetless mode to run with local fs-mode.
- Remove unstable tags on feature.network.outgoing.filter config
- Add option to have logs when running ext commands


### Fixed

- Added experimental disable_reuseaddr to bypass the issue
- `mirrord-intproxy` no longer lingers forever when the user tries to execute a
  non-existent binary.


## 3.121.1 - 2024-10-22


### Changed

- Improve error logging and reporting when a getaddrinfo-adjacent failure
  happens due to IO in the agent.
- Improve error checking for InvalidCertificate errors in mirrord-cli.
- Ignore CATALINA_HOME env by default
- Skip mirrord injections into `bazel-real`, considering it a build tool.


### Fixed

- Fix a bug where file mode was ignored when Go applications were creating
  local files.
- Update mirrord-container sidecar logs command to improve printing of errors.
- Fix SIP detection on scripts with no shebang, SIP of default interpreter is
  now used
- Bump dependencies, fix empty user in a kube context

## 3.121.0 - 2024-10-17


### Added

- Added support for Istio CNI
- Added `nodeSelector` option to agent config.


### Changed

- Allowed filtered steal requests to be retried when we get a Reset from
  hyper(h2).


### Fixed

- Fixed an issue where `mirrord exec ... -- npm run serve` in a Vue project was
  failing with `EAFNOSUPPORT: address family not supported ::1:80`. Added new
  `.experimental.hide_ipv6_interfaces` configuration entry that allows for
  hiding local IPv6 interface addresses from the user application.
- Fixed wrong warning being displayed when binding UDP port 0 and filtering HTTP.
- mirrord now respects `insecure-skip-tls-verify` option set in the kubeconfig
  when `accept_invalid_certificates` is not provided in the mirrord config.


## 3.120.1 - 2024-10-14


### Removed

- Remove support for IPv6 sockets with mirrord.


## 3.120.0 - 2024-10-13


### Added

- Added Kafka splitting feature.


### Changed

- Add analytics about usage of experimental features
- Add option to have logs when running ext commands
- update dependencies


### Fixed

- Fixed a bug where `all_of` and `any_of` HTTP filters were stealing all HTTP
  traffic.
- Handle IPv4 in IPv6, should help with regressions related to allowing
  AF_INET6


## 3.119.1 - 2024-10-09


### Changed

- Allow setting port for int/extproxy from the command line.


### Fixed

- Use new kube rs to support empty user.
- Allow using IPv6 sockets with mirrord.
- Fix mirrord making double bind of port 0 fail.

## 3.119.0 - 2024-10-07


### Added

- Add reverse port forwarding which can be used to proxy data from a remote
  port on the target pod to a local one -
  if only one port is specified, it will be used for both.
  ```
  mirrord port-forward [options] -R [remote_port:]local_port
  ```

  To use the incoming network mode and filters from a config file, use -f as
  normal:
  ```
  mirrord port-forward [options] -R [remote_port:]local_port -f
  config_file.toml
  ```


### Changed

- Dependency tree does not contain tonic 0.11.
- Use forked version of apple-codesign to remove RSA dependency


### Fixed

- Collect and pass environment variables to the process to be executed locally
  instead of setting them for the entire local environment, which was causing
  interference with analytics instrumentation.
- Don't drop RSTs, makes long-lived connections drop on steal start

## 3.118.1 - 2024-10-02


### Added

- Internal proxy now explicitly logs exit error.


### Changed

- Enabled readlink hook by default.
- Prompt user for intproxy logs (when intproxy crashes).
  Adds `.log` as a file type for intproxy default log file.
- Refactor how mirrord gets a target when the operator is enabled, and warn
  when randomly selecting a container in multi-container situations (if the
  user did not specify a container).


### Fixed

- Handle cases where target pod has IPv6


## 3.118.0 - 2024-09-22


### Added

- Add `cli_extra_args` field to `container` config to allow specifying custom
  arguments for `mirrord container` sidecar container.

  ```json
  {
    "container": {
      "cli_extra_args": ["--network", "host"]
    }
  }
  ```
  this config will spawn mirrord cli container with `<runtime> run --network
  host --rm -d ...`.


### Changed

- Increase timeout of layer-intproxy socket connection to a ludicrous amount.
- Have intproxy log to a file in /tmp by default.
- Bump dependencies


### Fixed

- Add a retry for port-forward agent connection if error was received via error
  channel after websocket was established.


## 3.117.0 - 2024-09-12


### Added

- Detect Telepresence's traffic-agent and warn user about incompatibility


## 3.116.3 - 2024-09-05


### Fixed

- Fixed `mirrord ls` hanging when there is a lot of possible targets in the
  cluster.
- Update detour for `dns_configuration_copy` to return remote value from
  "/etc/resolv.conf" to fix nodejs dns resolution not working on macos.


## 3.116.2 - 2024-09-05


### Changed

- Add option to have logs when running ext commands


## 3.116.1 - 2024-09-04


### Fixed

- Fixed upload of mirrord binaries' shasums to homebrew repository in the
  release action.
- Fix mirrord ls hanging by making so `KubeResourceSeeker` will list different
  kinds of resources sequentially instead of in parallel.

## 3.116.0 - 2024-09-03


### Added

- Add initial and very basic implementation of vpn
- Add warning when user tries to mirrord exec [container], pointing them to use
  mirrord container instead.
- Add support for hostname resolution in port-forward.
- Add support for all_of, any_of composite http filters in config.


### Changed

- mirrord now produces a more descriptive error message when it fails to call
  authentication command specified in the kubeconfig.
- SQS CRD field names changed to camelCase.


### Fixed

- Start on deprecating operator target list.


## 3.115.1 - 2024-08-21


### Fixed

- Add retry for checking intproxy logs to get its listening port, Prevents any
  issues when it takes a bit of time for intproxy to start when running in
  container mode.
- Fixed `mirrord-agent` not picking up graceful shutdown signal.

## 3.115.0 - 2024-08-21


### Added

- Adds a batching readdir requests, which should improve the performance when
  traversing large directories. Introduces a new `ReadDirBatched` message to the protocol.


### Fixed

- Fix hooking on arm64 Go on Linux


## 3.114.1 - 2024-08-18


### Fixed

- Make splitqueues optional to support old version


## 3.114.0 - 2024-08-16


### Added

- Add port forwarding feature which can be used to proxy data from a local port
  to a remote one -
  if the local port is not specified, it will default to the same as the remote
  ```
  mirrord port-forward [options] -L [local_port:]remote_ip:remote_port
  ```
- Client side support for the upcoming SQS queue splitting support in *mirrord
  for Teams*.

## 3.113.1 - 2024-08-15


### Fixed

- Fix small error in shared sockets that resulted in it adding the shared
  socket env several times.
- Specify that `mirrord container` is an unstable feature.
- Fix IncomingConfig json schema regression.
- Fix `arm64` version of `mirrord-cli` container image and add github cache for
  container builds.
- Fixed symbol hooks for Go 1.23.


## 3.113.0 - 2024-08-14


### Added

- Add new api to run mirrord inside container

  ```
  mirrord container [options] -- <docker/podman> run ...
  ```

  Because we need to run internal proxy process on the same network as the
  process loaded with `mirrord-layer`, to keep config and kubernetes
  comparability the communication to mirrord agent is made via external proxy
  that will run on the host machine.
  ```
                     ┌────────────────┐
                 k8s │ mirrord agent  │
                     └─────┬────▲─────┘
                           │    │
                           │    │
                     ┌─────▼────┴─────┐
      container host │ external proxy │
                     └─────┬────▲─────┘
                           │    │
                           │    │
                     ┌─────▼────┴─────┐◄──────┐
   sidecar container │ internal proxy │       │
                     └──┬─────────────┴──┐    │
          run container │ mirrord-layer  ├────┘
                        └────────────────┘
  ```


### Fixed

- Add custom handling for istio ambient mode where we set
  `/proc/sys/net/ipv4/conf/all/route_localnet` to `1` so it does require
  `agent.privileged = true` to work. (See
- Fix issue introduced in #2612 that broke configs with one-value definition
  for IncomingConfig for network feature.


## 3.112.1 - 2024-08-05


### Added

- Added `experimental.enable_exec_hooks_linux` switch to the mirrord config.


### Changed

- Change operator port from 3000 to 443 to work without any FW exceptions


### Fixed

- Fixed execve hook (fix data race on process initialization, might fix more stuff)
- Added new VSCode debugpy args layout to debugger port detection


## 3.112.0 - 2024-07-30


### Added

- Add fs mapping, under `feature.fs.mapping` now it's possible to specify regex
  match and replace for paths while running mirrord exec.

  Example:

  ```toml
  [feature.fs.mapping]
  "/var/app/temp" = "/tmp" # Will replace all calls to read/write/scan for
  "/var/app/temp/sample.txt" to "/tmp/sample.txt"
  "/var/app/.cache" = "/workspace/mirrord$0" # Will replace
  "/var/app/.cache/sample.txt" to
  "/workspace/mirrord/var/app/.cache/sample.txt" see
  [Regex::replace](https://docs.rs/regex/latest/regex/struct.Regex.html#method.replace)
  ```
- Warning when mirrord automatically picked one of multiple containers on the
  target.


### Changed

- Allows targeting StatefulSet without the copy_target feature (still requires
  operator though).


### Fixed

- Remove invalid schema doc mentioning podname as a valid pod target selector.
- Pass the list of UserSocket to child processes when exec is called through an
  env var MIRRORD_SHARED_SOCKETS.
- Fixed an issue where operator license was incorrectly recognized as expired
  when it was expiring later the same day.
- Fixed new exec hooks breaking execution of Flask apps.


## 3.111.0 - 2024-07-17


### Added

- Extended `feature.network.dns` config with an optional local/remote filter,
  following `feature.network.outgoing` pattern.


### Fixed

- Update loopback detection to include pod ip's
- Fixed a bug where enabling remote DNS prevented making a local connection
  with telnet.
- Remove automatic ignore of incoming/outgoing traffic for ports 50000-60000


## 3.110.0 - 2024-07-12


### Added

- Added experimental.trust_any_certificate to enable making app trust any
  certificate on macOS


### Fixed

- Fix empty request streaming hanging forever

## 3.109.0 - 2024-07-10


### Changed

- mirrord commands now provide a nicer error message when the operator required
  but not installed.
- Add Unknown target variant for forwards compatibility.


### Fixed

- Improved agent performance when mirroring is under high load.
- Don't include non-running pods in node capacity check
- Add exclusion for DOTNET_EnableDiagnostics to make DotNet debugging work by
  default


## 3.108.0 - 2024-07-02


### Added

- Added support for streaming HTTP responses.


### Changed

- Changed http path filter to include query params in match
- Configuration documentation contents order.
- Errors that occur when using discovery API to detect mirrord operator are no
  longer fatal. When such error is encountered, mirrord command falls back to
  using the OSS version.


### Fixed

- When using mesh use `lo` interface for mirroring traffic.


## 3.107.0 - 2024-06-25


### Added

- Added support for intercepting streaming HTTP requests with an HTTP filter.
- mirrord now queries kube discovery API to confirm that mirrord operator is
  not installed (when an attempt to use operator API fails).


### Fixed

- Fix network interface configuration not propagating to agent


## 3.106.0 - 2024-06-18


### Added

- Add cronjobs and statefulsets(/scale) to operator role setup.
- Allows a CronJob and StatefulSet to be used as a target when copy_target is
  enabled.


### Changed

- Put the copy_target config example in the proper place on the main complete
  config sample.
- Dependencies update


### Fixed

- A few changes to medschool - refactored the code, changed the algorithm
  taking into consideration we don't ever drop fields.
- Kill the intproxy child process when mirrord-cli execvp fails.
- mirrord CLI no longer incorrectly warns the user about soon license
  expiration (renewing licenses).
- Downgrade certificate dependency to avoid loss of support for older
  certificates
- Fix json snippets in configuration docs by escaping backslashes and removing
  trailing commas.
- Fixed crash on missing cwd/exe
- Fixed rustls initialization.


## 3.105.0 - 2024-06-12


### Added

- Add readlink hook (under experimental config).
- Display filtered and unfiltered stolen ports when filter is set.
- When an http filter is set and a port is bound that is not included in the
  filtered ports, and there are no unfiltered ports specified, emit a warning.


### Changed

- Now not accepting configs with the same port in
  `feature.network.incoming.ports` and in
  `feature.network.incoming.http_filter.ports`.


### Fixed

- Fixed SIP issue with Turbo
- Fixed mirrord-agent/cli protocol negotiation


## 3.104.0 - 2024-06-06


### Added

- Emit a warning when the `port_mapping` field of the configuration contains an
  unnecessary mapping of a port to itself.


### Changed

- Update syn to version 2.


### Fixed

- Fix HTTP2/1.1 translated messages dropping
- Clean hostname/name sent to operator to fix issue of hostname with linebreaks
- Fixed a bug where two mirrord sessions could not target the same pod while
  stealing from different ports.
- Fixed typo in auto-generated docs for mirrord config.


## 3.103.0 - 2024-05-29


### Added

- Allows a Job to be used as a target when copy_target is enabled.


### Changed

- Allows the user to set labels and annotations for the agent job and pod via
  agent config.


### Fixed

- mirrord now prints an informative error when the targeted pod is not in
  correct state (e.g. is not `Running` or the target container is not `ready`).
  When picking a pod from target deployment/rollout, mirrord filters out pods
  that are not in correct state.
- Fix config printout error showing repeated messages.
- Fixed listing targets when using operator ignoring namespace - always using
  default
- Fixed missing pods/deployments with more than 1 container when using operator
  ls


## 3.102.0 - 2024-05-22


### Removed

- Remove deprecated unstable pause feature


### Added

- Added json_log config under agent to control whether the agent produces logs
  as normal tracing or json.
- Added config info printout at session start.


### Fixed

- Fixed agent crashing when mirrord target is explicitly set to `targetless`.
- Fixed confusing errors produced when creating an agent.


## 3.101.0 - 2024-05-14


### Changed

- Use operator to list targets to avoid inconsistencies
- Don't print error on permission denied


### Fixed

- Fixed a bug where outgoing connections where not intercepted from bound
  sockets.


## 3.100.1 - 2024-05-06


### Fixed

- mirrord-agent now catches SIGTERM signal and cleans iptables during graceful
  deletion.
- Fixed ping pong logic for intproxy-agent communication. Intproxy now sends
  pings on a fixed schedule, regardless of any other messages.


## 3.100.0 - 2024-05-05


### Added

- Added experimental, temp feature for supporting hazelcast pings
- Provide value hint to Clap for generating shell completions for config_file
  to
  only resolve to files, not just first match.


### Changed

- Changed default env exclude to have `BUNDLE_WITHOUT`
- Append more permissions to operator clusterrole
- Improve tera templating config error to dig into source and give out more
  details.
- env.unset feature is now case insensitive


## 3.99.2 - 2024-04-30


### Fixed

- Fixed case where resolving DNS failed when setting HTTP filter


## 3.99.1 - 2024-04-30


### Changed

- Change agent resolver to only resolve IPv4
- Fallback to OSS when operator license expired


### Fixed

- Fix IntelliJ Rider newest version stuck on macOS
- Fix case were agent log message causes startup failure


## 3.99.0 - 2024-04-28


### Added

- added configuration option to control (local) hostname resolving
- Add ability to configure DNS params for agent (timeout, attempts)


### Changed

- Change ports and http_ports in incoming configuration to be checked upon
  mapped port instead of local
- Read /Network locally by default on macOS


### Fixed

- Fix medschool dropping fields sometimes
- Fix DNS resolving case on macOS + Java Netty


## 3.98.1 - 2024-04-23


### Changed

- Internal proxy now emits plain text instead of ANSI


### Fixed

- don't re-resolve when connecting to loopback on outgoing filter
- Added `JetBrains.Debugger.Worker` to the list of known build tools, fixing
  compatibility with Rider 2024.1.


## 3.98.0 - 2024-04-18


### Added

- Added `create` and `delete` verbs on `pods` resource in
  `clusterrole/mirrord-operator` for operator setup.


### Changed

- Set timeout of dns request to 1s and only attempt once


### Fixed

- Fix memory issue when binding


## 3.97.0 - 2024-04-16


### Added

- Agent now authenticates TLS connections, using a provided X509 certificate
  (mirrord for Teams only).


### Changed

- Changed port stealing configuration:
  1. Added new `ports` field to the `incoming` configuration. The field lists
  ports that should be stolen/mirrored. Other ports remain local.
  2. Changed the way `incoming.http_filter.ports` field is interpreted. Ports
  not listed in this field are not stolen, unless listed in `incoming.ports`.


### Fixed

- Change reqwest to use rustls with native certificates to work in more cases


## 3.96.1 - 2024-04-14


### Changed

- Increase max fd in internal proxy to fix connection limit issues


### Fixed

- Fixed layer making process zombie by calling panic from hookerror, also use
  `sigkill` instead of `sigterm`


## 3.96.0 - 2024-04-09


### Changed

- mirrord now listens on 0.0.0.0 when requested and changes address to
  localhost only when needed.


## 3.95.2 - 2024-04-07

No significant changes.


## 3.95.1 - 2024-04-07


### Fixed

- Allow `target` be a `string` in the JSON Schema
- Fixed excessive stack consumption in the `mirrord-layer` by reducing tracing
  in release profile.


## 3.95.0 - 2024-04-02


### Changed

- mirrord now unsets the env from within the process aswell


## 3.94.0 - 2024-04-01


### Added

- New config `env.unset` that allows user to unset environment variables in the
  executed process.
  This is useful for unsetting env like `HTTP_PROXY`, `AWS_PROFILE` that come
  from the local environment
  and cause undesired behavior (because those aren't needed for deployed apps).


## 3.93.1 - 2024-03-31


### Fixed

- Fix new IDE progress breaking older plugins.
  Three issues fixed:
  1. Show the new progress only when env var is set (to be set in newer IDE
  versions).
  2. Multi pod warning was showing everytime when no operator, not only when
  targeting a deployment + no operator.
  3. Show the message for rollouts as well.


## 3.93.0 - 2024-03-31


### Added

- Added handling HTTP upgrades in filtered connections (`mirrord-agent`).
  Refactored TCP stealer code.
- Add a new diagnostic command to calculate mirrord session latency


### Changed

- Changed `agent.image` config to also accept an extended version where you may
  specify both _registry_ and _tag_ with `agent.image.registry` and
  `agent.image.tag`.
- Proxy errors now don't propagate back to libc but exit with a message
- `use_proxy` behavior is now setting the proxy env to empty value instead of
  unsetting. This should help with cases where
  we need it to propagate to the extensions.


### Fixed

- Internal proxy and agent now properly handle connection shutdowns.
- Fix some open/fd potential issues
- Fixed the display of agent startup errors to the user.
- Fixed timeout set on new internal proxy connection in `fork` detour.


## 3.92.1 - 2024-03-17


### Removed

- Removed problematic DNS cache from internal proxy.


### Fixed

- Fixed a bug with handling hints passed to `getaddrinfo` function.


## 3.92.0 - 2024-03-13


### Added

- Added support for `statx` function.


### Fixed

- Fix incoming network interception via port-forward when "stealing" traffic
  with a mesh like linkerd or istio (Using the same `OUTPUT` iptable rules for
  both meshed and not meshed networks)
- Add Kuma mesh detection and support to mirrord-agent.
- Added sidecar exclusion for kuma mesh, fixing issues running in that setup


## 3.91.0 - 2024-03-05


### Added

- Adds operator session management commands to mirrord-cli, these are: `mirrord
  operator session kill-all`, `mirrord operator session kill --id {id}`, and
  the hidden `mirrrod operator session retain-active`.
- Notify user on license validity.


### Changed

- Adds a new `PolicyRule` for `delete` and `deletecollection` of `sessions` for
  `mirrord operator setup`.
- Change pause feature from unstable to deprecated
- Increased size of buffers used by TCP steal to read incoming streams (from 4k
  to 64k in the agent, from 1k to 64k in the internal proxy).
- Increased size of buffers used by outgoing feature to read streams (from 4k
  to 64k in the agent, from 1k to 64k in the internal proxy).


### Fixed

- Fixed a bug where `gethostbyname` calls where intercepted regardless of the
  remote dns feature status.
- Fixed a bug where non-existent hosts in outgoing filter would prevent the
  application from initiating outgoing connections.
- Remove special handling for DNS when dealing with UDP outgoing sockets
  (manual UDP resolving).


## 3.90.0 - 2024-02-27


### Added

- Add agent configuration to use nftables instead of iptables-legacy to make it
  work in mesh that uses nftables.
- The agent now processes all DNS queries concurrently. Also, client sessions
  in the agent do not block on the DNS queries.


### Changed

- change kubeconfig path expansion to use env as well
- Increase internal proxy timeout from 5 seconds to 10 seconds to fix long
  agent ops


## 3.89.1 - 2024-02-22


### Fixed

- Fixed issue with Golang calling fstat on Linux causing crash


## 3.89.0 - 2024-02-22


### Changed

- Change intproxy log to append
- use_proxy configuration now applies to mirrord operator status, and mirrord
  ls


## 3.88.0 - 2024-02-18


### Added

- Add log level and log destination for int proxy


### Changed

- 1. mirrord CLI now does not check target type when the `copy_target` feature
  is enabled. The check is now done only in the operator.
  2. `mirrord operator setup` not includes permissions to read and change
  rollouts scale.


### Fixed

- Incoming traffic was being mirrord when set to `false`.


## 3.87.0 - 2024-02-15


### Removed

- Remove pause tests as part of deprecation


### Added

- Changed internal proxy to allow for HTTP upgrades with filtered HTTP steal.
- Added support for selecting malfunctioning targets with `copy_target`
  feature.
- Added configuration option `feature.env.load_from_process`, which allows for
  changing the way mirrord loads environment variables from the remote target.


### Fixed

- Add missing permissions needed by operator for copy and scaledown


## 3.86.1 - 2024-02-05


### Fixed

- Added `runAsNonRoot: false` and `runAsUser: 0` to the security context of an
  ephemeral agent when running privileged (to prevent overriding these values
  with values from the pod spec).
- Disabled unix sockets being wrongfully sent to the agent when socket isn't
  connected

## 3.86.0 - 2024-01-29


### Changed

- `JAVA_TOOL_OPTIONS` is excluded by default from the environment variables
  that are fetched from the target.


## 3.85.1 - 2024-01-29


### Fixed

- Running `mirrod exec go run EXECUTABLE` on macOS with go1.21.
- Fixed a compilation bug in `mirrord-operator` crate tests.


## 3.85.0 - 2024-01-24


### Added

- Added license subscription id to operator status CRD. Adjusted
  `CredentialStore` to preserve signing key pair for the same operator license
  subscription id.
- CLI now sends machine host + username to show in mirrord operator status
  (not sent to our cloud!)
- Report port locks and filters in operator status


### Changed

- Change configuration parsing to be strict unallowing unknown fields
- Cluster DNS resolving now happens by nameserver order rather by statistics


### Fixed

- Running R on macOS.
- Running scripts with whitespaces in the shebang.


## 3.84.1 - 2024-01-19


### Fixed

- Add support for shebang containing spaces like asdf's node does


## 3.84.0 - 2024-01-18


### Added

- Report namespace for operator sessions


### Fixed

- add preadv and readv to fix erlang file reading


## 3.83.0 - 2024-01-11


### Changed

- Filesystem: File not found default filter happens after checking local filter
- The `copy_target` feature is now officially stable.
- `mirrord operator status` reports active copy targets.


### Fixed

- Remove guard from dlopen, making calls from within dlopen hookable,
  potentially fixing issues


## 3.82.0 - 2024-01-08


### Removed

- Removed `http_header_filter`. Please use `http_filter` with key
  `header_filter` instead.


### Added

- `mirrord operator setup` defines a `MirrordPolicy` CRD so that admins can
  block certain features by creating policies. When receiving a forbidden error
  from the operator for trying to steal traffic, mirrord shows an error and
  exits.


### Fixed

- Added userextras/oid to mirrord operator role to solve issues in some AKS
  clusters


## 3.81.0 - 2024-01-01


### Changed

- Changed setup to not create self signed, letting operator fallback to it
  automatically on runtime
- Update dependencies


### Fixed

- Fix opendir not being hooked on macOS arm64


## 3.80.0 - 2023-12-27


### Changed

- Remove unstable from ignore localhost


### Fixed

- Allow license key that starts with -
- Fix job lingering by exiting always successfully on agent


## 3.79.2 - 2023-12-24


### Fixed

- Added hook for realpath function, fixing issues on files not found in Java


## 3.79.1 - 2023-12-23


### Fixed

- Fix dirfd crashing on macOS


## 3.79.0 - 2023-12-21


### Removed

- Remove waitlist signup from CLI


### Added

- Added new `teams` command to the CLI.
- Remove support for old cri-o, use new CRI API (v1)


### Fixed

- Uses the `syscalls` crate to handle calling the syscalls for go. And adds
  `pwrite64`, `pread64`, `fsync` and `fdatasync` hooks for go.


## 3.78.2 - 2023-12-14


### Fixed

- Fixed config verification in IDE context when the config does not specify the
  target but uses the `scale_down` feature.


## 3.78.1 - 2023-12-13


### Fixed

- Removed confusing error from `mirrord exec` progress.
- Support binding [::] by resolving to ipv4 unspecified. Fix gRPC Python
  running in Docker


## 3.78.0 - 2023-12-12


### Fixed

- Fix create react apps by adding node related files to default local
- Fixed an issue with internal proxy timing out when the user application
  spawns lengthy build processes.


## 3.77.1 - 2023-12-11


### Fixed

- Fix asdf compatibility by adjusting local files read defaults


## 3.77.0 - 2023-12-07


### Added

- `mirrord verify-config` now outputs a list of available target types.


### Fixed

- Changed `operator` config to be optional. If the option is set to `true`,
  mirrord always uses the operator and aborts in case of failure. If the option
  is set to `false`, mirrord does not attempt to use the operator. If the
  option is not set at all, mirrord attempts to use the operator, but does not
  abort in case it could not be found.
- Fixed config verification in IDE context when `copy_target` feature is used.


## 3.76.0 - 2023-12-04


### Added

- Added support for connecting to the cluster with an HTTP proxy.


### Changed

- Improved incoming config docs


### Fixed

- Improved handling of operator-related errors.


## 3.75.3 - 2023-11-23


### Added

- Added new configuration 'use_proxy' that lets user disable usage of http/s
  proxy by mirrord even when env is set


### Fixed

- Changed the way `targetless` is printed in `mirrord verify-config` to allow
  the IDEs to properly show target selection dialogs.


## 3.75.2 - 2023-11-22


### Fixed

- Fixed issues with mirroring incoming TCP connections when targeting multi-pod
  deployments.


## 3.75.1 - 2023-11-14


### Fixed

- Add a hook for
  [gethostbyname](https://www.man7.org/linux/man-pages/man3/gethostbyname.3.html)
  to allow erlang/elixir to resolve DNS.
- Change spammy connect log's level from info to trace.


## 3.75.0 - 2023-11-08


### Added

- Added 'copy pod' operator feature to the CLI.
- Added option to scale down target deployment when using `copy target`
  feature.


### Fixed

- Don't drop mutex in child on fork_detour, fixes bug with elixir.
- Fixed `port_mapping` feature.
- Local file filter now applies to directory listing [regex] and not just
  underlying files


## 3.74.1 - 2023-10-31


### Fixed

- Support for cluster information in exec plugin (`KUBERNETES_EXEC_INFO`)
- Fixed logging using `mirrord-console`.


## 3.74.0 - 2023-10-27


### Added

- Added source identifier to waitlist register


### Fixed

- `tokio` runtime dropped from layer.


## 3.73.1 - 2023-10-24


### Fixed

- Fixed `KUBERNETES_EXEC_INFO` environment variable passed to `kubectl`
  authentication plugins.


## 3.73.0 - 2023-10-23


### Added

- Added k0s support - add k0s containerd socket path.


### Fixed

- Clarify more about the pre-defined FS exceptions in docs, link to lists.


## 3.72.1 - 2023-10-18


### Fixed

- Added `--mesh` option under `cli::Mode::Ephemeral`, allowing the agent to run
  in a mesh context with an ephemeral target.


## 3.72.0 - 2023-10-12


### Added

- SOCKS5 proxy is now supported.


### Fixed

- Implemented missing hooks for `readdir` and `readdir64`.


## 3.71.2 - 2023-10-10


### Fixed

- Reverted breaking change in CLI for config verify
- Adding some e2e tests for  to protect against breaking changes in the cli.


## 3.71.1 - 2023-10-04


### Fixed

- Adds the optional `--ide` flag to `mirrord verify-config [--ide] --path
  {/config/path}`, turning some errors into warnings (target related).


## 3.71.0 - 2023-10-03


### Added

- Add ability to override container resource requests/limits for job agents via
  `agent.resources` config.


### Fixed

- Propagate to the agent that we're in a mesh context (moved `MeshVendor` to a
  common crate), and handle the special case for `istio`, where the sniffer
  should capture traffic on the `lo` interface.


## 3.70.0 - 2023-09-27


### Added

- Added templating for mirrord config using Tera engine.


### Fixed

- Running `mix` works now (bug was calling `lstat` in `stat` bypass).
- Fix progress message shows wrong latest version


## 3.69.0 - 2023-09-26


### Removed

- Remove spammy messages from progress


### Added

- Added the ability to specify targetless in config file, to allow
  non-interactive targetless in IDEs


### Changed

- Change targetless + steal mode to warning instead of error.
- Changed file filter to exclude jar files from being read remote by default


### Fixed

- Fixes selecting container to use when using operator


## 3.68.0 - 2023-09-19


### Added

- New subcommand for generating shell completions for
  bash/fish/zsh/powershell/elvish


### Fixed

- Fix mirrord-cli verify-config command not serializing failures correctly due
  to serde not being able to serialize newtype pattern in tagged unions.


## 3.67.0 - 2023-09-13


### Added

- Add new command `mirrord verify-config [path]` to the mirrord-cli. It
  verifies a mirrord config file producing a tool friendly output.


### Fixed

- Support mirroring existing sessions by introducing an HTTP check when the
  sniffer receives a tcp packet.


## 3.66.0 - 2023-09-12


### Added

- Added support for pausing ephemeral containers. This feature requires the
  agent to have privileged access.


### Changed

- Add ruby related ENV to the default exclude list


## 3.65.2 - 2023-09-10


### Changed

- Add ruby related ENV to the default exclude list


### Fixed

- Fixed connecting to unspecified ip i.e 0.0.0.0


## 3.65.1 - 2023-09-07


### Changed

- Add some Ruby env to excluded: GEM_HOME, GEM_PATH


### Fixed

- Disable opendir hook on aarch macOS since it crashes due to arm64e issues


## 3.65.0 - 2023-09-06


### Added

- Add hooks for `opendir`, `readdir64_r`, `__lxstat`, `__lxstat64`,
  `__xstat64`. Also contains a refactor of the `*stat` family of hooks (they
  now call a shared function), and `openat64` as its own function.


### Changed

- CLI: Change operator setup to fetch from API version of operator to install
- Don't error on agent ready missing absent, add retry in connection to
  upstream agent


### Fixed

- Fixed agent crashing on flush connections not enabled
- CI: run macOS clippy on all of the codebase
- Fixed pause flakiness by improving our cleanup process [using watch]


## 3.64.2 - 2023-09-05


### Changed

- Changed the targeted and non targeted flows for creating agent.
- Add info about errors to TELEMETRY.md


### Fixed

- Change agent to use current thread runtime, multi thread is enabled by
  mistake.
  * Added a sleep before exiting both in layer and agent to allow
  `tokio::spawn` tasks spawned from `Drop` finish.
  * Changed implementation of pause guard to use `tokio::spawn` - fixes pause
  in combination with above change.
- Fix analytics not being sent on drop


## 3.64.1 - 2023-09-03


### Fixed

- Remove extra message of `{"type":"FinishedTask","name":"mirrord preparing to
  launch","success":true,"message":null}` that causes breakage in extensions.


## 3.64.0 - 2023-09-02


### Added

- Warn when running in a mesh service with mirror mode.


### Fixed

- Fixed detecting operator not found as an error
- Update hyper and hyper-util to fix double select call (and properly handle
  large http traffic).
- Fixed ongoing connections not being stolen by changing our flush mechanism -
  add a rule to drop marked connections and then mark existing connections when
  starting to steal.
- Fixed panic on crash analytics
- Fixed showing error on each file not found triggered by the file not found
  configuration
- Regex meta characters are now escaped in `$HOME` (`not-found` file filter).


## 3.63.0 - 2023-08-28


### Added

- Add the ability to send analytics on errors and not only on successful runs.
- Report back internal proxy error stream to cli


### Changed

- Changed config unstable/deprecations to be aggregated with other config
  warnings


### Fixed

- `not-found` file filter fixed to only match files inside the `$HOME`
  directory.
- Fix openshift detection taking too long by querying a subset instead of all
  APIs


## 3.62.0 - 2023-08-26


### Added

- Add analytics collection to operator session information.
- Added an extra `not-found` file filter to improve experience when using cloud
  services under mirrord.


### Changed

- Update telemetry.md with new info about mirrord for Teams
- Changed keep alive to happen from internal proxy to support cases where layer
  process is stuck [breakpoint/etc]
- Changed CLI progress to print warnings and not only set it as the last
  message of progress
- Changed config verify to return aggregated warnings list for user to print
  instead of warn in current progress - can fix issues with extension where we
  printed to stderr.


### Fixed

- Fix ephemeral agent creation api using agent namespace instead of target.
  Add note about agent namespace being irrelevant in ephemeral.
- Fix macOS SIP potential issues from exec having mirrord loaded into the code
  sign binary.
- Fix operator setup so `MIRRORD_OPERATOR_IMAGE` will function properly.
- Fixed issue connecting to ephemeral container when target is in different
  namespace


## 3.61.0 - 2023-08-22


### Added

- Support DNS resolution for the outgoing filter config.


### Fixed

- Fixed wrong errno being set by mirrord, fixing various flows that rely on
  errno even when return code is ok


## 3.60.0 - 2023-08-21


### Added

- Detect and warn when cluster is openshift
- Add missing hook for open64, fixing certificate loading on C# + Linux
- Small changes relevant to operator for #1782.


### Fixed

- Fixed environment on ephemeral container
  This is done by two things:

  1. There was an issue where we used `self` instead of `1` to obtain env based
  on pid.
  2. We didn't have container runtime to use for fetching, so now we also copy
  env from the original pod spec and set it to ours.


## 3.59.0 - 2023-08-18


### Added

- Add option to run agent container as privileged - `"agent" : {"privileged":
  true}`
  Should help with Bottlerocket or other secured k8s environments.
  Applicable for both job/ephemeral.


### Fixed

- Send only ResponseError::DnsLookup for all errors during DNS lookups


## 3.58.0 - 2023-08-17


### Added

- Introduced hooks for sendmsg and recvmsg, so mongodb+srv protocol (Csharp)
  may resolve DNS (implementation follows previous sendto and recvfrom patch).


### Fixed

- Fixed more complicated scenarios using Go on Linux Arm


## 3.57.2 - 2023-08-16


### Fixed

- Fix crash on forks by leaking HOOK_SENDER
- CLI now uses the json progress tracker as default.


## 3.57.1 - 2023-08-15

No significant changes.


## 3.57.0 - 2023-08-15


### Added

- Add hooks for Go 1.19 >= on Linux Arm
- Add node allocatabillity check to prevent OutOfPods error on agent job.


### Changed

- Incoming config now supports off mode, which is also used when `"incoming":
  "off"`.
  When incoming is off, listen requests go through.
  Changed targetless to warn on listen, since bind can happen on outgoing
  sockets as well.


### Fixed

- Replaced termspin with indicatif, fix multi line issues and refactored
  progress.


## 3.56.1 - 2023-08-09


### Fixed

- Add missing hook for `read$NOCANCEL`, fixes reading remote files in some
  scenarios.


## 3.56.0 - 2023-08-07


### Added

- Added `internal_proxy` timeout configurations to allow users specify timeouts
  in edge cases.


### Changed

- Change operator / cli version mismatch to show only when mirrord is older
  than operator


### Fixed

- Fix grpc errors caused by missing trailers on filtered http responses.


## 3.55.2 - 2023-08-06


### Fixed

- macOS - Running Go build with mirrord while Go binary is SIP protected is
  fixed by enabling file hooks on SIP load mode.


## 3.55.1 - 2023-08-06


### Fixed

- Try to resolve an issue where internal proxy is under heavy load since bash
  scripts does a lot of fork/exec by:
  1. Increasing internal proxy's listen backlog (might not help on macOS)
  2. Change internal proxy to create the upstream (agent) connection in a
  different task, allowing it to keep accepting.
- Fixed detecting skipped processes during layer injection.
- Fix fork issues by changing layer runtime to be current thread


## 3.55.0 - 2023-08-03


### Added

- Add support for selecting Kubeconfig context to use by either using:
  1. Configuration option `kube_context`.
  2. mirrord exec argument `--context`
  3. Environment variable `MIRRORD_KUBE_CONTEXT`


### Changed

- Add userextras/prinicipalid to operators cluster role
- Document behavior of deployment in OSS vs mirrord for Teams
- Skip HashiCorp Vault supporting containers


### Fixed

- Fix fork issue on macOS
- Add access for the operator's cluster role to argoproj rollouts
- Fixed warning on using deployment target with no operator


## 3.54.1 - 2023-08-01


### Fixed

- Sometimes the internal proxy doesn't flush before we do redirection then
  caller can't read port
  leading to "Couldn't get port of internal proxy"


## 3.54.0 - 2023-07-31


### Added

- Added mirrord-operator-user `ClusterRole` to operator setup with RBAC
  permissions to use operator.


### Fixed

- Exclude environment variable COMPLUS_EnableDiagnostics, fixes [mirrord
  intellij #67](https://github.com/metalbear-co/mirrord-intellij/issues/67)


## 3.53.3 - 2023-07-26


### Fixed

- Add `jspawnhelper` to build-tool list & fix skip_processes detection
- Use `shellexpand` to resolve tilde for `kubeconfig`.


## 3.53.2 - 2023-07-24


### Fixed

- Add automatic skip for build-tools `"skip_build_tools": boolean` to config
  [default: True] (build-tool list: `as`, `cc`, `ld`, `go`, `air`, `asm`,
  `cc1`, `cgo`, `dlv`, `gcc`, `git`, `link`, `math`, `cargo`, `hpack`, `rustc`,
  `compile`, `collect2`, `cargo-watch` and `debugserver`)
- Fix `feature.env.override` documentation "overrides" -> "override".
  [mirrord.dev#120](https://github.com/metalbear-co/mirrord.dev/issues/120)
- Specify default value for `agent.tolerations` in docs as json instead of
  yaml.


## 3.53.1 - 2023-07-23


### Changed

- Changed internal proxy to drop stdout/stderr after it finishes loading


## 3.53.0 - 2023-07-20


### Added

- Add support for `agent.tolerations` configuration field for setting agent
  `Toleration`s to work around `Taint`s in the cluster.


### Changed

- Added env var exclusion for `_JAVA_OPTIONS` to avoid loading remote jars or
  settings that wouldn't work locally.


## 3.52.1 - 2023-07-19

No significant changes.


## 3.52.0 - 2023-07-18


### Added

- Support for OIDC refresh token


### Fixed

- Fixed case where proxy can timeout since it holds a stale connection. Added
  heartbeat to the connection handling
- Fixed dynamic pause with operator not working - moved pause request to be
  from internal proxy
- Update the code to reimplement the fix but without moving the pinging source.


## 3.51.1 - 2023-07-16


### Changed

- 'mirrod ls' command now no longer lists crashed pods as targets


## 3.51.0 - 2023-07-16


### Added

- Add outgoing traffic filter feature.

  Adds a way of controlling from where outgoing traffic should go, either
  through the remote pod, or from the local app. Can be configured with the
  `remote` and `local` options under `feature.network.outgoing`.
- mirrord configuration now allows disabling Linux capabilities for the agent
  container.
- Add env to specify operator image


## 3.50.5 - 2023-07-13


### Fixed

- Make sure conntrack flushes the correct port.
- Added `CAP_NET_RAW` Linux capability agent.


## 3.50.4 - 2023-07-11


### Fixed

- Layer now passes detected debugger port down to the child processes.


## 3.50.3 - 2023-07-11


### Fixed

- Fixed double slash in uri path on `pause/unpause` requests in agent
- Fix agent steal crash by adding fallback to using only `PREROUTING` iptable
  chain.


## 3.50.2 - 2023-07-10


### Changed

- Changed agent job definition to tolerate everything


## 3.50.1 - 2023-07-09


### Fixed

- Small fix to operator setup command.


## 3.50.0 - 2023-07-07


### Removed

- Removed error capture error trace feature


### Added

- Add support for Argo rollout target.


### Changed

- Agent container is no longer privileged. Instead, it is given a specific set
  of Linux capabilities: `CAP_NET_ADMIN`, `CAP_SYS_PTRACE`, `CAP_SYS_ADMIN`.
- Changed agent job definition to include limits


### Fixed

- Running java 17.0.6-tem with mirrord.


## 3.49.1 - 2023-07-04


### Changed

- Small optimization in file reads to avoid sending empty data
- Changed internal proxy to close after 5s of inactivity instead of 1
- use Frida's replace_fast in Linux Go hooks


### Fixed

- Child processes of python application would hang after a fork without an
  exec.


## 3.49.0 - 2023-07-03


### Added

- Added new analytics, see TELEMETRY.md for more details.


## 3.48.0 - 2023-06-29


### Added

- Added Deployment to list of targets returned from `mirrord ls`.


### Changed

- Bump rust nightly to 2023-04-19 (latest nightly with support for const std
  traits).
- Change loglevel of warnings to info of logs that were mistakenly warning
- Moved IntelliJ to its own repository and versioning


### Fixed

- Hook send_to and recv_from, leveraging our existing UDP interceptor mechanism
  to manually resolve DNS (as expected by netty, especially relevant for
  macos).
- Add new rule to the OUTPUT chain of iptables in agent to support kubectl
  port-forward
- If the local user application closes a socket but continues running, we now
  also stop mirroring/stealing from the target.
- Add /home and /usr to the default file filter.
- Fixed reporting EADDRINUSE as an error


## 3.47.0 - 2023-06-20


### Added

- Added `listen_ports` to `incoming` config to control what port is actually
  being used locally
  so mirrored/stolen ports can still be accessed locally via those. If port
  provided by `listen_ports`
  isn't available, application will receive `EADDRINUSE`.
  Example configuration:
  ```json
  {
      "feature":
      {
          "incoming": {
              "listen_ports": [[80, 7111]]
          }
      }
  }
  ```
  will make port 80 available on 7111 locally, while stealing/mirroring port
  80.


### Changed

- Changed the logic of choosing local port to use for intercepting mirror/steal
  sockets
  now instead of assigning a random port always, we try to use the original one
  and if we fail we assign random port.
  This only happens if `listen_ports` isn't used.
- The path `/opt` itself is read locally by default (up until now paths inside
  that directory were read locally by default, but not the directory itself).
- Changed back required IntelliJ version to 222+ from 223+
- Moved VSCode extension to its own repository and versioning
  https://github.com/metalbear-co/mirrord-vscode


### Fixed

- Running python with mirrord on apple CPUs.


## 3.46.0 - 2023-06-14


### Added

- Add support for HTTP Path filtering


### Changed

- Refactor vscode-ext code to be more modular


### Fixed

- Fixed bogus warnings in the VS Code extension.
- Mirroring/stealing a port for a second time after the user application closed
  it once.
- fixed using dotnet debugger on VSCode
- Properly detecting and ignoring localhost port used by Rider's debugger.
- fix vscode SIP patch not working


## 3.45.2 - 2023-06-12

No significant changes.


## 3.45.1 - 2023-06-11


### Fixed

- Installation script now does not use `sudo` when not needed. This enbables
  installing mirrord in a `RUN` step in an ubuntu docker container, without
  installing `sudo` in an earlier step.
- fix crio on openshift
- Skipping `gcc` when debugging Go in VS Code extension.


## 3.45.0 - 2023-06-05


### Added

- Rider is now supported by the IntelliJ plugin.


### Fixed

- Changed agent to not return errors on reading from outgoing sockets, and
  layer to not crash in that case anyway


## 3.44.2 - 2023-06-01


### Changed

- Change phrasing on version mismatch warning.
- Add `/Volumes` to default local on macOS
- Change Ping interval from 60s down to 30s.
- Changed local read defaults - list now includes `^/sbin(/|$)` and
  `^/var/run/com.apple`.


### Fixed

- Running postman with mirrord works.
- Return valid error code when dns lookup fails, instead of -1.


## 3.44.1 - 2023-05-26


### Changed

- Never importing `RUST_LOG` environment variable from the cluster, regardless
  of configuration.


### Fixed

- Provide helpful error messages on errors in IDEs.
- Log level control when running targetless.
- Change to sticky balloon on warnings in intelliJ
- Setting the namespace via the configuration was not possible in the IDE
  without also setting a target in the configuration file.
- fixed IntelliJ failing silently when error happened on listing pods


## 3.44.0 - 2023-05-24


### Added

- Changed agent's pause feature. Now the pause is requested dynamically by CLI
  during setup and the agent keeps the target container paused until exit or
  the unpause request was received.
- Added support for NPM run configuration on JetBrains products.


### Changed

- Change mirrord ls to show only pods that are in running state (not
  crashing,starting,etc)
- Change fs mode to be local with overrides when targetless is used
- Make progress text consistently lowercase.


### Fixed

- Fix misalignment on IntelliJ not accepting complex path in target
- Add impersonate permissions for GCP specific RBAC in operator


## 3.43.0 - 2023-05-22


### Added

- Support for targetless execution: when not specifying any target for the
  agent, mirrord now spins up an independent agent. This can be useful e.g. if
  you are just interested in getting the cluster's DNS resolution and outgoing
  connectivity but don't want any pod's incoming traffic or FS.
- Support for targetless mode in IntelliJ based IDEs.
- Support for targetless mode in vscode.


### Changed

- If a user application tries to read paths inside mirrord's temp dir, we hook
  that and read the path outside instead.
- Don't print error if we fail checking for operator


### Fixed

- Added better detection for protected binaries, fixes not loading into Go
  binary
- Disallow binding on the same address:port twice. Solves part of issue 1123.
- Fix the lost update bug with config dropdown for intelliJ
  Fix the lost update bug with config dropdown for intelliJ.
- Fix intelliJ compatibility issue by implementing missing
  createPopupActionGroup


## 3.42.0 - 2023-05-15


### Added

- mirrord config dropdown for intelliJ.
- Log agent version when initializing the agent.


### Changed

- Remove quotes in InvalidTarget' target error message


### Fixed

- Use ProgressManager for mirrord progress on intelliJ
- Fixed `go run` failing because of reading remote files by maing paths under
  /private and /var/folders read locally by default.
- Fix not loading into Go because of SIP by adding into default patched
  binaries


## 3.41.1 - 2023-05-07


### Fixed

- Fixed regression in GoLand and NodeJS causing a crash


## 3.41.0 - 2023-05-06


### Added

- Last selected target is now remembered in IntelliJ extension and shown first
  in the target selection dialog.
- Warn user when their mirrord version doesn't match the operator version.


### Changed

- mirrord loading progress is displayed in the status indicator on IntelliJ,
  replacing the singleton notifier


### Fixed

- Fix crash on unexpected LogMessage
- Added hook for recvfrom to support cases where caller expects the messages to
  be from address they were sent to.


## 3.40.0 - 2023-05-01


### Added

- Add a message informing users of the operator when they impersonate
  deployments with mirrord.
- Last selected target is now remembered in VS Code and shown first in the
  quick pick widget.


### Fixed

- PyCharm plugin now detects `pydevd` debugger and properly excludes its port.
- VS Code extension now detects `debugpy` debugger and properly excludes its
  port.
- Fixed delve patch not working on GoLand macOS when running go tests
- Fixed issues when importing some packages in Python caused by PYTHONPATH to
  be used from the remote pod (add it to exclude)


## 3.39.1 - 2023-04-21


### Changed

- Updated IntelliJ usage gif.


### Fixed

- Add magic fix (by polling send_request) to (connection was not ready) hyper
  error. Also adds some more logs around HTTP stealer.


## 3.39.0 - 2023-04-19


### Added

- Support for Node.js on IntelliJ - run/debug JavaScript scripts on IntelliJ
  with mirrord.


### Fixed

- Use RemoteFile ops in gethostname to not have a local fd.


## 3.38.1 - 2023-04-19


### Fixed

- Release action should work now.

## 3.38.0 - 2023-04-18


### Added

- Add support for cri-o container runtime.
- A descriptive message is now presented in the IntelliJ extension when no
  target is available. Listing targets failure is now handled and an error
  notification is presented.
- Added waitlist registration via cli.
  Join the waitlist to try out first mirrord for Teams which is invite only at
  the moment.
- Add email option to help messages.


### Changed

- When patching for SIP, use arm64 if possible (running on aarch64 and an arm64
  binary is available).
- Changed our Discord invite link to https://discord.gg/metalbear


### Fixed

- Change detour bypass to be more robust, not crashing in case it can't update
  the bypass


## 3.37.0 - 2023-04-14


### Removed

- Removed armv7 builds that were wrongly added


### Added

- Add `ignore_ports` to `incoming` configuration so you can have ports that
  only listen
  locally (mirrord will not steal/mirror those ports).
- Add support for `xstatfs` to prevent unexpected behavior with SQLite.


### Changed

- Improved bad target error


## 3.36.0 - 2023-04-13


### Added

- Notify clients about errors happening in agent's background tasks.
- Add support for the imagePullSecrets parameter on the agent pod. This can be
  specified in the configuration file, under agent.image_pull_secrets.


## 3.35.0 - 2023-04-11


### Added

- Added an error prompt to the VS Code extension when there is no available
  target in the configured namespace.


### Changed

- HTTP traffic stealer now supports HTTP/2 requests.


### Fixed

- Executable field was set to null if present, but no SIP patching was done.
- Fixed random crash in `close_layer_fd` caused by supposed closing of
  stdout/stderr then calling to log that writes to it


## 3.34.0 - 2023-03-30


### Added

- Support for running SIP binaries via the vscode extension, for common
  configuration types.


### Changed

- Add the failed connection address on failure to debug easily
- New IntelliJ icons - feel free to give feedback


### Fixed

- Fix internal proxy receiving signals from terminal targeted for the mirrord
  process/parent process by using setsid
- fix listing pods failing when config file exists on macOS


## 3.33.1 - 2023-03-28


### Changed

- Add default requests and limits values to mirrord-operator setup
  (100m/100Mi).


### Fixed

- Change CLI's version update message to display the correct command when
  mirrord has been installed with homebrew.
- fix using config with WSL on JetBrains
- Fix internal proxy exiting before IntelliJ connects to it in some situations
  (maven). Issue was parent process closing causing child to exit. Fixed by
  waiting from the extension call to the child.
- mirrord-cli: update cli so failing to use operator will fallback to
  no-operator mode.
- Add option to install specific version using the `install.sh` script via
  command line argument or `VERSION` environment variable
- Change connection reset to be a trace message instead of error
- Error when agent exits.


## 3.33.0 - 2023-03-22


### Added

- Support for outgoing unix stream sockets (configurable via config file or
  environment variable).
- Add  version of hooked functions.


### Changed

- add `Hash` trait on `mirrord_operator::license::License` struct
- dependencies bump and cleanup
- fix mirrord loading twice (to build also) and improve error message when no
  pods found


### Fixed

- fix f-stream functions by removing its hooks and add missing underlying libc
  calls
- fix deadlock in go20 test (remove trace?)


## 3.32.3 - 2023-03-19


### Changed

- change outgoing connection drop to be trace instead of error since it's not
  an error


### Fixed

- Support stealing on meshed services with ports specified in
  --skip-inbound-ports on linkerd and itsio equivalent.


## 3.32.2 - 2023-03-14


### Fixed

- fix microk8s support by adding possible containerd socket path
- fix gethostname null termination missing
- Update webbrowser dependency to fix security issue.


## 3.32.1 - 2023-03-12


### Fixed

- fix mirroring not handling big requests - increase buffer size (in rawsocket
  dependency).
  also trace logs to not log the data.
- fix environment regression by mixing the two approaches together.
  priority is proc > oci (via container api)


## 3.32.0 - 2023-03-08


### Changed

- mirrord-layer: changed result of `getsockname` to return requested socket on
  `bind` instead of the detoured socket address
- mirrord-layer: Added `SocketId` to `UserSocket` as a better way of
  identifying sockets, part of #1054.
- CHANGELOG - changed to use towncrier
- Change socket error on reading from outgoing sockets and mirror to be info
  instead of error


### Fixed

- Possible bug when bound address is bypassed and socket stays in `SOCKETS`
  map.


## 3.31.0

### Added

- config: `ignore_localhost` to `outgoing` config for ignoring localhost connections, meaning it will connect to local
  instead of remote localhost.
- config: `ignore_localhost` to `incoming` config for ignoring localhost bound sockets, meaning it will not steal/mirror those.
- combination of `ignore_localhost` in `incoming` and `outgoing` can be useful when you run complex processes that does
  IPC over localhost.
- `sip_binaries` to config file to allow specifying SIP-protected binaries that needs to be patched
  when mirrord doesn't detect those. See.

### Fixed

- Unnecessary error logs when running a script that uses `env` in its shebang.
- VSCode extension: running Python script with debugger fails because it tries to connect to the debugger port remotely.
- Big file leading to timeout: we found out that `bincode` doesn't do so well with large chunked messages
  so we limited remote read size to 1 megabyte, and read operation supports getting partial data until EOF.
- mirrord-operator: fix silent fail when parsing websocket messages fails.

### Changed

- improved mirrord cli help message.
- mirrord-config: Change `flush_connections` default to `true`, related to.

## 3.30.0

### Added

- mirrord-layer: Added `port_mapping` under `incoming` configuration to allow mapping local ports to custom
  remote port, for example you can listen on port 9999 locally and it will steal/mirror
  the remote 80 port if `port_mapping: [[9999, 80]]`. See

### Fixed

- Fix issue when two (or more) containerd sockets exist and we use the wrong one. Fixes.
- Invalid toml in environment variables configuration examples.

### Changed

- Use container's runtime env instead of reading it from `/proc/{container_root_pid}/environ` as some processes (such as nginx) wipe it. Fixes
- Removed the prefix "test" from all test names -.
- Created symbolic link from the vscode directory to the `LICENSE` and `CHANGELOG.md` files so that mirrord developers
  don't need to copy them there before building the app.
- mirrord-layer: `socket` hook will now block ipv6 requests and will return EAFNOSUPPORT. See.

## 3.29.0

### Added

- mirrord debug feature (for mirrord developers to debug mirrord): Cause the agent to exit early with an error.
- mirrord E2E tests: support for custom namespaces.

### Fixed

- Unpause the target container before exiting if the agent exits early on an error and the container is paused -.
- intellij-plugin: fix issue where execution hangs when running using Gradle. Fixes.
- intellij-plugin: fix issue where mirrord doesn't load into gradle, was found when fixing.
- mirrord-agent: reintroduce `-o lo` back to iptable rules to prevent issue where outinging messages could be intersepted by mirrord as incoming ones.
- mirrord-layer: binding same port on different IPs leads to a crash due to `ListenAlreadyExists` error.
  This is now ignored with a `info` message since we can't know if the IP/Port was already bound
  or not. Created a follow up issue to complete implementation and error at application's bind.

## 3.28.4

### Fixed

- VSCode Extension: Fix wrong CLI path on Linux

## 3.28.3

### Fixed

- VSCode Extension: Fix wrong CLI path

## 3.28.2

### Fixed

- Fix error in VSCode extension compilation

## 3.28.1

### Fixed

- CI: fix error caused by missing dir

## 3.28.0

### Changed

- Change VSCode extension to package all binaries and select the correct one based on the platform. Fixes.
- agent: add log to error when handling a client message fails.

### Fixed

- agent: Make sniffer optional to support cases when it's not available and mirroring is not required.

## 3.27.1

### Changed

- Update operator version

## 3.27.0

### Fixed

- mirrord now handles it when the local app closes a forwarded stolen tcp connection instead of exiting with an error. Potential fix for.
- missing kubeconfig doesn't fail extensions (it failed because it first tried to resolve the default then used custom one)

### Changed

- layer: Don't print error when tcp socket faces error as it can be a normal flow.
- internal proxy - set different timeout for `mirrord exec` and running from extension
  fixing race conditions when running from IntelliJ/VSCode.
- Changed `with_span_events` from `FmtSpan::Active` to `FmtSpan::NEW | FmtSpan::CLOSE`.
  Practically this means we will have less logs on enter/exit to span and only when it's first created
  and when it's closed.
- JetBrains Plugin: Add debug logs for investigating user issues.
- JetBrains compatibility: set limit from 222 (2022.2.4) since 221 isn't supported by us.
- Make `kubeconfig` setting effective always by using `-f` in `mirrord ls`.
- mirrord agent can now run without sniffer, will not be able to mirror but can still steal.
  this is to enable users who have older kernel (4.20>=) to use the steal feature.

## 3.26.1

### Fixed

- VSCode Extension: Prevent double prompting of the user to select the target if not specified in config. See.

### Changed

- JetBrains enable support from 2021.3 (like we had in 3.14.3).

## 3.26.0

### Changed

- mirrord-agent: localhost traffic (like healthprobes) won't be stolen by mirrord on meshed targets to align behavior with non meshed targets. See [#1070](https://github.com/metalbear-co/mirrord/pull/1070)
- Filter out agent pods from `mirrord ls`, for better IDE UX. Closes.
- Not exiting on SIP-check fail. Instead, logging an error and letting the program fail as it would without mirrord.
  See.

### Fixed

- Fix cache does not work on test-agent workflow. See.
- CI: merge queue + branch protection issues

## 3.25.0

### Added

- `gethostname` detour that returns contents of `/etc/hostname` from target pod. See relevant.

### Fixed

- `getsockname` now returns the **remote** local address of the socket, instead of the
  **local fake** address of the socket.
  This should fix issues with Akka or other software that checks the local address and
  expects it to match the **local ip of the pod**.
  This breaks agent protocol (agent/layer need to match).
- GoLand debug fails because of reading `/private/var/folders` remotely (trying to access self file?). fixed with filter change (see below)

### Changed

- VSCode extension: update dialog message
- JetBrains: can now change focus from search field to targets using tab/shift+tab (for backward)
- Refactor - mirrord cli now spawns `internal proxy` which does the Kubernetes operations for
  the layer, so layer need not interact with k8s (solves issues with remote/local env mix)
- filter: add `/private/var/folders" to default local read override
- filter: fixed regex for `/tmp` default local read override
- disable flask e2e until we solve the glibc issue (probably fstream issue)

## 3.24.0

### Added

- Add a field to mirrord-config to specify custom path for kubeconfig , resolves.

### Changed

- Removed limit on future builds `untilBuild` in JetBrains plugin.
- IntelliJ-ext: change the dialog to provide a sorted list and make it searchable, resolves.
- mirrord-layer: Changed default to read AWS credentials + config from remote pod.
- Removed unused env var (`MIRRORD_EXTERNAL_ENV`)
- mirrord-agent: Use `conntrack` to flush stealer connections (temporary fix for.

### Fixed

- Added env guard to be used in cli + extension to prevent (self) misconfigurations (our kube settings being used from remote).

## 3.23.0

### Fixed

- mirrord-config: Fix disabled feature for env in config file, `env = false` should work. See.
- VS Code extension: release universal extension as a fallback for Windows and other platforms to be used with WSL/Remote development. Fixes
- Fix `MIRRORD_AGENT_RUST_LOG` can't be more than info due to dependency on info log.
- Fix pause feature not working in extension due to writing to stdout (changed to use trace)

### Changed

- `DNSLookup` failures changed to be info log from error since it is a common case.
- mirrord-agent: now prints "agent ready" instead of logging it so it can't be fudged with `RUST_LOG` control.
- mirrord-agent: `agent::layer_recv` changed instrumentation to be trace instead of info.
- mirrord-layer/agent: change ttl of job to be 1 second for cases where 0 means in cluster don't clean up.
- Convert go fileops e2e tests into integration tests. Part of.

## 3.22.0

### Changed

- Rust: update rust toolchain (and agent rust `DOCKERFILE`) to `nightly-2023-01-31`.
- exec/spawn detour refactor.
- mirrord-layer: Partially load mirrord on certain processes that spawn other processes to allow sip patch on the spawned process.
  This to prevent breaking mirrord-layer load if parent process is specified in `--skip-processes`.  (macOS only)

### Fixed

- mirrord-layer: DNS resolving doesn't work when having a non-OS resolver (using UDP sockets)
  since `/etc/resolv.conf` and `/etc/hosts` were in the local read override,
  leading to use the local nameserver for resolving. Fixes
- mirrord-agent: Infinite reading a file when using `fgets`/`read_line` due to bug seeking to start of file.
- Rare deadlock on file close that caused the e2e file-ops test to sometimes fail
  (.

## 3.21.0

### Added

- Support for Go's `os.ReadDir` on Linux (by hooking the `getdents64` syscall). Part of.
- Test mirrord with Go 1.20rc3.

### Changed

- mirrord-agent: Wrap agent with a parent process to doublecheck the clearing of iptables. See
- mirrord-layer: Change `HOOK_SENDER` from `Option` to `OnceLock`.

### Fixed

- mirrord-agent: Handle HTTP upgrade requests when the stealer feature is enabled
  (with HTTP traffic) PR [#973](https://github.com/metalbear-co/mirrord/pull/973).
- E2E tests compile on MacOS.
- mirrord could not load into some newer binaries of node -. Now hooking also `posix_spawn`, since node now uses
  `libuv`'s `uv_spawn` (which in turn calls `posix_spawn`) instead of libc's `execvp` (which calls `execve`).
- Read files from the temp dir (defined by the system's `TMPDIR`) locally, closes.

## 3.20.0

### Added

- Support impersonation in operator

### Fixed

- Go crash in some scenarios.
- Remove already deprecated `--no-fs` and `--rw` options, that do not do anything anymore, but were still listed in the
  help message.
- Bug: SIP would fail the second time to run scripts for which the user does not have write permissions.

### Changed

- Change layer/cli logs to be to stderr instead of stdout to avoid mixing with the output of the application. Closes

## 3.19.2

### Changed

- Code refactor: moved all file request and response types into own file.

## 3.19.1

### Fixed

- Changelog error failing the JetBrains release.

## 3.19.0

### Changed

- mirrord-operator: replace operator api to use KubernetesAPI extension. [#915](https://github.com/metalbear-co/mirrord/pull/915)

### Fixed

- tests: flaky passthrough fix. Avoid 2 agents running at the same time, add minimal sleep (1s)
- macOS x64/SIP(arm): fix double hooking `fstatat$INODE64`. Possible crash and undefined behavior.

### Added

- introduce `mirrord-console` - a utility to debug and investigate mirrord issues.

### Deprecated

- Remove old fs mode
  - cli: no `--rw` or `--no-fs`.
  - layer: no `MIRRORD_FILE_OPS`/`MIRRORD_FILE_RO_OPS`/`MIRRORD_FILE_FILTER_INCLUDE`/`MIRRORD_FILE_FILTER_EXCLUDE`

## 3.18.2

### Fixed

- crash when `getaddrinfo` is bypassed and libc tries to free our structure. Closes
- Stealer hangs on short streams left open and fails on short closed streams to filtered HTTP ports -.

## 3.18.1

### Fixed

- Issue when connect returns `libc::EINTR` or `libc::EINPROGRESS` causing outgoing connections to fail.
- config: file config updated to fix simple pattern of IncomingConfig. [#933](https://github.com/metalbear-co/mirrord/pull/933)

## 3.18.0

### Added

- Agent now sends error encountered back to layer for better UX when bad times happen. (This only applies to error happening on connection-level).
- Partial ls flow for Go on macOS (implemented `fdopendir` and `readdir_r`). Closes
- New feature: HTTP traffic filter!
  - Allows the user to steal HTTP traffic based on HTTP request headers, for example `Client: me` would steal requests that match this header,
    while letting unmatched requests (and non-HTTP packets) through to their original destinations.

### Fixed

- Update the setup-qemu-action action to remove a deprecation warning in the Release Workflow
- stat functions now support directories.
- Possible bugs with fds being closed before time (we now handle dup'ing of fds, and hold those as ref counts)

### Changed

- agent: Return better error message when failing to use `PACKET_IGNORE_OUTGOING` flag.

## 3.17.0

### Added

- Add brew command to README

### Fixed

- intellij plugin: mirrord icon should always load now.
- intellij plugin: on target selection cancel, don't show error - just disable mirrord for the run and show message.
- fixed setting a breakpoint in GoLand on simple app hanging on release build (disabled lto). - Fixes.

### Deprecated

- Removed `disabled` in favor of `local` in `fs` configuration.

### Changed

- update `kube` dependency + bump other
- update `dlv` packed with plugins.

## 3.16.2

### Fixed

- Add go to skipped processes in JetBrains plugin. Solving GoLand bug.

## 3.16.1

### Fixed

- Running on specific Kubernetes setups, such as Docker for Desktop should work again.

## 3.16.0

### Added

- Add golang stat hooks, closes

### Fixed

- agent: mount /var from host and reconfigure docker socket to /var/run/docker.sock for better compatibility
- Error on specifying namespace in configuration without path (pod/container/deployment). Closes
- IntelliJ plugin with new UI enabled now shows buttons. Closes
- Fix deprecation warnings (partially), update checkout action to version 3.

### Changed

- Refactored detours to use new helper function `Result::as_hook` to simplify flow. (no change in behavior)

## 3.15.2

### Added

- Logging for IntelliJ plugin for debugging/bug reports.

### Fixed

- Crash when mirroring and state is different between local and remote (happens in Mesh).
  We now ignore messages that are not in the expected state. (as we can't do anything about it).
- agent: Fix typo in socket path for k3s environments
- intellij-plugin: fix missing telemetry/version check

## 3.15.1

### Added

- Add `__xstat` hook, fixes

### Fixed

- Fix build scripts for the refactored IntelliJ plugin

## 3.15.0

### Added

- agent: Add support for k3s envs
- IntelliJ plugin - refactor, uses cli like vs code.

### Fixed

- getaddrinfo: if node is NULL just bypass, as it's just for choosing ip/port, Fixes

### Changed

- cli now loads env, removes go env stuff at load, might fix some bugs there.

## 3.14.3

### Fixed

- Create empty release to overcome temporary issue with VS Code marketplace publication

## 3.14.2

### Fixed

- vscode ext: use process env for running mirrord. Fixes

## 3.14.1

### Fixed

- layer + go - connect didn't intercept sometimes (we lacked a match). Fixes [851](https://github.com/metalbear-co/mirrord/issues/851).

## 3.14.0

### Changed

- cli: Set environment variables from cli to spawned process instead of layer when using `mirrord exec`.
- cli: use miette for nicer errors
- cli: some ext exec preparations, nothing user facing yet.
- vs code ext: use cli, fixes some env bugs with go and better user experience.

## 3.13.5

### Changed

- Don't add temp prefix when using `extract` command.
- VS Code extension: mirrord enable/disable to be per workspace.
- VS Code extension: bundle the resources
- Add `/System` to default ignore list.
- Remove `test_mirrord_layer` from CI as it's covered in integration testing.

### Fixed

- fd leak on Linux when using libuv (Node). Caused undefined behavior. Fixes.

### Misc

- Better separation in mirrord cli.

## 3.13.4

### Changed

- Adjust filters - all directory filters also filter the directory itself (for when lstat/stating the directory).
  Added `/Applications`

## 3.13.3

### Added

- Add `mirrord ls` which allows listing target path. Hidden from user at the moment, as for now it's meant for extension use only.

### Changed

- Refactor e2e tests: split into modules based on functionality they test.
- internal refactor in mirrord-agent: Stealer feature changed from working per connection to now starting with
  the agent itself ("global"). Got rid of `steal_worker` in favor of a similar abstraction to what
  we have in `sniffer.rs` (`TcpConnectionStealer` that acts as the traffic stealing task, and
  `TcpStealerApi` which bridges the communication between the agent and the stealer task).
- Tests CI: don't wait for integration tests to start testing E2E tests.

### Fixed

- Add missing `fstat`/`lstat`/`fstatat`/`stat` hooks.

## 3.13.2

### Fixed

- Weird crash that started happening after Frida upgrade on macOS M1.

## 3.13.1

### Fixed

- Fix asdf:
  - Add `/tmp` not just `/tmp/` to exclusion.
  - Add `.tool-version` to exclusion.
  - `fclose` was calling close which doesn't flush.

## 3.13.0

### Changed

- IntelliJ Plugin: downgrade Java to version 11.
- IntelliJ Plugin: update platform version to 2022.3.
- Disable progress in mirrord-layer - can cause issues with forks and generally confusing now
  that agent is created by cli (and soon to be created by IDE plugin via cli).
- Update to Frida 16.0.7
- Add more paths to the default ignore list (`/snap` and `*/.asdf/*`) - to fix asdf issues.
- Add `/bin/` to default ignore list - asdf should be okay now!
- Update GitHub action to use latest `rust-cache`

### Added

- mirrord-operator: Add securityContext section for deployment in operator setup

### Fixed

- Fix `--fs-mode=local` didn't disable hooks as it was supposed to.
- Fix hooking wrong libc functions because of lack of module specification - add function to resolve
  module name to hook from (libc on Unix,libsystem on macOS). Partially fixes asdf issue.

## 3.12.1

### Added

- E2E test for pause feature with service that logs http requests and a service that makes requests.
- mirrord-layer: automatic operator discovery and connection if deployed on cluster. (Discovery can be disabled with `MIRRORD_OPERATOR_ENABLE=false`).

### Changed

- Added `/tmp/` to be excluded from file ops by default. Fixes.

### Misc

- Reformatted a bit the file stuff, to make it more readable. We now have `FILE_MODE` instead of `FILE_OPS_*` internally.
- Changed fileops test to also test write override (mirrord mode is read and override specific path)

## 3.12.0

### Added

- `--pause` feature (unstable). See.
- operator setup cli feature.
- mirrord-layer: operator connection that can be used instead of using kubernetes api to access agents.

### Changed

- CI: cancel previous runs of same PR.
- cli: set canonical path for config file to avoid possible issues when child processes change current working directory.
- config: Refactor config proc macro and behavior - we now error if a config value is wrong instead of defaulting.
- layer: panic on error instead of exiting without any message.
- CI: don't run CI on draft PRs.
- Update dependencies.
- Update to clap v4 (cli parser crate).
- Started deprecation of fsmode=disabled, use fsmode=local instead.

### Fixed

- Typo in `--agent-startup-timeout` flag.

## 3.11.2

### Fixed

- Agent dockerfile: fix build for cross arch

### Changed

- Added clippy on macOS and cleaned warnings.

## 3.11.1

### Fixed

- release.yaml: Linux AArch64 for real this time. (embedded so was x64)

### Changed

- Create agent in the cli and pass environment variables to exec'd process to improve agent re-use.
- IntelliJ: change default log level to warning (match cli/vscode).
- IntelliJ: don't show progress (can make some tests/scenarios fail).
- release.yaml: Build layer/cli with Centos 7 compatible glibc (AmazonLinux2 support).
- Change CPU/memory values requested by the Job agent to the lowest values possible.

## 3.11.0

### Added

- MacOS: Support for executing SIP binaries in user applications. We hook `execve`
  and create a SIP-free version of the binary on-the-go and execute that instead of
  the SIP binary.
  This means we now support running bash scripts with mirrord also on MacOS.
  Closes.

### Changed

- Only warn about invalid certificates once per agent.
- Reduce tokio features to needed ones only.

### Fixed

- CI: Fix regex for homebrew formula
- Potentially ignoring write calls (`fd < 2`).
- CI: Fix release for linux aarch64. Fixes.
- Possible cases where we don't close fds correctly.

## 3.10.4

### Fixed

- VS Code Extension: Fix crash when no env vars are defined in launch.json

## 3.10.3

### Changed

- CLI: change temp lib file to only be created for new versions
- mirrord-config: refactored macro so future implementations will be easier

### Fixed

- Release: fix homebrew release step

## 3.10.2

### Fixed

- CI: fix `release_gh` zip file step

## 3.10.1

### Changed

- CI: download shasums and add git username/email to make the homebrew release work
- Remove `unimplemented` for some IO cases, we now return `Unknown` instead. Also added warning logs for these cases to track.
- Only recommend `--accept-invalid-certificates` on connection errors if not already set.
- Terminate user application on connection error instead of only stopping mirrord.

## 3.10.0

### Added

- CI: Update homebrew formula on release, refer

### Changed

- VS Code Extension: change extension to use the target specified in the mirrord config file, if specified, rather than show the pod dropdown

## 3.9.0

### Added

- `MIRRORD_AGENT_NETWORK_INTERFACE` environment variable/file config to let user control which network interface to use. Workaround for.
- mirrord-config: `deprecated` and `unstable` tags to MirrordConfig macro for messaging user when using said fields

### Changed

- VS Code Extension: change extension to use a mirrord-config file for configuration
- VS Code Extension: use the IDE's telemetry settings to determine if telemetry should be enabled

## 3.8.0

### Changed

- mirrord-layer: Remove `unwrap` from initialization functions.
- Log level of operation bypassing log from warn to trace (for real this time).
- Perform filesystem operations for paths in `/home` locally by default (for real this time).

### Added

- VS Code Extension: add JSON schema
- Bypass SIP on MacOS on the executed binary, (also via shebang).
  See [].
  This does not yet include binaries that are executed by the first binary.

### Fixed

- fix markdown job by adding the checkout action

## 3.7.3

### Fixed

- mirrord-agent: No longer resolves to `eth0` by default, now we first try to resolve
  the appropriate network interface, if this fails then we use `eth0` as a last resort.
  Fixes.

### Changed

- intelliJ: use custom delve on macos

## 3.7.2

### Fixed

- Release: fix broken docker build step caused by folder restructure

## 3.7.1

### Fixed

- using gcloud auth for kubernetes. (mistakenly loaded layer into it)
- debugging Go on VSCode. We patch to use our own delivered delve.
- Changed layer not to crash when connection is closed by agent. Closed.

### Changed

- IntelliJ: fallback to using a textfield if listing namespaces fails

## 3.7.0

### Added

- mirrord-config: New `mirrord-schema.json` file that contains docs and types which should help the user write their mirrord
  config files. This file has to be manually generated (there is a test to help you remember).

### Fixed

- IntelliJ: Fix occurring of small namespace selection window and make mirrord dialogs resizable
- IntelliJ: Fix bug when pressing cancel in mirrord dialog and rerunning the application no mirrord window appears again
- VS Code: Fix crash occurring because it used deprecated env vars.

### Changed

- mirrord-config: Take `schema` feature out of feature flag (now it's always on).
- mirrord-config: Add docs for the user config types.

## 3.6.0

### Added

- mirrord-layer: Allow capturing tracing logs to file and print github issue creation link via MIRRORD_CAPTURE_ERROR_TRACE env variable

### Fixed

- Fix vscode artifacts where arm64 package was not released.
- IntelliJ plugin: if namespaces can't be accessed, use the default namespace

### Changed

- Add `/home` to default file exclude list.
- Changed log level of `Bypassing operation...` from warning to trace.
- IntelliJ settings default to match CLI/VSCode.

## 3.5.3

### Fixed

- Fixed broken release step for VS Code Darwin arm64 version

## 3.5.2

### Fixed

- Fixed breaking vscode release step

## 3.5.1

### Fixed

- Fixed an issue with the release CI

### Changed

- Update target file config to have `namespace` nested inside of `target` and not a separate `target_namespace`.
  See

## 3.5.0

### Added

- aarch64 release binaries (no go support yet, no IntelliJ also).
- mirrord-layer: Add [`FileFilter`](mirrord-layer/src/file/filter.rs) that allows the user to include or exclude file paths (with regex support) for file operations.

### Changed

- mirrord-layer: Improve error message when user tries to run a program with args without `--`.
- Add tests for environment variables passed to KubeApi for authentication feature for cli credential fetch
- Remove openssl/libssl dependency, cross compilation is easier now. (It wasn't needed/used)
- mirrord-config: Changed the way [`fs`](mirrord-config/src/fs.rs) works: now it supports 2 modes `Simple` and `Advanced`,
  where `Simple` is similar to the old behavior (enables read-only, read-write, or disable file ops), and `Advanced`
  allows the user to specify include and exclude (regexes) filters for [`FileFilter`](mirrord-layer/src/file/filter.rs).
- Lint `README` and update it for `--target` flag.
- mirrord-layer: improve error message for invalid targets.

### Removed

- `--pod-name`, `--pod-namespace`, `--impersonated_container_name` have been removed in favor of `--target`, `--target-namespace`

### Fixed

- Env var to ignore ports used by a debugger for intelliJ/VSCode, refer

## 3.4.0

### Added

- Add changelog for intelliJ extension, closes
- Add filter for changelog to ci.yml
- Telemetry for intelliJ extension.

### Changed

- Update intelliJ extension: lint & bump java version to 17.
- Added `/Users` and `/Library` to path to ignore for file operations to improve UX on macOS.
- Use same default options as CLI in intelliJ extension.
- Improve UI layout of intelliJ extension.
- Separate tcp and udp outgoing option in intelliJ extension.
- Tighter control of witch environment variables would be passed to the KubeApi when fetching credentials via cli in kube-config. See

### Fixed

- Lint Changelog and fix level of a "Changed" tag.
- File operations - following symlinks now works as expected. Previously, absolute symlinks lead to use our own path instead of target path.
  For example, AWS/K8S uses `/var/run/..` for service account credentials. In many machines, `/var/run` is symlink to `/run`
  so we were using `/run/..` instead of `/proc/{target_pid}/root/run`.
- Fix not reappearing window after pressing cancel-button in intelliJ extension.

## 3.3.0

### Added

- Telemetries, see [TELEMETRY.md](./TELEMETRY.md) for more information.

### Changed

- Added timeout for "waiting for pod to be ready..." in mirrord-layer to prevent unresponsive behavior. See
- IntelliJ Extension: Default log level to `ERROR` from `DEBUG`

### Fixed

- Issue with [bottlerocket](https://github.com/bottlerocket-os/bottlerocket) where they use `/run/dockershim.sock`
  instead of the default containerd path. Add new path as fallback.

## 3.2.0

### Changed

- Extended support for both `-s` and `-x` wildcard matching, now supports `PREFIX_*`, `*_SUFFIX`, etc.
- Add to env default ignore `JAVA_HOME`,`HOMEPATH`,`CLASSPATH`,`JAVA_EXE` as it's usually runtime that you don't want
  from remote. Possibly fixes issue discussed on Discord (used complained that they had to use absolute path and not
  relative).
- Add `jvm.cfg` to default bypass for files.
- Clarify wrong target error message.
- mirrord-layer: Improve error message in `connection::handle_error`.

### Fixed

- Don't ignore passed `--pod-namespace` argument, closes
  []
- Replace deprecated environment variables in IntelliJ plugin
- Issues with IntelliJ extension when debugging Kotlin applications
- Scrollable list for pods and namespaces for IntelliJ extension,
  closes []

### Deprecated

- `--impersonated-container-name` and `MIRRORD_IMPERSONATED_CONTAINER_NAME` are
  deprecated in favor of `--target` or `MIRRORD_IMPERSONATED_TARGET`
- `--pod-namespace` and `MIRRORD_AGENT_IMPERSONATED_POD_NAMESPACE` are deprecated in
  favor of `--target-namespace` and `MIRRORD_TARGET_NAMESPACE`

## 3.1.3

### Changed

- release: VS Code extension release as stable and not pre-release.

### Fixed

- Dev container failing to execute `apt-get install -y clang`

## 3.1.2

### Changed

- Update some texts in documentation, READMEs, and extension package descriptions
- IntelliJ version check on enabling instead of on project start. Don't check again after less than 3 minutes.

## 3.1.1

### Fixed

- IntelliJ plugin crashing on run because both include and exclude were being set for env vars.

## 3.1.0

### Added

- `pwrite` hook (used by `dotnet`);

### Fixed

- Issue. Changed non-error logs from `error!` to `trace!`.

### Changed

- Agent pod definition now has `requests` specifications to avoid being defaulted to high values.
  See.
- Change VSCode extension configuration to have file ops, outgoing traffic, DNS, and environment variables turned on by
  default.
- update intelliJ extension: toggles + panel for include/exclude env vars

## 3.0.22-alpha

### Changed

- Exclude internal configuration fields from generated schema.

### Fixed

- Issue. We now detect NixOS/Devbox usage and add `sh` to
  skipped list.

## 3.0.21-alpha

### Added

- Reuse agent - first process that runs will create the agent and its children will be able to reuse the same one to
  avoid creating many agents.
- Don't print progress for child processes to avoid confusion.
- Skip istio/linkerd-proxy/init container when mirroring a pod without a specific container name.
- Add "linkerd.io/inject": "disabled" annotation to pod created by mirrord to avoid linkerd auto inject.
- mirrord-layer: support `-target deployment/deployment_name/container/container_name` flag to run on a specific
  container.
- `/nix/*` path is now ignored for file operations to support NixOS.
- Shortcut `deploy` for `deployment` in target argument.
- Added the ability to override environment variables in the config file.

### Changed

- Print exit message when terminating application due to an unhandled error in the layer.
- mirrord-layer: refactored `pod_api.rs` to be more maintainble.
- Use kube config namespace by default.
- mirrord-layer: Ignore `EAFNOSUPPORT` error reporting (valid scenario).

## 3.0.20-alpha

### Added

- `pread` hook (used by `dotnet`);
- mirrord-layer: ignore opening self-binary (temporal SDK calculates the hash of the binary, and it fails because it
  happens remotely)
- Layer integration tests with more apps (testing with Go only on MacOS because of
  known crash on Linux - [.
  Closes [].
- Added progress reporting to the CLI.
- CI: use [bors](https://bors.tech/) for merging! woohoo.

### Changed

- Don't report InProgress io error as error (log as info)
- mirrord-layer: Added some `dotnet` files to `IGNORE_FILES` regex set;
- mirrord-layer: Added the `Detour` type for use in the `ops` modules instead of `HookResult`. This type supports
  returning a `Bypass` to avoid manually checking if a hook actually failed or if we should just bypass it;
- mirrord-protocol: Reduce duplicated types around `read` operation;
- Layer integration tests for more apps. Closes
  [].
- Rename http mirroring tests from `integration` to `http_mirroring` since there are
  now also integration tests in other files.
- Delete useless `e2e_macos` CI job.
- Integration tests also display test process output (with mirrord logs) when they
  time out.
- CI: mirrord-layer UT and integration run in same job.
- .devcontainer: Added missing dependencies and also kind for running e2e tests.

### Fixed

- Fix IntelliJ Extension artifact - use glob pattern
- Use LabelSelector instead of app=* to select pods from deployments
- Added another
  protection [to not execute in child processes from k8s auth](https://github.com/metalbear-co/mirrord/issues/531) by
  setting an env flag to avoid loading then removing it after executing the api.

## 3.0.19-alpha

### Added

- Release image for armv7 (Cloud ARM)

### Fixed

- Release for non-amd64 arch failed because of lack of QEMU step in the github action. Re-added it

## 3.0.18-alpha

### Changed

- Replaced `pcap` dependency with our own `rawsocket` to make cross compiling faster and easier.

## 3.0.17-alpha

### Fixed

- Release CI: Remove another failing step

## 3.0.16-alpha

### Fixed

- Release CI: Temporarily comment out failing step

## 3.0.15-alpha

### Fixed

- Release CI: Fix checkout action position in intelliJ release.

## 3.0.14-alpha

### Added

- Layer integration test. Tests the layer's loading and hooking in an http mirroring simulation with a flask web app.
  Addresses but does not
  close [.

### Fixed

- Release CI: Fix paths for release artifacts

## 3.0.13-alpha

### Added

- mirrord-cli: added a SIP protection check for macos binaries,
  closes []

### Fixed

- Fixed unused dependencies issue, closes []

### Changed

- Remove building of arm64 Docker image from the release CI

## 3.0.12-alpha

### Added

- Release CI: add extensions as artifacts, closes []

### Changed

- Remote operations that fail logged on `info` level instead of `error` because having a file not found, connection
  failed, etc can be part of a valid successful flow.
- mirrord-layer: When handling an outgoing connection to localhost, check first if it's a socket we intercept/mirror,
  then just let it connect normally.
- mirrord-layer: removed `tracing::instrument` from `*_detour` functions.

### Fixed

- `getaddrinfo` now uses [`trust-dns-resolver`](https://docs.rs/trust-dns-resolver/latest/trust_dns_resolver/) when
  resolving DNS (previously it would do a `getaddrinfo` call in mirrord-agent that could result in incompatibility
  between the mirrored pod and the user environments).
- Support clusters running Istio. Closes [].

## 3.0.11-alpha

### Added

- Support impersonated deployments, closes []
- Shorter way to select which deployment/pod/container to impersonate through `--target`
  or `MIRRORD_IMPERSONATED_TARGET`, closes []
- mirrord-layer: Support config from file alongside environment variables.
- intellij-ext: Add version check, closes []
- intellij-ext: better support for Windows with WSL.

### Deprecated

- `--pod-name` or `MIRRORD_AGENT_IMPERSONATED_POD_NAME` is deprecated in favor of `--target`
  or `MIRRORD_IMPERSONATED_TARGET`

### Fixed

- tcp-steal working with linkerd meshing.
- mirrord-layer should exit when agent disconnects or unable to make initial connection

## 3.0.10-alpha

### Added

- Test that verifies that outgoing UDP traffic (only with a bind to non-0 port and a
  call to `connect`) is successfully intercepted and forwarded.

### Fixed

- macOS binaries should be okay now.

## 3.0.9-alpha

### Changed

- Ignore http tests because they are unstable, and they block the CI.
- Bundle arm64 binary into the universal binary for MacOS.

## 3.0.8-alpha

### Fixed

- release CI: Fix dylib path for `dd`.

## 3.0.7-alpha

### Fixed

- mirrord-layer: Fix `connect` returning error when called on UDP sockets and the
  outgoing traffic feature of mirrord is disabled.
- mirrord-agent: Add a `tokio::time:timeout` to `TcpStream::connect`, fixes golang issue where sometimes it would get
  stuck attempting to connect on IPv6.
- intelliJ-ext: Fix CLion crash issue, closes []
- vscode-ext: Support debugging Go, and fix issues with configuring file ops and traffic stealing.

### Changed

- mirrord-layer: Remove check for ignored IP (localhost) from `connect`.
- mirrord-layer: Refactor `connect` function to be less bloated.
- `.dockerignore` now ignores more useless files (reduces mirrord-agent image build time, and size).
- mirrord-agent: Use `tracing::instrument` for the outgoing traffic feature.
- mirrord-agent: `IndexAllocator` now uses `ConnectionId` for outgoing traffic feature.

## 3.0.6-alpha

### Changed

- mirrord-layer: Remove `tracing::instrument` from `go_env::goenvs_unix_detour`.
- mirrord-layer: Log to info instead of error when failing to write to local tunneled streams.

### Added

- mirrord-layer, mirrord-cli: new command line argument/environment variable - `MIRRORD_SKIP_PROCESSES` to provide a
  list of comma separated processes to not to load into.
  Closes []
  , []
- release CI: add arm64e to the universal dylib
- intellij-ext: Add support for Goland

## 3.0.5-alpha

### Fixed

- mirrord-layer: Return errors from agent when `connect` fails back to the hook (previously we were handling these as
  errors in layer, so `connect` had slightly wrong behavior).
- mirrord-layer: instrumenting error when `write_detour` is called to stdout/stderr
- mirrord-layer: workaround for `presented server name type wasn't supported` error when Kubernetes server has IP for CN
  in certificate. []

### Changed

- mirrord-layer: Use `tracing::instrument` to improve logs.

### Added

- Outgoing UDP test with node. Closes []

## 3.0.4-alpha

### Fixed

- Fix crash in VS Code extension happening because the MIRRORD_OVERRIDE_ENV_VARS_INCLUDE and
  MIRRORD_OVERRIDE_ENV_VARS_EXCLUDE vars being populated with empty values (rather than not being populated at all)
  .Closes [].
- Add exception to gradle when dylib/so file is not found.
  Closes []
- mirrord-layer: Return errors from agent when `connect` fails back to the hook (previously we were handling these as
  errors in layer, so `connect` had slightly wrong behavior).

## 3.0.3-alpha

### Changed

- Changed agent namespace to default to the pod namespace.
  Closes [].

## 3.0.2-alpha

### Added

- Code sign Apple binaries.
- CD - Update latest tag after release is published.

### Changed

- In `go-e2e` test, call `os.Exit` instead of sending `SIGINT` to the process.
- Install script now downloads latest tag instead of main branch to avoid downtime on installs.

### Fixed

- Fix Environment parsing error when value contained '='
  Closes [].
- Fix bug in outgoing traffic with multiple requests in quick succession.
  Closes [].

## 3.0.1-alpha

### Fixed

- Add missing dependency breaking the VS Code release.

## 3.0.0-alpha

### Added

- New feature: UDP outgoing, mainly for Go DNS but should work for most use cases also!
- E2E: add tests for python's fastapi with uvicorn
- Socket ops - `connect`: ignore localhost and ports 50000 - 60000 (reserved for debugger)
- Add "*.plist" to `IGNORE_REGEX`, refer [].

### Changed

- Change all functionality (incoming traffic mirroring, remote DNS outgoing traffic, environment variables, file reads)
  to be enabled by default. ***Note that flags now disable functionality***

### Fixed

- mirrord-layer: User-friendly error for invalid kubernetes api certificate
- mirrord-cli: Add random prefix to the generated shared lib to prevent Bus Error/EXC_BAD_ACCESS
- Support for Go 1.19>= syscall hooking
- Fix Python debugger crash in VS Code Extension. Closes [].

## 2.13.0

### Added

- Release arm64 agent image.

### Fixed

- Use selected namespace in IntelliJ plugin instead of always using default namespace.

## 2.12.1

### Fixed

- Fix bug where VS Code extension would crash on startup due to new configuration values not being the correct type.
- Unset DYLD_INSERT_LIBRARIES/LD_PRELOAD when creating the agent.
  Closes [].
- Fix NullPointerException in IntelliJ Extension. Closes [].
- FIx dylib/so paths for the IntelliJ Extension. Closes [[#337](https://github.com/metalbear-co/mirrord/pull/352)].

## 2.12.0

### Added

- Add more configuration values to the VS Code extension.
- Warning when using remote tcp without remote DNS (can cause ipv6/v4 issues).
  Closes

### Fixed

- VS Code needed restart to apply kubectl config/context change.
  Closes [316](https://github.com/metalbear-co/mirrord/issues/316).
- Fixed DNS feature causing crash on macOS on invalid DNS name due to mismatch of return
  codes..
- Fixed DNS feature not using impersonated container namespace, resulting with incorrect resolved DNS names.
- mirrord-agent: Use `IndexAllocator` to properly generate `ConnectionId`s for the tcp outgoing feature.
- tests: Fix outgoing and DNS tests that were passing invalid flags to mirrord.
- Go Hooks - use global ENABLED_FILE_OPS
- Support macOS with apple chip in the IntelliJ plugin.
  Closes.

## 2.11.0

### Added

- New feature: mirrord now supports TCP traffic stealing instead of mirroring. You can enable it by
  passing `--tcp-steal` flag to cli.

### Fixed

- mirrord-layer: Go environment variables crash - run Go env setup in a different stack (should
  fix

### Changed

- mirrord-layer: Add `#![feature(let_chains)]` to `lib.rs` to support new compiler version.

## 2.10.1

### Fixed

- CI:Release - Fix typo that broke the build

## 2.10.0

### Added

- New feature, [tcp outgoing traffic](https://github.com/metalbear-co/mirrord/issues/27). It's now possible to make
  requests to a remote host from the staging environment context. You can enable this feature setting
  the `MIRRORD_TCP_OUTGOING` variable to true, or using the `-o` option in mirrord-cli.
- mirrord-cli add login command for logging in to metalbear-cloud
- CI:Release - Provide zip and sha256 sums

### Fixed

- Environment variables feature on Golang programs. Issue #292 closed in #299

## 2.9.1

### Fixed

- CI - set typescript version at 4.7.4 to fix broken release action

## 2.9.0

### Added

- Support for Golang fileops
- IntelliJ Extension for mirrord

### Changed

- mirrord-layer: Added common `Result` type to to reduce boilerplate, removed dependency of `anyhow` crate.
- mirrord-layer: Split `LayerError` into `LayerError` and `HookError` to distinguish between errors that can be handled
  by the layer and errors that can be handled by the hook. (no more requiring libc errno for each error!).
  Closes

## 2.8.1

### Fixed

- CI - remove usage of ubuntu-18.04 machines (deprecated)

## 2.8.0

### Added

- E2E - add basic env tests for bash scripts

### Fixed

- mirrord-agent - Update pcap library, hopefully will fix dropped packets (syn sometimes missed in e2e).
- mirrord-agent/layer - Sometimes layer tries to connect to agent before it finished loading, even though pod is
  running. Added watching the log stream for a "ready" log message before attempting to connect.

### Changed

- E2E - describe all pods on failure and add file name to print of logs.
- E2E - print timestamp of stdout/stderr of `TestProcess`.
- E2E - Don't delete pod/service on failure, instead leave them for debugging.
- mirrord-agent - Don't use `tokio::spawn` for spawning `sniffer` (or any other namespace changing task) to avoid
  namespace-clashing/undefined behavior. Possibly fixing bugs.
- Change the version check on the VS Code extension to happen when mirrord is enabled rather than when the IDE starts
  up.

## 2.7.0

### Added

- mirrord-layer: You can now pass `MIRRORD_AGENT_COMMUNICATION_TIMEOUT` as environment variable to control agent
  timeout.
- Expand file system operations with `access` and `faccessat` hooks for absolute paths

### Fixed

- Ephemeral Containers didn't wait for the right condition, leading to timeouts in many cases.
- mirrord-layer: Wait for the correct condition in job creation, resolving startup/timeout issues.
- mirrord-layer: Add a sleep on closing local socket after receiving close to let local application respond before
  closing.
- mirrord-layer: Fix DNS issue where `ai_addr` would not live long enough (breaking the remote DNS feature).

### Changed

- Removed unused dependencies from `mirrord-layer/Cargo.toml`. (Closes #220)
- reduce e2e flakiness (add message sent on tcp listen subscription, wait for that message)
- reduce e2e flakiness - increase timeout time
- mirrord-layer - increase agent creation timeout (to reduce e2e flakiness on macOS)
- E2E - Don't do file stuff on http traffic to reduce flakiness (doesn't add any coverage value..)
- mirrord-layer - Change tcp mirror tunnel `select` to be biased so it flushes all data before closing it (better
  testing, reduces e2e flakiness)
- E2E - unify resolve_node_host for linux and macOS with support for wsl provided Docker & Kubernetes
- E2E - add `trace` for tests to have parameterized arguments printed
- mirrord-agent - add debug print of args to identify runs
- E2E - remove double `--extract-path` parameter in tests
- E2E - macOS colima start with 3 cores and 8GB of RAM.
- E2E - Increase agent communication timeout to reduce flakiness.
- mirrord-layer - add `DetourGuard` to prevent unwanted calls to detours from our code.
- mirrord-layer - extract reused detours to separate logic functions
- E2E - macOS run only sanity http mirror traffic with Python

## 2.6.0

### Added

- Add a flag for the agent, `--ephemeral-container`, to correctly refer to the filesystem i.e. refer to root path
  as `/proc/1/root` when the flag is on, otherwise `/`.
- Add support for Golang on amd64 (x86-64).

### Changed

- Assign a random port number instead of `61337`. (Reason: A forking process creates multiple agents sending traffic on
  the same port, causing addrinuse error.)
- `mirrord-layer/socket` now uses `socket2::SockAddr` to comply with Rust's new IP format.

### Fixed

- Fix filesystem tests to only run if the default path exists.
- Fix extension not running due to the node_modules directory not being packaged.

## 2.5.0

### Added

- New feature, [remote DNS resolving](https://github.com/metalbear-co/mirrord/issues/27#issuecomment-1154072686).
  It is now possible to use the remote's `addrinfo` by setting the `MIRRORD_REMOTE_DNS` variable to
  `true`, or using the `-d` option in mirrord-cli.
- New feature, [Ephemeral Containers](https://github.com/metalbear-co/mirrord/issues/172).
  Use Kubernetes beta feature `Ephemeral Containers` to mirror traffic with the `--ephemeral-container` flag.
- E2E tests on macos for Golang using the Gin framework.

### Changed

- Refactored `mirrord-layer/socket` into a module structure similar to `mirrord-layer/file`.
- Refactored the error part of the many `Result<Response, ResponseError>`.
- Refactored `file` related functions, created `FileHandler` and improved structure.
- Refactored error handling in mirrord-layer.
- E2E: Collect minikube logs and fix collecting container logs
- E2E: macOS use colima instead of minikube.
- Refactored `mirrord-layer/lib.rs` - no more passing many arguments! :)
- Refactored `mirrord-layer/lib.rs` - remove `unwrap()` and propagate error using `Result`

### Fixed

- Handle unwraps in fileops to gracefully exit and enable python fileops tests.
- Changed `addrinfo` to `VecDeque` - fixes a potential bug (loss of order)

## 2.4.1

### Added

- mirrord-cli `exec` subcommand accepts `--extract-path` argument to set the directory to extract the library to. Used
  for tests mainly.
- mirrord-layer provides `MIRRORD_IMPERSONATED_CONTAINER_NAME` environment variable to specify container name to
  impersonate. mirrord-cli accepts argument to set variable.
- vscode-ext provides quick-select for setting `MIRRORD_IMPERSONATED_CONTAINER_NAME`

### Changed

- Refactor e2e, enable only Node HTTP mirroring test.
- E2E: add macOS to E2E, support using minikube by env var.
- E2E: Skip loading to docker before loading to minikube (load directly to minikube..)
- layer: Environment variables now load before process starts, no more race conditions.

### Fixed

- Support connections that start with tcp flags in addition to Syn (on macOS CI we saw CWR + NS)
- `fcntl` error on macOS by a workaround.

## 2.3.1

### Changed

- Refactor(agent) - change `FileManager` to be per peer, thus removing the need of it being in a different task, moving
  the handling to the peer logic, change structure of peer handling to a struct.
- Don't fail environment variable request if none exists.
- E2E: Don't assert jobs and pods length, to allow better debugging and less flakiness.
- Refactor(agent) - Main loop doesn't pass messages around but instead spawned peers interact directly with tcp sniffer.
  Renamed Peer -> Client and ClientID.
- Add context to agent/job creation errors (Fixes #112)
- Add context to stream creation error (Fixes #110)
- Change E2E to use real app, closes

## 2.3.0

### Added

- Add support for overriding a process' environment variables by setting `MIRRORD_OVERRIDE_ENV_VARS` to `true`. To
  filter out undesired variables, use the `MIRRORD_OVERRIDE_FILTER_ENV_VARS` configuration with arguments such
  as `FOO;BAR`.

### Changed

- Remove `unwrap` from the `Future` that was waiting for Kube pod to spin up in `pod_api.rs`. (Fixes #110)
- Speed up agent container image building by using a more specific base image.
- CI: Remove building agent before building & running tests (duplicate)
- CI: Add Docker cache to Docker build-push action to reduce build duration.
- CD release: Fix universal binary for macOS
- Refactor: Change protocol + mirrord-layer to split messages into modules, so main module only handles general
  messages, passing down to the appropriate module for handling.
- Add a CLI flag to specify `MIRRORD_AGENT_TTL`
- CI: Collect mirrord-agent logs in case of failure in e2e.
- Add "app" = "mirrord" label to the agent pod for log collection at ease.
- CI: Add sleep after local app finishes loading for agent to load filter make tests less flaky.
- Handle relative paths for open, openat
- Fix once cell renamings, PR [#98165](https://github.com/rust-lang/rust/pull/98165)
- Enable the blocking feature of the `reqwest` library

## 2.2.1

### Changed

- Compile universal binaries for MacOS. (Fixes #131)
- E2E small improvements, removing sleeps. (Fixes #99)

## 2.2.0

### Added

- File operations are now available behind the `MIRRORD_FILE_OPS` env variable, this means that mirrord now hooks into
  the following file functions: `open`, `fopen`, `fdopen`, `openat`, `read`, `fread`, `fileno`, `lseek`, and `write` to
  provide a mirrored file system.
- Support for running x64 (Intel) binary on arm (Silicon) macOS using mirrord. This will download and use the x64
  mirrord-layer binary when needed.
- Add detours for fcntl/dup system calls, closes

### Changed

- Add graceful exit for library extraction logic in case of error.
- Refactor the CI by splitting the building of mirrord-agent in a separate job and caching the agent image for E2E
  tests.
- Update bug report template to apply to the latest version of mirrord.
- Change release profile to strip debuginfo and enable LTO.
- VS Code extension - update dependencies.
- CLI & macOS: Extract to `/tmp/` instead of `$TMPDIR` as the executed process is getting killed for some reason.

### Fixed

- Fix bug that caused configuration changes in the VS Code extension not to work
- Fix typos

## 2.1.0

### Added

- Prompt user to update if their version is outdated in the VS Code extension or CLI.
- Add support for docker runtime, closes.
- Add a keep-alive to keep the agent-pod from exiting, closes

## 2.0.4

Complete refactor and re-write of everything.

- The CLI/VSCode extension now use `mirrord-layer` which loads into debugged process using `LD_PRELOAD`
  /`DYLD_INSERT_LIBRARIES`.
  It hooks some of the syscalls in order to proxy incoming traffic into the process as if it was running in the remote
  pod.
- Mono repo
- Fixed unwraps inside
  of [agent-creation](https://github.com/metalbear-co/mirrord/blob/main/mirrord-layer/src/lib.rs#L75),
  closes

