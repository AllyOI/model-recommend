# Requirements Document — Model Recommend

**Project:** Model Recommend (Concentrate AI)
**Team:** Allison Oh, Aditya Pawar
**Advisors:** Jillian Aurisano (Department of Computer Science), Todd Liebermann (Concentrate AI)
**Document version:** 1.0 (2026-10-29). The companion *Technical_Specification.md* v1.0 traces to this version.

## 1. Version and Change Log

| Version | Date | Change | Author |
|---|---|---|---|
| 1.0 | 2026-10-29 | First version. Consolidates *User_Stories.md* (US-01–US-05, UC-01–UC-02, AC-01.1–AC-02.2), *Constraints_Essay.md* and the D2 traceability table into numbered requirements REQ-01–REQ-16 and NFR-05–NFR-16, and constraints CON-01–CON-05. | Allison Oh, Aditya Pawar |
| 1.0 | 2026-10-29 | ID change: D2 Part 4 used NFR-01–NFR-04 for the four essay constraints. Those four are now **CON-01–CON-04**, and NFR-01–NFR-04 are **retired, not reused**; new non-functional requirements start at NFR-05. | Aditya Pawar |
| 1.0 | 2026-10-29 | Unit fixed everywhere: `budget` is a ceiling in **USD per 1,000 requests** (D2 gap G5). The threshold "2 consecutive HTTP 429 errors" is used in REQ-06; the UC-01 text "more than 2" is logged as OQ-02. | Allison Oh |

No requirement has been withdrawn. A requirement that is dropped later keeps its ID, is marked *Withdrawn* with the reason, and gets a row here.

## 2. Purpose and Scope

**Purpose.** This document states what Model Recommend must do and for whom, in a form a teammate can mark pass or fail. It is the reference for the Technical Specification (how it is built) and for the Week 10 test plan, whose test cases verify the IDs below.

**Product.** A product manager at a client company submits a brief: a budget ceiling, a latency ceiling and a task description. Model Recommend sends sanitized sample prompts built from the task to the models of every vendor the client company's IT security reviewer has approved. It times, prices and grades the answers, and returns a ranked shortlist with a 0–100 fit score and a plain-language reason for each model.

**In scope for this release:** brief submission and validation; vendor approval; benchmark dispatch with rate-limit handling; scoring, ranking and reasoning; solutions-engineer review; model selection; report history; notifications; the vendor usage log. Initial vendors are OpenAI and Anthropic (D1 Part 4).

**Out of scope:** everything on the Won't list in Section 7.

## 3. Stakeholders and User Goals

From the Week 4 stakeholder map (*User_Stories.md*).

| Stakeholder | Category | Goal | Story |
|---|---|---|---|
| Product manager (client company) | Primary | Get a ranked shortlist that fits the budget and latency limits, without running a benchmarking project | US-01 |
| Product manager (client company) | Primary | See why each model is ranked where it is, to defend the choice to engineering leadership | US-02 |
| Solutions engineer (Concentrate AI) | Secondary | Annotate or adjust a generated recommendation before the client company sees it | US-03 |
| IT security reviewer (client company) | Hidden | Approve which vendors may receive the company's prompts before any benchmark runs | US-04 |
| Platform operator (Concentrate AI) | Hidden | Be warned before a vendor's rate limit is exceeded, so results are not delayed or lost | US-05 |
| Concentrate AI (sponsor) | Sponsor | Keep vendor spend metered and stay within vendor terms (CON-01, CON-04) | Constraints essay |

## 4. Glossary

Each term is defined once here and used with this meaning in both documents.

