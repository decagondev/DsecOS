# Getting Proxmox VE Source Code

> **Official, up-to-date source** for Proxmox VE is hosted on **git.proxmox.com** (public read-only).  
> All repositories are licensed under **GNU AGPLv3** — perfect for forking and rebranding as DsecOS.

---

## 1. Main Repository Browser

```
https://git.proxmox.com/
```

→ Click **"projects /"** → Full list of 100+ repos.

### Core Repositories You Need

| Repo | Purpose | Clone Command |
|------|--------|--------------|
| `pve-manager` | Web UI + API | `git clone https://git.proxmox.com/git/pve-manager.git` |
| `pve-common` | Shared Perl libraries | `git clone https://git.proxmox.com/git/pve-common.git` |
| `pve-container` | LXC tools | `git clone https://git.proxmox.com/git/pve-container.git` |
| `pve-qemu` | KVM/QEMU wrapper | `git clone https://git.proxmox.com/git/pve-qemu.git` |
| `pve-storage` | Storage plugins | `git clone https://git.proxmox.com/git/pve-storage.git` |
| `pve-cluster` | Clustering | `git clone https://git.proxmox.com/git/pve-cluster.git` |
| `pve-firewall` | nftables firewall | `git clone https://git.proxmox.com/git/pve-firewall.git` |
| `pve-kernel` | Custom kernel (latest) | `git clone https://git.proxmox.com/git/pve-kernel.git` |

---

## 2. One-Command Clone All Core Repos

```bash
mkdir dsecos-source && cd dsecos-source

git clone https://git.proxmox.com/git/pve-manager.git
git clone https://git.proxmox.com/git/pve-common.git
git clone https://git.proxmox.com/git/pve-container.git
git clone https://git.proxmox.com/git/pve-qemu.git
git clone https://git.proxmox.com/git/pve-storage.git
git clone https://git.proxmox.com/git/pve-cluster.git
git clone https://git.proxmox.com/git/pve-firewall.git
git clone https://git.proxmox.com/git/pve-kernel.git
```

> ~2 GB total — latest Proxmox VE 8.2+ code

---

## 3. GitHub Mirrors (Read-Only, Often Behind)

```bash
# Example: pve-manager mirror
git clone https://github.com/proxmox/pve-manager.git
```

> **Warning**: GitHub mirrors lag behind official git.proxmox.com by hours/days.

---

## 4. Quick Start – Build a Single Package

```bash
# Example: rebuild pve-manager
cd pve-manager

# Install build dependencies
sudo apt build-dep .

# Or use mk-build-deps (lighter)
sudo apt install devscripts
mk-build-deps -ir

# Build .deb packages
make deb

# Install on current system
sudo dpkg -i *.deb
```

> Repeat for other repos

---

## 5. Full DsecOS Fork Setup (Recommended)

```bash
# Create private repo
mkdir dsecos-private && cd dsecos-private

# Add as Git submodules (keeps upstream clean)
git init
git submodule add https://git.proxmox.com/git/pve-manager.git src/pve-manager
git submodule add https://git.proxmox.com/git/pve-common.git src/pve-common
git submodule add https://git.proxmox.com/git/pve-container.git src/pve-container
git submodule add https://git.proxmox.com/git/pve-qemu.git src/pve-qemu
git submodule add https://git.proxmox.com/git/pve-storage.git src/pve-storage
git submodule add https://git.proxmox.com/git/pve-cluster.git src/pve-cluster
git submodule add https://git.proxmox.com/git/pve-firewall.git src/pve-firewall
git submodule add https://git.proxmox.com/git/pve-kernel.git kernel/pve-kernel

# Update all submodules
git submodule update --init --recursive
```

> Now you can patch, rebrand, and commit on top.

---

## 6. Latest Stable Tags (as of Nov 06, 2025)

```bash
# Checkout latest release in each repo
cd src/pve-manager
git checkout 8.2.7    # or run: git describe --tags $(git rev-list --tags --max-count=1)

# Do this for every submodule
```

---

## 7. Build Full Custom ISO (Next Step)

```bash
# After forking → use Packer
mkdir packer && cd packer
nano dsecos.pkr.hcl
```

→ We’ll cover this in the **ISO Build Guide**.

---

## Useful Links

- **All repos**: https://git.proxmox.com/
- **Developer docs**: https://pve.proxmox.com/wiki/Developer_Documentation
- **Build tips**: https://pve.proxmox.com/wiki/Developer_Documentation#Building_packages

---

**You now have the exact source code used in Proxmox VE 8.2+**  
Ready for rebranding, kernel patching, and DsecOS MVP.

**Next**: Run the commands above → then we’ll start **Epic 1** (fork + submodule setup).  

`support@decadev.co.uk` for any issues.
