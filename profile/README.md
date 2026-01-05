# homestak-dev

**Homelab infrastructure, automated.**

![Release](https://img.shields.io/github/v/release/homestak-dev/bootstrap?include_prereleases&label=release)
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
                    │  bootstrap  │  ← curl|bash entry point
                    └──────┬──────┘
                           │ clones
    ┌──────────────────────┼──────────────────────┐
    │                      │                      │
    ▼                      ▼                      ▼
┌─────────────┐    ┌─────────────┐        ┌─────────────┐
│ site-config │    │ iac-driver  │        │   ansible   │
│ ┌─────────┐ │    │ orchestrate │        │  configure  │
│ │ hosts/  │ │    └──────┬──────┘        └─────────────┘
│ │ nodes/  │ │           │
│ │ envs/   │◄├───────────┼───────────────────────┐
│ │secrets  │ │           │                       │
│ └─────────┘ │           │                       │
└─────────────┘           │                       │
       ▲          ┌───────┴───────┐               │
       │          ▼               ▼               │
       │   ┌──────────┐    ┌──────────┐           │
       └───┤   tofu   │    │  packer  │ (optional)│
           │ provision│    │   build  │           │
           └──────────┘    └──────────┘           │
                 │                                │
                 └────────────────────────────────┘
                        config-loader reads
                        YAML configuration
```

## Configuration Model

Site-specific settings live in `site-config/` using normalized YAML (4NF):

```
site-config/
├── site.yaml        # Defaults (timezone, domain, datastore)
├── secrets.yaml     # Encrypted with SOPS + age
├── hosts/           # Physical machines (Ansible)
│   └── {host}.yaml
├── nodes/           # PVE instances (Tofu API access)
│   └── {node}.yaml
└── envs/            # Deployment topologies
    └── {env}.yaml
```

## Workflow

```
Fresh Debian/Proxmox Host                   Running VMs
       │                                         ▲
       │  curl|bash                              │
       ▼                                         │
 ┌───────────┐    ┌───────────┐    ┌───────────┐
 │ bootstrap │───►│  ansible  │───►│   tofu    │
 │  install  │    │ configure │    │ provision │
 └───────────┘    └───────────┘    └───────────┘
                         │               │
                         └───────┬───────┘
                                 ▼
                          site-config
                        (YAML + secrets)
```

## What You Can Do

```bash
homestak pve-setup                    # Configure Proxmox host
homestak scenario simple-vm-roundtrip # Provision → verify → destroy VM
homestak secrets decrypt              # Decrypt site-config secrets
homestak playbook user -e local_user=me
homestak status                       # Check installation
```

## Repositories

| Repo | Purpose |
|------|---------|
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point - installs `homestak` CLI and core repos |
| [site-config](https://github.com/homestak-dev/site-config) | Central configuration: nodes, envs, secrets (SOPS + age) |
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestrate multi-step workflows and scenarios |
| [ansible](https://github.com/homestak-dev/ansible) | Configure PVE hosts, install Proxmox on Debian |
| [tofu](https://github.com/homestak-dev/tofu) | Provision VMs with OpenTofu + config-loader |
| [packer](https://github.com/homestak-dev/packer) | Build custom Debian cloud images (optional) |

## License

Apache 2.0
