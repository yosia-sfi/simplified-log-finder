Using AI-DLC, start the INCEPTION phase for the following project. Follow the Inception flow below exactly: Context → Discover → Specify → Design → Plan → Human Review Gate. Do not write application code. Principle: "AI proposes, humans decide."

# 0. How to work (read first)

## 0.1 Flow and gates
Run five stages in order. At the end of EACH stage, stop and wait for my explicit approval before the next one. After Plan, run the HUMAN REVIEW GATE: present the full inception package for approval before anything moves to Construction.

| # | Stage | Purpose | Deliverable(s) |
|---|---|---|---|
| 1 | Context | Understand system and business | Business context and system context |
| 2 | Discover | Find ambiguity and questions | Actors, constraints and assumptions; clarification questions |
| 3 | Specify | Requirements with acceptance criteria | Requirements with traceable acceptance criteria |
| 4 | Design | Architecture and impacts | Architecture proposal and impacted components; data model / API changes where applicable |
| 5 | Plan | Implementation tasks | Implementation plan and open decisions |

If your built-in AI-DLC workflow uses different stage names, treat them as equivalent: Context ≈ workspace detection and context building; Discover ≈ requirements clarification; Specify ≈ requirements and user stories; Design ≈ application design; Plan ≈ workflow planning and units of work generation.

## 0.2 Rules
- Intent first, specification before construction. No code in this phase.
- Do not decide silently. Every gap becomes either a question or an explicit, labeled assumption that I must confirm.
- Ground every statement in a source I gave you. Do not invent APIs, endpoints, policies, product features, or numbers. If unsure, mark it "TBD – needs confirmation".
- Put clarification questions in a separate file: multiple-choice options plus "Other", each with an `[Answer]:` tag. Group by topic. Tag each as Blocker or Can-defer, and suggest an owner role (BA, System Architect, DevOps, QA, PM).
- Keep a decision log in `aidlc-docs/audit.md` (decision, rationale, who decided, date). If a later answer contradicts an earlier decision, flag it and ask which one stands.
- Bounded autonomy: you MAY analyze, draft, and propose. You MUST ask before deciding anything about security, data access, hosting, scope, or what is rejected below. You may NOT assume access to any system.
- Before each stage gate, run this self-check and show the result: (1) every acceptance criterion is testable; (2) every rule traces to a source; (3) design respects the stack, security and scope limits; (4) it reuses existing systems, roles and data where possible; (5) evidence of the check is shown; (6) open questions are flagged, not silently decided.
- Write artifacts in English. Keep Indonesian business terms and real request wording verbatim (they are test data).

# 1. Context brief (the five inputs)

## 1.1 CONTEXT: what exists, who it serves, why it matters
Project: Simplified Log Finder. Organization: Soluix (IT solution provider; banking clients, e.g. BCAS). Team: "The Investor's Team" (11 people): Project Advisor (Yosia Alvin); Project Managers (Ingwen Tannaris, Laurence Benedicta); System Architect (William Tanardi); Business Analysts (Ayuningtyas, Bernhard Praditya); Developers (Albert Rafer, Daniel Riyanto, Michael Christian); QA (Bimas Nugraha); DevOps (Hadi Septian).

Product: a greenfield, internal, self-serve web app in front of the log sources that already exist. It is NOT a new observability platform.

Problem (validated by the team):
- Symptom: BA, QA and other non-technical teammates keep pinging engineers to pull logs. 24 of 24 sampled chat captures are the same ask; only the time, the identifier and the feature change. Each ping interrupts an engineer who can open Grafana, the database or k9s. Raw logs are dense JSON that is hard to read.
- Root cause (5 Whys): there is no standard, self-serve path from a non-technical question to a readable log. Lookup is treated as a favor, not a capability with an owner. Access and query skill sit with a few people.
- Impact (estimates, to be baselined): more than 10 pings per day in high-load periods (UAT/regression); 5–10 engineer minutes per lookup, excluding troubleshooting.

Expected behavior: a non-technical teammate finds and reads the log themselves by time, identifier, service or error code, sees a simplified view with the raw log one click away, and engineers step in only when the lookup cannot be resolved.

Pilot hypothesis: if one real project team uses the finder for everyday lookups, engineer log-request pings fall by 65% and requesters get a readable log in under 2 minutes. Pilot setup: one real team (not a demo), baseline from the last 3 months of log-request chats, and behavior measurement (did they search themselves first; filters vs natural language). Decision after the pilot: target met → roll out; partly met → adjust and rerun; no change → revisit the root cause.

## 1.2 Decisions already made by the team (do not reopen without strong reason)
- Mechanism: Simplified Log Finder, manual filters first, natural-language search on top.
- Rejected: giving BA/QA direct Grafana / database / k9s access (security risk; logs stay unreadable).
- Support only: runbooks/training; shared dashboards and saved queries.

## 1.3 Proposed direction that needs my approval: hybrid "build vs reuse"
We want to reuse existing technology through its APIs instead of building new core logic. Proposal: the finder is a thin orchestration layer that calls the existing Loki HTTP API, and builds only what is missing. In Discover, challenge this proposal and present options with trade-offs (reuse as-is, hybrid, replace stack, build from scratch).

