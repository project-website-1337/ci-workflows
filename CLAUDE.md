# Repository rules

Reusable GitHub Actions workflows for all client website projects. This
repository is public; never commit secrets, client names or personal data.

## Language

- Talk to the user in German.
- Write workflows, comments, commit messages and documentation in English.

## Rules

- Follow the conventions in `README.md`: actions pinned to full commit SHAs,
  downloaded tools pinned by version and SHA-256, `permissions: {}` at the
  top, inputs reach shell scripts only through `env`.
- Check every change with actionlint and zizmor (`lint-workflows.yml` runs
  both in CI); test locally with `act` where possible.
- Commit directly to `main` once the checks pass. Never add
  `Co-Authored-By` or any other AI attribution.

## Releases

Projects use the moving tag `v1`. After a backwards compatible change has
passed the self-test on `main`, move the tag (force push, ask the user
first). A breaking change gets a new major tag instead.
