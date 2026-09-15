# Changelog — `armature-messaging`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

## [0.5.0] - 2026-09-15

### Changed

- **Breaking:** `async-nats` (0.49 → 0.50) and `mq-bridge` (0.3 → 0.4) are public dependencies — `NatsBackend::jetstream`, the `From<async_nats::…>` error conversions and `MqBridge::channel` expose their types — so the upgrade is breaking and the minor moves.
- Bumped dependencies: `tokio` 1.52→1.53, `uuid` 1.23→1.26, `lapin` 4.10→4.11, `async-nats` 0.49→0.50, `aws-config` 1.8→1.12, `aws-sdk-sqs` 1.93→1.109, `aws-sdk-sns` 1.94→1.111, `mq-bridge` 0.3→0.4. `aws-sdk-sqs`/`aws-sdk-sns` are held one release back from `cargo upgrade`'s picks (1.110/1.112) to stay on `aws-smithy-types` 1.6.x, matching the workspace-wide smithy resolution. No source changes were needed: none of the bumped crates' 0.x/1.x version jumps touched an API surface this crate uses, and `mq-bridge` 0.4 still exposes every feature (`kafka`, `amqp`, `nats`, `mqtt`, `http`, `full`) that this crate's `mq-bridge-*` features forward to.
- A direct `aws-smithy-types >=1.6.3, <1.7` requirement keeps a fresh resolve on the SDK releases held back above; without it the resolver picks `aws-sdk-*`/`aws-runtime` releases that need `aws-smithy-types` 1.7 and fail to build against `aws-config` 1.12.
- AWS SDK dependencies no longer enable their default features, dropping the SDK's legacy hyper-0.14 client and its `h2 0.3` (RUSTSEC-2026-0258); the hyper-1 `default-https-client` and `rt-tokio` (plus `sigv4a`/`http-1x` where the SDK enabled them by default) are kept.
- The MSRV CI job also checks `--all-features`, so the optional AWS SDK dependencies are built on the MSRV toolchain.

## [0.4.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.
