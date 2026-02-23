# User Stories: Payment Retry Logic

### Retry Configuration

**US-001**  
As a **subscription merchant**, I want to configure an automatic retry schedule for failed recurring payments (e.g., retry on day 1, day 3, day 7), so that I can recover failed renewals without manual intervention.

**US-002**  
As a **merchant**, I want the system to automatically distinguish between hard declines and soft declines, so that I do not waste retry attempts on permanently failed transactions.

**US-003**  
As a **merchant**, I want to set a maximum number of retry attempts per transaction, so that I stay within card network retry limits and avoid penalty fees.

---

### Retry Execution

**US-004**  
As a **merchant**, I want the system to apply intelligent retry timing based on the specific decline reason code (e.g., retry insufficient funds at end of month when customers are likely paid), so that retry success rates are maximised.

**US-005**  
As an **e-commerce merchant**, I want network timeout failures to be retried immediately (within 60 seconds), so that customers do not experience a failed checkout due to a temporary network issue.

**US-006**  
As a **merchant**, I want the system to stop all retry attempts if a card is reported as stolen, cancelled, or permanently blocked, so that I do not trigger fraud alerts by retrying hard declines.

---

### Visibility & Reporting

**US-007**  
As a **merchant**, I want to see the complete retry history for any failed transaction — including attempt timestamps, decline codes, and outcomes — so that I can diagnose persistent failures and make decisions about the account.

**US-008**  
As a **merchant**, I want to receive a weekly summary report of retry activity showing: total retried, success rate, recovered revenue, and top failure reasons, so that I can monitor the health of my payment recovery performance.

**US-009**  
As a **developer**, I want to receive a webhook event for every retry attempt — including the attempt number, outcome, and next scheduled retry — so that I can update order status in my system in real time.

---

### Customer Communication

**US-010**  
As a **subscription merchant**, I want the system to automatically send a payment failure notification email to the end customer (customisable template) after the first failed attempt, so that customers can update their payment details proactively.

---
