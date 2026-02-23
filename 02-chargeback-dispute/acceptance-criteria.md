# Acceptance Criteria: Chargeback Dispute Flow

### AC-001 — New Case Notification

**Given** the platform receives a chargeback notification from a card network,  
**When** the case is created in the system,  
**Then** the merchant receives an email notification within 30 minutes,  
**And** a case record appears in the merchant's dispute dashboard,  
**And** the response deadline is calculated and displayed correctly,  
**And** an in-app notification badge is shown.

---

### AC-002 — Evidence Submission Validation

**Given** a merchant uploads evidence files for a chargeback case,  
**When** the total file size exceeds 25MB,  
**Then** the system rejects the upload and displays an error: "Total evidence package exceeds 25MB limit. Please compress or remove files.",  
**And** no files are stored.

**When** all files are within limits,  
**Then** the system accepts the upload and shows a preview of the evidence package,  
**And** the merchant can add a rebuttal letter before confirming submission.

---

### AC-003 — Deadline Reminder Notifications

**Given:** an open chargeback case has a response deadline,  
**When:** the deadline is exactly 7 days away,  
**Then:** the system sends a reminder email to the merchant,  
**And:** when the deadline is 3 days away, another reminder is sent,  
**And:** when 24 hours remain, a final urgent reminder is sent with `[URGENT]` subject line prefix,  
**And:** if the case remains unresponded at 48 hours before deadline, it is escalated to the compliance queue.

---

### AC-004 — Evidence Package Generation

**Given:** a merchant confirms their evidence submission,  
**When:** the system generates the evidence package,  
**Then:** all uploaded files are combined into a single, ordered PDF bundle,  
**And:** the rebuttal letter is included as the first page,  
**And:** a submission confirmation email is sent with the case reference number,  
**And:** case status updates to `Evidence Submitted`.

---

### AC-005 — Case Resolution Notification

**Given:** a chargeback case reaches a final resolution (Won / Lost / Accepted),  
**When:** the card network returns a decision,  
**Then:** the merchant receives an email with the outcome and reason,  
**And:** the case status updates immediately in the dashboard,  
**And:** if Won, the disputed amount is reflected in the next settlement cycle,  
**And:** if Lost or Accepted, the chargeback amount is deducted from the merchant's balance.

---
