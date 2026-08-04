# Engineering Briefcase — AI-Powered Resume Screening & Recruitment Automation (OP-0002)

<!-- Generated on client accept by /generate-briefcase; stored in Workspace/projects/PRJ-0001/. Packaging only — no engineering reasoning. Section ownership and boundaries live in knowledge/briefcase-specification.md; this file is the output shape. -->

## 1. Engagement Header
- **Client / Opportunity:** OP-0002 — AI-Powered Resume Screening & Recruitment Automation (Human Resources, Upwork)
- **Date:** 2026-08-03 · **Status:** Accepted
- **Source artifacts:** `01-assessment/decision-package.md`, `02-proposal/01-proposal.md` (accepted), `02-proposal/00-winning-strategy.md` (Summary only), `00-input/job-post.md`

## 2. Executive Summary
An HR team of 11–50 employees processing 50–300 applicants/month wants their manual resume review replaced by an automated intake-and-screening workflow: form submission → PDF validation → Drive storage → content extraction → AI evaluation with score and recommendation → Sheets records → duplicate detection → candidate notification. $600 fixed, existing Google Workspace preserved. Well-enumerated scope with a genuine, volume-quantified value case in the practice's core stack — with two cautions the client hadn't raised: a recurring AI API cost, and the human-review gate functioning as a risk control rather than a convenience feature.

## 3. Business Context
- **Worth pursuing:** Qualified — n8n + Google Workspace + LLM integration is core practice capability; second opportunity in this exact stack.
- **Value validated:** Client-stated volume (50–300 applicants/month) and named failure mode ("time-consuming and inconsistent"); consistency of evaluation is a second, independent value driver alongside time saved.
- **Fit:** Strong — workflow orchestration, Google Workspace APIs, LLM integration, and document extraction are squarely in-capability. Named limitation: no delivered engagement track record yet at time of assessment.
- **Commercial:** Fixed price $600, client-stated; eleven objectives and five deliverables explicitly enumerated. Evaluation criteria were undefined at assessment time — the one genuine scope-creep exposure.
- **Strategic Value:** Portfolio Value High (first delivered engagement in this stack); Learning Value High (first build combining AI evaluation with workflow orchestration); Reusability Very High (pattern transfers to loan applications, vendor onboarding, grant review, claims intake); Revenue Potential Low-Medium; Relationship Potential Medium-High; Market Visibility Low-Medium.
- **Opportunity Status at assessment:** Posting age, competitive pressure, and availability confidence were all unknown — no marketplace signals were provided in the post.

## 4. Solution Context
**Overall Solution (identify-level):** An n8n-orchestrated recruitment intake and AI screening workflow that captures applications via an online form, validates and stores resumes in Google Drive, extracts and evaluates resume content with a language model to produce a score and recommendation, writes structured results to Google Sheets with duplicate detection, and notifies candidates by email.

