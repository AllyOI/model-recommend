# Test and Validation Plan — Model Recommend

**Project:** Model Recommend (Concentrate AI)
**Team:** Allison Oh, Aditya Pawar
**Test lead (document owner):** Allison Oh. Each component owner wrote the cases for their component: Aditya Pawar for the Recommendation Server, LLM adapter and Authentication; Allison Oh for the Recommendation Client, Report Store, Notifications and Configuration.
**Version:** 1.0 (2026-11-04). Traces to *Requirements_Document.md* v1.0 and *Technical_Specification.md* v1.1, whose traceability matrix names the test-case groups TC-01 to TC-29 used here.

**Numbering.** Each Week 9 matrix ID (for example TC-06) is a group; the cases in it are TC-06.1, TC-06.2 and so on. No group was added or renumbered. The sub-numbering is logged in the Technical Specification's version log (v1.1). Every case below has status **Not Run**: implementation starts in CS 5002 (milestone M0, 2027-01-20).

## 1. Scope

**This cycle** is the CS 5002 build, January to April 2027 (milestones M0 to M7 in *Planning/Plan.md*). Requirements are tested in risk order: anything that sends client data out or writes stored data first (REQ-03, REQ-04, REQ-05, NFR-08, NFR-09), then the core run (REQ-06, NFR-14, REQ-01, REQ-02, NFR-05), then everything else.

**In scope.**

| Component (D0/D2) | Requirements tested | Owner |
|---|---|---|
| Recommendation Server: validation, run lifecycle, A1 dispatch, A2 ranking, review, selection | REQ-01, REQ-02, REQ-03, REQ-04, REQ-06, REQ-08, REQ-09, REQ-13; NFR-05, NFR-07, NFR-14, NFR-15 | Aditya Pawar |
| LLM adapter (`callVendor`) and usage log | REQ-06, REQ-11; NFR-14 | Aditya Pawar |
| Authentication glue | NFR-08 | Aditya Pawar |
| Report Store (schema, repository methods, indexes) | REQ-05, REQ-07, REQ-12; NFR-06, NFR-15 | Allison Oh |
| Recommendation Client (screens S1–S6) | REQ-03 (client checks), REQ-05, REQ-07; NFR-10, NFR-11, NFR-13 | Allison Oh |
| Notifications glue | REQ-09, REQ-10 | Allison Oh |
| Configuration, logging and CI | NFR-07, NFR-09 (log redaction), NFR-10 | Allison Oh |
| Constraints | CON-01 to CON-04 through the cases listed in Section 4; CON-05 at milestone reviews | Both |

**Out of scope this cycle, and why.**

| Item | Reason |
|---|---|
| Could requirements REQ-14, REQ-15, REQ-16, NFR-12, NFR-16 | Deferred features. Their cases (TC-14, TC-15, TC-16, TC-24, TC-28) run only if the feature ships by M5; otherwise they stay Not Run and are reported as deferred, not failed. |
| Won't items W-01 to W-09 (Requirements Document, Section 7) | Not built, so not tested: self-hosted models, public comparisons, client-set weights, learned ranker, tool-calling and agentic benchmarks, a broker, admin screens, mobile app, multi-region. |
| Known limitation: one test location | Every latency number is measured from one region (CON-05, Tech Spec Section 5). |
| Known limitation: rule-based sanitizer | TC-21 tests the 200-item corpus NFR-09 defines; formats outside it are a known gap until OQ-05 is decided. |
| Known limitation: fewer than 3 models | TC-01.4 checks the documented behavior; whether AC-01.1 changes is OQ-01. |
| Third-party components not retested | OpenAI and Anthropic APIs, the OIDC identity provider, PostgreSQL itself, the hosting platform, the email provider, Slack, and libraries (Fastify, zod, p-limit, p-retry, jose, Kysely, pino). We test our use of them through mocks and staging, not the components themselves. |

## 2. Verification and Validation Strategy

**Verification** (built right, against the spec) is done by the team for every functional requirement and every non-functional requirement with a measured threshold: REQ-01 to REQ-16 and NFR-05 to NFR-10, NFR-13 to NFR-16. Almost all of it is automated and runs in CI against the mock vendor server, so results do not depend on vendor uptime or cost (CON-01). A smaller set repeats on staging with real vendor APIs (TC-06.4, TC-17.2, TC-26 subset).

**Validation** (the right product, for the stakeholders) is done with people outside the team:

- **Product managers** (US-01, US-02): usability sessions for NFR-11 with 5 first-time users, run first on staging and again in the pilot with Concentrate AI product managers.
- **Solutions engineer and advisor** (US-01, CON-03): ranking validation TC-01.5, where a Concentrate AI solutions engineer ranks the same candidates by hand and we compare.
- **IT security reviewer** (US-04): NFR-12 session, if built.
- **Todd Liebermann** (CON-04): sign-off on reasoning wording (TC-29) and the M4 demo.
- **Jillian Aurisano**: reviews this plan and the M6 test report.

**Model-based parts.** The quality grade comes from an LLM grader call, so ranking has run-to-run noise. Variables: brief, approved model set, prompts per model (5). Metrics: top-ranked model and fit score. Trials: 5 identical runs per brief on staging. Tolerance: the same top-1 model in at least 4 of 5 trials and a fit-score standard deviation of at most 5 points per model (TC-01.6).

**Test pyramid.**

| Layer | Approx. count | Tool | Coverage focus |
|---|---|---|---|
| Unit | 120 | vitest | Brief validation, A2 scoring and tie-breaks, reasoning template, sanitizer, limiter state machine, config validation, cost arithmetic |
| Integration | 45 | vitest + Docker PostgreSQL 16 + mock vendor | Endpoints E1–E12 with the database, auth matrix, run lifecycle, notifications outbox, usage log |
| System (end to end) | 25 | Playwright through the Client | UC-01 and UC-02 flows, error messages, review and selection |
| Load and fault | 6 scenarios | k6, Toxiproxy, mock vendor modes | NFR-05, NFR-06, NFR-07, NFR-14, NFR-15 |
| Acceptance and validation | 3 sessions + M4 demo | Scripted sessions, observer sheet | NFR-11, NFR-12, TC-01.5, TC-29 |

## 3. Success Criteria (Metrics)

| Metric | Target threshold | How it is measured | Req. ID |
|---|---|---|---|
| Run completion time | ≥ 95% of 20 consecutive runs at realistic size reach a terminal status ≤ 90 s after `202` | `runs.finished_at − runs.created_at` for each run | NFR-05 |
| Fit-score correctness | 100% of 30 fixture models match the reference spreadsheet exactly (integer) | Unit test against `fixtures/a2_reference.csv` | REQ-01 |
| Ranking validation (validation) | System top-1 is in the solutions engineer's top 3 for ≥ 4 of 5 briefs | Blind comparison, TC-01.5 | REQ-01, CON-03 |
| Ranking stability (validation) | Same top-1 in ≥ 4 of 5 trials; fit-score SD ≤ 5 points | 5 staging runs per brief, TC-01.6 | REQ-01 |
| Reasoning accuracy | 100% of numbers in reasoning text equal the stored entry values | String parse vs `report_entries` | REQ-02 |
| Rate-limit reaction | `delayed` ≤ 5 s after the 2nd 429; 0 calls during max(60 s, Retry-After) | Server log timestamps vs mock request log | REQ-06 |
| Approval activation | List applied to every run started ≥ 60 s after save | Mock request log per vendor | REQ-05 |
| Unapproved-vendor calls | 0 | Mock request log | REQ-04 |
| Read-endpoint latency | p95 ≤ 500 ms at 20 virtual users for 5 min, 10,000 reports | k6 report + `Server-Timing` header | NFR-06 |
| Calls per run and quotas | ≤ 2 × models × prompts + 1 retry per failure; 4th active run → 409; 21st run/hour → 429 | Count rows in `vendor_calls`; API responses | NFR-07 |
| Usage-log completeness and cost accuracy | 100% of calls logged ≤ 2 s after the call; cost = tokens × price within $0.000001 | Compare mock log with `vendor_calls` | REQ-11 |
| Cancellation | 0 vendor calls start > 5 s after cancel; no report stored | Mock log, `reports` table | REQ-13 |
| Auth matrix | 100% of role × endpoint × token cases return the expected code | Integration test table | NFR-08 |
| PII removal | 100% of 200 seeded items absent from outbound prompts; 0 prompts, completions or tokens in logs | Regex scan of mock log and log files | NFR-09 |
| Transport and secrets | TLS ≥ 1.2 only; 0 secrets found by gitleaks; token absent from browser storage | testssl.sh, CI scan, Playwright storage check | NFR-10 |
| Product-manager task success | ≥ 4 of 5 users finish within 3 min, unaided | Observer stopwatch and sheet | NFR-11 |
| Reviewer task success | ≥ 4 of 5 users finish within 2 min, unaided | Observer stopwatch and sheet | NFR-12 |
| Error-message clarity | 0 raw error codes shown for the 17 4xx codes | Playwright screenshot check | NFR-13 |
| Vendor isolation | Other vendors scored in 20 of 20 runs for each of 4 failure modes | Mock modes, run results | NFR-14 |
| No silent loss | Interrupted runs `failed` ≤ 30 s after restart; 0 non-terminal runs | SQL query after restart | NFR-15 |
| Notification retry | 5 retries at 30/60/120/240/480 s; 0 duplicates per key | `notifications` table | NFR-16 |

