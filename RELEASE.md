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

**Unified versioning:** All repos get the same version tag on each release, even if unchanged. This simplifies tracking - "homestak v0.8" means all repos at v0.8.

## Release Phases

### Phase 1: Pre-flight

- [ ] Review RELEASE.md for methodology updates since last release
- [ ] All PRs merged to main branches
- [ ] Working trees clean (`git status` on all repos)
- [ ] No existing tags for target version
- [ ] CLAUDE.md files reflect current state (see below)
- [ ] Organization README current (`.github/profile/README.md`)
- [ ] CHANGELOGs current
- [ ] Packer build smoke test on designated build host

#### CLAUDE.md Review (per .github#5)

Verify each repo's CLAUDE.md reflects current architecture:
- [ ] site-config - schema, defaults, file structure
- [ ] iac-driver - scenarios, actions, ConfigResolver
- [ ] tofu - modules, variables, workflow
- [ ] packer - templates, build workflow
- [ ] ansible - playbooks, roles, collections
- [ ] bootstrap - CLI, installation

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
## vX.Y - YYYY-MM-DD

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
git tag -a v0.X -m "Release v0.X"
git push origin v0.X
```

### Phase 4: Validation

Run integration tests before creating releases:

```bash
# Full nested-pve roundtrip (~8 min on father)
./run.sh --scenario nested-pve-roundtrip --host father

# Or constructor + destructor separately with context persistence
./run.sh --scenario nested-pve-constructor --host father -C /tmp/nested-pve.ctx
# ... verify inner PVE, check test VM ...
./run.sh --scenario nested-pve-destructor --host father -C /tmp/nested-pve.ctx

# Quick validation: simple-vm-roundtrip (~1 min)
./run.sh --scenario simple-vm-roundtrip --host father
```

**Attach report to release issue as proof.** Reports are generated in `iac-driver/reports/`:
- `YYYYMMDD-HHMMSS.passed.md` - Human-readable summary
- `YYYYMMDD-HHMMSS.passed.json` - Machine-readable details

### Phase 5: Packer Images

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

#### Image Versioning and `latest` Tag

For unified versioning, **every release includes packer images** attached to the version release, and `latest` is updated to point to the new version. This ensures "homestak v0.X" is complete and self-contained.

**If images were rebuilt this release:**
```bash
# Build and fetch new images
./run.sh --scenario packer-build-fetch --remote <build-host-ip>
```

**If images unchanged (reuse from previous release):**
```bash
# Fetch from current latest
mkdir -p /tmp/packer-images && cd /tmp/packer-images
gh release download latest --repo homestak-dev/packer --pattern '*.qcow2'
```

**Then update `latest` tag and release:**
```bash
# Update latest tag to point to new release
cd ~/homestak-dev/packer
git tag -f latest v0.X
git push origin latest --force

# Delete and recreate latest release with same assets
gh release delete latest --repo homestak-dev/packer --yes
gh release create latest --prerelease \
  --title "Latest Images" \
  --notes "Rolling release - points to v0.X" \
  --repo homestak-dev/packer \
  /tmp/packer-images/debian-12-custom.qcow2 \
  /tmp/packer-images/debian-13-custom.qcow2
```

See packer#5 for the `latest` tag convention details.

**Override:** Pin to specific version with `--packer-release v0.X` or `site.yaml`.

### Phase 6: GitHub Releases

Create releases in dependency order. **Use `--prerelease` flag until v1.0.**

```bash
# Source-only repos
gh release create v0.X --prerelease --title "v0.X" --notes "See CHANGELOG.md" --repo homestak-dev/<repo>

# Packer (with image assets)
gh release create v0.X --prerelease \
  --title "v0.X" \
  --notes "See CHANGELOG.md" \
  --repo homestak-dev/packer \
  /tmp/packer-images/debian-12-custom.qcow2 \
  /tmp/packer-images/debian-13-custom.qcow2
```

### Phase 7: Verification

```bash
for repo in site-config tofu ansible bootstrap packer iac-driver; do
  echo "=== $repo ==="
  gh release view v0.X --repo homestak-dev/$repo --json tagName,assets --jq '{tag: .tagName, assets: (.assets | length)}'
