# Stage 2 (Discover): Clarification Questions

AI-DLC Inception, Stage 2. Status: **Approved 2026-11-25** (all questions answered, including section 13: C-05 = 1, C-07 = 1, C-08 = 2; see `audit.md` entry "Stage 2 approved"). Decisions are logged in `audit.md` (entry "Stage 2 answers recorded"). This file is the requirements decision file for spec `simplified-log-finder`; Stage 3 (Specify) uses the answers.

> **How to answer:** fill in each `[Answer]:` line with an option number, or write your own answer under "Other". Recommended options are AI proposals; nothing is decided until you answer. You can answer "use the recommendations" for any group.
>
> **Tags:** **Blocker** = must be answered (or have an owner and due date) before Stage 3 is approved. **Can-defer** = may be answered later. **Can-defer (verify in Stage 4)** = verified by the technical team during Design; some of these block a specific build unit, as noted.
>
> **Due dates:** all original Blockers are answered, so brief §1.7 no longer requires due dates for them. Due dates apply only to Blockers that are still unanswered (as of 2026-11-25: C-05 in section 13).

## Already answered (not re-asked)

| Item | Answer | Source |
|---|---|---|
| Grafana Cloud vs self-hosted | Self-hosted | User 2026-11-25 (A-007) |
| Who uses Grafana today | Developers and DevOps only | User 2026-11-25 (A-009) |
| Environments in scope; is production excluded for MVP | UAT and RC only; production excluded | User 2026-11-25 (A-010) |
| Hosting direction | "Local first" (meaning asked in `H-01`) | User 2026-11-25 (A-011) |
| Loki access path | Unknown; verified by technical team in Stage 4 (tracked in `G-04`) | User 2026-11-25 (A-008) |

## Blocker index

| ID | Title | Owner |
|---|---|---|
| S-01 | Multi-project architecture vs one-team pilot (X-001) | PM, System Architect |
| S-02 | Pilot project and team | PM |
| H-01 | Meaning of "local first" hosting | DevOps, System Architect |
| H-03 | User login and identity provider | DevOps, System Architect |
| G-01 | Build-vs-reuse direction (P-001) | System Architect |
| G-06 | Service identity for Loki and the database, and who approves read-only access | DevOps, System Architect |
| G-07 | Project isolation with a service identity | System Architect |
| L-01 | May user sentences or log content go to an external LLM | PM, System Architect |
| L-02 | Client contract or regulatory limits | PM |
| L-03 | Confirm C2: LLM produces structured filters only (P-002) | System Architect |
| L-04 | Simplified view: deterministic parser, LLM, or both | System Architect, BA |
| M-01 | Identifier list and mapping for the pilot project | BA |
| M-03 | Owner of the business-term dictionary | PM |
| M-04 | Owner of the Project Admin role | PM |
| M-05 | Timezone rules for input and display | BA, System Architect |
| SEC-01 | Who may search which project and environment | PM, System Architect |
| SEC-02 | Masking rules | System Architect, PM |
| SEC-03 | "Copy raw": masked or unmasked, per role | System Architect, PM |
| SEC-04 | Audit logging: what, who reads it, retention | System Architect, PM |
| MET-01 | How log-request pings are counted | PM |
| MET-02 | How time to a readable log is measured | PM, QA |
| MET-03 | What counts as self-served vs escalated | PM |
| U-01 | FE and Mobile developers as searchers (added 2026-11-25) | PM, BA |
| M-10 | Requests without identifier or time (added 2026-11-25) | BA, PM |
| SEC-05 | "Copy raw" when the log is forwarded to the client (added 2026-11-25; linked to SEC-03) | System Architect, PM |
| SEC-06 | Sensitive data in the committed reference PDFs (added 2026-11-25) | PM, System Architect |
| C-01 | Default unmask and "true raw" rights (added 2026-11-25, section 12) | System Architect, PM |
| C-03 | Masked raw-log text sample timing (added 2026-11-25, section 12) | PM, DevOps |
| C-05 | Unmask / true-raw rights for BA/QA vs T-002 (added 2026-11-25, section 13) | PM, System Architect |

---

## 1. Scope and rollout

### S-01 Multi-project architecture vs one-team pilot (contradiction X-001)
- **Tag:** Blocker · **Owner:** PM, System Architect · **Due:** TBD (set by PM)
- **Question:** The brief says the MVP is "applied across projects" [§1.4] but also describes a one-team pilot [§1.1] and proposes C3. Which governs the MVP?
- **Why this matters:** It sets how many projects are onboarded in the MVP and whether per-project configuration is built now.
- **Options:**
  1. **(Recommended)** Multi-project architecture from day one; only the pilot project is onboarded in the MVP; other projects after the pilot decision. Rationale: matches C3 and the pilot hypothesis; avoids rework later.
  2. Single-project MVP; multi-project added after the pilot.
  3. Several projects onboarded in the MVP.
  4. Other: _______________
- [Answer]:1

### S-02 Pilot project and team
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** Which real project and team run the pilot?
- **Why this matters:** Identifiers, dictionary terms, Loki/DB bindings and the ping baseline all come from this project.
- **Options:**
  1. **(Recommended)** The project that produced most of the 24 sampled requests and has UAT/RC activity during the pilot window. Rationale: real requests become test data; enough traffic to measure. Name: BCAS
  2. The project with the next scheduled UAT/regression cycle. Name: _______________
  3. Other: _______________
- [Answer]:1

### S-03 UAT and RC sequencing in the pilot
- **Tag:** Can-defer · **Owner:** PM, DevOps
- **Question:** Both UAT and RC are in scope. Are both onboarded at pilot start?
- **Why this matters:** Each environment needs its own source bindings and access approval.
- **Options:**
  1. **(Recommended)** UAT first (the known log sample is from UAT), RC added within the pilot once its Loki data is confirmed (AS-05).
  2. Both at pilot start.
  3. Other: _______________
