# TrustEnv — Estimated Costs

## Overview

Costs split into two categories:
- **Development** — one-time cost to build Phase 1
- **Infrastructure** — recurring monthly cost to run the system

---

## Development Costs — Phase 1 (13 Weeks)

### Team Required

| Role                        | Responsibility                                               |
|-----------------------------|--------------------------------------------------------------|
| React Native Developer (x2) | Mobile app — all 4 role views, forms, camera, offline        |
| Backend / Supabase Dev (x1) | Schema, RLS, Edge Functions, auto-assignment, push notifs    |
| UI/UX Designer (x1)         | App design, design system, all screens across 4 roles        |
| QA Engineer (x1)            | Testing across iOS + Android, all roles, all survey types    |
| Project Manager (x1)        | Timeline, coordination, TrustEnv communication               |

### Cost Estimate (Market Rates)

| Role                        | Rate (USD/hr) | Hours (13 weeks) | Total           |
|-----------------------------|--------------|------------------|-----------------|
| React Native Dev × 2        | $100–150     | 520 × 2          | $104,000–156,000|
| Backend / Supabase Dev × 1  | $100–150     | 520              | $52,000–78,000  |
| UI/UX Designer × 1          | $75–120      | 400              | $30,000–48,000  |
| QA Engineer × 1             | $60–90       | 300              | $18,000–27,000  |
| Project Manager × 1         | $75–100      | 260              | $19,500–26,000  |
| **Total**                   |              |                  | **$223,500–335,000** |

> Range depends on whether team is freelance, agency, or in-house.
> In-house hires reduce per-hour cost but add overhead (benefits, equipment).

### Cost Reduction Options

| Option                          | Estimated Saving     |
|---------------------------------|----------------------|
| Offshore development team       | 40–60% on dev costs  |
| Reduce to 1 React Native dev    | ~$78,000 saved       |
| PM handled by TrustEnv internal | ~$22,000 saved       |
| Combined Backend + QA role      | ~$25,000 saved       |
| **Lean team total estimate**    | **~$100,000–140,000**|

---

## Infrastructure Costs — Phase 1 (Monthly)

Phase 1 runs entirely on Supabase + Expo. Low overhead.

| Service                  | Plan              | Monthly Cost | Notes                                      |
|--------------------------|-------------------|-------------|---------------------------------------------|
| Supabase                 | Pro               | $25         | 8GB DB, 100GB storage, 50GB bandwidth       |
| Supabase (if scaling)    | Team              | $599        | Unlimited projects, SOC2, priority support  |
| Expo EAS Build           | Production        | $99         | iOS + Android builds, OTA updates           |
| Expo Push Notifications  | Included in EAS   | $0          | Up to 1M notifications/month free           |
| Apple Developer Account  | Annual            | ~$8/mo      | Required for iOS App Store                  |
| Google Play Account      | One-time $25      | ~$0/mo      | Required for Android Play Store             |
| **Total (lean)**         |                   | **~$132/mo**|                                             |
| **Total (scaled)**       |                   | **~$706/mo**|                                             |

> Phase 1 infrastructure is intentionally low-cost.
> Supabase Pro handles up to ~10,000 active users comfortably.
> Upgrade to Team plan when approaching production with multiple admins or compliance needs.

---

## Infrastructure Costs — Phase 2 (Monthly Target: ~$30,000)

Phase 2 introduces decoupled microservices, a web portal, and external notifications.
This is where infrastructure investment scales significantly.