done
```

Expected: All repos have releases, packer has 2 assets.

### Phase 8: After Action Report (same day)

**Complete immediately after release while details are fresh.** Delaying AAR/retro results in lost insights.

Document on the release issue:

| Section | Content |
|---------|---------|
| Planned vs Actual | Timeline comparison |
| Deviations | What changed and why |
| Issues Discovered | Problems found during release |
| Artifacts Delivered | Final release inventory |

### Phase 9: Retrospective (same day)

Document on the release issue:

| Section | Content |
|---------|---------|
| What Worked Well | Keep doing these |
| What Could Improve | Process improvements |
| Suggestions | Specific ideas for next release |
| Open Questions | Decisions deferred |
| Follow-up Issues | Create issues for discoveries |

**Important:** Create GitHub issues for any problems discovered during the release. Link them in the retrospective comment and consider them for the next release scope.

#### Codify Lessons Learned

After the retrospective, update this RELEASE.md with any process improvements:
- New phases or steps discovered
- Commands or patterns that should be documented
- Gotchas to avoid in future releases

Commit with message: `Update RELEASE.md with vX.Y lessons learned`

## Version Numbering

**Pre-release:** `v0.X` (current phase)
- Simple major.minor versioning (e.g., v0.8, v0.9, v0.10)
- No patch numbers or release candidates while pre-release
- Add `-rc1`, `-rc2` or `.1`, `.2` only if actually needed
- No backward compatibility guarantees
- Delete/recreate tags acceptable for pre-releases

**Stable:** `v1.0+` (future)
- Semantic versioning with patch numbers
- Backward compatibility expectations
- No tag recreation

## Release Issue Template

Each release should have a coordination issue in `.github` repo:

```markdown
## Summary
Planning for vX.Y release.

## Scope
### repo-name
- [ ] Feature/fix description (#issue)

## Validation
- [ ] Integration test results
- [ ] Manual verification

## Deferred to Future Release
- repo#N - Description

## Release Checklist
### CLAUDE.md Review
- [ ] All repos verified current

### CHANGELOGs
- [ ] site-config ... iac-driver

### Tags & Releases
- [ ] site-config ... iac-driver

### Post-Release (same day - do not defer)
- [ ] After Action Report
- [ ] Retrospective
- [ ] Update RELEASE.md with lessons learned
- [ ] Close release issue

---
**Status:** Planning | In Progress | Complete
```

## Path to Stable

Before graduating from pre-release to v1.0.0:

- [ ] User-facing documentation (beyond CLAUDE.md)
- [ ] CI/CD pipeline for automated integration tests
- [ ] All "known issues" resolved or documented
- [ ] Core workflows fully working
- [ ] Security audit (secrets, SSH, API tokens)
- [ ] Bootstrap UX polished

## References

- [v0.6 Release](https://github.com/homestak-dev/.github/issues/4) - First release using this methodology
- [v0.7 Release](https://github.com/homestak-dev/.github/issues/6) - Gateway fix, state storage move, E2E validation
- [v0.8 Release](https://github.com/homestak-dev/.github/issues/11) - CLI robustness, `latest` packer release tag
- [v0.9 Release](https://github.com/homestak-dev/.github/issues/14) - Scenario annotations, --timeout, unit tests, CLAUDE.md audit

## Lessons Learned

### v0.9
- **Thorough CLAUDE.md verification pays off** - Found 6 documentation errors during release. Consider making this a standard release phase rather than optional.
- **Generate scenario tables from code** - Manually maintaining phase counts leads to drift. Consider `--list-scenarios --json` for automation.
- **Integration test is not optional** - Skipping nested-pve-roundtrip before release is risky. Make it a hard blocker.
- **Fetch before release work** - Run `git fetch` on all repos before starting release to avoid rebase surprises.
- **Explicitly verify all image uploads** - debian-13-pve was omitted initially. Add image checklist to release process.
- **GitHub 2GB release asset limit** - Large images (>2GB) must be split: `split -b 1900M image.qcow2 image.qcow2.part`, users reassemble with `cat image.qcow2.part* > image.qcow2`. Document in release notes.
- **Review organization README** - `.github/profile/README.md` was updated retroactively to fix terminology. Added to pre-flight checklist.

### v0.8
- **Complete AAR/retro immediately** - Deferred post-release tasks result in lost context. Block on these before starting next release work.
