# DsecOS Epic 5: Testing, Docs, and Release

This markdown file details **Epic 5** from the DsecOS Development Tasks Document. The goal of this epic is to implement a testing suite (unit, integration, security, and performance), finalize documentation, and build and tag the **MVP ISO** for release. This is prioritized for **MVP completion in Days 6–7 (November 12–13, 2025)** — **critical deadline**.

The structure follows: **PRs > Commits > Tasks > Sub-Steps**. Each sub-step is **very simple, beginner-friendly**, with **exact commands**, assuming a **Linux/Debian-based dev environment**.

---

## Prerequisites
- Epics 1–4 completed:  
  - Repo + Proxmox fork  
  - Rebranded UI/CLI  
  - Custom kernel (KVM, LXC, SELinux)  
  - Hardening (nftables, LUKS, OSSEC)  
  - Docker + Portainer + stack templates  
  - PXE + Ansible  
- Tools installed:
  ```bash
  sudo apt install -y python3-pip qemu-kvm virt-manager fio stress-ng
  pip3 install pytest
  ```
- **Test VM cluster**: At least 2 VMs (one for host, one for bare-metal sim via PXE).
- **Packer ISO build** ready from previous epics.

---

## PR 5.1: Testing Suite  
**Branch**: `feature/testing`  
**Description**: Add unit/integration tests, security scans, and performance benchmarks.  
**Estimated Time**: 6–8 hours  
**Merge Criteria**: All tests pass; Lynis ≥90%; IOPS >10,000 on SSD.

---

### Commit 5.1.1: Unit and Integration Tests

**Tasks**:

1. **Create test directory**  
   ```bash
   mkdir -p tests/{unit,integration}
   cd tests/unit
   ```

2. **Write CLI test (pytest)**  
   ```bash
   nano test_cli.py
   ```
   - Paste:
     ```python
     import subprocess
     import os

     def test_dsecos_deploy_help():
         result = subprocess.run(
             ['scripts/cli/dsecos-deploy'], 
             capture_output=True, text=True
         )
         assert "Usage: dsecos deploy" in result.stdout

     def test_docker_running():
         result = subprocess.run(
             ['docker', 'ps'], capture_output=True, text=True
         )
         assert result.returncode == 0
         assert 'portainer' in result.stdout
     ```
   - Save.

3. **Write integration test: stack deploy**  
   ```bash
   cd ../integration
   nano test_stack_deploy.py
   ```
   - Paste:
     ```python
     import subprocess
     import time

     def test_deploy_node_postgres():
         subprocess.run(['cp', '../../templates/stacks/node-postgres.yml', 'docker-compose.yml'])
         subprocess.run(['docker-compose', 'up', '-d'])
         time.sleep(10)
         ps = subprocess.run(['docker', 'ps'], capture_output=True, text=True)
         assert 'node' in ps.stdout
         assert 'postgres' in ps.stdout
         subprocess.run(['docker-compose', 'down', '-v'])
     ```

4. **Run tests**  
   ```bash
   cd ../..
   pytest -v
   ```
   - Fix any failures.

5. **Add to CI (GitHub Actions)**  
   ```bash
   nano .github/workflows/ci-tests.yml
   ```
   - Paste:
     ```yaml
     name: Tests
     on: [push, pull_request]
     jobs:
       test:
         runs-on: ubuntu-latest
         steps:
           - uses: actions/checkout@v3
           - name: Set up Python
             uses: actions/setup-python@v4
             with:
               python-version: '3.11'
           - name: Install deps
             run: |
               pip install pytest
               sudo apt update && sudo apt install -y docker.io
           - name: Run tests
             run: pytest -v
     ```

6. **Stage and commit**  
   ```bash
   git add tests .github/workflows/ci-tests.yml
   git commit -m "Add unit/integration tests"
   git push
   ```

---

### Commit 5.1.2: Security and Performance Tests

**Tasks**:

1. **Run Lynis audit**  
   ```bash
   cd tools/lynis
   sudo ./lynis audit system > ../../reports/lynis-report.txt
   ```
   - Open report: `grep "score" reports/lynis-report.txt` → **Target: ≥90%**

2. **Run OpenSCAP**  
   ```bash
   sudo oscap xccdf eval \
     --profile xccdf_org.ssgproject.content_profile_cis \
     --results results.xml \
     /usr/share/xml/scap/ssg/content/ssg-debian12-ds.xml
   ```
   - Fix failures (e.g., SSH config, file perms).

3. **Performance: IOPS with fio**  
   ```bash
   sudo apt install fio
   fio --name=write-test \
       --filename=testfile \
       --size=1G \
       --rw=write \
       --bs=4k \
       --numjobs=1 \
       --iodepth=32 \
       --runtime=30 \
       --time_based \
       --group_reporting
   ```
   - **Target**: >10,000 IOPS on SSD.