| Service                        | Plan / Size               | Monthly Cost     | Notes                                       |
|-------------------------------|---------------------------|-----------------|---------------------------------------------|
| **Compute (AWS ECS / GCP)**    |                           |                 |                                             |
| Jobs Service                   | 2 vCPU, 4GB RAM × 2       | ~$300           | Core job orchestration                      |
| Notifications Service          | 1 vCPU, 2GB RAM × 2       | ~$150           | Push + email dispatch                       |
| Reports Service (AI future)    | 4 vCPU, 8GB RAM × 2       | ~$600           | AI report generation when added             |
| Auth Service                   | Managed (Supabase / Auth0) | ~$500           | SSO, enterprise auth                        |
| Web App (Next.js on Vercel)    | Pro                       | ~$400           | TrustEnv admin portal                       |
| **Database**                   |                           |                 |                                             |
| Primary Postgres (RDS)         | db.r6g.xlarge             | ~$900           | Production DB with multi-AZ failover        |
| Read Replica                   | db.r6g.large              | ~$450           | Offload read queries                        |
| Redis (ElastiCache)            | cache.r6g.large           | ~$200           | Job queue, session caching                  |
| **Storage & CDN**              |                           |                 |                                             |
| AWS S3                         | ~5TB/month                | ~$115           | PDFs, photos, documents                     |
| CloudFront CDN                 | ~10TB transfer            | ~$850           | Fast asset delivery globally                |
| **Monitoring & Observability** |                           |                 |                                             |
| Datadog / New Relic            | Pro                       | ~$500           | APM, logs, alerts                           |
| Sentry                         | Team                      | ~$80            | Error tracking (mobile + backend)           |
| **Notifications**              |                           |                 |                                             |
| SendGrid                       | Pro (100k emails/mo)      | ~$90            | Email notifications, report delivery        |
| Expo Push                      | EAS Production            | ~$99            | Push notifications                          |
| **Security & Compliance**      |                           |                 |                                             |
| AWS WAF + Shield               | Standard                  | ~$300           | DDoS protection, firewall                   |
| SSL Certificates               | ACM (free on AWS)         | $0              |                                             |
| Secrets Manager                | AWS                       | ~$50            | API keys, credentials management            |
| **Backup & Disaster Recovery** |                           |                 |                                             |
| Automated DB backups           | RDS Snapshots             | ~$100           | 30-day retention                            |
| Cross-region replication       | S3 + RDS                  | ~$200           | Redundancy                                  |
| **DevOps & CI/CD**             |                           |                 |                                             |
| GitHub Actions                 | Team                      | ~$40            | CI/CD pipelines                             |
| Docker / ECR                   | AWS                       | ~$50            | Container registry                          |
| **AI (Future)**                |                           |                 |                                             |
| OpenAI / Anthropic API         | Usage-based               | ~$500–2,000     | Report generation (when added)              |
| GPU Compute (optional)         | On-demand                 | ~$1,000–3,000   | If self-hosted models                       |
| **Support & Maintenance**      |                           |                 |                                             |
| AWS Business Support           | 10% of usage              | ~$500–1,000     | 24/7 support SLA                            |
| **Total (Phase 2 base)**       |                           | **~$6,500–8,000/mo** | Without AI                           |
| **Total (Phase 2 with AI)**    |                           | **~$9,000–13,000/mo** | With AI report generation           |
| **Total (Phase 2 at scale)**   |                           | **~$25,000–30,000/mo** | High traffic, full redundancy      |

> The $30,000/month ceiling is reached at scale with full redundancy, AI services,
> high traffic, and enterprise support. Early Phase 2 will be $6,000–10,000/month.

---

## Total Cost Summary

| Stage                        | One-Time Cost         | Monthly Cost           |
|-----------------------------|----------------------|------------------------|
| Phase 1 Development          | $100,000–335,000     | —                      |
| Phase 1 Infrastructure       | —                    | $132–706               |
| Phase 2 Infrastructure (base)| —                    | $6,500–8,000           |
| Phase 2 Infrastructure (scale)| —                   | $25,000–30,000         |

---

## Cost Optimization Tips

1. **Start on Supabase Pro ($25/mo)** — don't over-provision early
2. **Use Expo EAS** instead of self-managed CI/CD for mobile builds
3. **Defer microservices** until Supabase shows signs of strain (>50k users)
4. **Use reserved instances** on AWS (1-year commitment = ~40% saving on compute)
5. **CDN aggressively** — serve PDFs and photos from CloudFront, not directly from S3
6. **Monitor from day 1** — Sentry + basic Datadog prevent costly surprise incidents
