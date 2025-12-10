# New Relic APM Agent Versions - Centralized Management

This repository provides a centralized Renovate preset for managing New Relic APM agent versions across Navigation node.js applications.

## Overview

Instead of each app updating to the latest New Relic agent independently, this preset ensures all apps converge to a **single approved version** defined in **versions/newrelic_versions.yaml**.

## How It Works

### 1. Central Version Definition

The approved New Relic agent version is defined in:

**`versions/newrelic_versions.yaml`**
```yaml
nodejs:
  development: "13.6.6"
```

This is the **single source of truth** for which New Relic version all Node.js apps should use.

### 2. Renovate Preset Configuration

The `renovate.json` in this repo is used as a **shareable preset** by all app repositories.

**Key behavior:**
- Apps will **only** update to the version specified in `allowedVersions` (currently `13.6.6`)
- Updates are **auto-merged** after CI passes
- Labeled with `nr-agent-update` and `centrally-managed`

### 3. App Repository Setup

Each Node.js app that should use centralized New Relic management must:

#### a. Add the preset to `package.json`:

```json
{
  "renovate": {
    "extends": [
      "github>pagerinc/renovate-config",
      "github>azradedicpager/newrelic-apm-agent-versions"
    ]
  }
}
```

#### b. Ensure `newrelic` is in dependencies:

```json
{
  "dependencies": {
    "newrelic": "12.8.1"
  }
}
```

Once configured, Renovate will automatically open a PR to update `newrelic` to the approved version (`13.6.6`).

## Quarterly Update Process

**When:** Quarterly (or as agreed by the team)

**Steps:**

### 1. Decide on New Target Version
- Review latest New Relic agent releases
- Choose new target version (e.g., `13.7.0`)

### 2. Update This Repository

**File 1: `versions/newrelic_versions.yaml`**

Update the version:
```yaml
nodejs:
  development: "13.7.0"  # Updated from 13.6.6
```

**File 2: `renovate.json`**

Update the `allowedVersions` constraint:
```json
{
  "packageRules": [
    {
      "matchPackagePatterns": ["^newrelic$"],
      "allowedVersions": "13.7.0",  // Updated from 13.6.6
      ...
    }
  ]
}
```

### 3. Commit and Push
```bash
git add versions/newrelic_versions.yaml renovate.json
git commit -m "chore: update New Relic target version to 13.7.0"
git push origin main
```

### 4. Wait for Renovate to Propagate

Within 1-2 hours (based on Renovate schedule), Renovate will:
- Scan all app repos using this preset
- Open PRs to update `newrelic` from old version to `13.7.0`
- **Automerge** PRs after CI passes (for minor/patch updates)

### 5. Handle Different App Types

**Actively Developed Apps:**
- Renovate PRs will automerge
- Updates will deploy through normal CI/CD (dev → QA → stage → prod)
- No manual intervention needed

**Non-Actively Developed Apps:**
- Renovate PRs will still automerge
- **SRE must manually deploy:**
  1. Create LP ticket for deployment
  2. Deploy to QA → run smoke tests
  3. Deploy to stage → SDET runs regression tests
  4. Deploy to prod after approval

## Integration with Pager CI/CD

Once a Renovate PR merges:

1. **Dev Environment**
   - CI builds Docker image with new New Relic agent baked in
   - Deployed to dev (via label/pipeline trigger)
   - Engineering tests run

2. **QA Promotion**
   - Image promoted to QA (`qa-*` tag)
   - E2E and regression tests run
   - Failures → alert and revert

3. **Stage Promotion**
   - Same image deployed to stage
   - Regression/test validation
   - Failures → patch required

4. **Production**
   - Same tested image promoted to production
   - **No rebuild** - exact artifact flows through all environments

## Version Constraints Explained

### `allowedVersions: "13.6.6"`

This tells Renovate:
- **Only** propose updates to version `13.6.6`
- Ignore any other versions (even if newer)
- Ensures all apps converge to the same approved version


With centralized version:
- **Single approved version** across all apps
- **Controlled rollout** quarterly or as needed
- **Consistent observability** in New Relic UI

## Renovate Schedule

From `pagerinc/renovate-config`:
```json
"schedule": [
  "after 6pm and before 8am on every weekday",
  "every weekend"
]
```

Renovate only runs during off-hours to avoid disrupting development.

## Observability

After apps deploy with updated agent:
- **New Relic APM UI** will show updated agent version
- **Agent Groundskeeper** should reflect the new version
- All apps should report the same updated agent version

## Troubleshooting

### Renovate isn't creating PRs

1. Check if Renovate GitHub App is enabled for the app repo
2. Verify `renovate` config in app's `package.json`
3. Check Renovate logs in the app repo's "Dependency Dashboard" issue

### PR was created but not automerged

1. Check if CI passed
2. Look for branch protection rules requiring reviews
3. Verify `automerge: true` in the preset

### Wrong version in PR

1. Check `allowedVersions` in this repo's `renovate.json`
2. Ensure it matches `versions/newrelic_versions.yaml`
3. PRs may take 1-2 hours to reflect changes after updating this repo

## Additional Resources

- [Renovate Documentation](https://docs.renovatebot.com/)
- [Renovate Tutorial](https://github.com/renovatebot/tutorial)
- [Pager Renovate Config](https://github.com/pagerinc/renovate-config)
- [New Relic Node.js Agent - npm](https://www.npmjs.com/package/newrelic)
- [New Relic Node.js Agent Releases](https://github.com/newrelic/node-newrelic/releases)

## Contact

For questions or issues with this preset:
- **SRE (Odin) Team** - for quarterly updates, deployment coordination, and Renovate configuration questions