**Solution Identification:**
- Core Stack — n8n (client-preferred; multi-step conditional orchestration across several services)
- Core Stack — LLM / language model API (evaluation producing score + recommendation; specific provider is CEF's engineering decision)
- Supporting — Google Drive API (storage), Google Sheets API (records), email delivery (likely Gmail, given the stated Workspace environment)
- Supporting — Online form with file upload (mechanism is a build-time choice)
- Supporting — PDF text extraction (straightforward for text-based PDFs, harder for scanned ones — a real unknown)
- Infrastructure — existing Google Workspace preserved; n8n instance hosting named, not configured

**Complexity:** High — eleven distinct objectives spanning intake, storage, AI reasoning, structured data recording, and notification. Real complexity drivers: undefined evaluation criteria and variable-quality PDF input, not the service count. **Rough Scale:** M.

**Conceptual Workflow:** Applicant Submits Form with PDF Resume → File Validation → Store in Google Drive → Extract Resume Content → AI Evaluation Against Criteria → Generate Score + Recommendation → Duplicate Check → Write Structured Record to Google Sheets → *[Human Review Gate — in-scope by commitment]* → Send Candidate Notification Email → Complete.

*(Identify-level only. Architecture, technical decomposition, and implementation planning are CEF — engineering decision.)*

## 5. Winning Strategy Summary
Positioned as AI that assists rather than replaces human hiring judgment. The value proposition is consistent screening at growing volume without ceding accountability. Promises made, implicit: existing Google Workspace stays untouched; evaluation criteria are adjustable without a rebuild; a human review step precedes hiring decisions; error handling covers concrete named failure modes. Client expectation created: a modular, documented, maintainable workflow the client can adjust themselves, plus awareness that AI evaluation carries a small ongoing per-resume cost.

## 6. Accepted Proposal & Scope
**In scope:** Form intake with PDF upload; file validation; Drive storage; resume content extraction; AI evaluation producing score + recommendation; structured Sheets records; duplicate detection; automated candidate email notifications; human review gate before any hiring decision; configurable evaluation criteria; documentation; deployment instructions; error handling on named failure modes (unreadable PDF, duplicate application, extraction failure); modular/extensible design.

**Out of scope:** The hiring decision itself; changes to the client's existing Google Workspace; a full applicant-tracking system; interview scheduling, offers, or onboarding; ongoing maintenance or support beyond delivery.

**Commercial terms (accepted):** Fixed price $600. Client informed that AI evaluation carries a small ongoing per-resume API cost, borne by the client, not included in the build price.

**Timeline (accepted):** 5–7 business days, Founder-confirmed 2026-07-31 (replacing the client's originally stated 2-week estimate), delivered via an AI-assisted engineering approach. Dependent on timely client-side access (form/Drive/Sheets, any required API credentials) and client feedback at the review checkpoint; scope changes shift the date.

## 7. Client Requirements
From the job post: accept applications through an online form with PDF resume upload; validate uploaded files before processing; store resumes in Google Drive; extract resume content; evaluate candidates using an AI model; generate a candidate score and recommendation; store structured recruitment records in Google Sheets; detect duplicate applications; send automated email notifications; be easy to maintain and extend. Preferred technology: n8n, workflow automation, Google Workspace APIs, AI/LLM integration, REST APIs, Python (if required). Nice to have: human review before final hiring decisions; configurable evaluation criteria; reusable workflow components.

## 8. Constraints & Assumptions
**Required Credentials:** Google Drive, Google Sheets, email/Gmail, LLM API key, n8n instance access, form service (if external).
**External Dependencies:** Google Workspace APIs, LLM provider availability, PDF extraction capability, form intake mechanism.
**Assumptions (labelled):** the majority of resumes are text-based PDFs rather than scanned images (scanned input would expand scope via OCR); the "online form" may use n8n's native form capability rather than a third-party service; a Workspace admin can grant the needed API access; 50–300/month sits well within normal API rate limits.

## 9. Success Criteria
Manual resume review, filing, spreadsheet updates, and candidate notification are replaced by a single automated workflow that processes applications consistently at the client's stated volume (50–300/month), preserves the existing Google Workspace environment untouched, keeps the final hiring decision with the client's team via a human review gate, and lets the client adjust evaluation criteria without a rebuild.

## 10. Risks & Open Questions
**Business risks:**
- Recurring AI evaluation API cost is not in the client's stated budget and was flagged pre-award, not after.
- Automated employment screening carries growing regulatory attention (bias-audit / disclosure obligations in some jurisdictions); client's jurisdiction is not stated. The human review gate functions as a risk control, not just a convenience feature.

**Known Unknowns (business-level, from Assessment):**
- The actual evaluation criteria — what makes a candidate score well — the single largest unknown, to be settled with the client before/at build start per the proposal's Close.
- Whether resumes are predominantly text-based or scanned PDFs.
- Whether an n8n instance already exists or must be stood up.
- Which LLM provider, and who bears its recurring cost.
- Applicant jurisdictions (bearing on data-protection obligations for candidate PII).

*(Engineering risks and acceptance-test design are CEF — engineering decision.)*

## 11. Founder Notes
None provided beyond what is already reflected in the accepted Proposal (§6) — the "Why Me" experience facts and the confirmed delivery timeline.

## 12. Engineering Context Handoff
**Solution class:** AI-assisted document-intake and evaluation workflow, orchestrated in n8n, integrated with Google Workspace (Drive, Sheets, Gmail) and an LLM provider.
**Conceptual Workflow:** Form intake → file validation → Drive storage → resume content extraction → AI evaluation (score + recommendation) → duplicate check → Sheets record → human review gate → candidate notification.
**Constraints CEF must design within:** existing Google Workspace stays untouched; evaluation criteria must be configurable without a rebuild; human review precedes any hiring decision; error handling must cover unreadable PDF, duplicate application, and extraction failure; modular/extensible design; delivery target 5–7 business days.
**Open items CEF should expect to resolve or escalate:** concrete evaluation criteria (pending client input per the proposal's Close), text-vs-scanned PDF mix, LLM provider selection and its recurring cost owner, and existing-vs-new n8n instance.

---
*Packaging only. CEF owns all engineering decisions and transforms this conceptual understanding into engineering specifications.*
