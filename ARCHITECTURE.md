# Centime Expense — Technical Architecture

## 1. Architecture Overview

Centime Expense is a mid-market corporate expense management solution targeting US companies with $5M–$100M in annual sales. The system follows a **3-tier architecture**: a React single-page application (frontend), a Node.js Backend for Frontend (BFF), and a Java Spring Boot middleware layer that orchestrates all business logic and external integrations.

The middleware delegates expense engine operations to the **Findity API platform** (Expense API for CRUD/OCR/approvals, Admin API for organization configuration/dimensions/policies, Connect API for card transaction ingestion and voucher export). ERP synchronization flows through **Centime GL APIs** with support for QuickBooks Online, QuickBooks Desktop, Sage Intacct, NetSuite, and Microsoft Dynamics 365. Reimbursement payments execute via **Centime AP** (ACH direct deposit), and user provisioning is handled through **Centime User Management**. Real-time card transaction feeds from **Visa RTN** and **Mastercard** are pushed to Findity via the Connect API.

### Design Principles

- **Cloud-native** — deployed entirely on AWS, leveraging managed services (Fargate, RDS, ElastiCache, SQS/SNS, S3)
- **API-first** — every capability is exposed as a versioned API; the frontend is a pure consumer
- **Multi-tenant** — shared infrastructure with row-level data isolation; tenant context propagated via JWT claims
- **White-label ready** — CSS custom properties, configurable branding, custom domain routing, and template-based email for bank partner deployments
- **Horizontally scalable** — stateless services on Fargate with auto-scaling; async processing via SQS for load leveling
- **ADA/WCAG 2.1 AA compliant** — semantic HTML, ARIA landmarks, keyboard navigation, screen reader support, automated accessibility testing

### Architecture Diagram

```
 ┌──────────────────────────────────────────────────────────────────────┐
 │                        CLIENTS                                      │
 │   Browser (React SPA / PWA)          Mobile (PWA / React Native)    │
 └──────────────────┬───────────────────────────┬──────────────────────┘
                    │ HTTPS                     │ HTTPS
                    ▼                           ▼
            ┌───────────────┐          ┌───────────────────┐
            │   Route 53    │          │    CloudFront      │
            │ (custom DNS)  │─────────▶│ (CDN / static      │
            └───────────────┘          │  assets / WL route) │
                                       └────────┬──────────┘
                                                │
                                       ┌────────▼──────────┐
                                       │  Application Load  │
                                       │     Balancer       │
                                       └───┬────────────┬───┘
                                           │            │
                              ┌────────────▼──┐   ┌─────▼──────────┐
                              │  BFF          │   │  API Gateway    │
                              │  (Node.js /   │   │  (WebSocket     │
                              │   Fastify)    │   │   endpoint)     │
                              │  [Fargate]    │   └─────┬──────────┘
                              └────────┬──────┘         │
                                       │                │
                              ┌────────▼────────────────▼──────────┐
                              │    MIDDLEWARE  (Spring Boot 3.2+)   │
                              │    Modular Monolith  [Fargate]     │
                              │                                     │
                              │  ┌──────────┐ ┌──────────────────┐ │
                              │  │ expense  │ │ approval-service │ │
                              │  │ service  │ │                  │ │
                              │  ├──────────┤ ├──────────────────┤ │
                              │  │ card     │ │ policy-service   │ │
                              │  │ service  │ │                  │ │
                              │  ├──────────┤ ├──────────────────┤ │
                              │  │ erp      │ │ procurement      │ │
                              │  │ service  │ │ service          │ │
                              │  ├──────────┤ ├──────────────────┤ │
                              │  │ reimb.   │ │ notification     │ │
                              │  │ service  │ │ service          │ │
                              │  ├──────────┤ ├──────────────────┤ │
                              │  │ admin    │ │                  │ │
                              │  │ service  │ │                  │ │
                              │  └──────────┘ └──────────────────┘ │
                              └──┬───┬───┬───┬───┬───┬────────────┘
                                 │   │   │   │   │   │
                ┌────────────────┘   │   │   │   │   └──────────────┐
                │         ┌──────────┘   │   │   └──────┐           │
                ▼         ▼              ▼   ▼          ▼           ▼
         ┌──────────┐ ┌────────┐   ┌─────┐ ┌─────┐ ┌────────┐ ┌──────────┐
         │ Findity  │ │Amazon  │   │ S3  │ │SQS/ │ │Centime │ │ Centime  │
         │ APIs     │ │RDS     │   │     │ │SNS  │ │  AP    │ │ User     │
         │(Expense, │ │(Pg 16) │   │     │ │     │ │(ACH)   │ │ Mgmt     │
         │ Admin,   │ └────────┘   └─────┘ └─────┘ └────────┘ └──────────┘
         │ Connect) │ ┌────────┐                    ┌────────┐
         └──────────┘ │Redis   │                    │Centime │
                      │(Cache) │                    │GL APIs │
                      └────────┘                    └────────┘
```

