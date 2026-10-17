# Security policy

## Reporting

Use [private vulnerability reporting](https://github.com/binbashburns/security-pipelines/security/advisories/new). Do not open a public issue.

Include the workflow file, the inputs and trigger involved, and the impact. A minimal caller workflow that reproduces it helps most.

This is a personal project. Expect acknowledgement within 7 days and an assessment within 30, best-effort.

## Supported refs

`main` only, until a release tag exists.

## Scope

In scope:

- Command or template injection through a workflow input
- Token or credential exposure in artifacts, logs, summaries, PR comments, or container mounts
- Requesting or leaking more `GITHUB_TOKEN` scope than needed
- A scan that passes silently: a missing or malformed report treated as clean, or a gate that does not fail
- Unpinned or weakly pinned actions and container images

Out of scope:

- Bugs, false positives, and false negatives in Semgrep, Syft, Grype, TruffleHog, ZAP, or lychee. Report those upstream, unless this repo's invocation makes them worse.
- Anything that requires write access to the repository running the workflows.
