---
name: deployment-protocol
description: Use when preparing to deploy or rollback a release.
---

# Deployment Protocol

## Overview

Deploy only verified artifacts. Every deploy must have a tested rollback plan.

**Core principle:** Staging proves the deploy is safe. Production proves the deploy is correct.

Read `skills/feature-factory/PROJECT_RULES.md` for project-specific values: deployment platform, environments, approval contacts, SLOs, rollback commands, feature-flag system, and secrets management.

## When to Use

Use before any release, hotfix, security patch, infrastructure change, database migration, configuration change, or rollback.

## Pre-Deploy Checklist

Complete all gates before deploying:

1. Artifact built from locked dependencies with deterministic tag
2. Full test suite passed: unit, integration, security scan
3. Container image signed and pushed to immutable registry
4. SBOM generated and archived
5. Staging deploy succeeded with E2E smoke tests
6. Rollback command tested in staging
7. Feature flags defined and default-OFF
8. Approver sign-off recorded per deployment type
9. Change log entry prepared with commit range and metrics baseline

**Gate:** Any unchecked item blocks deploy.

## Deployment Types and Approvals

| Type | Frequency | Risk | Approval Required | Rollback SLA |
|---|---|---|---|---|
| Hotfix | Emergency only | Critical | On-call lead | <2 minutes |
| Security patch | As needed | Critical | Security lead + On-call | <2 minutes |
| Feature release | Weekly batched | Medium | Product + Engineering lead | <2 minutes |
| Infrastructure change | Monthly | High | Architecture review | <5 minutes |
| Database migration | Bi-weekly max | High | DBA + Engineering lead | <5 minutes |
| Configuration change | As needed | Low | Engineering lead | <1 minute |

## The Deployment Pipeline

### Stage 0: Build Artifact

1. Verify lockfile hash
2. Build in clean container
3. Run full test suite
4. Build container image with deterministic layers
5. Tag: `<commit-sha>-<timestamp>-<build-number>`
6. Sign image
7. Push to immutable registry
8. Generate SBOM

**Gate:** Build fails = no artifact.

### Stage 1: Staging Validation

1. Deploy artifact to production-equivalent staging
2. Dry-run database migrations
3. Run E2E smoke tests on critical paths
4. Verify P95 latency under budget
5. Run security DAST scan
6. Run chaos test
7. Spot-check user-facing changes

**Gate:** Any failure = reject artifact. Debug, fix, rebuild, restart.

### Stage 2: Production Deploy

Choose the strategy defined in `skills/feature-factory/PROJECT_RULES.md`.

**Blue-Green:**
1. Deploy new version to Green environment
2. Run deep health checks
3. Run 3-minute smoke test on Green
4. Switch load balancer to Green
5. Keep Blue running for 15 minutes
6. Terminate Blue only after 15 minutes of healthy traffic

**Canary:**
1. Deploy to 5% of production
2. Observe for 10 minutes:
   - Error rate < baseline + 0.1%
   - P95 latency < 1.2x baseline
   - Business metrics normal
3. If healthy, progress: 5% → 25% → 50% → 75% → 100%
4. Observe 5 minutes between each step
5. If unhealthy at any step, rollback automatically

**Feature flag:**
1. Deploy code with flag OFF
2. Enable for internal users
3. Enable for 1% of real users
4. Monitor for 24 hours
5. Gradually increase: 10% → 25% → 50% → 100%
6. Remove flag after 7 days at 100%

### Stage 3: Post-Deploy Verification

Verify for 5 minutes before considering deploy successful:

1. Health checks return 200 with deep checks
2. Error rate < baseline + 0.1%
3. P95 latency < 1.2x baseline
4. Core business metrics normal
5. Database connections stable
6. Log volume normal
7. Alerting system healthy

**Gate:** Any anomaly = rollback. Do not wait for human judgment.

## Database Migrations