| Term | Definition |
|---|---|
| **Client company** | A business customer of Concentrate AI that requests recommendations. Not the same as the Recommendation Client. |
| **Recommendation Client** | The web application (React) that users interact with; a D0 component. |
| **Recommendation Server** | The back-end service that validates briefs, runs benchmarks and ranks models; a D0 component. |
| **Report Store** | The PostgreSQL database that holds every persisted record; a D0 component. |
| **Product manager (role `pm`)** | A user at a client company who submits briefs and selects a model. |
| **IT security reviewer (role `reviewer`)** | A user at a client company who approves the vendor list. |
| **Solutions engineer (role `solutions_engineer`)** | A Concentrate AI staff member who reviews reports before release. |
| **Platform operator (role `operator`)** | A Concentrate AI staff member who receives rate-limit and vendor-key alerts. |
| **Brief** | A product manager's request: budget ceiling, latency ceiling and task description. |
| **Budget ceiling** | The most the client company will pay, in USD per 1,000 requests; 0.01–10,000. |
| **Latency ceiling** | The longest acceptable response time in milliseconds; 100–60,000. |
| **Vendor** | A company whose model API receives prompts (for example OpenAI, Anthropic). Approval applies to vendors. |
| **Model** | One vendor model that can be benchmarked and ranked. Ranking applies to models. |
| **Approved vendor list** | The set of vendors a reviewer has approved for a client company, with an approval time and an optional expiry. It is *active* when it exists and has not expired. |
| **Run** | One execution of a brief: dispatch, scoring and saving. Its status follows Fig D2-3. |
| **Sample prompt** | A sanitized prompt derived from the task description and sent to a model; 5 per model by default. |
| **Sanitization** | Removing email addresses and phone numbers from text before it leaves the Recommendation Server. |
| **Grader call** | A call to an approved vendor's model that grades one completion on a 1–5 rubric. |
| **Report** | The stored result of a run: the brief, the ranked entries and the scoring version. |
| **Shortlist** | The ordered list of report entries shown to the product manager. |
| **Fit score** | An integer 0–100 from the weighted sum of quality, cost and latency sub-scores (D1 A2). |
| **Within limits** | True when a model's projected cost ≤ budget ceiling and median latency ≤ latency ceiling. |
| **Delayed** | Model status when its vendor returned 2 consecutive HTTP 429 errors and is paused; not scored. |
| **Unavailable** | Model status when it has fewer than 3 successful samples (errors, timeouts or deadline); not scored. |
| **Review** | A solutions engineer's annotation and optional reordering of a report before **release** (made visible to the product manager). |
| **Selection** | The product manager's recorded choice of one shortlist entry. |
| **Usage log** | One VendorCall row per vendor call with tokens, latency, outcome and cost. |
| **Notification** | A message queued in the Report Store and sent by email, in-app or ops webhook. |
| **Idempotency key** | A unique value sent with a brief so that resubmitting it returns the same run. |

## 5. Functional Requirements

Priority uses MoSCoW (Section 7). Verification: **T** automated test, **D** demonstration, **I** inspection. Source is the user story, use case or acceptance criterion in *User_Stories.md*.

| ID | Requirement | Priority | Source | Verification |
|---|---|---|---|---|
| REQ-01 | The system shall, for a valid brief from a client company with an active approved vendor list, return a shortlist containing every active model of the approved vendors, in which every model with status `ok` has an integer fit score from 0 to 100 and a unique rank, ordered by within limits (true first), fit score descending, projected cost ascending, then name. | Must | US-01, AC-01.1 | T |
| REQ-02 | The system shall attach to every shortlist entry a reasoning text that states the model's quality score, its projected cost per 1,000 requests with its percentage of the budget ceiling, and its median latency with its percentage of the latency ceiling (or, for a `delayed` or `unavailable` model, the reason it was not scored); the same inputs shall produce identical text. | Must | US-02 | T |
| REQ-03 | The system shall reject a brief whose budget ceiling, latency ceiling or task description (10–2,000 characters) is missing or out of range with HTTP 400 naming the field, creating 0 runs and making 0 vendor calls. | Must | AC-01.2 | T |
| REQ-04 | The system shall send sample prompts and grader calls only to vendors on the client company's approved vendor list that is active when the run starts; with no active list it shall reject the brief with HTTP 409, and if the list cannot be read it shall reject with HTTP 503, making 0 vendor calls in both cases. | Must | US-04, UC-02 postcondition | T |
| REQ-05 | The system shall let a user with the `reviewer` role save an approved vendor list of 1 to 50 vendors, store it with an approval timestamp, and apply it to every run started at least 1 minute after the save; a submission with 0 vendors shall be rejected with the message "at least 1 vendor required" and change nothing. | Must | US-04, AC-02.1, AC-02.2 | T |
| REQ-06 | The system shall, when a vendor returns HTTP 429 on 2 consecutive calls in a run, mark that vendor's models `delayed` within 5 seconds of the second error, send it no calls for max(60 s, Retry-After), and continue the other vendors' calls. | Must | AC-01.3, UC-01 exception flow | T (mock vendor) |
| REQ-07 | The system shall show the reviewer every vendor it can route prompts to, each with a link to that vendor's published data-retention policy; 100% of catalog vendors shall have a policy link. | Should | UC-02 step 2 | I, T |
| REQ-08 | The system shall let the product manager select exactly one entry with status `ok` from a released report and store the selection with a timestamp; selecting a `delayed` or `unavailable` entry, or a report not yet released, shall be rejected. | Should | UC-01 steps 8–9 | T |
| REQ-09 | The system shall, for a client company that requires review, hold each saved report in `pending_review`, notify solutions engineers, and let one add notes (≤ 500 characters each) and move at most 2 entries before release, keeping each entry's original rank; a review that moves 3 or more entries shall be rejected. | Should | US-03, UC-01 alternate flow | T |
| REQ-10 | The system shall send the platform operator a rate-limit warning through the ops webhook when a vendor's remaining-request header falls below 10% of its limit or on its first HTTP 429, at most once per vendor per 10 minutes. | Should | US-05 | T |
| REQ-11 | The system shall record every vendor call (benchmark or grader) in the usage log with model, outcome, input and output tokens, latency and cost, written within 2 seconds of the call ending. | Should | UC-01 postcondition, CON-01 | T |
| REQ-12 | The system shall let a product manager list their client company's reports newest first (page size 1–100, default 20) and reopen any of them with its brief and shortlist; a report of another client company shall return HTTP 404. | Should | UC-01 postcondition, US-02 | T |
| REQ-13 | The system shall let the product manager cancel an active run, aborting in-flight vendor calls within 5 seconds and storing no report; resubmitting a brief with the same idempotency key within 24 hours shall return the original run and make 0 new vendor calls. | Should | US-01, CON-01 | T |
| REQ-14 | The system shall let the reviewer set an approval expiry up to 90 days ahead and shall send that reviewer one reminder 7 days before expiry; after expiry the list is inactive (REQ-04). | Could | UC-02 alternate flow | T |
| REQ-15 | The system shall notify the client company's product managers within 1 minute of an approved vendor list becoming active. | Could | UC-02 step 7 | T |
| REQ-16 | The system shall release a report automatically after 24 hours in `pending_review` and notify solutions engineers that the review was missed. | Could | US-03 (design decision R-07) | T |

