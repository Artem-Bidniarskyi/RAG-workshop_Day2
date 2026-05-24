# RAG Test Questions & Evaluation — IT Recruitment Agency Corpus

External evaluation set. This file must NOT be indexed into the vector database.

---

## Q1 — CSV Filtering (Single Document)

**Question:**
> Which candidates currently have `active` status and are available to start immediately?

**Expected Answer (from `candidates.csv`):**
- Iryna Koval (C002) — Frontend Developer, React/TypeScript, immediately, remote only
- Maria Ivanova (C006) — Data Engineer, Python/Spark, immediately, open to hybrid

**Notes:** C004 is `placed` (not active) — must be excluded. C009 is `in_process` — must be excluded.

**Source:** `candidates.csv`
**Difficulty:** Easy — direct two-field filter
**Failure mode:** Model includes C004 or C009 because they also have `availability=immediately`

---

## Q2 — Contract Clause Lookup (PDF)

**Question:**
> What happens if TechVentures dismisses the placed candidate within the first two months?

**Expected Answer (from `contract_placement_TechVentures_J001.pdf`):**
If the candidate is dismissed at the Client's initiative within the first 60 days, the Agency shall provide one free replacement.

**Source:** `contract_placement_TechVentures_J001.pdf`, clause 2.3
**Difficulty:** Medium — requires locating a specific contract clause
**Failure mode:** Model says "nothing is specified" (retrieval miss) or invents a penalty fee

---

## Q3 — Policy Table Lookup (Markdown)

**Question:**
> How long does the agency retain data for a candidate who withdrew from the process themselves?

**Expected Answer (from `policy_gdpr_candidates.md`):**
6 months after the last contact (status: `declined`).

**Source:** `policy_gdpr_candidates.md`, retention table
**Difficulty:** Medium — must pick the correct row from a multi-row table
**Failure mode:** Model returns the `blacklisted` (1 year) or `placed` (2 years) row instead

---

## Q4 — Cross-Document Join (CSV + Markdown)

**Question:**
> A recruiter wants to submit candidate C007 for vacancy J010. Are there any blockers, and what must the recruiter check before making the call according to internal rules?

**Expected Answer (from `candidates.csv` + `recruiter_interview_checklist.md`):**
Candidate C007 (Sergiy Tkachenko) has a `blacklisted` status due to two no-shows at scheduled interviews. According to the recruiter checklist, CRM status must be checked before every call — if `blacklisted`, the call must not proceed. The blacklist decision must be confirmed by the Recruitment Team Lead.

**Source:** `candidates.csv` (C007 status) + `recruiter_interview_checklist.md` (pre-call section + blacklist grounds)
**Difficulty:** Hard — requires joining information from two documents
**Failure mode:** Model answers only about the vacancy fit (iOS/Swift) without flagging the blacklist

---

## Q5 — Cross-Document Join (CSV + PDF)

**Question:**
> CloudSystems has an open vacancy. What confidentiality rules apply when sharing candidate resumes with them, and what is the financial penalty for a breach?

**Expected Answer (from `listings.csv` + `contract_nda_CloudSystems.pdf`):**
CloudSystems has an open vacancy J004 (Senior DevOps Engineer, $4,000–5,000, fully remote). Per the NDA (Contract No. NDA/2026/05-CS), resumes may only be shared after the candidate's written or CRM-logged consent. The client must delete candidate data upon rejection. The penalty for each confidentiality breach is $2,000.

**Source:** `listings.csv` (J004) + `contract_nda_CloudSystems.pdf` (clauses 2.3, 3.2)
**Difficulty:** Hard — requires linking an active vacancy to a separate legal document
**Failure mode:** Model gives generic GDPR rules instead of the specific NDA penalty amount

---

## Evaluation Rubric

| Score | Criteria |
|---|---|
| 2 — Full credit | Correct answer, correct source cited, no hallucinated facts |
| 1 — Partial credit | Answer is partially correct OR correct but wrong/missing source |
| 0 — No credit | Wrong answer, hallucinated facts, or "not enough information" when data exists |

## Summary Table

| # | Question Type | Source Documents | Difficulty | Key Failure Mode |
|---|---|---|---|---|
| Q1 | CSV filter | candidates.csv | Easy | Including wrong-status candidates |
| Q2 | Clause lookup | contract PDF | Medium | Inventing terms not in contract |
| Q3 | Table row lookup | policy MD | Medium | Returning wrong retention period |
| Q4 | Cross-doc join | CSV + MD | Hard | Missing the blacklist flag |
| Q5 | Cross-doc join | CSV + PDF | Hard | Generic answer instead of NDA specifics |

Questions Q4 and Q5 are the most valuable for RAG evaluation — they test whether the system can connect information across different document types and chunk boundaries.