## 4. Test Cases

All cases use the seeded data in Section 5 (client companies Acme, Beta, Gamma; users `pm_acme`, `rev_acme`, `pm_beta`, `pm_gamma`, `se_1`, `se_2`, `op_1`; vendors V1–V4). "Mock" is the mock vendor server; its request log records every call it receives. "Standard brief" is `{budget: 3.00, latency_ms: 2000, task_description: "Summarize customer support tickets into two sentences."}`. Status for every case: **Not Run**.

### TC-01: Ranked shortlist (REQ-01, Must; owner Aditya Pawar)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-01.1 | Happy path ranking (REQ-01) | 1. Seed; set mock to `ok` with grades file `grades_a.json`. 2. As `pm_acme`, POST E1 with the standard brief and a new Idempotency-Key. 3. Poll E5 every 2 s until terminal. | Status `released`; 10 entries; every `ok` entry has an integer `fit_score` 0–100; ranks 1..10 unique; order equals `expected_order_a.json`. | Not Run |
| TC-01.2 | Tie-break order (REQ-01, edge) | 1. Load fixture with models M3 and M7 given identical samples, M3 cheaper; M8 and M9 identical in samples and price. 2. Run A2 unit test. | M3 ranks above M7; M8 above M9 (name order); repeated 10 times, same order. | Not Run |
| TC-01.3 | All models over budget (REQ-01, edge) | 1. As `pm_acme`, POST E1 with budget 0.01, otherwise standard. 2. Poll E5. | All entries `within_limits: false`, still ranked 1..10; each reasoning names the ceiling missed. | Not Run |
| TC-01.4 | Fewer than 3 rankable models (REQ-01, failure) | 1. Set mock V1, V2, V3 to `401`. 2. Run standard brief as `pm_acme`. | Only V4's models ranked (2); warning `fewer_than_3_models`; no unscored entry has a rank (OQ-01 behavior). | Not Run |
| TC-01.5 | Ranking validation (US-01, CON-03, validation) | 1. Pick 5 pilot briefs. 2. Solutions engineer ranks the candidate models by hand without seeing results. 3. Compare with the system's top-1. | System top-1 is in the engineer's top 3 for ≥ 4 of 5 briefs. | Not Run |
| TC-01.6 | Ranking stability (REQ-01, validation) | 1. On staging, run the same brief 5 times. 2. Record top-1 and fit scores. | Same top-1 in ≥ 4 of 5 runs; SD of each model's fit score ≤ 5. | Not Run |

### TC-02: Reasoning text (REQ-02, Must; owner Aditya Pawar)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-02.1 | Reasoning numbers (REQ-02) | 1. Run A2 on fixture `a2_reference.csv` with budget 3.00, latency 2000. 2. Parse each reasoning string. | Each contains quality (2 decimals), "$x.xx per 1,000 requests (y% of your $3.00 ceiling)" and "median latency L ms (z% of your 2,000 ms ceiling)", all equal to stored values. | Not Run |
| TC-02.2 | Same input, same text (REQ-02) | 1. Run A2 twice on identical samples. 2. Store a report, reopen it via E2 after 1 day (clock advanced). | Strings byte-identical across both runs and the reopened report. | Not Run |
| TC-02.3 | Unscored entry (REQ-02, edge) | 1. Set mock V2 to `429` always. 2. Run standard brief. | V2 entries have `fit_score: null` and reasoning stating the vendor was rate-limited and paused. | Not Run |

