# DsecOS Development Tasks Document

This document outlines the granular tasks for building DsecOS based on the v1.0 PRD. The structure is hierarchical: **Epics** (high-level phases), **Pull Requests (PRs)** (feature branches merging into main), **Commits** (atomic code changes within a PR), and **Tasks** (step-by-step breakdowns suitable for beginners or LLMs). 

The timeline prioritizes an MVP by November 13, 2025 (one week from November 6, 2025), focusing on essentials like forking, rebranding, kernel patching, and basic integrations. Remaining features target completion by December 6, 2025 (one month total).

Assumptions: 
- Development in a private GitHub repo (e.g., github.com/your-org/dsecos-private).
- Tools: Git, Debian-based build env, Packer for ISOs, basic shell scripting.
- Testing: Local VMs (e.g., VirtualBox) for quick iterations.
- Team: Solo or small (beginner-friendly steps).

## Epic 1: Project Setup and Forking (MVP Priority - Days 1-2)
High-level: Set up the repo, fork Proxmox, and establish build pipelines.

### PR 1.1: Initialize Repository and CI/CD
- **Branch**: feature/init-repo
- **Commits**:
  - Commit 1.1.1: Create repo structure and README.
  - Commit 1.1.2: Add GitHub Actions for CI.
  - Commit 1.1.3: Integrate Packer for ISO builds.

- **Tasks for Commit 1.1.1**:
  1. Log in to GitHub and create a new private repository named "dsecos-private".
  2. Clone the repo locally: `git clone git@github.com:your-org/dsecos-private.git`.
  3. Create directories: `mkdir -p src kernel ui docs scripts ansible`.
  4. Add a README.md with project overview from PRD (copy-paste mission statement and tagline).
  5. Add .gitignore for common files (e.g., from gitignore.io for Python, Node).
  6. Commit: `git add . && git commit -m "Initial repo structure and README"`.

- **Tasks for Commit 1.1.2**:
  1. Create .github/workflows directory.
  2. Add a YAML file: ci-build.yml with steps to lint code (e.g., shellcheck for scripts).
  3. Add test job: Run `echo "Test passed"` as placeholder.
  4. Commit: `git add .github && git commit -m "Add basic CI/CD with GitHub Actions"`.

- **Tasks for Commit 1.1.3**:
  1. Install Packer locally if needed (e.g., `sudo apt install packer` on Debian).
  2. Add packer/ directory with a basic JSON template for Debian ISO.
  3. Configure variables for Proxmox base.
  4. Commit: `git add packer && git commit -m "Integrate Packer for ISO builds"`.

- **Merge PR**: Review for structure; merge to main.

### PR 1.2: Fork Proxmox VE
- **Branch**: feature/fork-proxmox
- **Commits**:
  - Commit 1.2.1: Clone and import Proxmox source.
  - Commit 1.2.2: Apply initial patches for compatibility.

- **Tasks for Commit 1.2.1**:
  1. Go to Proxmox VE Git repo (git.proxmox.com) and fork/clone the latest stable (e.g., 8.x).
  2. In local dsecos repo, add as submodule: `git submodule add https://git.proxmox.com/git/pve-manager.git src/pve-manager`.
  3. Do the same for other components: pve-kernel, pve-qemu, etc.
  4. Commit: `git add .gitmodules src && git commit -m "Import Proxmox VE source as submodules"`.

- **Tasks for Commit 1.2.2**:
  1. Create a patch file for minor fixes (e.g., update Debian version in configs).
  2. Apply: `git apply my-patch.patch` in relevant submodules.
  3. Commit: `git commit -m "Apply initial compatibility patches to Proxmox fork"`.

- **Merge PR**: Test local build; merge to main.

## Epic 2: Rebranding (MVP Priority - Days 2-3)
High-level: Remove Proxmox branding, apply dark theme, ensure 100% consistency.

### PR 2.1: UI Rebranding
- **Branch**: feature/ui-rebrand
- **Commits**:
  - Commit 2.1.1: Replace logos and text.
  - Commit 2.1.2: Apply dark color scheme.
  - Commit 2.1.3: Evaluate Vite/React upgrade (MVP hybrid).