- [Answer]:1

---

## 2. Hosting and platform

### H-01 Meaning of "local first" hosting
- **Tag:** Blocker · **Owner:** DevOps, System Architect · **Due:** TBD (set by PM)
- **Question:** What does "local first" mean for where the finder runs?
- **Why this matters:** It decides whether a team pilot is possible, how the finder reaches Loki/DB/Kubernetes, where secrets are stored, and links to `L-01` (whether anything may leave the environment).
- **Options:**
  1. **(Recommended)** An internal Soluix server or VM on the same network as UAT/RC. Rationale: pilot users can reach it; secrets stay on a managed host; nothing leaves the internal network.
  2. Inside the UAT/RC Kubernetes cluster (as a workload). Network route to Loki and pod logs is likely simplest; needs cluster deployment approval.
  3. A developer laptop. Suitable for the Loki spike only; cannot serve a team pilot; secrets on a personal device.
  4. Other: _______________
- [Answer]:1

### H-02 Hosting after "local first"
- **Tag:** Can-defer · **Owner:** DevOps, System Architect
- **Question:** Is there a later target hosting (e.g. after the pilot) that the design must not block?
- **Why this matters:** Avoids designing for a local-only setup that must be rebuilt for rollout.
- **Options:**
  1. **(Recommended)** Decide after the pilot; Stage 4 keeps configuration and secrets external to the code so the host can change.
  2. Target is known now: _______________
  3. Other: _______________
- [Answer]:1

### H-03 User login and identity provider
- **Tag:** Blocker · **Owner:** DevOps, System Architect · **Due:** TBD (set by PM)
- **Question:** How do users log in, and where do roles/project membership come from?
- **Why this matters:** Access per project and environment (`SEC-01`) and audit (`SEC-04`) need a trusted user identity.
- **Options:**
  1. **(Recommended)** Soluix's existing identity provider via SSO, if one exists. Name: _______________. Rationale: no new passwords; leavers lose access automatically.
  2. Local accounts in the finder, managed by the Project Admin.
  3. Other: _______________
- [Answer]:2

### H-04 Tech stack constraints
- **Tag:** Can-defer · **Owner:** System Architect
- **Question:** Are there required languages, frameworks or runtimes for the finder?
- **Why this matters:** The team must be able to build and maintain it.
- **Options:**
  1. **(Recommended)** Use the team's existing standard stack. Name: _______________. Rationale: Developers can maintain it without new skills.
  2. No constraint; Stage 4 proposes options.
  3. Other: _______________
- [Answer]: Yes, for the backend, use go lang with echo, for the frontend, use typescript react with vite, tanstack router, tailwind, tanstack query, tanstack virtual, shadcn for UI, lucide react for icons, and use pactflow for contract testing on all sides, in the backend, frontend, and in the ci/cd pipeline. We should also confirm this with the developers later on, and confirm if there are any other libraries/frameworks needed

### R-01 Repository model
- **Tag:** Can-defer · **Owner:** System Architect
- **Question:** Is the finder built in a single repository?
- **Why this matters:** Affects Stage 5 units of work and parallel work only.
- **Options:**
  1. **(Recommended)** Single repository (provisional in `aidlc-state.md`). Rationale: small team, one product.
  2. Separate repositories (e.g. UI and backend). 
  3. Other: _______________
- [Answer]: It should live in a single repository for now, but in different directories as to ease context-switching while still enabling the repositories to split easily later on if the project's grown to a sufficient size

---

## 3. Non-functional

### N-01 Users and concurrent searches during the pilot
- **Tag:** Can-defer · **Owner:** PM
- **Question:** How many people use the finder during the pilot, and how many may search at the same time?
- **Why this matters:** Sizes limits on Loki and database load (A4).
- **Options (ranges are choices, not measured facts):**
  1. **(Recommended)** Pilot team size only (number: ___); concurrent searches = TBD by Stage 4 from that number.
  2. Fewer than 10 users.
  3. 10 to 30 users.
  4. Other: _______________
- [Answer]:2

### N-02 Response-time target
- **Tag:** Can-defer · **Owner:** PM, System Architect
- **Question:** The pilot target is a readable log in under 2 minutes end to end [§1.1]. Is a separate target needed for a single search?
- **Why this matters:** Gives Stage 3 a measurable NFR beyond the end-to-end target.
- **Options:**
  1. **(Recommended)** Keep only the end-to-end target for the MVP; Stage 4 proposes a per-search target after the Loki spike. Rationale: Loki performance on our data is unverified.
  2. Set a per-search target now: ___ seconds.
  3. Other: _______________
- [Answer]:1

### N-03 Availability
- **Tag:** Can-defer · **Owner:** PM, DevOps
- **Question:** What availability is expected during the pilot?
- **Why this matters:** Drives hosting and support effort.
- **Options:**
  1. **(Recommended)** Working hours on working days, best effort; if the finder is down, users fall back to asking engineers. Rationale: internal pilot tool.
  2. Available whenever UAT/RC testing runs, including off-hours.
  3. Other: _______________
- [Answer]:1

---

## 4. Reuse and access

### G-01 Build-vs-reuse direction (P-001)
- **Tag:** Blocker · **Owner:** System Architect · **Due:** TBD (set by PM)
- **Question:** Which option from `stage2-actors-constraints-assumptions.md` §5 do you approve?
- **Why this matters:** Sets the overall architecture for Stage 4.
- **Options:**
  1. **(Recommended)** O2 Hybrid: thin layer over the Loki HTTP API through a server-side connector with per-project source bindings; build only the differentiators. Rationale: fits C1–C5, keeps engineers' tools, works with or without a gateway.
  2. O2b Hybrid with query execution through the Grafana MCP server.
  3. O3 Replace stack, or O4 build from scratch (both conflict with C4 / "not a new observability platform").
  4. Other: _______________
