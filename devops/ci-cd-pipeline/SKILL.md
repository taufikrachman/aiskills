# CI/CD Pipeline Builder

Automated build → test → deploy pipelines with GitHub Actions / GitLab CI.

## Rules

### 1. Pipeline Stages
```
Push → Lint → Test → Build → Deploy Staging → Deploy Production
```
- Lint & test on every push. Deploy only on main/tags.
- Fail fast: lint fails first → skip tests. Tests fail → skip deploy.

### 2. GitHub Actions
```yaml
name: CI/CD
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env: { POSTGRES_PASSWORD: test, POSTGRES_DB: test }
        options: --health-cmd pg_isready --health-interval 10s
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: 'npm' }
      - run: npm ci
      - run: npm run lint
      - run: npm test
        env: { DATABASE_URL: postgres://postgres:test@localhost:5432/test }

  build-and-push:
    needs: test
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build Docker image
        run: docker build -t registry.example.com/app:${{ github.sha }} .
      - name: Push to registry
        run: docker push registry.example.com/app:${{ github.sha }}

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.VPS_HOST }}
          username: deploy
          key: ${{ secrets.SSH_KEY }}
          script: |
            docker pull registry.example.com/app:${{ github.sha }}
            docker stop app || true
            docker run -d --name app -p 3000:3000 registry.example.com/app:${{ github.sha }}
```

### 3. Secrets & Environments
- NEVER hardcode secrets. Use `${{ secrets.NAME }}`.
- Use environments (staging/production) for different configs.
- Production deployment requires manual approval (environment protection rule).

### 4. Cache Strategy
- `actions/setup-node` with `cache: 'npm'` — caches `node_modules`.
- Docker layer caching: `docker/build-push-action` with cache registry.
- Don't cache `dist/` or build artifacts — rebuild on every deploy.

## Anti-Patterns
- ❌ Skipping tests on main branch pushes
- ❌ Deploying without health check verification
- ❌ Hardcoded secrets in workflow files
- ❌ Single job doing everything (linter + test + deploy)
