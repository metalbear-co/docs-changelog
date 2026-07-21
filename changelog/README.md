---
title: Operator Changelog
date: 2023-08-15T00:00:00.000Z
lastmod: 2026-07-21T00:00:00.000Z
draft: false
images: []
weight: 100
toc: true
tags:
  - team
  - enterprise
description: >-
  The release changelog for the mirrord operator.
---

## 3.186.0 - 2026-07-21


### Added

- Add CockroachDB database branching support.
- Add MariaDB database branching support.
- Added IRSA support for AWS RDS branching.
- Operator now servces a no-session diagnostic endpoint used by `mirrord
  diagnose latency` command.
- Support a custom image per DB branch via the `image` field in the mirrord
  config (all db types), restrictable with the per-database
  `dbPod.allowedImages` Helm value.
- `operator.extraEnv` in the mirrord-operator chart now accepts mapped values,
  making it possible to
  define environment variables using `valueFrom` to access the Kubernetes
  downward API.

## 3.185.1 - 2026-07-19


### Fixed

- A queue splitting session whose `jq_filter` exceeds its evaluation time limit
  is now cancelled with an error, instead of staying alive while silently never
  matching a message. Applies to every queue type that supports jq filters:
  SQS, Kafka, GCP Pub/Sub, Azure Service Bus, Redis Pub/Sub, BullMQ, and
  Temporal.
- Idle previews waking on other services' messages.

## 3.185.0 - 2026-07-17


### Added

- Add support for db params with value pattern in preview env.
- Added idle mode for preview environments (`feature.preview.idle`).


### Changed

- Renamed `feature.preview.idle.timeout_secs` to `sleep_after_secs`.


### Fixed

- Fixed `mirrord operator session kill` and `kill-all` not killing
  multi-cluster sessions, and `operator status` showing an id `kill` rejects.
- Fixed operator API requests intermittently failing when running more than one
  replica, after a restart.
- Fixed orphaned Azure Service Bus subscriptions.

## 3.184.0 - 2026-07-14


### Added

- Added testing for INT-380
- Inject `mirrord-key` into all forwarded messages.
- It's now possible to create extra objects alongside the
  operator/license-server helm installations using the `extraObjects` field.
- Preview environments can now selectively filter which labels are copied from
  the target through the `feature.preview.labels.{include,exclude}` options,
  analogous to our existing `feature.env.{include,exclude}` options.
- Redis branch pods can now run TLS-enabled custom images.
- Support CronJob and Job database branching targets.


### Fixed

- DB branching now reports a clear error when required source database
  credential environment variables are missing.
- Fix `ASB` same subscription name config.
- Fixed copy target with an HTTP filter reliably stealing matching requests.

## 3.183.0 - 2026-07-13


### Security

- Added a `Strict-Transport-Security` (HSTS) header to management dashboard
  responses, so browsers only ever connect over HTTPS and cannot be downgraded
  to plaintext HTTP.


### Added

- Add Multi Cluster support for Preview Env Queue Splitting.
- Add `mTLS` support for `Temporal` queue splitting.
- Add a retryable agent error variant.
- Add support for generic db branching.
- Added Redis Pub/Sub support to mirrord subscribe.
- Added `feature.preview.secret_mounts` to mount files into a preview pod from
  a Kubernetes Secret, so sensitive files can be access-controlled via RBAC
  separately from the session.
- Added an operator API to list active mirrord sessions, cluster-wide or scoped
  to a namespace (`GET /sessions`, `GET /namespaces/{namespace}/sessions`).
- Added support for jq filters in Kafka queue splitting (`jq_filter` in the
  mirrord config). The jq program runs on a JSON representation of each Kafka
  message (`topic`, `partition`, `offset`, `timestamp`, `key`, `payload`, and
  `headers`), so sessions can filter on fields inside the message body.
  Combined with a header filter, a message must match both. Not supported with
  the Java Kafka client sidecar.
- Allow sharing preview environments.
- Queue splitting support for mirror mode (`queue_mode: mirror`), delivering
  matched messages to both the session and the deployed application.


### Changed

- Add queue splitting support for mirrord up.


### Fixed

- A database branch whose engine is disabled in the operator's configuration is
  now marked `Failed` with a clear reason, instead of staying `Pending` forever
  and leaving the CLI to time out waiting for it to become ready.
- Fixed `multi-cluster` envoy RBAC probe.
- Fixing dangling resources for `ASB`.
- Helm chart no longer prints a spurious "Missing license key" warning on
  install/upgrade when the operator is configured with only a cloud API key
  (`cloud.apiKey`).

## 3.182.0 - 2026-07-08


### Added

- Add support for ClickHouse.
- Add support for Google Spanner.
- Added BullMQ support to `mirrord subscribe`.
- Added support for RabbitMQ to `mirrord subscribe`
- Implement Flyway migrations.
- Operator telemetry now reports a stable per-cluster identifier (the UID of
  the `default` namespace), so events can be grouped by the cluster they
  originate from. **This requires an additional RBAC permission:** the operator
  ClusterRole now needs `get` on the `default` namespace. The bundled Helm
  chart adds this automatically; if you manage the operator's RBAC yourself,
  grant `get` on the `default` namespace or the identifier will be omitted.
- The operator Helm chart now accepts an optional `operator.clusterName`. When
  set, the operator reports this human-friendly name on its startup telemetry
  events, so the cluster shows up with a recognizable label (instead of only an
  opaque identifier) in the license server and the mirrord cloud.
- Wired up GCP PubSub queue splitting to `mirrord subscribe`.


### Changed

- Dashboard onboarding: revamped the getting-started wizard into a full-screen
  flow with a required operator step (generate an API key, install the
  operator, and verify it via usage) ahead of the CLI step, deriving progress
  from live usage instead of local storage (RFC 0008).


### Fixed

- Fixed cluster-scoped queue split sessions (and their temporary broker
  resources) leaking after a preview environment was stopped. The split session
  is now tracked by an annotation rather than an owner reference to the
  namespaced preview session, so the operator cleans it up when the preview
  session is gone.

## 3.181.0 - 2026-07-05


### Changed

- operator Helm chart: a cloud API key (`cloud.apiKey.key` / `keyRef` /
  `gsmRef`) now satisfies the credential requirement, so the operator can be
  installed without any `license.*` source — it obtains the license over the
  API (RFC 0008).


### Fixed

- operator: `helm uninstall` no longer hangs or strands operator CRs. The
  chart's `pre-delete` cleanup Job now deletes all operator CRs first — letting
  the still-running operator execute their finalizers, and stripping the
  finalizers of any stragglers after a bounded grace — and only then deletes
  the CRDs and the operator. Previously it deleted the CRDs first, which shut
  the operator down mid-cleanup (watch streams breaking is treated as a
  critical error) and could leave CRs stuck `Terminating`, blocking the
  uninstall indefinitely.
- operator: on shutdown, clear the session-TTL finalizer from
  `MirrordClusterSession`s that are already being deleted, so an exiting
  operator no longer strands them — and the sessions CRD — stuck in
  `Terminating`. Live sessions keep the finalizer: in an HA deployment the
  surviving replicas keep honouring the deletion-delay TTL, and live sessions
  present at uninstall are cleaned up by the chart's `pre-delete` hook.
- operator: the queue-splitting workload-splits task now exits on graceful
  shutdown instead of stalling it for the full 15s limit. The stall kept the
  terminating leader pod (and its leadership lease) alive, leaving the operator
  leaderless long enough for reconnecting clients to lose their sessions during
  a rolling upgrade.

## 3.180.0 - 2026-07-05


### Added

- Added Cloud SQL Auth Proxy support for MySQL database branching. When the
  target pod reaches a GCP Cloud SQL MySQL instance only through a loopback
  Auth Proxy sidecar, the branch init container now starts its own proxy to
  copy schema and data, matching the existing PostgreSQL behavior.
- Added support for fetching database branching connection parameters from
  Google Secret Manager. The branch init container fetches the secret at
  data-copy time using the target pod's service account (Workload Identity) via
  Application Default Credentials, so the operator never reads the secret and
  needs no extra permissions.
- Dashboard: a Settings tab to generate, view, and revoke the operator API key
  (RFC 0008).


### Changed

- Updated `kube-rs` to 4.0. The Kubernetes client now falls back to the
  OS-native trust store via the rustls platform verifier when no CA is
  configured, instead of bundled webpki roots.


### Fixed

- Clean up split resoures when the configuration is deleted.
- Preview sessions now expand the `"*"` wildcard in `split_queues`.

## 3.179.1 - 2026-07-03


### Fixed

- Fixed multi-cluster CRD broadcast sync failing with a 409 conflict when a
  resource on a remote cluster had fields owned by another field manager (e.g.
  from a past client-side `kubectl apply`).

## 3.179.0 - 2026-07-02


### Added

- Added a list form for the `split_queues` config so the same queue id can be
  used more than once across different broker types.
- Added a queue-splitting status API that serves a live, namespaced view of
  active split sessions from the operator's in-memory state, surfaced to users
  through `mirrord queues`. Each session reports its target, owner, and
  requested filters, plus a status with the lifecycle phase, the concrete
  queues resolved from the target (topic / consumer group / queue /
  subscription per broker), and per-pod patched/ready state. On the primary
  cluster the view also aggregates split sessions from all connected clusters.