---

## 2. Frontend

### Framework

React 18+ with TypeScript. Build tooling is **Vite** for fast development builds and optimized production bundles with code-splitting.

### State Management

- **TanStack Query** for server state — API caching, background polling, optimistic updates for expense edits, automatic revalidation after mutations.
- **Zustand** for UI state — sidebar open/close, modal visibility, form draft persistence, filter state for expense lists.

### Routing

React Router v7 with lazy-loaded route bundles. Route-based code splitting ensures the initial bundle stays small; feature routes (policy builder, procurement, admin) load on demand.

### Forms

React Hook Form with Zod schema validation. Used for expense entry forms, policy builder, user management, requisition forms, and approval delegation. Zod schemas are shared with the BFF for server-side validation parity.

### Styling

Tailwind CSS with a custom **FT-Design token system** that maps design tokens (brand colors, typography scale, spacing scale) to Tailwind's configuration. CSS custom properties enable runtime white-label theming — the BFF injects tenant-specific brand tokens into the document root, and Tailwind utilities reference those properties. This allows a single build to serve multiple branded experiences.

### Real-time

WebSocket client connects through API Gateway to receive live card transaction events. Used for the real-time card feed dashboard and in-app notifications (approval requests, payment status updates).

### Mobile

- **Phase 1:** Progressive Web App (PWA) with responsive design. Installable on home screen, offline receipt capture with background sync when connectivity returns.
- **Phase 2:** Native iOS/Android via React Native, sharing business logic and Zod schemas from the shared-types package.

### Accessibility

ADA/WCAG 2.1 AA compliance enforced through:

- **axe-core** automated testing integrated into CI pipeline
- Semantic HTML elements throughout (nav, main, article, section, form)
- ARIA landmarks and live regions for dynamic content updates
- Full keyboard navigation with visible focus indicators
- Screen reader support tested with VoiceOver (macOS/iOS) and NVDA (Windows)

### Testing

- **Vitest + React Testing Library** — unit and integration tests for components, hooks, and utility functions
- **Playwright** — end-to-end tests covering critical flows (expense submission, approval, reimbursement)
- **Storybook** — component library documentation and visual regression testing

### Key Libraries

| Library | Purpose |
|---------|---------|
| date-fns | Date formatting and manipulation |
| recharts | Charts and dashboards (spend analytics, budget utilization) |
| react-dropzone | Receipt image upload with drag-and-drop |
| @tanstack/react-table | Sortable, filterable, paginated data tables |
| cmdk | Command palette for power users (quick navigation, search) |

---

## 3. Backend for Frontend (BFF)

### Runtime

Node.js 20 LTS with the **Fastify** framework. Fastify provides high throughput, built-in schema validation, and a plugin architecture well suited to the gateway pattern.

### Purpose

Lightweight API gateway that sits between the React frontend and the Java middleware. It is not a business logic layer — it is an orchestration and adaptation layer that optimizes the frontend developer experience.

### Responsibilities

- **Session management and auth token handling** — validates JWTs from Cognito, handles token refresh flows, and manages session cookies for the SPA. The frontend never holds long-lived tokens directly.
- **Request aggregation** — combines multiple middleware calls into single frontend responses. For example, the expense detail view requires expense data (from expense-service), policy evaluation results (from policy-service), and approval chain status (from approval-service), all merged into one response.
- **WebSocket server** — subscribes to SNS/SQS for real-time card transaction events and approval notifications, maintains WebSocket connections with browser clients, and pushes events to the correct tenant/user.
- **SSR for initial page load** — server-side renders the app shell and critical above-the-fold content. SEO is not a concern (this is a SaaS product behind login), but SSR improves perceived load performance.
- **White-label theme resolution** — reads tenant branding configuration from Redis (or PostgreSQL on cache miss), injects CSS custom property values into the HTML response so the page renders with the correct branding immediately.
- **Rate limiting and request validation** — enforces per-tenant and per-user rate limits using Redis counters. Validates request payloads against Zod schemas before forwarding to middleware.
- **File upload proxy** — generates pre-signed S3 URLs for receipt uploads. The frontend uploads directly to S3 (never through the BFF or middleware), and the BFF notifies the middleware of the upload completion.

