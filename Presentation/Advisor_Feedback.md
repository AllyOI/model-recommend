# Advisor Feedback: Draft Presentation

**Project:** Model Recommend · **Team:** Allison Oh, Aditya Pawar · **Course:** CS5001 Senior Design\
**Advisor:** Dr. Jillian Aurisano (Department of Computer Science, UC)

## 1. Meeting Record

| Item | Detail |
|---|---|
| Date | November 17, 2026 |
| Duration | About 30 minutes |
| Attendees | Dr. Jillian Aurisano, Allison Oh, Aditya Pawar |
| Format | In person; Aditya presented the draft deck, followed by open discussion |
| Version reviewed | `ModelRecommend_Presentation_Draft.pptx` (12 slides), draft of November 17, 2026; committed in `Presentation/` (hash in the submission comment) |

## 2. Feedback Log

Responses: **Accepted**, **Modified** (accepted with changes) or **Declined** (with a reason). Slide numbers refer to the reviewed draft.

| # | Slides | What the advisor said | Response | Change made / planned | Owner | Status |
|---|---|---|---|---|---|---|
| F1 | 3, 9 | Asked whether recommendations come from benchmarks run in advance on similar kinds of data and then combined, or from sampling the customer's current data. | Accepted | We will benchmark in advance on representative task types and use customer logs only to classify the workload. To be shown on slide 3 and in the D2 algorithm description. | Both | Open |
| F2 | 3, 7 | Benchmarking live customer data spends tokens and limits how often and how much we can test. Testing in advance avoids that, but benchmarks must be re-run when new models come out. | Accepted | Pre-benchmark per task type. Re-run a model's benchmarks when it is added. Use the gateway pilot results as the first baseline. | Aditya | Open |
| F3 | 7 | Do not expose customer information if benchmarks touch customer data. | Accepted (already constraint CON-02) | Keep redaction before any vendor call and benchmark on synthetic or public prompts. Start only with customers who opt in. | Allison | Open |
| F4 | 3 | On our idea of a classification model that reports a workload mix (for example 60% coding, 40% planning): asked where the labeled data for it would come from. | Modified | Evaluate the classifier as one option alongside simple rule-based task tagging. Pick one after the December benchmark runs and record where its labels come from. | Aditya | Open |
| F5 | 3 | On losing the cache when switching models mid-task: users should be clearing context and keeping specification files. If their context becomes a spec, switching gets easier, so this may matter less than we think. | Accepted | Recommend at task or session boundaries, not mid-task. Add this as a design assumption in D2. | Aditya | Open |
| F6 | 3, 11 | Raised confidence, and the possibility of the system deciding not to switch. | Accepted | Add a confidence value and a "keep current model" outcome to the A2 scoring step. | Aditya | Open |
| F7 | 11 | Other tools may build this in and route automatically inside the API, so the "model" behind an API may already be several models. | Modified | v1 stays a recommendation that the customer approves. Automatic routing is listed as future work. | Both | Open |
| F8 | 9, 11 | Keep the Expo schedule in mind: the Expo is in April, but posters are usually handed off for printing in March. | Accepted | Plan the Expo around one headline number (monthly cost and run time before and after following the recommendation, on one sample workload) and have it ready for the March poster handoff. Revise slides 9 and 11 for the final deck. | Allison | Open |

## 3. Summary of Changes

Dr. Aurisano's questions centered on how the recommendation is actually made, which is the least developed part of our design. We are taking the approach she outlined: benchmark models in advance on representative task types, use a customer's logs only to classify their workload, and re-benchmark when new models arrive. That keeps token costs predictable and keeps customer prompts out of vendor calls. We will also add a confidence value, an explicit "keep current model" outcome and the assumption that switches happen at task boundaries. For the final deck we will rebuild the Expo slide around one before-and-after cost figure and plan to have it ready for the March poster handoff. No slides were changed for this submission; every change is tracked as an open item below.

## 4. Open Items

| Item | Feedback | Owner | Expected close |
|---|---|---|---|
| Do a test recording to check audio | Assignment | Aditya | Nov 21, 2026 |
| Add the speaker and timing backup slide (outline in `Slide_Outline.md`) | Assignment | Allison | Nov 24, 2026 |
| Revise slide 3 (how recommendations are made) and slide 11 (before/after cost demo) | F1, F8 | Allison | Nov 24, 2026 |
| Get Allison access to the development environment | Meeting next step | Aditya | Nov 24, 2026 |
| Run model and gateway benchmarks (Thanksgiving week and first week of December) | F2 | Aditya | Dec 4, 2026 |
| Decide the workload classification method and record its data source | F4 | Aditya | Dec 11, 2026 |
| Add confidence, the "keep current model" outcome and the task-boundary assumption to D2 | F5, F6 | Aditya | Dec 11, 2026 |
| Record the final video and post the deck and video links | Assignment | Aditya | Week 13 |
| Expo poster with the before/after cost figure, ready for printing handoff | F8 | Both | March 2027 |
