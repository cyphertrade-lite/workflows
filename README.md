# cyphertrade-lite/workflows

Reusable GitHub Actions workflows for **CypherLite** services.

| Workflow | When | What it does |
|----------|------|--------------|
| `skill-release-ci.yml` | on `gh release` | build image → push to `ghcr.io/cyphertrade-lite/<service>` (no deploy) |
| `feature-ci.yml` | on push / PR | `cargo fmt --check`, `clippy -D warnings`, `cargo test` |
| `frontend-ci.yml` | on push / PR | web-only Expo: `lint`, `tsc --noEmit`, `jest`, `expo export --platform web` (PWA + Telegram Mini App) |

**Deploy is manual.** CI only builds and pushes the image to GHCR. To deploy:

```bash
ssh root@167.233.107.69 'cd /root/cypherlite && docker compose pull <service> && docker compose up -d <service>'
```

## Wiring a service

`.github/workflows/release-ci.yml` in the service repo:

```yaml
name: release-ci
on:
  release:
    types: [published]
jobs:
  release:
    uses: cyphertrade-lite/workflows/.github/workflows/skill-release-ci.yml@master
    with:
      service: <service-name>
    secrets: inherit
```

`.github/workflows/feature-ci.yml`:

```yaml
name: feature-ci
on: [push, pull_request]
jobs:
  check:
    uses: cyphertrade-lite/workflows/.github/workflows/feature-ci.yml@master
    secrets: inherit
```

`.github/workflows/frontend-ci.yml` (web-only Expo apps — PWA + Telegram Mini App):

```yaml
name: frontend-ci
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
jobs:
  web:
    uses: cyphertrade-lite/workflows/.github/workflows/frontend-ci.yml@master
    with:
      NODE_VERSION: "22"
    secrets: inherit
```

## Secrets

- **None required for deploy** — deploy is manual.
- GHCR push uses the built-in `GITHUB_TOKEN` (`packages: write`); no extra secret needed.
