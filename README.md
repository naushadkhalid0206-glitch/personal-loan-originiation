# Agentic AI for Personal Loan Origination & Credit Underwriting

**Version:** 1.0  
**Target Users:** Loan Applicants, Credit/Risk Analysts, Underwriting Managers  
**Decision Output:** *Approve / Reject / Refer (Human Review)* recommendation (no disbursal, no policy overrides)  
**Region Assumption:** India (₹, RBI-regulated entities / partner NBFCs)

---

## 1) Executive Summary

Retail personal-loan underwriting is often manual, multi-step, and fragmented across documents, KYC checks, bureau pulls, income assessment, and policy rules. This causes long turnarounds, inconsistent checks, operational risk, and poor customer transparency.

This PRD defines an **agentic AI underwriting copilot** that:
- Collects and validates applicant inputs and documents
- Triggers KYC/ID and credit bureau checks
- Computes eligibility using configurable bank/NBFC policy rules
- Produces an **explainable recommendation** with reason codes and evidence links
- Routes exceptions, borderline decisions, and fraud suspicions to humans

It is designed to optimize for:
- **Decision speed**
- **Explainability and auditability**
- **Regulatory and policy compliance**
- **Operational consistency** (reduced human error)

---

## 2) Problem Context

Retail banks/NBFCs process personal loans through manual, multi-step verification. Credit and risk analysts gather documents, validate identity/KYC, check bureau scores, assess income stability, and apply policy rules using siloed tools (spreadsheets, emails, legacy portals). This is slow (hours/days), error-prone (missed red flags, inconsistent checks), and can introduce bias. Customers experience long approval times and low transparency on why a loan was approved/rejected.

---

## 3) Primary Goal

**The agent’s goal is to issue a loan approve/reject recommendation, subject to KYC/ID verification, credit bureau checks, and bank policy rules, while optimizing for decision speed, explainability, and compliance.**

---

## 4) Scope & Boundaries

### In Scope (Allowed)
- Application intake (conversational + form-based)
- Document collection, parsing, validation, and mismatch detection
- KYC/CIP initiation & verification through external tools
- Credit bureau pull and interpretation
- Income verification (payslip/bank statement checks; employer verification workflow hooks)
- Eligibility computation (DTI/FOIR, credit score thresholds, existing obligations, stability checks)
- Policy rules evaluation (hard rules + configurable thresholds)
- Fraud/anomaly flagging (rule-based + signals)
- Generating rationale: reason codes, evidence, audit trail
- Human routing: exceptions, missing docs, borderline cases
- Communication: next steps, document requests, decision explanation summary

### Must Stop / Ask Human (Human-in-the-loop)
- Borderline approvals/rejections (near thresholds)
- Fraud suspicion (identity mismatch, forged docs signals, synthetic identity indicators)
- Policy overrides / exception approvals (e.g., “approve despite score < threshold”)
- Any regulatory ambiguity, adverse media/manual review requirements, or high-risk segments
- Unrecognized scenarios not covered by rules/policy config

### Out of Scope (Explicit)
- Loan disbursal/payment execution
- Overriding regulatory/policy limits
- Collections/recovery operations
- Pricing optimization beyond policy (unless explicitly defined later)

---

## 5) Users & Personas

### 5.1 Loan Applicant
**Needs**
- Fast decision or clear next steps
- Transparent explanation of outcomes
- Simple document upload + status tracking

**Key Moments**
- Initial application
- Responding to doc requests
- Understanding decision / reapply guidance

### 5.2 Credit Analyst / Risk Analyst
**Needs**
- Standardized checklist execution
- Clear mismatch/fraud signals
- Evidence-backed recommendation
- Minimal manual data entry
- Audit-ready notes automatically generated

### 5.3 Underwriting Manager
**Needs**
- Exception queue with clear justification and risk impact
- Policy override workflow with approvals + audit logs
- Visibility into operational metrics and risk outcomes

---

## 6) Product Overview & Key Principles

### 6.1 Product Modules
1. **Conversational Intake + Case Builder**
2. **Document Intelligence & Validation**
3. **KYC/CIP Orchestration**
4. **Credit Bureau Orchestration**
5. **Income & Obligation Assessment**
6. **Policy Rule Engine (Hard/Soft Rules)**
7. **Risk Assessment Layer (Scorecard/Model Optional)**
8. **Decision Composer (Approve/Reject/Refer)**
9. **Explainability & Reason Codes**
10. **Human Review & Exception Routing**
11. **Audit Logging & Compliance Reporting**
12. **Admin Policy Studio (configurable thresholds, rules, and templates)**