Candidate technologies reviewed so far (verify claims; do not treat as guaranteed for our environment):
- Loki HTTP API: query a time range with LogQL (query_range), list labels and label values; multi-tenancy via the `X-Scope-OrgID` header; authentication is outside Loki's scope, so a gateway or proxy is needed.
- Grafana MCP server (open source): tools for querying logs and metrics, searching dashboards, generating deeplinks; uses a Grafana service account token with RBAC.
- Grafana Assistant: an LLM agent built into Grafana Cloud that turns natural language into queries. An assistant-cli can connect agents to it through an A2A API, but it requires a Grafana instance where Assistant is enabled. Its output is not a structured filter set, and it targets engineers.
- OpenObserve: an alternative observability platform (replacement for Loki + Grafana) with an NL assistant, an MCP server, Fluent Bit ingestion and redaction pipelines. Source is a vendor blog, so treat its cost and performance claims as marketing.

What must still be built (our differentiators): business-term dictionary per project; identifier-to-field mapping; parser that turns raw logs into a simplified view; masking and audit; database connector; non-technical UI with editable filter chips; pilot instrumentation.

## 1.4 CONSTRAINTS
- Security and compliance come first (banking clients). Logs may contain CIF, account numbers, hashcards, tokens and document content.
- The LLM must translate a sentence into structured filters only. The finder then executes predefined query templates. Free-form generated LogQL/SQL is not allowed. (Proposed; ask me to confirm.)
- Multi-project from day one in architecture; only the pilot project is onboarded first. (Proposed; ask me to confirm.)
- Must NOT replace Grafana, the database or k9s for engineers, and must NOT become a backdoor with broader raw access than intended.
- MVP scope: manual filters, natural-language search on top, formatted logs, three sources (Grafana/Loki, database, Kubernetes pod logs), applied across projects. Next phase (design must not block it): reading images, more input sources.

## 1.5 SOURCES (authoritative; follow these over your own memory)
- Team decks: idea sheet, "Investor's Team – Simplified Log Finder (AIDLC)" deck, and the presenter deck.
- Real log sample from Grafana/Loki (UAT): logs shipped via Fluent Bit with labels such as cluster, job and kubernetes_container_name. Each line wraps the application log inside a JSON "log" field, with stdout prefixes, ANSI color codes and nested escaped JSON. Application payload fields include project_name, project_module, module_version, correlation_id and log_data. correlation_id is often empty. Timestamps appear in both UTC and +0700 in the same line. Payloads can contain sensitive values.
- The 24 sampled chat requests (see section 2).
- The AI-DLC workshop deck (flow, deliverables, principles).

## 1.6 TOOLS (what you may read, run, change)
Read the uploaded documents and write only under `aidlc-docs/`. You have no access to Grafana, Loki, any database or any cluster. Do not call external systems. Do not write application code.

## 1.7 ACCEPTANCE (what "done" means for Inception)
Inception is done when: all six deliverables exist and pass the self-check; every blocker question is answered or has an owner and due date; every requirement has testable acceptance criteria that trace to a real request or a stated constraint; the plan has units of work with dependencies and an MVP slice; and I have approved the package at the human review gate.

# 2. Representative real requests (seed for acceptance criteria and NL tests)
Identifiers are placeholders. Treat as test data:
- "Get the log for change source account (ganti rek sumber) at 16:04:44, CIF <CIF>"
- "Log for card closure (penutupan kartu) at 17:23:26, CIF <CIF>"
- "Check the log for the CWLD submission (maybe external_opening_account) that failed in RC, around 15:11, CIFs <CIF1>, <CIF2>, <CIF3>"
- "UAT rejected unblock (buka blokir) at 10:28:32 for hashcard <SHA256_HASH>"
- "Card upgrade failed with program not found, program ID PROG-PHYSICAL-01"
- "Why does the balance (saldo) show a dot and account info is empty in UAT? Account <ACCOUNT_NO>, last hit around 11:22–11:23"
- "Which URL path returned 403?"
- "Did this endpoint ever get hit? /api/registration/.../generate-key"
- "H2H error again at 9:58, probably connection timeout"
- "Log for the refund", "log for the failed liveness / get token", "log for this transaction"

Patterns: (a) an exact time or window, (b) one or more identifiers (CIF, account number, hashcard, request ID, program ID, license ID, endpoint path), (c) a business feature name, often in Indonesian, (d) an environment (UAT, RC...), sometimes implicit.

# 3. Stage instructions

## Stage 1: Context
Output: business context and system context. Summarize what exists, who is served, and why it matters. Describe the current log landscape (Grafana/Loki, database, k9s: who can use each today) and the existing-technology landscape from section 1.3. List what you could NOT verify. Stop for approval.

