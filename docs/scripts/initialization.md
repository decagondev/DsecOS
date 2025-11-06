# DsecOS Initialization Scripts

> **Essential scripts** to bootstrap, harden, and manage your DsecOS node.  
> All scripts are located in `/scripts/` — executable, idempotent, and safe to re-run.

---

## 1. `bootstrap.sh` – First Boot Setup

```bash
#!/bin/bash
# /scripts/bootstrap.sh
# Run once after install

set -euo pipefail

log() { echo "[+] $1"; }

log "Updating system..."
apt update && apt upgrade -y

log "Installing core tools..."
apt install -y curl wget jq nftables docker.io

log "Enabling services..."
systemctl enable --now docker
usermod -aG docker root

log "Bootstrap complete."
```

```bash
chmod +x /scripts/bootstrap.sh
/scripts/bootstrap.sh
```

---

## 2. `firewall-setup.sh` – Zero-Trust nftables

```bash
#!/bin/bash
# /scripts/firewall-setup.sh

set -euo pipefail

NFT="/usr/sbin/nft"

log() { echo "[FW] $1"; }

log "Flushing existing rules..."
$NFT flush ruleset

log "Creating filter table..."
$NFT add table inet filter

log "Setting default policies..."
$NFT add chain inet filter input   { type filter hook input priority 0 \; policy drop \; }
$NFT add chain inet filter forward { type filter hook forward priority 0 \; policy drop \; }
$NFT add chain inet filter output  { type filter hook output priority 0 \; policy accept \; }

log "Allowing loopback..."
$NFT add rule inet filter input iif lo accept
$NFT add rule inet filter output oif lo accept

log "Allowing established/related..."
$NFT add rule inet filter input ct state established,related accept
$NFT add rule inet filter output ct state established,related,new accept

log "Allowing SSH (port 22)..."
$NFT add rule inet filter input tcp dport 22 accept

log "Allowing Web UI (port 9443)..."
$NFT add rule inet filter input tcp dport 9443 accept

log "Firewall active."
```

```bash
chmod +x /scripts/firewall-setup.sh
sudo /scripts/firewall-setup.sh
```

---

## 3. `deploy-stack.sh` – One-Click Stack Deployer

```bash
#!/bin/bash
# /scripts/deploy-stack.sh
# Usage: deploy-stack.sh <stack-name>

set -euo pipefail

STACK="$1"
TEMPLATE="/templates/stacks/${STACK}.yml"
CONTAINER_NAME="dsecos-${STACK}"

if [[ -z "$STACK" ]]; then
    echo "Usage: $0 <stack-name>"
    exit 1
fi

if [[ ! -f "$TEMPLATE" ]]; then
    echo "Template $STACK not found!"
    exit 1
fi

log() { echo "[DEPLOY] $1"; }

log "Creating LXC container: $CONTAINER_NAME"
pct create 999 local:vztmpl/debian-12-standard_12.0-1_amd64.tar.zst \
    --hostname "$CONTAINER_NAME" \
    --memory 2048 \
    --cores 2 \
    --net0 name=eth0,bridge=vmbr0,ip=dhcp \
    --rootfs local-lvm:16 \
    --unprivileged 1

log "Starting container..."
pct start 999

log "Installing Docker in container..."
pct exec 999 -- bash -c "
    apt update && apt install -y docker.io docker-compose
    systemctl enable docker
"

log "Deploying stack via docker-compose..."
pct push 999 "$TEMPLATE" /root/docker-compose.yml
pct exec 999 -- docker-compose -f /root/docker-compose.yml up -d

log "$STACK deployed! Access via SDN-exposed ports."
```

```bash
chmod +x /scripts/deploy-stack.sh
sudo /scripts/deploy-stack.sh node-postgres
```

---

## 4. `harden-ssh.sh` – Secure SSH Access

```bash
#!/bin/bash
# /scripts/harden-ssh.sh

set -euo pipefail

SSHD_CONFIG="/etc/ssh/sshd_config"

log() { echo "[SSH] $1"; }

log "Hardening SSH..."

# Backup
cp "$SSHD_CONFIG" "${SSHD_CONFIG}.bak"

# Apply secure settings
sed -i \
    -e 's/#PermitRootLogin.*/PermitRootLogin no/' \
    -e 's/#PasswordAuthentication.*/PasswordAuthentication no/' \
    -e 's/#PubkeyAuthentication.*/PubkeyAuthentication yes/' \
    -e 's/#X11Forwarding.*/X11Forwarding no/' \
    "$SSHD_CONFIG"

log "Restarting SSH..."
systemctl restart ssh

log "SSH hardened. Use key-based auth only."
```

```bash
chmod +x /scripts/harden-ssh.sh
sudo /scripts/harden-ssh.sh
```

---

## 5. `check-security.sh` – Daily Health Check

```bash
#!/bin/bash
# /scripts/check-security.sh
# Run via cron: 0 2 * * * /scripts/check-security.sh

set -euo pipefail

log() { echo "[CHECK] $1"; }

log "Security status at $(date)"

log "SELinux: $(sestatus | grep 'Current mode' | awk '{print $3}')"
log "Firewall: $(( $($NFT -L | grep -c accept) )) allow rules"
log "Docker: $(docker ps --filter status=running | wc -l) containers"
log "Fail2Ban: $(fail2ban-client status sshd 2>/dev/null | grep "Banned IP" || echo "0")"

# Lynis quick scan
lynis audit system --quick --quiet

log "Check complete."
```

```bash
chmod +x /scripts/check-security.sh

# Add to crontab
(crontab -l; echo "0 2 * * * /scripts/check-security.sh >> /var/log/dsecos-check.log 2>&1") | crontab -
```

---

## 6. `dsecos` – Unified CLI Wrapper

```bash
#!/bin/bash
# /usr/local/bin/dsecos
# Symlink: ln -s /scripts/dsecos.sh /usr/local/bin/dsecos

set -euo pipefail

case "$1" in
    deploy)
        /scripts/deploy-stack.sh "$2"
        ;;
    harden)
        /scripts/harden-ssh.sh
        /scripts/firewall-setup.sh
        ;;
    check)
        /scripts/check-security.sh
        ;;
    list)
        ls /templates/stacks/*.yml | xargs -n1 basename | sed 's/\.yml$//'
        ;;
    *)
        echo "Usage: dsecos <deploy|harden|check|list> [stack]"
        exit 1
        ;;
esac
```

```bash
sudo cp /scripts/dsecos.sh /usr/local/bin/dsecos
sudo chmod +x /usr/local/bin/dsecos
```

---

## Directory Structure

```bash
/scripts/
├── bootstrap.sh
├── firewall-setup.sh
├── deploy-stack.sh
├── harden-ssh.sh
├── check-security.sh
└── dsecos.sh
```

---

## Run All at Once (Post-Install)

```bash
# After fresh install
sudo /scripts/bootstrap.sh
sudo /scripts/firewall-setup.sh
sudo /scripts/harden-ssh.sh
sudo dsecos deploy node-postgres
```

---

**Next**: Add to Packer build → automated ISO creation.  
**Support**: `support@dsecos.io`