### TC-03: Brief validation (REQ-03, Must; owner Aditya Pawar)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-03.1 | Missing field (REQ-03, AC-01.2) | 1. Note `runs` count and mock log size. 2. POST E1 as `pm_acme` without `budget`. 3. Repeat without `latency_ms`, then without `task_description`. | Each returns 400 `invalid_brief` with `field` set to the missing name; `runs` count and mock log unchanged. | Not Run |
| TC-03.2 | Boundaries (REQ-03, edge) | POST E1 with budget 0.01, 10000, 0.009, 10000.01; latency 100, 60000, 99, 60001; task 10, 2000, 9, 2001 characters (12 requests, other fields standard). | The six in-range values return 202; the six out-of-range values return 400 naming the field; 0 vendor calls for rejected ones. | Not Run |
| TC-03.3 | Bad input in the Client (REQ-03, NFR-13) | 1. In S1 type budget "3,00", latency "1500.5", a 2,001-character task. 2. Try Submit. 3. Send the same values directly to E1. | S1 shows the field messages and Submit stays disabled; direct API calls return 400. | Not Run |

### TC-04: Approved vendors only (REQ-04, Must; owner Aditya Pawar)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-04.1 | Only approved vendors called (REQ-04) | 1. Set Acme's list to V1, V2. 2. Run standard brief. 3. Filter mock log by vendor. | Benchmark and grader calls go only to V1 and V2; 0 calls to V3, V4. | Not Run |
| TC-04.2 | No list (REQ-04, failure) | As `pm_beta` (no approvals) POST E1. | 409 `no_approved_vendors`; 0 mock calls; S1 shows the "ask your IT reviewer" message. | Not Run |
| TC-04.3 | Expired list (REQ-04, edge) | 1. Set Acme's `expires_at` to 1 minute ago. 2. POST E1. | 409 `no_approved_vendors`; 0 mock calls. | Not Run |
| TC-04.4 | Store down, fail closed (REQ-04, failure) | 1. Cut the database connection with Toxiproxy. 2. POST E1. | 503 `store_unavailable`; 0 mock calls. | Not Run |

### TC-05: Vendor approval (REQ-05, Must; owner Allison Oh)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-05.1 | Save and apply (REQ-05, AC-02.1) | 1. As `rev_acme`, open S4, select V1 and V3, save. 2. Call E11. 3. Wait 60 s; run standard brief as `pm_acme`. | 200 with `approved_at` within 2 s of server time; E11 lists V1, V3; the run calls only V1, V3. | Not Run |
| TC-05.2 | Zero vendors (REQ-05, AC-02.2, failure) | 1. Record E11. 2. In S4 clear all boxes and save. | Message "at least 1 vendor required"; 400 `at_least_one_vendor_required`; E11 unchanged. | Not Run |
| TC-05.3 | Wrong role and bad input (REQ-05, edge) | 1. PUT E4 as `pm_acme`. 2. As `rev_acme`, PUT 51 vendor IDs. 3. PUT one unknown UUID. | 403 `reviewer_access_required`; 400; 404 `unknown_vendor` listing the ID. | Not Run |

### TC-06: Rate-limit handling (REQ-06, Must; owner Aditya Pawar)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-06.1 | Pause after two 429s (REQ-06, AC-01.3) | 1. Set mock V2 to `429` always. 2. Run standard brief. 3. Compare server log and mock log timestamps. | V2 `delayed` ≤ 5 s after the 2nd 429; 0 calls to V2 for 60 s; V1, V3, V4 ranked. | Not Run |
| TC-06.2 | Longer Retry-After (REQ-06, edge) | Set V2 to `429` with `Retry-After: 120`; run standard brief. | 0 calls to V2 for 120 s. | Not Run |
| TC-06.3 | Non-consecutive 429s (REQ-06, edge) | Set V2 to alternate 429, 200, 429, 200. | V2 never `delayed`; its models scored. | Not Run |
| TC-06.4 | Real vendor on staging (REQ-06) | On staging, lower V1's limit with a test key at 2 requests/min; run standard brief. | Same behavior as TC-06.1 with the real API. | Not Run |

