# Release Process

This document defines the release process for the homestak-dev organization.
All repositories follow a coordinated "release train" approach with shared versions.

## Repositories

| Repository | Purpose | Special Handling |
|------------|---------|------------------|
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestration engine, E2E tests | Runs validation gate |
| [ansible](https://github.com/homestak-dev/ansible) | Playbooks and roles | - |
| [tofu](https://github.com/homestak-dev/tofu) | OpenTofu modules | - |
| [packer](https://github.com/homestak-dev/packer) | Debian cloud images | Artifact uploads (.qcow2) |
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point, `homestak` CLI | Clones other repos at install |
| [site-config](https://github.com/homestak-dev/site-config) | Configuration template | Template repo - users fork |

## Versioning Strategy

### Semantic Versioning

All repositories follow semantic versioning: `v{MAJOR}.{MINOR}.{PATCH}[-{PRERELEASE}]`

| Version Component | When to Increment |
|-------------------|-------------------|
| MAJOR | Breaking changes to CLI, config format, or cross-repo interfaces |
| MINOR | New features, new scenarios, new roles |
| PATCH | Bug fixes, documentation updates |
| PRERELEASE | `-rc1`, `-rc2`, etc. for release candidates |

### Release Train

All 6 repos share the same version number for coordinated releases.
This simplifies compatibility - if repos are at the same version, they work together.

### When to Release

- **RC (Release Candidate)**: When feature work is complete and needs validation
- **Stable**: After RC has been tested in production environments
- **Hotfix**: Critical bug fixes (increment PATCH, skip RC)

## E2E Validation Gate

**REQUIRED**: The `nested-pve-roundtrip` scenario must pass before any release.

### What It Validates

```
Outer PVE Host (pve)
└── VM 99913 (nested-pve) - Inner PVE
    ├── Debian 13 + Proxmox VE installation
    └── VM 99901 (test) - Test VM
        └── Debian 12, cloud-init boot
```

This validates the full stack:
1. tofu provisions VM on outer PVE
2. ansible installs PVE on Debian 13
3. tofu provisions VM inside nested PVE
4. SSH connectivity verified through jump host
5. Clean destruction in reverse order

### Running the Validation

```bash
cd /opt/homestak/iac-driver
./run.sh --scenario nested-pve-roundtrip --host pve --verbose
```

**Expected duration**: ~8.5 minutes

Reports are generated in `iac-driver/reports/YYYYMMDD-HHMMSS.{passed|failed}.md`

### Failure Handling

If validation fails:
1. Do NOT proceed with release
2. Fix the issue in the relevant repo
3. Re-run validation
4. Only proceed after full PASS

## CHANGELOG Workflow

Each repo maintains a `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/) format.

### During Development

Add entries under the `## Unreleased` section:

```markdown
## Unreleased

### Features
- Add foo capability (#123)

### Bug Fixes
- Fix bar issue (#124)

### Breaking Changes
- Remove deprecated baz option
```

### At Release Time

1. Create a new version header with date
2. Move all Unreleased content under the new header
3. Add a fresh Unreleased section

**Before:**
```markdown
## Unreleased

### Features
- Add foo capability (#123)
```

**After:**
```markdown
## Unreleased

## v0.6.0 - 2026-01-15

### Features
- Add foo capability (#123)
```

### Commit Message

```
docs: update CHANGELOG for v0.6.0 release
```

## Release Checklist

### Pre-Release (1-2 days before)

- [ ] All feature PRs merged to main/master
- [ ] No open blockers or critical issues
- [ ] CHANGELOG.md has entries for all changes
- [ ] Run `nested-pve-roundtrip` validation on a clean system
- [ ] Verify all 6 repos are at same base version

### Release Day - Per Repository

Repeat for each repo in dependency order (see [Release Dependency Order](#release-dependency-order)):

- [ ] Update CHANGELOG.md (Unreleased → vX.Y.Z - YYYY-MM-DD)
- [ ] Commit: `docs: update CHANGELOG for vX.Y.Z release`
- [ ] Create and push tag: `git tag vX.Y.Z && git push origin vX.Y.Z`
- [ ] Create GitHub Release with release notes
- [ ] Verify release appears on GitHub

### Special: packer Artifacts

After tagging packer:

- [ ] GitHub Actions creates the release automatically
- [ ] Build images locally: `./build.sh` (select both Debian 12 and 13)
- [ ] Upload artifacts:
  ```bash
  gh release upload vX.Y.Z images/debian-12/debian-12-custom.qcow2
  gh release upload vX.Y.Z images/debian-13/debian-13-custom.qcow2
  ```
- [ ] Verify images downloadable: `gh release download vX.Y.Z --pattern '*.qcow2'`

### Special: site-config

This is a template repository that users fork.

- [ ] Ensure no site-specific examples remain (no real hostnames/IPs)
- [ ] Test fork workflow: fork → `make setup` → `make decrypt` works
- [ ] Update any version references in templates

### Post-Release Verification

- [ ] Test fresh bootstrap installation:
  ```bash
  curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/master/install.sh | bash
  ```
- [ ] Run `homestak status` to verify all components installed
- [ ] Run `homestak scenario simple-vm-roundtrip` on test host

## Tagging and GitHub Releases

### Tag Format

```
vMAJOR.MINOR.PATCH[-PRERELEASE]

Examples:
v0.5.0-rc1    # Release candidate
v0.5.0        # Stable release
v0.5.1        # Patch release
```

### Creating Tags

```bash
# Annotated tag with message
git tag -a v0.6.0 -m "Release v0.6.0"

# Push tag to remote
git push origin v0.6.0
```

### GitHub Release Creation

#### Manual (most repos)

```bash
gh release create v0.6.0 \
  --title "v0.6.0" \
  --notes "See CHANGELOG.md for details"
```

#### Automated (packer only)

The packer repo has `.github/workflows/release.yml` that:
1. Triggers on `v*` tag push
2. Creates GitHub release with template body
3. Generates release notes from commits

After automated release, manually upload built images.

### Release Notes Template

```markdown
## Highlights

- [Key feature or change 1]
- [Key feature or change 2]

## What's Changed

[Copy from CHANGELOG.md]

## Upgrade Notes

[Any migration steps if applicable]

## Full Changelog

https://github.com/homestak-dev/{repo}/compare/v0.5.0...v0.6.0
```

## Release Dependency Order

```
                      iac-driver (E2E gate)
                            │
           ┌────────────────┼────────────────┐
           │                │                │
        ansible           tofu           packer
           │                │                │
           └────────────────┼────────────────┘
                            │
                        bootstrap
                            │
                       site-config
```

### Why This Order?

1. **iac-driver first**: Runs E2E validation, orchestrates all tools
2. **ansible, tofu, packer**: Core tools, can be released in parallel
3. **bootstrap**: References other repos, needs their tags to exist
4. **site-config last**: Template repo, least dependent on others

### Parallel Release (Advanced)

If confident in coordination, ansible/tofu/packer can be released simultaneously:

```bash
for repo in ansible tofu packer; do
  (cd /path/to/$repo && git tag v0.6.0 && git push origin v0.6.0) &
done
wait
```

## Hotfix Process

For critical bugs that need immediate release.

### When to Hotfix

- Security vulnerabilities
- Data corruption bugs
- Complete feature breakage

### Hotfix Steps

1. Create fix on main/master branch
2. Increment PATCH version (v0.5.0 → v0.5.1)
3. Update CHANGELOG with hotfix section:
   ```markdown
   ## v0.5.1 - 2026-01-10

   ### Security
   - Fix credential exposure in log output (#999)
   ```
4. Run abbreviated validation (simple-vm-roundtrip if nested-pve not affected)
5. Tag and release affected repo(s) only
6. Hotfixes do NOT require full release train

### Cross-Repo Hotfixes

If hotfix affects multiple repos, release them in dependency order.
All hotfixed repos get the same patch version.

## Process Flow

```
 Development                    Pre-Release                      Release
┌───────────┐                  ┌────────────┐                  ┌──────────┐
│           │   Feature        │            │                  │          │
│ Unreleased│ ──complete────▶  │  E2E Gate  │ ──PASS────────▶  │   Tag    │
│ CHANGELOG │                  │ nested-pve │                  │  vX.Y.Z  │
│           │                  │ roundtrip  │                  │          │
└───────────┘                  └────────────┘                  └──────────┘
      │                              │                              │
      │                              │ FAIL                         │
      │                              ▼                              ▼
      │                        ┌──────────┐                   ┌──────────┐
      └◀─────────────────────  │   Fix    │                   │  GitHub  │
         (add fix to           │  Issues  │                   │ Release  │
          Unreleased)          └──────────┘                   └──────────┘
                                                                   │
                                                                   ▼
                                                             ┌──────────┐
                                                             │ packer:  │
                                                             │  Upload  │
                                                             │  .qcow2  │
                                                             └──────────┘
                                                                   │
                                                                   ▼
                                                             ┌──────────┐
                                                             │  Verify  │
                                                             │bootstrap │
                                                             │ install  │
                                                             └──────────┘
```

## Automation Roadmap

### Currently Automated

| Repo | Automation |
|------|------------|
| packer | GitHub Actions creates release on tag push |

### Future Automation

| Task | Priority | Notes |
|------|----------|-------|
| Release workflow for all repos | High | Template from packer |
| CHANGELOG validation | Medium | Verify version header exists before release |
| E2E validation on PR | Medium | Run nested-pve-roundtrip in CI |
| Cross-repo version sync | Low | Validate all repos at same version |

### Proposed Release Workflow Template

```yaml
# .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*']

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validate CHANGELOG
        run: |
          VERSION=${GITHUB_REF_NAME#v}
          grep -q "## v$VERSION" CHANGELOG.md || exit 1

      - name: Create Release
        uses: softprops/action-gh-release@v2
        with:
          generate_release_notes: true
```

## Version History

| Version | Status | Date | Notes |
|---------|--------|------|-------|
| v0.5.0-rc1 | Pre-release | 2026-01-04 | First coordinated release |
