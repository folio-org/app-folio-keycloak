# New Application Repository Setup

This file guides you through the initial setup after creating a repository from the template. Delete this file when setup is complete.

## Prerequisites

- [ ] Access to the `folio-org` GitHub organization
- [ ] Your GitHub team slug (e.g., `erm`, `spitfire`, `acquisitions`)
- [ ] List of backend modules (`mod-*`) for your application
- [ ] List of UI modules (`folio_*`) for your application (if any)
- [ ] Known dependencies on other applications (`app-*`)

**Important:** When creating this repository from the template, you must check **"Include all branches"** in the GitHub UI. This ensures both the `snapshot` and `master` branches are copied. If you missed this step, you will need to manually create the missing branch.

## Step 1: Configure the `snapshot` branch

```bash
git checkout snapshot
```

### Edit `pom.xml`

| Placeholder | Replace with | Example |
|---|---|---|
| `__APPNAME__` | Application name (without `app-` prefix) | `agreements` |
| `__APP_DESCRIPTION__` | Brief application description | `Application descriptor for the FOLIO Agreements application` |

### Edit `application.template.json`

Replace placeholder entries with your actual modules, UI modules, and dependencies:

- **dependencies** — other `app-*` applications yours depends on. Use `-SNAPSHOT` version constraints (e.g., `^1.0.0-SNAPSHOT`). Remove the `dependencies` array entirely if your application has no dependencies.
- **modules** — backend modules. Keep `"version": "latest"` and `"preRelease": "only"`.
- **uiModules** — frontend modules. Same version settings. Remove the `uiModules` array if your application has no UI modules.

### Edit `README.md`

Replace `__APPNAME__` and `__APP_DESCRIPTION__`. Update module tables with your actual module names.

### Commit and push

```bash
git add pom.xml application.template.json README.md
git commit -m "Configure application for snapshot branch"
git push
```

## Step 2: Configure the `master` branch

```bash
git checkout master
```

### Edit `pom.xml`

Same `__APPNAME__` and `__APP_DESCRIPTION__` replacements as the snapshot branch.

### Edit `application.template.json`

Same modules and dependencies as the snapshot branch, but with release-specific settings:

- **dependencies** — use stable version constraints (e.g., `^1.0.0`), set `"preRelease": "false"`
- **modules** — use semver constraints (e.g., `^1.0.0`), set `"preRelease": "false"`
- **uiModules** — same as modules

### Edit `README.md`

Same replacements as the snapshot branch.

### Edit `.github/update-config.yml`

Replace `__TEAM_NAME__` with your GitHub team slug in the `pr_reviewers` section.

**Important:** The `enabled` field is set to `false` by default. Do not change this until the Kitfox team has completed infrastructure setup (Step 3).

### Edit `.github/CODEOWNERS`

Replace `__TEAM_NAME__` with your GitHub team slug.

### Commit and push

```bash
git add pom.xml application.template.json README.md .github/update-config.yml .github/CODEOWNERS
git commit -m "Configure application for master branch"
git push
```

## Step 3: Request infrastructure setup

Contact the **Kitfox team** to:

- Set `snapshot` as the default branch
- Configure branch rulesets
- Set up GitHub App integration (EUREKA_CI_APP_ID / EUREKA_CI_APP_KEY secrets)
- Enable merge queue

Set the `SLACK_NOTIF_CHANNEL` repository variable: navigate to Settings → Secrets and variables → Actions → Variables, and create the variable with your team's Slack channel ID.

## Step 4: Enable automated updates

Once the Kitfox team confirms infrastructure setup is complete:

1. On the `master` branch, edit `.github/update-config.yml`
2. Change `enabled: false` to `enabled: true`
3. Commit and push

## Step 5: Request platform inclusion

If your application needs to be included in FOLIO environments:

- Request inclusion in the snapshot environment build cycle
- Coordinate with Kitfox for `platform-lsp/platform-descriptor.json` updates

## Step 6: Clean up

Delete this `SETUP.md` file from the `master` branch:

```bash
git checkout master
git rm SETUP.md
git commit -m "Remove setup guide after initial configuration"
git push
```

## Verification checklist

- [ ] `snapshot` branch: update-scheduler workflow triggers within 20 minutes and resolves module versions
- [ ] `snapshot` branch: `application.lock.json` is generated with resolved module versions
- [ ] `snapshot` branch: application descriptor is published to FAR
- [ ] `master` branch: `eureka-ci / validate-application` check passes
- [ ] Slack notifications are received in your team channel

## Workflows

This repository ships four workflows (update scheduler, feature build, release version, release
preparation). See [`.github/docs/workflows.md`](.github/docs/workflows.md) for what each does, its
inputs, and links to the underlying reusable workflows.
