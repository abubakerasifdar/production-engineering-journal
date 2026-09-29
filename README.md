# Production Engineering Journal

**Case studies from 2+ years as a founding engineer at a SaaS studio.**

I ship production systems. This journal documents the architecture, decisions, and outcomes of that work — without exposing any proprietary code, product names, or client identities.

---

## Why This Journal Exists

My best work lives on private company repositories. I can't show the code. But I can show the thinking.

This journal is my public proof: what I built, why I built it that way, what broke, and what I'd do differently. It's the same way senior engineers at closed-source companies demonstrate their work.

**No company names. No client names. No proprietary code. Just architecture and outcomes.**

---

## What's Inside

| # | Case Study | Focus |
|---|---|---|
| 1 | Multi-Tenant SaaS CI/CD Pipeline | Deploy time cut 78% (2800s → 600s) |
| 2 | Multi-Tenant SaaS Platform (0 → Production) | Full-stack build: auth, campaigns, automation, payments |
| 3 | Notification Module (Multi-Role, Multi-Channel) | Email + SMS across 3 user roles |
| 4 | Desktop + Web Application Deployment | SSL, Cognito, CI/CD, screenshot infrastructure |
| 5 | 40+ Production Failures Diagnosed and Fixed | RDS, VPC, Docker, Auth0, TypeORM, CORS, SSL |
| 6 | AI-Native Development Workflow | Claude Code daily, 9+ features shipped |

---

## At a Glance

| Metric | Value |
|---|---|
| Production products shipped | 6 |
| CI/CD pipelines built | 6 |
| Production failures fixed | 40+ |
| Deploy time reduction | 78% (2800s → 600s) |
| Frontend LOC reduction | 83% (1800 → 300) |
| Developers trained on CI/CD | 4+ |
| Contributions (last 12 months, private repos) | 603 |

---

## Tech Stack

**Backend:** NestJS, Node.js, TypeScript, PostgreSQL, TypeORM, Redis, BullMQ, JWT, Auth0, AWS Cognito

**Frontend:** Next.js, React, TypeScript, Tailwind

**DevOps:** Docker, GitHub Actions, AWS (ECS, ECR, RDS, S3, SES, Lambda, VPC, CloudFront), CI/CD, auto-rollback, health checks

**Integrations:** Stripe, QuickBooks, Brevo, AWS SES, Telnyx, Twilio, SendGrid, HubSpot, WordPress

**AI Tools:** Claude Code (daily, 2+ months), Claude, Figma + Claude workflow

---

## How to Read This Journal

Each case study follows the same structure:

1. **The Problem** — What was broken, slow, or missing
2. **The Action** — What I built and why
3. **The Result** — Measurable outcome
4. **What I'd Do Differently** — Reflection and growth

No fluff. No marketing. Just engineering.

---

# Case Study 1: Multi-Tenant SaaS CI/CD Pipeline

**Role:** Founding Engineer (full ownership)
**Stack:** Docker, GitHub Actions, AWS ECS, AWS ECR, AWS RDS, NestJS, TypeScript
**Timeline:** 3 weeks to design, build, and roll out across 6 products

## The Problem

Deployments were manual. Every release required a developer to SSH into a server, pull code, rebuild, and restart. Average build time: **~2800 seconds (46+ minutes)**. Failures were common. Rollbacks were manual and error-prone. There was no health verification after deploy. Any bad deploy meant downtime until someone noticed.

## The Action

Architected and built a full CI/CD pipeline:

- **Build:** GitHub Actions builds Docker image on every push to `main`
- **Registry:** Image pushed to AWS ECR with commit SHA tagging
- **Deploy:** ECS service updated with new task definition
- **Health check:** Automated health endpoint verification post-deploy
- **Auto-rollback:** If health check fails, task definition reverts to previous version automatically
- **Environment injection:** Dynamic env vars injected per environment (staging, production)

Reduced build time by optimizing the Dockerfile:

- Multi-stage builds
- Layer caching for `node_modules`
- Removed unnecessary dependencies from production image
- Fixed crypto and dist folder issues causing repeated failures

