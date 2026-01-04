# homestak-dev

Infrastructure-as-Code for Proxmox VE environments.

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/homestak-dev/bootstrap/master/install.sh | bash
```

This installs the `homestak` CLI for managing Proxmox infrastructure.

## Repositories

| Repo | Purpose |
|------|---------|
| [bootstrap](https://github.com/homestak-dev/bootstrap) | Entry point - curl\|bash setup and `homestak` CLI |
| [ansible](https://github.com/homestak-dev/ansible) | Proxmox host configuration and PVE installation |
| [iac-driver](https://github.com/homestak-dev/iac-driver) | Orchestration engine and E2E testing |
| [tofu](https://github.com/homestak-dev/tofu) | VM provisioning with OpenTofu |
| [packer](https://github.com/homestak-dev/packer) | Custom Debian cloud images (optional) |

## Architecture

```
bootstrap → clones ansible, iac-driver, tofu
         → installs 'homestak' CLI
         → runs 'make install-deps' per repo

homestak pve-setup      # Configure Proxmox host
homestak scenario ...   # Run orchestration workflows
homestak status         # Show installation status
```

## License

All repositories are licensed under Apache 2.0.