- [Answer]: the simplified log finder we're building should be an input-output-agnostic tool/engine that can work with all sorts of input, and it shouldn't do anything to break or majorly modify the existing tooling in each source other than connecting them to the engine, so yes, the hybrid approach would be the best choice, but we need to keep in mind that each input source would need its own discussion session to determine which approach is best

### G-02 Grafana MCP server usage
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** System Architect, DevOps
- **Question:** May the Grafana MCP server be used only for label/data-source discovery, with query execution limited to our templates?
- **Why this matters:** Query execution through MCP tools could bypass C2.
- **Options:**
  1. **(Recommended)** Discovery only, if Stage 4 confirms it works with our self-hosted Grafana and a least-privilege service account.
  2. Not used in the MVP.
  3. Also used for query execution.
  4. Other: _______________
- [Answer]: we don't know whether or not the source has the mcp enabled or not, and whether or not it is capable of doing so, so we should consider it optional

### G-03 Grafana Assistant
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** Per the brief, Grafana Assistant is built into Grafana Cloud; our Grafana is self-hosted. Do we drop it from the MVP and revisit later?
- **Why this matters:** Avoids planning on a feature that is likely unavailable (X-003).
- **Options:**
  1. **(Recommended)** Not used in the MVP; DevOps confirms availability in Stage 4; revisit after the pilot.
  2. Check availability now before Stage 3.
  3. Other: _______________
- [Answer]: we don't know whether or not the source has the grafana assistant enabled or not, and whether or not it is capable of doing so, so we should consider it optional

### G-04 Loki access path (gateway/proxy and auth)
- **Tag:** Can-defer (verify in Stage 4); **blocks the Loki connector spike/build** · **Owner:** DevOps (Hadi Septian); backup System Architect (William Tanardi)
- **Question:** How is Loki reached and authenticated? Use the checklist in `stage2-actors-constraints-assumptions.md` §6.
- **Why this matters:** Without it the finder cannot query Loki, and C5 risk is unknown if Loki has no auth in front.
- **Options:**
  1. Through a gateway / proxy / ingress with authentication (type: ___).
  2. Directly to Loki without authentication (network-restricted only).
  3. Only reachable via Grafana.
  4. Other: _______________
- **Recommended action:** run checks (a)–(d) and record results here.
- [Answer]:2

### G-05 Loki topology
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** Is there one Loki per project, a shared multi-tenant Loki, or a shared Loki separated by labels?
- **Why this matters:** Decides how project isolation is enforced (A3).
- **Options:**
  1. One Loki per project (current working assumption).
  2. Shared Loki, multi-tenant (`auth_enabled: true`, `X-Scope-OrgID`).
  3. Shared Loki, single tenant, projects separated by labels.
  4. Other: _______________
- **Recommended action:** answer from checklist steps (a) and (c). The design proposal (source bindings) works for all three.
- [Answer]:1

### G-06 Service identity and approval of read-only access
- **Tag:** Blocker · **Owner:** DevOps, System Architect · **Due:** TBD (set by PM)
- **Question:** Which identity/token does the finder use to reach Loki and the database, and who approves read-only access?
- **Why this matters:** C1 and C5; an over-privileged identity turns the finder into a backdoor.
- **Options:**
  1. **(Recommended)** A dedicated read-only service identity per project and per source, approved by DevOps and the project's lead (name: Yosia Alvin). Rationale: least privilege; revocable per project.
  2. One shared read-only service identity for all projects.
  3. Users' own credentials passed through (not possible for BA/QA under T-002).
  4. Other: _______________
- [Answer]: 1

### G-07 Project isolation with a service identity
- **Tag:** Blocker · **Owner:** System Architect · **Due:** TBD (set by PM)
- **Question:** How is project isolation enforced if the finder holds service identities?
- **Why this matters:** A user of project A must not read project B's logs (C5).
- **Options:**
  1. **(Recommended)** Both layers: per-project credentials/tenants at the source where available, plus server-side project bindings injected into every template and checked against the user's project membership; every query audited.
  2. Finder-side checks only (one identity for all projects).
  3. Other: _______________
- [Answer]: 1

### G-08 Who uses the database and k9s today
- **Tag:** Can-defer · **Owner:** DevOps
- **Question:** Which roles can use the database and k9s today (Grafana is answered: Developers and DevOps)?
- **Why this matters:** Confirms escalation targets and that the finder does not give BA/QA more than intended.
- **Options:**
  1. Developers and DevOps only (same as Grafana).
  2. DevOps only.
  3. Other: _______________
- [Answer]:1

---

## 5. Database

### D-01 Relevant engines, schemas and tables
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** Developers, DevOps
- **Question:** Which database engine(s), schemas and tables do engineers query for the sampled requests?
- **Why this matters:** Database templates and identifier-to-column mapping depend on it.
- **Options:**
  1. **(Recommended)** Developers list, for the pilot project only, the tables they actually open for the 24 sampled requests.
  2. DevOps provides the full schema for the pilot project.
  3. Other: _______________
- [Answer]:1

### D-02 Read-replica
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** Do the UAT/RC databases have a read-replica?
- **Why this matters:** Without one, the finder queries the primary (A4 risk).
- **Options:**
  1. No read-replica (current working assumption).
  2. Read-replica exists for: ___.
  3. Other: _______________
- [Answer]:1

### D-03 Limits on the primary database and their approver
- **Tag:** Can-defer (verify in Stage 4); **blocks enabling the database source** · **Owner:** DevOps
- **Question:** Who approves database query limits (timeout, row cap, time range, concurrency), and when are values set?
- **Why this matters:** Protects UAT/RC testers from slow queries if there is no replica.
- **Options:**
  1. **(Recommended)** Stage 4 proposes values; DevOps (and the database owner, name: Hadi Septian) approves before the database source is enabled.
  2. Values given now: timeout ___, row cap ___, max range ___, concurrency ___.
  3. Other: _______________
