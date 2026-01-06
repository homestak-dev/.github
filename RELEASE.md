# Release Methodology

Standard process for homestak releases across all repositories.

## Repository Dependency Order

Releases must follow this order (downstream depends on upstream):

1. **site-config** - Configuration and secrets
2. **tofu** - VM provisioning modules
3. **ansible** - Host configuration playbooks
4. **bootstrap** - Installation and CLI
5. **packer** - Custom images (requires build host)
6. **iac-driver** - Orchestration (depends on all above)

## Release Phases

### Phase 1: Pre-flight

- [ ] All PRs merged to main branches
- [ ] Working trees clean (`git status` on all repos)
- [ ] No existing tags for target version
- [ ] CHANGELOGs current
- [ ] Packer build smoke test on designated build host

```bash
# Quick status check
for repo in site-config tofu ansible bootstrap packer iac-driver; do
  echo "=== $repo ==="
  cd ~/homestak-dev/$repo
  git status --short
  git tag -l "v0.*" | tail -3
done
```

### Phase 2: CHANGELOGs

Update CHANGELOGs in dependency order. Each should have:

```markdown
## vX.Y.Z-rcN - YYYY-MM-DD

### Features
- Item (closes #N)

### Bug Fixes
- Item

### Changes
- Item
```

### Phase 3: Tags

Create and push tags in dependency order:

```bash
# For each repo
git tag -a v0.X.0-rcN -m "Release v0.X.0-rcN"
git push origin v0.X.0-rcN
```

### Phase 4: Packer Images

Build images on a host with QEMU/KVM support:

```bash
# Prerequisites (one-time on build host)
curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/main/install.sh | bash
homestak install packer

# Build and fetch images
cd ~/homestak-dev/iac-driver
./run.sh --scenario packer-build-fetch --remote <build-host-ip>

# Images downloaded to /tmp/packer-images/
```

**Build hosts:** father, mother (any PVE host with `homestak install packer`)

**Dev workflow:** Use `packer-sync-build-fetch` to test local packer changes before committing.

### Phase 5: GitHub Releases

Create releases in dependency order:

```bash
# Source-only repos
gh release create v0.X.0-rcN --title "v0.X.0-rcN" --notes "See CHANGELOG.md" --repo homestak-dev/<repo>

# Packer (with image assets)
gh release create v0.X.0-rcN \
  --title "v0.X.0-rcN" \
  --notes "See CHANGELOG.md" \
  --repo homestak-dev/packer \
  /tmp/packer-images/debian-12-custom.qcow2 \
  /tmp/packer-images/debian-13-custom.qcow2
```

### Phase 6: Verification

```bash
for repo in site-config tofu ansible bootstrap packer iac-driver; do
  echo "=== $repo ==="
  gh release view v0.X.0-rcN --repo homestak-dev/$repo --json tagName,assets --jq '{tag: .tagName, assets: (.assets | length)}'
done
```

Expected: All repos have releases, packer has 2 assets.

### Phase 7: After Action Report

Document on the release issue:

| Section | Content |
|---------|---------|
| Planned vs Actual | Timeline comparison |
| Deviations | What changed and why |
| Issues Discovered | Problems found during release |
| Artifacts Delivered | Final release inventory |

### Phase 8: Retrospective

Document on the release issue:

| Section | Content |
|---------|---------|
| What Worked Well | Keep doing these |
| What Could Improve | Process improvements |
| Suggestions | Specific ideas for next release |
| Open Questions | Decisions deferred |

## Version Numbering

**Pre-release:** `v0.X.0-rcN` (current phase)
- No backward compatibility guarantees
- RC increments for post-tag additions (rc1 → rc2)
- Delete/recreate tags acceptable for pre-releases

**Stable:** `v1.0.0+` (future)
- Semantic versioning
- Backward compatibility expectations
- No tag recreation

## Release Issue Template

Each release should have a coordination issue in `.github` repo:

```markdown
## Summary
Planning for vX.Y.Z-rcN release.

## Scope
### repo-name
- [ ] Feature/fix description (#issue)

## Validation
- [ ] E2E test results
- [ ] Manual verification

## Deferred to Future Release
- repo#N - Description

## Release Checklist
### CHANGELOGs
- [ ] site-config ... iac-driver

### Tags & Releases
- [ ] site-config ... iac-driver

### Post-Release
- [ ] After Action Report
- [ ] Retrospective

---
**Status:** Planning | In Progress | Complete
```

## Path to Stable

Before graduating from pre-release to v1.0.0:

- [ ] User-facing documentation (beyond CLAUDE.md)
- [ ] CI/CD pipeline for automated E2E tests
- [ ] All "known issues" resolved or documented
- [ ] Core workflows fully working
- [ ] Security audit (secrets, SSH, API tokens)
- [ ] Bootstrap UX polished

## References

- [v0.6.0-rc1 Release](.github/issues/4) - First release using this methodology
- [v0.7.0-rc1 Planning](.github/issues/6) - Next release