## 6. Non-Functional Requirements

NFR-01–NFR-04 are retired (Section 1). "Realistic size" means 4 approved vendors, 10 models and 5 sample prompts per model (D1 A1).

| ID | Category | Requirement | Priority | Source | Verification |
|---|---|---|---|---|---|
| NFR-05 | Performance | The system shall bring at least 95% of 20 consecutive runs at realistic size to a terminal status within 90 seconds of returning HTTP 202, and every report's fit scores shall lie in 0–100. | Must | AC-01.1 | T (load, T15) |
| NFR-06 | Performance | The system shall answer the read endpoints E2, E3, E5, E7, E11 and E12 with a 95th-percentile server time ≤ 500 ms at 20 concurrent users over 5 minutes with 10,000 stored reports. | Should | US-01, US-02 | T (load) |
| NFR-07 | Cost | The system shall make no more than 2 × models × sample prompts vendor calls per run plus one retry per failed call (≤ 200 at realistic size), and allow each client company at most 3 active runs and 20 runs per hour. | Should | CON-01, US-01 | T |
| NFR-08 | Security | The system shall reject every API request without a valid identity-provider token (signature, issuer, audience, expiry within 30 s skew) with HTTP 401, a request from the wrong role with HTTP 403, and a request for another client company's data with HTTP 404, in 100% of the role-by-endpoint test matrix. | Must | US-03, US-04 | T |
| NFR-09 | Security | The system shall remove 100% of the email addresses and phone numbers in a 200-item seeded test set before any prompt is sent to a vendor, and application logs from a full test run shall contain 0 prompt texts, completions or bearer tokens. | Must | CON-02, US-04 | T |
| NFR-10 | Security | The system shall use HTTPS (TLS 1.2 or later) for all network traffic and TLS to the database, keep the access token in browser memory only, and hold 0 secrets in the repository as checked by a secret scanner on every push. | Should | CON-02 | I, T |
| NFR-11 | Usability | At least 4 of 5 first-time product managers shall submit a valid brief and name the top-ranked model and one reason for its rank within 3 minutes, without help. | Should | US-01, US-02 | Usability test |
| NFR-12 | Usability | At least 4 of 5 first-time reviewers shall save an approved vendor list within 2 minutes, without help. | Could | US-04 | Usability test |
| NFR-13 | Usability | The system shall show every 4xx error from E1–E12 as a plain-language message that names the field or the next action; 0 raw error codes shall be shown to users. | Should | AC-01.2, AC-02.2 | I, T |
| NFR-14 | Reliability | When one of 4 vendors returns 429s, 20-second timeouts, 5xx errors or 401s, the system shall complete the run with the other vendors' models scored in 20 of 20 runs for each failure type. | Must | AC-01.3, US-05 | T (mock vendor) |
| NFR-15 | Reliability | The system shall lose no run silently: a failed report write is retried once and then returned marked `not_saved`, and after a server restart every interrupted run is marked `failed` within 30 seconds of startup. | Should | UC-01 postcondition | T |
| NFR-16 | Reliability | The system shall retry each failed notification up to 5 times with backoff of 30 s × 2^n and deliver each notification at most once per deduplication key. | Could | US-05, UC-02 step 7 | T |