### 6.2 Key Principles
- **Deterministic decisioning:** final recommendation must be computed from rules/models; LLM can orchestrate + explain, but must not “invent” policy.
- **Evidence-first:** every key conclusion must link to data evidence (document field, KYC response, bureau attribute, computed ratio).
- **Privacy & security by design:** purpose limitation, consent, minimal data collection, role-based access.
- **Human oversight:** escalate at defined gates; preserve accountability and approval authority.
- **Fairness guardrails:** avoid using prohibited/sensitive attributes; monitor bias via outcomes and drift.

---

## 7) Success Metrics (KPIs)

### Speed & Efficiency
- Median time to recommendation (target: minutes, not hours)
- % of applications auto-processed without human touch (straight-through processing rate)
- Analyst time per case (target reduction)

### Quality & Risk
- Decision consistency score (variance across analysts decreases)
- Fraud catch-rate proxy metrics (flags validated by humans)
- Early delinquency proxy (if later integrated with performance data)
- Policy breach rate (target near zero)

### Customer Experience
- Applicant drop-off rate during doc upload
- Applicant understanding score (CSAT / comprehension survey)
- Re-application conversion (post-reject guidance effectiveness)

### Compliance & Audit
- % cases with complete audit trail (target 100%)
- Explainability completeness (reason codes present for all declines/refers)
- Data-access audit anomalies (target 0)

---

## 8) End-to-End User Journeys

### 8.1 Applicant Journey (Happy Path)
1. Applicant: “Apply for ₹3L personal loan.”
2. Agent creates case, collects basic demographics + employment + income + consent.
3. Agent requests documents (ID, address proof, income proof, bank statement).
4. Agent performs KYC/CIP verification and bureau pull.
5. Agent computes eligibility and policy decision.
6. Agent returns: **Approve/Reject** recommendation + reasons + next steps.

### 8.2 Applicant Journey (Doc Mismatch → Human Review)
1. Agent detects income mismatch (declared vs payslip/bank statement).
2. Agent asks applicant for clarification or additional proof (latest payslip, employer letter).
3. If mismatch persists: agent escalates to analyst with a summarized discrepancy report.
4. Analyst confirms/updates. Agent re-runs checks and finalizes recommendation.

### 8.3 Analyst Journey (Exception Queue)
1. Analyst opens “Refer” queue.
2. Sees:
   - Policy hits (soft/hard)
   - Bureau highlights (DPD, inquiries, utilization)
   - Income and FOIR summary
   - Fraud flags
   - Evidence snippets + source links
3. Analyst chooses:
   - Approve recommendation (send to manager if exception)
   - Reject recommendation
   - Request more docs
4. Manager approves exceptions and signs off.

---

## 9) Functional Requirements (FR)

### FR-1: Application Intake & Case Creation
- Support conversational intake + structured form mode.
- Create unique Application ID and Case Workspace.
- Capture consent checkpoints (KYC, bureau pull, data usage, third-party sharing) and store audit trail.
- Validate mandatory fields (PAN/ID, DOB, phone, address, employment type, income).

**Acceptance Criteria**
- Application can be created with minimal viable fields.
- Missing data triggers a structured request list.

---

### FR-2: Document Collection & Validation
- Support uploads: PDF/JPG/PNG; optional email forwarding.
- Document types:
  - Identity (PAN/Aadhaar/Passport/Driving License)
  - Address proof
  - Payslip (last 3 months)
  - Bank statements (last 6 months)
  - Employment proof (optional)
- Extract key fields:
  - Name, DOB, address, ID number
  - Employer name, salary credits, net pay
- Validate:
  - Cross-document name match
  - DOB match
  - Address match
  - Tamper heuristics (basic: metadata anomalies, blur, inconsistent fonts; advanced optional)

**Acceptance Criteria**
- Agent produces a “Doc Validation Report” with match/mismatch flags and confidence.
- All extracted fields stored with source references (page/region) for audit.

---

### FR-3: KYC/CIP Orchestration
- Trigger KYC via integrated vendor(s) or internal CKYC system.
- Handle statuses:
  - Verified
  - Pending
  - Failed
  - Needs manual review
- Maintain a KYC checklist aligned to RBI KYC direction expectations (CIP/CDD workflow). :contentReference[oaicite:0]{index=0}
- Any KYC failure routes to human review or reject based on policy.

**Acceptance Criteria**
- KYC must complete (or be explicitly waived by policy) before final recommendation.

---

