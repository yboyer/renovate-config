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

Custom defaults:

- Uses `rangeStrategy: "bump"`
- Disables `platformAutomerge`
- Marks internal checks as success with `internalChecksAsSuccess: true`
- Requires a `minimumReleaseAge` of `3 days`
- Enables OSV vulnerability alerts
- Disables `separateMajorMinor`

Package rules:

- Handles `vulnerability` and `lockFileMaintenance` updates immediately
  - Lock file maintenance remains safe: the configured `npmrc` applies `min-release-age=3`, so npm excludes packages released within the last three days.
- Adds the `breaking` label to major updates
- Groups `yboyer/actions` GitHub Actions and regex updates, and handles them immediately without a stability delay
- Groups updates for `@biomejs/biome` and `@yboyer/config` as `Biome + config`

Vulnerability alert behavior:

- No minimum release age
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

## Updating the preset

Edit `default.json`, commit, and push changes. Repositories extending this preset will pick up the new configuration on future Renovate runs.
