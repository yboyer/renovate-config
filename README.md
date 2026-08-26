# renovate-config

Shared Renovate preset for my repositories.

## Purpose

This repo publishes a default Renovate configuration from [`default.json`](./default.json). It is meant to be consumed via `extends` in other repositories.

## What it does

Base presets:

- [`config:recommended`](https://docs.renovatebot.com/presets-config/#configrecommended)
- [`docker:pinDigests`](https://docs.renovatebot.com/presets-docker/#dockerpindigests)
- [`:configMigration`](https://docs.renovatebot.com/presets-default/#configmigration)
- [`abandonments:recommended`](https://docs.renovatebot.com/presets-abandonments/#abandonmentsrecommended)
- [`security:minimumReleaseAgeNpm`](https://docs.renovatebot.com/presets-security/#securityminimumreleaseagenpm)
- [`:pinAllExceptPeerDependencies`](https://docs.renovatebot.com/presets-default/#pinallexceptpeerdependencies)
- [`:disableRateLimiting`](https://docs.renovatebot.com/presets-default/#disableratelimiting)
- [`:maintainLockFilesWeekly`](https://docs.renovatebot.com/presets-default/#maintainlockfilesweekly)
- [`group:recommended`](https://docs.renovatebot.com/presets-group/#grouprecommended)
- [`helpers:pinGitHubActionDigests`](https://docs.renovatebot.com/presets-helpers/#helperspingithubactiondigests)

Custom defaults:

- Uses `rangeStrategy: "bump"`
- Disables `platformAutomerge`
- Marks internal checks as success with `internalChecksAsSuccess: true`
- Requires a `minimumReleaseAge` of `3 days`
- Enables OSV vulnerability alerts
- Disables `separateMajorMinor`

Package rules:

- Automerges `patch`, `pin`, and `digest` updates
- Handles `vulnerability` updates immediately

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
