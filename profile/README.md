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
     │   tofu   │                │  packer  │  (optional)
     │ provision│                │   build  │
     └──────────┘                └──────────┘
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
```

## What You Can Do

```bash
homestak pve-setup                    # Configure Proxmox host
homestak scenario simple-vm-roundtrip # Provision → verify → destroy VM
homestak playbook user -e local_user=me
homestak status                       # Check installation
```

## Repositories

| Repo | Purpose |
|------|---------|
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point - installs `homestak` CLI and core repos |
| [site-config](https://github.com/homestak-dev/site-config) | Site-specific secrets and configuration (SOPS + age) |
| [ansible](https://github.com/homestak-dev/ansible) | Configure PVE hosts, install Proxmox on Debian |
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestrate multi-step workflows |
| [tofu](https://github.com/homestak-dev/tofu) | Provision VMs with OpenTofu |
| [packer](https://github.com/homestak-dev/packer) | Build custom Debian cloud images (optional) |

## License

Apache 2.0
