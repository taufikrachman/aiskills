# Docker & Containerization

Production Docker patterns for Node.js/Next.js applications.

## Rules

### 1. Dockerfile Best Practices
```dockerfile
# Multi-stage build
FROM node:22-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

FROM node:22-alpine AS runner
RUN apk add --no-cache tini
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
EXPOSE 3000
USER node
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "dist/main.js"]
```
- Multi-stage: build in one stage, run in minimal stage.
- Use specific base tags (not `:latest`).
- Run as non-root user.
- Use `tini` as init process (handles signals, zombie reaping).

### 2. docker-compose.yml
```yaml
services:
  app:
    build: .
    ports: ['3000:3000']
    env_file: .env
    depends_on:
      db:
        condition: service_healthy
    healthcheck:
      test: ['CMD', 'wget', '--spider', 'http://localhost:3000/health']
      interval: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    volumes: ['pgdata:/var/lib/postgresql/data']
    environment:
      POSTGRES_DB: myapp
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U postgres']
      interval: 5s

volumes:
  pgdata:
```
- Always use healthchecks.
- Secrets via env_file, never hardcoded.
- Named volumes for persistent data.

### 3. Image Optimization
- `.dockerignore`: exclude `node_modules`, `.git`, `dist`, `*.md`, tests.
- Use `.dockerignore` aggressively — smaller context = faster builds.
- Layer caching: copy `package.json` first, then `npm ci`, then source.
- Scan for vulnerabilities: `docker scout` or `trivy`.
- Tag with git hash: `myapp:${GIT_SHA}`.

### 4. Production Checklist
- [ ] Non-root user
- [ ] Health check endpoint
- [ ] Read-only root filesystem (if possible)
- [ ] Resource limits: `deploy.resources.limits.memory: 512M`
- [ ] Logs to stdout/stderr (not files)
- [ ] Graceful shutdown on SIGTERM