### TC-07 to TC-13: Should requirements

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-07.1 | Vendor list (REQ-07; Allison) | As `rev_acme`, GET E7; open S4. | 4 vendors, each with a non-empty `https://` policy link; the link opens in a new tab. | Not Run |
| TC-07.2 | Missing policy link (REQ-07, failure) | Insert a vendor row with a null `retention_policy_url`. | Insert rejected by the NOT NULL constraint. | Not Run |
| TC-08.1 | Select a model (REQ-08; Aditya) | After TC-01.1, POST E9 with rank-1 `entry_id`; GET E2. | 200 with `selected_at`; E2 shows `selected_entry_id`. | Not Run |
| TC-08.2 | Not selectable (REQ-08, failure) | POST E9 with a `delayed` entry; then on a `pending_review` report. | 400 `entry_not_selectable`; 409 `report_not_released`. | Not Run |
| TC-09.1 | Review hold (REQ-09; Aditya) | Run standard brief as `pm_gamma` (requires review); check E5 as PM and E12 as `se_1`. | Status `pending_review`; PM sees no report; `se_1` has `report_needs_review`. | Not Run |
| TC-09.2 | Review and release (REQ-09) | As `se_1`, POST E8 moving 2 entries with notes, `release: true`; GET E5 as PM. | New order shown; `original_rank` kept; notes shown. | Not Run |
| TC-09.3 | Review limits (REQ-09, failure) | POST E8 moving 3 entries; then a 501-character note; then `se_2` releases after `se_1`. | 400 `too_many_reorders`; 400; 409 `already_released`. | Not Run |
| TC-10.1 | Low-headroom warning (REQ-10; Allison) | Set mock V1 remaining-requests header to 9%; run standard brief. | One ops-webhook post ≤ 10 s with `type`, `vendor`, `remaining_pct`, `run_id`. | Not Run |
| TC-10.2 | First 429 warning (REQ-10) | Set V2 to return one 429. | One `rate_limit_warning` for V2. | Not Run |
| TC-10.3 | Deduplication (REQ-10, edge) | Trigger 20 warnings for V1 within 10 min. | Exactly 1 webhook post. | Not Run |
| TC-11.1 | Every call logged (REQ-11; Aditya) | Run standard brief; compare mock log with `vendor_calls`. | 100 rows; each written ≤ 2 s after the call; cost = tokens × price within $0.000001. | Not Run |
| TC-11.2 | Failed call logged (REQ-11, edge) | Set V3 to `timeout` (25 s). | V3 rows have outcome `timeout`. | Not Run |
| TC-11.3 | Log write fails (REQ-11, failure) | Block writes to `vendor_calls` during a run. | Run still completes; error logged; gap flagged for the operator. | Not Run |
| TC-12.1 | History paging (REQ-12; Allison) | Seed 25 Acme reports; GET E3, then E3 with `next_cursor`. | 20 newest first; then the remaining 5; `next_cursor` null. | Not Run |
| TC-12.2 | Bad paging input (REQ-12, edge) | GET E3 with `limit=0`, `limit=101`, a tampered cursor. | 400 `invalid_limit` twice; 400 `invalid_cursor`. | Not Run |
| TC-12.3 | Other company's report (REQ-12, failure) | As `pm_beta`, GET E2 with an Acme `report_id`, then with a random UUID. | Both 404 `report_not_found` with identical bodies. | Not Run |
| TC-13.1 | Cancel (REQ-13; Aditya) | Start standard brief; after 10 s POST E6. | `cancelled`; 0 mock calls start > 5 s after cancel; no `reports` row. | Not Run |
| TC-13.2 | Idempotent resubmit (REQ-13) | POST E1 twice with the same Idempotency-Key 1 min apart. | Same `run_id`; mock call count unchanged by the second. | Not Run |
| TC-13.3 | Cancel a finished run (REQ-13, edge) | POST E6 on a `released` run. | 409 `run_not_active`. | Not Run |

### TC-14 to TC-16: Could requirements (run only if built by M5)

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-14.1 | Expiry reminder (REQ-14) | Save list expiring in 8 days; advance clock 1 day; run reminder job twice. | One email to `rev_acme`; `reminder_sent_at` set; a 91-day expiry is rejected with 400. | Not Run |
| TC-15.1 | List-active notice (REQ-15) | Save a list as `rev_acme`. | Each Acme PM gets `vendor_list_active` ≤ 60 s. | Not Run |
| TC-15.2 | No email address (REQ-15, edge) | Remove the email of one Acme PM; save a list. | That PM gets the in-app notice only; a warning is logged. | Not Run |
| TC-16.1 | Auto-release (REQ-16) | Leave a Gamma report in review; advance clock 24 h. | Report released; `review_overdue` to engineers. | Not Run |
| TC-16.2 | Just before the limit (REQ-16, edge) | Advance clock 23 h 59 min; run the timeout job. | Report still `pending_review`; no notification. | Not Run |

