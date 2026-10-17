# security-pipelines

Reusable DevSecOps workflows for GitHub Actions. Reference: [devsecops.binbashburns.com](https://devsecops.binbashburns.com).

| Workflow | Tool | Pinned version |
| --- | --- | --- |
| `secrets.yml` | TruffleHog | v3.97.4 (action and container) |
| `sast.yml` | Semgrep | `semgrep/semgrep:1.176.1` (digest-pinned) |
| `sbom-vuln.yml` | Syft + Grype | `anchore/sbom-action` v0.24.2, `anchore/scan-action` v7.4.2 |
| `link-check.yml` | lychee | `lychee-action` v2.9.0 (lychee v0.24.2) |
| `dast.yml` | ZAP baseline | `ghcr.io/zaproxy/zaproxy:2.17.0` |

## Quick start

Copy a template from [`examples/`](examples/), or:

```yaml
name: security

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

concurrency:
  group: security-${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: ${{ github.event_name == 'pull_request' }}

permissions: {}

jobs:
  secrets:
    uses: binbashburns/security-pipelines/.github/workflows/secrets.yml@main
    permissions:
      contents: read

  sast:
    needs: secrets
    uses: binbashburns/security-pipelines/.github/workflows/sast.yml@main
    permissions:
      contents: read
      pull-requests: write
    with:
      language: javascript,typescript
```

Findings fail the build only when you set the workflow's gate input. Tool errors and missing reports always fail.

No release tag exists yet. Pin `@main` to a commit SHA for anything you depend on.

## secrets.yml

| Input | Default | Description |
| --- | --- | --- |
| `extra-args` | `--only-verified` | Extra trufflehog args. The action already adds `--fail`. |

Scans the PR or push range; falls back to full history for new branches and force-pushes. Fails on any verified secret. Produces no artifact or summary, so read the job log.

## sast.yml

| Input | Default | Description |
| --- | --- | --- |
| `language` | `auto` | `auto`, or a comma-separated pack list: `javascript`, `typescript`, `csharp`, `ruby`, `python`, `go`, `java`, `php`. A pack list also runs `p/ci`. |
| `severity-fail` | `none` | `ERROR`, `WARNING`, `INFO`, or `none` |
| `comment-on-pr` | `true` | Post or update a PR comment |

Also fails on error-level Semgrep scan errors. Artifact: `semgrep-report`.

## sbom-vuln.yml

| Input | Default | Description |
| --- | --- | --- |
| `path` | `.` | Directory to scan |
| `severity-cutoff` | `medium` | `critical`, `high`, `medium`, `low`, `negligible` |
| `fail-build` | `false` | Fail on findings at or above the cutoff |
| `only-fixed` | `false` | Only report vulnerabilities with a fix |
| `upload-sarif` | `false` | Upload SARIF to code scanning. Needs `security-events: write`. |
| `comment-on-pr` | `true` | Post or update a PR comment |

Grype is skipped when the SBOM contains only GitHub Action PURLs. Artifacts: `sbom.cdx.json`, `grype-sarif`.

## link-check.yml

| Input | Default | Description |
| --- | --- | --- |
| `paths` | `**/*.md **/*.html` | Space-separated globs |
| `fail-on-error` | `false` | Fail on dead links |
| `create-issue` | `false` | Open, update, and close a tracking issue (push events only) |
| `site-artifact` | `""` | Built-site artifact from this run |
| `site-path` | `_site` | Where to download `site-artifact` |
| `root-dir` | `""` | Root for site-absolute links. Defaults to `site-path` with an artifact. |

Set `root-dir` when checking HTML in place, or every `/about/`-style link reports dead. The tracking issue is found by its `link-check` label. Artifact: `lychee-report`.

## dast.yml

| Input | Default | Description |
| --- | --- | --- |
| `target-url` | `http://host.docker.internal:8000` | URL to scan |
| `serve-port` | `8000` | Local static server port. `0` scans `target-url` directly. |
| `serve-path` | `.` | Directory to serve |
| `fail-on` | `none` | `high`, `medium`, `low`, or `none` |
| `comment-on-pr` | `true` | Post or update a PR comment |
| `site-artifact` | `""` | Built-site artifact from this run |

`fail-on` counts alert instances, not distinct alerts. Artifact: `zap-reports`.

## Permissions

Grant these on the calling job. A workflow that requests a scope the caller did not grant fails the run.

| Workflow | `contents` | `pull-requests` | `issues` | `security-events` |
| --- | --- | --- | --- | --- |
| `secrets.yml` | read | | | |
| `sast.yml` | read | write (`comment-on-pr`) | | |
| `sbom-vuln.yml` | read | write (`comment-on-pr`) | | write (`upload-sarif`) |
| `link-check.yml` | read | | write (`create-issue`) | |
| `dast.yml` | read | write (`comment-on-pr`) | | |

## Notes

- PR comments are skipped on fork and Dependabot PRs. Scans and gates still run.
- The workflows set no concurrency. Set it in the caller, as the examples do.
- Self-hosted runners need runner 2.327.1 or later (Node 24 actions). `sast.yml` and `dast.yml` need Docker.

## Examples

| Template | Shows |
| --- | --- |
| [`csharp.yml`](examples/csharp.yml) | secrets, `sast` (`csharp`), `sbom` |
| [`javascript.yml`](examples/javascript.yml) | Multi-pack SAST |
| [`static-site.yml`](examples/static-site.yml) | In-place link check with `root-dir`, weekly cron |
| [`jekyll.yml`](examples/jekyll.yml) | Build once, pass the site artifact to `link-check` and `dast` |

This repo runs its own workflows plus actionlint and zizmor in [`self-scan.yml`](.github/workflows/self-scan.yml).

See [CONTRIBUTING.md](CONTRIBUTING.md) and [SECURITY.md](SECURITY.md). AGPL-3.0.
