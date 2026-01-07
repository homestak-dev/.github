<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg">
  <img alt="homestak" src="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg" width="280">
</picture>

### Homelab infrastructure, automated.

You've got the hardware. You've got Proxmox running. Now you want to stop doing everything by hand and start treating your homelab like real infrastructure — repeatable, testable, version-controlled.

That's what homestak is for.

[![Proxmox](https://img.shields.io/badge/Proxmox-E57000?logo=proxmox&logoColor=white)](https://www.proxmox.com/)
[![Debian](https://img.shields.io/badge/Debian-A81D33?logo=debian&logoColor=white)](https://www.debian.org/)
[![Ansible](https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white)](https://www.ansible.com/)
[![OpenTofu](https://img.shields.io/badge/OpenTofu-813cf3?logo=opentofu&logoColor=white)](https://opentofu.org/)
[![Packer](https://img.shields.io/badge/Packer-02A8EF?logo=packer&logoColor=white)](https://www.packer.io/)
[![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue)](LICENSE)

---

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/master/install.sh | bash
```

This installs the `homestak` CLI and clones the core repos. From there, you can configure your Proxmox host, provision VMs, and run end-to-end scenarios.

---

## How It Works

```
                    ┌─────────────┐
                    │  bootstrap  │  ← curl|bash entry point
                    └──────┬──────┘
                           │
    ┌──────────────────────┼──────────────────────┐
    │                      │                      │
    ▼                      ▼                      ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│ site-config │    │ iac-driver  │    │   ansible   │
│   secrets   │◄──►│ orchestrate │◄──►│  configure  │
└─────────────┘    └──────┬──────┘    └─────────────┘
                          │
            ┌─────────────┼─────────────┐
            ▼                           ▼
     ┌──────────┐                ┌──────────┐
     │   tofu   │                │  packer  │
     │ provision│                │   build  │
     └──────────┘                └──────────┘
```

**bootstrap** gets you started. **iac-driver** orchestrates multi-step workflows. **ansible** configures hosts. **tofu** provisions VMs. **packer** builds custom images. **site-config** keeps your secrets encrypted and version-controlled.

---

## Typical Workflow

```
Fresh Debian/Proxmox Host                   Running VMs
       │                                         ▲
       │  curl|bash                              │
       ▼                                         │
 ┌───────────┐    ┌───────────┐    ┌───────────┐
 │ bootstrap │───►│  ansible  │───►│   tofu    │
 │  install  │    │ configure │    │ provision │
 └───────────┘    └───────────┘    └───────────┘
```

Start with a fresh Debian or Proxmox host. Bootstrap installs dependencies and the CLI. Ansible configures the host. Tofu provisions your VMs. Rinse and repeat.

---

## What Can You Do?

```bash
homestak pve-setup                    # Configure a Proxmox host
homestak scenario simple-vm-roundtrip # Provision → verify → destroy a VM
homestak playbook user -e local_user=me
homestak status                       # Check your installation
```

The `homestak` CLI wraps the underlying tools so you don't have to remember the incantations.

---

## Repositories

| Repo | What it does |
| --- | --- |
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point — installs the CLI and clones core repos |
| [site-config](https://github.com/homestak-dev/site-config) | Your secrets and site-specific config (SOPS + age encrypted) |
| [ansible](https://github.com/homestak-dev/ansible) | Playbooks for PVE host configuration |
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestrates multi-step workflows and integration tests |
| [tofu](https://github.com/homestak-dev/tofu) | OpenTofu modules for VM provisioning |
| [packer](https://github.com/homestak-dev/packer) | Build custom Debian cloud images (optional) |

Each repo has its own README and `CLAUDE.md` with deeper context.

---

## Design Philosophy

**Repeatability over flexibility.** We prefer conventions that "just work" over infinite configurability. If you want to tweak everything, you can — but the defaults should get you running.

**Debian-rooted, Proxmox-current.** The platform is built on Debian, with Proxmox as the virtualization layer. The architecture leaves room for bare QEMU/KVM on Debian if you ever need it.

**Testable infrastructure.** The integration scenarios in iac-driver actually spin up nested Proxmox environments to validate the full stack. This isn't a collection of scripts — it's tested automation.

---

## License

Apache 2.0
