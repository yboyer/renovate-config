# renovate-config

Shared Renovate preset for my repositories.

## Purpose

This repo publishes a default Renovate configuration from [`default.json`](./default.json). It is meant to be consumed via `extends` in other repositories.

## What it does

Base presets:

- [`abandonments:recommended`](https://docs.renovatebot.com/presets-abandonments/#abandonmentsrecommended)
- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended)
- [`docker:pinDigests`](https://docs.renovatebot.com/presets-docker/#dockerpindigests)
- [`group:recommended`](https://docs.renovatebot.com/presets-group/#grouprecommended)
- [`helpers:pinGitHubActionDigests`](https://docs.renovatebot.com/presets-helpers/#helperspingithubactiondigests)
- [`security:minimumReleaseAgeNpm`](https://docs.renovatebot.com/presets-security/#securityminimumreleaseagenpm)
- [`:automergePatch`](https://docs.renovatebot.com/presets-default/#automergepatch)
- [`:configMigration`](https://docs.renovatebot.com/presets-default/#configmigration)
- [`:disableRateLimiting`](https://docs.renovatebot.com/presets-default/#disableratelimiting)
- [`:maintainLockFilesWeekly`](https://docs.renovatebot.com/presets-default/#maintainlockfilesweekly)
- [`:pinAllExceptPeerDependencies`](https://docs.renovatebot.com/presets-default/#pinallexceptpeerdependencies)
- [`:rebaseStalePrs`](https://docs.renovatebot.com/presets-default/#rebasestaleprs)
- [`:semanticCommits`](https://docs.renovatebot.com/presets-default/#semanticcommits)
- [`:semanticCommitScope(deps)`](https://docs.renovatebot.com/presets-default/#semanticcommitscopescope)
- [`:updateNotScheduled`](https://docs.renovatebot.com/presets-default/#updatenotscheduled)

Custom defaults:

- Uses `rangeStrategy: "bump"`
- Disables `platformAutomerge`
- Marks internal checks as success with `internalChecksAsSuccess: true`
- Requires a `minimumReleaseAge` of `3 days`
- Runs daily between 00:00 and 04:59 (`* 0-4 * * *`)
- Updates existing branches outside the scheduled window
- Enables OSV vulnerability alerts
- Disables `separateMajorMinor`

Custom manager:

- Scans YAML GitHub Actions workflows in `.github/workflows/` for literal Docker image references with at least one slash, such as `ghcr.io/org/image:TAG`, `registry:5000/team/image:TAG` or `namespace/image:TAG`. References may be quoted. Bare names such as `alpine:TAG` are excluded to reduce false positives.
- Matches references anywhere in the workflow, without parsing Docker options, so multiline commands need no annotation. Use this convention for literal image references; the manager does not distinguish them from similarly formatted non-image strings.
- Uses the Docker datasource and Docker versioning to update image tags and optional 64-character SHA-256 digests.

```yaml
- run: |
    docker run --rm -v "$PWD:/repo" -w /repo \
      ghcr.io/gitleaks/gitleaks:v8.24.2 \
      git --verbose --redact --no-color .
```

Package rules:

- Handles `lockFileMaintenance` updates without a Renovate release-age delay and creates their PRs immediately within the weekly maintenance schedule.
  - The merged `npmrc` applies `min-release-age=3` for ordinary updates and lock file maintenance when supported by the npm version in use. This npm option does not provide a release-age guarantee for other package managers.
- Handles vulnerabilities through `vulnerabilityAlerts`, rather than an unsupported `vulnerability` update type.
- Adds the `breaking` label to major updates
- Groups all `patch` and `minor` updates into separate PRs, except for the explicit groups below. Major updates retain inherited grouping unless an explicit group applies.
- Groups `yboyer/actions` and nested `yboyer/actions/**` updates from the `github-actions` and `custom.regex` managers, and handles them without a stability delay; the normal creation schedule still applies.
- Groups `@biomejs/biome` and `@yboyer/config` npm updates under `Biome and shared config`.
- Both explicit group rules come after the global patch/minor rules so their group names take precedence. With `separateMajorMinor: false`, their major and non-major updates can share a PR.

Vulnerability alert behavior:

- No Renovate minimum release age; creates security PRs immediately.
- Overrides the injected npm release-age filter with `npmrc: "min-release-age=0"`, while retaining `npmrcMerge: true`, so recent security fixes can be installed when generating npm lockfiles.
- PR prefix: `[SECURITY]`
- Branch topic: `{{{datasource}}}-{{{depNameSanitized}}}-vulnerability`
- Adds `security` label
- Assigns PRs to `yboyer`

## Usage

In a target repository, extend this preset from Renovate config:

```json
{
  "extends": ["github>yboyer/renovate-config"]
}
```

If you want to compose it with repo-specific settings:

```json
{
  "extends": ["github>yboyer/renovate-config"],
  "labels": ["dependencies"]
}
```

## Repo layout

- `default.json` — default shared preset consumed by Renovate
- `renovate.json` — applies the shared preset to this repository and adds the root `default.json` to the `renovate-config` manager through `managerFilePatterns`; the standard Renovate config files remain included. This setting applies only to this repository.

The `renovate-config` manager updates supported versioned preset references and tool constraints. Built-in presets and unversioned preset references do not produce dependency updates, so scanning `default.json` does not necessarily add dependencies to the Dependency Dashboard.

## Updating the preset

Edit `default.json`, commit, and push changes. Repositories extending this preset will pick up the new configuration on future Renovate runs.