### Why Separate from Middleware

Separating the BFF from the middleware provides three key benefits:

1. **Frontend team autonomy** — the frontend team owns the BFF and can iterate on API contracts, aggregation logic, and response shapes without coordinating Java deployments.
2. **WebSocket efficiency** — Node.js handles thousands of concurrent WebSocket connections with minimal resource overhead, which would be wasteful to run in the Spring Boot process.
3. **Independent scaling** — the BFF scales based on connected clients; the middleware scales based on business logic processing load. Their scaling profiles are different.

---

## 4. Middleware (Core Services)

### Runtime

Java 21 LTS with **Spring Boot 3.2+**. Uses virtual threads (Project Loom) for efficient handling of blocking I/O calls to external APIs (Findity, Centime GL, Centime AP).

### Architecture

**Modular monolith** — a single deployable Spring Boot application with clear module boundaries enforced through Java package structure, Spring application contexts, and ArchUnit tests. Each module has its own domain model, repository layer, and API surface. Modules communicate through a shared event bus (Spring ApplicationEvents) and well-defined internal interfaces.

This is intentionally not microservices at launch. A modular monolith avoids distributed system complexity (service discovery, distributed transactions, network latency between services) while preserving the option to extract modules into separate services as scale demands. See Phase 3 in the delivery plan.

### Modules

#### expense-service
Orchestrates Findity Expense API calls for expense CRUD operations, report submission, and OCR processing. Enriches Findity responses with Centime-specific data (white-label branding metadata, custom display fields, procurement references). Handles split coding logic where a single transaction is allocated across multiple GL codes, departments, or projects. Manages receipt-to-expense matching when a receipt arrives separately from the transaction.

#### approval-service
Wraps the Findity approval engine with Centime-specific workflow rules. Manages configurable multi-step approval chains, escalation timers (auto-escalate if no action within N hours), delegation rules (out-of-office rerouting), and batch approval logic (approve multiple expenses in one action). Publishes approval state change events to SQS for notification delivery and audit trail.

#### card-service
Receives real-time card transaction data from Visa RTN and Mastercard via webhook endpoints. Validates and enriches transactions (merchant category normalization, currency conversion). Pushes processed transactions to Findity via the Connect API. Orchestrates transaction-to-expense matching. Publishes events to SNS for real-time card feed distribution to connected WebSocket clients.

#### policy-service
Centime-owned policy engine (not delegated to Findity). Stores complex, multi-dimensional policy rules in PostgreSQL. Evaluates expenses against policies including: spending limits (per transaction, daily, monthly), receipt requirements (by amount threshold), category restrictions, preferred vendor enforcement, card usage requirements, and time-based rules (weekend spending, after-hours). Returns a structured result distinguishing **warnings** (soft policy — user can submit with justification) from **hard blocks** (expense cannot be submitted). Syncs dimension-level policy configurations to Findity Admin API for server-side enforcement.

#### erp-service
Orchestrates Centime GL APIs for bi-directional ERP synchronization. Handles GL account mapping (Centime expense categories to ERP chart of accounts), multi-entity support (expenses posted to correct subsidiary), journal entry creation with proper debit/credit structure, and comprehensive error handling with retry logic. Supports all five ERP targets:

- QuickBooks Online
- QuickBooks Desktop
- Sage Intacct
- NetSuite
- Microsoft Dynamics 365

Inbound sync pulls chart of accounts, departments, classes, projects, vendors, and employees on a 15-minute polling schedule. Outbound sync pushes approved expense journal entries on event (triggered by approval completion).