## Stage 2: Discover
Output: actors, constraints and assumptions, plus the question file. Find ambiguities and gaps. Treat these DevOps answers as UNCONFIRMED ASSUMPTIONS (the responsible person was unavailable), never as facts: (1) Grafana is Grafana Cloud; (2) Loki is accessed through a gateway; (3) there is one Loki per project; (4) the database has no read-replica. For each, state what changes in the design if it is true or false, and ask me to confirm. Make the design robust to either answer where possible. Treat "no read-replica" as a risk: querying the primary database needs limits, timeouts and explicit approval. Stop for approval.

## Stage 3: Specify
Output: requirements with traceable acceptance criteria. Functional and non-functional requirements, each traced to a request in section 2, a root-cause statement or a constraint. Personas are PRODUCT USER ROLES, not project team roles. Minimum set:
- Business Analyst and QA Engineer (primary searchers).
- Developer/Engineer (escalation when the finder cannot resolve a lookup).
- Project Admin (a product role that configures the finder per project: business-term dictionary, identifier-to-field mapping, environments, who can access which project). Ask who should own this role in practice; it may be a BA or DevOps member.
- Project Manager (reads pilot metrics).
Write user stories with Given/When/Then acceptance criteria. Use the section 2 requests as concrete examples. Stop for approval.

## Stage 4: Design
Output: architecture proposal and impacted components; data model / API changes where applicable. Cover: build-vs-reuse options and recommendation; component view (UI, NL-to-filter layer, query builder, source connectors, parser and renderer, masking, audit); per-project configuration model; security model; how the Loki API, Grafana and the database are used; and risks. Note that Loki indexes labels, so searching an identifier inside log lines can be slow over wide time ranges; propose a mandatory time range and result limits. Cover the timezone mismatch (UTC vs +0700) and empty correlation_id. Stop for approval.

## Stage 5: Plan
Output: implementation plan and open decisions. Split work into units of work (U1, U2, ...) with dependencies and owner roles. Each unit will run as one or more Bolts (prepare → plan → build → verify → review), so give each unit its own acceptance criteria and verification evidence. Propose an MVP slice following the team's decision: foundation and access; a short Loki API spike plus manual filters on one source (using a real request from section 2); simplified and raw view; natural-language search; database and Kubernetes sources; pilot instrumentation as an explicit unit. List open decisions with owners and due dates. Stop for approval.

## Human Review Gate
Present the complete inception package, the self-check results, all unconfirmed assumptions, open blockers and the retrospective below. Do not proceed to Construction until I approve.

# 4. Mandatory clarifications (minimum)
Scope and rollout
- "Applied to all projects" vs a one-team pilot: multi-project architecture from day one with only the pilot project onboarded first?
- Which project and team run the pilot? Which environments are in scope? Is production excluded for the MVP?

Reuse and access
- Confirm or correct the four unconfirmed DevOps assumptions in Stage 2.
- Is Grafana Assistant enabled and licensed? Should an Assistant trial be deferred until after the MVP?
- May the Grafana MCP server be used only for label/data-source discovery, with query execution limited to our templates?
- Which identity and token does the finder use to reach Loki and the database, and who approves read-only access?
- How is project isolation enforced when the finder holds one service identity?
- Database: which engines, schemas and tables are relevant; what limits (timeout, row cap, time range) are acceptable on a primary database?
- Kubernetes: is direct pod-log reading needed, or does Loki already hold the same logs? How are short-lived pods handled?

Search and mapping
- Full list of identifiers and their mapping to log fields or DB columns, per project.
- Who owns the business-term dictionary (ganti rek sumber, penutupan kartu, buka blokir, refund, H2H, VKYC, liveness...) and how is it maintained per project?
- Timezone rules for input and display (users speak WIB; logs mix UTC and +0700).
- Default time window and maximum result size for vague times ("yesterday afternoon", "around 15.11").
- Behavior when correlation_id is empty and a request spans services.

Output
- Which fields make up the simplified view, and are they configurable per project?
- Is the simplified view produced by deterministic parsing, by an LLM, or both? If an LLM is involved, how do we prevent wrong content in a view people forward to clients?
- Should results be shareable by link, and should there be a deeplink to Grafana for escalation?

Security, privacy, compliance
- Who may search which project and environment?
- Masking rules for CIF, account numbers, hashcards, tokens and document content; does "Copy raw" return masked or unmasked data, and for which roles?
- May log content go to an external LLM? If not, where must it run? Any client contract or regulatory limits?
- Audit logging: what, who, and retention.

Pilot metrics
- How are log-request pings counted (chat export, keyword tagging, manual log)? How is time to a readable log measured? What counts as self-served vs escalated?

Non-functional
- Users, concurrent searches, response-time target (readable log in under 2 minutes end to end), availability, hosting, tech stack constraints, SSO.

# 5. Inception retrospective (record at the gate)
Report for this Inception run: Speed (time from brief to approved inception package), Clarity (requirement gaps you raised before any code), First-pass quality (how much of your first draft was accepted unchanged), Rework (times I corrected or re-prompted you), Human effort (hours spent by the team), Confidence (1–5 to start Construction). Also note what worked, what failed, and where human judgment was essential.

Begin with Stage 1: Context. Present the context summary and the list of what you could not verify, then wait for my approval before Discover.