### FR-4: Credit Bureau Pull & Interpretation
- Integrate with credit bureau(s) as configured.
- Parse bureau report into normalized attributes:
  - Score
  - Active tradelines, total obligations
  - DPD history, write-offs/settlements
  - Inquiries (recent)
  - Utilization (if available)
- Provide reason codes aligned to bureau + internal policy.

Also ensure bureau use and confidentiality controls align with RBI’s credit information reporting framework expectations (confidentiality, grievance, and consumer correction mechanisms). :contentReference[oaicite:1]{index=1}

**Acceptance Criteria**
- Bureau report pull is logged with timestamp, consent record, and request/response IDs.
- Applicant-facing explanation never reveals raw bureau data beyond permitted summarizations.

---

### FR-5: Income & Stability Assessment
Compute:
- Monthly net income (payslip + bank salary credits reconciliation)
- Income stability score:
  - Salary regularity
  - Employer continuity (if data exists)
  - Large unexplained cash credits (flag)
- FOIR/DTI:
  - Existing EMIs from bureau + bank statement parsing
  - Proposed EMI estimate (policy-configured pricing bands)

**Acceptance Criteria**
- Agent produces a “Financial Summary” object: income, obligations, FOIR, buffers, anomalies.

---

### FR-6: Policy Rule Engine (Deterministic)
Support configurable rules:
- **Hard Declines** (auto reject):
  - KYC failed
  - Bureau score below minimum threshold
  - Recent severe delinquency (e.g., 90+ DPD in last X months)
  - High FOIR above maximum
  - Fraud flags above threshold
- **Soft Rules** (refer/human):
  - Score in grey zone
  - Thin file / no hit
  - Minor doc mismatch
  - Recent multiple inquiries
- **Eligibility Rules**:
  - Age min/max
  - Income minimum by employment type
  - Tenure limits
  - Loan amount caps by risk band
  - Employer category constraints (optional)

**Acceptance Criteria**
- Every decision includes:
  - Decision outcome
  - Triggered rules
  - Computed values vs thresholds
  - Evidence pointers

---

### FR-7: Risk Assessment Layer (Optional but Recommended)
Two-stage approach:
1. **Rule Engine** (mandatory) → ensures compliance and deterministic policy application
2. **Risk Model/Scorecard** (optional) → rank-order risk within policy-eligible population

Constraints:
- Model must be interpretable enough to support adverse-action reasons (e.g., reason codes derived from top drivers).
- Model must not use prohibited sensitive features.

**Acceptance Criteria**
- Model output cannot override a hard decline.
- Model output can move “Approve” to “Refer” if uncertainty is high.

---

### FR-8: Decision Composer (Approve / Reject / Refer)
Decision must include:
- Primary reason codes (top 3–5)
- Secondary contributing factors
- Required next steps:
  - If Refer: what human needs to validate
  - If Reject: what applicant can do (e.g., correct KYC, provide updated income proof, reduce obligations)

**Acceptance Criteria**
- Decision payload is structured + human-readable.
- Applicant response is simpler than analyst response.

---

### FR-9: Explainability & Customer Transparency
Generate:
- **Applicant explanation**
  - Plain-language reasons (no internal policy thresholds exposed unless allowed)
  - Next steps
- **Analyst explanation**
  - Full details: triggered rules, computations, evidence
- Provide “Why was I rejected?” standardized templates.

Also incorporate digital lending disclosure expectations such as providing key fact statements and clear borrower communications for digital journeys. :contentReference[oaicite:2]{index=2}

**Acceptance Criteria**
- For every reject/refer, at least one “actionable” reason must be provided when permissible.

---

### FR-10: Human-in-the-Loop Workflow
Queues:
- Borderline (score/FOIR near threshold)
- Suspected fraud
- Doc mismatch unresolved
- Policy exception requested
- System errors / integration failures

Actions:
- Approve recommendation
- Reject recommendation
- Request more info
- Override (manager only; requires mandatory notes and reason)

**Acceptance Criteria**
- Every manual decision logs: who, when, why, evidence reviewed.

---

### FR-11: Grievance & Audit Support (Digital Lending)
For digital loans:
- Capture and display grievance redressal contact points (RE + any LSP interface) consistent with RBI digital lending requirements. :contentReference[oaicite:3]{index=3}
- Maintain complaint references in case file (if integrated later).

**Acceptance Criteria**
- Case record stores grievance officer details and versioned policy used.

---

