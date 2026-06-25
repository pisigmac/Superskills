# DEPLOY.md — Production-Grade Deployment Protocol v3.0

Deploying is not "pushing code." Deploying is **promoting a verified artifact to production with zero downtime, full observability, and instant reversibility.**

---

## Philosophy

**Production is not a test environment. It is the only environment that matters.**
Staging proves the deploy is safe. Production proves the deploy is correct.

**Immutable infrastructure. Reversible by design. Observable by default.**
If you can't explain what changed, don't deploy. If you can't undo it in 2 minutes, don't deploy.

---

## Deployment Types & Gates

| Type | Frequency | Risk Level | Approval Required | Rollback SLA |
|------|-----------|------------|-------------------|--------------|
| **Hotfix** | Emergency only | Critical | On-call lead | <2 minutes |
| **Security Patch** | As needed | Critical | Security lead + On-call | <2 minutes |
| **Feature Release** | Weekly (batched) | Medium | Product + Engineering lead | <2 minutes |
| **Infrastructure Change** | Monthly | High | Architecture review | <5 minutes |
| **Database Migration** | Bi-weekly max | High | DBA + Engineering lead | <5 minutes |
| **Configuration Change** | As needed | Low | Engineering lead | <1 minute |

**Rule:** Any deploy without a tested rollback plan is a scheduled outage.

---

## The Immutable Deploy Pipeline

### Stage 0: Artifact Build (Reproducible)
```
1. Lock dependencies (lockfile hash verification)
2. Build in clean container (no host system leakage)
3. Run full test suite (unit + integration + security scan)
4. Build container image with deterministic layers
5. Tag: <commit-sha>-<timestamp>-<build-number>
6. Sign image with cosign / Notary
7. Push to immutable registry (no overwrites allowed)
8. Generate SBOM (Software Bill of Materials)
```

**Gate:** Build fails = no artifact. No artifact = no deploy.

### Stage 1: Staging Validation (Production-Equivalent)
```
1. Deploy artifact to staging (same infra spec as production)
2. Apply database migrations (if any) — dry-run first
3. Run E2E smoke tests (100% critical paths)
4. Run performance baseline (P95 < budget)
5. Run security DAST scan (OWASP ZAP)
6. Run chaos test (kill 1 pod, verify recovery)
7. Manual spot-check for user-facing changes (5 minutes)
```

**Gate:** Any failure = artifact rejected. Debug, fix, rebuild, restart pipeline.

### Stage 2: Production Deploy (Blue-Green or Canary)

#### Option A: Blue-Green (Recommended for Stateful Services)
```
1. Green environment receives new artifact
2. Health checks pass on Green (deep: DB connectivity, cache, queues)
3. Run 3-minute smoke test on Green
4. Switch load balancer from Blue → Green
5. Keep Blue running for 15 minutes (instant rollback)
6. After 15 minutes of healthy traffic, terminate Blue
```

**Rollback:** Switch load balancer back to Blue. <30 seconds.

#### Option B: Canary (Recommended for Stateless / High-Traffic)
```
1. Deploy new artifact to 5% of production pods
2. Monitor for 10 minutes:
   - Error rate < baseline + 0.1%
   - P95 latency < 1.2x baseline
   - Custom business metrics (orders, signups) normal
3. If healthy: 5% → 25% → 50% → 75% → 100%
   - Each step: 5-minute observation window
4. If unhealthy at any step: automatic rollback to previous version
```

**Rollback:** Scale new pods to 0, scale old pods up. <2 minutes.

#### Option C: Feature Flag (Recommended for All User-Facing Changes)
```
1. Deploy code with feature OFF (flag default = false)
2. Enable flag for internal users (dogfooding)
3. Enable flag for 1% of real users (canary within canary)
4. Monitor for 24 hours (error rate, latency, business metrics)
5. Gradually increase: 10% → 25% → 50% → 100%
6. After 7 days at 100%, remove flag and clean up code
```

**Rollback:** Set flag to false. <10 seconds. No deploy needed.

### Stage 3: Post-Deploy Verification (Mandatory)
```
1. Health check endpoints return 200 (deep checks)
2. Error rate < baseline + 0.1% for 5 minutes
3. P95 latency < 1.2x baseline for 5 minutes
4. Core business metrics normal (orders, signups, payments)
5. Database connections stable (no pool exhaustion)
6. Log volume normal (not spiking with errors)
7. Alerting system healthy (test alert fires and resolves)
```

**Gate:** Any anomaly = automatic rollback. No human judgment required.

---

## Database Migrations (Zero-Downtime)

