# DEPLOY.md — Product Developer Deployment Protocol

Deploying is not "pushing code." Deploying is **moving user value from your machine to theirs** with minimal risk.

---

## Philosophy

**Deploy small, deploy often, deploy safely.**
A deploy that changes 500 lines is scary. A deploy that changes 50 lines is routine.

**Production is not a test environment.**
If you wouldn't do it to a user's bank account, don't do it in production.

---

## Deployment Types

| Type | Frequency | Risk | Rollback Time |
|------|-----------|------|---------------|
| **Hotfix** | As needed | High | <2 minutes |
| **Feature** | Daily | Medium | <5 minutes |
| **Refactor** | Weekly | Low | <5 minutes |
| **Infrastructure** | Monthly | High | <10 minutes |

**Rule:** If you can't roll back in the time listed, don't deploy.

---

## Pre-Deploy Checklist

### Code
- [ ] All tests pass (unit + integration)
- [ ] E2E smoke tests pass
- [ ] No console errors in staging
- [ ] No new security vulnerabilities (scan completed)
- [ ] CHANGELOG updated (one line per deploy)

### Data
- [ ] Database migrations are reversible
- [ ] Backup completed (if migration touches user data)
- [ ] New fields have defaults or are nullable
- [ ] No destructive schema changes without dual-write period

### Monitoring
- [ ] Error tracking active (Sentry, Rollbar)
- [ ] Performance baseline recorded
- [ ] Alert thresholds configured
- [ ] On-call rotation aware (if applicable)

### Communication
- [ ] Stakeholders notified (if user-facing change)
- [ ] Customer support briefed (if behavior changes)
- [ ] Status page ready (if major feature)

---

## Deployment Pipeline

### Stage 1: Build
```
1. Install dependencies (lockfile only — no floating versions)
2. Run build step (compile, bundle, transpile)
3. Run lint + type check
4. Run unit tests
5. Build Docker image (if containerized)
6. Tag image with commit SHA + timestamp
```

### Stage 2: Staging
```
1. Deploy to staging environment
2. Run database migrations (if any)
3. Run E2E smoke tests
4. Manual QA for user-facing changes (5-minute spot check)
5. Performance check (no >20% regression)
```

### Stage 3: Production (Canary or Full)

#### Option A: Canary Deploy (Recommended for >100 users)
```
1. Deploy to 5% of traffic
2. Monitor for 10 minutes (error rate, latency, custom metrics)
3. If healthy, ramp to 25% → 50% → 100%
4. If unhealthy, auto-roll back to previous version
```

#### Option B: Full Deploy (OK for <100 users or low-risk)
```
1. Deploy to 100% of traffic
2. Run 3-minute smoke test immediately
3. Monitor for 15 minutes
4. If issues detected, roll back immediately
```

### Stage 4: Post-Deploy Verification
```
1. Health check endpoints return 200
2. Error rate < baseline + 0.1%
3. Core user flows functional (smoke test)
4. Database connections stable
5. Log volume normal (not spiking)
```

---

## Database Migrations

### Rules
1. **Never delete a column in the same deploy that stops using it.**
   - Deploy 1: Stop reading/writing column
   - Deploy 2: Remove column

2. **Never rename a table/column.**
   - Create new → dual-write → migrate data → switch reads → drop old

3. **Always make new columns nullable or have defaults.**
   - Existing rows must not break

4. **Large tables (>1M rows): migrate in batches.**
   - Use background jobs, not single transaction

### Migration Safety Template
```sql
-- SAFE: Add nullable column
ALTER TABLE users ADD COLUMN newsletter_opt_in BOOLEAN DEFAULT NULL;

-- SAFE: Add index concurrently (PostgreSQL)
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- UNSAFE: Drop column immediately
-- ALTER TABLE users DROP COLUMN old_field; -- NEVER in same deploy

-- SAFE: Rename via dual-write
-- 1. Add new_column
-- 2. Write to both old_column and new_column
-- 3. Backfill new_column
-- 4. Switch reads to new_column
-- 5. Stop writing old_column
-- 6. Drop old_column
```

---

## Rollback Procedures

### Automatic Rollback Triggers
- Error rate > 1% for 2 minutes
- P95 latency > 2x baseline for 3 minutes
- Health check fails 3 times in 60 seconds
- Any critical alert fires within 5 minutes of deploy

### Manual Rollback Steps
```
1. Identify the bad deploy (commit SHA, timestamp)
2. Run rollback command (see below by platform)
3. Verify previous version is serving traffic
4. Run smoke test on rolled-back version
5. Notify team
6. Create incident ticket
7. Debug in staging (never in production)
```

### Platform-Specific Rollback