### FR-12: Data Consent, Minimization, and Storage Controls
From RBI digital lending directions:
- Data collection must be **need-based** with **explicit consent** and audit trail; avoid unnecessary access to device resources. :contentReference[oaicite:4]{index=4}
- Implement consent revocation and data deletion workflows.
- Define storage location controls (India data residency requirements in the RBI directions). :contentReference[oaicite:5]{index=5}

From India’s DPDP Act:
- Processing must be for lawful purposes with appropriate obligations on data fiduciaries; system must support notice/consent and retention controls. :contentReference[oaicite:6]{index=6}

**Acceptance Criteria**
- Every external pull (KYC, bureau, bank statements) requires recorded consent artifact.
- Data retention and deletion requests are operationally supported.

---

## 10) AI / Agentic System Requirements

### 10.1 Agent Roles (Conceptual)
1. **Orchestrator Agent (State Manager)**
   - Controls workflow state transitions
   - Enforces boundaries (when to stop/ask human)

2. **Doc Intelligence Agent**
   - Extracts structured fields and performs doc cross-checks
   - Produces doc validation report with evidence pointers

3. **KYC Agent**
   - Calls KYC tools, interprets results, decides next steps

4. **Bureau Agent**
   - Calls bureau tools, parses report to normalized schema
   - Derives bureau reason codes

5. **Income & Obligations Agent**
   - Computes income stability and FOIR/DTI
   - Detects anomalies

6. **Policy Engine (Non-LLM / Deterministic Service)**
   - Applies rules and returns triggered rule set + decision constraints

7. **Decision Explainer Agent**
   - Converts structured reasons into applicant/analyst narratives
   - Must be constrained to provided facts (no hallucination)

8. **Escalation Router**
   - Builds referral packet for analysts/managers

### 10.2 Guardrails (Non-negotiable)
- LLM agents may *not*:
  - Approve loans beyond policy
  - Modify thresholds
  - Fabricate verification results
  - Exfiltrate or reveal sensitive raw bureau data
- Any uncertainty → **Refer**
- All steps must write to an immutable audit log.

### 10.3 Memory & State
- Use **case-scoped memory** only (no cross-customer mixing).
- Store only:
  - Structured extracted fields
  - Tool responses (redacted where needed)
  - Reason codes and computations
- Never store secrets in prompts or logs.

---

## 11) Data Model (High-Level)

### 11.1 Core Entities
- **Applicant**
- **Application**
- **Documents**
- **Verifications** (KYC, bureau, bank statements)
- **FinancialSummary**
- **PolicyEvaluation**
- **DecisionRecommendation**
- **AuditLogEvent**
- **HumanReviewTask**

### 11.2 Decision Output Schema (Concept)
- decision: APPROVE | REJECT | REFER
- recommended_amount
- tenure_months
- apr_band (if applicable)
- primary_reason_codes[]
- secondary_factors[]
- evidence_refs[]
- required_actions[]
- policy_version_id
- confidence_level (LOW/MED/HIGH) – operational confidence, not “model confidence”

---

## 12) Integrations

### 12.1 Required External Tools (via adapters)
- KYC/ID verification provider(s)
- Credit bureau provider(s)
- Bank statement analysis / Account Aggregator (optional)
- Internal CRM/Loan Origination System (LOS)
- Document storage (secure)
- Notification (SMS/email/WhatsApp via approved vendors)

### 12.2 Integration Reliability Requirements
- Retries with idempotency keys
- Timeouts and circuit breakers
- “Degraded mode”:
  - If bureau down → Refer (not approve)
  - If KYC pending → Pending/Refer

---

## 13) Compliance & Regulatory Requirements (Productized)

### 13.1 RBI KYC Compliance Support
- CIP/CDD workflow support and recordkeeping aligned to RBI KYC master direction structure. :contentReference[oaicite:7]{index=7}

### 13.2 RBI Digital Lending Requirements Support (If digital channel)
- Borrower creditworthiness assessment must capture minimum borrower economic profile info and retain for audit. :contentReference[oaicite:8]{index=8}
- Provide key disclosures (KFS) and ensure borrower communications flow digitally after execution. :contentReference[oaicite:9]{index=9}
- Data collection must be consented, need-based; avoid invasive device permissions. :contentReference[oaicite:10]{index=10}
- Implement grievance redressal contact points. :contentReference[oaicite:11]{index=11}
- Ensure disbursal/repayment constraints are respected *at the system boundary* (even if disbursal is out of scope, the system must not design flows that contradict these requirements). :contentReference[oaicite:12]{index=12}

(Reference: RBI Digital Lending Directions, 2025.) :contentReference[oaicite:13]{index=13}