- Added support for DynamoDB database branching. Configure it under
  `feature.db_branches` with `type: dynamodb` and enable it with the
  `operator.dynamodbBranching` Helm option.
- Added the `GET /v1/dashboard/deployments` endpoint that powers the
  deployments view: per-deployment mirrord coverage (Covered / Uncovered /
  Unmatched), defect attribution to the most recent prior deploy of the same
  service and env, per-team defect rates, coverage and defect-reduction KPIs,
  and trend series. Postgres-only.
- operator Helm chart: `cloud.apiKey` value (`key` / `keyRef` / `gsmRef`) to
  configure the operator's cloud API key — the default credential for
  authenticating to the mirrord cloud and obtaining the license over the API
  (RFC 0008).


### Fixed

- Fixed multi-cluster CRD sync failing with a 409 conflict when the source
  resource carried the `kubectl.kubernetes.io/last-applied-configuration`
  annotation from a client-side `kubectl apply`. The broadcast sync now strips
  it before applying to remote clusters.

## 3.178.0 - 2026-07-01


### Added

- Added a Deployments tab to the dashboard (gated by `deploymentsEnabled`):
  coverage and defect-reduction KPIs, staging-defect-rate and defects-prevented
  trends, a per-team defect-rate breakdown, recommended actions, and a
  per-deployment detail table showing whether each release was tested with
  mirrord first (Covered / Uncovered / Unmatched).
- The mirrord Operator can now authenticate to the mirrord cloud with a
  dedicated, rotatable, revocable API key, separate from the license. Generate
  and manage it from the dashboard, then provide it to the chart via the new
  `cloud.apiKey` value — either inline (`key`), a Kubernetes secret (`keyRef`),
  or a Google Secret Manager reference (`gsmRef`). The Operator exchanges the
  key for short-lived tokens to call the cloud. When unset, the Operator keeps
  authenticating with its license key, so existing installs are unaffected
  until they opt in.


### Fixed

- Azure Service Bus topic splitting no longer fails to start when its ingest
  subscription or routing rules already exist from a crashed, aborted, or
  quickly retried prior session; the existing resources are now adopted
  instead.
- Fix sessions receiving messages from unspecified queues.
- Fixed DB branching params mode defaults for when host is not passed in
- Fixed MYSQL had no defaults previously.

## 3.177.0 - 2026-06-30


### Added

- Added a `dashboard.deploymentsEnabled` Helm option (off by default) that
  gates the dashboard's deployments view, wiring
  `DASHBOARD_DEPLOYMENTS_ENABLED` into the license server and requiring
  `database.kind=postgres`, mirroring the existing adoption view gate.
- The PostgreSQL branch initializer now applies `connectionSettings`.


### Fixed

- Fix `mirrord operator session kill` suggestion syntax.

## 3.176.0 - 2026-06-30


### Added

- Added a `POST /v1/defects` license-server endpoint that ingests defect events
  in batches (upsert by id, so a detector can retry without creating
  duplicates), feeding the deployments dashboard backend.
- Added a `POST /v1/deployments` license-server endpoint that records deploy
  events (upsert by id), the first slice of the deployments dashboard backend.
- Added support for Kafka events to `mirrord subscribe` (Kafka Streams not
  supported).
- Added support for SQS to `mirrord subscribe`.


### Changed

- Dashboard: the CSV export is now a button in the top header bar (next to
  settings) instead of only inside the settings panel, and the adoption heatmap
  shows each team's size next to its name (same font) instead of tucked in the
  bottom-left corner.
- More adoption and usage dashboard refinements: choose the trend-chart
  interval (daily / weekly / bi-weekly / monthly) per selected range, dismiss
  suggested actions, hover any status colour to see its meaning and count, the
  usage tab now says "Service" instead of "Target" and shows each engineer's
  team, and the ROI calculator pre-fills the developer count from your data.
- Updated adoption tab copy (bucket labels and tooltips) to match the design
  review.


### Fixed

- Azure Service Bus topic splitting no longer fails to start when its ingest
  subscription already exists.
- Copy target sessions now apply jq message filters for SQS, GCP Pub/Sub and
  Azure Service Bus queue splits.
