# Acceptance Criteria: Merchant Onboarding Flow

**AC-001** — OTP Registration  
**Given** a new merchant enters a valid business email,  
**When** they request an OTP,  
**Then** the system sends an OTP to that email within 60 seconds,  
**And** the OTP expires after 10 minutes,  
**And** the system allows maximum 3 attempts before locking for 15 minutes.

---

**AC-002** — Document Upload Validation  
**Given** a merchant uploads an identity document,  
**When** the file is below minimum quality threshold (blurry/incomplete),  
**Then** the system rejects the file immediately with a specific error message,  
**And** prompts the merchant to re-upload with guidance tips.

---

**AC-003** — Duplicate Business Registration  
**Given** a merchant submits a business registration number,  
**When** that number already exists in the system,  
**Then** the system blocks submission and displays an error,  
**And** escalates a flag to the compliance team queue automatically.

---

**AC-004** — Auto-Approval  
**Given** a merchant's document quality score is above threshold (≥ 85),  
**And** the business category is not on the high-risk list,  
**When** the compliance pipeline processes the application,  
**Then** the account is activated automatically within 2 hours,  
**And** an activation email with API keys is sent immediately.

---

**AC-005** — Manual Review Routing  
**Given** a merchant's business category is flagged as high-risk,  
**When** they complete document submission,  
**Then** the application is routed to manual review regardless of document score,  
**And** the merchant receives an email stating review will take 2–3 business days.
