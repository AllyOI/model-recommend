# Run Sheet and Rehearsal Log: Fall Design Presentation

**Project:** Model Recommend · **Team:** Allison Oh, Aditya Pawar · **Course:** CS5001 Senior Design\
**Deck:** `ModelRecommend_Presentation_Final.pptx` / `.pdf` (12 slides plus 3 backup slides) · **Limit:** 10 minutes of presenting, then 5 minutes of questions

## 1. Speaking Roles

| Member | Role | Slides | Planned time |
|---|---|---|---|
| Aditya Pawar | Opens with the problem, presents the design and progress, closes and hands off to questions | 1, 2, 6, 8, 9, 12 | 4:00 |
| Allison Oh | Presents the goal, team and abstract, constraints, division of work and Expo demo | 3, 4, 5, 7, 10, 11 | 4:30 |
| **Timekeeper: Allison Oh** | Keeps the clock and signals her partner with a hand sign: at **4:30** we should be on slide 7, at **8:00** on slide 11, at **9:30** wrap up at once | | |

The planned talk is 8:30, which leaves a 1:30 buffer inside the 10-minute limit. The speaking-time difference is 30 seconds.

## 2. Slide Run Sheet

| Slide | Title | Speaker | Start | Sec | Hand-off or bridge line (last line on the slide) |
|---|---|---|---|---|---|
| 1 | Model Recommend | Aditya | 0:00 | 20 | "We're building this with Concentrate AI, so let's start with the problem." |
| 2 | Purpose | Aditya | 0:20 | 45 | "So picking a model is guesswork today. **Allison** will show how we fix that." |
| 3 | Goal | Allison | 1:05 | 60 | "Here's who's behind it." |
| 4 | People | Allison | 2:05 | 25 | "And here's the whole project in a hundred words." |
| 5 | Project Abstract | Allison | 2:30 | 45 | "**Aditya** will show who this is for and how we designed it." |
| 6 | User Stories and Design Diagrams | Aditya | 3:15 | 75 | "A design like this lives inside four constraints. **Allison** will take those." |
| 7 | Major Project Constraints | Allison | 4:30 | 65 | "With those limits in mind, **Aditya** will show what we've already built." |
| 8 | Our Progress | Aditya | 5:35 | 45 | "Here's where that sits on the road to April." |
| 9 | Timeline and Expected Accomplishments | Aditya | 6:20 | 45 | "**Allison** will show how we split that work." |
| 10 | Division of Work | Allison | 7:05 | 30 | "And here's what you'll see from us at the Expo." |
| 11 | Expected Demo at Expo | Allison | 7:35 | 45 | "**Aditya** will close us out." |
| 12 | Questions? | Aditya | 8:20 | 10 | "Thank you. We'd be happy to take questions." |
| B1–B3 | Backup: Speakers and Timing; D2 Data Model; D2 Run Lifecycle | Owner of the question | Q&A | — | Shown only if a question needs them |

## 3. Anticipated Questions

The two questions marked ★ are ones we cannot fully answer yet. For those we say so and give the next step.

| # | Likely question | Owner | Short answer |
|---|---|---|---|
| Q1 | How do you decide which model is best for a customer? | Aditya | We benchmark models ahead of time by task type, classify the customer's workload from their logs, and score each model for that mix on cost, speed and quality (slide 3; REQ-01, REQ-02). |
| Q2 ★ | Where does the labeled data for the workload classifier come from? | Aditya | Not resolved yet. Next step: compare simple rule-based task tags with a classifier on the pilot logs and choose one by December 11. |
| Q3 | How do you protect customer data? | Allison | Prompts are redacted before any vendor call (NFR-09) and go only to vendors the customer approved (REQ-04, CON-02). We start with customers who opt in. |
| Q4 | Do vendor terms let you compare their models? | Allison | We follow each vendor's benchmarking terms. Results stay accurate and internal to the customer and are never published as head-to-head claims (CON-04). |
| Q5 | What if switching models mid-task loses the cache? | Aditya | We recommend at task or session boundaries, not mid-task. If confidence is low, the answer is "keep current model". |
| Q6 | How will you know a recommendation was right? | Allison | The Test Plan checks a shortlist returns within 90 s (NFR-05) and that the top pick matches a reference ranking on held-out workloads (AC-01.1, AC-01.3). |
| Q7 ★ | How do you measure answer quality on a customer's own tasks? | Aditya | Today we use GPQA Diamond, with a repeated run to measure noise. Task-specific quality checks are open. Next step: pick graders per task type after the December runs. |
| Q8 | Why compare against OpenRouter? | Aditya | It's the largest model marketplace and serves the same models, so running the same request through both gateways isolates the gateway's own cost and speed. |
| Q9 | What does benchmarking cost, and who pays? | Aditya | Concentrate AI approved $20 a day for the pilot, with a hard cap in the scheduler. Spend is metered per run (CON-01, NFR-07). |
| Q10 | New models come out every week. How do you keep up? | Allison | Benchmarks re-run when a model is added, and scheduled runs repeat three times a day. |
| Q11 | How is the work split, and what if someone falls behind? | Allison | 118 h vs 119 h in the effort matrix. We log hours weekly and rebalance at each milestone (slide 10). |
| Q12 | What exactly will the Expo show? | Allison | One sample workload, with monthly cost and run time before and after following the recommendation. A recorded run is the backup (slide 11). The poster goes to print in early March. |

## 4. Rehearsal Log

| # | Date | Format | Who | Total | Per section | Notes |
|---|---|---|---|---|---|---|
| R1 | Nov 18, 2026 | Full timed run, recorded in Teams (`ModelRecommend-Presentation-Trimmed.mp4`) | Allison Oh, Aditya Pawar | **8:15** | Opening, slides 1–2: 1:03 · Goal, team and abstract, slides 3–5: 2:03 · Design and constraints, slides 6–7: 2:30 · Progress and plan, slides 8–11: 2:33 · Close, slide 12: 0:06 | 1:45 under the limit. Aditya spoke 3:48 and Allison 4:27, a 39 s gap. Slide 6 ran 78 s and slide 7 ran 72 s. After this run we revised slides 3, 9 and 11 and added backup slides B1–B3. |
| R2 | | Full timed run on the final deck | | | | To do after the deck freeze: confirm the times in section 2 and the split within 30 s. |

Per-slide times for R1 were read from slide changes in the recording, to within about 3 s: 18, 45, 54, 24, 45, 78, 72, 45, 36, 33, 39, 6 seconds.

## 5. Backup Copy

| Item | Plan | Done |
|---|---|---|
| USB drive | `ModelRecommend_Presentation_Final.pptx` and `.pdf` | ☐ |
| Offline cloud copy | Both files in the shared OneDrive folder, marked "Always keep on this device" | ☐ |
| Tested on another machine | Open the PPTX on a computer that isn't ours and check fonts and images | ☐ |