- **Tasks for Commit 2.1.1**:
  1. Navigate to src/pve-manager/www (ExtJS UI files).
  2. Find logo.png/svg; replace with custom DsecOS logo (create simple one via tool like Canva if needed).
  3. Search/replace "Proxmox" with "DsecOS" in JS/HTML files (use `grep -rl Proxmox | xargs sed -i 's/Proxmox/DsecOS/g'`).
  4. Commit: `git commit -m "Replace logos and text branding"`.

- **Tasks for Commit 2.1.2**:
  1. Edit CSS files (e.g., manager6/css/ext6-pve.css).
  2. Change base colors: Background #121212, accents #00BFFF (use hex replacements).
  3. Test in browser: Build UI and load locally.
  4. Commit: `git commit -m "Apply dark color scheme to UI"`.

- **Tasks for Commit 2.1.3**:
  1. Assess ExtJS files; identify key components (e.g., dashboard).
  2. Set up Vite/React in ui/ directory: `npm init vite@latest -- --template react`.
  3. For MVP, patch one component (e.g., login page) as hybrid.
  4. Estimate full rewrite: 2-4 weeks (doc in comments).
  5. Commit: `git commit -m "Hybrid Vite/React setup for UI upgrade evaluation"`.

- **Merge PR**: Visual inspection for no Proxmox remnants; merge.

### PR 2.2: CLI and Docs Rebranding
- **Branch**: feature/cli-rebrand
- **Commits**:
  - Commit 2.2.1: Rename CLI tools.
  - Commit 2.2.2: Update documentation.

- **Tasks for Commit 2.2.1**:
  1. Find Proxmox CLI (e.g., pveum, pct).
  2. Create wrappers: Script `dsecos` that calls them with rebranded output.
  3. Replace strings in help texts.
  4. Commit: `git commit -m "Rebrand CLI tools to dsecos"`.

- **Tasks for Commit 2.2.2**:
  1. Copy Proxmox docs to docs/.
  2. Search/replace branding.
  3. Add PRD as docs/prd.md.
  4. Commit: `git commit -m "Rebrand and update documentation"`.

- **Merge PR**: Test CLI commands; merge.

## Epic 3: Kernel Customization and Hardening (MVP Priority - Days 3-4)
High-level: Patch kernel for KVM/OpenVZ, add SELinux, apply security configs.

### PR 3.1: Kernel Patching
- **Branch**: feature/kernel-patch
- **Commits**:
  - Commit 3.1.1: Enable KVM and LXC/OpenVZ.
  - Commit 3.1.2: Add SELinux support.

- **Tasks for Commit 3.1.1**:
  1. Clone Proxmox kernel source to kernel/.
  2. Edit .config: Set CONFIG_KVM=y, CONFIG_VZ=y (or LXC equivalents).
  3. Compile: `make -j$(nproc)`.
  4. Test in VM: Boot and check `lsmod | grep kvm`.
  5. Commit: `git commit -m "Patch kernel for KVM and OpenVZ/LXC"`.

- **Tasks for Commit 3.1.2**:
  1. Add SELinux patches: Download from kernel.org if needed.
  2. Configure: Set CONFIG_SECURITY_SELINUX=y.
  3. Recompile and install policy tools (e.g., policycoreutils).
  4. Commit: `git commit -m "Add SELinux to custom kernel"`.

- **Merge PR**: Boot test; merge.

### PR 3.2: System Hardening
- **Branch**: feature/hardening
- **Commits**:
  - Commit 3.2.1: Firewall and encryption setup.
  - Commit 3.2.2: Auditing and intrusion detection.
  - Commit 3.2.3: Run Lynis/OpenSCAP benchmarks.

- **Tasks for Commit 3.2.1**:
  1. Script nftables rules: Default deny-all.
  2. Add LUKS setup in installer: Use cryptsetup.
  3. Commit: `git commit -m "Implement firewall and encryption hardening"`.

