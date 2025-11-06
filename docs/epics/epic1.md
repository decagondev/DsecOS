# DsecOS Epic 1: Project Setup and Forking

This markdown file details Epic 1 from the DsecOS Development Tasks Document. The goal of this epic is to set up the private GitHub repository, initialize the project structure, integrate basic CI/CD, and fork the Proxmox VE source code. This is prioritized for MVP completion in Days 1-2 (November 6-7, 2025).

The structure follows the hierarchy: PRs > Commits > Tasks > Sub-Steps. Each sub-step is broken down into very simple, beginner-friendly actions (e.g., exact commands where possible, assuming a Linux/Debian-based development environment like Ubuntu). 

Prerequisites:
- You have a GitHub account with access to create private repositories.
- Git is installed on your local machine (`sudo apt install git` if not).
- Basic tools like a text editor (e.g., nano or VS Code) and terminal access.
- Node.js/npm for any UI-related setups (install via `sudo apt install nodejs npm` if needed).

## PR 1.1: Initialize Repository and CI/CD
**Branch**: feature/init-repo  
**Description**: Create the private repo, set up basic structure, add CI/CD workflows, and integrate Packer for building ISOs.  
**Estimated Time**: 4-6 hours.  
**Merge Criteria**: Repo clones successfully; CI runs without errors.

### Commit 1.1.1: Create Repo Structure and README
**Tasks**:
1. Log in to GitHub and create the repository.
   - Open a web browser.
   - Go to github.com and sign in to your account.
   - Click the "+" icon in the top right and select "New repository".
   - Name it "dsecos-private" (or your chosen name).
   - Check the "Private" option.
   - Add a description: "Private repo for DsecOS hypervisor development".
   - Click "Create repository".
   - Copy the repo URL (e.g., git@github.com:your-org/dsecos-private.git).

2. Clone the repo locally.
   - Open your terminal.
   - Navigate to a working directory (e.g., `cd ~/projects`).
   - Run: `git clone git@github.com:your-org/dsecos-private.git`.
   - Change into the repo: `cd dsecos-private`.

3. Create directories for organization.
   - In the terminal, run: `mkdir -p src kernel ui docs scripts ansible`.
   - This creates folders like src/, kernel/, etc., nested if needed.

4. Add a README.md file.
   - Run: `nano README.md` (or open in your editor).
   - Paste in the project overview: Include the mission statement ("To provide a hardened hypervisor OS...") and tagline ("Secure Hypervisor for Modern Apps: Isolate, Deploy, Protect.") from the PRD.
   - Save and exit (Ctrl+O, Enter, Ctrl+X in nano).

5. Add a .gitignore file.
   - Run: `nano .gitignore`.
   - Add common ignores: Go to gitignore.io in a browser, generate for Python, Node, etc., and paste (e.g., *.pyc, node_modules/).
   - Save and exit.

6. Stage and commit changes.
   - Run: `git add .`.
   - Run: `git commit -m "Initial repo structure and README"`.
   - Push: `git push origin main`.

### Commit 1.1.2: Add GitHub Actions for CI
**Tasks**:
1. Create the workflows directory.
   - In the terminal, run: `mkdir -p .github/workflows`.

2. Add a basic CI YAML file.
   - Run: `nano .github/workflows/ci-build.yml`.
   - Paste a simple workflow:
     ```
     name: CI Build
     on: [push, pull_request]
     jobs:
       build:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v3
           - name: Lint scripts
             run: |
               sudo apt install shellcheck
               shellcheck scripts/*.sh || true
           - name: Test placeholder
             run: echo "Test passed"
     ```
   - Save and exit.

3. Test the YAML syntax locally if possible.
   - Install act if you have it (optional: `go install github.com/nektos/act@latest`), or just proceed.
   - Commit without pushing yet.

4. Stage and commit.
   - Run: `git add .github`.
   - Run: `git commit -m "Add basic CI/CD with GitHub Actions"`.
   - Push: `git push`.

5. Verify on GitHub.
   - Go to the repo on GitHub.
   - Click "Actions" tab.
   - See if the workflow runs successfully on the push.

### Commit 1.1.3: Integrate Packer for ISO Builds
**Tasks**:
1. Install Packer locally.
   - In terminal: `sudo apt update && sudo apt install packer` (or download from packer.io if needed).