4. **Stress test**  
   ```bash
   stress-ng --cpu 4 --vm 2 --timeout 60s
   ```

5. **Automate benchmarks**  
   ```bash
   cd ../../scripts/hardening
   nano run-benchmarks.sh
   ```
   - Paste:
     ```bash
     #!/bin/bash
     echo "Running Lynis..."
     cd ../../tools/lynis && ./lynis audit system > ../../reports/lynis.txt
     echo "Running fio..."
     fio --name=bench --output=../../reports/fio.txt [config above]
     ```
   - Make executable: `chmod +x run-benchmarks.sh`

6. **Stage and commit**  
   ```bash
   git add reports/ scripts/hardening/run-benchmarks.sh
   git commit -m "Add security/performance tests"
   git push
   ```

---

**Merge PR 5.1**:
```bash
git checkout -b feature/testing
git push origin feature/testing
```
- GitHub: PR → CI passes → Lynis ≥90% → Merge.

---

## PR 5.2: Documentation and Release  
**Branch**: `feature/release`  
**Description**: Finalize docs, build MVP ISO, tag v0.1-mvp.  
**Estimated Time**: 4–6 hours  
**Merge Criteria**: ISO boots, all features work, docs complete.

---

### Commit 5.2.1: Final Documentation

**Tasks**:

1. **Create docs structure**  
   ```bash
   mkdir -p docs/{install,usage,security,dev}
   ```

2. **Installation Guide**  
   ```bash
   nano docs/install/iso-install.md
   ```
   - Paste:
     ```markdown
     # DsecOS Installation (ISO)

     1. Download `dsecos-v0.1-mvp.iso`
     2. Burn to USB or load in VM
     3. Boot → Follow wizard
     4. Login: `root` / `dsecos`
     ```

3. **PXE Install Guide**  
   ```bash
   nano docs/install/pxe-install.md
   ```
   - Include TFTP setup steps.

4. **Usage: Deploy Stack**  
   ```bash
   nano docs/usage/deploy-stack.md
   ```
   - Example:
     ```bash
     dsecos deploy node-postgres
     ```

5. **Security & Compliance**  
   ```bash
   nano docs/security/hardening.md
   ```
   - List: SELinux, nftables, LUKS, OSSEC, Lynis score.

6. **Add PRD**  
   ```bash
   cp ../prd.md docs/prd.md
   ```

7. **Stage and commit**  
   ```bash
   git add docs
   git commit -m "Finalize documentation"
   git push
   ```

---

### Commit 5.2.2: Build and Tag MVP ISO

**Tasks**:

1. **Update Packer config**  
   ```bash
   cd packer
   nano dsecos.pkr.hcl  # or .json
   ```
   - Ensure all provisioners run:
     - Docker + Portainer
     - Hardening scripts
     - PXE files
     - Custom kernel

2. **Build ISO**  
   ```bash
   packer build dsecos.pkr.hcl
   ```
   - Output: `output-qemu/dsecos-v0.1-mvp.iso`

3. **Test ISO boot**  
   - In VirtualBox/KVM:
     - Create VM → Attach ISO → Boot
     - Verify:
       - UI loads (dark theme, DsecOS logo)
       - `dsecos --version`
       - Portainer on port 9443
       - `docker ps`

4. **Tag release**  
   ```bash
   git tag -a v0.1-mvp -m "MVP Release - All core features"
   git push origin v0.1-mvp
   ```

5. **Upload ISO to GitHub Release**  
   - On GitHub: Releases → Draft new → v0.1-mvp → Attach ISO → Publish.

6. **Stage and commit**  
   ```bash
   git add packer output-qemu
   git commit -m "Build and tag MVP ISO"
   git push
   ```

---

**Merge PR 5.2**:
```bash
git checkout -b feature/release
git push origin feature/release
```
- GitHub: PR → ISO boots → Docs complete → Merge → **MVP ACHIEVED**

---

## MVP Validation Checklist (Nov 13, 2025)

| Feature                     | Status |
|----------------------------|--------|
| ISO boots with custom kernel | ☐     |
| UI: Dark theme, no Proxmox  | ☐     |
| `dsecos deploy node-postgres` | ☐     |
| Portainer accessible        | ☐     |
| PXE boot works              | ☐     |
| Ansible hardens system      | ☐     |
| Lynis ≥90%                  | ☐     |
| IOPS >10,000                | ☐     |
| Docs complete               | ☐     |

---

## Next Steps
- **Beta Phase (Weeks 2–4)**:  
  - Full React UI  
  - HA clustering  
  - Compliance testing (GDPR/SOC 2)  
  - User feedback loop

**MVP DELIVERED ON TIME** — **November 13, 2025**
