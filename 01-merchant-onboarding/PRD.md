# PRD: Merchant Onboarding Flow
**Version:** 1.0  
**Status:** Draft  
**Author:** Sukumal  
**Last Updated:** 23-Feb-2025

**Demo** [**Interactive Wireframe**](https://merchant-onboarding-wirframe.vercel.app/)

<img width="431" height="677" alt="Image" src="https://github.com/user-attachments/assets/f1140101-e548-4508-b53a-8d21c168d1b7" /> <img width="423" height="455" alt="Image" src="https://github.com/user-attachments/assets/4ada45ef-61c6-485d-b201-b9e7e628820c" />

---

## 1. Overview & Problem Statement

### Background
Merchants applying to join the payment platform currently complete 
onboarding through a fragmented, manual process involving email 
submissions, PDF forms, and phone verification. Average time-to-activation 
is 5–7 business days.

### Problem
- High drop-off rate (est. 40%) during document submission step
- Support team spends ~30% of tickets handling onboarding status queries
- No real-time status visibility for merchants

### Opportunity
Build a self-serve, digitised onboarding flow that reduces 
time-to-activation to under 24 hours and cuts onboarding-related 
support tickets by 50%.

---

## 2. Goals & Success Metrics

| Goal | Metric | Target |
|------|--------|--------|
| Faster activation | Time-to-activation | < 24 hours |
| Reduce drop-off | Onboarding completion rate | > 80% |
| Reduce support load | Onboarding-related tickets | ↓ 50% |
| Improve satisfaction | Merchant onboarding CSAT | > 4.2 / 5.0 |

---

## 3. User Personas

**Persona A — Small Business Owner (Primary)**
- Thai SME owner, 35–50 years old
- Low-to-medium technical literacy
- Wants to accept online payments quickly
- Pain point: confused by KYC document requirements

**Persona B — Enterprise Merchant (Secondary)**
- Finance/ops manager at mid-size company
- Needs multi-user access and audit trail
- Pain point: cannot delegate onboarding steps to team members

---

## 4. User Stories

See [`user-stories.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/01-merchant-onboarding/user-stories.md)

---

## 5. Functional Requirements

### FR-01: Registration
- System shall allow merchant to register using business email
- System shall send OTP verification within 60 seconds
- System shall support Thai ID card and passport as identity documents

### FR-02: Document Upload
- System shall accept JPG, PNG, PDF formats (max 10MB per file)
- System shall provide real-time upload progress indicator
- System shall auto-detect and flag blurry or incomplete documents 
  using image validation

### FR-03: KYC Review Pipeline
- System shall route submitted documents to compliance review queue
- System shall auto-approve low-risk merchant profiles 
  (based on business type + document quality score)
- System shall notify merchant via email + in-app at each status change

### FR-04: Activation
- System shall generate API keys upon approval
- System shall send onboarding completion email with 
  quick-start guide link
- System shall track first transaction within 7 days post-activation

---

## 6. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Document upload < 3 seconds on 4G mobile |
| Availability | 99.9% uptime during onboarding flow |
| Security | All documents encrypted at rest (AES-256) |
| Compliance | Must meet Bank of Thailand e-KYC guidelines |
| Accessibility | WCAG 2.1 AA compliant |
| Localisation | Thai and English language support |

---

## 7. Acceptance Criteria

See [`acceptance-criteria.md`](https://github.com/DustFlame0011/prd-fintech-payment-library/blob/main/01-merchant-onboarding/acceptance-criteria.md)

---

## 8. Edge Cases & Out of Scope

### Edge Cases
- Merchant submits expired ID document → system flags and prompts reupload
- Upload interrupted mid-way → system saves draft and resumes from last step
- Duplicate business registration number submitted → system blocks and alerts compliance team
- Merchant in high-risk business category → routes to manual review regardless of document score

### Out of Scope (v1.0)
- Multi-user account delegation (planned for v1.1)
- Automated video KYC
- Integration with Revenue Department API for business verification