## 7. MoSCoW Summary

Must is 10 of 28 requirements (36%); Should 13, Could 5. The Must set is the minimum viable product: without any one of them the product cannot be released.

- **Must (10):** REQ-01, REQ-02, REQ-03, REQ-04, REQ-05, REQ-06; NFR-05, NFR-08, NFR-09, NFR-14. Without these the product either gives no recommendation, gives one it cannot explain, or sends client data where it may not go.
- **Should (13):** REQ-07, REQ-08, REQ-09, REQ-10, REQ-11, REQ-12, REQ-13; NFR-06, NFR-07, NFR-10, NFR-11, NFR-13, NFR-15.
- **Could (5):** REQ-14, REQ-15, REQ-16; NFR-12, NFR-16.
- **Won't (this release):**
  - W-01 Self-hosted open-weight models for evaluation (rejected under CON-01).
  - W-02 Public or head-to-head vendor comparisons, leaderboards or published benchmark results (CON-04).
  - W-03 Scoring weights set by the client company; weights stay in versioned configuration.
  - W-04 A learned (machine-learning) ranker (D1 A2 alternative c).
  - W-05 Tool-calling, long-context and agentic task benchmarks.
  - W-06 A message broker or background workers so runs survive a server restart (D1, R-02).
  - W-07 Billing, subscription and user-role administration screens (cut from US-04 in Week 4).
  - W-08 A native mobile app.
  - W-09 Benchmarking from more than one region.

## 8. Inherited Constraints

CON-01–CON-04 come from the Week 3 *Constraints_Essay.md*; CON-05 comes from the D0 pattern justification. Their consequences for the design are in the Technical Specification, Section 5.

| ID | Type | Constraint |
|---|---|---|
| CON-01 | Economic | Concentrate AI funds cloud and vendor API costs as needed, not against a fixed cap, but vendor spend must be metered and kept to sampled evaluation rather than exhaustive testing. |
| CON-02 | Security | Customer-submitted text is sensitive: it must not be logged in plaintext or sent unredacted to any vendor, and it may go only to vendors the client company approved. |
| CON-03 | Social | Recommendations must explain their cost, latency and quality trade-offs; the system must not act as a black box. |
| CON-04 | Legal | Vendor terms of service limit benchmarking and publishing comparisons; results must stay accurate without becoming public head-to-head claims. |
| CON-05 | Project | The team is two people and the course ends in April 2027 (M7), so the system must be buildable and operable by two developers in that time. |

## 9. Assumptions and Open Questions

**Assumptions.**

| ID | Assumption | If wrong |
|---|---|---|
| AS-01 | Concentrate AI provides vendor API keys, hosting, managed PostgreSQL and an identity-provider tenant (task T1). | MI-1 slips; the schedule's critical path moves. |
| AS-02 | Realistic size is 4 vendors, 10 models and 5 prompts per model. | NFR-05 and NFR-07 thresholds are re-derived. |
| AS-03 | Vendors return token counts but not cost; cost is tokens × configured price. | REQ-11 changes to use the vendor's reported cost. |
| AS-04 | Every user has an email address from the identity provider. | Notifications fall back to in-app only (EC-39). |
| AS-05 | Briefs and prompts are in English. | Sanitization rules and the grader rubric need review. |

**Open questions.**

| ID | Question | Affects | Owner | Due |
|---|---|---|---|---|
| OQ-01 | AC-01.1 asks for at least 3 models, but a run can return fewer when vendors fail (D2 gap G3). Amend the criterion, or keep the gap? | REQ-01, NFR-05 | Allison Oh | 2026-11-06 |
| OQ-02 | UC-01's exception flow says "more than 2" consecutive 429s; AC-01.3 and REQ-06 say 2. Confirm 2 and correct the use case. | REQ-06 | Allison Oh | 2026-11-06 |
| OQ-03 | Is 10% remaining requests the right warning threshold for operators? | REQ-10 | Aditya Pawar (with the Concentrate AI platform operator) | 2026-12-04 |
| OQ-04 | Which identity provider, and how are roles assigned to users? | NFR-08 | Aditya Pawar (with Todd Liebermann) | 2026-12-04 |
| OQ-05 | Is the rule-based sanitizer enough, or do we adopt a PII-detection library? | NFR-09 | Aditya Pawar | 2027-01-20 |
| OQ-06 | Do OpenAI's and Anthropic's terms allow showing a client company scored comparisons of their models? | CON-04, REQ-02 | Todd Liebermann | 2026-12-04 |
| OQ-07 | Are the scoring weights (quality 0.5, cost 0.3, latency 0.2) right for Concentrate AI's customers? | REQ-01 | Todd Liebermann | 2026-11-20 |