1. **Additive-only in one deploy.** Add columns as nullable, add tables/indexes/constraints. Never drop, rename, or change column type in the same deploy.
2. **Destructive changes require a dual-write period:**
   - Deploy 1: Add new schema, write to both old and new
   - Deploy 2: Backfill data, switch reads to new
   - Deploy 3: Stop writing old, verify no reads
   - Deploy 4: Drop old schema
3. **Create indexes safely:**
   - PostgreSQL: `CREATE INDEX CONCURRENTLY`
   - MySQL: `ALGORITHM=INPLACE, LOCK=NONE`
   - Large tables: build index in background job
4. **Migration safety checklist:**
   - Down migration exists and is tested
   - Dry-run completed in staging with production-like data volume
   - Estimated execution time < 30 seconds, or batched
   - No table locks > 1 second
   - Backup completed before touching user data
   - Migration runs in transaction

## Rollback Procedures

### Automatic Rollback Triggers

- Error rate > 0.5% for 2 minutes
- P95 latency > 2x baseline for 3 minutes
- Health check fails 3 times in 60 seconds
- Critical alert fires within 5 minutes of deploy
- Business metric drops >20% from baseline
- Database connection pool saturation >90%

### Rollback Steps

1. Execute the project-specific rollback command from `skills/feature-factory/PROJECT_RULES.md`
2. Verify previous version is serving traffic
3. Run smoke test on rolled-back version
4. Verify error rate and latency return to baseline
5. Notify team via incident channel
6. Create incident ticket
7. Debug in staging, never in production

## Feature Flags

- Every user-facing change starts behind a flag
- Deploy with flag OFF
- Toggle in <10 seconds without deploy
- Audit every toggle: who, when, why
- Set expiration date; alert at 30 days
- Keep kill switch ready
- Remove flag and clean up code after 7 days at 100%

## Monitoring and Observability

- Define SLOs in `skills/feature-factory/PROJECT_RULES.md`
- Track SLIs: latency, error rate, availability, business metrics
- Propagate trace ID on every request
- Alert tiers:
  - P0: <5 minutes, page/SMS
  - P1: <15 minutes, Slack + email
  - P2: <1 hour, Slack
  - P3: <24 hours, ticket

## Security Gates

- SAST scan clean
- Dependency scan clean
- Container scan clean
- Secrets scan clean
- DAST scan clean
- SBOM generated and archived
- Image signed
- No critical or high vulnerabilities

## Environment Management

- Maintain at least three environments: local, staging, production
- Staging must match production: OS, versions, resource limits
- Configuration differs only in values, never in structure
- Secrets managed via vault
- Required env vars fail fast on startup; no insecure defaults

## Incident Response

1. Detect: alert, report, or anomaly
2. Respond: acknowledge within 5 minutes for SEV1/SEV2
3. Mitigate: rollback first, debug second
4. Communicate: update status page and stakeholders
5. Resolve: verify fix, monitor for 1 hour
6. Review: post-mortem within 48 hours for SEV1/SEV2
7. Prevent: update runbooks, tests, monitoring

## Red Flags

**Never:**
- Deploy without a tested rollback plan
- Skip staging validation
- Skip post-deploy verification
- Deploy on Friday except emergency with written justification
- Make destructive schema changes in one deploy
- Run `terraform apply` from a local machine
- Use insecure env-var defaults
- Comment-out code instead of removing a feature via flag

**Stop and escalate when:**
- Blocked for >10 minutes
- Scope creeps mid-deploy
- A "quick fix" touches more than 3 files
- Rollback fails

## Integration

**Required workflow skills:**
- **superskills:feature-factory** — Provides `PROJECT_RULES.md` with project-specific deploy values
- **superskills:test-driven-development** — Use for migration tests and verification scripts
- **superskills:verification-before-completion** — Verify every gate before claiming deploy success
- **superskills:systematic-debugging** — Use when post-deploy anomalies need root-cause investigation