Added deployment documentation so any developer could use the pipeline.

## The Result

- **Deploy time: 2800s → under 600s** (78% reduction)
- Deployments became one-click (or auto on merge to main)
- Health checks catch broken deploys before they reach users
- Auto-rollback eliminated manual panic during bad releases
- **Trained 4+ developers** on how to use the pipeline
- Same pipeline template rolled out to **6 products**

## What I'd Do Differently

- Add blue/green deployment instead of rolling updates
- Add automated integration tests before deploy step
- Add Slack notifications on success/failure
- Move to Terraform for infrastructure-as-code

---

# Case Study 2: Multi-Tenant SaaS Platform (0 → Production)

**Role:** Founding Engineer (full ownership)
**Stack:** NestJS, Next.js, TypeScript, PostgreSQL, TypeORM, Redis, BullMQ, AWS Cognito, Auth0, Stripe, QuickBooks, Brevo, AWS SES
**Timeline:** ~9 months from first commit to production launch

## The Problem

The company needed a SaaS marketing platform that could serve multiple organizations with complete data isolation — email campaigns, SMS campaigns, contact management, automation, and payment processing. No existing code. No spec. No design handoff. Just a business need and a blank repo.

## The Action

Built the entire platform from zero.

**Authentication & Multi-Tenancy:**

- Auth0 + AWS Cognito integration with organization-based isolation
- JWT + refresh token rotation (extended from 1hr to 24hr with lead approval)
- Role-based guards (owner, admin, member)
- Separate user flows for organization owners vs. customer users

**Core Features:**

- Contact management (CRUD, bulk import, segmentation, filters)
- Template engine with rich text editor integration
- Campaign system (email + SMS, scheduling, dynamic content)
- Automation engine with event triggers and form submissions
- Invoice module with Stripe + QuickBooks sync
- Payment gateway integration with provider-agnostic architecture
- Onboarding flow with domain verification and DNS records (SPF, DKIM)

**Email & SMS Pipelines:**

- Email: Brevo + AWS SES with per-org custom sender domains
- SMS: Telnyx + Twilio hybrid with credit tracking
- Template CRUD, per-role customization, dynamic content injection

**Frontend:**

- Migrated from React to Next.js
- Reduced frontend code by **83% (1800 → 300 LOC)** while adding features
- Tailwind + TypeScript hardening

**Infrastructure:**

- Deployed on AWS with CI/CD pipeline (see Case Study 1)
- RDS for PostgreSQL, S3 for assets, SES for email
- Multi-environment setup (dev, staging, production)

## The Result

- Platform launched in production with paying customers
- Multi-tenant architecture serving multiple organizations
- Full email + SMS pipeline operational
- Stripe + QuickBooks integration live
- Automation engine running in production
- Onboarding flow handling real user signups

## What I'd Do Differently

- Introduce feature flags earlier (would have made rollouts safer)
- Add a proper message queue from day 1 (Redis + BullMQ was added late)
- Add unit tests in the first sprint, not later
- Add a proper staging environment from the start

---

# Case Study 3: Notification Module (Multi-Role, Multi-Channel)

**Role:** Founding Engineer (sole owner of module)
**Stack:** NestJS, PostgreSQL, TypeORM, Brevo, AWS SES, Telnyx, AWS Cognito
**Timeline:** ~8 weeks

## The Problem

A production SaaS needed a notification system that could send emails and SMS to three user roles (customer, owner, admin) across multiple booking lifecycle events (created, rescheduled, cancelled, confirmed). No system existed. Email and SMS providers were fragmented. Templates needed to be customizable per organization.

## The Action

Built the notification module end-to-end:

- **Event triggers:** booking created, rescheduled, cancelled, confirmed
- **Per-role templates:** separate templates for customer, owner, admin
- **Multi-channel:** email (Brevo + AWS SES) and SMS (Telnyx)
- **Custom sender:** per-org custom email sender (e.g., `info@org.com`)
- **Template customization:** logo, address, email values per organization
- **Settings merge:** consolidated two separate settings pages into one notification settings page
- **Payment check:** integrated with booking payment state
- **Time zone handling:** fixed timezone mismatch bug for scheduled notifications
- **Delivery tracking:** integrated with CloudWatch for SMS logs and delivery status