#### procurement-service
Centime-owned module for purchase requisitions and purchase order management. Handles the full requisition lifecycle: creation, multi-level approval, conversion to purchase order, and PO posting to the ERP via Centime GL APIs. Includes budget management with real-time consumption tracking — each approved PO or expense reduces available budget, and budget overruns trigger configurable alerts or hard blocks.

#### reimbursement-service
Calculates reimbursable amounts for personal card expenses (corporate card expenses are excluded from reimbursement). Applies deductions (policy violations, advance balances). Queues payment requests to Centime AP for ACH direct deposit. Tracks payment status end-to-end (queued, processing, paid, failed) and publishes status events for user notification.

#### notification-service
Multi-channel notification delivery: email via Amazon SES, push notifications to PWA/mobile clients, and Slack/Teams bot integration. Uses a template engine that supports white-label email branding (logo, colors, sender name, footer). Templates are stored per tenant and fall back to Centime defaults. Notification preferences are configurable per user (channel, frequency, digest vs. immediate).

#### admin-service
User management orchestration through Centime User Management API. RBAC role and permission management. Audit trail queries and exports. Tenant configuration (branding, policies, approval workflows, ERP connections, notification preferences). Organization hierarchy management (departments, cost centers, projects).

### Cross-cutting Concerns

- **Authentication** — Spring Security configured as an OAuth 2.0 Resource Server. Validates JWTs issued by Amazon Cognito. Extracts tenant_id, user_id, roles, and permissions from JWT claims.
- **Multi-tenancy** — tenant_id extracted from JWT and propagated through ThreadLocal context (or Scoped Values with virtual threads). Row-level security in PostgreSQL ensures queries are always tenant-scoped. ArchUnit tests verify that no repository method lacks tenant filtering.
- **Distributed tracing** — Micrometer instrumentation with AWS X-Ray as the tracing backend. Trace IDs propagated across BFF → Middleware → external API calls for end-to-end request visibility.
- **Structured logging** — SLF4J with Logback in JSON format, shipped to CloudWatch Logs. Every log entry includes traceId, tenantId, userId, and requestId for correlation.
- **Circuit breakers** — Resilience4j circuit breakers wrap all Findity API calls and Centime GL API calls. Prevents cascade failures when an external service degrades.
- **Idempotency** — idempotency keys enforced for all financial operations (expense submission, GL posting, reimbursement request). Keys stored in PostgreSQL with a 24-hour TTL. Duplicate requests within the window return the original response.

---

## 5. Database Layer

### PostgreSQL 16 (Amazon RDS)

Multi-AZ deployment for high availability with automatic failover. Encrypted at rest with AES-256.

**Key tables:**

| Table Group | Tables |
|-------------|--------|
| Tenant config | `tenants`, `tenant_branding` |
| Users & access | `users`, `roles`, `permissions`, `role_permissions`, `user_roles` |
| Policies | `policies`, `policy_rules`, `policy_conditions` |
| Approval workflows | `approval_workflows`, `workflow_steps`, `approval_delegations` |
| Procurement | `requisitions`, `requisition_items`, `budgets`, `budget_entries` |
| ERP integration | `erp_connections`, `erp_sync_log`, `gl_mappings` |
| Audit & system | `audit_log`, `notifications`, `idempotency_keys` |

**Important note:** Expense data (transactions, receipts, reports) lives in Findity. PostgreSQL stores Centime-specific configuration, policies, procurement data, audit trail, and cached reference data (users, GL accounts). This avoids data duplication and ensures Findity remains the system of record for expense data.

Row-level security is implemented using `tenant_id` on every table. PostgreSQL RLS policies enforce tenant isolation at the database level as a defense-in-depth measure beyond application-level filtering.

Read replicas are provisioned for reporting and analytics queries to avoid impacting transactional workloads.

### Redis 7 (Amazon ElastiCache)

Cluster-mode enabled with encryption at rest and in transit.

| Use Case | TTL | Notes |
|----------|-----|-------|
| BFF session cache | 15 min | Sliding expiration on activity |
| JWT token cache | 15 min | Avoids re-validation on every request |
| Policy evaluation cache | 5 min | Hot policy rules for fast evaluation |
| Card feed buffer | 30 sec | Pub/sub for WebSocket distribution |
| Rate limiting counters | 1 min | Sliding window per tenant/user |
| Findity OAuth tokens | Token lifetime - 5 min | Cached until near expiry |

