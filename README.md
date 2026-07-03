# infantex-team — organization defaults

This repository holds **default community health files** for the [infantex-team](https://github.com/infantex-team) organization.

GitHub applies these files to **any org repository that does not define its own** version of the same file. They are not copied into each repo; they are injected when opening PRs/issues or viewing community health links.

> This repo must stay **public** for organization-wide PR and issue templates to work.

## Contents

| File | Purpose |
|------|---------|
| [`.github/pull_request_template.md`](.github/pull_request_template.md) | Default PR body with security checklist |
| [`security-checklist.md`](security-checklist.md) | OWASP-aligned security standard (org canonical) |
| [`SECURITY.md`](SECURITY.md) | Vulnerability reporting policy |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Default contribution guidelines |

## Security checklist adoption

| Project type | How to use |
|--------------|------------|
| **Any repo** | PR template links to [Quick PR checklist](security-checklist.md#quick-pr-checklist) |
| **Non-AIDE** | Optionally copy `security-checklist.md` → `docs/security-checklist.md` for offline / IDE rules |
| **AIDE hub** | Sync into `knowledge-base/conventions/security-checklist.md` and run `kb-index.sh` |

Copy into a local repo:

```bash
mkdir -p docs
curl -fsSL https://raw.githubusercontent.com/infantex-team/.github/main/security-checklist.md \
  -o docs/security-checklist.md
```

## Override per repository

To use a custom PR template in a single repo, add `.github/pull_request_template.md` (or `pull_request_template.md` at repo root). That repo will **stop** using the org default.

## Related

- [AIDE platform template](https://github.com/infantex-team/quick-win) — `template/knowledge-base/conventions/security-checklist.md` should stay in sync with this file