## The Result

- Notification module live in production across all 3 user roles
- Email + SMS working end-to-end with delivery tracking
- Custom sender domains verified and operational
- Notification settings consolidated into single client-side page
- Fixed live production issue where owner emails weren't sending
- Reduced support tickets related to booking communication

## What I'd Do Differently

- Add retry logic with exponential backoff from day 1
- Add delivery status dashboard for admins
- Add per-user notification preferences (frequency, channels)
- Add template versioning so orgs can roll back custom changes

---

# Case Study 4: Desktop + Web Application Deployment

**Role:** DevOps + full-stack owner
**Stack:** Docker, AWS ECS, AWS ECR, AWS RDS, AWS Cognito, Node.js, React
**Timeline:** ~6 weeks

## The Problem

A desktop + web application needed a proper deployment pipeline. Builds were failing. Environment variables weren't being injected correctly. SSL was broken (off by default). Cognito user isolation was inconsistent. The screenshot carousel feature had routing bugs. A 140-file backend refactor had broken dependencies.

## The Action

- Fixed SSL configuration (was off by default, now enforced)
- Built CI/CD pipeline (GitHub Actions → ECR → ECS)
- Fixed environment variable injection (per-environment config)
- Fixed AWS Cognito multi-tenant user isolation
- Deployed new version with multi-project support
- Fixed screenshot carousel routing bug
- Fixed desktop app screenshot URL issue by comparing old vs. new code
- Merged 140-file backend refactor and fixed resulting dependency issues
- Rebuilt TypeORM setup after migration issues
- Fixed password reset endpoint
- Fixed forgot password flow

## The Result

- Desktop + web app deployed with stable CI/CD
- SSL fixed and enforced
- Cognito user isolation working
- Multi-project support shipped
- Screenshot feature working across desktop and web
- New version tested with lead and QA
- Full app QA passed with only minor improvement issues remaining

## What I'd Do Differently

- Add automated visual regression tests for the screenshot feature
- Add deployment documentation from day 1
- Add health checks to the desktop app as well as the web app

---

# Case Study 5: 40+ Production Failures Diagnosed and Fixed

**Role:** Production reliability owner
**Stack:** AWS (ECS, ECR, RDS, VPC, S3, SES, Lambda), Docker, TypeORM, Auth0, Cognito, Nginx, CloudFront

## The Problem

Six production systems, all requiring reliable uptime. Failures occurred across database, infrastructure, container, auth, and networking layers. Some failures caused full downtime. Others caused silent data inconsistencies. All needed to be diagnosed and fixed in production.

## Representative Failures Fixed

| Layer | Failure | Fix |
|---|---|---|
| Database | RDS connection refused in prod | Fixed VPC security group rules |
| Database | TypeORM schema sync breaking | Disabled sync, moved to migrations |
| Database | Migration conflict on deploy | Cleaned migration order, added rollback |
| Infrastructure | ECS task definitions missing node pages | Rewrote Dockerfile and task definition end-to-end |
| Infrastructure | CD pipeline stuck on deploy | Fixed health check timeout and rollback logic |
| Infrastructure | ECR image architecture mismatch | Rebuilt with Linux ARM64 target |
| Infrastructure | Load balancer health check failing | Fixed port config and target group |
| Container | Docker build failing on crypto | Removed dist, fixed crypto import order |
| Container | Docker build slow (2800s) | Multi-stage build, layer caching, removed unused deps |
| Auth | Auth0 token audience mismatch | Fixed dev vs. prod audience config |
| Auth | Cognito user limit exceeded | Cleaned up old users, added proper deletion flow |
| Auth | Refresh token expiring too fast | Extended rotation window to 24hr |
| Networking | CORS blocking frontend calls | Fixed order of CORS middleware in NestJS |
| Networking | Frontend calling localhost:5000 in prod | Replaced all hardcoded URLs with env-driven config |
| Networking | CloudFront redirect loop to /blog | Fixed WordPress + CloudFront + React routing |
| Email | SES not sending in production | Fixed domain verification, DKIM records, and sender identity |
| Email | Emails going to spam | Added SPF, DKIM, DMARC records |
| Email | Email verification already verified error | Fixed verification state handling |
| SMS | Telnyx error 20012 (inactive account) | Verified account, added funds, updated sender |
| SMS | SMS not delivering | Fixed sender name (brand name vs. generic) |
| SMS | SMS credits not tracked | Added credit tracking and invoice generation |
| Payments | Stripe onboarding connect setup missing | Added missing step to complete connected account |
| Payments | Payment status not updating | Fixed webhook handling and status sync |

