# Acceptance Criteria: Payment Retry Logic

### AC-001 — Hard Decline Block

**Given** a transaction is declined with reason code `43` (Stolen Card),  
**When** the decline classification engine processes the failure,  
**Then** the transaction is classified as `HARD_DECLINE`,  
**And** no retry is scheduled,  
**And** the merchant is notified with classification: "Card reported stolen — retry not permitted",  
**And** a `payment.retry.exhausted` webhook is fired immediately with reason `hard_decline`.

---

### AC-002 — Soft Decline Retry Scheduling

**Given** a transaction is declined with reason code `51` (Insufficient Funds),  
**When** the decline classification engine processes the failure,  
**Then** the transaction is classified as `SOFT_DECLINE`,  
**And** a retry is scheduled for 24 hours later (platform default),  
**And** a `payment.retry.scheduled` webhook is fired within 500ms,  
**And** the transaction status in the dashboard updates to `Retry Scheduled`.

---

### AC-003 — Network Timeout Immediate Retry

**Given** a transaction fails with reason code `91` (Issuer System Unavailable),  
**When** the decline classification engine processes the failure,  
**Then** the transaction is classified as `SOFT_DECLINE — TIMEOUT`,  
**And** a retry is executed immediately (within 60 seconds),  
**And** if the immediate retry succeeds, the transaction is marked `Succeeded — Recovered`,  
**And** the customer and merchant receive a success confirmation,  
**And** no further retries are scheduled.

---

### AC-004 — Card Network Retry Limit Enforcement

**Given** a merchant has configured a custom retry schedule of 20 attempts,  
**When** the merchant saves the configuration,  
**Then** the system rejects the configuration and displays: "Retry schedule exceeds Visa limit of 15 attempts per 30 days.",  
**And** the configuration is not saved,  
**And** the merchant is shown the compliant maximum for each card network.

**Given** a transaction has been retried 15 times within a 30-day period (Visa),  
**When** the 16th retry would be scheduled,  
**Then** the system blocks the retry,  
**And** marks the transaction as `Retry Exhausted — Network Limit Reached`,  
**And** fires a `payment.retry.exhausted` webhook with reason `network_limit_reached`.

---

### AC-005 — Idempotency on Retry Execution

**Given** a retry attempt is in progress for a transaction,  
**When** the retry engine receives a duplicate trigger for the same transaction and attempt number (e.g., due to system retry on timeout),  
**Then** the system executes the payment only once,  
**And** returns the existing attempt result without creating a duplicate charge,  
**And** logs the duplicate trigger event in the audit log.

---

### AC-006 — Retry Success Recovery

**Given** a soft-declined transaction has been retried,  
**When** the retry attempt succeeds,  
**Then** the transaction status updates to `Succeeded — Recovered`,  
**And** the recovered amount is included in the next settlement cycle,  
**And** a `payment.retry.succeeded` webhook is fired within 30 seconds,  
**And** all remaining scheduled retries for this transaction are cancelled,  
**And** if customer notification is enabled, a payment success email is sent to the cardholder.
