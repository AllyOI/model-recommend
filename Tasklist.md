# Tasklist.md

**Project:** Model Recommend (Concentrate AI) · **Team:** Allison Oh, Aditya Pawar

Module names follow *Project-Description.md* and *Design_Diagrams* D0–D2 (components, interfaces I1–I4, algorithms A1–A2, endpoints E1–E12, figures Fig D2-1 to D2-5).

T1. Obtain the vendor API keys, the identity-provider tenant, and the managed PostgreSQL and container hosting from Concentrate AI for the Recommendation Server and Report Store. Owner: Aditya Pawar.

T2. Research the rate-limit headers, token-usage fields, prices and benchmarking terms of the OpenAI and Anthropic APIs for the Recommendation LLM adapter (I3). Owner: Aditya Pawar.

T3. Develop the Report Store schema, migrations and repository methods (Fig D2-1, I4) in PostgreSQL. Owner: Allison Oh.

T4. Develop the Configuration and Logging glue components (getConfig, structured logging, VendorCall usage log). Owner: Allison Oh.

T5. Develop the Authentication glue component (JWT verification, AppUser roles, requireRole) for the Recommendation Server. Owner: Aditya Pawar.

T6. Develop the Recommendation LLM adapter callVendor for OpenAI and Anthropic, with cost computed from token counts (I3). Owner: Aditya Pawar.

T7. Develop A1 parallel benchmark dispatch with per-vendor rate-limit handling and the rate_limit_warning in the Recommendation Server. Owner: Aditya Pawar.

T8. Develop A2 weighted scoring, ranking and the reasoning template in the Recommendation Server. Owner: Aditya Pawar.

T9. Develop the run lifecycle (Fig D2-3) and endpoints E1, E5 and E6 in the Recommendation Server. Owner: Aditya Pawar.

T10. Develop the Notifications glue component (outbox, dispatcher, approval reminder job; Fig D2-5). Owner: Allison Oh.

T11. Develop the vendor-approval, review and selection endpoints E2–E4 and E7–E12 in the Recommendation Server. Owner: Allison Oh.

T12. Develop the Recommendation Client brief form, run progress and shortlist screens S1–S3 (Fig D2-2). Owner: Allison Oh.

T13. Develop the Recommendation Client vendor-approval, review-queue and notification screens S4–S6. Owner: Allison Oh.

T14. Test the Recommendation Client and Recommendation Server end to end against AC-01.2, AC-02.1 and AC-02.2 and the edge cases EC-01 to EC-46. Owner: Allison Oh.

T15. Validate AC-01.1 (90-second shortlist) and AC-01.3 (rate-limit pause) under real vendor load and refine the A1 defaults. Owner: Aditya Pawar.
