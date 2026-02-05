# .github

GitHub organization configuration for homestak-dev.

For project vision, architecture, and development guidance, see [homestak-dev/CLAUDE.md](https://github.com/homestak-dev/homestak-dev/blob/master/CLAUDE.md).

## Contents

| File | Purpose |
|------|---------|
| `profile/README.md` | Organization profile shown on github.com/homestak-dev |
| `.github/PULL_REQUEST_TEMPLATE.md` | Default PR template for all repos |

## CI/CD Patterns

### Workflow Triggers

All repos follow consistent CI patterns:

| Trigger | Action |
|---------|--------|
| Push to master | Run lint/validation |
| Pull request | Run lint/validation, block merge on failure |
| Release tag | (Future) Build and publish artifacts |

### Current Workflows

| Repo | Workflow | Purpose |
|------|----------|---------|
| ansible | ansible-lint | Lint playbooks and roles |
| bootstrap | shellcheck, bats | Lint scripts and run tests |
| homestak-dev | shellcheck, bats | Lint release.sh and run tests |
| iac-driver | pylint, pytest | Lint and test Python code |
| packer | shellcheck, bats, packer validate | Lint scripts, run tests, validate templates |
| site-config | YAML validate | Validate YAML syntax |
| tofu | tofu validate | Validate OpenTofu modules |

### Runner Configuration

- **GitHub-hosted**: `ubuntu-latest` for all lint/validation workflows
- **Self-hosted**: (Future) Required for KVM access (packer builds, integration tests)

## Branch Protection (Rulesets)

All public repos use GitHub Rulesets on master:

| Setting | Value |
|---------|-------|
| Require PR | Yes (1 approving review) |
| Required checks | Repo-specific lint workflow |
| Bypass | OrganizationAdmin (pull_request mode) |
| Auto-merge | Enabled |

PRs are created by `homestak-bot` so the operator can review and approve. Auto-merge completes the merge after approval.

See [REPO-SETTINGS.md](https://github.com/homestak-dev/homestak-dev/blob/master/docs/REPO-SETTINGS.md) in homestak-dev for full standards.

## Dependabot

Enabled on all repos for:
- GitHub Actions version updates
- Language-specific dependency updates (pip, npm where applicable)

Update schedule: Weekly

## Secret Scanning

- Push protection: Enabled
- Alert notifications: Enabled

## Adding a New Workflow

1. Create `.github/workflows/<name>.yml` in the target repo
2. Follow existing patterns (trigger on push/PR to master)
3. Use `ubuntu-latest` runner unless KVM required
4. Add required check to ruleset after first successful run

## Related Issues

- homestak-dev#13 - CI/CD strategy epic
- .github#26 - Tag validation automation
- .github#27 - Branch protection bypass policy