### Amazon S3

| Bucket | Contents | Lifecycle |
|--------|----------|-----------|
| `{env}-receipts` | Receipt images and attachments | S3 IA after 90 days, Glacier after 1 year |
| `{env}-exports` | CSV, Excel, PDF report exports | Delete after 30 days |
| `{env}-branding` | White-label assets (logos, favicons) | No expiration |

All buckets enforce:

- No public access (bucket policy + Block Public Access)
- VPC endpoint for internal access (no internet transit for service-to-service)
- Pre-signed URLs for secure client upload/download (files never transit through application servers)
- Server-side encryption (SSE-S3)
- Versioning enabled on receipts bucket for audit trail

---

## 6. AWS Infrastructure

```
                        ┌─────────────┐
                        │  Route 53   │  ← Custom domains per tenant
                        └──────┬──────┘
                               │
                        ┌──────┴──────┐
                        │ CloudFront  │  ← Static assets, white-label routing
                        └──────┬──────┘
                               │
                  ┌────────────┴────────────┐
                  │    Application Load      │
                  │       Balancer           │
                  └────┬──────────────┬─────┘
                       │              │
              ┌────────┴───┐   ┌──────┴────────┐
              │  BFF       │   │  API Gateway   │
              │  (Fargate) │   │  (WebSocket)   │
              └────────┬───┘   └──────┬────────┘
                       │              │
              ┌────────┴──────────────┴────────┐
              │      Middleware (Fargate)       │
              │      Spring Boot Cluster       │
              └──┬───┬───┬───┬───┬───┬────────┘
                 │   │   │   │   │   │
    ┌────────┐ ┌─┴─┐ │ ┌─┴─┐ │ ┌─┴─┐ │
    │Findity │ │RDS│ │ │S3 │ │ │SQS│ │
    │  APIs  │ │PG │ │ │   │ │ │SNS│ │
    └────────┘ └───┘ │ └───┘ │ └───┘ │
    ┌────────┐ ┌─────┴┐ ┌────┴──┐ ┌──┴───────┐
    │Centime │ │Redis │ │Centime│ │Centime   │
    │GL APIs │ │Cache │ │  AP   │ │User Mgmt │
    └────────┘ └──────┘ └───────┘ └──────────┘
```

### Compute

**ECS Fargate** (serverless containers) — no EC2 instance management. Auto-scaling policies based on CPU utilization, memory utilization, and request count (ALB target tracking).

| Service | Task Range | Resources per Task | Scaling Trigger |
|---------|------------|-------------------|-----------------|
| BFF | 2–8 tasks | 0.5 vCPU / 1 GB | Request count, WebSocket connections |
| Middleware | 2–16 tasks | 1 vCPU / 2 GB | CPU > 60%, request count |
| Worker | 1–4 tasks | 1 vCPU / 2 GB | SQS queue depth |

### Networking

VPC with public and private subnets across 3 Availability Zones. NAT Gateway in each AZ for outbound internet access from private subnets. VPC endpoints for S3, SQS, SNS, RDS, and ECR to keep internal traffic off the public internet.

### API Gateway

WebSocket API for the real-time card transaction feed and live notifications. REST API available as an optional alternative entry point for partner integrations.

### SQS Queues

| Queue | Producer | Consumer | Purpose |
|-------|----------|----------|---------|
| `card-transactions` | Card webhook endpoint | card-service | Incoming card feed processing |
| `gl-posting` | approval-service | erp-service | Approved expense batches → ERP posting |
| `reimbursement` | reimbursement-service | Worker (→ Centime AP) | Payment request submission |
| `notifications` | Various services | notification-service | Email/push/Slack/Teams delivery |
| `audit-events` | All services | Worker (→ PostgreSQL) | Async audit trail writes |

Each queue has a corresponding Dead Letter Queue (DLQ) with CloudWatch alarms that fire after messages land in the DLQ, indicating processing failures requiring investigation.

### SNS Topics

| Topic | Subscribers | Purpose |
|-------|------------|---------|
| `expense-events` | notification queue, audit queue, BFF (WebSocket) | Fan-out for approval status changes, submission events |
| `card-events` | card-transactions queue, BFF (WebSocket), audit queue | Fan-out for card transaction arrival, receipt matching triggers |

### Supporting Services

