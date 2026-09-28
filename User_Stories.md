# User Stories and Use Cases — Concentrate AI Model Recommendation System

**Team:** Allison Oh, Aditya Pawar

## Stakeholder Map

| Category | Stakeholder | Why they're in this category |
|---|---|---|
| Primary | Product Manager (client company) | Directly uses the system to get and act on a model recommendation |
| Secondary | Concentrate AI Solutions Engineer | Doesn't request recommendations themselves, but reviews and adjusts them before a client sees the result |
| Hidden | Client IT Security Reviewer | Never opens the recommendation report, but must approve which vendors may receive the client's prompts before any benchmark can run |
| Hidden | Concentrate AI Platform Operator | Only surfaces after launch, when vendor API rate limits and outages start affecting benchmark runs in production |

## User Stories

**US-01** (primary)
As a product manager evaluating AI vendors for my company,
I want to receive a ranked shortlist of LLM providers that fit my budget and latency needs,
so that I can choose a model without running my own benchmarking project.

**US-02** (primary)
As a product manager evaluating AI vendors,
I want to see the specific reasoning behind each model's ranking,
so that I can defend the choice to my own engineering leadership.

**US-03** (secondary)
As a Concentrate AI solutions engineer,
I want to annotate or override a generated recommendation before it reaches a client,
so that I can catch cases the automated benchmarking missed.

**US-04** (hidden — compliance)
As a client IT security reviewer,
I want to see which third-party vendors will receive any of our submitted prompts before a benchmark runs,
so that I can approve the vendor list against our company's data-handling policy.

**US-05** (hidden — downstream system / operations)
As a Concentrate AI platform operator,
I want to be notified when a vendor API's rate limit is close to being exceeded during a benchmark run,
so that I can throttle requests before customer-facing results are delayed or fail.

## INVEST Self-Check

I = Independent, N = Negotiable, V = Valuable, E = Estimable, S = Small, T = Testable. All five stories pass all six checks; the notes column records what was cut, split, or reworded to get there.

| ID | I | N | V | E | S | T | Notes |
|---|---|---|---|---|---|---|---|
| US-01 | Y | Y | Y | Y | Y | Y | Split from an earlier "see recommendations and reasoning" story so ranking and reasoning could be scoped separately |
| US-02 | Y | Y | Y | Y | Y | Y | Depends on US-01's output existing, but delivers value on its own once a shortlist exists |
| US-03 | Y | Y | Y | Y | Y | Y | Originally "add an edit button to the recommendation screen"; reworded to name the goal (catching missed cases) instead of the UI element |
| US-04 | Y | Y | Y | Y | Y | Y | Cut from a larger "manage account settings" story that mixed vendor approval with billing and user roles |
| US-05 | Y | Y | Y | Y | Y | Y | No dependency on the other four stories; testable against a fixed rate-limit threshold |

## Use Cases

### UC-01: Generate Model Recommendation

**Expands:** US-01

**Primary actor:** Product Manager (client company)

**Secondary actors:** Recommendation Engine (system), Vendor LLM APIs (e.g., OpenAI, Anthropic), Concentrate AI Solutions Engineer

**Preconditions:**
1. The client's account has an approved vendor list on file (see UC-02).
2. The product manager is logged into an account with an active Concentrate AI subscription.

**Main success flow:**
1. Product manager submits a project brief specifying budget ceiling, latency requirement, and a task description.
2. System validates that all three required fields are present and rejects the brief otherwise.
3. Product manager confirms submission.
4. System dispatches sanitized sample prompts to each vendor on the client's approved vendor list.
5. System collects each vendor's benchmark response, recording latency, cost per call, and a quality score.
6. System ranks the candidate models by a weighted score against the stated budget and latency constraints.
7. System presents a ranked shortlist to the product manager, with the reasoning behind each ranking.
8. Product manager selects a model from the shortlist.
9. System records the selection and generates a summary report tied to the client's account.

**Alternate flow (solutions engineer review):** After step 7, before the product manager sees the shortlist, a Concentrate AI solutions engineer may open the same shortlist, annotate or reorder up to two entries, and release it; the product manager then resumes at step 8 with the annotated version.

**Exception flow (vendor rate limit):** During step 4 or 5, if a vendor API returns a rate-limit error for more than 2 consecutive requests, the system pauses requests to that vendor for at least 60 seconds, marks that vendor "delayed" in the in-progress shortlist, and continues benchmarking the remaining vendors rather than failing the whole run.

**Postcondition:** A recommendation report naming the selected model and its justification is stored on the client's account, and the vendor API usage log is updated with the calls made during the run.

**Acceptance criteria:**

AC-01.1 Given an approved vendor list and a complete project brief with budget and latency values,
When the product manager submits the brief,
Then the system returns a ranked shortlist of at least 3 candidate models within 90 seconds, each with a numeric fit score from 0-100.

AC-01.2 Given an incomplete project brief missing the budget or latency field,
When the product manager attempts to submit it,
Then the system rejects the submission and names the specific missing field, without dispatching any prompts to vendor APIs.

AC-01.3 Given a benchmark run in progress,
When a vendor API returns a rate-limit error (HTTP 429) for 2 consecutive requests,
Then the system pauses requests to that vendor for at least 60 seconds and flags that vendor as "delayed" in the shortlist within 5 seconds of the second failure, while the remaining vendors' benchmarks continue.

### UC-02: Approve Vendor List for Benchmark

**Expands:** US-04

**Primary actor:** Client IT Security Reviewer

**Secondary actors:** Recommendation Engine (system), Product Manager (client company, notified on completion)

**Preconditions:**
1. The client's account exists in the system.
2. The reviewer has been granted reviewer-level access to that client account.

**Main success flow:**
1. IT security reviewer opens the vendor approval screen for their organization.
2. System displays every vendor Concentrate AI could route prompts to, each linked to that vendor's published data-retention policy.
3. Reviewer selects which vendors are approved to receive their organization's prompts.
4. System validates that at least one vendor is selected.
5. Reviewer submits the approved list.
6. System stores the approved vendor list, time-stamped as effective immediately.
7. System notifies the product manager on the account that an approved vendor list is now active.

**Alternate flow (time-limited approval):** At step 5, the reviewer sets a 90-day expiration on the approval instead of an indefinite one; the system schedules a re-approval reminder to the reviewer 7 days before expiration and treats the list as active in the interim.

**Exception flow (zero vendors selected):** At step 4, if the reviewer submits with no vendors selected, the system rejects the submission, displays "at least 1 vendor required" on the approval screen, and leaves the client account without an active approved vendor list, which blocks UC-01 from dispatching any prompts for that account.

**Postcondition:** The client account has a stored, time-stamped, approved vendor list that the recommendation engine checks before dispatching any prompt under UC-01.

**Acceptance criteria:**

AC-02.1 Given a client account with reviewer access granted,
When the IT security reviewer submits a list of 1 or more approved vendors,
Then the system stores the list with a timestamp and makes it active for benchmarking within 1 minute.

AC-02.2 Given the vendor approval screen is open,
When the reviewer submits the form with zero vendors selected,
Then the system rejects the submission, displays the message "at least 1 vendor required," and the client account remains without an active approved vendor list.
