# Deployment Environments

> Defines the system's environments, their purposes, access, and key configurations.
> **Rule:** If you need to access production directly, something is wrong in the process.
> Most problems should be detected in staging.

---

## Project environments

| Environment | Purpose | Update | Access | Data |
|-------------|---------|--------|--------|------|
| **local** | Individual development (`docker compose`) | Manual | Individual dev | Synthetic / seed |
| **dev** | Continuous integration (`dev` branch) | Automatic (PR merge to `dev`) | Entire team | Synthetic (Testcontainers) |
| **staging** | Pre-production, QA, pilot demos (`main`) | Manual / release candidate | Team + Instructor / PO | Anonymized |
| **production** | Post-formative live residential complex | Manual (double approval) | Tech Lead + DevOps only | Real (Ley 1581 protected) |

---

## Environment variables per environment

> Specific variables go in the environment's secrets system (Vault, AWS Secrets Manager, Kubernetes Secrets).
> Local values go in `.env.example` (never real credentials).

### Naming convention

```
Standardized 12-Factor environment variables per service (`05-architecture/deployment.md`):
  SERVER_PORT
  DB_URL
  DB_USERNAME
  DB_PASSWORD
  JWT_SECRET
  RABBITMQ_HOST / RABBITMQ_PORT
```

### Variables per environment (synthetic local examples — production injected via Vault)

| Variable | Local | Dev (CI) | Staging | Production |
|----------|-------|----------|---------|------------|
| `DB_URL` | `jdbc:mysql://localhost:3306/{service}_db?useSSL=false&serverTimezone=UTC` | Testcontainers MySQL 8 | Vault | Vault |
| `RABBITMQ_HOST` | `localhost` (`:5672`) | Testcontainers RabbitMQ | Vault | Vault |
| `JWT_SECRET` | Local ≥ 256-bit dev key | CI secret | Vault | Vault |
| `LOG_LEVEL` | `DEBUG` | `INFO` | `INFO` | `WARN` |
| `SPRING_PROFILES_ACTIVE` | `local` | `ci` | `staging` | `prod` |
| `CORS_ALLOWED_ORIGINS` | `http://localhost:3000,http://127.0.0.1:5500` | `https://dev.resicomplex.com` | `https://staging.resicomplex.com` | `https://app.resicomplex.com` |

---

## CI/CD Pipeline

```
Developer
  │
  │ git push feature/HU-XXX
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  PR PIPELINE (runs on every push to a branch)           │
│  ─────────────────────────────────────────────          │
│  1. Lint + Format check      (< 2 min)                  │
│  2. Unit Tests               (< 5 min)                  │
│  3. Build                    (< 3 min)                  │
│  ─────────────────────────────────────────────          │
│  If it fails: blocks the PR merge                       │
└─────────────────────────────────────────────────────────┘
  │
  │ PR approved and merged to `develop`
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  DEVELOP PIPELINE (deploy to dev)                       │
│  ─────────────────────────────────────────────          │
│  1. Lint + Build + Unit Tests   (< 10 min)              │
│  2. Integration Tests           (< 15 min)              │
│  3. Build Docker image          (< 5 min)               │
│  4. Deploy to dev               (< 5 min)               │
│  5. Smoke tests in dev          (< 3 min)               │
└─────────────────────────────────────────────────────────┘
  │
  │ Release: merge develop → main
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  MAIN PIPELINE (deploy to staging)                      │
│  ─────────────────────────────────────────────          │
│  1. All tests                   (< 20 min)              │
│  2. Contract Tests              (< 5 min)               │
│  3. Build + push image with tag (< 5 min)               │
│  4. Deploy to staging           (< 5 min)               │
│  5. Smoke tests in staging      (< 5 min)               │
│  6. Performance tests (k6)      (< 15 min)              │
│  7. Notify PO (ready for QA)                            │
└─────────────────────────────────────────────────────────┘
  │
  │ Manual approval by Tech Lead
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  DEPLOY TO PRODUCTION                                   │
│  ─────────────────────────────────────────────          │
│  1. Canary deploy (10% of traffic)                      │
│  2. Monitor metrics for 15 min                          │
│  3. If OK: 100% of traffic                              │
│  4. If KO: automatic rollback                           │
└─────────────────────────────────────────────────────────┘
```

---

## Production deploy strategy

### Available options

| Strategy | Description | Risk | Rollback |
|----------|-------------|------|---------|
| **Canary** | 10% → 50% → 100% with validation at each step | Low | Automatic |
| **Blue-Green** | Instant switch between two versions | Medium | Immediate |
| **Rolling** | Replaces instances one by one | Low | Slow |
| **Recreate** | Takes everything down and starts the new version | High | Manual |

**Adopted strategy:** [Canary / Blue-Green / Rolling]

### Production deploy checklist

- [ ] The release PR was reviewed and approved
- [ ] Staging tests passed (including performance)
- [ ] The PO approved the HUs included in this release
- [ ] The Tech Lead approved the deploy
- [ ] Someone is available to monitor the first 30 min post-deploy
- [ ] If there are database changes: migration ran before the code deploy
- [ ] The rollback runbook is available: `[link to runbook]`

### Rollback

```bash
# Rollback Kubernetes to the previous version
kubectl rollout undo deployment/[service-name] -n [namespace]

# Verify the rollback
kubectl rollout status deployment/[service-name] -n [namespace]

# If there is an incompatible DB migration (EMERGENCY — consult Tech Lead first)
npm run db:migrate:revert
```

---

## Deploy log

| Date | Service | Version | Environment | Deployed by | Issues |
|------|---------|---------|-------------|-------------|--------|
| [date] | [service] | v[X.Y.Z] | production | [name] | None |

---

## Correlations

- Local setup → `10-devops/local-setup.md`
- Post-deploy observability → `13-operations/observability.md`
- Runbook per service → `09-microservices/services/XX/runbook.md`
- Branch policy → `00-governance/git-conventions.md`
