# Centime Expense — Detailed Product Specification

**Version:** 1.0
**Date:** March 9, 2026
**Status:** Draft

---

## 1. Executive Summary

Centime Expense is a corporate expense management solution built in partnership with Findity (API provider) with Centime delivering the user experience. It is part of the broader Centime suite, with reimbursement payments handled by Centime AP.

### Strategic Objectives

- **Drive commercial card adoption** — The primary business model is card interchange revenue. The solution is free when a Centime-issued card (US Bank or FNBO) or partner bank card is used. Active monthly user fees apply when third-party cards are used.
- **Card agnostic** — Support any Visa/Mastercard corporate or personal card.
- **Bank partner friendly** — Enable bank partners to offer the solution under their own brand or as a value-add.
- **Direct-to-market and channel** — Sell directly to mid-market companies and through bank partner channels.
- **Speed of implementation** — Rapid onboarding and deployment for customers.
- **Private label** — Full white-label/private-label branding support for bank partners and resellers.

---

## 2. Target Market

- **Segment:** Mid-market US companies — $5M–$100M in annual sales
- **Personas:** CFOs, Controllers, AP Managers, Procurement Managers, Finance Teams, Employees, Approvers, Administrators
- **Competitive Set:** Fyle, PayHawk, Expensify, SAP Concur (expense); Procurify, Coupa (procurement crossover)
- **Differentiators:** Card interchange model, bank partnership ecosystem, tight AP integration, procurement-to-expense workflows, private labeling

---

## 3. Architecture Overview

### 3.1 Technology Partners

| Layer | Provider | Role |
|-------|----------|------|
| Expense Engine & APIs | Findity | Expense API, Admin API, Connect API — expense workflows, OCR, approvals, dimensions, export |
| UX / Frontend | Centime | Web app, mobile app, white-label framework |
| GL / ERP Integration | Centime GL APIs | Bi-directional sync with QuickBooks, Sage Intacct, NetSuite, Dynamics |
| User Management | Centime User Mgmt | Finance team user provisioning, roles, SSO |
| Payments / Reimbursement | Centime AP | ACH/check payments for out-of-pocket reimbursements |
| Card Issuing | US Bank, FNBO | T&E / Expense Management credit cards |
| Card Networks | Visa, Mastercard | Real-time transaction feeds |

