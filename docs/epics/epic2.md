# DsecOS Epic 2: Rebranding

This markdown file details Epic 2 from the DsecOS Development Tasks Document. The goal of this epic is to remove all Proxmox branding, apply a dark color scheme, and ensure 100% consistency across UI, CLI, and documentation. This is prioritized for MVP completion in Days 2-3 (November 7-8, 2025).

The structure follows the hierarchy: PRs > Commits > Tasks > Sub-Steps. Each sub-step is broken down into very simple, beginner-friendly actions (e.g., exact commands where possible, assuming a Linux/Debian-based development environment like Ubuntu). 

Prerequisites:
- Epic 1 is completed: Repo is set up with Proxmox submodules.
- You have a text editor (e.g., nano or VS Code) and terminal access.
- For logo creation: Access to a free online tool like Canva (browser-based) or install Inkscape (`sudo apt install inkscape`).
- For UI testing: A web browser and ability to build/run the Proxmox UI locally (e.g., via npm if ExtJS needs it).
- Node.js/npm installed if not already (`sudo apt install nodejs npm`).
- Git knowledge from Epic 1.

## PR 2.1: UI Rebranding
**Branch**: feature/ui-rebrand  
**Description**: Replace logos, text, and apply a dark theme to the web UI; evaluate a hybrid Vite/React upgrade for MVP.  
**Estimated Time**: 6-8 hours.  
**Merge Criteria**: UI builds and loads without Proxmox branding; dark theme applies consistently; basic hybrid test works.

### Commit 2.1.1: Replace Logos and Text
**Tasks**:
1. Navigate to the UI source directory.
   - Open your terminal.
   - Change to the repo root: `cd ~/projects/dsecos-private` (adjust path if needed).
   - Navigate to the Proxmox UI submodule: `cd src/pve-manager/www`.

2. Locate logo files.
   - Run: `ls | grep logo` or use `find . -name "*logo*"` to search for files like logo.png, logo.svg, or proxmox_logo.png.
   - Open a browser if needed to view the Proxmox UI docs (search "Proxmox web UI files" for reference).

