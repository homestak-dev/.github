# homestak-dev

This file provides guidance to Claude Code when working with this repository.

## Vision

**homestak** makes setting up and running a homelab repeatable and manageable. The infrastructure-as-code in this repository is a means to an end: creating a platform for "best in class" self-hosted applications like Home Assistant, Jellyfin, Vaultwarden, and other highly desirable home apps.

### Open Source + Commercial Model

| Organization | Purpose |
|--------------|---------|
| **homestak-dev** | Open-source IaC components (this repo) |
| **homestak-com** | Commercial offering: verified releases, remote monitoring/management, cloud backup, high availability, community and live support |

The open-source foundation enables the commercial layer, not the other way around.

## Technical Foundation

### Debian-Rooted, Proxmox-Current

The platform is rooted in **Debian**, with **Proxmox VE** as the current virtualization solution. Proxmox is at the heart of current workflows, but the architecture should leave the door open for:

- QEMU/KVM without Proxmox (bare Debian hosts)
- Alternative virtualization platforms built on Debian

Design decisions should favor Debian primitives over Proxmox-specific features when practical.

### Full Homelab Stack (Roadmap)

Current focus is VM provisioning and PVE host configuration. Future scope includes:

- Kubernetes (k3s, kubeadm)
- Storage (ZFS, Ceph)
- Advanced networking (VLANs, SDN, firewalls)
- Application deployment (the actual goal)

## Repository Structure

```
homestak-dev/
├── bootstrap/      # Entry point - curl|bash installer, homestak CLI
├── site-config/    # Site-specific secrets and configuration
├── iac-driver/     # Orchestration engine - scenario-based workflows
├── ansible/        # Playbooks for host configuration
├── tofu/           # OpenTofu modules for VM provisioning
└── packer/         # Custom Debian cloud images (optional)
```

Each component has its own `CLAUDE.md` with detailed context:

| Component | Focus |
|-----------|-------|
| `bootstrap/CLAUDE.md` | Installation, homestak CLI, dependency management |
| `site-config/CLAUDE.md` | Secrets encryption, SOPS/age, host credentials |
| `iac-driver/CLAUDE.md` | Scenarios, actions, integration testing |
| `ansible/CLAUDE.md` | Playbooks, roles, inventory, execution models |
| `tofu/CLAUDE.md` | Modules, environments, configuration inheritance |
| `packer/CLAUDE.md` | Templates, build workflow, image optimization |

## Value Propositions

1. **Integrated workflow** - Unified tooling across packer→tofu→ansible with orchestration
2. **Proxmox-optimized** - Purpose-built for Proxmox VE homelabs (with Debian escape hatch)
3. **Opinionated defaults** - Sensible choices for homelab (SDN, cloud-init, security profiles)
4. **Testable infrastructure** - Nested PVE integration testing validates the full stack

## Design Principles

- **Do it right** - Prefer proper solutions over quick workarounds. If a task is worth doing, invest in the reusable, maintainable approach rather than one-off hacks. Today's shortcut becomes tomorrow's technical debt.
- **Repeatability over flexibility** - Prefer conventions that "just work" over infinite configurability
- **Local-first execution** - Run on the host being configured to avoid SSH connection issues
- **Idempotent operations** - Safe to run multiple times
- **Secrets in code, encrypted** - SOPS + age in site-config repo, git hooks for auto-encrypt/decrypt
- **Component independence** - Each repo installs its own dependencies via `make install-deps`

## Conventions

- **VM IDs**: 5-digit (10000+ dev, 20000+ k8s, 99900+ integration test)
- **MAC prefix**: BC:24:11:*
- **Networks**: dev 10.10.10.0/24, k8s 10.10.20.0/24, management 10.0.12.0/24
- **Hostnames**: `{cluster}{instance}` (dev1, kubeadm1, router)
- **Environments**: dev (permissive) vs prod (hardened)

## Host Capabilities

Not all hosts have the same capabilities. Key distinctions:

| Host | QEMU/KVM | PVE API | Notes |
|------|----------|---------|-------|
| father | Yes | Yes | Primary build host for packer images |
| mother | Yes | Yes | Secondary PVE host |
| dev machines | Maybe | No | May lack nested virtualization |

**Packer builds require QEMU/KVM.** Use `packer-build-fetch` scenario to build on capable hosts:
```bash
./run.sh --scenario packer-build-fetch --remote father
```

## Bootstrap Pattern

The `bootstrap` repo provides capability installation via `homestak install <module>`:

```bash
# Initial setup (on any Debian host)
curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/main/install.sh | bash

# Add capabilities as needed
homestak install packer    # QEMU, packer, templates
homestak install tofu      # OpenTofu
homestak install ansible   # Ansible + collections
```

This pattern enables any Debian host to become a build/deploy host without manual setup.

## Release Process

See [RELEASE.md](RELEASE.md) for the release methodology, including:
- Repository dependency order
- 8-phase release workflow
- After action reports and retrospectives

## License

Apache 2.0
