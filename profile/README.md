<picture>
  <source media="(prefers-color-scheme: dark)" srcset="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg">
  <source media="(prefers-color-scheme: light)" srcset="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg">
  <img alt="homestak" src="https://raw.githubusercontent.com/homestak-dev/.github/main/profile/homestak-logo.svg" width="280">
</picture>

### Homelab infrastructure, automated.

One command. Fresh Proxmox host to running VMs — repeatable, testable, version-controlled.

```bash
curl -fsSL https://raw.githubusercontent.com/homestak/bootstrap/master/install | bash
```

[![Proxmox](https://img.shields.io/badge/Proxmox-E57000?logo=proxmox&logoColor=white)](https://www.proxmox.com/)
[![Debian](https://img.shields.io/badge/Debian-A81D33?logo=debian&logoColor=white)](https://www.debian.org/)
[![Ansible](https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white)](https://www.ansible.com/)
[![OpenTofu](https://img.shields.io/badge/OpenTofu-813cf3?logo=opentofu&logoColor=white)](https://opentofu.org/)
[![Packer](https://img.shields.io/badge/Packer-02A8EF?logo=packer&logoColor=white)](https://www.packer.io/)
[![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-Apache_2.0-blue)](LICENSE)

---

### Declare your topology, let homestak deploy it

Define your infrastructure as a manifest — nodes, types, sizes, parent relationships — and let the orchestration engine handle the rest.

```yaml
# A PVE hypervisor with a VM running inside it
nodes:
  - name: root-pve
    type: pve
    preset: vm-large
    image: pve-9

  - name: app-server
    type: vm
    preset: vm-small
    image: debian-12
    parent: root-pve
    execution:
      mode: pull          # async — self-configures on first boot
```

```bash
homestak manifest apply -M my-topology -H my-host
```

The engine walks the node graph, provisions in dependency order, and configures each node. Two deployment models: **synchronous** (push config over SSH, wait for completion) or **asynchronous** (provision with a signed token, node self-configures on first boot). Supports N-level nesting: PVE inside PVE inside PVE, with VMs at any level.

---

### What you get

- **Manifest-driven orchestration** — graph-based topologies with N-level nested PVE support
- **Sync and async deployment** — push config over SSH or let nodes self-configure on first boot
- **Pre-built VM images** — custom Debian and PVE images that boot in ~16 seconds
- **Encrypted secrets** — SOPS + age, version-controlled alongside your config
- **Integration tested** — real Proxmox environments validate the full stack, not just unit tests

---

### How it works

```
 curl | bash                    config (secrets + topology)
     │                                      │
     ▼                                      ▼
 bootstrap ──► iac-driver ──► ansible ──► configured host
                   │
                   ├──► tofu ──► provisioned VMs
                   └──► packer ──► custom images
```

Bootstrap installs the CLI and clones the repos. **iac-driver** orchestrates everything — it reads your manifests, resolves config, and coordinates ansible, tofu, and packer to get you from bare metal to running VMs.

---

### Repositories

**[homestak](https://github.com/homestak)** — Core

| Repo | Purpose |
|------|---------|
| **[bootstrap](https://github.com/homestak/bootstrap)** | Entry point — installs the CLI and clones core repos |
| **[config](https://github.com/homestak/config)** | Secrets, manifests, specs, presets, and site-specific config |

**[homestak-iac](https://github.com/homestak-iac)** — Infrastructure automation

| Repo | Purpose |
|------|---------|
| **[iac-driver](https://github.com/homestak-iac/iac-driver)** | Orchestration engine — manifest-driven node lifecycle |
| **[ansible](https://github.com/homestak-iac/ansible)** | Playbooks for PVE host and VM configuration |
| **[tofu](https://github.com/homestak-iac/tofu)** | OpenTofu modules for VM provisioning |
| **[packer](https://github.com/homestak-iac/packer)** | Custom Debian cloud images (optional) |

**[homestak-dev](https://github.com/homestak-dev)** — Developer experience

| Repo | Purpose |
|------|---------|
| **[meta](https://github.com/homestak-dev/meta)** | Release scripts, docs, development process |

---

### Design philosophy

**Repeatability over flexibility.** Sensible defaults that just work. Tweak everything if you want — but you shouldn't have to.

**Debian-rooted, Proxmox-current.** Built on Debian with Proxmox as the virtualization layer. The architecture leaves room for bare QEMU/KVM if you ever need it.

**Testable infrastructure.** This isn't a collection of scripts. Integration tests spin up real Proxmox environments and validate the full stack — from bootstrap through provisioning to configuration.

---

Apache 2.0