### The Migration Rules
1. **Additive-Only in One Deploy**
   - Add columns as nullable with defaults
   - Add tables, indexes, constraints
   - Never drop, rename, or change column type in same deploy

2. **Destructive Changes Require Dual-Write Period**
   - Deploy 1: Add new column/table, write to both old and new
   - Deploy 2: Backfill data, switch reads to new
   - Deploy 3: Stop writing old, verify no reads
   - Deploy 4: Drop old column/table

3. **Index Creation**
   - PostgreSQL: `CREATE INDEX CONCURRENTLY` (never locks table)
   - MySQL: `ALGORITHM=INPLACE, LOCK=NONE`
   - Large tables (>10M rows): build index in background job

4. **Migration Safety Checklist**
   - [ ] Migration is reversible (down migration exists and is tested)
   - [ ] Dry-run in staging with production-like data volume
   - [ ] Estimated execution time < 30 seconds (or batched)
   - [ ] No table locks > 1 second
   - [ ] Backup completed before migration (if touching user data)
   - [ ] Migration runs in transaction (all-or-nothing)

### Migration Template
```sql
-- SAFE: Add nullable column with default
ALTER TABLE users 
ADD COLUMN newsletter_opt_in BOOLEAN DEFAULT NULL;

-- SAFE: Add index concurrently
CREATE INDEX CONCURRENTLY idx_users_email 
ON users(email);

-- SAFE: Add new table
CREATE TABLE user_preferences (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    theme VARCHAR(20) DEFAULT 'light',
    created_at TIMESTAMP DEFAULT NOW()
);

-- UNSAFE: Drop column (do in Deploy 4 of dual-write sequence)
-- ALTER TABLE users DROP COLUMN old_field;

-- UNSAFE: Rename (use dual-write instead)
-- ALTER TABLE users RENAME COLUMN old_name TO new_name;
```

### Migration Rollback
```sql
-- Reversible: Remove added column
ALTER TABLE users DROP COLUMN IF EXISTS newsletter_opt_in;

-- Reversible: Remove added index
DROP INDEX CONCURRENTLY IF EXISTS idx_users_email;

-- Reversible: Remove added table (cascade handles references)
DROP TABLE IF EXISTS user_preferences CASCADE;
```

---

## Rollback Procedures (Instant)

### Automatic Rollback Triggers
- Error rate > 0.5% for 2 minutes
- P95 latency > 2x baseline for 3 minutes
- Health check fails 3 times in 60 seconds
- Any critical alert fires within 5 minutes of deploy
- Business metric (orders, payments) drops >20% from baseline
- Database connection pool saturation >90%

### Manual Rollback Commands

| Platform | Rollback | Time |
|----------|----------|------|
| **Kubernetes** | `kubectl rollout undo deployment/app` | <30s |
| **AWS ECS** | Update service to previous task definition | <60s |
| **Docker Swarm** | `docker service update --rollback` | <30s |
| **Vercel** | Dashboard → Revert to previous deployment | <10s |
| **Railway** | `railway up` with previous commit SHA | <60s |
| **Render** | Dashboard → Manual Deploy → Previous commit | <60s |
| **Heroku** | `heroku releases:rollback` | <30s |
| **Fly.io** | `fly deploy --image <previous-tag>` | <60s |
| **VPS (systemd)** | Swap symlink to previous binary + `systemctl restart` | <30s |
| **Blue-Green** | Switch load balancer to Blue environment | <10s |

### Rollback Verification
```
1. Execute rollback command
2. Verify previous version is serving traffic (check response headers)
3. Run smoke test on rolled-back version
4. Verify error rate returns to baseline
5. Verify latency returns to baseline
6. Notify team via incident channel
7. Create incident ticket
8. Debug in staging (never in production)
```

---

## Infrastructure as Code (IaC)

### Mandatory Practices
- All infrastructure defined in Terraform / Pulumi / CloudFormation
- State files in remote backend (S3, Terraform Cloud) with locking
- Drift detection runs daily (alert if manual changes detected)
- Plan output reviewed before every apply
- No `terraform apply` from local machines (CI/CD only)

### IaC Testing
- `terraform plan` in CI for every PR
- Policy-as-code (OPA, Sentinel) for security/compliance gates
- Cost estimation before apply (Infracost)
- State file backup before every apply

---

## Environment Management

### The 3 Environments (Minimum)
| Environment | Purpose | Data | Access |
|-------------|---------|------|--------|
| **Local** | Development | Generated fake data | Developer only |
| **Staging** | Pre-production validation | Production snapshot (anonymized) | Engineering + QA |
| **Production** | Live users | Real user data | On-call only (read-only) |

