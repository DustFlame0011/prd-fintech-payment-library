# References & Source Documentation

> This document lists all authoritative sources referenced across PRDs in this repository.  
> Maintained for transparency, interview preparation, and ongoing research.

---

## Table of Contents

1. [Chargeback & Dispute Management](#1-chargeback--dispute-management)
2. [Payment Retry Logic & Card Network Rules](#2-payment-retry-logic--card-network-rules)
3. [Security Standards — Encryption & TLS](#3-security-standards--encryption--tls)
4. [Thailand Regulatory Context — BOT & KYC](#4-thailand-regulatory-context--bot--kyc)
5. [General Payments & Fintech Reference](#5-general-payments--fintech-reference)
6. [Referenced In](#6-referenced-in)

---

## 1. Chargeback & Dispute Management

### Official Card Network Documentation

| Source | Description | URL |
|--------|-------------|-----|
| **Visa** | Dispute Management Guidelines for Merchants (June 2024) — official PDF covering all reason codes, timelines, and evidence requirements | https://usa.visa.com/content/dam/VCOM/global/support-legal/documents/merchants-dispute-management-guidelines.pdf |
| **Stripe Docs** | Evidence requirements by dispute category — developer-friendly breakdown of required documents per reason code | https://docs.stripe.com/disputes/reason-codes-defense-requirements |

### Supplementary Reading

| Source | Description | URL |
|--------|-------------|-----|
| Chargebacks911 | Visa reason code reference — all codes with timeline and required evidence | https://chargebacks911.com/chargeback-reason-codes/visa/ |
| Checkout.com Blog | Visa Compelling Evidence 3.0 (CE3.0) update — April 2023 rule change for fraud code 10.4 | https://www.checkout.com/blog/visa-compelling-evidence-3-0 |
| Sift | Visa reason code overview — 4 main categories (Fraud 10, Auth 11, Processing 12, Consumer 13) | https://sift.com/resources/trust-and-safety-university/visa-reason-codes/ |

### Key Facts for Reference

- Visa dispute categories: Fraud (10), Authorization (11), Processing Errors (12), Consumer Disputes (13)
- Visa timeline: Cardholder has 120 days to file; merchant has 20 days to respond after notification
- Evidence package must be submitted as a single formatted bundle to card network

---

## 2. Payment Retry Logic & Card Network Rules

### Official / Near-Official Documentation

| Source | Description | URL |
|--------|-------------|-----|
| **PayPal / Braintree** | Avoiding excessive retry penalties — Visa retry categories 1–4 and Mastercard TPE program explained | https://www.paypal.com/us/brc/article/avoid-excessive-retries-penalties |
| **Mastercard (via Braintree Docs)** | TPE Program fee schedule 2024–2025, MAC codes, Authorization Optimizer | https://developer.paypal.com/braintree/articles/risk-and-security/compliance/network-updates/2023/mastercard-f23 |
| **Checkout.com Blog** | Payment retry guide — hard vs soft decline explanation, dunning strategy, recovery benchmarks | https://www.checkout.com/blog/payment-retries-guide |

### Key Facts for Reference

- **Mastercard TPE**: Max 10 retries within 24 hours; penalty applies beyond 35 retries in 30 days ($0.50/transaction from 2025)
- **Mastercard MAC 03 & 21**: Retrying a transaction flagged as fraudulent or lost/stolen within 30 days incurs penalty
- **Visa Category 1**: Hard decline — retrying triggers an immediate fee per attempt
- **Industry benchmark**: ~20% of failed payments can be recovered via retry engine (Checkout.com data)
- **Consumer behaviour**: 45% of consumers will not retry a failed payment themselves

---

## 3. Security Standards — Encryption & TLS

### Official Standards Bodies

| Source | Description | URL |
|--------|-------------|-----|
| **PCI Security Standards Council** | Official FAQ — which TLS versions comply with PCI DSS; why SSL and early TLS are prohibited | https://www.pcisecuritystandards.org/faq/articles/Frequently_Asked_Question/does-pci-dss-define-which-versions-of-tls-must-be-used/ |
| **Evervault Blog** | PCI DSS encryption requirements 2025 — AES-256, RSA, key management explained clearly | https://evervault.com/blog/encryption-requirements-for-PCI-compliance-2025 |
| **Thoropass Blog** | PCI DSS 4.0.1 encryption requirements — latest version summary | https://www.thoropass.com/blog/pci-dss-encryption-requirements |

### Key Facts for Reference

- **PCI DSS Requirement 3**: Protect stored cardholder data → AES-256 for data at rest
- **PCI DSS Requirement 4**: Protect data in transit → TLS 1.2 minimum; SSL and TLS 1.0/1.1 prohibited
- **AES-256 vs AES-128**: Both NIST-approved; AES-256 is the gold standard for large-volume financial data
- **PCI DSS 4.0.1**: Current version as of 2024 — replaces 3.2.1 which expired March 2024
- Payment platforms must be PCI DSS compliant to legally process card transactions

---

## 4. Thailand Regulatory Context — BOT & KYC

### Official Regulatory Documents

| Source | Description | URL |
|--------|-------------|-----|
| **Bank of Thailand (Official PDF)** | KYC Regulations for e-Money Service — BOT Notification B.E. 2563 (2020) | https://www.bot.or.th/content/dam/bot/fipcs/documents/FPG/2563/EngPDF/25630117.pdf |
| **Tilleke & Gibbins** | BOT Draft Guidelines for Digital Fraud Management 2025 — latest regulatory update | https://www.tilleke.com/insights/bank-of-thailand-releases-draft-guidelines-for-digital-fraud-management/12/ |
| **Know Your Customer Blog** | Thailand AML/KYB regulatory landscape — covers AMLO, BOT, FATF requirements | https://knowyourcustomer.com/insights/articles/thai-aml-kyb-regulations/ |

### Key Facts for Reference

- BOT requires KYC/CDD framework across entire customer lifecycle under Payment System Act B.E. 2560 (2017)
- BOT Notification 2563: e-money operators must complete 2-step KYC — Identify + Verify; corporate merchants must identify authorized persons
- BOT mandates continuous monitoring including risk assessment for mule accounts
- AMLO (Anti-Money Laundering Office) oversight applies to transactions above reporting thresholds
- Non-compliance with BOT KYC requirements can result in license suspension

---

## 5. General Payments & Fintech Reference

### Industry Knowledge

| Source | Description | URL |
|--------|-------------|-----|
| Stripe Documentation | Comprehensive payment concepts, webhooks, idempotency, decline codes | https://docs.stripe.com |
| Adyen Documentation | Chargeback process flow, result codes, and dispute handling | https://docs.adyen.com/risk-management/disputes-and-chargebacks |
| Omise / Opn Payments Docs | SEA-specific payment platform reference — relevant to Thai market context | https://docs.opn.ooo |
| 2C2P Documentation | SEA payment gateway reference — Thai market | https://developer.2c2p.com |

### SEA Payments Market Context

| Source | Description | URL |
|--------|-------------|-----|
| Bank of Thailand — Payment Systems | BOT payment statistics and oversight framework | https://www.bot.or.th/en/financial-innovation/payment-systems.html |
| Electronic Transactions Development Agency (ETDA) | Thailand digital payment and e-commerce policy | https://www.etda.or.th/en |

---

## 6. Referenced In

| Reference Area | PRD File | Section |
|----------------|----------|---------|
| Visa Chargeback Reason Codes | `02-chargeback-dispute/PRD.md` | §5 FR-01, Appendix B |
| Evidence Requirements Matrix | `02-chargeback-dispute/PRD.md` | §5 FR-02 |
| Card Network Response Timeline | `02-chargeback-dispute/PRD.md` | §5 FR-01, §6 NFR |
| Mastercard/Visa Retry Limits | `03-payment-retry-logic/PRD.md` | §5 FR-02 |
| Decline Code Classification | `03-payment-retry-logic/PRD.md` | §5 FR-01 |
| Webhook Idempotency | `03-payment-retry-logic/PRD.md` | §6 NFR, §7 AC-005 |
| AES-256 / TLS 1.2+ | `02-chargeback-dispute/PRD.md` | §6 NFR |
| AES-256 / TLS 1.2+ | `01-merchant-onboarding/PRD.md` | §6 NFR |
| PCI DSS Compliance | All PRDs | §6 NFR |
| BOT e-KYC Guidelines | `01-merchant-onboarding/PRD.md` | §5 FR-02, §6 NFR |

---

*Last updated: Feb 2025*  
*Maintained by: Sukumal*
