# DsecOS Epic 3: Kernel Customization and Hardening

This markdown file details Epic 3 from the DsecOS Development Tasks Document. The goal of this epic is to patch the kernel for KVM, OpenVZ/LXC, and SELinux, and apply system-wide hardening including firewall, encryption, auditing, and benchmarks. This is prioritized for MVP completion in Days 3-4 (November 8-9, 2025).

The structure follows the hierarchy: PRs > Commits > Tasks > Sub-Steps. Each sub-step is broken down into very simple, beginner-friendly actions (e.g., exact commands where possible, assuming a Linux/Debian-based development environment like Ubuntu). 

Prerequisites:
- Epics 1 and 2 are completed: Repo with Proxmox submodules, rebranded UI/CLI.
- Install kernel build tools: Run `sudo apt update && sudo apt install build-essential linux-headers-$(uname -r) bc bison flex libssl-dev libelf-dev dwarves` (for kernel compilation).
- For SELinux: `sudo apt install selinux-basics selinux-policy-default policycoreutils`.
- For testing: A virtual machine setup (e.g., VirtualBox or KVM on host) to boot the custom kernel safely.
- Git and text editor (nano/VS Code) from previous epics.
- Caution: Kernel building can take time (30-60min per compile); use a machine with multiple cores.

## PR 3.1: Kernel Patching
**Branch**: feature/kernel-patch  
**Description**: Patch the Proxmox kernel to enable KVM, LXC/OpenVZ modules, and add SELinux for enhanced security.  
**Estimated Time**: 6-8 hours (includes compile time).  
**Merge Criteria**: Kernel compiles successfully; modules load in a test VM; SELinux status shows enforcing.

### Commit 3.1.1: Enable KVM and LXC/OpenVZ
**Tasks**:
1. Navigate to the kernel source directory.
   - Open your terminal.
   - Change to the repo root: `cd ~/projects/dsecos-private` (adjust path if needed).
   - Navigate to the Proxmox kernel submodule: `cd kernel/pve-kernel`.

2. Copy the kernel source for customization.
   - Run: `cp -r . ../custom-kernel` (to work in a new dir: kernel/custom-kernel).
   - Change dir: `cd ../custom-kernel`.

3. Prepare for configuration.
   - If no .config: Run `make olddefconfig` or copy from /boot/config-$(uname -r) if applicable.
   - Run: `make menuconfig` (install ncurses if needed: `sudo apt install libncurses5-dev`).

4. Enable KVM in config.
   - In menuconfig: Use arrow keys to navigate to "Virtualization".
   - Select "Kernel-based Virtual Machine (KVM)" and press Y to set =y.
   - Under KVM, enable Intel/AMD support as needed (e.g., KVM_INTEL=y).
   - Save and exit (select <Exit> until back, then <Yes> to save .config).

5. Enable LXC/OpenVZ.
   - Re-run `make menuconfig` if needed.
   - Navigate to "General setup" > "Control Group support" (enable if not).
   - Search (/) for "LXC": Enable CONFIG_NAMESPACES, CONFIG_UTS_NS, etc.
   - For OpenVZ compat: Enable CONFIG_VZ (if available; else use LXC equivalents like CONFIG_CGROUPS).
   - Save and exit.

6. Compile the kernel.
   - Run: `make -j$(nproc) bzImage` (for the kernel image; use -j for parallel).
   - Then: `make -j$(nproc) modules` (for modules).
   - This may take 30+ minutes; monitor with `top`.

7. Test in a VM.
   - Copy bzImage to a test VM (e.g., via scp or shared folder).
   - Boot VM with custom kernel (edit GRUB or use qemu: `qemu-system-x86_64 -kernel bzImage` for basic test).
   - In VM: Run `lsmod | grep kvm` to check KVM loaded.
   - Run `lsmod | grep vz` or check LXC with `lxc-checkconfig`.

8. Stage and commit.
   - From custom-kernel dir: `cd ../..` to root.
   - Run: `git add kernel/custom-kernel`.
   - Run: `git commit -m "Patch kernel for KVM and OpenVZ/LXC"`.
   - Push to branch.

### Commit 3.1.2: Add SELinux Support
**Tasks**:
1. Re-enter the custom kernel directory.
   - From repo root: `cd kernel/custom-kernel`.

2. Download SELinux patches if needed.
   - Open browser, go to kernel.org or selinuxproject.org.
   - Search "SELinux kernel patches for Linux 6.x" (Proxmox kernel version).
   - Download patch file (e.g., selinux.patch) to local: Save to ~/Downloads, then `cp ~/Downloads/selinux.patch .`.

3. Apply the patch.
   - Run: `patch -p1 < selinux.patch` (adjust if errors; dry-run first: --dry-run).
   - If manual: Edit files as per patch hunks using `nano`.

4. Configure SELinux in kernel.
   - Run: `make menuconfig`.
   - Navigate to "Security options".
   - Enable "NSA SELinux Support" (CONFIG_SECURITY_SELINUX=y).
   - Enable boot-time disable if desired (CONFIG_SECURITY_SELINUX_BOOTPARAM=y).
   - Save and exit.

5. Recompile the kernel.
   - Run: `make -j$(nproc) clean` (to reset).
   - Then: `make -j$(nproc) bzImage && make -j$(nproc) modules`.
   - Install modules if testing: `make modules_install` (in VM or carefully on host).