- **Secrets Manager** — stores Findity OAuth client credentials, ERP connection strings, Centime API keys, and tenant-specific secrets. Secrets are rotated automatically where supported.
- **CloudWatch** — centralized metrics, logs, and alarms. Custom dashboards track API latency (p50/p95/p99), error rates, queue depths, Findity API health, and ERP sync status.
- **X-Ray** — distributed tracing across BFF, middleware, and external API calls. Enables end-to-end latency analysis and bottleneck identification.

---

## 7. Authentication & Security

### Identity Provider

**Amazon Cognito User Pools** serve as the identity provider.

- **SAML 2.0 federation** for enterprise SSO — supports Okta, Azure AD, OneLogin, and any SAML 2.0 compliant IdP
- **OIDC support** for standards-based identity federation
- **SCIM provisioning** (Phase 2) via Cognito Lambda triggers for automated user lifecycle management
- **Multi-tenant approach:** one user pool per tenant for strong isolation, or a shared pool with custom attributes for simpler management (configurable per deployment)

### Authorization

JWT tokens carry structured claims: `tenant_id`, `user_id`, `roles[]`, and `permissions[]`.

RBAC is enforced at two layers:

- **BFF (route-level)** — Fastify middleware checks that the caller's role has access to the requested route before forwarding to middleware
- **Middleware (method-level)** — Spring Security `@PreAuthorize` annotations enforce permission checks on every service method

**Built-in roles:**

| Role | Description |
|------|-------------|
| Employee | Submit expenses, view own reports, upload receipts |
| Manager | Approve/reject team expenses, view team reports |
| Finance Admin | Full expense visibility, GL mapping, ERP config, policy management |
| Executive | Dashboard access, batch approvals, budget oversight |
| Procurement Manager | Requisition approvals, PO management, budget administration |

Custom roles with granular permission matrices are supported through the admin interface.

### Data Security

- **Encryption at rest** — AES-256 for RDS, S3, and ElastiCache
- **Encryption in transit** — TLS 1.2+ enforced on all connections (client-to-CDN, service-to-service, service-to-database, service-to-external-API)
- **PII handling** — bank account numbers and card numbers are masked in all logs, API responses, and UI displays. Full values are never stored in Centime systems (card data lives in Findity/card networks)
- **S3 security** — bucket policies enforce no public access; VPC endpoint restricts access to internal services only

### API Security

- **Rate limiting** — per-tenant limits enforced at API Gateway and BFF layers using Redis sliding window counters
- **Webhook authentication** — request signing validation for card transaction webhook endpoints (HMAC signature verification)
- **CORS** — restricted to registered tenant domains; wildcard origins are never allowed
- **Content Security Policy** — strict CSP headers prevent XSS; script-src limited to self and trusted CDN origins

### Compliance

- **SOC 2 Type II** — audit trail, access controls, encryption, and monitoring designed for SOC 2 compliance
- **PCI DSS** — card data handling is delegated to Findity and the card networks. Centime never stores, processes, or transmits full card numbers (PAN). This reduces PCI scope to SAQ-A level.

---

## 8. Integration Patterns

### Findity API Integration

All three Findity APIs (Expense, Admin, Connect) authenticate via **OAuth 2.0 client credentials flow**. Access tokens are cached in Redis with a 5-minute buffer before expiry to ensure uninterrupted service.

- **Circuit breaker** — opens after 5 consecutive failures within a 30-second window. Enters half-open state after 60 seconds, allowing a probe request through. Closes after 3 successful probes.
- **Retry** — exponential backoff with jitter (1s, 2s, 4s base delays), maximum 3 attempts. Only retries on transient errors (5xx, network timeouts). Never retries on 4xx.
- **Idempotency** — Findity expense IDs serve as idempotency keys for write operations. Duplicate submissions return the original response.
- **Webhook receiver** — the middleware exposes endpoints for Findity async event callbacks (approval status changes, OCR completion, report status updates). Events are validated via signature verification and processed idempotently.

### Centime GL API Integration

