# DsecOS Installation Guide (ISO)

> **Complete step-by-step guide** for installing DsecOS using the official ISO image on bare metal or virtual machines.

---

## Prerequisites

| Requirement | Specification |
|-----------|----------------|
| **CPU** | x86_64 with virtualization (Intel VT-x / AMD-V) |
| **RAM** | 16 GB minimum, 64 GB+ recommended |
| **Storage** | 500 GB NVMe/SSD (RAID optional) |
| **Network** | DHCP + internet (for updates) |
| **Boot Media** | USB ≥8 GB or ISO in VM |

---

## Step 1: Download the ISO

```bash
# Download latest release
wget https://releases.decadev.co.uk/v0.1-mvp/dsecos-v0.1-mvp.iso

# Optional: Verify integrity
wget https://releases.decadev.co.uk/v0.1-mvp/SHA256SUMS
sha256sum -c SHA256SUMS
```

---

## Step 2: Create Bootable Media

### USB (Linux/macOS/Windows)

**Linux/macOS**:
```bash
# List drives
lsblk

# Write to USB (replace /dev/sdX)
sudo dd if=dsecos-v0.1-mvp.iso of=/dev/sdX bs=4M status=progress && sync
```

**Windows**: Use [Rufus](https://rufus.ie) → Select ISO → Start

### Virtual Machine
- **VMware**: Create VM → Attach ISO to CD/DVD
- **VirtualBox**: Settings → Storage → Add ISO
- **Proxmox/QEMU**: Upload to `local` storage → assign to VM

---

## Step 3: Boot from ISO

1. Insert USB or start VM
2. Enter BIOS (`F2`, `Del`, `F10`, `Esc`)
3. Set **boot order**: USB/CD first
4. **Enable virtualization** in CPU settings
5. Save & exit

→ Boots into **DsecOS Live Environment**

---

## Step 4: Launch Installer

```text
╔══════════════════════════════════════════════════════╗
║               DsecOS Installer v0.1-mvp              ║
║                                                      ║
║  [1] Install DsecOS                                  ║
║  [2] Rescue Shell                                    ║
║  [3] Reboot                                          ║
╚══════════════════════════════════════════════════════╝
```

Press **`1`** → **Install DsecOS**

---

## Step 5: Installation Wizard

### 1. Disk Configuration
| Option | Recommended |
|-------|-----------|
| **Target Disk** | `/dev/sda` (full wipe) |
| **Partitioning** | Auto (GPT) |
| **Encryption** | **Enable LUKS** (full disk) |
| **Passphrase** | 12+ characters, store securely |

### 2. Network Setup
- **DHCP** (default)
- Or **Static IP**:
  ```text
  IP: 192.168.1.100/24
  Gateway: 192.168.1.1
  DNS: 8.8.8.8, 1.1.1.1
  ```

### 3. Hostname & Credentials
```text
Hostname: dsecos-node1
Root Password: [Set strong password]
```

### 4. Confirm & Install
- Review summary
- Press **Install** → ~5–10 minutes

---

## Step 6: First Boot

```text
DsecOS v0.1-mvp
Linux 6.5.0-dsecos-selinux on x86_64

dsecos-node1 login: root
Password: ********

[ OK ] SELinux: enforcing
[ OK ] nftables: default DROP
[ OK ] Docker + Portainer: running
[ OK ] Web UI: https://192.168.1.100:9443
```

---

## Step 7: Access the Web Interface

1. Open browser:
   ```
   https://your-server-ip:9443
   ```
2. **Accept self-signed certificate**
3. Login:
   - **Username**: `admin@pve`
   - **Password**: (set during install)

→ Dark-themed dashboard appears

---

## Step 8: Deploy Your First Application

```bash
# SSH in
ssh root@your-server-ip

# Deploy full-stack app
dsecos deploy node-postgres
```

**App URL**: `http://your-server-ip:3000`

---

## Post-Installation Checklist

| Task | Command |
|------|--------|
| Update system | `apt update && apt upgrade -y` |
| Verify SELinux | `sestatus` → `enforcing` |
| Check firewall | `nft list ruleset` |
| Test Portainer | `https://your-ip:9443/portainer` |
| Backup config | `tar czf /root/dsecos-backup.tar.gz /etc/dsecos/` |

---

## Troubleshooting

| Problem | Fix |
|-------|----|
| No network | `ip link`, check cable, reboot |
| LUKS unlock fails | Boot to rescue → `cryptsetup luksOpen` |
| Web UI down | `systemctl restart pveproxy` |
| Docker not starting | `journalctl -u docker` |

---

**Next**: [PXE Provisioning →](pxe-install.md)  
**Support**: `support@decadev.co.uk`