| Platform | Rollback Command |
|----------|-----------------|
| Vercel | `vercel --rollback` or dashboard revert |
| Railway | `railway up` with previous commit |
| Render | Dashboard → Manual Deploy → Previous commit |
| Heroku | `heroku releases:rollback` |
| AWS ECS | Update service to previous task definition |
| Kubernetes | `kubectl rollout undo deployment/app` |
| Docker Compose | `docker-compose pull <prev-image> && docker-compose up -d` |
| VPS (systemd) | `systemctl restart app` with previous binary |

---

## Environment Management

### The 3 Environments
| Environment | Purpose | Data |
|-------------|---------|------|
| **Local** | Development | Generated fake data |
| **Staging** | Pre-production validation | Production snapshot (anonymized) |
| **Production** | Live users | Real user data |

### Environment Variables
- Never commit `.env` files
- Use secret managers (Doppler, 1Password, AWS Secrets Manager)
- Staging and production configs should differ only in values, not structure
- Required env vars should fail fast on startup (don't run with defaults)

```python
# Good: Fail fast
DATABASE_URL = os.environ["DATABASE_URL"]  # KeyError if missing

# Bad: Silent failure with insecure default
DATABASE_URL = os.environ.get("DATABASE_URL", "postgres://localhost/dev")
```

---

## Feature Flags

### When to Use
- New feature not ready for all users
- A/B testing
- Gradual rollout (canary)
- Kill switch for risky features

### Implementation
```python
# Simple feature flag
if feature_enabled("new_dashboard", user_id=user.id):
    return render_new_dashboard()
else:
    return render_old_dashboard()
```

### Rules
1. **Every new feature starts behind a flag.**
2. **Flags must be removable in <1 day if broken.**
3. **Clean up flags after 30 days.** (Technical debt accumulates fast)
4. **Default to OFF.** (Opt-in, not opt-out)

---

## Monitoring & Alerting

### The 4 Golden Signals
| Signal | What | Alert Threshold |
|--------|------|-----------------|
| **Latency** | Request duration | P95 > 500ms |
| **Traffic** | Requests per second | Drop > 50% |
| **Errors** | 5xx rate | > 0.5% for 2 min |
| **Saturation** | CPU / memory / disk | > 80% for 5 min |

### Dashboards
- Uptime/status page (public)
- Error tracking (Sentry, Rollbar)
- Performance (Datadog, New Relic, Grafana)
- Business metrics (Mixpanel, Amplitude, PostHog)

### On-Call Rotation (Solo Founder Edition)
Even if you're solo, set up:
- PagerDuty/Opsgenie for critical alerts
- Slack/email for warnings
- SMS for "site is down"
- Escalation to yourself (obviously)

---

## Security Checklist

- [ ] Dependencies scanned (Snyk, npm audit, pip-audit)
- [ ] No secrets in code (use `git-secrets` or `truffleHog`)
- [ ] HTTPS only (HSTS headers)
- [ ] CORS configured correctly (not `*` in production)
- [ ] Rate limiting active (prevent abuse)
- [ ] Input validation on all endpoints
- [ ] SQL injection impossible (ORM parameterized queries)
- [ ] XSS prevention (output encoding, CSP headers)

---

## Emergency Procedures

### Site is Down
```
1. Check: Is it DNS? (dig +trace your domain)
2. Check: Is it the server? (ping, curl health endpoint)
3. Check: Is it the database? (connection count, slow queries)
4. If unclear: Roll back to last known good version
5. If rollback fails: Restart services
6. If still down: Check infrastructure provider status page
7. Communicate: Update status page, notify users if >15 min
```

### Database is Locked / Slow
```
1. Check active queries (pg_stat_activity, SHOW PROCESSLIST)
2. Kill long-running queries if safe
3. Check connection pool saturation
4. If migration caused it: rollback migration, restore from backup
5. Scale database resources if traffic spike
```

### Deploy Broke Critical Feature
```
1. Roll back immediately (don't try to fix forward)
2. Verify rollback restored functionality
3. Debug in staging with the broken commit
4. Fix, test, deploy again
```

---

## Cost Optimization

### Right-Sizing
- Start small (1 CPU, 512MB RAM) — scale up based on metrics
- Use serverless for sporadic workloads (Vercel, Cloudflare Workers)
- Database: start with managed service, not self-hosted

### Auto-Scaling Rules
- Scale up at 70% CPU
- Scale down at 30% CPU
- Minimum 2 instances (for zero-downtime deploys)

### Cleanup
- Delete old Docker images (<5 recent tags)
- Remove unused databases/environments
- Archive old logs (30-day retention for most apps)

---

## Checklist: Deploy Day

- [ ] Pre-deploy checklist complete
- [ ] Staging deploy verified
- [ ] Database migration reversible
- [ ] Rollback command tested (run it once to confirm)
- [ ] Monitoring dashboards open
- [ ] Smoke test script ready
- [ ] Team/support notified (if needed)
- [ ] Deploy during low-traffic window
- [ ] Post-deploy verification complete
- [ ] 15-minute monitoring window clear

---

*Companion to CLAUDE.md v2.0 + TESTING.md | Last updated: 2026-05-10*