- Fixed two dashboard search issues: the Session Activity search now matches
  users by display name (not just the raw identifier), and search inputs no
  longer show a duplicate clear button (the browser's native search "x" is
  suppressed in favor of the component's own).
- Queue splitting and other session features now work through ingress proxies
  (e.g. GKE Connect Gateway) that reject the encoded connect URL query string,
  by accepting the connect parameters in a header.

## 3.175.0 - 2026-06-29


### Added

- Added a "Never started" list to the adoption dashboard's engineer-activity
  row and the group detail dialog, surfacing engineers who have never run a
  mirrord session by name. Previously they only appeared as an aggregate count.
- The operator's license key can now be set to a Google Secret Manager secret
  name using the new `license.keyGsmRef` option on the Helm chart.


### Changed

- Polished the adoption dashboard: a single gradual colour scale, shared across
  the funnel, team heatmap, group comparison and services and ordered
  worst-to-best left-to-right; reworded the suggested actions; the scope filter
  now shows the full group-type label; and the header uses a crisp logo in dark
  mode.
- The adoption dashboard API now attributes each service to the teams whose
  engineers actually used it in a session (derived from operator session
  events), in addition to the teams it was explicitly assigned to, so a team
  with active sessions is no longer reported as owning zero services it
  demonstrably used.


### Fixed

- Fixed the adoption trend chart showing less history on the "All" range than
  on shorter ranges, and the group detail dialog now counts the services a team
  actually used in sessions rather than only those explicitly assigned to it.
- Legacy SQS `MirrordWorkloadQueueRegistry` splits again honor
  `operator.sqsSplittingLingerTimeout` as their default drain timeout when they
  do not carry their own.
- Preview Environments no longer fail to start when targetting a workload
  containing multiple containers that expose the same port.

## 3.174.0 - 2026-06-24


### Changed

- Azure Service Bus queue splitting now includes Azure's response body in its
  REST errors and logs failed entity operations, so a failure like a missing
  topic is debuggable. A `NotFound` while creating the operator's ingest
  subscription is now reported as a clear "topic not found" message that points
  at the topic name and the Service Bus namespace to check, instead of a bare
  "resource not found" on an internal subscription name.

## 3.173.0 - 2026-06-24


### Added

- Multi-cluster queue splitting now handles the
  `operator.metalbear.co/skip-mc-sync` label on `MirrordSplitConfig` and
  `MirrordPropertyList`. When set to `true`, the operator does not broadcast
  that resource from the primary to the remote clusters, so users who deploy
  these CRDs to every cluster themselves.


### Fixed

- Fixed a batch of adoption dashboard scoping and edge-case issues: the header
  counts (engineers / services / teams) now reflect the selected scope instead
  of always showing company totals; the funnel's "vs company average"
  comparison now appears when scoping to a business unit, not only a single
  team; the active-users trend now plots zero-activity days instead of dropping
  them and can no longer read above 100%; the service list no longer shows an
  empty page after filtering; the heatmap shows a clear message when a scope
  has no teams; and the user-classification tooltips were corrected, with
  "going cold" alerts now also flagging long-abandoned teams.
- Fixed the adoption dashboard's team comparison chart collapsing back to the
  top-level group view when you scoped to a single team. Scoping to a leaf team
  now keeps the chart at its parent level (e.g. "Teams in Engineering") with
  the selected team outlined, while the rest of the page scopes to that team.
  The scope filter also no longer shows an empty "No options" dropdown once you
  reach a team with no sub-groups.
- Queue splitting `drainTimeout` works as follows: absent value waits
  indefinitely for the fallback queues to drain, and `0` tears down immediately
  so the workload env patch reverts as soon as no sessions remain.

## 3.172.0 - 2026-06-23


### Added

- `GCP Pub/Sub` queue splitting now  `ack_deadline_seconds`,
  `message_retention_seconds`, and `expiration_seconds` keys from a queue's
  `queueConfig` `MirrordPropertyList`, letting you configure the temporary
  subscriptions.

## 3.171.0 - 2026-06-22


### Changed

- The adoption dashboard's trend chart now reflects the selected scope:
  filtering to a team (or any group) shows that group's "% active over time"
  instead of the org-wide series.


### Fixed

- Azure Service Bus queue splitting now authenticates with the operator pod's
  Azure Workload Identity when the property list sets only
  `fully_qualified_namespace`.

## 3.170.0 - 2026-06-19


### Added

- Add `/events` endpoint for `mirrord subscribe` (only HTTP and azure events
  supported).
- Added a `url` property to the RabbitMQ `MirrordPropertyList`, allowing
  connection details (host, port, vhost, username, password, scheme, SASL
  mechanism) to be supplied as a single AMQP URI. Individually set properties
  still take precedence over values parsed from the URL.
- Added a deployment-level option to rewrite how user identifiers are displayed
  on the utilization dashboard (for example, stripping an identity-provider
  prefix such as `keycloak:`), configured via the chart rather than in-app, and
  stopped surfacing raw identifiers (such as certificate fingerprints) next to
  resolved user names.
- Added configurable `pg_dump` and `mysqldump` arguments for PostgreSQL and
  MySQL database branches.


### Changed

- Migrating `Kafka` queue splitting to the unified `operator-queue-splitting`
  crate and `CRDs`.
- Migrating `RabbitMQ` queue splitting to the unified
  `operator-queue-splitting` crate and `CRDs`.
- mirrord admin dashboard: addressed product review of the adoption view —
  consistent active-left bucket order across both charts, swapped the
  distribution and per-group sections, moved bucket definitions into hover
  tooltips with a compact legend, consistent section title at any hierarchy
  depth, and clearer wording on the current-totals note.
- mirrord admin dashboard: group types in the directory upload are now
  free-form labels. The dashboard derives the hierarchy from the uploaded
  relationships and shows each org their own vocabulary instead of the
  previously fixed bu / cio_domain / team set.


### Fixed

- Fix edge case where old active `CRDs` with the `*` selctor were not migrated
  correctly.
- Fixed PostgreSQL branch creation failing when the source schema references
  roles missing on the branch. The source roles are now recreated as empty
  placeholders before the schema is restored.
- mirrord admin dashboard: fixed the directory join diagnostic suggesting
  display names instead of kubernetes usernames, added template downloads to
  the adoption onboarding checklist, clarified team-scoped engineer stats on
  service views, and several small copy and accessibility fixes.

## 3.169.0 - 2026-06-16


### Added

- Added a `valuePattern` field to queue splitting `appConfig` references.


### Changed

- GCP Pub/Sub errors now include the attempted operation and target resource,
  so permission failures report which call was denied instead of a generic
  `PERMISSION_DENIED`.
- Migrating `SQS` to the unified `CRDs` and unfiiying more logic between queue
  splitting services.
- Utilization dashboard's SAML proxy now uses an image from
  ghcr.io/metalbear.co.
- mirrord admin dashboard: the adoption view is now off by default and opt-in
  via the `dashboard.adoptionEnabled` chart value (it requires the Postgres
  backend). When off, the Adoption tab and its directory/service-ownership
  upload sections in Settings are hidden, so deployments on other backends no
  longer surface adoption UI that only returns errors. The frontend learns
  whether the feature is on via a new GET /api/v1/dashboard/config endpoint.


### Fixed

- Fixed two HTTP 500s in the utilization report (GET /api/v1/reports/usage):
  one when a user had mirrord sessions with no recorded duration (null
  aggregate), and one when the selected time range had equal start and end
  (division by zero). The per-user query now counts all sessions, treats
  missing durations as zero time, and guards the daily-average division. Also
  replaced the dev-only "kubectl proxy" error hint shown when the dashboard
  fails to load.

## 3.168.0 - 2026-06-14


### Added

- Add `POST /v1/dashboard/directory` to upload a directory snapshot, returning
  an import report with per-reference skip reasons.
- Add support for `Redis Pub/Sub`.
- Add support for redis with `location: remote`.
- Add the adoption tab to the admin dashboard: org tree scoping, per-group
  adoption funnel, trend, team heatmap, champions, and per-service usage, all
  driven by the directory and operator-session data.
- Added an option to specify a fallback string for the Kafka topic name when
  the expected environment variable does not exist in the workload.
- Added support for splitting [Temporal](https://temporal.io/) task queues
  (experimental/unstable).
- Temporal queue splitting can now filter on headers, search attributes, memo,
  and input for both activity and workflow tasks. `jq_filter` runs against a
  decoded task document exposing these fields (input is under `.input`, and a
  `task_type` field marks `activity` vs `workflow`), and `message_filter`
  supports `header.<name>` and search attribute keys.
- The license server now supports AWS IAM authentication for RDS database
  connections, enabled via the `DATABASE_AWS_RDS_AUTH` environment variable
  (`.Values.database.auth.awsIam=true` via Helm).
- mirrord-license-server chart: optional Cloud SQL Auth Proxy sidecar
  (`database.cloudSqlProxy`) for connecting to a GCP Cloud SQL Postgres
  instance with IAM authentication.


### Fixed

- Fix path for `temporal` emulator for `mc`.
- Fixed an issue where the operator was not able to run queue splitting with
  Kafka clusters configured to use namespace-local broker names.

## 3.167.0 - 2026-06-11


### Added

- A new `mirrord-operator-ci` role has been added, which can be used to manage
  permissions for machine sessions (e.g: `mirrord ci` and `mirrord preview`) in
  CI runners.
- Add an "Export all data" action under Settings in the admin dashboard that
  downloads the full license-server history as a ZIP of CSV files: usage
  metrics, users, targets, session activity, CI, friction action items,
  operator errors, trends, and the adoption directory.
- Added `tls.useExistingSecret` to the operator Helm chart, which mounts the
  secret named by `tls.secret` without the chart creating it (e.g. when it's
  provisioned by an ExternalSecret).
- Azure Service Bus topic queue splitting now uses native Service Bus
  subscription rules, so topics are split without creating a temporary topic..
- DB Branching with PostgreSQL now supports copying data from PostgreSQL
  database over cloud-sql-proxy
- PostgreSQL database branching now supports source instances reached through a
  Cloud SQL Auth Proxy sidecar: when the target pod runs the proxy and
  `gcp_cloud_sql` IAM auth is configured, the branch init container starts its
  own proxy so it can load data from the instance. The init proxy inherits the
  target proxy's private-IP setting, so private-only instances work as well.
- Preview Environments now fail to start when the deployed pods or their
  containers fail, crash or otherwise raise an error, such as failing to pull
  the image, instead of silently succeeding to create the session without
  reporting that something went wrong.
- The admin dashboard now surfaces adoption friction. Connection attempts the
  operator refuses (policy denials, missing or unauthorized targets, disabled
  features, license errors, ...) are reported as ranked action items showing
  the reason, the specific policy/feature/target, and how many distinct users
  each one blocks, so an admin can see exactly what to fix to help their team
  adopt.
- The operator Helm chart now supports `agent.extraEnv` for adding environment
  variables to agent pods.


### Fixed

- Fixed OpenShift SCC template using hardcoded namespace and service account
  name instead of Helm release values, causing agent pods to be rejected by SCC
  when the operator is installed in a namespace other than "mirrord" or with a
  custom service account name.
- Fixed a postgres query failure when getting active CI sessions.
- Fixed the admin dashboard All Time Metrics table showing a user as `unknown`
  when their sessions carried no client username; it now falls back to the
  user's Kubernetes identity instead of letting the `unknown` placeholder mask
  it.
- Fixed the dashboard SAML proxy not forwarding the authenticated user to the
  license server: `X-Remote-User` was sourced from `REMOTE_USER`, which
  `mod_auth_mellon` does not expose as a CGI env var, so the header was never
  set and the dashboard could not load any data after a successful SAML login.
  It is now sourced from the `MELLON_NAME_ID` env var the module does export.
- When doing Kafka splitting set `min.insync.replicas=1` on created ephemeral
  topic.
- `MirrordKafkaTopicsConsumer` now requires exactly one of `groupIdSources` or
  `applicationIdSources` to be set, returning a validation error instead of an
  internal server error when both are absent.

## 3.166.0 - 2026-06-03


### Added

- Added an option to migrate the data from local sqlite to postgress.
- Include the client's mirrord CLI version in the Session Start functional log.
- Preview environments can now control the number of pod replicas in a session
  through the new `feature.preview.replicas` option.


### Fixed

- Fixed `value_pattern` regex not being applied when the init container
  connects to the source database.
- Kafka splitting no longer stops forwarding messages between sessions. The
  forwarder stays subscribed as long as the split is alive.

## 3.165.0 - 2026-05-31


### Added

- Added support for seqpacket outgoing connections.
- Added support for using an existing ServiceAccount in the mirrord Operator
  Helm chart.
- Preview environments now can be temporarily "paused" by a local `mirrord
  exec` session configured to use the same target + key as the preview session.
  While the local session is active incoming traffic won't be directed torwards
  the preview session and the preview pod won't exist, so no outgoing traffic
  is produced either. When the local session ends, the preview environment is
  automatically unpaused.
- Preview environments now survive operator restarts, means that your active
  preview sessions will survive operator upgrades. Note that the session's TTL
  is still respected even across restarts.


### Changed

- Env var to control accepting client certs so you can ignore certs when
  stealing TLS traffic.


### Fixed

- Fixed a transient error during queue splitting where patched resource owner
  reference update error would cause the split to not work.
- Reduced the log noise produced by the preview environment seat controller on
  `preview stop` command.

## 3.164.0 - 2026-05-27


### Added

- Adds support for optional parsing of S3 event notifications in SQS/SNS
  messages. S3 object metadata is exposed to jq filters via S3Metadata.


### Fixed

- Branch database pods now always use a static mock password instead of the
  source database password.
- Fixed branch pod password mismatch when password source is a Secret without
  `env_var_name`.
- GCP Pub/Sub queue splitting now copies source subscription and topic settings
  onto temporary resources instead of using Pub/Sub defaults.
- On licenses using `KubernetesUser` seat tracking, copy-target and
  multi-cluster sessions now consume seats by k8s username, matching regular
  sessions. Previously these paths keyed seats on the client certificate, so a
  single user across ephemeral containers or multiple machines consumed
  multiple seats and reported as multiple users in analytics.

## 3.163.0 - 2026-05-22


### Added

- Added support to license-server with multiple replicas. (Including support
  for postgres database)
- Added total count and session duration metrics for CI and preview.
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


### Fixed

- Fixed an issue where the operator was printing bogus error logs related to
  listing PreviewSession resources.
- Kafka splitting no longer fails with `MessageSizeTooLarge` when the original
  topic has a `max.message.bytes` limit above 1 MB. The forwarder producer's
  client-side cap is lifted, and ephemeral (fallback and session) topics now
  inherit `max.message.bytes` from the original topic.

## 3.162.0 - 2026-05-19


### Added

- Add `iam` auth for `mysql`.
- Add support for unified queue splitting `azure` + `gcp` for preview env.
- Added `authType: aks` for multi-cluster authentication using Azure Workload
  Identity. The operator can now authenticate to remote `AKS` clusters.


### Fixed

- Fixed db branching session env overrides when all connection params use
  literal values or secret sources. The overrides incorrectly returned the
  source database credentials instead of the branch pod's connection details,
  causing the app to connect to the source database instead of the branch.

## 3.161.0 - 2026-05-16


### Added

- Add support for `Azure Servicebus` queue splitting.
- The operator now listens on IPv6 by default for both the kube API/webhook
  port and the Prometheus metrics endpoint, with an automatic IPv4 fallback
  when the primary bind fails. Configurable via `OPERATOR_FALLBACK_ADDR` and
  `OPERATOR_METRICS_FALLBACK_ADDR`.


### Changed

- Add the max concurrent CI session count within a time frame to the license
  server usage report.
- Changes how we display the user name from `identifier` (an id like
  `jx73h371ha9`) to `client_username` (`meowjesty`, when this name is
  available).


### Fixed

- Added missing `branchcredentials` create permission to the
  `mirrord-operator-user` ClusterRole.
- Check if the ci finalizer has been removed before logging the ci_controller
  cleanup BUG message.
- Fixed a bug where the configured keepalive period of unused mirrord-agents
  was not respected.

## 3.160.0 - 2026-05-11


### Security

- Set container-level security contexts for the operator Deployment to run as
  non-root and drop all Linux capabilities.


### Changed

- Update `injectSessionKeyHeader` doc comment to mention HTTP responses and SQS
  messages.


### Fixed

- Dashboard now respects the license seat tracking mode when displaying user
  identifiers, instead of always preferring the Kubernetes username.

## 3.159.0 - 2026-05-10


### Added

- Surface preview environments alongside normal sessions in
  `MirrordOperator.status.sessions` so existing clients (`mirrord ls`, `mirrord
  ui`, browser extension) list them without a separate code path.


### Fixed

- Fixed URL path formatting in MetalBear API client.
- Kafka splitting issue with multiple sessions split the same topics.
- The `APIService` no longer flips to `FailedDiscoveryCheck` after the
  cluster's front-proxy `CA` rotates. The operator now reloads its `TLS` client
  verifier when `kube-system/extension-apiserver-authentication` changes.
- kafka auth on MSK/AWS with pod identity failing after some time

## 3.158.0 - 2026-04-30


### Added

- Added support for GCP Pub Sub.
- Support for single cluster sessions in Multi Cluster.


### Changed

- Cluster-wide Secret permissions for db branching are now gated behind
  `operator.dbBranchingLiteralCredentials` (defaults to `true`).


### Fixed

- Fixed `mirrord dump` failing in multi-cluster setups with 2+ workload
  clusters due to duplicate protocol responses from the multi-cluster router.
- Fixed queue splitting timeout on StatefulSet targets when `IsolatePods`
  restart strategy is enabled. StatefulSets now always use the `Standard`
  restart strategy, since `IsolatePods` relies on orphaning pods from their
  ReplicaSet, which does not apply to StatefulSets.
- Update RMQ splitting logic so if there are 2 entries with the same name but
  different cluster then both clusters will have a split queue and forwarder
  up.
- When splitting multiple Kafka topics on the same workload, the target pod now
  restarts only once instead of once per topic. All topics share a single
  `MirrordClusterWorkloadPatchRequest`.
- kafka ignore transient auth errors

## 3.157.2 - 2026-04-28

No significant changes.


## 3.157.1 - 2026-04-24

No significant changes.


## 3.157.0 - 2026-04-24


### Added

- Support for composite environment variables (with value_pattern regex) and
  multi source connection parameters in db branching.
- Support for pure value credentials in db branching.


### Fixed

- DB branch credentials from a literal value or Secret reference now override
  the local env vars correctly, even when the target pod does not set them.
- Leftover sessions for clusters that do not exist in the registry no longer
  crash the operator during cleanup.

## 3.156.0 - 2026-04-19


### Changed

- Bump mirrord version to 2.203.1
- Removed the 15 minute MAX limit on database branch `ttl_secs`.

## 3.155.0 - 2026-04-16


### Changed

- Adapt text of operator to use mirrord sessions
- Preview environments now ignore the config option
  `feature.network.incoming.http_filter.ports` to prevent accidentally stealing
  traffic without a filter. This means that all HTTP filters now
  unconditionally apply to all intercepted ports.
- Replaced the purple accent line under the operator dashboard's AppBar with a
  subtle top highlight and a theme-appropriate bottom border, matching the
  direction in the session monitor.


### Fixed

- Fixed MySQL branch database init failing on older MySQL images (e.g. 5.7) by
  building `mysql-branch-init` against glibc 2.17.
- Fixed SQS splitting in multi-cluster failing when the queue filter uses a
  wildcard (`*`).
- Fixed bogus error logs printed when preview environments are not disabled.

## 3.154.0 - 2026-04-09


### Changed

- Preview environments no longer override the labels/annotations from the
  target's pod spec.
  As a side effect of this change, pods created by preview environments will
  never be in the "Ready" state, this is intentional.
- mirrord-agent pods and ephemeral container specs now use both `command` and
  `args` instead of just `command`. This is to enable allowlisting with
  [WorkloadAllowlists](https://docs.cloud.google.com/kubernetes-engine/docs/reference/crds/workloadallowlist).

## 3.153.0 - 2026-04-07


### Added

- Branch database status now includes the created pod name.
- Preview env support for Multi Cluster.


### Fixed

- ArgoCD does not like empty array values so we remove them when generating the
  schema.
- Preview environments can now target StatefulSets that contain `volumeMount`s
  in their spec.
- Removed default values in chart templates that makes ArgoCD sync failure
- Value in helm chart of tls.apiService.insecureSkipTLSVerify can cause
  perpetual diff in ArgoCD sync fixed
- some Helm compatability issues resolved

## 3.152.0 - 2026-03-31


### Added

- Added Support for RabbitMQ splitting. This includes new `queueType` for
  `MirrordWorkloadQueueRegistry` resource and new `MirrordPropertyList`
  resource to save connection and queue properties.


## 3.151.2 - 2026-03-29

No significant changes.


## 3.151.1 - 2026-03-29

No significant changes.


## 3.151.0 - 2026-03-28


### Added

- Added operator errors reporting. Error events retention period can be
  configured with `.server.retention.operatorErrors` (defaults to 30 days).
  Error reports are accessible at  `/api/v1/reports/errors`.
- Preview environments now support DB branching and queue splitting.
- Preview environments now support `feature.network.incoming.ignore_ports`.
- Preview environments now support environment variable manipulation
  (`feature.env`).
- Support for Copy Target for Multi Cluster.


### Changed

- OpenAPI schema is generated once and serves an etag to allow caching
- Removed API group and resource pairs that don't exist from cluster role
  template.


### Fixed

- Fixed a minor issue where `license.key` is read for CI counting purpose even
  for regular sessions.
- Fixed incorrect YAML indentation for extraVolumes in the mirrord-operator
  deployment template.
- Fixed protobuf openapi endpoint failing due to schema change introduced.


## 3.150.0 - 2026-03-26


### Added

- Operator now sends agent spawn errors in telemetry.
- Support setting labels/annotations on preview pods using
  operator.preview.labels and operator.preview.annotations in the chart.


## 3.149.0 - 2026-03-21


### Added

- Dashboard tables now have resizable columns using @metalbear/ui.
- Add support for JQ-powered header filters
- Preview environment's usage is now collected through our telemetry system
- Preview environments now respect the cluster's incoming traffic policies.
- Session keys will now be injected into HTTP responses as well (in addition to
  requests).


### Changed

- Add mirrord for ci when license-server is not set up, by using the metalbear
  backend.
- Added a CI flag in operator session event to differentiate regular sessions
  from CI sessions. As a result, CI runners who are identified by
  MIRRORD_CI_API_KEY will not be counted as active users.
- In SQS, the session key header will now be called `mirrord-key` (instead of
  `X-Mirrord-Key`) to match injected header names in regular HTTP
  requests/responses and HTTP conventions.
- Preview environment's incoming traffic router is now more robust, reutilizing
  mirrord's existing `intproxy` logic instead of reimplementing it from
  scratch.


### Fixed

- Fixed detection of message attributes in FIFO SNS envelopes.


## 3.148.2 - 2026-03-10


### Fixed

- Fixed issues with Kafka splitting using the librdkafka client.

## 3.148.1 - 2026-03-10


### Fixed

- MySQL branch init fails when tables have foreign key dependencies.

## 3.148.0 - 2026-03-09


### Added

- Added integration with the Java Kafka sidecar.

## 3.147.0 - 2026-03-05


### Changed

- Make db branching type to be optional, now it always checks for env and
  env_from.


### Fixed

- Fix env_from for db params.
- Fixed some bogus warning and error logs.

## 3.146.0 - 2026-03-04


### Added

- SQS splitting now supports message attribute injection using the existing key
  feature.


## 3.145.0 - 2026-03-02


### Added

- You can now update the configuration and/or image of an existing preview
  environment session by re-running `preview start` with the same key, target,
  image registry and image repository.
- The `feature.preview.ttl_mins`/`--ttl` setting for preview environments now
  accepts the `"infinite"` string value, which makes the session live
  indefinitely until being manually stopped.


### Fixed

- Fixed an issue where session message processing could get stuck, leaving 
  a persistent port lock that could not be released without restarting 
  the mirrord operator.


## 3.144.0 - 2026-02-26


### Added

- Support for jq filters for SQS splitting.


### Changed

- Provide better guidance on what to do on port lock conflict


## 3.143.1 - 2026-02-24


### Fixed

- Fixed a issue where operator telemetry sender strip license server address.
- Fixed preview environment's "request was dropped" error when intercepting
  incoming traffic.

## 3.143.0 - 2026-02-24


### Added

- Add multi cluster session to the CLI status.
- The `mirrord preview status` command will now show the remaining TTL of each
  preview environment session.


### Fixed

- Update reqwest dependency - should solve trusting custom TLS if user provides


## 3.142.0 - 2026-02-19


### Added

- Added more `debug` and `warn` logs when resolving librdkafka configuration
  for Kafka splitting.
- Added operator admin dashboard for visualizing utilization metrics.
- DB Branching pods limits are now configurable. Aligned all limits to be same
  for all db branching pods as baseline.
- Added feature for injecting `Mirrord-Session-Key` headers into HTTP requests.
- mirrord now supports "Preview Environments" - a new type of mirrord session
  that lives directly in the cluster and can be shared with other users.


### Changed

- Lowered log level for `TooManyRequests` errors originating from Kubernetes
  watch streams.
  Such errors are quite common at operator startup.
- Use proper LicenseKey rather than license hash for ci routes.


### Fixed

- When targeting a StatefulSet with `copy_target`, the copy will now correctly
  mount volumes from PersistentVolumeClaims, fixing failed Pod creation.


## 3.141.0 - 2026-02-16


### Added

- Added `format=json` option to `GET /api/v1/reports/usage` endpoint for admin
  dashboard metrics.
- Operator will now suspend flux resources when there's an active workload
  patch that rescales its target.


### Fixed

- Child session can become stale and never cleaned when using multi cluster.
- Multi Cluster sessions were never cleaned when they Fail.
- Single cluster session was always checking for parent session before it was
  created.


## 3.140.1 - 2026-02-12


### Fixed

- Changed duration key to `session_duration` in Session End log as documented.
- Fixed bogus warning on OTEL propagation.
- Reduced per-session memory usage.
- Added IAM auth for Multi Cluster.


## 3.140.0 - 2026-02-09

No significant changes.


## Added

- Support for new multi-cluster sessions.


## 3.139.0 - 2026-02-06


### Changed

- Add concurrent ci sessions count to `mirrord operator status`.
- operator now emits as user_id the hashed k8s user (k8s user + salt of org id)
  to analytics


### Fixed

- Fixed an issue that json log config flag is ignored.
- Fixed an issue where the operator was using wrong images for branch pod init
  containers.


## 3.138.0 - 2026-01-30


### Added

- Added agent metrics (currently only for bypassed requests) to the operator
  metrics.
- The operator now propagates OTel context (`baggage`, `traceparent` headers)
  in http requests. Also, traces and logs can now be exported to URLs given in the 
  operator environment (`OTLP_LOGS_URL`, `OTLP_TRACES_URL`).


### Changed

- SQS error logs that originate in AWS SQS service errors now include the
  original error.

## 3.137.0 - 2026-01-23


### Added

- Add support for db branches IAM.
- Added CI cert verification bypass feature in operator status feature list.
- Db Branching support for MongoDB.
- SQS splitting sessions are now recovered transparently, without restarting
  the target workload again.


### Fixed

- Added test that checks scaledown feature of `copy_target`
- Changed leader election logic to drop leadership only after all background
  tasks are done.
- Fixed potential database corruption at license server shutdown.
- Outgoing network connections will now only request reverse DNS lookups to the
  agent when the user has configured hostname-based outgoing network policies.
  This prevents reverse DNS lookup failures from spamming the logs when they
  aren't even required in the first place.
- Reduced log spam from `operator_session::ci_controller`.


## 3.136.0 - 2026-01-13


### Changed

- Allow bypassing user credentials check when the session is started from
  mirrord ci start.


### Fixed

- Fixed `Session` verbs exposed in operator's `APIResourceList` - replaced
  `list` with `deletecollection`.
- Fixed an issue where `kubectl get v1.operator.metalbear.co.targets` could
  hang indefinitely in some Kubernetes setups.
- Fixed staled copy target state cleanup.


## 3.135.1 - 2026-01-06

## 3.135.0 - 2026-01-06


### Added

- Support for limiting outgoing connections from mirrord sessions using a
  kubernetes policy has been added to the operator.
- We now log a warning every time a license with an expired issuer is loaded.


## 3.134.0 - 2025-12-24


### Added

- Added `envFrom` as a new type of connection source in DB branching.


### Changed

- Enable iptables cleanup on startup by default (instead of erroring out if
  there are leftover rules).


### Fixed

- Fixed an issue with operator's APIService availability with multiple
  replicas.


## 3.133.0 - 2025-12-21


### Added

- Added the `CiController` to the operator (handles mirrord sessions started
  with mirrord ci start). Also added ci session handling to the backend
  (license-server sql now has a new table for bookkeeping of mirrord sessions
  started from ci, and routes that interact with this table).


### Fixed

- fix openapi v2/v3 spec leading to terraform failing


## 3.132.1 - 2025-12-10


### Fixed

- The docker image will now correctly print the license upon startup

## 3.132.0 - 2025-12-10


### Added

- Added security context config for agent pod.
  [#agent-config.security-context](https://github.com/metalbear-co/operator/issues/agent-config.security-context)
- SQS splitting sessions can now be recovered and resumed after operator
  restart.
- Support for the protocol messages introduced in the newest mirrord version
  has been added, which enable using the `ftruncate` and `futimens` libc functions.

### Fixed

- Ignore `fchown` and `fchmod` requests from the layer for security reasons.
- SQS messages were filtered wrong when the body was JSON and SNS support was
  enabled.


## 3.131.0 - 2025-12-04


### Added

- Added PostgreSQL branching.
- Added a functional log in license server after each session ends.


### Changed

- Changed operator image to be distroless.
- The operator now always uses mutating webhooks for patching targets.


### Fixed

- Fixed a bug where operator was reconciling unused `MirrordClusterSession`s
  too rarely.


## 3.130.0 - 2025-11-19


### Added

- The user report generated by the `/v1/reports/usage` endpoint can now be
  aggregated by a custom key, specified by the new `primary_key` parameter.
  Check the
  [docs](https://metalbear.com/mirrord/docs/managing-mirrord/license-server#getting-a-utilisation-report-from-the-license-server)
  to learn more.


## 3.129.2 - 2025-11-13


### Added

- Added readiness probe in MySQL branch pod.


## 3.129.1 - 2025-11-05


### Fixed

- Fixed a bug where the operator was overloading the Kubernetes API server with
  PATCH requests for `MirrordClusterSessions`.


## 3.129.0 - 2025-10-29


### Added

- Added multiple data copy modes for bootstrapping a MySQL branch.


### Fixed

- Fixed a bug in the `scaledown` feature.
- Operator status not showing data for SQS split queues.


## 3.128.0 - 2025-10-21


### Added

- Added Kafka splitting info to the operator status.


### Changed

- API calls to MetalBear's servers (e.g: fetching the operator's license) now
  have a timeout, avoiding infinite hangs when the requests can't complete
  because of network settings.
- Improved logs for lingering temporary SQS queues.
- Updated the SQS empty-queue check, according to the SQS docs.


### Fixed

- Fixed a bogus error log related to `SingleWatchTask`.


## 3.127.1 - 2025-10-09


### Changed

- Improved pod stability check done during the `isolatePods` restart. Statuses
  of pod's containers are now taken into account.


## 3.127.0 - 2025-10-06


### Removed

- Removed the legacy method of exposing user session statistics with the
  `mirrordoperatoruser` resource.
  Usage can be tracked with the license server.


### Added

- Added a warning when an agent is slow to spawn (taking longer than 5
  seconds).


### Changed

- Adjusted log levels for various reconcilation logs produced by the operator.
- Changed the way how the operator creates empty MySQL database if
  a database name is given by the user. The operator now spawns the
  branch DB pod with a init container which creates the empty database.
  The new approach works around potential network policy that limits
  cross-namespace connections.


### Fixed

- Fix `OperatorSessionInfo::hostname` to return the actual hostname.


## 3.126.0 - 2025-09-16


### Added

- Added support for using an HTTP filter in traffic mirroring.


### Changed

- Create an empty database using the name in db brnaching configuration.
- `MirrordSqsSession` resources now have 4 random letters in their name.


### Fixed

- Fixed an issue where the operator was not detecting conflicts between
  identical `any_of` HTTP filters.
- `MirrordSqsSession` resources that failed to start were not being deleted.


## 3.125.0 - 2025-09-11


### Added

- Add option to disable waiting for patched pods when starting queue splitting
  sessions.

  A new global operator configuration option
  `queue_splitting_wait_for_ready_target` has been added to control whether the
  operator waits for patched pods to become ready when starting Kafka/SQS
  splitting sessions. This option defaults to `true` (maintaining existing
  behavior) but can be set to `false` to speed up session start time in
  environments where pod readiness waiting is not necessary or causes delays
  due to cluster conditions.

  Configuration:
  - Environment variable: `OPERATOR_QUEUE_SPLITTING_WAIT_FOR_READY_TARGET`
  - Command line flag: `--queue-splitting-wait-for-ready-target`
  - Default: `true` (waits for ready pods)
- Report SQS splitting metrics to prometheus.
- The license server now logs the loaded license at startup.


### Changed

- Changed MySQL database branch's maximum TTL to 15 minutes.


### Fixed

- Fixed a bogus error log related to the `TargetWatch` task.
- Fixed a bug in the ArgoCD Rollout restart logic.
- Improved the formatting for license server usage reports, including tab names
  and number display formats.


## 3.124.0 - 2025-09-02


### Added

- Add mysql branch database CR controller and support for using mysql branches
  in connections to targets and copy targets.
- Created an endpoint at `/api/v1/reports/usage` in the license server to
  provide mirrord usage reports in .xlsx format.
- This version of the operator respects the new `workloadRestartTimeout` field
  in `MirrordWorkloadQueueRegistry`, that controls the timeout for the target
  workload to restart on the first SQS session start of a target.


### Fixed

- Fixed a bug where SQS sessions were failing with a `background task
  responsible for target monitoring is dead` error.
- Fixed a bug where sessions were sometimes failing with a port lock conflict
  after reconnecting to the operator.


## 3.123.1 - 2025-09-01


### Changed

- Removed the session ID from labels of Prometheus counters.


### Fixed

- Fixed a compatability issue with openapi v2 for operator CRDs.


## 3.123.0 - 2025-08-27


### Changed

- When the operator fails to find any applicable
  `MirrordWorkloadQueueRegistry`, the returned error contains names of all
  registries found in the session namespace.


### Fixed

- Fixed labels and buckets of the `mirrord_operator_ping_latency` Prometheus
  metric.


## 3.122.0 - 2025-08-22


### Added

- Added rtt metrics (prometheus) for checking mirrord and operator latency.
- Add metrics for stolen connections and stolen requests.
- Added `OPERATOR_SQS_SPLITTING_LINGER_TIMEOUT` configuration option,
  that allows for controlling how long (in millis) SQS splits can be kept alive
  when there are no more SQS sessions,
  but the are unconsumed unfiltered messages.


### Fixed

- Fixed a bug where the operator was not handling stolen TLS messages
  correctly.
- The operator now returns more descriptive errors when it fails to spawn pod
  agents.

## 3.121.0 - 2025-08-13


### Added

- Added support for blocking `scale_down` when using `copy_target`.

## 3.120.0 - 2025-08-08


### Added

- Ignore configured containers and init containers when using the `copy_target`
  feature.

## 3.119.2 - 2025-08-06


### Changed

- Bumped mirrord dependencies version. Default mirrord-agent version is now 3.156.0.


## 3.119.1 - 2025-07-29


### Changed

- The operator now sends telemetry data to MetalBear's analytics server even if a license server is configured
  (unless telemetry is disabled).


### Fixed

- Fixed a bug in the `isolatePods` restart strategy.


## 3.119.0 - 2025-07-29


### Added

- Added support for fetching license files from Google Secret Manager
  (operator and license server).
- Added an option to configure max interval between actions taken during an
  `isolatePods` restart, with a default value of 60 seconds.
  This allows the restart to always make progress, even in case of
  frequent/persistent pod failures.
- Added support for Kafka splitting with MSK.


### Changed

- Workflow dispatched from mirrord triggers operator e2e tests and waits for it
  to finish.
- All errors from failed SQS operations now contain the name or URL of the
  queue.


### Fixed

- Fixed an issue where the operator was prematurely deleting SQS main output
  queues, failing pods in the target workload after the end of the session.


## 3.118.1 - 2025-07-22


### Added

- Added more informative logs to the code that calls MetalBear API.


### Fixed

- Improved SQS splitting code to remove the transient "registry is dirty" error
  that was occasionally failing SQS sessions' startup.


## 3.118.0 - 2025-07-16


### Added

- Added a timeout when waiting for target pods with patched environment (SQS
  session startup).
- Added support for filtering SNS messages by `Number` message attributes.


### Fixed

- Fixed a bug in the `isolatePods` restart strategy related to Argo Rollouts
  `rollouts-pod-template-hash` label.


## 3.117.0 - 2025-07-09


### Added

- Added a custom workload restart `IsolatePods` strategy, that can be enabled
  with `OPERATOR_ISOLATE_PODS_RESTART=true`.
  With this strategy, the operator will restart patched workloads by isolating
  workload pods from their replica set one by one.
  The new strategy can be used with mutating webhooks.
- Added support for selecting SQS queues environment variables with regular
  expressions.
- Added support for specifying a catch-all queue id (specified as `*` queue id)
  to SQS queue splitting. Only for SQS splits currently.

  ```json5
  {
        "split_queues": {
            // * means all queues under this target
            "*": {
                "queue_type": "SQS",
                "message_filter": {
                    "x-pg-tenant": "Aviram"
                }
            }
        }
  }
  ```
- Added support for using queue `fallback_name` in the MirrordWorkloadQueueRegistry.
  If source variables doesn't exist, fallback to that name.


### Changed

- Added validation when running license-gen to generate a new license (now we
  check for a bunch of things that would result in errors or warnings in the
  operator).
- Improved errors and warnings produced by the SQS splitting code.
- Validate a couple more things in license-gen command to avoid invalid license
  being created.


### Fixed

- Fixed issues with mirrord sessions using both the copy target and and the SQS
  splitting features.
- Improved how the operator handles client requests when no mirrord-agent is
  alive for the given session.


## 3.116.2 - 2025-07-01


### Changed

- The operator now prepares and creates copied pods in the background,
  returning an early success response from the create handler. The progress of
  the copied target is exposed in the resource status.

## 3.116.1 - 2025-06-24


### Added

- Add the default agent image version that the operator will use as a label.


### Changed

- Change incorrect license key behavior to not print the used key.
- When attempting to report metrics to Jira, levels of log messages emitted on
  failure are now improved.


## 3.116.0 - 2025-06-17


### Added

- Add session duration reporting when a session ends if a Jira webhook URL has
  been configured.
- Support for getting SQS queue names/urls from env vars that originate in
  config maps.


### Changed

- Update error and code comments about namespaced profile.


## 3.115.0 - 2025-06-10


### Changed

- Bump operator `mirrord` version to `3.144.0`.


### Fixed

- The operator now exits without unnecessary delay when it receives SIGTERM.

## 3.114.2 - 2025-06-09


### Changed

- Change `active_users` field in "Max seat count reached!" log to number
  instead of slice.


## 3.114.1 - 2025-06-04


### Fixed

- Fix issue with scaling where a target pod can remain "unavailble" from
  operator's perspective even though the target has started and is available.


## 3.114.0 - 2025-05-28


### Added

- Add config `OPERATOR_KAFKA_SPLITTING_TOPIC_FORMAT` to control Kafka splitting
  topic names.
  with default value that results in same format as previous versions
  `mirrord-tmp-{{RANDOM}}{{FALLBACK}}{{ORIGINAL_TOPIC}}`


### Changed

- Change operator to always send telemetries when using license server while
  asserting it's not configured to be our endpoint
- The operator now returns 404 response when a requested copied pod is not
  found.

## 3.113.1 - 2025-05-16


### Fixed

- Fixed an issue where the operator was not exposing the OpenAPI V2 schema.

## 3.113.0 - 2025-05-14


### Added

- Add option to toggle our `/openapi/v2` with `OPERATOR_OPENAPI_V2_ENABLE`
  environment variable, this can be useful if there is some incopetability
  issue when operator is installed. (hashicorp's terraform module can
  experiance issues with openapi v2 enabled)

## 3.112.0 - 2025-05-12


### Added

- Add alternative tracking method for license seats where the kuberentes
  username will be the counter.


### Changed

- Renamed CRD `MirrordProfile` to `MirrordClusterProfile`.


## 3.111.0 - 2025-05-06


### Added

- `runAsUser` field in copy-target containers can be configured by operator
  admin.


### Changed

- Adds a reflector for caching mirrord policies.


### Fixed

- Fixed an issue were Kafka splitting sessions without copy target could fail
  on target readiness timeout.


## 3.110.0 - 2025-04-28


### Added

- Added `OPERATOR_LICENSE_KEY` and `OPERATOR_LICENSE_PATH` values to
  operator-license-server (equivalent to `LICENSE_KEY` and `LICENSE_PATH`)
- Added an option to configure TTL for idle Kafka splits.


### Changed

- SQS filters are now case insensitive by default.


### Fixed

- Stopping SQS queue splitters faster at the end of a splitting session.


## 3.109.1 - 2025-04-17


### Fixed

- Fixed a bug where agents were getting OOM killed due a to traffic redirection loop.


## 3.109.0 - 2025-04-15


### Added

- Add initial e2e tests for license-server.
- Add new flag named `OPERATOR_LICENSE_ALLOW_SEAT_OVERAGES` that allows
  enterpirse licenses to go past defined max seats (enabled by default)
- Added support for Kafka splitting without copying the session target.


### Changed

- Names of temporary Kafka topics created during Kafka splitting now end with
  the original topic name.
- Policies can now be applied to copy-targets (based on the original target).


### Fixed

- Kafka splitting sessions now fail early when the operator is unable to create
  temporary topics.
- Put rdkafka compilation behind a feature to fix CI cmake issue.
- More correct dates for fetch of active-users to use the "last billing date"
  as from date not just "last 30 days".
- The operator now correctly removes the `operator.metalbear.co/patched`
  annotation from target workloads that are no longer patched.


## 3.108.0 - 2025-03-29


### Added

- Support for passing queue URL instead of queue name for SQS queue splitting.
- The operator now enforces mirrord profile selection, based on
  `spec.requireProfile` and `spec.profileAllowlist` fields of mirrord policies.


### Fixed

- Fixed an issue where the operator was loosing connections to agents that were
  producing large quantities of messages.
- Reintroduce the openapi v2 endoint after fix verification.


## 3.107.2 - 2025-03-19


### Changed

- Disable `/openapi/v2` route due to critical breakage.


### Fixed

- Remove pod-template-hash from copied pod (fixes copy-target when targeting
  pods).

## 3.107.1 - 2025-03-17


### Fixed

- Fixed a bug where operator was not able to revert scaledown of Rollouts using
  `workloadRef`.
- Small fix to license-server telemetry endpoint.

## 3.107.0 - 2025-03-16


### Fixed

- Session Error status was not being set, and users got a copy-target timeout
  instead of the actual SQS error.
- Add support for `/openapi/v2` with addition of several fixes to the v3
  variant.
- Fix ArgoCD warning and potentially auto sync disable not working


## 3.106.1 - 2025-03-03


### Removed

- Removed per-pod `SubjectAccessReview`s done in the operator during multi-pod
  sessions.


## 3.106.0 - 2025-02-24


### Added

- Added support for filtered stealing of HTTPS requests.


### Fixed

- Added a 30 second timeout for fetching topics metadata from a Kafka cluster.
- Agent and copy-target pod controllers no longer print unnecessary warnings
  when deleting a pod fails with 404.

## 3.105.1 - 2025-02-19


### Fixed

- Fixed issues with Rollout targets without a label selector.


## 3.105.0 - 2025-02-06


### Added

- Added support for enforcing patterns on HTTP header filters.
- Added a configuration option to set the port on which the agents accept
  operator connections.
  The port number can be set with the `OPERATOR_AGENT_PORT` environment
  variable.
- Added support for Kubernetes ReplicaSet targets.


### Fixed

- Fixed a bug where the operator was not able to communicate with the agents
  over TLS.


## 3.104.2 - 2025-01-28


### Fixed

- Fixed how the operator negotiates `mirrord-protocol` version with the client
  during targeted sessions.
- `agent.privileged` no longer affects targetless agents' pods.

## 3.104.1 - 2025-01-27


### Added

- Operator now detects conflicts between port subscriptions with HTTP filters.
  A confict occurs when an active subscription uses an HTTP filter that matches
  a superset of requests matched by the HTTP filter used by the new
  subscription request.


### Fixed

- use correct mirrord dependencies

## 3.104.0 - 2025-01-27


### Added

- Support for `rmdir`, `statfs`, `unlink` and `unlinkat` file requests.
- Support for targeting a Kubernetes `Service`.


### Fixed

- Operator now performs a subject access review on each pod targeted in a user
  session.
- Operator now correctly sends session termination reason to the client.
- Operator no longer fails to start a targetless session when the agent config
  specifies the ephemeral container agent mode. The operator now always spawns
  pod agents for targetless sessions.


## 3.103.0 - 2025-01-14


### Added

- Added policy to control file ops.
- Added policy to exclude env vars.


### Changed

- Updated DataDog dashboard to include list of users in last 30 days.

## 3.102.0 - 2025-01-02


### Added

- Allow users to use the original image for copy target pod by specifying a
  flag.
- Added support for `mkdir` and `mkdirat` hooks.


### Changed

- Added new trusted license issuer valid until January 2027.


### Fixed

- The operator now issues a warning when fetched
  `MirrordPolicy`/`MirrordClusterPolicy` contains an unknown blocked feature.


## 3.101.0 - 2024-12-12


### Added

- Added Prometheus endpoint with `/metrics` route and enabled with
  `OPERATOR_METRICS_ENABLED` environment variable

  - `mirrord_license_valid_seconds` - seconds left for current license validity
  - `mirrord_sessions_create_total` - count of created sessions, labels:
  `client_hostname`, `client_name`, `client_user` and `user_id`
  - `mirrord_sessions_duration` - histogram for session durations after they
  are ended, labels: `client_hostname`, `client_name`, `client_user` and
  `user_id`
- Added support for `MirrordClusterPolicy`.


### Fixed

- Operator no longer changes target container name when copying target.

## 3.100.0 - 2024-12-10


### Added

- Added SQS to `mirrord operator status` reporting.
- Added support for `SASL_SSL` security protocol in Kafka splitting.
- Added support for blocking `mirror` feature in `MirrordPolicy`.
- Allow Kafka properties to be set from the optional `.spec.loadFromSecret`
  field in `MirrordKafkaClientConfig`.
  By default, the operator can only read secrets in the operator's namespace
  (`mirrord` by default).


### Changed

- Update dependencies.


### Fixed

- Fixed a bug where target workload scale was not being restored after operator
  restart (`copy_target` + `scale_down`).
- Added "transaction" like behavour so ArgoCD `Application` will unpatch if any error had
  occured while patching the train.
- Eliminated SQS-operator panic when a container in the target workload does
  not have environment variables.


## 3.99.0 - 2024-11-20


### Added

- Added support for ArgoCD Application auto-sync pausing when mirrord is used
  with `scale_down` enabled.

## 3.98.1 - 2024-11-08


### Changed

- Bump dependencies [mirrord agent version]

## 3.98.0 - 2024-11-06


### Added

- Exposed user session statistics, accessible with `kubectl get
  mirrordoperatorusers -o json`.


### Changed

- Adds a new match for EOF on websocket. Plus some error logs that should help
  showing other potential websocket issues.
- Bump dependencies
- Kafka splitting component now evaluates message filters in order of session
  start. If two user filters match a certain message, this message is received
  by the user that started their mirrord session first.

## 3.97.0 - 2024-10-17


### Added

- Added a configuration option for timeout when a session has no available pod
  targets.
- Added Kafka splitting components.


### Changed

- Bumped dependencies.
- Updated rust version.


## 3.96.0 - 2024-10-13


### Added

- Added a configuration option for timeout when a session has no available pod
  targets.
- Added Kafka splitting components.


### Changed

- Bump dependencies


## 3.95.3 - 2024-10-07


### Changed

- Refactor ResolvedTarget (and many of its checks) to mirrord. Now the GET
  target here is on its way to being deprecated.


### Fixed

- Failed and completed agent pods are now correctly removed by the operator.


## 3.95.2 - 2024-09-24


### Fixed

- Fixed operator unable to handle requests with too much headers


## 3.95.1 - 2024-09-12

No significant changes.


## 3.95.0 - 2024-09-11


### Added

- More logs from the SQS-splitting operator and improved existing logs.


### Fixed

- Fixed `Operator Monthly Users` status not counting users that were last seen
  on the current day.
  [#mau](https://github.com/metalbear-co/operator/issues/mau)
- Update so `SubjectAccessReviewSpec` values that are derived from header
  fields are properly unescaped


## 3.94.0 - 2024-09-03


### Added

- Added support for using multiple HTTP filters in one session.


### Fixed

- Operator container now fails when the default client CA cannot be fetched.
- Update operator to not use Impersonation API.

  Now there is another request to check whether the user should be allowed via
  `SubjectAccessReview`.

## 3.93.0 - 2024-08-21


### Added

- Added `ReadDirBatch` support in the operator.


## 3.92.1 - 2024-08-18


### Fixed

- incompatibility with older clients
  [metalbear-co/mirrord#2675](https://github.com/metalbear-co/mirrord/issues/2675)


## 3.92.0 - 2024-08-17


### Added

- Split SQS queues between multiple mirrord sessions/users.


## 3.91.0 - 2024-08-14


### Fixed

- Update Datadog Mirrord Operator Dashboard to group and disply by user_id and
  not client_name.
- Fixed a bug where initial license fetch failure would stop the operator from
  refreshing the license.


## 3.90.0 - 2024-07-31


### Added

- Adds new env var OPERATOR_SESSION_MAX_TIME allowing the user to put a time
  limit on mirrord sessions.


### Fixed

- Fixed a bug where targetless agents were not terminating after finished
  mirrord session.

## 3.89.0 - 2024-07-31


### Changed

- Allows targeting StatefulSet without the copy_target feature, both as an
  immediate target, and as a workload ref in targeted rollouts.


## 3.88.1 - 2024-07-18


### Fixed

- Handle log messages


## 3.88.0 - 2024-07-17


### Fixed

- Operator API now lists all targetable resources for mirrord CLI versions
  >=3.109.0.


## 3.87.0 - 2024-07-12


### Changed

- Introduce enforcement of client certificates when accessing operator api's
  other than `/health`.
- Changed loglevel of instrument license verify to trace


### Fixed

- Fixed a bug where multi-pod sessions where targeting copied pods. Removed
  copied pods from target list returned by the operator.
- Fixed bogus error log that was often printed during mirrord sessions.
- Removed some unnecessary error logs.


## 3.86.0 - 2024-07-02


### Added

- Functional logs now contain unique client id produced from client's
  certificate.
- Added support for streamed HTTP responses.


## 3.85.0 - 2024-06-25


### Added

- Operator now adjusts to changes in the target state that occur during the
  mirrord session (e.g. scaling targeted deployments/rollouts, targeted
  container restarts).
- Added support for streamed HTTP requests.


### Fixed

- Operator now properly handles case where mirrord CLI uses a higher version of
  mirrord protocol.
- `ImagePullSecrets` on copied pods now respect secrets specified in the agent
  config.
- Fixed panic in a background task responsible for watching agent config file.


## 3.84.0 - 2024-06-18


### Added

- Add support for CronJob and StatefulSets with copy target


## 3.83.0 - 2024-06-12


### Removed

- Removed `pause` feature that is no longer supported in new agent versions.


### Added

- Add hot-reload for agent config file.


### Fixed

- Fix handling rollout with malformed resources spec


## 3.82.0 - 2024-06-02


### Added

- Adds Job as a possible target when copy_target feature is enabled (checked in
  mirrord).
- Add hot-reload for license file.


### Changed

- Use the labels and annotations from agent_config when creating an agent pod.
- `port-locks` subresource is deprecated. The endpoint is left for
  compatibility, but returns nothing.


### Fixed

- Make CopyTarget feature use only pods without jobs.


## 3.81.0 - 2024-05-26


### Changed

- Bump mirrord agent version to have new agent json_log config.
- Begin enforcing `max_seats` and syncing usage with cloud on online licenses.
- Update dependecies


### Fixed

- Refactor how the operator stores the active license to be a watched task. Now
  each part that access the license for information holds a receiver, instead
  of a copy of the license, and has access to the updated license.
- Fixed agent version check performed when using agent TLS connections.


## 3.80.0 - 2024-04-28


### Added

- Operator now passes the TLS cert to the agent, allowing for authentication.
- Add example of DataDog dashboard for operator.
  ([integrations/datadog/Mirrord_Operator_Dashboard.json](/integrations/datadog/Mirrord_Operator_Dashboard.json))


### Changed

- Dropped using `Job` for agents and started spawing `Pods` directly.
- Normalized events for important events.

  * Copy Target
  * Port Steal
  * Port Mirror
  * Port Release
  * Session Start
  * Session End

  |field|description|events|
  |---|---|---|
  |client_hostname|`whoami::hostname` of client|`All`|
  |client_name|`whoami::realname` of client|`All`|
  |client_user|kubenetes user of client (via k8s RBAC)|`All`|
  |http_filter|http filter for stealing http connections|`Port Steal`|
  |port|port number (if relevant)|`Port Steal` `Port Mirror` `Port Release`|
  |scale_down|if target was scaled down|`Copy Target`|
  |session_id|unique id for individial mirrord execution (base64)|`Port Steal`
  `Port Mirror` `Port Release` `Session Start` `Session End`|
  |session_duration|time in seconds of session's existance|`Session End`|
  |target|kubernetes resource targeted|`All`|
- Bump mirrord dependencies to 3.99.0
- On telemetry send error, don't drop all events - just truncate


### Fixed

- Fixed DAU/MAU counters not being reset
- Fixed the issue where stealing/mirroring from multi-pod targets would result
  in duplicate locks in operator status.


## 3.79.0 - 2024-04-14


### Added

- Add option to output logs in JSON format using `OPERATOR_JSON_LOG` = true
- Added an option to connect with the agents using TLS.


## 3.78.1 - 2024-04-10


### Changed

- Make copy job always run as user 1000
- Run operator as user to have better permissions control


### Fixed

- Reworked port locking: system-wide `PortLocks` holds all active port locks,
  layer-scoped `LayerPortLocks` owns port locks made by specific
  `LayerHandler`. Removed separate port locks info from the operator session.
- License validation now properly handles `4xx` responses from the licensing
  server. Such responses are now treated as non-transient and block usage.
- Copy job now sets terminationGracePeriodSeconds to 1 to fix cleanup issues


## 3.78.0 - 2024-04-03


### Changed

- Use the new agent config for agent-image + Bump mirrord version.


## 3.77.1 - 2024-03-06


### Fixed

- Re-add the OperatorSession event on end of session.


## 3.77.0 - 2024-03-03


### Added

- Adds operator's session management API exposed through kube. The new routes
  allows the user to `kill_all`, `kill_one` or `retain_active` sessions, and
  are (currently) available through `mirrord operator session` cli commands.


### Fixed

- Enable telemetry even if the user set OPERATOR_TELEMETRY_ENABLED to false
  (for teams licenses).
- For copy-target feature changed so creation of `pod_template` is done with
  impersonated client before copying said target to check the user is allowed
  to copy the resource.


## 3.76.2 - 2024-02-27


### Changed

- Updated dependencies


## 3.76.1 - 2024-02-27


### Changed

- Updated dependencies


## 3.76.0 - 2024-02-21


### Added

- Added support for Argo rollouts in `copy_target` feature.


### Fixed

- Remove always returning weird answer for any unmatching request, resolving
  issue when kube asks for OpenAPIv2 schema and we just say here take garbage..
- Fixed a bug where duplicate port locks where displayed in operator status
  when targeting multi-pod targets.


## 3.75.1 - 2024-02-13


### Fixed

- Fixed filtered steal port lock being displayed as mirror in operator status.
- Don't impersonate for copy job


## 3.75.0 - 2024-01-24


### Added

- Report port locks and filters in operator status


### Changed

- Make telemetry optional only on Enterprise, failing if no communication is
  done in max time (1h).
- Use the same operator certificate dates (valid_from, valid_to) for user
  certificates (the ones signed by the operator).
- User id is now produced from user certificate public key instead of
  fingerprint.


### Fixed

- Show local name and hostname in mirrord operator status
- Sidecars and init containers are now copied for deployments in copy feature


## 3.74.0 - 2024-01-16


### Added

- Extended certificate extension with additional data, exposed license
  generation methods, added license id to telemetry events


### Fixed

- Increased max headers from 100 to 1000


## 3.73.0 - 2024-01-14


### Added

- Add namespace to the session status
- Adds a LicenseExtension type that we insert into our certificates as an
  extension to distinguish between Teams and Enterprise.


### Changed

- Do not allow a teams license to be added using the local (certificate) file
  flow (reserved for enterprise license).
- Include copy targets in operator status.
- license_server url is now defined at compile-time only (either comptime env
  var, or default value).


### Fixed

- Fixed copy job idle timeout logic - the job now terminates immediately when
  the agent is unloaded.
  [#copy-job-ttl](https://github.com/metalbear-co/operator/issues/copy-job-ttl)


## 3.72.0 - 2024-01-08


### Added

- The operator checks the policies that applies to each run and blocks port
  subscription if forbidden by a policy.


### Changed

- Listing includes container targets for resources with multiple containers.


## 3.71.0 - 2024-01-05


### Added

- Added support for reading agent config from file so we can use configmap


### Changed

- Check if the license has been issued by us (also allows mock values via env
  var OPERATOR_LICENSE_ISSUER_PUBLIC_KEY). Refactored the initialization of
  OperatorLicense to take this into account.
- Change logging/tracing to only log on new/exit of functions instead of each
  enter/leave to reduce logs


## 3.70.0 - 2024-01-01


### Changed

- Operator now can run without certificate, falling back to creating
  self-signed on runtime


## 3.69.1 - 2023-12-26


### Changed

- Do not deny route access if the license check failed due to a network error.
  Implement From for converting between &LicenseError and TelemetryEvent.


### Fixed

- Fixed format of error responses. Errors are now handled properly by the
  `kube` create in the CLI.
- Add copying of init containers and side containers in copy feature
- Create copy target in configured default namespace (not kube's default
  namespace) if not specified.
- Fixed intervals of license checks.


## 3.69.0 - 2023-12-19


### Added

- Added license authorization check (via [axum
  middleware](https://docs.rs/axum/latest/axum/middleware/index.html)) to block
  unlicensed/expired licenses from accessing certain functionalities in the
  operator.


## 3.68.1 - 2023-11-30


### Fixed

- Fixed connectionid reuse when deployment with multiple pods


## 3.68.0 - 2023-11-08


### Added

- Added a `scale down` feature to the `copy target` feature.
- Added support for container selection
- Add initial implementation for `CopyTargetCrd`.


## 3.67.0 - 2023-09-18

No significant changes.


## 3.65.2 - 2023-09-11


### Changed

- When in targetless mode always use non ephemeral containers.


## 3.64.2 - 2023-09-05

No significant changes.


## 3.63.0 - 2023-08-28

No significant changes.


## 3.62.0 - 2023-08-27

No significant changes.


## 3.61.0 - 2023-08-22

No significant changes.


## 3.60.0 - 2023-08-21


### Added

- Add handling for OutOfPods error where nodes are too full.


## 3.59.0 - 2023-08-18

No significant changes.


## 3.58.0 - 2023-08-17

No significant changes.


## 3.57.1 - 2023-08-15


### Added

- Add Towncrier.
- Add startup event and alive event

