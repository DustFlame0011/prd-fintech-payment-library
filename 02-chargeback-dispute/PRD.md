# PRD: Chargeback Dispute System

**Version:** 1.0  
**Status:** Draft  
**Author:** Sukumal 

**Last Updated:** 23-Feb-2025  
**PRD Owner:** Product - Payments & Risk  
**Stakeholders:** Compliance, Risk, Merchant Ops, Engineering, Legal

---

## 1. Overview & Problem Statement

### Background

A chargeback occurs when a cardholder disputes a transaction with their issuing bank, requesting a forced reversal of funds. Payment platforms act as intermediaries — the merchant must submit evidence to challenge the dispute, or the funds are automatically returned to the cardholder.

Currently, merchants on the platform manage chargebacks through a fragmented process: they receive notifications via email, submit evidence as email attachments, and have no real-time visibility into case status. The compliance team manually tracks all cases in spreadsheets.

### Problem

- Merchants miss chargeback deadlines (typically 7–14 days from issuing bank) due to email notifications going to spam or being overlooked
- Evidence submission process is error-prone — merchants submit wrong document types, incomplete files, or files that exceed size limits
- No centralised dashboard means merchants contact support to check case status, generating high ticket volume
- Compliance team spends ~40% of time on manual data entry and status updates across cases
- Win rate for disputes is below industry benchmark due to poor evidence quality at submission

### Opportunity

Build a self-serve Chargeback Dispute Management portal that gives merchants real-time case visibility, guided evidence collection, automated deadline reminders, and structured submission workflows — reducing manual compliance overhead by 60% and improving merchant dispute win rate to above 65%.

---

## 2. Goals & Success Metrics

| Goal | Metric | Baseline | Target |
|------|--------|----------|--------|
| Reduce missed deadlines | % cases submitted before deadline | ~55% | > 90% |
| Improve dispute win rate | % of disputed cases won | ~42% | > 65% |
| Reduce support ticket volume | Chargeback-related support tickets/month | ~380 | < 120 |
| Reduce manual compliance work | Hours/week on manual case tracking | ~18 hrs | < 5 hrs |
| Faster evidence submission | Avg. time from notification to submission | 4.2 days | < 1 day |

---

## 3. User Personas

### Persona A — SME Merchant (Primary)
- Thai small-to-medium business owner, 30–50 years old
- Low-to-medium familiarity with payment dispute processes
- Pain points: confused by chargeback reason codes, unsure what evidence to submit, no visibility into timelines
- Motivation: Recover lost revenue from fraudulent disputes or customer errors

### Persona B — Enterprise Merchant Finance Team (Secondary)
- Finance manager or accounts receivable team at a mid-to-large company
- Handles 20–100+ chargebacks per month
- Needs: bulk case management, export capabilities, team access with role permissions
- Pain points: cannot delegate case management to junior staff without losing oversight

### Persona C — Compliance Officer (Internal)
- Internal platform compliance team member
- Needs: full visibility into all open cases, ability to add notes, escalate to card networks, download evidence packages
- Pain points: manual status tracking in spreadsheets, no audit trail for case decisions

---

## 4. User Stories