- **Async GL posting** — approved expense batches are queued to the `gl-posting` SQS queue for guaranteed delivery. The erp-service worker processes entries with retry logic and DLQ fallback.
- **Batch posting** — multiple journal entries are grouped into a single API request where the target ERP supports it, reducing API call volume.
- **Error handling** — transient failures (network, rate limiting) are retried automatically with exponential backoff. Data errors (invalid GL account, missing dimension) are routed to a manual review queue visible to Finance Admins.
- **Bi-directional sync:**
  - **Inbound** — chart of accounts, departments, classes, projects, vendors, and employees are pulled from the ERP on a 15-minute polling schedule and cached in PostgreSQL.
  - **Outbound** — approved expenses are pushed to the ERP as journal entries, triggered by approval completion events.

### Centime AP Integration

- **REST API** — reimbursement-service calls Centime AP to queue ACH direct deposit payments.
- **Webhook callback** — Centime AP sends payment status updates (processing, paid, failed) to a webhook endpoint. Status updates are persisted and pushed to users via notification-service.
- **Batch payments** — multiple reimbursements are grouped into a single ACH batch where supported, reducing transaction fees and processing time.

### Real-time Card Feed

1. **Visa RTN** and **Mastercard** push transaction data to webhook endpoints hosted on the middleware.
2. Transactions are validated (signature verification, schema validation) and enriched (merchant category normalization, currency conversion).
3. Enriched transactions are pushed to **Findity Connect API** for storage as unmatched transactions.
4. Events are published to the `card-events` **SNS topic**, which fans out to:
   - `card-transactions` SQS queue for receipt matching processing
   - BFF WebSocket handler for real-time UI display
   - `audit-events` queue for audit trail

**Latency target:** less than 5 seconds from card swipe to UI display.

### Slack/Teams Integration

- **Bot framework** — users can submit receipts by uploading photos in a Slack or Teams channel/DM with the Centime bot.
- **Notification delivery** — approval requests, expense reminders, and payment confirmations delivered as bot messages.
- **Actionable messages** (Phase 2) — approve/reject buttons directly in Slack/Teams messages, eliminating the need to open the web app for simple approvals.

### Email Integration

- **Outbound** — Amazon SES for transactional email (approval notifications, payment confirmations, policy violation alerts). Templates support white-label branding.
- **Inbound** — receipt forwarding via SES receive rules. Users email receipts to `receipts@{tenant}.centime.com`. A Lambda function processes the inbound email, extracts attachments, uploads to S3, and creates an unmatched receipt in the expense-service.

---

## 9. Multi-Tenancy & White-Label

### Tenant Isolation

- **Shared infrastructure, isolated data** — all tenants run on the same Fargate clusters, RDS instances, and Redis clusters. Data isolation is enforced through row-level security in PostgreSQL (every table includes `tenant_id`, and RLS policies restrict access).
- **Tenant context propagation** — the `tenant_id` claim in the JWT is extracted at the BFF, forwarded as a header to the middleware, and set in the ThreadLocal/ScopedValue context for the duration of the request. All database queries, cache keys, and external API calls include the tenant context.
- **Findity isolation** — each Centime tenant maps to a separate Findity organization. The mapping is stored in the `tenants` table, and the middleware resolves the correct Findity org credentials from Secrets Manager for each request.

### White-Label Branding

Tenant branding configuration is stored in PostgreSQL (`tenant_branding` table) and cached in Redis:

| Property | Example |
|----------|---------|
| Primary color | `#1A3B6F` |
| Secondary color | `#E8F0FE` |
| Accent color | `#FF6B35` |
| Logo URL | `s3://branding/tenant-123/logo.svg` |
| Favicon URL | `s3://branding/tenant-123/favicon.ico` |
| App title | "First National Expense Manager" |
| Email from name | "First National Bank" |
| Email from address | `expenses@firstnational.com` |
| Custom domain | `expenses.firstnational.com` |
| Email templates | Custom HTML templates per notification type |

**Runtime theming:** the BFF reads tenant branding from cache, injects CSS custom property values (`--color-primary`, `--color-secondary`, etc.) into the HTML shell. Tailwind CSS utilities reference these custom properties, so the same compiled CSS renders differently per tenant.

**Custom domain routing:** Route 53 alias records and CloudFront alternate domain names map `expenses.partnerbankname.com` to the correct tenant context. The BFF resolves the tenant from the hostname and applies the corresponding branding.

### Tenant Provisioning

Tenant creation is API-driven, used by Centime sales operations and bank partner onboarding workflows. Provisioning creates:

1. Cognito user pool (or tenant entry in shared pool)
2. Findity organization via Admin API
3. Initial admin user account
4. Default policy set (receipt thresholds, spending limits)
5. Default approval workflow (manager approval)
6. Branding configuration (Centime defaults, customizable post-creation)
7. DNS and CloudFront configuration for custom domain (if applicable)

**Target:** fully automated provisioning completes in under 5 minutes.

---

## 10. DevOps & CI/CD

### Repository Structure

Monorepo managed with **Turborepo** (or Nx) for coordinated builds and dependency management:

```
centime-expense/
  packages/
    frontend/          # React SPA
    bff/               # Node.js Fastify BFF
    middleware/         # Java Spring Boot
    shared-types/      # TypeScript types + Zod schemas (shared between frontend & BFF)
    infrastructure/    # Terraform modules
```

### CI/CD Pipeline (GitHub Actions)

**On pull request:**
1. Lint + type-check (ESLint, tsc, Checkstyle)
2. Unit tests (Vitest, JUnit)
3. Security scan (Snyk or Dependabot for dependency vulnerabilities)
4. Build verification (Vite build, Gradle build, Docker image build)

**On merge to main:**
1. Build production artifacts
2. Run integration tests against Findity sandbox
3. Deploy to **staging** environment
4. Run smoke tests (Playwright against staging)
5. Deploy to **production** (with manual approval gate)

### Environments

| Environment | Instances | Findity | Data | Purpose |
|-------------|-----------|---------|------|---------|
| dev | Single instance | Shared sandbox | Developer seed data | Local development, feature testing |
| staging | Production-like (multi-AZ) | Dedicated sandbox | Synthetic data | Integration testing, UAT, demo |
| production | Full scale (multi-AZ, auto-scaled) | Production orgs | Real customer data | Live service |

### Deployment Strategy

**Blue/green deployments** via ECS with health checks. The new version (green) is deployed alongside the current version (blue). Traffic shifts to green after health checks pass. Blue remains available for instant rollback.

Zero-downtime deployments are achieved through rolling updates with connection draining.

### Monitoring & Alerting

**CloudWatch dashboards** with the following panels:

- API latency: p50 / p95 / p99 response times by endpoint
- Error rate: 4xx and 5xx by service
- Queue depth: messages in each SQS queue and DLQ
- Findity API health: latency and error rate for each Findity API
- ERP sync status: success/failure counts by ERP type

**PagerDuty integration** for critical alerts:

- Service downtime (health check failures)
- Payment processing failures (reimbursement errors)
- ERP sync failures exceeding threshold
- DLQ message count > 0
- Findity API circuit breaker open

**Business metrics** tracked for operational insight:

- Expense submission rate (by tenant, over time)
- Approval turnaround time (median, p90)
- Corporate card adoption rate
- OCR accuracy and manual correction rate
- Reimbursement processing time

---

## 11. Performance Targets

| Metric | Target |
|--------|--------|
| API response time (p95) | < 200 ms |
| Page load (Largest Contentful Paint) | < 2 s |
| Card feed latency (swipe to UI) | < 5 s end-to-end |
| OCR processing | < 10 s (Findity SLA) |
| GL posting | < 30 s per batch |
| Concurrent users per tenant | 500+ |
| Uptime | 99.9% |
| Recovery Time Objective (RTO) | < 1 hour |
| Recovery Point Objective (RPO) | < 5 minutes |

---

## 12. Phased Delivery

| Phase | Infrastructure Scope |
|-------|---------------------|
| **MVP** | React SPA + BFF + Middleware (modular monolith) + RDS + Redis + S3 + SQS + Cognito + Fargate. All 5 ERP integrations. Real-time card feed. PWA mobile. |
| **Phase 2** | React Native mobile apps. SCIM provisioning. Virtual card management APIs. Advanced WebSocket features. Read replicas for reporting. |
| **Phase 3** | Extract high-traffic modules to separate services (card-service, notification-service). Add ElasticSearch for advanced expense search. ML pipeline for anomaly detection (SageMaker). |
| **Phase 4** | Travel booking integration. Multi-region deployment. Advanced analytics (Redshift/Athena). |

---

*For product requirements, see [SPEC.md](SPEC.md). For mockups, see [mockups/](mockups/).*
