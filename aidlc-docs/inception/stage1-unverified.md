# Stage 1: What Could Not Be Verified

AI-DLC Inception, Stage 1 (Context). Status: **Approved with corrections 2026-11-25** (see `audit.md` A-006 to A-012).
Items marked **Resolved** or **Partly resolved** use the user's answer of 2026-11-25. Items marked **Deferred to Stage 4** are verified by the technical team during Design. All other items are **unchanged and carried into Stage 2** (`stage2-questions.md`). Nothing unresolved is used as fact in later stages.

## A. Source documents
Originally only `prompt-inception.md` existed. **Updated 2026-11-25:** the user committed three reference files (commit `ac124f7`). Review and evidence: `reference-docs-review.md`.
1. Idea sheet. **Resolved 2026-11-25:** `Ideation - Simple Log Finder.pdf`.
2. "Investor's Team – Simplified Log Finder (AIDLC)" deck. **Unclear 2026-11-25:** may be the Presenter's Deck (cover reads "Simplified Log Finder, Built with AIDLC, AIDLC – The Investor's Team"). Asked in Stage 2 (`SRC-02`, owner PM).
3. Presenter deck. **Resolved 2026-11-25:** `Presenter's Deck - Investor's Team - Simplified Log Finder.pdf`.
4. AI-DLC workshop deck. **Resolved 2026-11-25:** `Reference AI DLC Workshop v3.pptx`. Matches the brief; metric wording differs for Speed and Human effort, and the brief [§5] governs.
5. The real Grafana/Loki UAT log sample. **Partly resolved 2026-11-25:** one Grafana Explore screenshot [Ideation p.17]. Still needed: a masked text export of a few raw lines and the full common-label list (owner DevOps or a Developer).
6. The full 24 sampled chat requests. **Resolved 2026-11-25:** [Ideation p.4–11], also [Presenter p.4–5].
7. The intermediate steps of the 5-Whys analysis. **Resolved 2026-11-25:** [Presenter p.6].

## B. Infrastructure and access
8. Grafana is Grafana Cloud or self-hosted.
   **Resolved 2026-11-25 (user):** Grafana is **self-hosted**. The DevOps assumption "Grafana Cloud" is false. Consequence to verify in Stage 4: the brief describes Grafana Assistant as built into Grafana Cloud, so it is likely unavailable to us (see item 19).
9. Whether Loki is reached through a gateway/proxy, and how it authenticates.
   **Deferred to Stage 4 technical verification (user, 2026-11-25).** Owner: DevOps (Hadi Septian); backup: System Architect (William Tanardi). Blocks the Loki connector spike/build. Suggested checks: `stage2-actors-constraints-assumptions.md` §6 "How to check Loki access".
10. Loki topology: one Loki per project, shared Loki with tenants (`X-Scope-OrgID`), or other. Unchanged: working assumption "one per project", verify in Stage 4.
11. Database engines, schemas, tables, and read-replica availability. Unchanged: working assumption "no read-replica", treated as a risk, verify in Stage 4.
12. Kubernetes clusters/namespaces, and whether Loki already holds the same pod logs. Unchanged, carried into Stage 2.
13. Who can use Grafana, the database and k9s today, by name, account or RBAC role.
   **Partly resolved 2026-11-25 (user):** only **Developers and DevOps** can use Grafana. Who can use the database and k9s is still unknown (carried into Stage 2). Named accounts and RBAC roles are still unknown.
14. Which identity/token the finder would use, and who approves read-only access. Unchanged, carried into Stage 2.
15. Environments in scope.
   **Resolved 2026-11-25 (user):** only **UAT and RC** are in scope. Production and any other environment are out of MVP scope.
16. Hosting location, SSO/identity provider, tech stack constraints.
   **Partly resolved 2026-11-25 (user):** hosting is **"local first"**. What "local" means (developer laptop, internal Soluix server, or inside the UAT/RC cluster/network) is still open and is a Stage 2 Blocker question. SSO/identity provider and tech stack constraints: unchanged, carried into Stage 2.

## C. Technology claims [§1.3]
17. Loki HTTP API capabilities (`query_range`, labels, label values, `X-Scope-OrgID`) as they behave in our Loki version. Unchanged: verify in Stage 4.
18. Grafana MCP server capabilities and the RBAC scope of a service account token in our **self-hosted** Grafana. Unchanged: verify in Stage 4.
19. Grafana Assistant: enabled and licensed for us; A2A API availability. Updated: with Grafana self-hosted, availability is likely "no" per the brief's description; verify in Stage 4.
20. OpenObserve capability, cost and performance claims (vendor blog source). Unchanged.

## D. Log data
Status: carried into Stage 2. **Updated 2026-11-25** with partial evidence from the log screenshot [Ideation p.17] (see `reference-docs-review.md` §3). Screenshot evidence is re-verified in Stage 4.
21. Whether any identifier (CIF, account number, hashcard, etc.) exists as a Loki label or only inside log lines. **Partial evidence:** visible labels are `cluster`, `job`, `kubernetes_container_name` (+3 hidden); no identifier label visible. Still open until the full label list is provided.
22. Exact timestamp formats and which one is authoritative (UTC vs +0700). **Partial evidence:** each line carries a Loki/CRI UTC timestamp (`Z`) and an app timestamp with `+0700`; Grafana displays local time. Which is authoritative: still open (`M-05`).
23. How often `correlation_id` is empty, and how requests spanning services can be linked. **Partial evidence:** empty in every visible line; some payloads carry `X-Request-Id` and `traceparent` headers as a possible link (`M-09`). Frequency still unknown.
24. The full list of sensitive field types and where they appear in payloads. **Partial evidence:** base64 document body (`BodyRaw`), document numbers, document template IDs, S3 bucket and object keys, ETag seen in `log_data`. Full list still open (`SEC-02`).

## E. Business numbers and pilot
Status: unchanged, carried into Stage 2.
25. Baseline of pings per day (estimate: more than 10 in high-load periods, to be baselined)
26. Engineer minutes per lookup (estimate: 5–10, to be baselined)
27. Pilot project and team
28. Method to count pings and measure time to a readable log
29. Thresholds for "partly met" vs "no change" in the post-pilot decision rule

## F. Policy and compliance
Status: unchanged, carried into Stage 2. Item 30 is now linked to the hosting question (item 16).
30. Whether log content or user sentences may go to an external LLM; client contract or regulatory limits (e.g. for BCAS)
31. Masking rules and whether "Copy raw" is masked, per role
32. Audit requirements and retention
33. Owner of the Project Admin role and the business-term dictionary

## G. Scope wording
34. §1.4 says the MVP is "applied across projects", while §1.1 and constraint C3 describe a one-team pilot. Which one governs MVP rollout. Unchanged: contradiction X-001, asked in Stage 2.
