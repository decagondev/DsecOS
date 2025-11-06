# DsecOS Diagrams – Architecture, Flows & Processes

> **Comprehensive visual reference** for DsecOS using **Mermaid.js** syntax.  
> Render in VS Code, Obsidian, MkDocs, or [Mermaid Live Editor](https://mermaid.live).

---

## 1. System Architecture Overview

```mermaid
graph TD
    subgraph "Hardware & Provisioning"
        HW[Bare Metal Servers] -->|PXE| PXE[TFTP + pxelinux]
        PXE --> ISO["DsecOS ISO<br/>(Custom Kernel)"]
    end

    subgraph "DsecOS Core Node"
        K[Custom Linux Kernel<br/>KVM + LXC + SELinux] --> VM[KVM/QEMU VMs]
        K --> CT["LXC Containers<br/>(OpenVZ-style)"]
        K --> D[Docker Engine]
        D --> PORT[Portainer CE]
        K --> ZFS[ZFS Storage Pool]
        ZFS --> LUKS[LUKS Encryption]
    end

    subgraph "High Availability (Future)"
        CEPH[Ceph Cluster<br/>MON + OSD] <--> CEPH
        HA[Corosync + Pacemaker] <--> HA
    end

    subgraph "Management & Control"
        UI[React + Vite UI<br/>Dark Theme] --> API[FastAPI Backend<br/>JWT Auth]
        API --> PROX[Proxmox API Wrapper]
        PROX --> K
        UI --> PORT
        API --> LIC["License Server<br/>(Enterprise)"]
    end

    subgraph "Observability"
        PROM[Prometheus] --> EXP[Node Exporter]
        GRAF[Grafana] --> PROM
        AI["AI Anomaly Detector<br/>(Isolation Forest)"] --> PROM
        AI --> API
    end

    style K fill:#121212,stroke:#00BFFF,color:#FFF
    style UI fill:#1E1E1E,stroke:#00BFFF,color:#FFF
    style LIC fill:#8B0000,color:#FFF
```

---

## 2. User Onboarding & Deployment Flow

```mermaid
journey
    title DsecOS User Journey – From Zero to Running App
    section Provisioning
      Power on server: 5: Admin
      PXE boot from network: 5: BIOS, TFTP Server
      Select DsecOS installer: 4: Admin
    section Installation
      Auto-partition + LUKS: 5: Installer
      Install custom kernel: 5: System
      First boot + license check: 4: Boot Script, License Server
    section Access
      Open browser → https://ip:9443: 5: Admin
      Login (JWT): 5: React UI, FastAPI
      Dashboard loads: 4: React UI
    section Deploy
      Click "Stacks" → Choose "Node + Postgres": 5: Admin
      Review resources → Deploy: 4: UI, Proxmox API
      Container starts: 5: LXC + Docker
      App live on port 3000: 4: SDN Firewall
    section Monitor
      View logs in Portainer: 3: Admin
      Check AI alerts in Grafana: 3: Grafana
```

---

## 3. Stack Deployment Process Flow

```mermaid
flowchart TD
    A[User Selects Stack Template] --> B{License Valid?}
    B -->|No| Z[Block Deployment<br/>Show License Error]
    B -->|Yes| C[Parse docker-compose.yml]
    C --> D[Call Proxmox API<br/>→ Create LXC Container]
    D --> E[Bind Mount Ceph/ZFS Volume]
    E --> F[Start Docker Daemon]
    F --> G[docker-compose up -d]
    G --> H[Configure SDN Firewall Rules]
    H --> I[Register in Portainer]
    I --> J[Update UI Dashboard]
    J --> K[Send Success Toast]

    style B fill:#8B0000,color:#FFF
    style K fill:#006400,color:#FFF
    style Z fill:#DC143C,color:#FFF
```

---

## 4. Boot & Security Hardening Sequence

```mermaid
flowchart LR
    A[Power On] --> B[GRUB → Custom Kernel]
    B --> C[SELinux: Enforcing Mode]
    C --> D[nftables: Load Rules<br/>Default DROP]
    D --> E[Mount LUKS Root]
    E --> F[Start Docker + AppArmor]
    F --> G[Start OSSEC HIDS]
    G --> H[Run Lynis Audit → /var/log/lynis]
    H --> I[Start Prometheus Scraping]
    I --> J[AI Model Loads]
    J --> K[System Ready]

    style C fill:#121212,stroke:#00BFFF,color:#FFF
    style K fill:#228B22,color:#FFF
```

---

## 5. License Validation Sequence

```mermaid
sequenceDiagram
    participant U as Admin (Browser)
    participant UI as React Frontend
    participant API as FastAPI
    participant LS as License Server
    participant OS as DsecOS Host

    Note over OS,LS: First Boot
    OS->>LS: POST /validate {hw_id, version}
    LS-->>OS: 200 + JWT (valid) or 403
    alt Valid
        OS->>OS: Continue boot
    else Invalid
        OS->>OS: Lock UI, show license screen
    end

    U->>UI: Open https://ip:9443
    UI->>API: POST /login {user, pass}
    API->>API: Check local auth
    API->>LS: POST /token {claims}
    LS-->>API: Signed JWT
    API-->>UI: Set HttpOnly Cookie
    UI->>API: GET /dashboard
    API->>API: Verify JWT
    API-->>UI: Serve Dashboard
```

---

## 6. AI Monitoring & Alert Pipeline

```mermaid
graph LR
    A["Node Exporter<br/>(CPU, Disk, Net)"] --> B[Prometheus<br/>Time-Series DB]
    B --> C[collect_metrics.py<br/>→ CSV Dataset]
    C --> D[Retrained Weekly<br/>Isolation Forest Model]
    D --> E[detect.py<br/>Real-time Scoring]
    E -->|Anomaly| F[Alert → FastAPI]
    F --> G[Grafana Alert Panel]
    F --> H[Email / Slack Webhook]
    E -->|Normal| I[Continue]

    style D fill:#1E1E1E,stroke:#00BFFF,color:#FFF
    style F fill:#B22222,color:#FFF
```

---

## 7. Development & Release Pipeline

```mermaid
gitGraph
    commit id: "init"
    branch epic/1-setup
    checkout epic/1-setup
    commit id: "Repo + CI"
    commit id: "Fork Proxmox"
    checkout main
    merge epic/1-setup

    branch epic/2-rebrand
    checkout epic/2-rebrand
    commit id: "Dark UI"
    commit id: "CLI Wrapper"
    checkout main
    merge epic/2-rebrand

    branch epic/3-kernel
    checkout epic/3-kernel
    commit id: "KVM + SELinux"
    commit id: "Hardening"
    checkout main
    merge epic/3-kernel tag: "MVP v0.1"

    branch epic/6-react
    checkout epic/6-react
    commit id: "Vite + React"
    checkout main
    merge epic/6-react tag: "Beta v0.2"

    branch epic/7-ha
    checkout epic/7-ha
    commit id: "Ceph + Pacemaker"
    commit id: "AI Monitoring"
    commit id: "Enterprise License"
    checkout main
    merge epic/7-ha tag: "GA v1.0"
```

---

## 8. Data Flow – Stack Template to Running App

```mermaid
flowchart TD
    T[templates/stacks/node-postgres.yml] --> CLI[dsecos deploy node-postgres]
    CLI --> API[FastAPI → /deploy]
    API --> PROX[Proxmox API<br/>POST /nodes/localhost/lxc]
    PROX --> LXC[Create LXC Container]
    LXC --> DOCK[Start Docker]
    DOCK --> COMP[docker-compose up]
    COMP --> APP[Node App Running]
    APP --> PORT[Port 3000 Exposed]
    PORT --> UI[Visible in Portainer + React UI]

    style T fill:#2F4F4F,color:#FFF
    style APP fill:#228B22,color:#FFF
```

## 9. Kernel Boot & Module Load Sequence

```mermaid
flowchart TD
    A[BIOS/UEFI] --> B[GRUB2 Bootloader]
    B --> C["Load vmlinuz<br/>(Custom Kernel 6.x)"]
    C --> D[Initramfs<br/>→ Load Modules]
    D --> E[modprobe kvm]
    E --> F[modprobe vhost-net]
    F --> G[modprobe lxc]
    G --> H[modprobe overlay]
    H --> I[SELinux: Load Policy<br/>enforcing]
    I --> J[Execute /etc/rc.local<br/>→ firewall-setup.sh]
    J --> K[Start systemd<br/>→ docker, portainer, ossec]
    K --> L[System Ready]

    style C fill:#121212,stroke:#00BFFF,color:#FFF
    style I fill:#8B0000,color:#FFF
    style L fill:#228B22,color:#FFF
```

---

## 10. VM/Container Lifecycle Management

```mermaid
stateDiagram-v2
    [*] --> Creating
    Creating --> Starting: qm start / pct start
    Starting --> Running
    Running --> Stopping: qm stop / pct stop
    Stopping --> Stopped
    Stopped --> Starting
    Running --> Migrating: Live Migration
    Migrating --> Running
    Stopped --> Destroying: qm destroy / pct destroy
    Destroying --> [*]

    state Running {
        [*] --> Active
        Active --> Suspended: qm suspend
        Suspended --> Active: qm resume
    }
```

---

## 11. Zero-Trust Network Policy Enforcement

```mermaid
flowchart LR
    A[Container/VM Start] --> B[Generate SDN Zone]
    B --> C[Assign VLAN ID]
    C --> D[Apply nftables Rules<br/>Default: DROP]
    D --> E{Allowed Ports?}
    E -->|Yes| F[Open Port via API]
    F --> G[Update Firewall Chain]
    E -->|No| H[Log Attempt → OSSEC]
    H --> I[AI Anomaly Detector]
    I --> J[Block + Alert]

    style D fill:#121212,stroke:#00BFFF,color:#FFF
    style J fill:#B22222,color:#FFF
```

---

## 12. Backup & Snapshot Workflow

```mermaid
sequenceDiagram
    participant U as Admin UI
    participant API as FastAPI
    participant P as Proxmox
    participant S as Storage (ZFS/Ceph)

    U->>API: POST /backup {vmid, schedule}
    API->>P: Schedule Snapshot (zfs send / vzdump)
    P->>S: Create Incremental Snapshot
    S-->>P: Snapshot ID
    P-->>API: 200 OK + ID
    API-->>U: "Backup Scheduled"

    Note over P,S: Nightly @ 02:00<br/>Retention: 7 days
    alt Retention Exceeded
        P->>S: Delete Old Snap
    end
```

---

## 13. Fail2Ban + OSSEC Integration

```mermaid
graph TD
    A[SSH Login Attempt] --> B[auth.log]
    B --> C[Fail2Ban<br/>Jail: sshd]
    C -->|5 fails| D[iptables-nft BAN IP]
    D --> E[Log to OSSEC]
    E --> F[OSSEC Active Response<br/>→ Permanent Block]
    F --> G[Alert → Grafana]

    style C fill:#1E1E1E,stroke:#00BFFF,color:#FFF
    style F fill:#8B0000,color:#FFF
```

---

## 14. Template Rendering & Deployment Pipeline

```mermaid
flowchart TD
    T[templates/stacks/*.yml] --> R["Jinja2 Render<br/>{% if env == 'prod' %}"]
    R --> C[Customized docker-compose.yml]
    C --> V[Validate Schema]
    V --> D[Deploy via dsecos CLI]
    D --> LXC[Spawn LXC]
    LXC --> DOCK[docker-compose up]
    DOCK --> APP[App Running]

    style R fill:#2F4F4F,color:#FFF
    style APP fill:#228B22,color:#FFF
```

---

## 15. PXE + Kickstart Automated Install

```mermaid
sequenceDiagram
    participant HW as Bare Metal
    participant DHCP as DHCP Server
    participant TFTP as TFTP Server
    participant HTTP as HTTP (Kickstart)

    HW->>DHCP: Discover
    DHCP-->>HW: Offer (IP + PXE)
    HW->>TFTP: Request pxelinux.0
    TFTP-->>HW: Send kernel + initrd
    HW->>HTTP: GET ks.cfg
    HTTP-->>HW: Auto-install config
    HW->>HW: Partition + Install DsecOS
    HW->>HW: Reboot → Ready

    Note over HW,HTTP: ks.cfg includes:<br/>LUKS, SELinux, Docker
```

---

## 16. CI/CD Pipeline with Security Gates

```mermaid
gitGraph
    commit id: "Code"
    branch feature/new-stack
    checkout feature/new-stack
    commit id: "Add Ruby Template"
    commit id: "Run Tests"
    commit id: "Lynis Scan"
    checkout main
    merge feature/new-stack
    commit tag: "v0.3.1"

    branch release/v1.0
    checkout release/v1.0
    commit id: "Bump Version"
    commit id: "Build ISO"
    commit id: "Sign ISO"
    checkout main
    merge release/v1.0
    commit tag: "v1.0.0-ga"
```

---

## 17. Multi-Tenant Isolation Model

```mermaid
graph TD
    subgraph Tenant_A
        VA[VM A1] --> LA[LXC A1]
        LA --> DA[Docker A1]
    end
    subgraph Tenant_B
        VB[VM B1] --> LB[LXC B1]
        LB --> DB[Docker B1]
    end

    VA --> NET_A[SDN Zone A]
    VB --> NET_B[SDN Zone B]
    NET_A -.->|No Route| NET_B

    LA --> SEL_A[SELinux: tenant_a_t]
    LB --> SEL_B[SELinux: tenant_b_t]

    style NET_A fill:#00BFFF,color:#000
    style NET_B fill:#DC143C,color:#FFF
```

---

## 18. Disaster Recovery Flow

```mermaid
flowchart TD
    A[Node Failure Detected] --> B[Pacemaker Fencing]
    B --> C[Restart Services on Peer]
    C --> D[Promote Replica OSD]
    D --> E[Update DNS VIP]
    E --> F[Notify Admin → Grafana]
    F --> G[Auto-Recovery Complete]

    style A fill:#B22222,color:#FFF
    style G fill:#228B22,color:#FFF
```

---

**DsecOS – Engineered for Security, Automation, and Resilience.**  
*From boot to production in under 15 minutes.*

## Rendering Instructions

- **VS Code**: Install **"Markdown Preview Mermaid Support"**
- **MkDocs**: Use `markdown_extensions: [pymdownx.superfences, pymdownx.inlinehilite, mermaid]`
- **Online**: [mermaid.live](https://mermaid.live)

---

**DsecOS – Secure. Fast. Enterprise-Ready.**  
*Built for isolation, scaled for production.*
