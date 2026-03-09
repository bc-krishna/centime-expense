# Centime Expense — Product Overview

## What It Is

Centime Expense is a mid-market corporate expense management solution built on Findity's API platform with a Centime-designed UX. It is part of the Centime financial operations suite, where reimbursement payments are handled by Centime AP.

## Strategic Positioning

**Business Model:** Centime generates revenue primarily through credit card interchange. The solution is **free** for companies using Centime-issued cards (US Bank / FNBO) or partner bank cards. Companies using third-party cards pay a per-active-user monthly fee.

**Go-to-Market:** Sold directly to mid-market companies and through bank partner channels with full private-label branding support.

**Target:** Mid-market US companies — $5M–$100M in annual sales.

**Competitive Set:** Fyle, PayHawk, Expensify, SAP Concur (expense); Procurify, Coupa (procurement crossover) — with differentiation through the card interchange model, bank partnership ecosystem, procurement-to-expense bridge, and tight integration with Centime AP/payments.

## Key Design Principles

| Principle | What It Means |
|-----------|--------------|
| **Card Agnostic** | Any Visa/Mastercard corporate or personal card is supported |
| **Bank Partner Friendly** | White-label ready; bank partners can offer under their own brand |
| **Corporate Card First** | Customers can mandate corporate card use and disable personal cards |
| **ERP Agnostic** | Pluggable integration framework with deep connectors for QuickBooks (Online + Desktop), Sage Intacct, NetSuite, and Microsoft Dynamics |
| **Mobile Aware** | Mobile-first design; all features work on iOS, Android, and responsive web |
| **ADA Compliant** | WCAG 2.1 Level AA across all interfaces |
| **Fast Implementation** | Self-service in < 1 day; guided deployment in < 2 weeks |
| **Private Label** | Custom branding, domains, colors, logos, and email templates per tenant |

## Core Capabilities

### 1. Expense Management
- Manual and automated expense entry (multi-currency, mileage, per diem, split coding)
- Expense reports with templates, batch operations, and auto-submit rules
- Attendee tracking for meals and entertainment
- Delegate/assistant support for executive admins

### 2. Receipt Intelligence
- Mobile camera capture with real-time OCR
- Email-in receipts, drag-and-drop upload, SMS/text forwarding
- E-receipt integrations (Uber, Lyft, airlines, hotels)
- AI-powered extraction: merchant, amount, date, tax, line items
- Auto-categorization and receipt-to-transaction matching
- Lost receipt affidavit workflow

### 3. Corporate Card Integration
- **Real-time transaction feeds** from Visa and Mastercard — transactions appear instantly
- Automatic posting to user accounts with merchant enrichment
- Smart receipt matching (amount + date + merchant + GPS proximity)
- Card controls: spend limits, MCC restrictions, virtual cards (Phase 2)

### 4. Policy & Compliance
- No-code policy builder with rules for spending limits, receipt requirements, preferred vendors, personal card restrictions
- Out-of-policy warnings vs. hard blocks (configurable)
- Multiple policy sets by entity, department, role, or location

### 5. Approval Workflows
- Multi-level approval chains (manager hierarchy, cost center, project, custom)
- Amount-based routing with auto-approval for low-risk expenses
- Mobile approvals with push notifications
- Escalation, delegation, and batch approval
- Slack and Microsoft Teams integration (notifications, receipt submission, approve/reject from bot)

### 6. Reimbursement
- Out-of-pocket expenses reimbursed via **Centime AP** (ACH direct deposit)
- Corporate card reconciliation against statements
- Payment status tracking end-to-end
- Payroll integration option (Phase 2)

### 7. ERP Integration
- **All at launch:** QuickBooks Online, QuickBooks Desktop, Sage Intacct, NetSuite, Microsoft Dynamics 365
- Bi-directional sync: master data in, approved expenses out
- Configurable GL mapping, multi-entity support, error handling

### 8. Reporting & Analytics
- Executive dashboard with spend trends, policy compliance, card adoption, and approval metrics
- Standard report library (by department, category, project, employee)
- Custom report builder with export to CSV/Excel/PDF
- Scheduled reports via email

### 9. Purchase Requisitions & PO Creation
- **Purchase requisitions** — Employee-initiated with item details, budget checks, and catalog support
- **Requisition approvals** — Multi-level approval chains, amount-based routing, auto-approve for catalog items
- **PO creation & posting** — Auto-generate POs from approved requisitions, post to ERP via Centime GL APIs
- **Budget management** — Department/project budgets, real-time consumption, threshold alerts
- *Vendor management, PO matching, and downstream processes handled by Centime AP*

### 10. Administration & Security
- SSO (SAML 2.0, OIDC) and SCIM provisioning
- Role-based access control with custom roles
- Full audit trail on all actions
- SOC 2 Type II, PCI DSS, data encryption (AES-256 / TLS 1.2+)
- 99.9% uptime SLA

## Architecture Summary

```
Centime UX (Web + Mobile + White-Label)
                    │
             Centime Middleware
    ┌────────┬──────┼───────┬──────────┐
    │        │      │       │          │
 Findity  Visa/MC  Centime  Centime   Centime AP
 APIs     Real-time GL APIs  User Mgmt (Payments)
 (Expense, Card    (ERP    (Finance
  Admin,   Feeds    Sync)   Users)
  Connect)
```

- **Findity** provides the expense engine (Expense API), org/dimension management (Admin API), and ERP export pipeline (Connect API) — including OCR, approvals, field rendering, and card transaction ingestion
- **Centime GL APIs** handle bi-directional ERP sync (QuickBooks, Sage Intacct, NetSuite, Dynamics)
- **Centime User Management** handles finance team user provisioning, roles, and SSO
- **Centime** owns the UX, middleware, branding, and customer experience
- **Centime AP** handles reimbursement payment execution
- **Visa/Mastercard** deliver real-time transaction data → pushed to Findity via Connect API

## Pricing Summary

| Card Used | Cost to Customer |
|-----------|-----------------|
| Centime-issued (US Bank / FNBO) | Free |
| Partner bank card | Free |
| Third-party corporate card | Per active user / month |
| Personal card only | Per active user / month |

*Active user = submitted, approved, or had card transactions in a calendar month.*

## Phasing

| Phase | Scope |
|-------|-------|
| **Phase 1 — MVP** | Core expense management, receipt OCR, policy engine, approvals, real-time card feeds, receipt matching, all 5 ERP integrations (QBO, QBD, Sage Intacct, NetSuite, Dynamics), mobile apps, white-label, ADA compliance, Centime AP reimbursement, attendee tracking, e-receipts, Slack/Teams, purchase requisitions, PO creation/posting, budget tracking |
| **Phase 2 — Fast Follow** | SCIM, virtual cards, card controls, GPS mileage, per diem, custom fields, delegation, advanced white-label, VAT/GST tax reclaim, payroll reimbursement, Slack/Teams actionable approvals, advanced budgets |
| **Phase 3 — Growth** | AI anomaly detection, multi-entity, international, voice entry, marketplace integrations |
| **Phase 4 — Future** | Travel booking, pre-trip approvals, itinerary management |

## Success Targets

- Time to first expense: **< 5 minutes**
- Guided implementation: **< 2 weeks**
- Receipt auto-match rate: **> 85%**
- OCR accuracy: **> 95%**
- Centime card adoption: **> 60%**
- Mobile submissions: **> 40%**
- Uptime: **99.9%**

---

*For full details, see [SPEC.md](SPEC.md).*