3. Create a custom DsecOS logo.
   - Open a web browser.
   - Go to canva.com (sign up if needed, free tier).
   - Search for "logo template", create a simple one with "DsecOS" text, dark background (#121212), accent color (#00BFFF).
   - Export as PNG and SVG.
   - Save to your local machine, then copy to the UI dir: `cp ~/Downloads/dsecos-logo.png .` (replace with actual paths).

4. Replace existing logos.
   - For each found logo file: Run `mv original-logo.png original-logo.png.bak` to backup.
   - Then: `cp path/to/dsecos-logo.png original-logo.png` (match file names and formats).
   - If SVG: Do the same for .svg files.

5. Search and replace text strings.
   - Run: `grep -rl "Proxmox" .` to find files with "Proxmox".
   - For each file: `nano filename.js` (or .html, .css).
   - Replace "Proxmox" with "DsecOS" (use Ctrl+W to search, edit manually).
   - Alternatively, bulk: `grep -rl "Proxmox" . | xargs sed -i 's/Proxmox/DsecOS/g'` (be careful, test on one file first).

6. Test changes locally.
   - Build the UI: Follow Proxmox docs (e.g., `make` or `npm install && npm run build` if ExtJS setup).
   - Open in browser: Serve with `python -m http.server` and visit localhost:8000.

7. Stage and commit.
   - From UI dir: `cd ../../..` to root.
   - Run: `git add src/pve-manager/www`.
   - Run: `git commit -m "Replace logos and text branding"`.
   - Push to branch.

### Commit 2.1.2: Apply Dark Color Scheme
**Tasks**:
1. Locate CSS files for theming.
   - In terminal, from src/pve-manager/www: `find . -name "*.css"`.
   - Key files: Likely manager6/css/ext6-pve.css or similar (search "Proxmox ExtJS CSS" in browser for ref).

2. Backup original CSS.
   - For main CSS: `cp ext6-pve.css ext6-pve.css.bak` (adjust filename).

3. Edit CSS for dark theme.
   - Run: `nano ext6-pve.css`.
   - Search for background colors (e.g., Ctrl+W "background").
   - Replace light colors with dark: e.g., #FFFFFF to #121212 for backgrounds.
   - For text: Change dark text to light (e.g., #000000 to #FFFFFF).
   - Accents: Replace blues/greens with #00BFFF.
   - Common changes: Body background, panel headers, buttons.

4. Handle any JS-based styles.
   - If styles in JS: `grep -rl "color:" .` and edit similarly.

5. Test the theme.
   - Rebuild UI as in previous commit.
   - Load in browser, check for consistency (e.g., dashboard, login page).
   - Toggle browser dark mode if needed to verify.

6. Stage and commit.
   - From root: `git add src/pve-manager/www`.
   - Run: `git commit -m "Apply dark color scheme to UI"`.
   - Push.

### Commit 2.1.3: Evaluate Vite/React Upgrade (MVP Hybrid)
**Tasks**:
1. Assess ExtJS components.
   - In terminal: `cd src/pve-manager/www/manager6`.
   - Run: `ls` to list files; identify key ones like Dashboard.js, Login.js.

2. Set up Vite/React in a new directory.
   - From repo root: `mkdir ui/react-upgrade`.
   - Change dir: `cd ui/react-upgrade`.
   - Run: `npm init vite@latest . -- --template react` (answer prompts: project name "dsecos-ui", React with JS).
   - Install deps: `npm install`.

3. Create a hybrid component for MVP.
   - Run: `nano src/App.js`.
   - Add simple React component: e.g., a login form mimicking Proxmox.
   - To hybrid: Copy one ExtJS file (e.g., Login.js) to react dir, convert basics (e.g., replace Ext.create with React components).

4. Document effort estimate.
   - In commit message or a note file: Add "Full rewrite estimated 2-4 weeks; MVP hybrid patches one component."
   - Run: `nano NOTES.md` in ui/, add details.

5. Test hybrid setup.
   - Run: `npm run dev` to start Vite server.
   - Visit localhost:5173 in browser, check if component renders.

6. Stage and commit.
   - From root: `git add ui/react-upgrade`.
   - Run: `git commit -m "Hybrid Vite/React setup for UI upgrade evaluation"`.
   - Push.

**Merge PR 2.1**:
1. Create the branch if not: `git checkout -b feature/ui-rebrand`.
2. Push branch: `git push origin feature/ui-rebrand`.
3. On GitHub, create PR from feature/ui-rebrand to main.
4. Review: Build UI, visual check for branding/theme; run Vite test.
5. Merge.

## PR 2.2: CLI and Docs Rebranding
**Branch**: feature/cli-rebrand  
**Description**: Rename CLI tools and update documentation to remove Proxmox branding.  
**Estimated Time**: 4-6 hours.  
**Merge Criteria**: CLI commands show DsecOS branding; docs have no Proxmox references.

### Commit 2.2.1: Rename CLI Tools
**Tasks**:
1. Locate Proxmox CLI tools.
   - From repo root: `cd src/pve-manager`.
   - Run: `find . -name "pve*" -type f` (e.g., pveum, pct, qm).

2. Create wrapper scripts.
   - mkdir scripts/cli from root.
   - Run: `nano scripts/cli/dsecos`.
   - Add bash script: `#!/bin/bash\n# Wrapper for Proxmox tools\necho "DsecOS CLI"\n pct "$@"` (example for pct; expand for others).
   - Make executable: `chmod +x scripts/cli/dsecos`.

3. Replace strings in help texts.
   - For each CLI file: `nano bin/pct` (example).
   - Search/replace "Proxmox" with "DsecOS" in usage/help strings.
   - Bulk: `grep -rl "Proxmox" bin/ | xargs sed -i 's/Proxmox/DsecOS/g'`.

4. Test CLI.
   - Run: `./scripts/cli/dsecos --help` (should show DsecOS branding).
   - If installed, test in a VM.

5. Stage and commit.
   - From root: `git add scripts/cli src`.
   - Run: `git commit -m "Rebrand CLI tools to dsecos"`.
   - Push.

### Commit 2.2.2: Update Documentation
**Tasks**:
1. Copy Proxmox docs.
   - From src/pve-manager: `cp -r debian/doc ../../docs/proxmox-docs` (adjust if docs elsewhere).

2. Search and replace in docs.
   - From docs/: `grep -rl "Proxmox" . | xargs sed -i 's/Proxmox/DsecOS/g'`.
   - Manually edit key files: `nano index.md` (or HTML), verify changes.

3. Add PRD to docs.
   - Copy PRD file: `cp path/to/prd.md docs/prd.md`.
   - Edit if needed for branding.

4. Test docs rendering.
   - If Markdown: Use a viewer like grip (`pip install grip`, `grip docs/`).
   - Check in browser for no Proxmox text.

5. Stage and commit.
   - Run: `git add docs`.
   - Run: `git commit -m "Rebrand and update documentation"`.
   - Push.

**Merge PR 2.2**:
1. Create branch: `git checkout -b feature/cli-rebrand`.
2. Push: `git push origin feature/cli-rebrand`.
3. Create PR on GitHub.
4. Review: Run CLI help, browse docs files.
5. Merge to main.

## Next Steps After Epic 2
- Proceed to Epic 3 (Kernel Customization and Hardening).
- Daily check: Rebuild full system if possible, visual/functional tests in a VM.
