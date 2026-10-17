# Contributing

## Verify

```bash
docker run --rm -v "$PWD:/repo:ro" -w /repo \
  rhysd/actionlint:1.7.12@sha256:b1934ee5f1c509618f2508e6eb47ee0d3520686341fec936f3b79331f9315667 \
  -color .github/workflows/*.yml examples/*.yml

docker run --rm -v "$PWD:/workspace:ro" -w /workspace -e GH_TOKEN="$(gh auth token)" \
  ghcr.io/zizmorcore/zizmor:1.30.0 --persona=regular --min-severity=low --format=plain -- \
  . examples/*.yml

docker run --rm -v "$PWD:/workspace:ro" -w /workspace -e GH_TOKEN="$(gh auth token)" \
  ghcr.io/zizmorcore/zizmor:1.30.0 --persona=pedantic --min-severity=low --format=plain -- \
  . examples/*.yml
```

All three must exit 0. actionlint needs both globs; any file argument turns off its auto-discovery.

Every input is declared under both `workflow_call` and `workflow_dispatch`, and actionlint does not check that they match:

```bash
python3 - <<'PY'
import yaml, pathlib
for p in sorted(pathlib.Path('.github/workflows').glob('*.yml')):
    t = yaml.safe_load(p.read_text())[True]
    if 'workflow_call' in t and 'workflow_dispatch' in t:
        a = t['workflow_call'].get('inputs') or {}
        b = t['workflow_dispatch'].get('inputs') or {}
        print(f'{p.name}: {"OK" if a == b else "DRIFT"}')
PY
```

Then dispatch the changed workflows on GitHub. Local runs do not replace a hosted runner.

## Rules

- Keep logic inline in the YAML. The checkout is the caller's repo, so scripts in this repo do not exist at runtime.
- Tool crashes and missing or malformed reports fail. Findings fail only through the gate input. Gates parse the report; do not use a scanner's own `--error` or `--fail`.
- PR comments use `continue-on-error: true` and are skipped on fork and Dependabot PRs.
- Pass inputs through `env:` and quote them. No `${{ inputs.* }}` in `run:`.
- No `pull_request_target`. Keep `persist-credentials: false`.
- Do not set concurrency in a reusable workflow. `github.workflow` resolves to the caller.

## Updated by hand

- Images started with `docker run` in `sast.yml` and `dast.yml`. zizmor cannot see them. Registry tags have no leading `v`.
- Actions in `examples/`. Dependabot cannot reach that directory.
- `python3` heredocs. actionlint does not lint them.
- When pinning an action, resolve annotated tags to the commit SHA, not the tag object.

## Pull requests

- Conventional commit subjects: `fix(secrets): ...`, `feat(dast): ...`, `docs: ...`, `chore(deps): ...`
- One change per PR.
- Update the README tables when inputs or permissions change.
- Link the workflow runs you dispatched.

## Releases

1. Merge to `main` with checks green.
2. Tag `vX.Y.Z`, then move `vX` to the same commit.
3. Update `examples/` refs on a major bump.

Removing or renaming an input, changing a default, or requiring a new permission is a breaking change.
