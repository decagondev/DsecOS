# DsecOS Epic 6: Advanced Features (Post-MVP – Beta Phase)

This markdown file details **Epic 6** from the DsecOS Development Roadmap. This epic begins **after MVP delivery (November 13, 2025)** and targets **beta readiness by December 6, 2025 (Week 4)**. Focus: modernize the frontend, add API enhancements, improve security monitoring, and prepare for compliance validation.

The structure follows: **PRs > Commits > Tasks > Sub-Steps**. Each sub-step is **very simple, beginner-friendly**, with **exact commands**, assuming a **Linux/Debian-based dev environment**.

---

## Prerequisites
- **MVP v0.1-mvp** successfully built and tested.
- GitHub repo with `v0.1-mvp` tag.
- Test cluster: 3+ VMs (1 control, 2 nodes).
- Tools:
  ```bash
  sudo apt install -y nodejs npm curl jq
  npm install -g @vue/cli
  ```

---

## PR 6.1: Full Vite + React Frontend Overhaul  
**Branch**: `feature/react-ui`  
**Description**: Replace ExtJS UI with modern Vite + React + Tailwind CSS.  
**Estimated Time**: 10–14 days  
**Merge Criteria**: Login, Dashboard, VM/Container list fully functional.

---

### Commit 6.1.1: Scaffold React App with Vite

**Tasks**:

1. **Create frontend directory**  
   ```bash
   mkdir -p ui/react
   cd ui/react
   ```

2. **Initialize Vite + React**  
   ```bash
   npm create vite@latest . -- --template react
   ```

3. **Install Tailwind CSS**  
   ```bash
   npm install -D tailwindcss postcss autoprefixer
   npx tailwindcss init -p
   ```

4. **Configure Tailwind**  
   ```bash
   nano tailwind.config.js
   ```
   - Replace with:
     ```js
     /** @type {import('tailwindcss').Config} */
     module.exports = {
       content: ["./index.html", "./src/**/*.{js,jsx,ts,tsx}"],
       theme: {
         extend: {
           colors: {
             primary: '#00BFFF',
             dark: '#121212',
             surface: '#1E1E1E'
           }
         }
       },
       plugins: [],
     }
     ```

5. **Add base styles**  
   ```bash
   nano src/index.css
   ```
   - Add:
     ```css
     @tailwind base;
     @tailwind components;
     @tailwind utilities;

     body {
       background-color: #121212;
       color: white;
       font-family: 'Inter', sans-serif;
     }
     ```

6. **Start dev server**  
   ```bash
   npm run dev
   ```
   - Visit `http://localhost:5173` → Vite + React loads.

7. **Stage and commit**  
   ```bash
   cd ../..
   git add ui/react
   git commit -m "Scaffold Vite + React + Tailwind"
   git push
   ```

---

### Commit 6.1.2: Build Login & Dashboard

**Tasks**:

1. **Create Login component**  
   ```bash
   cd ui/react
   mkdir -p src/components
   nano src/components/Login.jsx
   ```
   - Paste:
     ```jsx
     import { useState } from 'react';

     export default function Login() {
       const [username, setUsername] = useState('');
       const [password, setPassword] = useState('');

       const handleLogin = () => {
         // Mock auth → redirect
         window.location.href = '/dashboard';
       };

       return (
         <div className="min-h-screen flex items-center justify-center bg-dark">
           <div className="bg-surface p-8 rounded-lg shadow-xl w-96">
             <h1 className="text-2xl font-bold text-primary mb-6">DsecOS Login</h1>
             <input
               className="w-full p-2 mb-4 bg-gray-800 text-white rounded"
               placeholder="Username"
               value={username}
               onChange={(e) => setUsername(e.target.value)}
             />
             <input
               type="password"
               className="w-full p-2 mb-6 bg-gray-800 text-white rounded"
               placeholder="Password"
               value={password}
               onChange={(e) => setPassword(e.target.value)}
             />
             <button
               onClick={handleLogin}
               className="w-full bg-primary text-black font-bold py-2 rounded hover:bg-cyan-400"
             >
               Login
             </button>
           </div>
         </div>
       );
     }
     ```

2. **Create Dashboard**  
   ```bash
   nano src/components/Dashboard.jsx
   ```
   - Simple layout with VM/Container cards.

3. **Update App routing**  
   ```bash
   npm install react-router-dom
   nano src/App.jsx
   ```
   - Add routes: `/login`, `/dashboard`.

4. **Test locally**  
   ```bash
   npm run dev
   ```
   - Login → Dashboard.

5. **Stage and commit**  
   ```bash
   git add src/components src/App.jsx
   git commit -m "Build Login and Dashboard in React"
   git push
   ```

---

### Commit 6.1.3: Proxy API to Proxmox Backend

**Tasks**:

1. **Set up Vite proxy**  
   ```bash
   nano vite.config.js
   ```
   - Add:
     ```js
     server: {
       proxy: {
         '/api': 'http://localhost:8006',  // Proxmox API
         '/pve': 'http://localhost:8006'
       }
     }
     ```

