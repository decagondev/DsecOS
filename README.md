# DsecOS – Secure Hypervisor Platform

**Secure. Isolated. Production-Ready.**

DsecOS is a hardened, rebranded hypervisor built on Proxmox VE, designed for secure application hosting with zero-trust isolation. It combines KVM, LXC, Docker, and enterprise-grade security into a single, easy-to-deploy platform.

---

## Features

| Feature | Description |
|-------|-----------|
| **Custom Hardened Kernel** | Linux 6.x with KVM, LXC, and SELinux in enforcing mode |
| **Zero-Trust Networking** | nftables default-deny + SDN zones |
| **Full-Stack Templates** | One-click deploy: Node.js + PostgreSQL, Java + MySQL, Python + Redis, Ruby + SQLite |
| **Docker + Portainer** | Native container runtime with web UI |
| **Dark Themed Web UI** | Modern React + Vite frontend (coming in v0.2) |
| **Bare-Metal Provisioning** | PXE + Kickstart for automated installs |
| **Automated Hardening** | CIS-compliant, LUKS, AppArmor, Fail2Ban, OSSEC |
| **Enterprise Licensing** | JWT-based license validation (v1.0+) |
| **AI-Powered Monitoring** | Anomaly detection via Prometheus + ML (v1.0+) |

---

## Quick Start

### 1. Download ISO
```bash
wget https://releases.dsecos.io/v0.1-mvp/dsecos-v0.1-mvp.iso
```

### 2. Boot & Install
- Burn to USB or load in VM
- Follow installer (auto-partitions + LUKS)
- Default credentials: `root` / `dsecos`

### 3. Access Web UI
```url
https://your-server-ip:9443
```
> Self-signed cert — accept in browser

### 4. Deploy Your First Stack
```bash
dsecos deploy node-postgres
```
App live at: `http://your-server-ip:3000`

---

## Documentation

- **[Installation Guide](docs/install/iso-install.md)**
- **[PXE Provisioning](docs/install/pxe-install.md)**
- **[Stack Templates](docs/usage/deploy-stack.md)**
- **[Security & Hardening](docs/security/hardening.md)**
- **[Architecture Diagrams](docs/diagrams.md)**

---

## Roadmap

| Version | Target | Key Features |
|-------|--------|------------|
| **v0.1 MVP** | Nov 13, 2025 | Rebrand, kernel, Docker, stacks, PXE |
| **v0.2 Beta** | Dec 6, 2025 | React UI, FastAPI, AI alerts |
| **v1.0 GA** | Mar 31, 2026 | HA clustering, Ceph, enterprise license |

---

## Development

```bash
git clone https://github.com/decagondev/DecaOS.git
cd DecaOS
```

---

## Security

- SELinux: Enforcing by default
- All traffic: Encrypted + authenticated
- Regular Lynis scans: Target >90% compliance
- Report issues: [security@decadev.co.uk](mailto:security@decadev.co.uk)

---

## License

- **Community**: MIT (pre-v1.0)
- **Enterprise**: Proprietary (v1.0+)

---

**DsecOS – Because your apps deserve better than shared hosting.**

[https://dsecos.io](https://dsecos.io) | [Contact](mailto:hello@dsecos.io)