### Environment Parity Rules
- Staging must match production: same OS, same versions, same resource limits
- Configuration differs only in values, never in structure
- Secrets managed via Doppler / 1Password / AWS Secrets Manager
- Required env vars fail fast on startup (no defaults, no fallbacks)

```python
# Production-grade: Fail fast
DATABASE_URL = os.environ["DATABASE_URL"]  # KeyError if missing — correct

# Unacceptable: Silent failure with insecure default
DATABASE_URL = os.environ.get("DATABASE_URL", "postgres://localhost/dev")  # WRONG
```

---

## Feature Flags (Mandatory for All Changes)

### Flag Lifecycle
```
1. Deploy code with flag OFF (default = false)
2. Internal dogfooding (flag ON for team)
3. Canary: 1% real users → 10% → 25% → 50% → 100%
4. Monitor at each stage for 24-48 hours
5. At 100% for 7 days: remove flag, clean up code
6. Archive flag in feature flag system (audit trail)
```

### Flag Requirements
- Every new feature starts behind a flag
- Flags must be toggleable in <10 seconds (no deploy required)
- Flags must be auditable (who toggled, when, why)
- Flags must have expiration dates (auto-alert at 30 days)
- Kill switch: any feature can be disabled instantly if causing issues

### Flag Implementation
```python
if feature_enabled("new_dashboard_v2", user_id=user.id, default=False):
    return render_new_dashboard()
else:
    return render_legacy_dashboard()
```

---

## Monitoring & Observability (SLO/SLI Framework)

### Service Level Objectives (SLOs)
| SLO | Target | Measurement Window | Alert |
|-----|--------|-------------------|-------|
| **Availability** | 99.9% | 30 days | <99.5% for 5 min |
| **Latency (P95)** | <200ms | 7 days | >300ms for 10 min |
| **Error Rate** | <0.1% | 7 days | >0.5% for 2 min |
| **Throughput** | >1000 RPS | 1 day | <500 RPS for 5 min |

### Service Level Indicators (SLIs)
| Indicator | Metric | Tool |
|-----------|--------|------|
| Request latency | `http_request_duration_seconds` | Prometheus + Grafana |
| Error rate | `http_requests_total{status=~"5.."}` | Prometheus + Grafana |
| Availability | `up{job="api"}` | Prometheus |
| Business metrics | `orders_created_total`, `payments_succeeded_total` | Custom metrics |

### Alerting Tiers
| Tier | Response Time | Channel | Example |
|------|-------------|---------|---------|
| **P0 — Critical** | <5 minutes | Page/SMS | Site down, payment failure, data loss |
| **P1 — High** | <15 minutes | Slack + Email | Error rate spike, latency degradation |
| **P2 — Medium** | <1 hour | Slack | Disk space >80%, non-critical service down |
| **P3 — Low** | <24 hours | Ticket | Cost anomaly, minor metric drift |

### Distributed Tracing
- Every request gets a trace ID (propagated via headers)
- Spans for: API gateway, auth service, business logic, database, external API
- Trace sampling: 100% for errors, 1% for success (adjust based on volume)
- Tools: Jaeger, Zipkin, AWS X-Ray, Datadog APM

---

## Security & Compliance

### Pre-Deploy Security Gates
- [ ] SAST scan clean (Semgrep, SonarQube)
- [ ] Dependency scan clean (Snyk, OWASP Dependency-Check)
- [ ] Container scan clean (Trivy, Clair)
- [ ] Secrets scan clean (GitLeaks, TruffleHog)
- [ ] DAST scan clean (OWASP ZAP)
- [ ] SBOM generated and archived
- [ ] Image signed (cosign)
- [ ] No critical or high vulnerabilities

### Runtime Security
- HTTPS only (HSTS headers, TLS 1.3)
- CORS configured per endpoint (never `*` in production)
- Rate limiting per user + per IP + per endpoint
- WAF active (OWASP Core Rule Set)
- DDoS protection (Cloudflare, AWS Shield)
- Input validation on all endpoints (schema validation)
- Output encoding (prevent XSS)
- SQL injection impossible (ORM parameterized queries only)
- AuthZ checked on every endpoint (not just authN)

### Compliance Requirements
| Regulation | Deploy Gate |
|------------|-------------|
| **GDPR** | Data deletion tested, consent tracking verified |
| **SOC2** | Change management log, access controls verified |
| **PCI-DSS** | Card data never touches application servers |
| **HIPAA** | PHI encrypted at rest and in transit |

---

## Disaster Recovery

