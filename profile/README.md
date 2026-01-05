# homestak-dev

**Homelab infrastructure, automated.**

![Ansible](https://img.shields.io/badge/Ansible-EE0000?logo=ansible&logoColor=white)
![OpenTofu](https://img.shields.io/badge/OpenTofu-813cf3?logo=opentofu&logoColor=white)
![Packer](https://img.shields.io/badge/Packer-02A8EF?logo=packer&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-Apache_2.0-blue)

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/master/install.sh | bash
```

## Architecture

```
                     ┌─────────────┐
                     │  bootstrap  │  curl|bash entry point
                     └──────┬──────┘
                            │
     ┌──────────────────────┼──────────────────────┐
     │                      │                      │
     ▼                      ▼                      ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ site-config │     │ iac-driver  │     │   ansible   │
│   4NF YAML  │────▶│ orchestrate │◀───▶│  configure  │
└─────────────┘     └──────┬──────┘     └─────────────┘
                           │
                    ConfigResolver
                           │
             ┌─────────────┴─────────────┐
             ▼                           ▼
      ┌──────────┐                ┌──────────┐
      │   tofu   │                │  packer  │
      │ provision│                │   build  │
      └──────────┘                └──────────┘
```

## Configuration Model

site-config uses a normalized 4-entity YAML structure:

```
site-config/
├── site.yaml              # Site-wide defaults
├── secrets.yaml           # Encrypted (SOPS + age)
├── hosts/                 # Physical machines (SSH access)
├── nodes/                 # PVE instances (API access)
├── vms/                   # VM templates + size presets
│   └── presets/           # xsmall, small, medium, large, xlarge
└── envs/                  # Deployment topologies (node-agnostic)
```

**Resolution flow:** presets → templates → env instances → ConfigResolver → tfvars.json → tofu

## What You Can Do

```bash
# Configure Proxmox host
homestak pve-setup

# Deploy VMs (single or multi-VM environments)
homestak scenario simple-vm-roundtrip --host pve
homestak scenario simple-vm-constructor --host pve --env dev

# Full E2E validation (~8.5 min)
homestak scenario nested-pve-roundtrip --host pve

# User management
homestak playbook user -e local_user=me

# Check installation
homestak status
```

## Workflow

```
Fresh Debian/Proxmox Host                   Running VMs
       │                                         ▲
       │  curl|bash                              │
       ▼                                         │
 ┌───────────┐    ┌───────────┐    ┌───────────┐
 │ bootstrap │───▶│  ansible  │───▶│   tofu    │
 │  install  │    │ configure │    │ provision │
 └───────────┘    └───────────┘    └───────────┘
                                         │
                        ┌────────────────┴────────────────┐
                        │                                 │
                   Single VM                         Multi-VM
                  (test env)                    (dev, k8s envs)
```

## Repositories

| Repo | Purpose |
|------|---------|
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point - installs `homestak` CLI and core repos |
| [site-config](https://github.com/homestak-dev/site-config) | 4NF YAML configuration with SOPS + age encryption |
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestration engine with ConfigResolver |
| [ansible](https://github.com/homestak-dev/ansible) | Configure PVE hosts, install Proxmox on Debian 13 |
| [tofu](https://github.com/homestak-dev/tofu) | Provision VMs with OpenTofu (dumb executor) |
| [packer](https://github.com/homestak-dev/packer) | Build custom Debian cloud images (~16s boot) |

## Key Features

- **ConfigResolver** - All config logic in Python, tofu receives flat tfvars
- **VM Templates** - Define once, deploy anywhere with size presets
- **Multi-VM Environments** - Deploy complex topologies in one command
- **Node-Agnostic Deploys** - Same env template works on any PVE host
- **E2E Validation** - Nested PVE testing validates full stack
- **Community Roles** - Evaluating lae.proxmox for PVE installation

## License

Apache 2.0
