# PRD: Payment Retry Logic

**Version:** 1.0  
**Status:** Draft  
**Author:** Sukumal  
**Last Updated:** 23-Feb-2025  
**PRD Owner:** Product — Core Payments  
**Stakeholders:** Engineering, Data/Analytics, Merchant Success, Finance, Risk

**Demo**[Interative Wireframe](https://payment-retry-wireframe.vercel.app/)

<img width="542" height="614" alt="Image" src="https://github.com/user-attachments/assets/1de1ae98-ed44-4078-850a-b1efde657206" /><img width="509" height="723" alt="Image" src="https://github.com/user-attachments/assets/6045a4b6-3dc0-49b0-be4e-f69ba20a39f7" />

---

## Table of Contents

1. [Overview & Problem Statement](#1-overview--problem-statement)
2. [Goals & Success Metrics](#2-goals--success-metrics)
3. [User Personas](#3-user-personas)
4. [User Stories](#4-user-stories)
5. [Functional Requirements](#5-functional-requirements)
6. [Non-Functional Requirements](#6-non-functional-requirements)
7. [Acceptance Criteria](#7-acceptance-criteria)
8. [Edge Cases & Out of Scope](#8-edge-cases--out-of-scope)

---

## 1. Overview & Problem Statement

### Background

A payment failure occurs when a transaction cannot be completed at the point of processing. Failures fall into two categories: **hard declines** (permanent failures such as stolen cards or closed accounts) and **soft declines** (temporary failures such as insufficient funds, network timeouts, or issuer system downtime). Soft declines represent a recoverable revenue opportunity — if retried at the right time, a meaningful percentage will succeed.

Currently, the platform returns all failed transactions to merchants without any automated retry logic. Merchants either implement their own retry mechanisms (inconsistently) or lose the revenue entirely. There is no intelligence applied to retry timing, frequency, or decision-making based on failure reason codes.

### Problem

- An estimated 15–25% of soft-declined transactions could succeed on retry, representing significant unrecovered revenue for merchants
- Without standardised retry logic, some merchants retry immediately and repeatedly, which increases the risk of card network penalties for excessive retries (Visa and Mastercard impose fees after 15 retry attempts on a declined transaction within 30 days)
- No differentiation between hard and soft declines means merchants waste retry attempts on permanently failed transactions
- No visibility into retry attempt history makes debugging payment failures time-consuming for both merchants and the support team

### Opportunity

Build an intelligent, configurable Payment Retry Engine that automatically identifies retryable failures, applies smart timing logic based on decline reason codes, respects card network retry rules, and provides merchants with full visibility into retry attempt history — recovering estimated 20% of soft-declined revenue while keeping the platform compliant with Visa and Mastercard retry regulations.

---

## 2. Goals & Success Metrics

| Goal | Metric | Baseline | Target |
|------|--------|----------|--------|
| Recover soft-declined revenue | Retry success rate (soft declines) | N/A (no retry) | > 20% |
| Reduce hard decline retry waste | % hard declines retried | ~100% (no filter) | < 2% |
| Maintain network compliance | Retry attempts exceeding card network limits | Unknown | 0 violations |
| Merchant visibility | % merchants using retry dashboard | 0% | > 60% |
| Reduce support tickets | Payment failure support tickets/month | ~210 | < 80 |

---

## 3. User Personas

### Persona A — Subscription / Recurring Billing Merchant (Primary)
- SaaS company or subscription box service processing recurring charges
- High impact from soft declines: a failed renewal = churned subscriber
- Needs: automatic retry with configurable schedule, customer notification options, dunning management
- Pain point: no systematic way to recover failed renewals without building custom retry logic

### Persona B — E-commerce Merchant (Secondary)
- Online retailer processing one-time purchases
- Moderate impact: failed checkout = abandoned sale
- Needs: immediate retry on timeout errors, clear failure messaging to end customer
- Pain point: cannot distinguish between "customer has no funds" and "bank system was down"

### Persona C — Platform Developer / Technical Integration (Internal)
- Developer building or maintaining the merchant's payment integration
- Needs: clear retry status in API responses, webhook events for each retry attempt, ability to query retry history via API
- Pain point: no programmatic visibility into whether a retry has been scheduled or has already occurred

---

## 4. User Stories

See [`user-stories.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/03-payment-retry-logic/user-stories.md)

---

## 5. Functional Requirements

### FR-01: Decline Classification Engine

- System shall classify every failed transaction as either `HARD_DECLINE` or `SOFT_DECLINE` based on the issuer response code (see Decline Code Matrix below)
- System shall never schedule a retry for `HARD_DECLINE` transactions
- System shall log the classification reason for every failure decision
- System shall update classification in real time if card status changes between retry attempts (e.g., card reported stolen after first soft decline)

### Decline Code Classification Matrix

| Category | Codes | Classification | Retry? |
|----------|-------|---------------|--------|
| Insufficient Funds | 51 | Soft | ✅ Yes — retry after 24–72hrs |
| Do Not Honour (Generic) | 05 | Soft | ✅ Yes — retry after 24hrs |
| Network / Issuer Timeout | 91, 96 | Soft | ✅ Yes — retry immediately (< 60s) |
| Exceeds Limit | 61, 65 | Soft | ✅ Yes — retry after 48hrs |
| Stolen Card | 43 | Hard | ❌ Never |
| Lost Card | 41 | Hard | ❌ Never |
| Invalid Card Number | 14 | Hard | ❌ Never |
| Closed Account | 46 | Hard | ❌ Never |
| Fraudulent Transaction (Issuer) | 59 | Hard | ❌ Never |
| Card Expired | 54 | Hard | ❌ Notify customer to update |
| Restricted Card | 36, 62 | Hard | ❌ Never |

### FR-02: Retry Scheduling Engine

- System shall schedule retry attempts according to the retry strategy configured by the merchant or applied by platform default
- System shall enforce card network retry limits:
  - **Visa**: Maximum 15 retry attempts on a declined transaction within 30 days
  - **Mastercard**: Maximum 10 retry attempts within 14 days for most reason codes
- System shall apply a minimum interval of 24 hours between retry attempts (except for network timeout retries, which may be immediate)
- System shall stop all retries if maximum attempts are reached or if card status changes to `HARD_DECLINE` at any point

### Default Retry Schedule (Platform Default)

| Attempt | Timing from Initial Failure | Condition |
|---------|----------------------------|-----------|
| Attempt 1 | Immediate (if timeout) / 24hrs (if soft decline) | All soft declines |
| Attempt 2 | +72 hours | If Attempt 1 failed |
| Attempt 3 | +7 days | If Attempt 2 failed |
| Attempt 4 | +14 days | Subscription merchants only |
| Stop | After Attempt 4 | Or card network limit reached |

### FR-03: Merchant Configuration

- System shall allow merchants to configure a custom retry schedule (timing and number of attempts) within card network compliance limits
- System shall allow merchants to enable or disable automatic customer notification emails per retry event
- System shall allow merchants to set a `hard stop` rule: e.g., "stop retrying if total outstanding amount exceeds 3,000 THB"
- System shall validate merchant configuration against card network limits before saving — reject configurations that would violate limits

### FR-04: Retry Execution & Webhooks

- System shall execute retry attempts via the same payment processor and method as the original transaction
- System shall fire a webhook event for every retry event:
  - `payment.retry.scheduled` — when a retry is queued
  - `payment.retry.attempted` — when a retry is executed (includes attempt number and outcome)
  - `payment.retry.succeeded` — when a retry succeeds (includes recovered amount)
  - `payment.retry.exhausted` — when all retry attempts are used without success
- System shall update transaction status in merchant dashboard in real time

### FR-05: Retry Dashboard & Reporting

- System shall provide a Retry Activity dashboard showing: total retried transactions, success rate, recovered revenue, top failure reason codes
- System shall allow merchant to view full retry history for any transaction (filterable by date, status, amount, card type)
- System shall generate a weekly automated retry summary report delivered by email
- System shall allow merchants to manually trigger an immediate retry on an eligible soft-declined transaction from the dashboard

---

## 6. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Retry scheduling must complete within 500ms of failure event receipt |
| Reliability | Retry engine must maintain 99.99% uptime — missed retries = lost revenue |
| Idempotency | Each retry attempt must be idempotent — system must never double-charge |
| Compliance | Must comply with Visa Transaction Retry Programme (TRP) and Mastercard Merchant Advice Code (MAC) rules |
| Auditability | All retry decisions (scheduled, skipped, stopped) must be logged with timestamp and reason |
| Webhook Delivery | Webhook events must be delivered within 30 seconds of retry event with at-least-once delivery guarantee |
| Security | Retry engine must not store full card numbers — use tokenised card references only |
| Scalability | Engine must handle 10,000 concurrent retry jobs without degradation |

---

## 7. Acceptance Criteria

See [`acceptance-criteria.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/03-payment-retry-logic/acceptance-criteria.md)

---

## 8. Edge Cases & Out of Scope

### Edge Cases

| Scenario | Expected Behaviour |
|----------|-------------------|
| Card updated (new number) mid-retry cycle | System receives account updater signal. Update tokenised card reference. Retry with new card details. Log card update event. |
| Merchant suspends subscription during retry cycle | Cancel all pending retries for that subscription's transactions. Log cancellation reason as `merchant_cancelled`. Do not charge on suspended accounts. |
| Partial authorisation returned on retry | If issuer approves only part of the transaction amount, reject partial auth and mark as failed. Do not process partial charges without merchant opt-in to partial auth feature. |
| Duplicate transaction detected (same card, same amount, within 5 minutes) | Block transaction as potential duplicate. Alert merchant. Do not execute retry. Require manual confirmation. |
| Merchant's balance insufficient to absorb a chargeback on recovered transaction | Flag merchant account. Proceed with recovery. Add note to compliance queue for review. |
| Retry succeeds but webhook delivery fails | System retries webhook delivery with exponential backoff: 30s, 2min, 10min, 1hr, 24hrs. If all webhook retries fail, log failure and notify merchant via email. |
| Card network changes retry rules mid-schedule | System applies new rules to all future retries. Existing scheduled retries within old rules are honoured if the attempt date is before the rule change date. |

### Out of Scope (v1.0)

- Alternative payment method fallback on card decline (e.g., retry with PromptPay if card fails) — planned v1.3
- AI-powered optimal retry timing prediction based on merchant-specific historical data — planned v2.0
- Real-time account updater integration for automatic card detail refresh — planned v1.2
- Customer self-service payment update portal (dunning management UI for end customers) — planned v1.1
- Multi-currency retry logic (non-THB/USD) — planned v1.3
- 3DS re-authentication on retry for SCA-required transactions — planned v1.2

---

## Appendix A: Retry State Machine

```
[Transaction Failed]
        │
        ▼
[Classify: Hard or Soft?]
        │
   ┌────┴────┐
   │         │
HARD       SOFT
   │         │
   ▼         ▼
[Block]   [Schedule Retry]
[Notify]       │
               ▼
          [Execute Retry] ◄──── (repeat per schedule)
               │
         ┌─────┴─────┐
         │           │
      SUCCESS      FAILED
         │           │
         ▼           ▼
   [Recover]   [Max attempts?]
   [Settle]         │
              ┌─────┴─────┐
              │           │
             YES          NO
              │           │
              ▼           ▼
         [Exhausted]  [Reschedule]
         [Notify]
```

## Appendix B: Webhook Payload Reference

### `payment.retry.scheduled`
```json
{
  "event": "payment.retry.scheduled",
  "transaction_id": "txn_abc123",
  "attempt_number": 2,
  "scheduled_at": "2025-01-14T10:00:00Z",
  "decline_code": "51",
  "classification": "SOFT_DECLINE",
  "retry_reason": "insufficient_funds"
}
```

### `payment.retry.succeeded`
```json
{
  "event": "payment.retry.succeeded",
  "transaction_id": "txn_abc123",
  "attempt_number": 2,
  "succeeded_at": "2025-01-14T10:00:04Z",
  "recovered_amount": 1500,
  "currency": "THB"
}
```

### `payment.retry.exhausted`
```json
{
  "event": "payment.retry.exhausted",
  "transaction_id": "txn_abc123",
  "total_attempts": 4,
  "exhausted_reason": "max_attempts_reached",
  "final_decline_code": "51",
  "total_amount_unrecovered": 1500,
  "currency": "THB"
}
```