See [`user-stories.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/02-chargeback-dispute/user-stories.md)

---

## 5. Functional Requirements

### FR-01: Case Creation & Notification

- System shall automatically create a chargeback case record upon receiving dispute notification from card network (Visa/Mastercard)
- System shall send in-app notification, email, and SMS (if opted in) to merchant within 30 minutes of case creation
- System shall parse chargeback reason code and map it to human-readable description and evidence requirements
- System shall calculate and display the response deadline based on card network rules (Visa: 30 days from notification, Mastercard: 45 days)

### FR-02: Evidence Collection & Submission

- System shall display a checklist of recommended evidence types based on reason code (see Evidence Requirements Matrix below)
- System shall accept file uploads in PDF, JPG, PNG, XLSX formats up to 25MB total per submission
- System shall validate file count, total size, and format before submission
- System shall allow merchant to add a freeform rebuttal letter (max 2,000 characters)
- System shall generate a formatted evidence package (PDF bundle) for card network submission
- System shall send submission confirmation with case reference number

### Evidence Requirements Matrix

| Reason Code Category | Required Evidence |
|----------------------|------------------|
| Fraud / Unauthorised | Transaction receipt, IP/device log, cardholder auth confirmation |
| Item Not Received | Proof of delivery, tracking number, shipping confirmation |
| Item Not as Described | Product description, photos, terms and conditions, customer communications |
| Duplicate Transaction | Transaction logs showing both charges, customer acknowledgement |
| Credit Not Processed | Refund confirmation, customer communications, return receipt |
| Subscription Cancelled | Subscription agreement, cancellation policy, service usage logs |

### FR-03: Case Tracking & Status Updates

- System shall display real-time case status: `Open` → `Evidence Submitted` → `Under Review` → `Won` / `Lost` / `Accepted`
- System shall notify merchant via email and in-app at every status change
- System shall display outcome reason when case is resolved (won/lost), including card network decision notes
- System shall track win/loss history and display win rate metrics on merchant dashboard

### FR-04: Automated Reminders

- System shall send reminder notifications at: 7 days, 3 days, and 24 hours before response deadline
- System shall escalate unresponded cases to compliance team queue at 48 hours before deadline
- System shall mark cases as `Expired` if deadline passes without submission and notify merchant of outcome

### FR-05: Compliance Dashboard (Internal)

- System shall provide compliance team with a paginated case queue across all merchants
- System shall allow compliance officer to filter by: merchant, status, card network, reason code, deadline proximity
- System shall allow compliance officer to add internal notes to any case (not visible to merchant)
- System shall generate weekly summary report: total cases opened, resolved, win rate by reason code

---

## 6. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Case dashboard loads within 2 seconds for up to 500 cases |
| Availability | 99.95% uptime — chargebacks are time-critical |
| Security | All evidence files encrypted in transit (TLS 1.2+) and at rest (AES-256) |
| Compliance | Audit log maintained for all case actions (who did what, when) |
| Data Retention | Case records and evidence retained for 7 years per financial regulation |
| Notifications | Email delivery SLA < 5 minutes from trigger event |
| Localisation | Thai and English language support for all merchant-facing UI |
| Accessibility | WCAG 2.1 AA compliant |
| Card Network Integration | Supports Visa Resolve Online (VROL) and Mastercard Connect API |

---

## 7. Acceptance Criteria

See [`acceptance-criteria.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/02-chargeback-dispute/acceptance-criteria.md)

---

## 8. Edge Cases & Out of Scope

### Edge Cases

| Scenario | Expected Behaviour |
|----------|-------------------|
| Merchant submits evidence after deadline | System rejects submission. Display: "This case has expired. Evidence can no longer be submitted." Notify compliance team. |
| Chargeback filed on already-refunded transaction | System flags case as `Pre-Arbitration — Refund Already Issued`. Prompt merchant to upload refund confirmation as primary evidence. |
| Same transaction disputed twice (re-presentment) | System links both cases and displays history. Alert compliance team of re-presentment scenario. |
| Merchant accepts dispute without submitting evidence | System allows merchant to mark case as `Accepted` before deadline. Record acceptance reason (dropdown: fraudulent transaction, refund issued, customer error, other). |
| File upload interrupted mid-way | System saves uploaded files to draft. On return, show previously uploaded files and allow resumption. |
| Card network integration timeout | System queues the submission for retry (max 3 attempts, 5-minute intervals). Notify compliance team if all retries fail. |
| Merchant account suspended during active case | Compliance team notified. Case remains open and managed internally. Merchant notified via email that their case is being handled by the team. |

### Out of Scope (v1.0)

- Pre-dispute alerts / Ethoca / Verifi integration (planned v1.2)
- Automated rebuttal letter generation using AI (planned v2.0)
- Multi-currency chargeback handling beyond THB and USD (planned v1.3)
- American Express dispute network integration
- Team-level role permissions for enterprise merchants (planned v1.1)

---

## Appendix: Chargeback Reason Code Reference

| Network | Code | Description |
|---------|------|-------------|
| Visa | 10.4 | Fraud — Card Absent Environment |
| Visa | 13.1 | Merchandise / Services Not Received |
| Visa | 13.3 | Not as Described or Defective |
| Visa | 12.6 | Duplicate Processing |
| Mastercard | 4853 | Cardholder Dispute |
| Mastercard | 4855 | Goods or Services Not Provided |
| Mastercard | 4837 | No Cardholder Authorisation |

*Full reason code mapping maintained in: `/api/config/reason-codes`*
