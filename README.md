# infantex-team — organization defaults

This repository holds **default community health files** for the [infantex-team](https://github.com/infantex-team) organization.

GitHub applies these files to **any org repository that does not define its own** version of the same file. They are not copied into each repo; they are injected when opening PRs/issues or viewing community health links.

> This repo must stay **public** for organization-wide PR and issue templates to work.

## Contents

| File | Purpose |
|------|---------|
| [`.github/pull_request_template.md`](.github/pull_request_template.md) | Default PR body with security checklist |
| [`.github/workflows/security-scan.yml`](.github/workflows/security-scan.yml) | PR security scan (Semgrep, Trivy, Gitleaks) — **must be copied** into each repo |
| [`security-checklist.md`](security-checklist.md) | OWASP-aligned security standard (org canonical) |
| [`SECURITY.md`](SECURITY.md) | Vulnerability reporting policy |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | Default contribution guidelines |

> **Note:** Unlike PR/issue templates, GitHub does **not** auto-apply workflows from this org `.github` repo. You must add `security-scan.yml` to every repository that should run it.

## Add the security scan to a new repository

### 1. Prerequisites

- Repository default branch is named **`main`** (the workflow only runs on PRs targeting `main`). If your default branch is `master` or something else, edit the `branches:` list in the workflow after copying.
- GitHub Actions is enabled for the repo: **Settings → Actions → General → Actions permissions** → allow actions.

### 2. Copy the workflow file

From your new repo’s root:

```bash
mkdir -p .github/workflows
curl -fsSL https://raw.githubusercontent.com/infantex-team/.github/main/.github/workflows/security-scan.yml \
  -o .github/workflows/security-scan.yml
git add .github/workflows/security-scan.yml
git commit -m "ci: add security scan workflow"
git push
```

Or create `.github/workflows/security-scan.yml` in the GitHub UI and paste the contents from [security-scan.yml](.github/workflows/security-scan.yml).

Then search the file for `TODO:` and adjust before merging:

| TODO | What to change |
|------|----------------|
| Protected branch | `on.pull_request.branches` → this repo’s default/protected branch |
| Semgrep configs | `--config` language packs → this repo’s stack ([registry](https://semgrep.dev/r)) |
| Trivy severity | `severity` / `exit-code` if you need a softer ramp-up |
| Gitleaks license | Add `GITLEAKS_LICENSE` secret for org repos (next step) |

### 3. Add the Gitleaks license secret (org repos)

`gitleaks-action` requires a free license when the repo belongs to a **GitHub organization** (not needed for personal accounts).

1. Get a free key at [gitleaks.io](https://gitleaks.io).
2. Add a secret named **`GITLEAKS_LICENSE`**:
   - **One repo:** **Settings → Secrets and variables → Actions → New repository secret**
   - **Whole org (recommended):** org **Settings → Secrets and variables → Actions → New organization secret**, and grant access to the repos that need it

`GITHUB_TOKEN` is provided automatically — you do not need to create it. Semgrep runs in Community Edition with no App token.

### 4. Verify it works

1. Open a pull request into **`main`**, or go to **Actions → Security Scan → Run workflow** (manual run).
2. Confirm three jobs finish: **sast** (Semgrep), **dependency-scan** (Trivy), **secrets-scan** (Gitleaks).
3. Optional: under **Settings → Branches → Branch protection rules** for your protected branch, enable **Require status checks to pass before merging**, then select the Security Scan jobs (`sast`, `dependency-scan`, `secrets-scan`).

### What the workflow does

| Job | Tool | Fails the PR when… |
|-----|------|--------------------|
| `sast` | Semgrep (`p/default`, `p/owasp-top-ten`, `p/typescript`) | Findings match those rules |
| `dependency-scan` | Trivy FS | CRITICAL/HIGH vulns (unfixed only ignored) |
| `secrets-scan` | Gitleaks | Secrets detected in git history |

Dependabot PRs are skipped. Jobs cancel older runs on the same branch when a newer commit is pushed.

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