### 13.3 Credit Information Reporting & Handling
- Use bureau data with confidentiality, consumer correction pathways, and secure handling consistent with RBI’s Credit Information Reporting Directions framework intent. :contentReference[oaicite:14]{index=14}

### 13.4 DPDP Act (Privacy)
- Consent/notice support, purpose limitation, retention controls, and security controls. :contentReference[oaicite:15]{index=15}

---

## 14) Non-Functional Requirements (NFR)

### Performance
- P95 time to recommendation (no-human-touch): target < 5–10 minutes depending on external tool latency
- Document parsing: target < 60 seconds per document set (async)

### Reliability
- 99.5%+ service uptime for core workflow engine
- External dependency failures must degrade to “Refer” not “Approve”

### Security
- Encryption in transit + at rest
- Strict RBAC and least privilege
- PII redaction in logs
- Secure secret management (KMS/Vault)
- Prompt injection defenses (tool outputs treated as untrusted)

### Auditability
- Immutable audit log for:
  - Data access
  - Tool calls
  - Rule evaluations
  - Human decisions
  - Final outcomes

---
## 15) Repository Structure

This repository implements an agentic AI system for personal loan origination and credit underwriting.  
It follows a structured **Perceive → Plan → Act** workflow, enforced by deterministic policy rules, compliance guardrails, and human-in-the-loop escalation.

### High-Level Layout

- `prompts/` — Versioned prompt artifacts for Perceive/Plan/Act and explanation templates.
- `agents/` — Markdown documentation defining each specialized agent role.
- `memory/` — Canonical application state schema and sample state/audit logs.
- `configs/` — Environment settings, underwriting policy thresholds, reason codes, and routing rules.
- `src/plo_agent/` — Python package containing the runnable agent graph, agents, policy engine, tools, and APIs.
- `docs/` — Architecture, PRD, agent graphs, policy configuration guidance, compliance/audit docs.
- `tests/` — Unit/integration/end-to-end tests with fixtures (synthetic and non-sensitive).
  
### Tree

```text
personal-loan-origination-agent/
├── prompts/                # Perceive/Plan/Act prompt templates + explainer prompts
├── agents/                 # Agent role definitions (documentation)
├── memory/                 # State schema + sample application/audit logs
├── configs/                # Policy YAMLs, reason codes, routing rules, env configs
├── docs/                   # PRD, architecture, agent graph, compliance, runbooks
├── src/plo_agent/          # Python implementation (agents, graphs, tools, policy engine)
├── tests/                  # Unit/integration/e2e tests + fixtures
└── scripts/                # Developer utilities (seed demo cases, export audit reports)
```


## 16) Analytics, Monitoring & Model Governance

### Operational Dashboards
- Funnel: started → docs complete → KYC complete → bureau complete → decision
- Queue health: refer backlog, SLA breaches
- Top reject reasons and doc mismatch rates

### Risk Monitoring
- Drift monitoring: score distribution changes
- Policy override frequency
- Fraud flag true-positive validation rate

### Governance Artifacts
- Policy versioning
- Model card (if ML added)
- Release notes and change approvals

---

## Appendix A: Reason Code Taxonomy (Starter)

### Identity/KYC
- KYC_FAILED
- NAME_MISMATCH
- DOB_MISMATCH
- ADDRESS_MISMATCH
- KYC_PENDING_MANUAL_REVIEW

### Bureau
- SCORE_BELOW_MIN
- RECENT_HIGH_DPD
- WRITE_OFF_OR_SETTLEMENT
- EXCESSIVE_RECENT_INQUIRIES
- HIGH_UTILIZATION

### Income/Banking
- INCOME_BELOW_MIN
- INCOME_UNSTABLE
- SALARY_CREDIT_NOT_FOUND
- HIGH_CASH_CREDITS_UNEXPLAINED

### Policy
- AGE_OUT_OF_RANGE
- FOIR_ABOVE_MAX
- LOAN_AMOUNT_ABOVE_ELIGIBLE
- EMPLOYMENT_TYPE_OUT_OF_SCOPE

### Fraud/Anomaly
- DOCUMENT_TAMPER_SUSPECTED
- DEVICE_BEHAVIOR_ANOMALY (if used)
- SYNDICATE_PATTERN_MATCH (future)

---

## Appendix B: Human Review Packet Contents
- Applicant summary (structured)
- Doc validation report (mismatches + evidence links)
- KYC status + response reference IDs
- Bureau highlights + normalized attributes
- Financial summary (income, obligations, FOIR)
- Triggered policy rules + thresholds
- Proposed decision and confidence gate
- Questions for reviewer (explicit checklist)

