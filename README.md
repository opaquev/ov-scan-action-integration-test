# `ov-scan-action-integration-test`

Daily integration smoke test of the published [`opaquev/ov-scan-action`](https://github.com/opaquev/ov-scan-action) release.

> **If this badge is red, the published action is broken — do NOT integrate it until smoke is green again.**

[![smoke](https://github.com/opaquev/ov-scan-action-integration-test/actions/workflows/smoke.yml/badge.svg?branch=main)](https://github.com/opaquev/ov-scan-action-integration-test/actions/workflows/smoke.yml)

## What this is

A separate consumer repo that pins `opaquev/ov-scan-action@v1` (and a parallel job pinning the immutable `v1.0.0` SHA) and runs the action against real fixtures on a daily cron + on every push to `main`.

It catches failure modes that pre-merge CI in the action repo can't:

| Failure mode | Caught by |
|---|---|
| The `@v1` floating tag points at a broken SHA | This repo |
| The `v1.0.0` SHA-pin works but `@v1` was force-moved | This repo (compares both jobs) |
| `releases.opaquevault.com/v0.10.0/` returns 404 (CDN dropped the artifact) | This repo |
| jedisct1 minisign release artifacts changed | This repo |
| GitHub Actions runner image drift breaks bash 3.2 / coreutils assumptions | This repo |
| The action repo's `entrypoint.sh` regressed but tests pass | Pre-merge CI |
| The action's bats contracts are wrong | Pre-merge CI |

This repo only catches **(consume-from-marketplace correctness)**. The action's own `tests/contracts.bats` covers (logic correctness).

## What it tests

`smoke.yml` runs three matrix-variant jobs:

1. **`clean-repo`** — `fixtures/clean-repo/` has no credentials. The action must exit 0 with `findings-count=0`.
2. **`dirty-repo`** — `fixtures/dirty-repo/` has fixture credentials with `XXXFAKE_` prefix. The action must exit non-zero with `findings-count > 0`.
3. **`version-floor`** — passes `min-ov-version: v999.0.0` (impossibly high). The action must exit 2 with the version-floor error.

Each variant runs on:
- `ubuntu-latest` (linux_amd64)
- `ubuntu-24.04-arm` (linux_arm64)
- `macos-latest` (darwin_arm64)

Each variant also runs in two pin modes:
- `@v1` (floating tag — what most casual users will use)
- `@<v1.0.0-sha>` (immutable SHA pin — the recommended production pattern)

## Schedule

| Trigger | Frequency |
|---|---|
| `push` to `main` | every commit |
| `schedule` cron | daily, 12:00 UTC |
| `workflow_dispatch` | on-demand from the Actions tab |

## On failure

When smoke fails, the workflow opens (or updates) a tracking issue in this repo:

- Title: `smoke failed on <date> — <runner>:<variant>`
- Body: link to the failing run + the assertion that broke

The same alert posts as a comment on the **action repo's** main branch via `peter-evans/create-or-update-comment` so a maintainer sees it without having to subscribe to two repos.

## How to use this for your own validation

If you're integrating `ov-scan-action` and want a sanity check:

1. Fork this repo
2. Update the `smoke.yml` matrix to match your runner expectations
3. Add your own `fixtures/your-repo-shape/` directory with representative content
4. Run `workflow_dispatch` to confirm the action behaves as you expect

You can also just refer to the [latest run on this badge above](https://github.com/opaquev/ov-scan-action-integration-test/actions/workflows/smoke.yml) — if it's green, the published action works.

## Trust model

This repo:
- Has only one maintainer (`@huntrock17`) — same trust model as the action repo
- Does NOT pin `actions/checkout@v4` — it pins to a SHA per the action repo's own SHA-pinning gospel
- Does NOT receive any secrets — the smoke workflow runs with `permissions: contents: read` only

If this repo is compromised, the worst an attacker can do is hide a smoke failure (false-green). They cannot inject a malicious `ov-scan-action` because the SHA-pinned job is the source of truth — a forged `v1.0.0` SHA would fail with "tag does not match expected commit" via GitHub's resolver.

## See also

- [`opaquev/ov-scan-action`](https://github.com/opaquev/ov-scan-action) — the action being tested
- Action's [v1.0.0 release](https://github.com/opaquev/ov-scan-action/releases/tag/v1.0.0)
- Action's [threat model](https://github.com/opaquev/ov-scan-action/blob/main/docs/threat-model.md)
