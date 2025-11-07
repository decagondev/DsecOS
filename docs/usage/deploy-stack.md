# DsecOS Stack Templates

> **One-click application stacks** for rapid, secure deployment.  
> Each template includes runtime, database, and configuration — ready in under 2 minutes.

---

## Available Templates

| Stack | Runtime | Database | Use Case |
|-------|--------|---------|---------|
| `node-postgres` | Node.js 20.x | PostgreSQL 16 | Full-stack web apps |
| `java-mysql` | OpenJDK 21 | MySQL 8 | Enterprise Java services |
| `python-redis` | Python 3.12 | Redis 7 | AI/ML, caching, APIs |
| `ruby-sqlite` | Ruby 3.3 | SQLite 3 | Lightweight scripts, CMS |

---

## Deploy a Stack

```bash
# SSH into your DsecOS node
ssh root@your-dsecos-ip

# List available stacks
dsecos stack list

# Deploy a stack
dsecos deploy node-postgres
```

> Stack runs in isolated LXC container with dedicated SDN zone

---

## Template Structure

All templates located in: `/templates/stacks/`

### Example: `node-postgres.yml`

```yaml
version: '3.8'
services:
  app:
    image: node:20-alpine
    working_dir: /app
    volumes:
      - app_data:/app
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/myapp
    command: sh -c "npm install && npm start"
    depends_on:
      - db

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: securepass123
    volumes:
      - pg_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  app_data:
  pg_data:
```

---

## Customize Templates

1. **Edit template**:
   ```bash
   nano /templates/stacks/node-postgres.yml
   ```

2. **Change environment variables**, ports, or add services

3. **Redeploy**:
   ```bash
   dsecos deploy node-postgres --force
   ```

---

## Access Your App

| Stack | URL |
|------|-----|
| `node-postgres` | `http://your-ip:3000` |
| `java-mysql` | `http://your-ip:8080` |
| `python-redis` | `http://your-ip:5000` |
| `ruby-sqlite` | `http://your-ip:4567` |

> All traffic encrypted via SDN + nftables

---

## Monitor & Manage

- **Portainer**: `https://your-ip:9443/portainer`
- **Logs**:
  ```bash
  dsecos logs node-postgres
  ```
- **Stop/Remove**:
  ```bash
  dsecos stack rm node-postgres
  ```

---

## Create Custom Stack

```bash
# Create new template
cp /templates/stacks/node-postgres.yml /templates/stacks/myapp.yml

# Edit as needed
nano /templates/stacks/myapp.yml

# Deploy
dsecos deploy myapp
```

---

## Security Features (Per Stack)

| Feature | Enabled |
|-------|--------|
| SELinux isolation | `container_t` |
| AppArmor profile | `docker-default` |
| Firewall zone | Auto-isolated SDN |
| Rootless Docker | Yes |
| Auto-updates | Weekly |

---

**Next**: [Security & Hardening →](hardening.md)  
**Support**: `support@decadev.co.uk`
