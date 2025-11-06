# DsecOS PXE Provisioning Guide

> **Automated bare-metal deployment** using PXE boot and Kickstart-style configuration.  
> Ideal for data centers, labs, or large-scale rollouts.

---

## Overview

DsecOS supports fully unattended installation over the network via:
- **DHCP** → IP assignment
- **TFTP** → Bootloader (pxelinux)
- **HTTP/NFS** → Kickstart config + ISO contents

No manual intervention required after initial server setup.

---

## Prerequisites

| Component | Requirement |
|---------|------------|
| **Network** | Dedicated VLAN/subnet for provisioning |
| **DHCP Server** | ISC DHCP or similar |
| **TFTP Server** | `tftpd-hpa` |
| **Web Server** | `nginx` or `apache2` (serves ISO + ks.cfg) |
| **Hardware** | PXE-capable NICs on target nodes |

---

## Step 1: Prepare the Provisioning Server

```bash
# Install required packages
sudo apt update
sudo apt install -y tftpd-hpa nginx syslinux pxelinux
```

### Directory Structure
```bash
/srv/tftp/          # TFTP root
├── pxelinux.0
├── ldlinux.c32
└── pxelinux.cfg/
    └── default

/srv/www/dsecos/    # Web root
├── dsecos-v0.1-mvp.iso
└── ks.cfg
```

---

## Step 2: Extract Boot Files from ISO

```bash
# Mount ISO
sudo mkdir /mnt/iso
sudo mount -o loop dsecos-v0.1-mvp.iso /mnt/iso

# Copy kernel + initrd
sudo cp /mnt/iso/live/vmlinuz /srv/tftp/
sudo cp /mnt/iso/live/initrd.img /srv/tftp/

# Copy syslinux boot files
sudo cp /usr/lib/PXELINUX/pxelinux.0 /srv/tftp/
sudo cp /usr/lib/syslinux/modules/bios/ldlinux.c32 /srv/tftp/

sudo umount /mnt/iso
```

---

## Step 3: Configure PXE Menu

```bash
sudo nano /srv/tftp/pxelinux.cfg/default
```

```text
DEFAULT dsecos-install
LABEL dsecos-install
    KERNEL vmlinuz
    APPEND initrd=initrd.img boot=live fetch=http://192.168.1.10/dsecos/ks.cfg
    IPAPPEND 2

TIMEOUT 50
PROMPT 1
```

> Replace `192.168.1.10` with your provisioning server IP

---

## Step 4: Create Kickstart File (`ks.cfg`)

```bash
sudo nano /srv/www/dsecos/ks.cfg
```

```ks
# DsecOS Automated Install
url --url=http://192.168.1.10/dsecos/

# Language & Keyboard
lang en_GB.UTF-8
keyboard uk

# Root password (pre-hashed)
rootpw --iscrypted $6$abc123...

# Network
network --bootproto=dhcp --hostname=dsecos-node01

# Disk partitioning + LUKS
clearpart --all --initlabel
part /boot --fstype=ext4 --size=512
part pv.01 --size=1000 --grow
volgroup vg0 pv.01
logvol / --vgname=vg0 --name=root --size=50000 --fstype=ext4 --encrypted --passphrase=YourSecurePassphrase

# Packages
%packages --ignoremissing
@core
docker
portainer
%end

# Post-install
%post
# Enable services
systemctl enable docker portainer

# Harden SSH
sed -i 's/#PermitRootLogin.*/PermitRootLogin no/' /etc/ssh/sshd_config
systemctl restart sshd

# Finalize
%end

reboot
```

---

## Step 5: Configure Web Server (nginx)

```bash
sudo nano /etc/nginx/sites-available/dsecos
```

```nginx
server {
    listen 80;
    server_name 192.168.1.10;

    root /srv/www/dsecos;
    index ks.cfg;

    location / {
        autoindex on;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/dsecos /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

---

## Step 6: Configure DHCP (Example with ISC DHCP)

```bash
sudo nano /etc/dhcp/dhcpd.conf
```

```conf
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8;
    filename "pxelinux.0";
    next-server 192.168.1.10;
}
```

```bash
sudo systemctl restart isc-dhcp-server
```

---

## Step 7: Boot Target Node

1. Power on server
2. Enter BIOS → Enable **PXE/Network Boot**
3. Save & reboot
4. Node receives IP → downloads `pxelinux.0` → loads kernel
5. Kickstart runs → **fully automated install**

> Installation complete in **~8–12 minutes**

---

## Verification

After reboot:
```bash
ssh root@192.168.1.x
sestatus | grep enforcing
nft list ruleset | grep DROP
docker ps | grep portainer
```

---

## Advanced: Custom Per-Node Config

Use MAC-based Kickstart files:
```bash
/srv/tftp/pxelinux.cfg/01-aa-bb-cc-dd-ee-ff  # lowercase MAC with dashes
```

---

## Troubleshooting

| Issue | Check |
|------|------|
| No PXE menu | `tcpdump -i eth0 port 69` (TFTP) |
| Kernel panic | Verify `vmlinuz` + `initrd.img` match ISO |
| ks.cfg not found | `curl http://192.168.1.10/ks.cfg` |
| LUKS fail | Check passphrase in `ks.cfg` |

---

**Next**: [Stack Templates →](deploy-stack.md)  
**Support**: `support@dsecos.io`