### Recovery Objectives
| Metric | Target | Test Frequency |
|--------|--------|--------------|
| **RPO (Recovery Point Objective)** | <5 minutes | Quarterly |
| **RTO (Recovery Time Objective)** | <15 minutes | Quarterly |
| **Backup Retention** | 30 days daily, 1 year weekly | Monthly verification |

### Backup Strategy
- Database: Continuous replication + daily snapshots
- File storage: Cross-region replication (S3, GCS)
- Configuration: Git-backed IaC (infrastructure is code)
- Secrets: Managed vault with versioning (1Password, HashiCorp Vault)

### Restore Testing (Quarterly)
```
1. Spin up isolated environment
2. Restore database from latest backup
3. Verify data integrity (row counts, checksums)
4. Verify application functions correctly
5. Document restore time and issues
6. Update runbook with findings
```

### Runbook Automation
- Every critical failure has an automated runbook
- Runbooks are tested in staging quarterly
- On-call engineer follows runbook, does not improvise
- Post-incident: update runbook with lessons learned

---

## Cost Management

### Right-Sizing
- Start with production minimums, scale based on metrics
- Use auto-scaling: scale up at 70% CPU, scale down at 30%
- Minimum 2 instances for zero-downtime deploys
- Use spot/preemptible instances for background jobs

### Cost Alerts
- Monthly budget alert at 80%
- Daily anomaly detection (unexpected cost spikes)
- Tag all resources (environment, service, owner)
- Cleanup: delete unused resources weekly

---

## Incident Response

### Severity Classification
| Severity | Criteria | Response | Communication |
|----------|----------|----------|---------------|
| **SEV1** | Revenue impact, data loss, security breach | All hands | Immediate status page + stakeholder call |
| **SEV2** | Major feature degraded, significant user impact | On-call + team lead | Status page update within 15 min |
| **SEV3** | Minor feature issue, workaround exists | On-call | Slack update within 1 hour |
| **SEV4** | Cosmetic, no user impact | Next business day | Ticket only |

### Incident Response Playbook
```
1. DETECT: Alert fires, user report, or monitoring anomaly
2. RESPOND: On-call acknowledges within 5 minutes (SEV1/2)
3. MITIGATE: Rollback first, debug second (SEV1/2)
4. COMMUNICATE: Update status page, notify stakeholders
5. RESOLVE: Verify fix in production, monitor for 1 hour
6. REVIEW: Post-mortem within 48 hours (SEV1/2)
7. PREVENT: Update runbooks, tests, monitoring (SEV1/2)
```

### Post-Mortem Template
```
- Incident ID and timeline
- Impact (users affected, revenue lost, data touched)
- Root cause (5 Whys)
- Detection time + Response time + Resolution time
- What went well
- What went poorly
- Action items (owner + due date)
- Lessons learned
```

---

## Change Management

### Approval Workflow
| Change Type | Approver | Documentation Required |
|-------------|----------|----------------------|
| Hotfix | On-call lead | Incident ticket + post-mortem |
| Security patch | Security lead + On-call | Vulnerability report + test results |
| Feature release | Product + Engineering lead | Design doc + test plan + rollback plan |
| Infrastructure | Architecture review | RFC + risk assessment + rollback plan |
| Database migration | DBA + Engineering lead | Migration plan + dual-write schedule |

### Change Log
Every deploy is logged with:
- Deploy ID (artifact tag)
- Commit range
- Approver name
- Test results (pass/fail per stage)
- Rollback command tested (yes/no)
- Feature flags toggled
- Business metrics before/after

---

## Emergency Escapes

**When stuck for >10 minutes:** Stop. State what's unclear. Escalate to team lead.
**When scope creeps mid-deploy:** Abort. New scope = new change request = new approval.
**When a "quick fix" touches >3 files:** It's not quick. Write a plan, get approval, schedule deploy.
**When deleting a feature:** Feature flag OFF → wait 7 days → delete code → remove flag. Never comment-out.
**When deploying on Friday:** Forbidden. Emergency fixes require written justification + on-call standby.
**When rollback fails:** Escalate to SEV1. All hands. Use disaster recovery runbook.

---

## The Deployment Creed

> We do not deploy to see if it works. We deploy because we have proven it works.
> We have proven it works because we tested it in staging, verified it against production data, and planned for every failure mode.
> Production is not a gamble. It is the execution of a validated plan.
> If the plan fails, we roll back. If the rollback fails, we recover. If recovery fails, we learn.
> But we never, ever ship without knowing exactly how to undo it.

---

*Companion to CLAUDE.md v3.0 + TESTING.md v3.0 — Production-Grade Shipping | Last updated: 2026-05-10*