### TC-17 to TC-22: Performance, cost and security

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-17.1 | Run time, mock (NFR-05; Aditya) | Mock latency median 800 ms, p95 2.5 s; 20 consecutive standard runs at realistic size. | ≥ 19 of 20 terminal ≤ 90 s. | Not Run |
| TC-17.2 | Run time, real vendors (NFR-05) | Same 20 runs on staging. | ≥ 19 of 20 terminal ≤ 90 s. | Not Run |
| TC-17.3 | Slow vendor (NFR-05, edge) | Set V4 to 15 s per call. | Run terminal ≤ 90 s with warning `deadline_reached`. | Not Run |
| TC-18.1 | Read latency (NFR-06; Allison) | Seed 10,000 reports; k6 with 20 VUs for 5 min over E2, E3, E5, E7, E11, E12. | p95 ≤ 500 ms per endpoint. | Not Run |
| TC-18.2 | Deep paging (NFR-06, edge) | Page E3 to the 50th page. | Each page ≤ 500 ms. | Not Run |
| TC-19.1 | Calls per run (NFR-07; Allison) | Run standard brief; count `vendor_calls`. | ≤ 100 plus one per failed call. | Not Run |
| TC-19.2 | Active-run cap (NFR-07, failure) | Start 4 runs for Acme within 5 s. | 4th returns 409 `run_in_progress`. | Not Run |
| TC-19.3 | Hourly cap (NFR-07, edge) | Start 21 runs in one hour (clock controlled). | 21st returns 429 with `Retry-After`. | Not Run |
| TC-20.1 | Role matrix (NFR-08; Aditya) | Call E1–E12 with each of 4 roles, no token, expired token, bad signature, wrong audience (96 calls). | 100% match `auth_matrix.csv`. | Not Run |
| TC-20.2 | Clock skew (NFR-08, edge) | Token expired 29 s ago; then 31 s ago. | Accepted; then 401. | Not Run |
| TC-20.3 | Identity provider down (NFR-08, failure) | Block JWKS URL for 11 min. | Requests succeed for 10 min on cached keys, then 503 `auth_unavailable`. | Not Run |
| TC-21.1 | PII corpus (NFR-09; Aditya) | Embed the 200 items of `pii_corpus.txt` (100 emails, 100 phone formats) in 20 briefs; run them; regex-scan mock log. | 0 seeded values in any outbound prompt. | Not Run |
| TC-21.2 | Log redaction (NFR-09) | After TC-21.1, grep application logs for prompt text, completions and `Bearer `. | 0 matches. | Not Run |
| TC-21.3 | Grader path (NFR-09, edge) | Inspect grader calls from TC-21.1. | Grader prompts also contain 0 seeded values. | Not Run |
| TC-22.1 | TLS (NFR-10; Allison) | Run testssl.sh against staging API and Client. | Only TLS 1.2 and 1.3 offered. | Not Run |
| TC-22.2 | Secrets in repo (NFR-10, failure) | Push a branch containing a fake API key. | CI gitleaks step fails; main stays at 0 findings. | Not Run |
| TC-22.3 | Token storage (NFR-10) | Sign in via Playwright; read localStorage and sessionStorage. | No token present. | Not Run |

### TC-23 to TC-29: Usability, reliability and constraints