- **Tasks for Commit 3.2.2**:
  1. Install Fail2Ban and OSSEC via scripts.
  2. Config logging to /var/log/audit.
  3. Commit: `git commit -m "Add auditing and intrusion detection"`.

- **Tasks for Commit 3.2.3**:
  1. Install Lynis: `git clone https://github.com/CISOfy/lynis`.
  2. Run scan and aim for 90%+ score.
  3. Adjust configs based on output.
  4. Commit: `git commit -m "Benchmark with Lynis/OpenSCAP"`.

- **Merge PR**: Security scan pass; merge.

## Epic 4: Integrations and Features (MVP Priority - Days 4-6)
High-level: Add Docker/Portainer, stacks, PXE, Ansible.

### PR 4.1: Container and Stack Support
- **Branch**: feature/containers
- **Commits**:
  - Commit 4.1.1: Embed Docker and Portainer.
  - Commit 4.1.2: Create stack templates.

- **Tasks for Commit 4.1.1**:
  1. Add Docker install script: Curl from get.docker.com.
  2. Integrate Portainer: Docker run command in setup.
  3. Test: `docker ps` and access UI.
  4. Commit: `git commit -m "Embed Docker and Portainer"`.

- **Tasks for Commit 4.1.2**:
  1. Create templates dir: YAML for Node 20.x, Java 21, etc.
  2. Include DB: e.g., docker-compose with Postgres 16.
  3. Add deploy command in CLI.
  4. Commit: `git commit -m "Add granular stack templates"`.

- **Merge PR**: Deploy test; merge.

### PR 4.2: Provisioning and Automation
- **Branch**: feature/provisioning
- **Commits**:
  - Commit 4.2.1: Add PXE bare-metal support.
  - Commit 4.2.2: Integrate Ansible playbooks.

- **Tasks for Commit 4.2.1**:
  1. Set up PXE server config in scripts/.
  2. Use pxelinux for boot menu.
  3. Test network install in VM.
  4. Commit: `git commit -m "Implement PXE bare-metal provisioning"`.

- **Tasks for Commit 4.2.2**:
  1. Add ansible/ dir with playbooks (e.g., harden.yml).
  2. Inventory file for local host.
  3. Run: `ansible-playbook harden.yml`.
  4. Commit: `git commit -m "Integrate Ansible for automation"`.

- **Merge PR**: Provision test; merge.

## Epic 5: Testing, Docs, and Release (Days 6-7 for MVP; Weeks 2-4 for Full)
High-level: Validate, document, package MVP.

### PR 5.1: Testing Suite
- **Branch**: feature/testing
- **Commits**:
  - Commit 5.1.1: Unit and integration tests.
  - Commit 5.1.2: Security and perf tests.

- **Tasks for Commit 5.1.1**:
  1. Add tests/ dir with pytest for CLI.
  2. Write test_deploy.py.
  3. Run and fix.
  4. Commit: `git commit -m "Add unit/integration tests"`.

- **Tasks for Commit 5.1.2**:
  1. Fuzz kernel with syzkaller (setup basic).
  2. Benchmark IOPS with fio.
  3. Commit: `git commit -m "Add security/performance tests"`.

- **Merge PR**: All tests pass; merge.

### PR 5.2: Documentation and Release
- **Branch**: feature/release
- **Commits**:
  - Commit 5.2.1: Final docs.
  - Commit 5.2.2: Build MVP ISO.

- **Tasks for Commit 5.2.1**:
  1. Expand docs/ with install guide.
  2. Add compliance notes.
  3. Commit: `git commit -m "Finalize documentation"`.

- **Tasks for Commit 5.2.2**:
  1. Run Packer to build ISO.
  2. Test boot in VM.
  3. Tag v0.1-mvp.
  4. Commit: `git commit -m "Build and tag MVP ISO"`.

- **Merge PR**: Release ready; merge.

## Post-MVP Epics (Weeks 2-4)
- Epic 6: Advanced Features (e.g., API enhancements, full React UI).
- Epic 7: Compliance and Beta Testing (e.g., HIPAA modules, user feedback).
- Follow similar PR/Commit/Task breakdown, iterating on MVP feedback.