- [Answer]:1

---

## 6. Kubernetes

### K-01 Direct pod-log reading
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** Is direct pod-log reading needed, or does Loki already hold the same logs (for UAT and RC)?
- **Why this matters:** A direct Kubernetes connector needs cluster RBAC and is more work.
- **Options:**
  1. **(Recommended)** Loki only, if DevOps confirms Loki holds the same pod logs for UAT and RC; the Kubernetes "source" is then served through Loki.
  2. Direct pod-log reading for logs not shipped to Loki (which: ___).
  3. Both.
  4. Other: _______________
- [Answer]:1

### K-02 Short-lived pods
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** How are logs from pods that no longer exist handled?
- **Why this matters:** Direct pod reading cannot see deleted pods.
- **Options:**
  1. **(Recommended)** Rely on Loki retention (period: TBD).
  2. Direct reading only for running pods; accept the gap.
  3. Other: _______________
- [Answer]:1

---

## 7. Search and mapping

### M-01 Identifier list and mapping for the pilot project
- **Tag:** Blocker · **Owner:** BA (with Developers) · **Due:** TBD (set by PM)
- **Question:** What is the full list of identifiers and where does each live (log field or DB column) for the pilot project?
- **Why this matters:** Search by identifier is the core use case [§2]; Stage 3 acceptance criteria use it.
- **Options:**
  1. **(Recommended)** BA and Developers compile the pilot list starting from §2 (CIF, account number, hashcard, request ID, program ID, license ID, endpoint path), each with its log field / DB column; other projects later.
  2. Full list for all projects now.
  3. Other: _______________
- [Answer]:1

### M-02 Identifiers as Loki labels
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps
- **Question:** Is any identifier (CIF, account number, hashcard, etc.) a Loki label, or are they all inside log lines?
- **Why this matters:** Line search over wide ranges can be slow; decides mandatory time range and limits.
- **Options:**
  1. All inside log lines (working assumption AS-06).
  2. Some are labels: ___.
  3. Other: _______________
- [Answer]:1

### M-03 Owner of the business-term dictionary
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** Who owns and maintains the business-term dictionary per project (e.g. "ganti rek sumber", "penutupan kartu", "buka blokir", "saldo", refund, H2H, VKYC, liveness)?
- **Why this matters:** The dictionary maps Indonesian feature names to services/endpoints; stale entries give wrong results.
- **Options:**
  1. **(Recommended)** BA owns the terms per project; Developers confirm the mapping to services/endpoints; changes go through the Project Admin.
  2. Developers own it.
  3. Other: _______________
- [Answer]:1

### M-04 Owner of the Project Admin role
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** Who holds the Project Admin role in practice?
- **Why this matters:** This role sets dictionary, identifier mapping, environments and who can access which project.
- **Options:**
  1. **(Recommended)** Split: a BA manages dictionary and identifier mapping; DevOps manages environments, source bindings and access. Rationale: business content and security settings have different owners.
  2. A BA holds the whole role.
  3. A DevOps member holds the whole role.
  4. Other: _______________
- [Answer]:1

### M-05 Timezone rules for input and display
- **Tag:** Blocker · **Owner:** BA, System Architect · **Due:** TBD (set by PM)
- **Question:** Users state times in WIB; logs mix UTC and +0700 in the same line. What are the rules?
- **Why this matters:** A wrong conversion puts the search window 7 hours off and the log is not found.
- **Options:**
  1. **(Recommended)** Users enter and see times in WIB (+0700); the finder converts to UTC for queries; the raw view keeps original timestamps unchanged.
  2. Show both WIB and UTC everywhere.
  3. Other: _______________
- [Answer]:1

### M-06 Default time window and maximum result size
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** System Architect, BA
- **Question:** What window is used for an exact time ("16:04:44"), an approximate time ("around 15:11", "11:22–11:23") and a vague time ("yesterday afternoon"), and what is the maximum result size?
- **Why this matters:** Requirement-level rule is needed in Stage 3; values depend on Loki performance (Stage 4).
- **Options:**
  1. **(Recommended)** Stage 3 requires a mandatory, bounded window and a result cap, with defaults shown as editable chips; Stage 4 proposes values after the Loki spike; vague times always ask the user to confirm the window.
  2. Set values now: exact ±___ min, approximate ±___ min, max range ___, max results ___.
  3. Other: it should be parameterizable
- [Answer]: 3

### M-07 Empty correlation_id across services
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** System Architect, Developers
- **Question:** When `correlation_id` is empty and a request spans services, what should the finder show?
- **Why this matters:** Users may see only part of a flow.
- **Options:**
  1. **(Recommended)** Show matching lines from each service in the same window, grouped by service and labelled "not linked by correlation_id"; link by a shared identifier (e.g. CIF, request ID) only when present.
  2. Show single-service results only.
  3. Other: _______________
- [Answer]:1

### SRC-01 Missing source documents
- **Tag:** Can-defer · **Owner:** BA, PM
- **Question:** Can the team add the decks, the full 24 sampled requests and a (masked) Loki log sample to the workspace?
- **Why this matters:** Stage 3 acceptance criteria must trace to real requests; only 10 of 24 are available.
- **Options:**
  1. **(Recommended)** Add the 24 requests (identifiers replaced by placeholders) and one masked log sample before Stage 3.
  2. Proceed with the 10 requests in the brief.
  3. Other: _______________
- [Answer]:1

---

## 8. LLM and output

