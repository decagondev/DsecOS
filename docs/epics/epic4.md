# DsecOS Epic 4: Integrations and Features

This markdown file details **Epic 4** from the DsecOS Development Tasks Document. The goal of this epic is to integrate Docker and Portainer, create granular application stack templates (Node.js, Java, Python, Ruby with optional databases), implement PXE bare-metal provisioning, and add Ansible automation playbooks. This is prioritized for **MVP completion in Days 4–6 (November 9–11, 2025)**.

The structure follows the hierarchy: **PRs > Commits > Tasks > Sub-Steps**. Each sub-step is broken down into **very simple, beginner-friendly actions** with exact commands, assuming a **Linux/Debian-based development environment** (e.g., Ubuntu).

---

## Prerequisites
- Epics 1–3 completed: Repo initialized, Proxmox forked, rebranded, kernel patched with KVM/LXC/SELinux, hardening in place.
- Tools installed:
  ```bash
  sudo apt update && sudo apt install -y docker.io docker-compose curl jq git ansible
  ```
- Test environment: A **dedicated VM** (e.g., VirtualBox or KVM) for integration testing.
- Network access for pulling Docker images and templates.
- Git and text editor (nano/VS Code) ready.

---

## PR 4.1: Container and Stack Support  
**Branch**: `feature/containers`  
**Description**: Embed Docker runtime, integrate Portainer UI, and create one-click stack templates.  
**Estimated Time**: 6–8 hours  
**Merge Criteria**: Docker runs, Portainer accessible, at least 2 stack templates deploy successfully.

---

### Commit 4.1.1: Embed Docker and Portainer

**Tasks**:

1. **Create Docker setup script**  
   - From repo root:  
     ```bash
     mkdir -p scripts/containers
     cd scripts/containers
     nano install-docker.sh
     ```
   - Paste:
     ```bash
     #!/bin/bash
     echo "Installing Docker..."
     curl -fsSL https://get.docker.com | sh
     systemctl enable --now docker
     usermod -aG docker $SUDO_USER
     echo "Docker installed and running."
     ```
   - Save (`Ctrl+O`, `Enter`, `Ctrl+X`), then:
     ```bash
     chmod +x install-docker.sh
     ```

2. **Create Portainer installation script**  
   ```bash
   nano install-portainer.sh
   ```
   - Paste:
     ```bash
     #!/bin/bash
     echo "Deploying Portainer..."
     docker volume create portainer_data
     docker run -d -p 9000:9000 -p 9443:9443 \
       --name portainer \
       --restart=always \
       -v /var/run/docker.sock:/var/run/docker.sock \
       -v portainer_data:/data \
       portainer/portainer-ce:latest
     echo "Portainer running on https://localhost:9443"
     ```
   - Save and:
     ```bash
     chmod +x install-portainer.sh
     ```

3. **Integrate into Packer build**  
   - Edit Packer template:
     ```bash
     cd ../../../packer
     nano dsecos.json
     ```
   - Add a provisioner:
     ```json
     {
       "type": "shell",
       "script": "../scripts/containers/install-docker.sh"
     },
     {
       "type": "shell",
       "script": "../scripts/containers/install-portainer.sh"
     }
     ```
   - Save.

4. **Test in VM**  
   - Build ISO with Packer (later in Epic 5), or test scripts manually:
     ```bash
     sudo ./scripts/containers/install-docker.sh
     sudo ./scripts/containers/install-portainer.sh
     ```
   - Check:
     ```bash
     docker ps | grep portainer
     ```
   - Open browser: `https://your-vm-ip:9443` → Accept self-signed cert → Set admin password.

5. **Stage and commit**  
   ```bash
   cd ../../..
   git add scripts/containers packer
   git commit -m "Embed Docker and Portainer"
   git push
   ```

---

### Commit 4.1.2: Create Stack Templates

**Tasks**:

1. **Create templates directory**  
   ```bash
   mkdir -p templates/stacks
   cd templates/stacks
   ```

2. **Node.js + PostgreSQL template**  
   ```bash
   nano node-postgres.yml
   ```
   - Paste:
     ```yaml
     version: '3.8'
     services:
       app:
         image: node:20-alpine
         working_dir: /app
         volumes:
           - ./app:/app
         ports:
           - "3000:3000"
         command: sh -c "npm install && npm start"
         depends_on:
           - db
       db:
         image: postgres:16-alpine
         environment:
           POSTGRES_DB: myapp
           POSTGRES_USER: user
           POSTGRES_PASSWORD: securepass
         volumes:
           - pgdata:/var/lib/postgresql/data
         ports:
           - "5432:5432"
     volumes:
       pgdata:
     ```
   - Save.

3. **Java (Spring Boot) + MySQL template**  
   ```bash
   nano java-mysql.yml
   ```
   - Paste minimal:
     ```yaml
     version: '3.8'
     services:
       app:
         image: openjdk:21-jdk-slim
         working_dir: /app
         volumes:
           - ./app:/app
         ports:
           - "8080:8080"
         command: java -jar app.jar
       db:
         image: mysql:8
         environment:
           MYSQL_ROOT_PASSWORD: rootpass
           MYSQL_DATABASE: appdb
         ports:
           - "3306:3306"
     ```

4. **Python + Redis and Ruby + SQLite templates**  
   - Repeat pattern:
     ```bash
     nano python-redis.yml
     nano ruby-sqlite.yml
     ```
   - Use official images: `python:3.12`, `redis:7`, `ruby:3.3`, `sqlite:3`.