6. Install SELinux tools.
   - In your dev env or test VM: `sudo apt install policycoreutils selinux-utils setools`.

7. Test SELinux.
   - Boot with custom kernel in VM.
   - Run: `sestatus` (should show "SELinux status: enabled" and "Current mode: enforcing").
   - If permissive: `setenforce 1`.

8. Stage and commit.
   - From root: `git add kernel/custom-kernel`.
   - Run: `git commit -m "Add SELinux to custom kernel"`.
   - Push.

**Merge PR 3.1**:
1. Create the branch if not: `git checkout -b feature/kernel-patch`.
2. Push branch: `git push origin feature/kernel-patch`.
3. On GitHub, create PR from feature/kernel-patch to main.
4. Review: Compile logs clean; VM boot test with modules/SELinux.
5. Merge.

## PR 3.2: System Hardening
**Branch**: feature/hardening  
**Description**: Implement firewall rules, encryption, auditing, intrusion detection, and run security benchmarks.  
**Estimated Time**: 6-8 hours.  
**Merge Criteria**: Hardening scripts run without errors; benchmarks score >90%; test VM passes scans.

### Commit 3.2.1: Firewall and Encryption Setup
**Tasks**:
1. Create hardening scripts directory.
   - From repo root: `mkdir scripts/hardening`.
   - Change dir: `cd scripts/hardening`.

2. Set up nftables firewall script.
   - Run: `nano firewall-setup.sh`.
   - Add: `#!/bin/bash\nnft flush ruleset\nnft add table inet filter\nnft add chain inet filter input { type filter hook input priority 0; policy drop; }\n# Add rules: nft add rule inet filter input tcp dport 22 accept`, etc. (default deny-all, allow essentials).
   - Make executable: `chmod +x firewall-setup.sh`.

3. Add LUKS encryption setup.
   - Run: `nano encryption-setup.sh`.
   - Add: `#!/bin/bash\ncryptsetup luksFormat /dev/sda1\ncryptsetup luksOpen /dev/sda1 encrypted\nmkfs.ext4 /dev/mapper/encrypted` (example for storage; integrate into installer).
   - Note: Test on non-critical disk/VM.

4. Integrate into build.
   - Edit Packer template (packer/dsecos.json): Add provisioner to run these scripts.

5. Test scripts.
   - In a test VM: Run `./firewall-setup.sh`, check `nft list ruleset`.
   - For encryption: Simulate on loopback device (`dd if=/dev/zero of=test.img bs=1M count=100; losetup /dev/loop0 test.img`).

6. Stage and commit.
   - From root: `git add scripts/hardening packer`.
   - Run: `git commit -m "Implement firewall and encryption hardening"`.
   - Push.

### Commit 3.2.2: Auditing and Intrusion Detection
**Tasks**:
1. Add auditing setup script.
   - From scripts/hardening: `nano auditing-setup.sh`.
   - Add: `#!/bin/bash\napt install auditd -y\nauditctl -a always,exit -F arch=b64 -S execve -k exec_log` (example rules).
   - Configure rsyslog: `echo "*.* /var/log/audit/audit.log" >> /etc/rsyslog.conf`.

2. Install Fail2Ban and OSSEC.
   - In script: `apt install fail2ban ossec-hids -y`.
   - Config Fail2Ban: `cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local; nano /etc/fail2ban/jail.local` (enable sshd).

3. Set up OSSEC.
   - Add: `ossec-config --enable-agentless` (basic; edit /var/ossec/etc/ossec.conf for alerts).

4. Test detection.
   - Run script in VM.
   - Simulate attack: Failed SSH logins, check /var/log/fail2ban.log.
   - Check OSSEC alerts: /var/ossec/logs/alerts.log.

5. Stage and commit.
   - From root: `git add scripts/hardening`.
   - Run: `git commit -m "Add auditing and intrusion detection"`.
   - Push.

### Commit 3.2.3: Run Lynis/OpenSCAP Benchmarks
**Tasks**:
1. Install Lynis.
   - From root: `git clone https://github.com/CISOfy/lynis.git tools/lynis`.
   - Change dir: `cd tools/lynis`.

2. Run Lynis scan.
   - Run: `./lynis audit system`.
   - Review output; aim for 90%+ score.
   - Fix suggestions: e.g., Edit configs based on warnings.

3. Install OpenSCAP.
   - Run: `sudo apt install openscap-scanner scap-security-guide`.

4. Run OpenSCAP scan.
   - Run: `oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis /usr/share/xml/scap/ssg/content/ssg-debian12-ds.xml` (for Debian; adjust profile).
   - Review report; fix issues.

5. Automate in script.
   - Nano scripts/hardening/benchmark.sh: Add commands to run scans and log results.

6. Stage and commit.
   - From root: `git add tools/lynis scripts/hardening`.
   - Run: `git commit -m "Benchmark with Lynis/OpenSCAP"`.
   - Push.

**Merge PR 3.2**:
1. Create branch: `git checkout -b feature/hardening`.
2. Push: `git push origin feature/hardening`.
3. Create PR on GitHub.
4. Review: Scripts execute; scans pass thresholds.
5. Merge to main.

## Next Steps After Epic 3
- Proceed to Epic 4 (Integrations and Features).
- Daily check: Build custom ISO with Packer, boot in VM, verify hardening.