### L-01 External LLM
- **Tag:** Blocker · **Owner:** PM, System Architect · **Due:** TBD (set by PM)
- **Question:** May user sentences or log content be sent to an external LLM? If not, where must it run?
- **Why this matters:** Sentences contain identifiers (CIF, hashcard); logs contain banking data (C1). Linked to `H-01`.
- **Options:**
  1. **(Recommended)** Nothing leaves the Soluix environment: the LLM runs internally/locally; only the user sentence is sent to it, never log content. Rationale: safest until `L-02` is answered.
  2. User sentence only, with identifiers replaced by placeholders before sending, to an approved external LLM; no log content.
  3. External LLM allowed for sentences and log content.
  4. Other: _______________
- [Answer]:1

### L-02 Client contract or regulatory limits
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** Do client contracts (e.g. BCAS) or regulations limit where UAT/RC log data may be processed? Who confirms?
- **Why this matters:** Can rule out options in `L-01` and `H-01`.
- **Options:**
  1. **(Recommended)** PM checks with the client contract owner and Soluix compliance (names: ___) before Stage 4.
  2. Known: no limits for UAT/RC data.
  3. Known limits: ___.
  4. Other: _______________
- [Answer]:2

### L-03 Confirm C2: structured filters only (P-002)
- **Tag:** Blocker · **Owner:** System Architect · **Due:** TBD (set by PM)
- **Question:** Confirm that the LLM only produces structured filters and the finder runs predefined templates (no free-form LogQL/SQL)?
- **Why this matters:** Main control against injection and over-broad queries (C5).
- **Options:**
  1. **(Recommended)** Confirm C2 as written.
  2. Allow LLM-generated LogQL, validated against an allow-list.
  3. Other: _______________
- [Answer]: 1

### L-04 How the simplified view is produced
- **Tag:** Blocker · **Owner:** System Architect, BA · **Due:** TBD (set by PM)
- **Question:** Is the simplified view produced by deterministic parsing, by an LLM, or both? If an LLM is involved, how do we prevent wrong content in a view people forward to clients?
- **Why this matters:** A wrong summary forwarded to a bank client is a quality and trust risk.
- **Options:**
  1. **(Recommended)** Deterministic parsing only for the MVP (strip prefixes and ANSI codes, unwrap nested JSON, select fields). Rationale: repeatable and testable; no invented content.
  2. Deterministic view plus an optional LLM summary, clearly labelled, never in copied/shared output.
  3. LLM-produced view.
  4. Other: _______________
- [Answer]: 1

### O-01 Fields in the simplified view
- **Tag:** Can-defer · **Owner:** BA
- **Question:** Which fields make up the simplified view, and are they configurable per project?
- **Why this matters:** Defines what BA/QA read and forward.
- **Options:**
  1. **(Recommended)** Default set from the known payload: time (WIB), environment, `project_name`, `project_module`, `module_version`, `correlation_id`, and the parsed `log_data` (masked); Project Admin can adjust per project.
  2. Fixed set, same for all projects.
  3. Other: _______________
- [Answer]:1

### O-02 Sharing and Grafana deeplinks
- **Tag:** Can-defer · **Owner:** BA, System Architect
- **Question:** Should results be shareable by link, and should there be a Grafana deeplink for escalation?
- **Why this matters:** Sharing must respect access rules; only Developers and DevOps use Grafana.
- **Options:**
  1. **(Recommended)** Share links inside the finder that re-check the viewer's access; Grafana deeplink shown only to Developer/DevOps roles.
  2. No sharing in the MVP.
  3. Other: _______________
- [Answer]:2

### O-03 Escalation path
- **Tag:** Can-defer · **Owner:** BA, PM
- **Question:** When a lookup cannot be resolved, how does the requester reach an engineer?
- **Why this matters:** Escalations are counted in the pilot (`MET-03`).
- **Options:**
  1. **(Recommended)** An "Ask an engineer" action that records the escalation in the finder and produces a link with the search filters to paste in the team chat (tool: ___).
  2. Direct message to the team chat from the finder.
  3. Other: _______________
- [Answer]:1

---

## 9. Security, privacy and compliance

### SEC-01 Who may search which project and environment
- **Tag:** Blocker · **Owner:** PM, System Architect · **Due:** TBD (set by PM)
- **Question:** Who may search which project and environment (UAT, RC)?
- **Why this matters:** C5 and client confidentiality.
- **Options:**
  1. **(Recommended)** Only members of a project, per environment, granted by the Project Admin; default no access.
  2. All Soluix staff for all projects.
  3. Other: access controls and privileges should be parameterizable but done by enforcing principle of least privilege, because some member can be part of multiple projects
- [Answer]: 3

### SEC-02 Masking rules
- **Tag:** Blocker · **Owner:** System Architect, PM · **Due:** TBD (set by PM)
- **Question:** What masking applies to CIF, account numbers, hashcards, tokens and document content? Is UAT/RC data real client data or synthetic?
- **Why this matters:** C1; users still need enough of an identifier to confirm they found the right log.
- **Options:**
  1. **(Recommended)** Mask in all views by default: tokens and document content fully hidden; CIF, account numbers and hashcards partially masked (pattern TBD); the identifier the user searched for may be shown in full to that user.
  2. Mask everything fully.
  3. No masking for UAT/RC (only if the data is confirmed synthetic).
  4. Other: option 1, but again it should be parameterizable and may be toggle-able if the user has the rights
- [Answer]: 4

### SEC-03 "Copy raw": masked or unmasked, per role
- **Tag:** Blocker · **Owner:** System Architect, PM · **Due:** TBD (set by PM)
- **Question:** Does "Copy raw" return masked or unmasked data, and for which roles?
- **Why this matters:** Unmasked copy is the most direct backdoor risk (C5).
- **Options:**
  1. **(Recommended)** Always masked for all roles; engineers who need unmasked data use Grafana / database / k9s as today (C4).
  2. Unmasked for Developer/DevOps roles only, with audit.
  3. Other: either or, based on the rights the user has and whether they want true raw data or not
- [Answer]: 3