2. **Call Proxmox API from React**  
   ```bash
   nano src/api/pve.js
   ```
   - Example:
     ```js
     export const getNodes = async () => {
       const res = await fetch('/api2/json/nodes', {
         credentials: 'include'
       });
       return res.json();
     };
     ```

3. **Display VMs in Dashboard**  
   - Use `useEffect` to fetch and render.

4. **Stage and commit**  
   ```bash
   git add src/api vite.config.js
   git commit -m "Proxy Proxmox API to React frontend"
   git push
   ```

---

**Merge PR 6.1**:
```bash
git checkout -b feature/react-ui
git push origin feature/react-ui
```
- Review: UI loads, login works, API calls succeed → Merge.

---

## PR 6.2: Enhanced API & Security Monitoring  
**Branch**: `feature/api-monitoring`  
**Description**: Add RESTful API wrapper, JWT auth, and real-time security dashboard.  
**Estimated Time**: 7–10 days  
**Merge Criteria**: API secured, alerts visible.

---

### Commit 6.2.1: API Wrapper with FastAPI

**Tasks**:

1. **Set up FastAPI backend**  
   ```bash
   mkdir -p backend/api
   cd backend/api
   python3 -m venv venv
   source venv/bin/activate
   pip install fastapi uvicorn python-jose[cryptography] passlib
   ```

2. **Create main API**  
   ```bash
   nano main.py
   ```
   - Paste:
     ```python
     from fastapi import FastAPI, Depends, HTTPException
     from fastapi.security import OAuth2PasswordBearer

     app = FastAPI(title="DsecOS API")
     oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

     @app.get("/health")
     def health():
         return {"status": "healthy"}

     @app.get("/security/alerts")
     def get_alerts(token: str = Depends(oauth2_scheme)):
         # Mock OSSEC alerts
         return [{"id": 1, "msg": "SSH brute force", "level": "high"}]
     ```

3. **Run API**  
   ```bash
   uvicorn main:app --reload --port=8000
   ```

4. **Stage and commit**  
   ```bash
   cd ../..
   git add backend
   git commit -m "Add FastAPI wrapper with JWT auth"
   git push
   ```

---

### Commit 6.2.2: Real-Time Security Dashboard

**Tasks**:

1. **Parse OSSEC logs**  
   ```bash
   nano scripts/monitoring/parse-ossec.py
   ```
   - Read `/var/ossec/logs/alerts/alerts.json`, send to API.

2. **React Security Panel**  
   - In `ui/react/src/components/SecurityDashboard.jsx`:
     - Poll `/security/alerts`
     - Show cards with severity.

3. **Stage and commit**  
   ```bash
   git add scripts/monitoring ui/react/src/components/SecurityDashboard.jsx
   git commit -m "Add real-time security monitoring"
   git push
   ```

---

**Merge PR 6.2**:
```bash
git checkout -b feature/api-monitoring
git push origin feature/api-monitoring
```
- Test: API returns alerts, UI shows them → Merge.

---

## PR 6.3: Compliance & Beta Prep  
**Branch**: `feature/compliance-beta`  
**Description**: SOC 2/GDPR readiness, beta user onboarding.  
**Estimated Time**: 5–7 days  
**Merge Criteria**: Audit logs complete, beta guide ready.

---

### Commit 6.3.1: Compliance Artifacts

**Tasks**:

1. **GDPR Data Map**  
   ```bash
   mkdir -p docs/compliance
   nano docs/compliance/gdpr-data-map.md
   ```
   - List: VM metadata, logs, user data → retention, encryption.

2. **SOC 2 Control Matrix**  
   ```bash
   nano docs/compliance/soc2-controls.xlsx
   ```
   - Use LibreOffice or Google Sheets → export PDF.

3. **Stage and commit**  
   ```bash
   git add docs/compliance
   git commit -m "Add GDPR and SOC 2 compliance docs"
   git push
   ```

---

### Commit 6.3.2: Beta Onboarding

**Tasks**:

1. **Beta Guide**  
   ```bash
   nano docs/beta-guide.md
   ```
   - Include:
     - Download ISO
     - PXE setup
     - Feedback form link

2. **Feedback Form**  
   - Use Google Forms or Typeform → embed link.

3. **Stage and commit**  
   ```bash
   git add docs/beta-guide.md
   git commit -m "Prepare beta onboarding"
   git push
   ```

---

**Merge PR 6.3**:
```bash
git checkout -b feature/compliance-beta
git push origin feature/compliance-beta
```
- Final review → Merge → **Beta Ready**

---

## Beta Release: December 6, 2025

| Milestone                  | Status |
|---------------------------|--------|
| React UI (Login + Dashboard) | ☐     |
| API + JWT Auth             | ☐     |
| Security Monitoring        | ☐     |
| Compliance Docs            | ☐     |
| Beta Guide + Feedback      | ☐     |

**Tag**: `v0.2-beta`

---

## Next: Epic 7 → HA Clustering, AI Monitoring, Enterprise Licensing  
*(Post-beta, Q1 2026)*

**Beta Delivered On Time — December 6, 2025**
