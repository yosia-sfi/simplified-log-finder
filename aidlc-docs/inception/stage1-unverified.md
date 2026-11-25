# Stage 1: What Could Not Be Verified

AI-DLC Inception, Stage 1 (Context). Everything below is **unverified**. None of it is used as fact in later stages until confirmed. Questions with options, owners and Blocker/Can-defer tags are produced in Stage 2 (Discover).

## A. Source documents not available in the workspace
Only `prompt-inception.md` exists. The following are referenced as authoritative [§1.5] but were not provided as files:
1. Idea sheet
2. "Investor's Team – Simplified Log Finder (AIDLC)" deck
3. Presenter deck
4. AI-DLC workshop deck
5. The real Grafana/Loki UAT log sample (only its description is in the brief)
6. The full 24 sampled chat requests (only 10 representative ones are in the brief)
7. The intermediate steps of the 5-Whys analysis

## B. Infrastructure and access
8. Grafana is Grafana Cloud or self-hosted (DevOps answer "Grafana Cloud" is an unconfirmed assumption)
9. Whether Loki is reached through a gateway/proxy, and how it authenticates (unconfirmed assumption: gateway)
10. Loki topology: one Loki per project, shared Loki with tenants (`X-Scope-OrgID`), or other (unconfirmed assumption: one per project)
11. Database engines, schemas, tables, and read-replica availability (unconfirmed assumption: no read-replica)
12. Kubernetes clusters/namespaces, and whether Loki already holds the same pod logs
13. Who can use Grafana, the database and k9s today, by name, account or RBAC role
14. Which identity/token the finder would use, and who approves read-only access
15. Environments in scope (UAT, RC named in requests; others and production status unknown)
16. Hosting location, SSO/identity provider, tech stack constraints

## C. Technology claims [§1.3]
17. Loki HTTP API capabilities (`query_range`, labels, label values, `X-Scope-OrgID`) as they behave in our Loki version
18. Grafana MCP server capabilities and the RBAC scope of a service account token in our Grafana
19. Grafana Assistant: enabled and licensed for us; A2A API availability
20. OpenObserve capability, cost and performance claims (vendor blog source)

## D. Log data
21. Whether any identifier (CIF, account number, hashcard, etc.) exists as a Loki label or only inside log lines
22. Exact timestamp formats and which one is authoritative (UTC vs +0700)
23. How often `correlation_id` is empty, and how requests spanning services can be linked
24. The full list of sensitive field types and where they appear in payloads

## E. Business numbers and pilot
25. Baseline of pings per day (estimate: more than 10 in high-load periods, to be baselined)
26. Engineer minutes per lookup (estimate: 5–10, to be baselined)
27. Pilot project and team
28. Method to count pings and measure time to a readable log
29. Thresholds for "partly met" vs "no change" in the post-pilot decision rule

## F. Policy and compliance
30. Whether log content or user sentences may go to an external LLM; client contract or regulatory limits (e.g. for BCAS)
31. Masking rules and whether "Copy raw" is masked, per role
32. Audit requirements and retention
33. Owner of the Project Admin role and the business-term dictionary

## G. Scope wording
34. §1.4 says the MVP is "applied across projects", while §1.1 and constraint C3 describe a one-team pilot. Which one governs MVP rollout.