| ID | Description | Steps | Expected Result | Status |
|---|---|---|---|---|
| TC-23.1 | PM task, staging (NFR-11; Allison) | 5 first-time users get the script "Find the best model for summarizing tickets under $3 per 1,000 requests and 2 s"; observer times from S1 load. | ≥ 4 of 5 submit a valid brief and name the top model and one reason ≤ 3 min, unaided. | Not Run |
| TC-23.2 | PM task, pilot (NFR-11, validation) | Repeat TC-23.1 with 5 Concentrate AI product managers. | Same threshold. | Not Run |
| TC-24.1 | Reviewer task (NFR-12, Could) | 5 first-time users approve 2 vendors from S4. | ≥ 4 of 5 ≤ 2 min, unaided. | Not Run |
| TC-25.1 | Error messages (NFR-13; Allison) | Trigger each of the 17 4xx codes from E1–E12 through the Client (mocked API responses). | Each shows its mapped plain-language message; 0 raw codes. | Not Run |
| TC-25.2 | Network loss (NFR-13, edge) | Go offline in Playwright during polling for 130 s. | Banner with Retry; Retry resumes the same `run_id`. | Not Run |
| TC-26.1 | Vendor 429 (NFR-14; Aditya) | 20 runs with V2 in `429` mode. | V1, V3, V4 scored in 20 of 20. | Not Run |
| TC-26.2 | Vendor timeout (NFR-14) | 20 runs with V2 in `timeout` mode. | Others scored in 20 of 20. | Not Run |
| TC-26.3 | Vendor 5xx (NFR-14) | 20 runs with V2 returning 503. | Others scored in 20 of 20. | Not Run |
| TC-26.4 | Vendor 401 (NFR-14) | 20 runs with V2 returning 401. | Others scored in 20 of 20; one operator alert. | Not Run |
| TC-26.5 | All vendors fail (NFR-14, failure) | Set V1–V4 to 503. | Run `failed` with `all_vendors_unavailable`; no report. | Not Run |
| TC-27.1 | Report write fails (NFR-15; Allison) | Block database writes during `storeReport`. | One retry; run `released_unsaved`; E5 shows `save_status: not_saved` and the shortlist. | Not Run |
| TC-27.2 | Restart mid-run (NFR-15, failure) | Kill the server 20 s into a run; restart. | Run `failed` with `run_interrupted` ≤ 30 s after startup. | Not Run |
| TC-27.3 | No stuck runs (NFR-15) | After TC-27.2, count runs in `queued` or `dispatching`. | 0. | Not Run |
| TC-28.1 | Notification retry (NFR-16, Could) | Point email at a failing stub; queue one notification. | 5 retries at 30/60/120/240/480 s, then `failed`; a duplicate key adds no row. | Not Run |
| TC-29.1 | Reasoning wording (CON-04; Allison, Todd Liebermann) | Generate 50 reasoning strings from varied briefs; scan for the banned list ("better than", "beats", "best model", vendor-vs-vendor wording); Todd reviews. | 0 banned phrases; Todd signs off. | Not Run |

**Constraint coverage.** CON-01: TC-11, TC-13, TC-19. CON-02: TC-04, TC-20, TC-21. CON-03: TC-02, TC-01.5. CON-04: TC-12.3, TC-29. CON-05 has no test case; it is checked at milestone reviews.

## 5. Test Environment

| Item | Development | Staging | Pilot |
|---|---|---|---|
| Purpose | Unit, integration, system tests on every push | Real-vendor, load and usability tests before reviews | Validation with Concentrate AI users (M4–M6) |
| Hosts | macOS 15 laptops (Apple silicon) and GitHub Actions `ubuntu-24.04` | Managed container platform (Concentrate AI account), 1 instance, 1 vCPU / 2 GB | Same platform, separate project |
| Runtime | Node.js 22 LTS (pinned in `.nvmrc`), Docker 27 | Same Docker image as CI | Same image, tagged release |
| Database | PostgreSQL 16 (`postgres:16` Docker image), migrated with node-pg-migrate | Managed PostgreSQL 16, TLS required | Managed PostgreSQL 16, separate database |
| Vendors | Mock vendor server with modes `ok`, `429`, `timeout`, `5xx`, `401`, `empty`, `truncated`, headers configurable | Real OpenAI and Anthropic test keys with spend caps | Real keys under Concentrate AI's account |
| Identity | Local OIDC stub issuing signed test tokens | Identity-provider staging tenant (OQ-04) | Production tenant, pilot users only |
| Browsers | Chromium, Firefox and WebKit bundled with the Playwright version pinned in `package-lock.json` | Chrome, Safari and Firefox current stable; versions recorded per run | Users' own browsers; version recorded in session sheet |
| Network | Local; Toxiproxy for faults; Playwright offline and Slow 3G profiles | Public internet from Cincinnati | Users' own networks |
| Tools | vitest, Playwright, k6, Toxiproxy, gitleaks | k6, testssl.sh | Observer sheet, stopwatch |

**Seeded data** (`npm run seed:test`, deterministic): client companies Acme (V1–V4 approved), Beta (no approvals), Gamma (requires review); one user per role per company plus `se_1`, `se_2`, `op_1`; vendors V1–V4 with policy links; 10 models with fixed prices; `grades_a.json`, `expected_order_a.json`, `a2_reference.csv`, `auth_matrix.csv`, `pii_corpus.txt` (200 items); 20 briefs; and, for load tests only, 10,000 reports.

## 6. Pass and Fail Criteria

**Each test case.** Pass when every condition in Expected Result holds exactly, including each threshold. Fail when any condition does not hold or the system crashes. Blocked when the environment, not the system, prevents the run (for example a vendor outage on staging); a blocked case is rerun and never counted as a pass. Not Run means it has not been executed. A failure is logged as a GitHub issue with severity:

- **Sev 1:** data leaves to an unapproved vendor, PII or tokens leak, another company's data is visible, or stored data is lost or corrupted.
- **Sev 2:** a Must requirement fails, or a metric in Section 3 misses its threshold.
- **Sev 3:** a Should requirement fails, with a workaround.
- **Sev 4:** cosmetic.

**Metrics.** A metric passes when the measured value meets the Section 3 threshold on the stated number of trials. For sampled metrics (NFR-05, NFR-14, NFR-11) the sample is fixed in advance and not extended to make a result pass.

**The plan as a whole** passes at M6 when:

1. 100% of Must cases (TC-01 to TC-06, TC-17, TC-20, TC-21, TC-26) pass.
2. At least 90% of Should cases pass, and every failing one has a logged Sev 3 issue.
3. Every Section 3 metric for Must and Should requirements meets its threshold.
4. 0 open Sev 1 or Sev 2 defects.
5. Validation cases TC-01.5, TC-23.2 and TC-29.1 pass, or the advisors accept a documented exception.

It fails if any of these is not met. Could cases do not count either way.

## 7. Schedule

Unit and integration tests run on every push during sprints; system and acceptance tests run before each review and demo. Order follows risk (Section 1).

| Milestone (date) | Test activity | Owner | Evidence |
|---|---|---|---|
| MI-1 Dev environment (2026-12-04) | Mock vendor server, OIDC stub, seed script and CI skeleton built; smoke call to each real vendor | Aditya Pawar | Mock server in repo; CI run log; vendor smoke output |
| M0 CI and config (2027-01-20) | Unit tests for config and logging; gitleaks in CI (TC-22.2) | Allison Oh | Green CI run; failing run on planted key |
| M1 Walking skeleton (2027-01-29) | Integration: auth matrix (TC-20), store schema and constraints (TC-07.2), brief validation (TC-03) | Aditya Pawar, Allison Oh | CI report with case IDs |
| M2 Real vendor benchmarks (2027-02-16) | Integration: approved vendors only (TC-04), rate limits (TC-06.1–06.3), usage log (TC-11), isolation (TC-26); staging TC-06.4 | Aditya Pawar | CI report; mock and staging logs |
| M3 Ranked shortlist (2027-03-09) | Unit and integration: ranking and reasoning (TC-01.1–01.4, TC-02); load TC-17.1, TC-17.3; cost caps TC-19 | Aditya Pawar | A2 test report; k6 and run-time CSV |
| M4 MVP advisor demo (2027-03-25) | System: UC-01 end to end in Playwright; usability TC-23.1; ranking validation TC-01.5; wording TC-29 | Allison Oh | Playwright report; session sheets; advisor notes |
| M5 Feature complete (2027-04-10) | System: approval (TC-05), review and selection (TC-08, TC-09), notifications (TC-10), history (TC-12), cancel (TC-13), errors (TC-25); Could cases if built | Allison Oh | Playwright report; issue list |
| M6 Validation complete (2027-04-28) | Full regression; staging TC-17.2, TC-18, TC-21, TC-22, TC-27; stability TC-01.6; pilot TC-23.2 | Allison Oh (tests), Aditya Pawar (load) | Test report with pass/fail per case and metric; advisors sign off |
| M7 Final presentation and handoff (2027-04-30) | Report results; hand over test suite | Both | Final report section; repository |

## 8. Definition of Done

**A feature is done when:**

- [ ] Its test cases in Section 4 are written as automated tests (or a session script for usability) and pass in CI or on staging.
- [ ] Every edge and failure case in its group passes.
- [ ] The code has been reviewed and approved by the other team member.
- [ ] Unit-test line coverage for its module is at least 80%.
- [ ] The API, configuration or screen change is reflected in the Technical Specification and README.
- [ ] It has 0 open Sev 1 or Sev 2 defects.
- [ ] Its traceability-matrix row is updated (status and test-case IDs).

**The project is done when:**

- [ ] The plan-level pass criteria in Section 6 are met.
- [ ] Every Must and Should requirement has a passing test or an advisor-accepted exception.
- [ ] Every Section 3 metric has a recorded measured value.
- [ ] The test report, Requirements Document, Technical Specification and this plan are at matching versions.
- [ ] 0 open Sev 1 or Sev 2 defects; open Sev 3 and 4 issues are listed in the handoff.
- [ ] Jillian Aurisano and Todd Liebermann have signed off on the M6 test report.