## The Result

- All 6 production systems stable
- Deploy failures reduced dramatically after CI/CD rollout
- Email deliverability improved (SPF, DKIM, DMARC)
- SMS pipeline operational
- Team trained to diagnose similar issues independently
- Documented fixes in internal runbook for future reference

## What I'd Do Differently

- Build an internal runbook of "known issues and fixes" from day 1
- Add monitoring and alerting (CloudWatch alarms, PagerDuty) earlier
- Add automated smoke tests post-deploy
- Add staging environment with production-like data

---

# Case Study 6: AI-Native Development Workflow

**Role:** Founding Engineer using AI tools daily
**Tools:** Claude Code, Claude (extended use), Figma + Claude design workflow
**Duration:** 2+ months of daily use

## The Problem

Solo founding engineers need to ship faster than teams. Traditional development (writing every line by hand) doesn't scale when one person owns architecture, backend, frontend, DevOps, and production. There's no room for busywork when you're the only engineer.

## The Action

Integrated Claude Code into daily workflow:

- **Scaffolding:** Generated initial modules, controllers, services from prompts
- **Debugging:** Used Claude to diagnose TypeORM, Docker, and CI/CD failures
- **Refactoring:** AI-assisted decomposition of large files (used to reduce frontend LOC by 83%)
- **Design conversion:** Claude → Figma design workflow for UI iteration
- **Session management:** Worked through Claude session timeouts and reconnection issues
- **Verification discipline:** Every AI output reviewed, tested, and debugged before shipping
- **Cross-project reuse:** Built prompt patterns for common tasks (module scaffolding, Dockerfile templates, health checks)

## The Result

- **9+ production features shipped** using Claude Code as primary tool
- Faster iteration: significantly reduced time from idea → working code
- AI hallucination caught and corrected before shipping (verification is critical)
- Reduced context-switching: one tool for scaffolding, debugging, and refactoring
- Personal productivity: roughly 3x faster on repetitive tasks
- Debugging complex issues (Docker, TypeORM, CI/CD) became collaborative with AI

## What I'd Do Differently

- Document AI-assisted decisions from day 1 (audit trail)
- Build prompt library for common patterns (reusable across projects)
- Add automated tests specifically targeting AI-generated code edge cases
- Define a "verification checklist" for AI-generated PRs before merge

---

# Cross-Cutting Lessons

Across all 6 case studies, these patterns held true:

**1. Ownership beats specialization.** Solo founding engineers need to know backend, frontend, DevOps, and production. Specialization is for teams.

**2. Feedback loops save lives.** Health checks, auto-rollback, and monitoring prevent most downtime. Build them early.

**3. Migrations > sync.** TypeORM sync in production is a footgun. Always use migrations.

**4. Environment variables are the #1 source of prod bugs.** Never hardcode anything. Ever.

**5. AI is a multiplier, not a replacement.** It makes you faster. It does not make you smarter. Verification is your job.

**6. Documentation pays compound interest.** Every runbook, README, and case study saves future you hours.

**7. Talk to the founder daily.** The best technical decisions come from understanding the business.

---

## Contact

- GitHub: [@abubakerasifdar](https://github.com/abubakerasifdar)
- LinkedIn: [linkedin.com/in/abubaker-asif-dar-a19039247](https://linkedin.com/in/abubaker-asif-dar-a19039247)
- Email: abubakerasifdar100@gmail.com

---
