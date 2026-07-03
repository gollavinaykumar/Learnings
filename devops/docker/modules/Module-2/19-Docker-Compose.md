
# 🐳 Docker Mastery Roadmap
# Module 2 — Docker CLI & Everyday Docker
## Chapter 19 — Docker Compose

> **Learning Objectives**
>
> By the end of this chapter you will understand:
>
> - Why Docker Compose exists
> - compose.yaml structure
> - Services
> - Networks
> - Volumes
> - Environment variables
> - depends_on
> - Health checks
> - Restart policies
> - Profiles
> - Secrets & Configs
> - Scaling
> - Production Compose

---

# 📖 Why Docker Compose?

Modern applications contain multiple services.

Example:

```text
Frontend
Backend
PostgreSQL
Redis
Nginx
```

Running each container manually is difficult.

Docker Compose lets you define the entire application in one file.

---

# What is compose.yaml?

A YAML file describing your application.

```text
compose.yaml
        │
docker compose up
        │
        ▼
All Services Start
```

---

# Basic Structure

```yaml
services:
  app:
    image: nginx
```

Main sections:

- services
- networks
- volumes

---

# Services

Each service represents one container.

```yaml
services:
  backend:
    image: my-backend

  postgres:
    image: postgres:16
```

---

# Networks

```yaml
networks:
  backend-network:
```

Attach services:

```yaml
services:
  backend:
    networks:
      - backend-network
```

Compose automatically provides DNS.

---

# Volumes

Persist data.

```yaml
volumes:
  postgres-data:
```

Mount:

```yaml
services:
  postgres:
    volumes:
      - postgres-data:/var/lib/postgresql/data
```

---

# Environment Variables

```yaml
environment:
  NODE_ENV: production
  DB_HOST: postgres
```

Or:

```yaml
env_file:
  - .env
```

---

# depends_on

Start dependencies first.

```yaml
services:
  backend:
    depends_on:
      - postgres
      - redis
```

Note:

It controls startup order, not application readiness.

---

# Health Checks

```yaml
healthcheck:
  test: ["CMD","curl","-f","http://localhost:3000/health"]
  interval: 30s
```

Useful for orchestration and monitoring.

---

# Restart Policies

```yaml
restart: always
```

Options:

- no
- on-failure
- unless-stopped
- always

---

# Profiles

Run only selected services.

```yaml
profiles:
  - development
```

Start:

```bash
docker compose --profile development up
```

---

# Secrets

Store sensitive data securely.

```yaml
secrets:
  db_password:
    file: ./db_password.txt
```

---

# Configs

Reusable configuration files.

Useful for:

- nginx.conf
- application.properties

---

# Scaling

```bash
docker compose up --scale backend=3
```

Runs three backend containers.

---

# MERN Stack Example

```yaml
services:
  frontend:
    build: ./frontend

  backend:
    build: ./backend

  postgres:
    image: postgres:16

  redis:
    image: redis:8
```

---

# Spring Boot Example

```yaml
services:
  app:
    build: .

  postgres:
    image: postgres:16
```

---

# Production Best Practices

- Use named volumes.
- Use custom bridge networks.
- Pin image versions.
- Keep secrets outside source code.
- Add health checks.
- Use restart policies.

---

# Common Commands

Start:

```bash
docker compose up
```

Detached:

```bash
docker compose up -d
```

Stop:

```bash
docker compose down
```

Logs:

```bash
docker compose logs
```

Rebuild:

```bash
docker compose up --build
```

---

# Hands-on Lab

Create:

```yaml
services:
  nginx:
    image: nginx
    ports:
      - "8080:80"
```

Run:

```bash
docker compose up
```

Stop:

```bash
docker compose down
```

---

# Interview Questions

**Why use Docker Compose?**

To define and manage multi-container applications using a single YAML file.

**Difference between docker run and docker compose?**

`docker run` manages one container.

Docker Compose manages an entire application stack.

**Does depends_on wait for PostgreSQL to become ready?**

No. It only controls startup order. Use health checks for readiness.

---

# Summary

- Compose manages multi-container applications.
- Applications are described in compose.yaml.
- Compose creates networks automatically.
- Volumes provide persistence.
- Health checks improve reliability.
- Scaling and profiles simplify development.

---

# Next Chapter

## Chapter 20 — Docker Registries

You'll learn:

- Docker Hub
- Private Registries
- Image tagging
- Push & Pull
- Authentication
- Enterprise registry strategies