### 3.2 Integration Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                       Centime Expense UX                         │
│              (Web App / Mobile App / White-Label)                 │
├──────────────────────────────────────────────────────────────────┤
│                     Centime Middleware / BFF                      │
├──────────┬──────────┬───────────┬───────────┬────────────────────┤
│ Findity  │ Visa/MC  │ Centime   │ Centime   │ Centime AP         │
│ APIs     │ Real-time│ GL APIs   │ User Mgmt │ (Reimbursement)    │
│ (Expense,│ Card     │ (ERP      │ (Finance  │                    │
│  Admin,  │ Feeds    │  Sync)    │  Users)   │                    │
│  Connect)│          │           │           │                    │
└──────────┴──────────┴───────────┴───────────┴────────────────────┘
```

### 3.3 Key Design Principles

- **Mobile-first, responsive design** — All features available on mobile
- **ADA/WCAG 2.1 AA compliant** — Full accessibility compliance
- **ERP agnostic** — Pluggable integration framework
- **Multi-tenant** — Single codebase, per-tenant configuration and branding
- **API-first** — All functionality exposed via APIs for partner integrations

---

## 4. Core Features

### 4.1 Expense Reporting

#### 4.1.1 Expense Entry
- Manual expense creation with all standard fields (date, merchant, amount, currency, category, description)
- Multi-currency support with automatic exchange rate lookup
- Itemized expenses (e.g., hotel folio line items)
- Mileage tracking with GPS and map-based route entry
- Per diem calculations (GSA rates and custom rates)
- Recurring expenses
- Duplicate detection and warnings
- Split expenses across multiple cost centers, projects, or GL codes
- Personal expense flagging
- Attendee tracking for meals and entertainment (names, titles, business purpose)
- Billable expense flagging (mark expenses as client-billable)

#### 4.1.2 Expense Reports
- Group expenses into reports for submission
- Report templates (by trip, by period, by project)
- Pre-populated reports from card transactions
- Auto-submit rules for low-value / policy-compliant expenses
- Draft, submitted, approved, rejected, paid statuses
- Report cloning and templates
- Batch operations (select multiple expenses, bulk categorize, bulk submit)

#### 4.1.3 Delegates and Assistants
- Expense entry delegation (executive assistants can submit on behalf of others)
- Configurable delegation permissions

### 4.2 Receipt Management

#### 4.2.1 Receipt Capture
- Mobile camera capture with real-time edge detection and image enhancement
- Email forwarding (receipts@[tenant].centime-expense.com)
- Drag-and-drop upload on web
- Bulk receipt upload
- Forward from email inbox
- E-receipt integrations (direct capture from Uber, Lyft, airlines, hotels)
- SMS/text message receipt forwarding
- Slack and Microsoft Teams receipt submission
- Lost receipt affidavit/declaration workflow

#### 4.2.2 Receipt Processing (OCR/AI)
- Automatic extraction of: merchant name, date, total amount, currency, tax, tip, line items
- Multi-language receipt support
- Handwritten receipt support
- Confidence scoring on extracted data
- Auto-categorization based on merchant
- Receipt-to-transaction matching (see 4.3.3)

#### 4.2.3 Receipt Storage
- Unlimited receipt storage
- Original image preservation
- PDF, JPEG, PNG support
- Receipt audit trail (who uploaded, when, modifications)
- IRS-compliant receipt archival

### 4.3 Corporate Card Management

#### 4.3.1 Card Support
- **Centime-issued cards:** US Bank and FNBO T&E/Expense Management credit cards
- **Partner bank cards:** Cards issued by bank partners in the Centime ecosystem
- **Third-party corporate cards:** Any Visa or Mastercard corporate card
- **Personal cards:** Supported but policy-controllable (customers can disable personal card use)

#### 4.3.2 Real-Time Transaction Feed
- **Visa Real-Time Notifications (RTN)** — Receive authorization-level transaction data in real-time
- **Mastercard Transaction Notifications** — Real-time push of card transactions
- Automatic posting of transactions to user accounts
- Pending vs. settled transaction states
- Transaction enrichment (clean merchant names, logos, categories, MCC codes)
- Declined transaction visibility

#### 4.3.3 Receipt-Transaction Matching
- Automatic matching of uploaded receipts to card transactions based on amount, date, and merchant
- Confidence-scored matching with manual override
- Unmatched transaction alerts (reminder to attach receipt)
- Unmatched receipt suggestions
- Smart matching using location proximity (mobile GPS vs. merchant location)

#### 4.3.4 Card Controls (if supported by issuer)
- Spend limits (per transaction, daily, monthly)
- Merchant category restrictions
- Geographic restrictions
- Temporary card freeze/unfreeze
- Virtual card issuance for one-time or subscription purchases

### 4.4 Policy Engine

#### 4.4.1 Policy Rules
- Spending limits by category, role, department, project
- Receipt requirements (threshold-based — e.g., require receipt for expenses > $25)
- Duplicate expense detection
- Weekend/holiday spending rules
- Preferred vendor enforcement
- Per diem limits
- Mileage rate caps
- Advance booking requirements (travel)
- Personal expense restrictions (enable/disable personal card use)
- Out-of-policy warnings vs. hard blocks (configurable)
- Attendee requirements for meal/entertainment expenses (threshold-based)
- Itemization requirements (e.g., hotel folios over $X must be itemized)

#### 4.4.2 Policy Configuration
- Policy builder UI (no-code rule creation)
- Multiple policy sets (by entity, department, role, location)
- Policy inheritance and overrides
- Policy effective dates
- Audit log for policy changes

### 4.5 Approval Workflows

#### 4.5.1 Workflow Configuration
- Multi-level approval chains
- Approval by: manager hierarchy, cost center owner, project owner, custom
- Amount-based routing (e.g., < $500 auto-approve, $500–$5000 manager, > $5000 VP + Finance)
- Sequential and parallel approval paths
- Auto-approval rules for policy-compliant expenses below threshold
- Escalation rules and timeout handling
- Out-of-office delegation / backup approvers

#### 4.5.2 Approver Experience
- Approve/reject with comments
- Approve individual line items or entire reports
- Send back for revision
- Batch approvals
- Mobile push notification for pending approvals
- Email digest of pending items
- Approval history and audit trail

### 4.6 Reimbursement

#### 4.6.1 Out-of-Pocket Reimbursement
- Integration with Centime AP for payment processing
- ACH direct deposit to employee bank accounts
- Payment status tracking (pending, processing, paid)
- Reimbursement history
- Multi-currency reimbursement
- Payroll integration option (reimburse via paycheck as alternative to AP)

#### 4.6.2 Corporate Card Reconciliation
- Automatic reconciliation of approved expenses against card statement
- Unreconciled transaction reporting
- Statement-level sign-off

### 4.7 Collaboration & Messaging

#### 4.7.1 In-App Communication
- Comments and notes on individual expenses and reports
- @mention team members in comments
- Conversation threads on expense items (submitter ↔ approver dialogue)
- Attachment support in comments

#### 4.7.2 Third-Party Messaging Integrations
- **Slack** — Approval notifications, receipt submission via Slack bot, approve/reject from Slack
- **Microsoft Teams** — Approval notifications, receipt submission via Teams bot, approve/reject from Teams
- Configurable bot commands (submit expense, check status, view pending approvals)

### 4.8 Procurement — Purchase Requisition & PO Creation

*Note: Vendor management, PO matching, receiving, and downstream procurement processes are handled by Centime AP. This module focuses on the upstream requisition-to-PO workflow.*

#### 4.8.1 Purchase Requisitions
- Employee-initiated purchase requests with item details, quantity, estimated cost, and business justification
- Catalog-based requests (pre-approved items/vendors) and free-form requests
- Budget check at request time (warn or block if over budget)
- Link purchase requests to projects, departments, or cost centers
- Request templates for frequently ordered items
- Requisition status tracking (draft, submitted, approved, rejected, converted to PO)

#### 4.8.2 Requisition Approval Workflows
- Multi-level approval chains (separate from expense approval)
- Amount-based routing (e.g., < $1,000 manager, > $1,000 department head + finance)
- Sequential and parallel approval paths
- Auto-approval for pre-approved catalog items below threshold
- Escalation rules and timeout handling
- Approval audit trail

#### 4.8.3 Purchase Order Creation & Posting
- Auto-generate POs from approved requisitions
- PO numbering with configurable sequences
- PO posting to ERP via Centime GL APIs (journal entries or PO documents)
- Email PO to vendor directly from the system
- PO change orders and amendments with re-approval
- PO status tracking (draft, approved, posted to ERP, sent to vendor)

#### 4.8.4 Budget Management
- Department and project budgets with period controls (monthly, quarterly, annual)
- Real-time budget consumption tracking (committed + spent)
- Budget alerts at configurable thresholds (e.g., 75%, 90%, 100%)
- Budget vs. actual reporting
- Multi-level budget hierarchy (company → division → department → project)

### 4.9 Tax Compliance (Phase 2)

- VAT/GST capture on receipts (OCR extraction)
- Tax reclaim workflow for eligible expenses
- Tax code mapping to ERP
- Country-specific tax rules and rates
- Tax summary reports for reclaim filing

### 4.10 Travel (Phase 3 — Future)
- Travel booking integration
- Pre-trip approval workflows
- Itinerary management
- Travel policy enforcement at point of booking

---

## 5. ERP Integrations

### 5.1 Supported ERPs

| ERP | Priority | Integration Type |
|-----|----------|-----------------|
| QuickBooks Online | P0 — Launch | API (REST) |
| QuickBooks Desktop | P0 — Launch | Web Connector / SDK |
| Sage Intacct | P0 — Launch | API (REST/SOAP) |
| Oracle NetSuite | P0 — Launch | SuiteTalk API / RESTlet |
| Microsoft Dynamics 365 | P0 — Launch | Dataverse API / OData |

### 5.2 Integration Data Flows

#### 5.2.1 Master Data Sync (ERP → Centime Expense)
- Chart of Accounts / GL codes
- Cost centers / Departments
- Projects / Classes
- Vendors / Employees
- Tax codes
- Currencies and exchange rates
- Payment terms

#### 5.2.2 Expense Export (Centime Expense → ERP)
- Approved expense reports as journal entries, bills, or expense transactions
- Configurable mapping (which GL accounts, dimensions, classes)
- Batch or real-time sync
- Credit card liability posting
- Tax allocation
- Multi-entity / multi-subsidiary support

#### 5.2.3 Reconciliation
- Sync status tracking per report
- Error handling with retry and manual resolution
- Duplicate prevention
- Audit trail of all sync operations

### 5.3 Integration Framework
- Pluggable connector architecture for adding new ERPs
- Field mapping UI for administrators
- Data transformation rules
- Scheduled and on-demand sync
- Webhook support for real-time events

---

## 6. Mobile Experience

### 6.1 Platform Support
- iOS (latest 2 major versions)
- Android (latest 2 major versions)
- Progressive Web App (PWA) as fallback

### 6.2 Mobile-Specific Features
- Receipt capture with camera (real-time OCR)
- Push notifications (approvals, policy violations, receipt reminders)
- Offline expense entry (sync when connected)
- GPS-based mileage tracking
- Location-based receipt matching
- Biometric authentication (Face ID, Touch ID, fingerprint)
- Mobile-optimized approval workflows
- Expense entry via voice (Phase 2)
- Dark mode support

### 6.3 Responsive Web
- Fully responsive web application for tablet and desktop
- Consistent feature parity between web and mobile

---

## 7. Accessibility (ADA / WCAG Compliance)

### 7.1 Standards
- **WCAG 2.1 Level AA** compliance across all interfaces
- **Section 508** compliance for government/regulated customers

### 7.2 Requirements
- Full keyboard navigation
- Screen reader compatibility (ARIA labels, roles, states)
- Color contrast ratios meeting AA standards (4.5:1 for text, 3:1 for UI components)
- Focus management and visible focus indicators
- Text resizing up to 200% without loss of content
- Alternative text for all images and icons
- Form labels and error messages associated with inputs
- Skip navigation links
- Consistent navigation patterns
- Motion/animation controls (respect prefers-reduced-motion)
- Touch targets minimum 44x44px on mobile
- Caption support for any video content
- Accessibility statement and feedback mechanism

### 7.3 Testing
- Automated accessibility testing in CI/CD (axe-core, Lighthouse)
- Manual testing with screen readers (NVDA, JAWS, VoiceOver)
- Regular third-party accessibility audits

---

## 8. White-Label / Private-Label Branding

### 8.1 Branding Configuration
- Custom logo (header, login, email, mobile app icon)
- Custom color scheme (primary, secondary, accent colors)
- Custom favicon
- Custom login page background
- Custom email templates with partner branding
- Custom domain support (expenses.partnerbankname.com)
- Custom mobile app listing (via wrapper or configuration)
- Custom terms of service and privacy policy links

### 8.2 Brand Management
- Tenant-level branding configuration in admin console
- Real-time preview of branding changes
- Brand asset upload and management
- Brand inheritance (parent brand → child tenants)

---

## 9. Reporting & Analytics

### 9.1 Standard Reports
- Expense summary by department, category, project, employee
- Spend trends over time (monthly, quarterly, annual)
- Policy violation reports
- Top spenders
- Card utilization reports (corporate vs. personal card usage)
- Outstanding / pending expense reports
- Reimbursement aging
- Receipt compliance rates
- Budget vs. actual by cost center
- Duplicate expense reports
- Tax reclaim reports (VAT/GST — Phase 2)
- Attendee compliance reports (meals & entertainment)

### 9.2 Dashboard
- Executive dashboard with KPIs:
  - Total spend (MTD, QTD, YTD)
  - Pending approvals count and age
  - Policy compliance rate
  - Card adoption rate
  - Average processing time
  - Top categories
- Role-based dashboards (Admin, Finance, Manager, Employee)

### 9.3 Custom Reports
- Report builder with drag-and-drop fields
- Filters, grouping, and sorting
- Export to CSV, Excel, PDF
- Scheduled report delivery via email
- Saved report templates

---

## 10. Administration

### 10.1 User Management
- SSO integration (SAML 2.0, OIDC)
- SCIM provisioning for automated user lifecycle
- Role-based access control (Admin, Finance Admin, Approver, Employee, Auditor, Delegate)
- Custom roles and permissions
- Multi-entity / subsidiary user assignment
- User import via CSV
- Bulk user operations

### 10.2 Organization Setup
- Company hierarchy (entities, departments, cost centers)
- Project/job code management
- Category management with GL mapping
- Tax configuration
- Currency settings
- Fiscal year and period configuration
- Custom fields (per-expense, per-report)

### 10.3 Audit & Compliance
- Complete audit trail on all actions (create, edit, approve, delete)
- IP-based access restrictions
- Data retention policies
- GDPR compliance (data export, right to erasure)
- SOC 2 Type II compliance target
- Data encryption at rest and in transit (AES-256, TLS 1.2+)

---

## 11. Notifications

### 11.1 Channels
- In-app notifications
- Push notifications (mobile)
- Email notifications
- Slack notifications and actionable messages
- Microsoft Teams notifications and actionable messages
- SMS (optional, Phase 2)

### 11.2 Notification Events
- Expense report submitted (→ approver)
- Expense report approved / rejected (→ submitter)
- Expense report sent back for revision (→ submitter)
- Receipt required reminder (→ cardholder, configurable frequency)
- Policy violation alert (→ submitter, approver, admin)
- New card transaction posted (→ cardholder)
- Reimbursement processed / paid (→ employee)
- Approval pending reminder (→ approver, configurable escalation)
- ERP sync failure (→ admin)
- Delegate action notification (→ delegator)

### 11.3 Notification Preferences
- Per-user notification preferences (opt in/out by channel per event)
- Admin-enforced notifications (cannot be disabled)
- Quiet hours / do not disturb settings

---

## 12. Pricing Model

### 12.1 Pricing Tiers

| Scenario | Price |
|----------|-------|
| Centime-issued card (US Bank / FNBO) | **Free** — Revenue from interchange |
| Partner bank card | **Free** — Revenue from interchange sharing agreement |
| Third-party corporate card | **Per active user / month** — Tiered pricing |
| Personal card only | **Per active user / month** — Tiered pricing |

### 12.2 Active User Definition
- A user is "active" if they submit, approve, or have card transactions posted in a given calendar month.
- Administrators and auditors who only configure/view are not counted as active users.

### 12.3 Billing
- Monthly billing in arrears based on active user count
- Annual contract options with discount
- Partner bank billing through Centime channel agreements

---

## 13. Security & Infrastructure

### 13.1 Security
- SOC 2 Type II compliance
- Data encryption at rest (AES-256) and in transit (TLS 1.2+)
- PCI DSS compliance for card data handling
- Multi-factor authentication (MFA)
- SSO (SAML 2.0, OIDC)
- Role-based access control with least privilege
- Session management (timeout, concurrent session limits)
- API authentication via OAuth 2.0
- Rate limiting and DDoS protection
- Vulnerability scanning and penetration testing
- Incident response plan

### 13.2 Infrastructure
- Cloud-hosted (AWS or Azure)
- Multi-region availability
- 99.9% uptime SLA
- Automated backups with point-in-time recovery
- Horizontal scaling for peak loads
- CDN for static assets and global performance
- Monitoring, alerting, and logging (APM, error tracking)

### 13.3 Data Residency
- US data residency (primary)
- Configurable data residency for regulated customers (Phase 2)

---

## 14. Implementation & Onboarding

### 14.1 Onboarding Flow
1. Tenant provisioning (automated)
2. Branding configuration (self-service or assisted)
3. Organization setup (hierarchy, cost centers, projects)
4. ERP connection and field mapping
5. Policy configuration
6. Approval workflow setup
7. Card feed activation
8. User provisioning (SSO/SCIM or bulk import)
9. Admin training
10. Employee rollout (with in-app guided tour)

### 14.2 Time-to-Value Targets
- **Self-service setup:** < 1 day for basic configuration
- **Guided implementation:** < 2 weeks for full deployment with ERP integration
- **Data migration:** Tools for importing historical data from competing solutions

### 14.3 Support
- In-app help center and knowledge base
- In-app chat support
- Email support
- Phone support (premium tier)
- Dedicated CSM for enterprise accounts
- Partner support portal for bank partners

---

## 15. Phasing / Roadmap

### Phase 1 — MVP (Launch)
- Expense entry, reporting, and submission
- Receipt capture and OCR (via Findity)
- Policy engine (core rules)
- Approval workflows (multi-level)
- Real-time card transaction feed (Visa + Mastercard)
- Receipt-to-transaction matching
- Corporate and personal card support
- QuickBooks Online integration
- QuickBooks Desktop integration
- Sage Intacct integration
- Oracle NetSuite integration
- Microsoft Dynamics 365 integration
- Mobile app (iOS + Android)
- Web app (responsive)
- ADA/WCAG 2.1 AA compliance
- White-label branding (core)
- Standard reports and dashboard
- SSO (SAML 2.0)
- Centime AP integration for reimbursements
- Attendee tracking for meals & entertainment
- E-receipt integrations (Uber, Lyft, airlines, hotels)
- Lost receipt affidavit workflow
- Slack and Microsoft Teams notifications and receipt submission
- Purchase requisitions, approval workflows, and PO creation/posting
- Budget tracking (department and project level)

### Phase 2 — Fast Follow
- SCIM user provisioning
- Advanced reporting / report builder
- Virtual card issuance
- Card controls (limits, restrictions)
- Mileage GPS tracking
- Per diem automation
- Custom fields
- Delegation / assistant support
- Advanced white-label (custom domain, app listing)
- VAT/GST capture and tax reclaim workflows
- Payroll integration for reimbursement
- Slack/Teams actionable approvals (approve/reject from bot)
- Advanced budget management (multi-level hierarchy)

### Phase 3 — Growth
- AI-powered anomaly detection
- Multi-entity / multi-subsidiary
- International support (multi-language, multi-currency reimbursement)
- Voice-based expense entry
- Advanced analytics and benchmarking
- Marketplace integrations (direct booking data from Uber, Lyft, airlines, hotels)

### Phase 4 — Future
- Travel booking integration
- Pre-trip approval workflows
- Itinerary management

---

## 16. Success Metrics

| Metric | Target |
|--------|--------|
| Time to first expense submitted | < 5 minutes |
| Implementation time (guided) | < 2 weeks |
| Receipt auto-match rate | > 85% |
| OCR accuracy | > 95% |
| Card adoption rate (Centime-issued) | > 60% of users |
| Policy compliance rate | > 90% |
| Mobile usage rate | > 40% of submissions |
| Approval turnaround time | < 24 hours |
| User satisfaction (NPS) | > 50 |
| Uptime | 99.9% |

---

## Appendix A: Findity API Capabilities

*Source: https://developer.findity.com/ (Beta)*

### Authentication
- OAuth 2.0 (client credentials via `/oauth/token`)
- Token revocation (`/oauth/revoke`)
- User authorization grants (`/oauth/authorization-grant`)
- IP address whitelisting

### Expense API (End-User / Embedded UX)

**User Management**
- Get/update logged-in user (`/me`)
- Profile image CRUD
- Email and phone verification challenges
- Password management
- Extra recipients (e.g., CC on receipts)
- User settings (fetch/update/transient)
- Organization counters

**Organization Context**
- User organizations listing
- Expense types and categories per org
- Representation types, car types, fuel types
- Currencies, locales, countries, time zones
- Payment types, guests
- Policy documents
- Custom fields per org
- Commuting data (years/months/days)

**Expense CRUD**
- Create/list/get/update/delete expenses
- Fields-format variant (dynamic field rendering)
- Expense images (receipt attachments)
- Duplicate detection (built-in)

**Expense Reports**
- Create/list/get/update/delete expense reports
- Fields-format variant for dynamic UX
- Report status tracking

**Content & Receipt Management**
- Create/get/delete content (receipt images, documents)
- Trigger actions on content (OCR processing)
- Retrieve as image or binary

**Mileage & Routes**
- Place suggestions (autocomplete)
- Route directions
- Fleet management trips
- Mileage creation (standard and fields format)

**Approvals**
- List approvals (by person, by chain)
- Approve/reject expense reports
- Expense-level approval listing
- Temporary approver search

**Notifications**
- List/get/bulk-update/delete notifications
- Push notification pipeline

**Impersonation**
- Search users for delegation/impersonation

**Field Rendering System**
- Dynamic UI field configuration per expense type
- Initialize and update field sets
- Approval-specific field views
- Expense report field customization

### Admin API (Organization Management)

**Organizations**
- CRUD operations for organizations
- Report counters per org
- Organization lookup (multiple ID formats)

**User & Personnel Management**
- List/add/update/remove employees
- Person data retrieval and updates
- Accounting consultant management

**Dimensions (GL Coding)**
- Create/manage dimensions (cost centers, projects, departments, etc.)
- Dimension values CRUD
- Approver assignment per dimension value
- Maps to GL/ERP coding structures

**Organizational Units**
- Hierarchical org unit management

**Chart of Accounts**
- CRUD operations for GL accounts
- Lock/unlock accounts

**Custom Fields**
- Define custom field schemas
- Manage field values per org

**Categories & Policies**
- Export category management
- IRIS category updates
- Policy document CRUD

**Carbon Emissions (Sustainability)**
- Total, monthly, per-dimension emissions
- Top emitters, intensity metrics
- Distance by vehicle type
- Export emission reports

**Partner Management**
- Partner consultant and organization listing
- Partner expense report access

**Licenses**
- Organization, partner, and application license tracking

**Files**
- File listing and retrieval
- Export file access

### Connect API (ERP/Accounting Integration)

**Exports (Voucher Pipeline)**
- List/get exports per organization
- Get export vouchers (structured accounting entries)
- Process/update export content
- Update export status (mark as synced/posted)
- Recipient setup required (API recipient type)

**Card Transactions**
- POST card transactions into Findity (`/card-transactions`)
- Real-time transaction ingestion from Visa/Mastercard feeds
- Transaction-to-expense matching

**Integration Webhooks**
- Webhook-based event notifications for real-time sync

### Centime Integration Points

| Findity Capability | Centime Usage |
|---|---|
| Expense API | Powers the Centime Expense UX — all expense entry, editing, reporting |
| Content API | Receipt capture, OCR processing, image storage |
| Approvals API | Multi-level approval workflows in Centime UX |
| Admin API — Dimensions | Maps to GL codes, cost centers, projects from Centime GL APIs |
| Admin API — Users | Synced with Centime User Management for finance team users |
| Admin API — Organizations | Multi-tenant setup, white-label org provisioning |
| Connect API — Exports | Centime GL APIs consume vouchers for ERP posting |
| Connect API — Card Transactions | Centime middleware ingests Visa/MC real-time feeds → pushes to Findity |
| Field Rendering System | Centime UX uses dynamic fields for configurable expense forms |
| Impersonation API | Powers delegate/assistant functionality |
| Notifications API | Feeds Centime notification system (push, email, Slack/Teams) |

## Appendix B: Glossary

| Term | Definition |
|------|-----------|
| Active User | A user who submits, approves, or has card transactions in a calendar month |
| BFF | Backend for Frontend — middleware layer between UX and APIs |
| Interchange | Fee paid by merchant's bank to cardholder's bank on card transactions |
| MCC | Merchant Category Code — 4-digit code classifying business type |
| OCR | Optical Character Recognition — extracting text from images |
| PO | Purchase Order — formal document authorizing a purchase from a vendor |
| RTN | Real-Time Notifications — Visa's real-time transaction notification service |
| T&E | Travel & Entertainment |
| White-label | Solution branded as the partner's own product |
