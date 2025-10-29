---
title: Operator Changelog
date: 2023-08-15T00:00:00.000Z
lastmod: 2025-10-29T00:00:00.000Z
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

