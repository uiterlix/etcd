

Previous change logs can be found at [CHANGELOG-3.0](https://github.com/coreos/etcd/blob/master/CHANGELOG-3.0.md).


## [v3.1.15](https://github.com/coreos/etcd/releases/tag/v3.1.15) (TBD 2018-05)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.14...v3.1.15) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- Purge old [`*.snap.db` snapshot files](https://github.com/coreos/etcd/pull/7967).
  - Previously, etcd did not respect `--max-snapshots` flag to purge old `*.snap.db` files.
  - Now, etcd purges old `*.snap.db` files to keep maximum `--max-snapshots` number of files on disk.

### Go

- Compile with [*Go 1.8.7*](https://golang.org/doc/devel/release.html#go1.8).


## [v3.1.14](https://github.com/coreos/etcd/releases/tag/v3.1.14) (2018-04-24)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.13...v3.1.14) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Metrics, Monitoring

- Add [`etcd_server_is_leader`](https://github.com/coreos/etcd/pull/9587) Prometheus metric.

### etcd server

- Add [`--initial-election-tick-advance`](https://github.com/coreos/etcd/pull/9591) flag to configure initial election tick fast-forward.
  - By default, `--initial-election-tick-advance=true`, then local member fast-forwards election ticks to speed up "initial" leader election trigger.
  - This benefits the case of larger election ticks. For instance, cross datacenter deployment may require longer election timeout of 10-second. If true, local node does not need wait up to 10-second. Instead, forwards its election ticks to 8-second, and have only 2-second left before leader election.
  - Major assumptions are that: cluster has no active leader thus advancing ticks enables faster leader election. Or cluster already has an established leader, and rejoining follower is likely to receive heartbeats from the leader after tick advance and before election timeout.
  - However, when network from leader to rejoining follower is congested, and the follower does not receive leader heartbeat within left election ticks, disruptive election has to happen thus affecting cluster availabilities.
  - Now, this can be disabled by setting `--initial-election-tick-advance=false`.
  - Disabling this would slow down initial bootstrap process for cross datacenter deployments. Make tradeoffs by configuring `--initial-election-tick-advance` at the cost of slow initial bootstrap.
  - If single-node, it advances ticks regardless.
  - Address [disruptive rejoining follower node](https://github.com/coreos/etcd/issues/9333).

### Go

- Compile with [*Go 1.8.7*](https://golang.org/doc/devel/release.html#go1.8).


## [v3.1.13](https://github.com/coreos/etcd/releases/tag/v3.1.13) (2018-03-29)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.12...v3.1.13) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Improved

- Adjust [election timeout on server restart](https://github.com/coreos/etcd/pull/9415) to reduce [disruptive rejoining servers](https://github.com/coreos/etcd/issues/9333).
  - Previously, etcd fast-forwards election ticks on server start, with only one tick left for leader election. This is to speed up start phase, without having to wait until all election ticks elapse. Advancing election ticks is useful for cross datacenter deployments with larger election timeouts. However, it was affecting cluster availability if the last tick elapses before leader contacts the restarted node.
  - Now, when etcd restarts, it adjusts election ticks with more than one tick left, thus more time for leader to prevent disruptive restart.

### Metrics, Monitoring

- Add missing [`etcd_network_peer_sent_failures_total` count](https://github.com/coreos/etcd/pull/9437).

### Go

- Compile with [*Go 1.8.7*](https://golang.org/doc/devel/release.html#go1.8).


## [v3.1.12](https://github.com/coreos/etcd/releases/tag/v3.1.12) (2018-03-08)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.11...v3.1.12) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- Fix [`mvcc` "unsynced" watcher restore operation](https://github.com/coreos/etcd/pull/9297).
  - "unsynced" watcher is watcher that needs to be in sync with events that have happened.
  - That is, "unsynced" watcher is the slow watcher that was requested on old revision.
  - "unsynced" watcher restore operation was not correctly populating its underlying watcher group.
  - Which possibly causes [missing events from "unsynced" watchers](https://github.com/coreos/etcd/issues/9086).
  - A node gets network partitioned with a watcher on a future revision, and falls behind receiving a leader snapshot after partition gets removed. When applying this snapshot, etcd watch storage moves current synced watchers to unsynced since sync watchers might have become stale during network partition. And reset synced watcher group to restart watcher routines. Previously, there was a bug when moving from synced watcher group to unsynced, thus client would miss events when the watcher was requested to the network-partitioned node.

### Go

- Compile with [*Go 1.8.7*](https://golang.org/doc/devel/release.html#go1.8).


## [v3.1.11](https://github.com/coreos/etcd/releases/tag/v3.1.11) (2017-11-28)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.10...v3.1.11) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- [#8411](https://github.com/coreos/etcd/issues/8411),[#8806](https://github.com/coreos/etcd/pull/8806) backport "mvcc: sending events after restore"
- [#8009](https://github.com/coreos/etcd/issues/8009),[#8902](https://github.com/coreos/etcd/pull/8902) backport coreos/bbolt v1.3.1-coreos.5

### Go

- Compile with [*Go 1.8.5*](https://golang.org/doc/devel/release.html#go1.8).


## [v3.1.10](https://github.com/coreos/etcd/releases/tag/v3.1.10) (2017-07-14)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.9...v3.1.10) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Added

- Tag docker images with minor versions.
  - e.g. `docker pull quay.io/coreos/etcd:v3.1` to fetch latest v3.1 versions.

### Go

- Compile with [*Go 1.8.3*](https://golang.org/doc/devel/release.html#go1.8).
  - Fix panic on `net/http.CloseNotify`


## [v3.1.9](https://github.com/coreos/etcd/releases/tag/v3.1.9) (2017-06-09)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.8...v3.1.9) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- Allow v2 snapshot over 512MB.

### Go

- Compile with [*Go 1.7.6*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.8](https://github.com/coreos/etcd/releases/tag/v3.1.8) (2017-05-19)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.7...v3.1.8) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.7](https://github.com/coreos/etcd/releases/tag/v3.1.7) (2017-04-28)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.6...v3.1.7) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.6](https://github.com/coreos/etcd/releases/tag/v3.1.6) (2017-04-19)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.5...v3.1.6) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- Fill in Auth API response header.
- Remove auth check in Status API.

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.5](https://github.com/coreos/etcd/releases/tag/v3.1.5) (2017-03-27)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.4...v3.1.5) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd server

- Fix raft memory leak issue.
- Fix Windows file path issues.

### Other

- Add `/etc/nsswitch.conf` file to alpine-based Docker image.

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.4](https://github.com/coreos/etcd/releases/tag/v3.1.4) (2017-03-22)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.3...v3.1.4) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.3](https://github.com/coreos/etcd/releases/tag/v3.1.3) (2017-03-10)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.2...v3.1.3) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd gateway

- Fix `etcd gateway` schema handling in DNS discovery.
- Fix sd_notify behaviors in `gateway`, `grpc-proxy`.

### gRPC Proxy

- Fix sd_notify behaviors in `gateway`, `grpc-proxy`.

### Other

- Use machine default host when advertise URLs are default values(`localhost:2379,2380`) AND if listen URL is `0.0.0.0`.

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.2](https://github.com/coreos/etcd/releases/tag/v3.1.2) (2017-02-24)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.1...v3.1.2) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### etcd gateway

- Fix `etcd gateway` with multiple endpoints.

### Other

- Use IPv4 default host, by default (when IPv4 and IPv6 are available).

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.1](https://github.com/coreos/etcd/releases/tag/v3.1.1) (2017-02-17)

See [code changes](https://github.com/coreos/etcd/compare/v3.1.0...v3.1.1) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Go

- Compile with [*Go 1.7.5*](https://golang.org/doc/devel/release.html#go1.7).


## [v3.1.0](https://github.com/coreos/etcd/releases/tag/v3.1.0) (2017-01-20)

See [code changes](https://github.com/coreos/etcd/compare/v3.0.0...v3.1.0) and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md) for any breaking changes. **Again, before running upgrades from any previous release, please make sure to read change logs below and [v3.1 upgrade guide](https://github.com/coreos/etcd/blob/master/Documentation/upgrades/upgrade_3_1.md).**

### Improved

- Faster linearizable reads (implements Raft [read-index](https://github.com/coreos/etcd/pull/6212)).
- v3 authentication API is now stable.

### Breaking Changes

- Deprecated following gRPC metrics in favor of [go-grpc-prometheus](https://github.com/grpc-ecosystem/go-grpc-prometheus).
  - `etcd_grpc_requests_total`
  - `etcd_grpc_requests_failed_total`
  - `etcd_grpc_active_streams`
  - `etcd_grpc_unary_requests_duration_seconds`

### Dependency

- Upgrade [`github.com/ugorji/go/codec`](https://github.com/ugorji/go) to [**`ugorji/go@9c7f9b7`**](https://github.com/ugorji/go/commit/9c7f9b7a2bc3a520f7c7b30b34b7f85f47fe27b6), and [regenerate v2 `client`](https://github.com/coreos/etcd/pull/6945).

### Security, Authentication

See [security doc](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/security.md) for more details.

- SRV records (e.g., infra1.example.com) must match the discovery domain (i.e., example.com) if no custom certificate authority is given.
  - `TLSConfig.ServerName` is ignored with user-provided certificates for backwards compatibility; to be deprecated.
  - For example, `etcd --discovery-srv=example.com` will only authenticate peers/clients when the provided certs have root domain `example.com` as an entry in Subject Alternative Name (SAN) field.

### etcd server

- Automatic leadership transfer when leader steps down.
- etcd flags
  - `--strict-reconfig-check` flag is set by default.
  - Add `--log-output` flag.
  - Add `--metrics` flag.
- etcd uses default route IP if advertise URL is not given.
- Cluster rejects removing members if quorum will be lost.
- Discovery now has upper limit for waiting on retries.
- Warn on binding listeners through domain names; to be deprecated.
- v3.0 and v3.1 with `--auto-compaction-retention=10` run periodic compaction on v3 key-value store for every 10-hour.
  - Compactor only supports periodic compaction.
  - Compactor records latest revisions every 5-minute, until it reaches the first compaction period (e.g. 10-hour).
  - In order to retain key-value history of last compaction period, it uses the last revision that was fetched before compaction period, from the revision records that were collected every 5-minute.
  - When `--auto-compaction-retention=10`, compactor uses revision 100 for compact revision where revision 100 is the latest revision fetched from 10 hours ago.
  - If compaction succeeds or requested revision has already been compacted, it resets period timer and starts over with new historical revision records (e.g. restart revision collect and compact for the next 10-hour period).
  - If compaction fails, it retries in 5 minutes.

### client v3

- Add `SetEndpoints` method; update endpoints at runtime.
- Add `Sync` method; auto-update endpoints at runtime.
- Add `Lease TimeToLive` API; fetch lease information.
- replace Config.Logger field with global logger.
- Get API responses are sorted in ascending order by default.

### etcdctl v3

- Add `lease timetolive` command.
- Add `--print-value-only` flag to get command.
- Add `--dest-prefix` flag to make-mirror command.
- `get` command responses are sorted in ascending order by default.

### gRPC Proxy

- Experimental gRPC proxy feature.

### Other

- `recipes` now conform to sessions defined in `clientv3/concurrency`.
- ACI has symlinks to `/usr/local/bin/etcd*`.

### Go

- Compile with [*Go 1.7.4*](https://golang.org/doc/devel/release.html#go1.7).