2. Create a packer directory.
   - Run: `mkdir packer`.

3. Add a basic Packer template.
   - Run: `nano packer/dsecos.json` (or .pkr.hcl for newer format).
   - Paste a simple template for Debian base:
     ```
     {
       "builders": [{
         "type": "virtualbox-iso",
         "iso_url": "https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.7.0-amd64-netinst.iso",
         "iso_checksum": "sha256:your_checksum_here",
         "guest_os_type": "Debian_64",
         "ssh_username": "root",
         "ssh_password": "password",
         "shutdown_command": "shutdown -P now"
       }]
     }
     ```
     - Note: Replace checksum by looking it up on debian.org (open browser, search "debian 12 iso checksum", copy SHA256).

4. Add variables file if needed.
   - Run: `nano packer/variables.json`.
   - Add placeholders: `{ "proxmox_base": "8.x" }`.
   - Save.

5. Test Packer validate.
   - Run: `packer validate packer/dsecos.json`.

6. Stage and commit.
   - Run: `git add packer`.
   - Run: `git commit -m "Integrate Packer for ISO builds"`.
   - Push: `git push`.

**Merge PR 1.1**:
1. Create the branch first if not: `git checkout -b feature/init-repo`.
2. Push branch: `git push origin feature/init-repo`.
3. On GitHub, create PR from feature/init-repo to main.
4. Review: Check files, run CI.
5. Merge.

## PR 1.2: Fork Proxmox VE
**Branch**: feature/fork-proxmox  
**Description**: Import Proxmox source code as submodules and apply initial patches.  
**Estimated Time**: 4-6 hours.  
**Merge Criteria**: Submodules initialize correctly; basic build test passes.

### Commit 1.2.1: Clone and Import Proxmox Source
**Tasks**:
1. Identify Proxmox repositories.
   - Open browser, go to git.proxmox.com.
   - Note key repos: pve-manager, pve-kernel, pve-qemu-kvm, etc. (search "Proxmox VE repositories" if needed for list).

2. Add as Git submodules.
   - In terminal, from repo root: `git submodule add https://git.proxmox.com/git/pve-manager.git src/pve-manager`.
   - Repeat for others: `git submodule add https://git.proxmox.com/git/pve-kernel.git kernel/pve-kernel`.
   - Add at least: pve-qemu-kvm to src/pve-qemu, pve-container to src/pve-container (for LXC).

3. Initialize submodules.
   - Run: `git submodule update --init --recursive`.

4. Verify.
   - Run: `ls src/pve-manager` to see files.

5. Stage and commit.
   - Run: `git add .gitmodules src kernel`.
   - Run: `git commit -m "Import Proxmox VE source as submodules"`.
   - Push.

### Commit 1.2.2: Apply Initial Patches for Compatibility
**Tasks**:
1. Create a patch file.
   - Run: `nano patches/initial-compat.patch`.
   - Add example changes: e.g., update Debian version in a config file (diff format).

2. Apply the patch.
   - Navigate to a submodule: `cd src/pve-manager`.
   - Run: `git apply ../../patches/initial-compat.patch`.
   - If errors, edit manually: Use `nano somefile` to change (e.g., replace Debian 11 refs to 12 if needed).

3. Commit in submodule.
   - In submodule dir: `git add . && git commit -m "Apply compatibility patches"`.
   - Back to root: `cd ../..`.
   - Update main repo: `git add src/pve-manager`.

4. Test basic build.
   - Follow Proxmox build docs (browser: search "build Proxmox from source").
   - Run make or ./configure if applicable in submodule.

5. Stage and commit in main.
   - Run: `git commit -m "Apply initial compatibility patches to Proxmox fork"`.
   - Push.

**Merge PR 1.2**:
1. Create branch: `git checkout -b feature/fork-proxmox`.
2. Push: `git push origin feature/fork-proxmox`.
3. Create PR on GitHub.
4. Review: Init submodules locally, check for errors.
5. Merge to main.

## Next Steps After Epic 1
- Proceed to Epic 2 (Rebranding).
- Daily check: Run CI, test in a VM (e.g., install VirtualBox, create test machine).
