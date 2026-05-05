# 🚀 Complete Setup Guide (Clean & Improved)

## 🧹 Step 1 — Clean Previous Setup

Stop and remove existing containers:

```bash
docker compose down
```

Remove old environment file (if exists):

```bash
rm -f .env
```

Remove old external network (if exists):

```bash
docker network rm n8n-ai-net 2>/dev/null || true
```

Remove old volumes:

```bash
docker volume rm n8n-workflow n8n-workflow-db evolution-api evolution-redis evolution-postgres 2>/dev/null || true
```

Clean unused Docker resources:

```bash
docker system prune -a --volumes -f
```

> ⚠️ This will delete unused containers, images, and volumes permanently.

---

## 📄 Step 2 — Create / Update `docker-compose.yml`

Create a file named:

```bash
docker-compose.yml
```

Paste the following content:

```
services:

  # =========================
  # POSTGRES
  # =========================
  postgres-db:
    image: postgres:15
    container_name: postgres-db
    restart: unless-stopped
    environment:
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: adminpass
    volumes:
      - postgres-db:/var/lib/postgresql/data
      - ./init-db.sh:/docker-entrypoint-initdb.d/init-db.sh
    networks:
      - app-net

  # =========================
  # N8N
  # =========================
  n8n-workflow:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n-workflow
    restart: unless-stopped
    ports:
      - "5678:5678"
    depends_on:
      - postgres-db
    networks:
      - app-net
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres-db
      - DB_POSTGRESDB_PORT=5432
      - DB_POSTGRESDB_DATABASE=n8n
      - DB_POSTGRESDB_USER=admin
      - DB_POSTGRESDB_PASSWORD=adminpass
      - GENERIC_TIMEZONE=Asia/Kolkata
      - TZ=Asia/Kolkata
      - WEBHOOK_URL=http://n8n-workflow:5678
      - N8N_SECURE_COOKIE=false
    volumes:
      - n8n-workflow:/home/node/.n8n

  # =========================
  # EVOLUTION API
  # =========================
  evolution-api:
    image: evoapicloud/evolution-api:latest
    container_name: evolution-api
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - postgres-db
      - redis-cache
    networks:
      - app-net
    volumes:
      - evolution-api:/evolution/instances
    environment:
      - SERVER_TYPE=http
      - SERVER_PORT=8080
      - AUTHENTICATION_API_KEY=evo@12345
      - DATABASE_PROVIDER=postgresql
      - DATABASE_CONNECTION_URI=postgresql://admin:adminpass@postgres-db:5432/evolution
      - CACHE_REDIS_ENABLED=true
      - CACHE_REDIS_URI=redis://redis-cache:6379
      - WEBSOCKET_ENABLED=true
      - LOG_LEVEL=ERROR

  # =========================
  # REDIS
  # =========================
  redis-cache:
    image: redis:latest
    container_name: redis-cache
    restart: unless-stopped
    command: redis-server --appendonly yes
    volumes:
      - redis-cache:/data
    networks:
      - app-net

# =========================
# NETWORK
# =========================
networks:
  app-net:
    driver: bridge

# =========================
# VOLUMES
# =========================
volumes:
  postgres-db:
  n8n-workflow:
  evolution-api:
  redis-cache:
```

---

## 📄 Step 3 — Create `init-db.sh`

Create a file:

```bash
init-db.sh
```

Paste:

```
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" <<-EOSQL
    CREATE DATABASE n8n;
    CREATE DATABASE evolution;
EOSQL
```

Make it executable:

```bash
chmod +x init-db.sh
```

---

## ▶️ Step 4 — Start Services

```bash
docker compose up -d
```

---

## 🌐 Step 5 — Access Services

* n8n → [http://localhost:5678](http://localhost:5678)
* Evolution Manager → [http://localhost:8080/manager](http://localhost:8080/manager)

---

## 🔐 Step 6 — Login

Password:

```
Tirtha@2026
```

---

## 📲 Step 7 — Connect WhatsApp

1. Open Evolution Manager
2. Create new instance

   * Name: `smch`
   * Method: `baileys`
3. Go to **Settings → QR Code**
4. Scan using WhatsApp → Linked Devices

---

## ⚠️ Notes

* First startup may take **20–40 seconds**
* If QR doesn’t appear → refresh or wait
* If DB error occurs → restart stack:

```bash
docker compose restart
```

---

## ✅ Result

You now have:

* Shared PostgreSQL (2 databases auto-created)
* n8n automation system
* Evolution API (WhatsApp integration)
* Redis caching layer
* Fully Dockerized setup

# ⚡ Quick Links

| Service                  | URL                                                            |
| ------------------------ | -------------------------------------------------------------- |
| n8n UI                   | [http://localhost:5678](http://localhost:5678)                 |
| Evolution Manager        | [http://localhost:8080/manager](http://localhost:8080/manager) |