### SEC-04 Audit logging
- **Tag:** Blocker · **Owner:** System Architect, PM · **Due:** TBD (set by PM)
- **Question:** What is audited, who may read the audit log, and how long is it kept?
- **Why this matters:** Banking-client compliance and pilot evidence.
- **Options:**
  1. **(Recommended)** Audit user, time, project, environment, filters or sentence (masked), result count, raw-view opens, copy and share actions, escalations; readable by PM and System Architect; retention set by compliance (period TBD).
  2. Audit searches only.
  3. Other:
- [Answer]: 1

---

## 10. Pilot metrics

### MET-01 Counting log-request pings
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** How are log-request pings counted for the 3-month baseline and during the pilot?
- **Why this matters:** The 65% reduction target cannot be measured without it.
- **Options:**
  1. **(Recommended)** Chat export for the last 3 months, keyword tagging, then manual review by a BA; same method during the pilot.
  2. Engineers log each ping manually from now on.
  3. Other: _______________
- [Answer]:1

### MET-02 Measuring time to a readable log
- **Tag:** Blocker · **Owner:** PM, QA · **Due:** TBD (set by PM)
- **Question:** How is "readable log in under 2 minutes" measured?
- **Why this matters:** Second half of the pilot hypothesis.
- **Options:**
  1. **(Recommended)** Finder instrumentation: time from the first search to the first simplified-view open in a session.
  2. User self-report after each lookup.
  3. Other: _______________
- [Answer]:1

### MET-03 Self-served vs escalated
- **Tag:** Blocker · **Owner:** PM · **Due:** TBD (set by PM)
- **Question:** What counts as a self-served lookup vs an escalated one? Do we record whether the user searched first, and filters vs natural language?
- **Why this matters:** Behavior measurement in the pilot setup [§1.1].
- **Options:**
  1. **(Recommended)** Self-served = user opened a result and did not escalate within the same session; escalated = "Ask an engineer" used or a chat ping about the same request; record search mode (filters / natural language) per search.
  2. Users mark each lookup as resolved or not.
  3. Other: _______________
- [Answer]:1

### MET-04 Decision thresholds after the pilot
- **Tag:** Can-defer · **Owner:** PM
- **Question:** What separates "partly met" from "no change"?
- **Why this matters:** The roll-out decision rule needs numbers.
- **Options:**
  1. **(Recommended)** Set before the pilot starts, after the baseline is counted: "met" ≥ 65% reduction and under 2 minutes; "partly met" and "no change" thresholds: partly met 35%-64%, no change: 0%-34%.
  2. Decide after the pilot.
  3. Other: _______________
- [Answer]:1

---

## 11. Added 2026-11-25 after reference review

Source: `reference-docs-review.md` (Ideation PDF, Presenter PDF, Workshop PPTX, commit `ac124f7`). Existing questions are not renumbered. `SRC-01` is now largely answered by the committed files; what remains is in `reference-docs-review.md` §6.

### U-01 FE and Mobile developers as searchers
- **Tag:** Blocker · **Owner:** PM, BA · **Due:** TBD (set by PM)
- **Question:** The chat captures show an FE developer and a Mobile developer asking for logs, not only BAs and QAs [Ideation p.4–11]. Are they in scope as searchers in the MVP?
- **Why this matters:** Personas drive Stage 3 user stories and who gets access per project (`SEC-01`).
- **Options:**
  1. **(Recommended)** Yes, as searchers with the same rights as BA/QA (masked views, project membership required). Rationale: they send the same kind of requests to engineers. Whether FE/Mobile developers count as "Developers" with Grafana access today is unknown; please confirm.
  2. No; MVP is for BA/QA only; FE/Mobile developers keep asking engineers.
  3. Yes, but with a separate role that may see more detail (define: ___).
  4. Other: _______________
- [Answer]:1

### M-08 Environment-to-label mapping
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** DevOps, System Architect
- **Question:** The screenshot shows payload `project_name` = `bcas-uat-1b` while the `cluster` label = `xip-development` [Ideation p.17]. How does the finder know which logs belong to UAT vs RC?
- **Why this matters:** A wrong mapping returns logs from the wrong environment or nothing at all.
- **Options:**
  1. **(Recommended)** Mapping configured per project and per environment in the source binding (label selectors plus, if needed, a payload filter such as `project_name`); DevOps confirms the real labels in Stage 4.
  2. Use the `cluster` label only.
  3. Use the payload `project_name` only.
  4. Other: _______________
- [Answer]:1