5. **Add CLI deploy command**  
   ```bash
   cd ../../scripts/cli
   nano dsecos-deploy
   ```
   - Paste:
     ```bash
     #!/bin/bash
     STACK=$1
     if [ -z "$STACK" ]; then
       echo "Usage: dsecos deploy <stack-name>"
       exit 1
     fi
     docker-compose -f /templates/stacks/$STACK.yml up -d
     ```
   - Make executable:
     ```bash
     chmod +x dsecos-deploy
     ```

6. **Test deployment**  
   - Copy a template to test dir:
     ```bash
     mkdir ~/test-stack && cp node-postgres.yml ~/test-stack/docker-compose.yml
     cd ~/test-stack
     docker-compose up -d
     ```
   - Verify: `docker ps`, check ports.

7. **Stage and commit**  
   ```bash
   cd ../../../..
   git add templates scripts/cli
   git commit -m "Add granular stack templates"
   git push
   ```

---

**Merge PR 4.1**:
```bash
git checkout -b feature/containers
git push origin feature/containers
```
- On GitHub: Create PR → Review scripts → Test in VM → Merge.

---

## PR 4.2: Provisioning and Automation  
**Branch**: `feature/provisioning`  
**Description**: Add PXE bare-metal boot and Ansible playbooks for setup/hardening.  
**Estimated Time**: 6–8 hours  
**Merge Criteria**: PXE boots ISO, Ansible runs hardening playbook.

---

### Commit 4.2.1: Add PXE Bare-Metal Support

**Tasks**:

1. **Create PXE server config**  
   ```bash
   mkdir -p scripts/pxe
   cd scripts/pxe
   ```

2. **Download pxelinux**  
   ```bash
   mkdir -p tftpboot/pxelinux.cfg
   curl -L -o tftpboot/pxelinux.0 http://deb.debian.org/debian/pool/main/s/syslinux/syslinux-common_6.04~git20190206.bf6db5b4+dfsg1-3_all.deb
   # Extract manually or use syslinux package
   sudo apt install syslinux pxelinux
   cp /usr/lib/PXELINUX/pxelinux.0 tftpboot/
   ```

3. **Create boot menu**  
   ```bash
   nano tftpboot/pxelinux.cfg/default
   ```
   - Paste:
     ```
     DEFAULT dsecos
     LABEL dsecos
       KERNEL vmlinuz
       APPEND initrd=initrd.img boot=live
     ```

4. **Add kernel and initrd from ISO**  
   - After building ISO (Epic 5), extract:
     ```bash
     mkdir iso-mount
     sudo mount -o loop dsecos.iso iso-mount
     cp iso-mount/live/vmlinuz tftpboot/
     cp iso-mount/live/initrd.img tftpboot/
     sudo umount iso-mount
     ```

5. **Start TFTP server**  
   ```bash
   sudo apt install tftpd-hpa
   sudo nano /etc/default/tftpd-hpa
   # Set TFTP_DIRECTORY="/path/to/dsecos-private/scripts/pxe/tftpboot"
   sudo systemctl restart tftpd-hpa
   ```

6. **Test PXE boot**  
   - In VM: Set network boot (PXE), boot → should load DsecOS menu.

7. **Stage and commit**  
   ```bash
   cd ../../..
   git add scripts/pxe
   git commit -m "Implement PXE bare-metal provisioning"
   git push
   ```

---

### Commit 4.2.2: Integrate Ansible Playbooks

**Tasks**:

1. **Initialize Ansible structure**  
   ```bash
   mkdir -p ansible/{playbooks,inventory,roles}
   cd ansible
   ```

2. **Create inventory**  
   ```bash
   nano inventory/hosts.ini
   ```
   - Paste:
     ```ini
     [dsecos]
     localhost ansible_connection=local
     ```

3. **Create hardening playbook**  
   ```bash
   nano playbooks/harden.yml
   ```
   - Paste:
     ```yaml
     ---
     - name: Harden DsecOS
       hosts: dsecos
       become: yes
       tasks:
         - name: Disable root login
           lineinfile:
             path: /etc/ssh/sshd_config
             regexp: '^PermitRootLogin'
             line: 'PermitRootLogin no'
         - name: Update system
           apt:
             upgrade: dist
             update_cache: yes
     ```

4. **Create stack deployment role**  
   ```bash
   mkdir -p roles/deploy_stack/tasks
   nano roles/deploy_stack/tasks/main.yml
   ```
   - Paste:
     ```yaml
     - name: Deploy stack
       docker_compose:
         project_src: /templates/stacks/{{ stack_name }}
         state: present
     ```

5. **Test Ansible**  
   ```bash
   ansible-playbook -i inventory/hosts.ini playbooks/harden.yml
   ```

6. **Stage and commit**  
   ```bash
   cd ../..
   git add ansible
   git commit -m "Integrate Ansible for automation"
   git push
   ```

---

**Merge PR 4.2**:
```bash
git checkout -b feature/provisioning
git push origin feature/provisioning
```
- On GitHub: Create PR → Test PXE + Ansible → Merge.

---

## Next Steps After Epic 4
- Proceed to **Epic 5: Testing, Docs, and Release** (Days 6–7).
- **Daily MVP Check**:  
  - Boot VM from PXE  
  - Run `dsecos deploy node-postgres`  
  - Access Portainer  
  - Run `ansible-playbook harden.yml`

**MVP Goal by Nov 11, 2025**: Fully functional, rebranded, secure, deployable DsecOS with Docker + stacks.
