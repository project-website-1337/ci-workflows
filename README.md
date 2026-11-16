# ci-workflows

Reusable GitHub Actions workflows shared by all client website projects.
A fix or improvement made here reaches every project that references the
workflow.

## Usage

Add this file to a project as `.github/workflows/ci.yml`:

```yaml
name: CI

on:
  pull_request:
  push:
    branches: [main]

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

permissions: {}

jobs:
  ci:
    permissions:
      contents: read
    uses: project-website-1337/ci-workflows/.github/workflows/ci.yml@v1
```

### Project contract

The project must provide these `package.json` scripts:

| Script      | Purpose                                         |
| ----------- | ----------------------------------------------- |
| `lint`      | Lint and format check                           |
| `typecheck` | TypeScript type check                           |
| `test`      | Unit tests                                      |
| `build`     | Production build                                |
| `test:e2e`  | Playwright tests (only if `run-e2e` is enabled) |

It must also define `packageManager` in `package.json`, commit
`pnpm-lock.yaml`, and declare the Node.js version in `.nvmrc`.

### Inputs

| Input                 | Default                                         | Description                                          |
| --------------------- | ----------------------------------------------- | ---------------------------------------------------- |
| `working-directory`   | `.`                                             | Directory that contains `package.json`               |
| `node-version-file`   | `.nvmrc`                                        | Node.js version file, relative to working directory  |
| `run-e2e`             | `true`                                          | Run Playwright end-to-end tests                      |
| `playwright-browsers` | `chromium`                                      | Space-separated browsers to install                  |
| `audit-level`         | `high`                                          | Minimum severity that fails the dependency audit     |
| `run-semgrep`         | `true`                                          | Run Semgrep static security analysis                 |
| `semgrep-config`      | `p/typescript p/react p/nextjs p/owasp-top-ten` | Space-separated Semgrep rule sets                    |

## What the pipeline does

Two jobs run in parallel:

- **verify**: install, lint, typecheck, unit tests, build, end-to-end tests.
  The steps share one job on purpose: GitHub bills each job rounded up to a
  full minute and each job would repeat the dependency install.
- **security**: secret scan of the full git history (gitleaks), audit of
  production dependencies (`pnpm audit`), static analysis (Semgrep).

Caches: pnpm store, Next.js build cache, Playwright browsers.

## Conventions

- Third-party actions are pinned to full commit SHAs with the version as a
  trailing comment. Downloaded tools are pinned to a version and verified by
  SHA-256 checksum.
- Workflows start from `permissions: {}`; each job grants only what it needs.
- Untrusted values (inputs) reach shell scripts through `env`, never through
  `${{ }}` interpolation inside `run`.

## Releases

Projects reference the moving major tag `v1`. After merging a backwards
compatible change, move the tag:

```sh
git tag --force v1 && git push --force origin v1
```

Breaking changes (renamed inputs, changed script contract) get a new major
tag (`v2`), so existing projects keep working until they are migrated.

## Self-test

`.github/workflows/self-test.yml` lints all workflows with actionlint and
zizmor and runs the reusable pipeline against the minimal project in
`test/fixture`. The fixture covers the pipeline plumbing; the Playwright path
is exercised by the projects that use it.