### M-09 Linking requests by X-Request-Id or traceparent
- **Tag:** Can-defer (verify in Stage 4) · **Owner:** System Architect, Developers
- **Question:** `correlation_id` is empty in all visible lines, but some `log_data` payloads carry `X-Request-Id` and `traceparent` headers [Ideation p.17]. May the finder use them to link lines across services?
- **Why this matters:** Without a link, users see only part of a flow (`M-07`).
- **Options:**
  1. **(Recommended)** Yes, as a fallback when `correlation_id` is empty, once Developers confirm these headers are propagated between the services; links shown as "linked by X-Request-Id/traceparent".
  2. No; link only by `correlation_id` or a shared business identifier.
  3. Developers fix `correlation_id` population instead (outside the finder's scope).
  4. Other: _______________
- [Answer]:1

### M-10 Requests without identifier or time
- **Tag:** Blocker · **Owner:** BA, PM · **Due:** TBD (set by PM)
- **Question:** By the orchestrator's count, 12 of 24 sampled requests give no identifier and no time ("minta log lagi", "log untuk trx ini") and rely on an earlier photo or message. What should the finder do with such a search?
- **Why this matters:** About half of real requests look like this; the "Must" rule asks for a clear result or a clear "nothing found" [Ideation p.14].
- **Options:**
  1. **(Recommended)** Ask for the missing filters with guidance (e.g. "add a CIF or a time"), and offer to reopen one of the user's previous searches from search history. Rationale: keeps queries bounded (C5, A4) and covers "the same as before" requests.
  2. Search history / reopen previous search only.
  3. Offer "Ask an engineer" escalation immediately (`O-03`).
  4. Reject with guidance only.
  5. Other: _______________
- [Answer]:1

### SEC-05 "Copy raw" when the log is forwarded to the client
- **Tag:** Blocker · **Owner:** System Architect, PM · **Due:** TBD (set by PM) · **Linked to:** `SEC-03`
- **Question:** Several requests ask for the log in order to forward it to the client (BCA/BCAS) [Ideation p.4–11, p.16]. For copies that leave Soluix, is "Copy raw" masked by default, and does sending need approval?
- **Why this matters:** Masking of "Copy raw" is client-facing: an unmasked copy can send other customers' data or tokens to the client (C1, C5). Answer together with `SEC-03`.
- **Options:**
  1. **(Recommended)** Masked by default for all roles; the copy keeps the searched identifier only if `SEC-02` allows it; every copy is audited (`SEC-04`); no approval step in the MVP.
  2. Masked by default, plus a "client copy" action that needs approval by a Developer or the System Architect before unmasking.
  3. Unmasked copy allowed for named roles, with audit.
  4. Other: again, masking should be togglable based on the users' rights, but should be on by default, regarding the copies that leave soluix, it is out of scope, as there is no easy way to detect that
- [Answer]: 4

### SRC-02 Is the Presenter's Deck the "AIDLC deck"?
- **Tag:** Can-defer · **Owner:** PM
- **Question:** Is `Presenter's Deck - Investor's Team - Simplified Log Finder.pdf` the "Investor's Team – Simplified Log Finder (AIDLC)" deck named in the brief [§1.5]?
- **Why this matters:** Closes item 2 of `stage1-unverified.md`; otherwise a source is still missing.
- **Options:**
  1. **(Recommended)** Yes, same deck (its cover reads "Simplified Log Finder, Built with AIDLC, AIDLC – The Investor's Team").
  2. No; the AIDLC deck is a different file and will be added.
  3. Other: _______________
- [Answer]:1

### SEC-06 Sensitive data in the committed reference PDFs
- **Tag:** Blocker · **Owner:** PM, System Architect · **Due:** TBD (set by PM)
- **Question:** The committed PDFs (commit `ac124f7`) contain what appear to be real CIFs, account numbers, hashcards, a phone number, request IDs, S3 bucket names and base64 document content. How should they be handled? (Risk RK-001.)
- **Why this matters:** The repository is used on several laptops and may be pushed to a shared remote; the data is in git history. Replacing files does not remove history.
- **Options:**
  1. **(Recommended)** Replace the PDFs with masked copies and rewrite git history to remove the originals before pushing to any shared remote; every laptop re-clones afterwards. Rationale: only a history rewrite removes the values.
  2. Replace with masked copies going forward; keep history as is (accept the risk; remote must be restricted).
  3. Keep as is; the repository stays private to the team.
  4. Other: _______________
- [Answer]: 3

### O-04 "Example endpoint response" requests
- **Tag:** Can-defer · **Owner:** PM, BA
- **Question:** One sampled request asks for an example endpoint response, not a log [Ideation p.4–11]. Is that in MVP scope?
- **Why this matters:** Sets the MVP boundary between log lookup and API documentation.
- **Options:**
  1. **(Recommended)** Out of MVP scope, unless the response is visible in a log line found by a normal search. Rationale: the finder reads logs; API examples belong to API documentation.
  2. In scope: show a logged response for an endpoint as an example.
  3. Other: _______________
- [Answer]:1

---

## 12. Added 2026-11-25 after Blocker review

Source: review of all answers in this file after S-02, G-06 and SEC-04 were fixed. Existing questions are not renumbered. `C-01` turns audit note N-002 into a question.

### C-01 Default unmask and "true raw" rights
- **Tag:** Blocker · **Owner:** System Architect, PM · **Due:** TBD (set by PM) · **Linked to:** `SEC-02`, `SEC-03`, `SEC-05`, audit note N-002
- **Question:** The `SEC-02`, `SEC-03` and `SEC-05` answers make masking on by default but toggleable "if the user has the rights", and allow a "true raw" copy. Who can hold the right to unmask views and copy true raw data?
- **Why this matters:** Stage 3 acceptance criteria need to know who can hold this right. Team decision T-002 rejected giving BA/QA direct raw access, and C5 forbids broader raw access than intended.
- **Options:**
  1. **(Recommended)** No one has the right by default; the access manager (DevOps half of Project Admin, `M-04`) grants it per user, per project and per environment; only Developer/DevOps users are eligible in the MVP; every unmask and true-raw copy is audited (`SEC-04`). Rationale: least privilege, consistent with T-002 and C5.
  2. Developer and DevOps users have the right by default in projects they belong to; BA/QA/FE/Mobile never.
  3. Any user, including BA/QA, can be granted the right by the Project Admin, with audit. Note: weakens T-002; the team must confirm.
  4. Other: _______________
- [Answer]:3

### C-02 Confirm the BCAS approver
- **Tag:** Can-defer; **needed before read-only access is requested for the Loki spike** · **Owner:** PM
- **Question:** `G-06` names Bernhard Praditya as the approver, and the brief lists Bernhard Praditya as a Business Analyst of The Investor's Team. The answer also says each project has different leads. Is Bernhard Praditya the BCAS project lead who approves read-only access?
- **Why this matters:** Read-only access for the Loki spike needs the right approver for the pilot project, and the approver must be recorded per project.
- **Options:**
  1. **(Recommended)** Yes, Bernhard Praditya is the BCAS project lead who approves read-only access together with DevOps; the approver is stored per project in the finder's project settings.
  2. A different BCAS lead approves (name: Yosia Alvin).
  3. Other: _______________
- [Answer]:2

### C-03 Masked raw-log text sample timing
- **Tag:** Blocker · **Owner:** PM, DevOps · **Due:** TBD (set by PM) · **Linked to:** `SRC-01`
- **Question:** The `SRC-01` answer (option 1) asked for the 24 requests and one masked log sample before Stage 3. The 24 requests are now in the Ideation PDF, but the log sample is only a screenshot (Ideation p.17; `reference-docs-review.md` §6 N1). When must a masked text sample be provided?
- **Why this matters:** Parser acceptance criteria and parser tests need real log lines; waiting for the sample before Stage 3 may delay Stage 3.
- **Options:**
  1. **(Recommended)** Start Stage 3 with the screenshot as the source for parser acceptance criteria; a masked text export of a few raw lines plus the full label list is required before the parser unit is built (Stage 5 / Construction); owner DevOps or a Developer. Rationale: Stage 3 states behavior, not parsing code.
  2. Keep the `SRC-01` answer: provide the masked text sample before Stage 3 starts.
  3. Other: _______________
- [Answer]:1

### C-04 Is UAT/RC data real client data or synthetic?
- **Tag:** Can-defer · **Owner:** PM · **Linked to:** `SEC-02`
- **Question:** The second half of the `SEC-02` question was not answered. Is UAT/RC data real client data or synthetic?
- **Why this matters:** Masking is on by default either way, so this does not block Stage 3, but it sets how strict the masking acceptance criteria and default patterns must be. The PDFs contain values that look real.
- **Options:**
  1. **(Recommended)** Treat UAT/RC data as real client data for the MVP.
  2. Confirmed synthetic (who confirmed: Yosia Alvin).
  3. Mixed or unknown; treat as real.
  4. Other: _______________
- [Answer]:2

---

## 13. Added 2026-11-25 after section 12 review

Source: review of the section 12 answers (C-01 to C-04) and of the Can-defer answers filled in on 2026-11-25. Existing questions are not renumbered. Decisions are logged in `audit.md` (entry "Section 12 answers, Can-defer answers, deck added").

> **C-06 not raised.** A conflict between `G-06` and `C-02` was expected, but the `G-06` option 1 text already names Yosia Alvin, which matches the `C-02` answer (option 2, Yosia Alvin). Yosia Alvin is recorded as the BCAS approver (audit A-060, supersedes A-057). The ID C-06 is not used, so the other IDs in this section match earlier references.

### C-05 Unmask / true-raw rights for BA/QA vs team decision T-002
- **Tag:** Blocker · **Owner:** PM, System Architect · **Due:** TBD (set by PM) · **Linked to:** `C-01`, `SEC-02`, `SEC-03`, `SEC-05`, `SEC-04`, T-002, C5, contradiction X-004
- **Question:** `C-01` option 3 lets the Project Admin grant the unmask and true-raw right to any user, including BA/QA. Team decision T-002 rejected direct raw access for BA/QA, and C5 says the finder must not become a backdoor with broader raw access than intended. Which reading stands?
- **Why this matters:** Stage 3 acceptance criteria for masking, "Copy raw" and access grants depend on it. If both are kept without a clear boundary, the requirements contradict a team decision that was marked "do not reopen without strong reason".
- **Options:**
  1. **(Recommended)** Both stand. T-002 covers direct Grafana / database / k9s access only. Inside the finder, the Project Admin may grant the unmask / true-raw right to any user (including BA/QA) per user, per project and per environment; the right is off by default and every use is audited (`SEC-04`). Rationale: keeps the `C-01` answer while staying within C5, because access is granted explicitly and recorded.
  2. T-002 stands as written: BA/QA cannot be granted the unmask / true-raw right; `C-01` changes to option 1.
  3. `C-01` stands and T-002 is reopened and replaced (the team must re-decide T-002).
  4. Other: _______________
- [Answer]:1

### C-07 "No sharing in the MVP" vs the escalation link
- **Tag:** Can-defer · **Owner:** BA, PM · **Linked to:** `O-02`, `O-03`, `MET-03`, contradiction X-006
- **Question:** `O-02` = option 2 (no sharing in the MVP), but `O-03` = option 1, where "Ask an engineer" produces a link with the search filters to paste into the team chat. Is that escalation link allowed?
- **Why this matters:** The escalation link is a form of sharing. Stage 3 needs one rule for links, and escalations are counted in the pilot (`MET-03`).
- **Options:**
  1. **(Recommended)** The escalation link is allowed. It re-checks the viewer's access when opened and is the only link in the MVP; general result sharing is out of the MVP. Rationale: keeps the escalation flow measurable without adding a general sharing feature.
  2. No links at all; "Ask an engineer" copies the filters as text.
  3. Other: _______________
- [Answer]:1

### C-08 Source of the G-04 answer (Loki reached without authentication)
- **Tag:** Can-defer; **blocks the Loki spike** · **Owner:** DevOps (Hadi Septian), System Architect · **Linked to:** `G-04`, `G-05`, `G-06`, `G-07`, risk RK-002
- **Question:** `G-04` = option 2 (directly to Loki without authentication, network-restricted only). This was planned as a Stage 4 verification using the checklist in `stage2-actors-constraints-assumptions.md` §6. Who verified it, and how (steps a–d)?
- **Why this matters:** If true, there is no per-project credential at Loki, so Loki isolation under `G-06` / `G-07` relies on one Loki per project (`G-05`) plus the finder's own server-side checks. Anyone with a network route to Loki can query it today (new risk RK-002). The Loki connector design and the spike depend on whether this is a verified fact or an assumption.
- **Options:**
  1. **(Recommended)** Verified by DevOps with the §6 checklist; record the date and the result of each step (a–d) here. Rationale: turns the answer into a fact the design can rely on.
  2. Not yet verified; treat as a working assumption until the Loki spike.
  3. Other: _______________
- [Answer]:2
