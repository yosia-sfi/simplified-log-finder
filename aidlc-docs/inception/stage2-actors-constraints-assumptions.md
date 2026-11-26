# Stage 2 (Discover): Actors, Constraints and Assumptions

AI-DLC Inception, Stage 2 (Discover). Status: **Approved 2026-11-25**.
Sources: `prompt-inception.md` (the brief), cited `[§x.y]`; the user's Stage 1 corrections, cited `[User 2026-11-25]`; Stage 1 artifacts in this folder; from 2026-11-25 the committed reference files, cited `[Ideation p.N]` / `[Presenter p.N]` (see `reference-docs-review.md`). Anything not in these sources is marked **TBD – needs confirmation**. Everything labelled "AI proposal" or "AI observation" is a proposal for the team to accept or reject ("AI proposes, humans decide").
Questions raised here are in `stage2-questions.md` and referenced by ID (e.g. `H-01`).

---

## 1. Actors

### 1.1 Product user roles (personas)

These are roles inside the product, not project-team roles [§3 Stage 3]. Full user stories come in Stage 3.

| ID | Role | What they do with the finder | Source | Open points |
|---|---|---|---|---|
| R1 | Business Analyst | Primary searcher. Finds and reads logs by time, identifier, service or error code; reads the simplified view; opens the raw log | §1.1, §3 Stage 3 | Which projects/environments each BA may search: `SEC-01` |
| R2 | QA Engineer | Primary searcher, heavy during UAT/regression; same tasks as R1 | §1.1, §3 Stage 3 | Same as R1 |
| R3 | Developer / Engineer | Receives escalations when the finder cannot resolve a lookup. Keeps using Grafana, the database and k9s directly (C4). Today only Developers and DevOps use Grafana | §1.1, §1.4, User 2026-11-25 | Escalation channel: `O-03`; unmasked raw access: `SEC-03` |
| R4 | Project Admin | Configures the finder per project: business-term dictionary, identifier-to-field mapping, environments, who can access which project | §3 Stage 3 | Who holds this role in practice (BA or DevOps): `M-04` |
| R5 | Project Manager | Reads pilot metrics (pings, time to readable log, self-served vs escalated) | §1.1, §3 Stage 3 | Metric definitions: `MET-01` to `MET-04` |

**Observed requesters (added 2026-11-25):** the 24 chat captures also show an **FE developer** and a **Mobile developer** asking for logs [Ideation p.4–11]. Whether they are in scope as searchers (same rights as R1/R2, or a separate role) is pending question `U-01`. Engineers pinged include the System Architect and backend developers (R3).

AI observation: the brief does not name a security/compliance reviewer who reads the audit log. Whether such a role exists: **TBD – needs confirmation** (`SEC-04`).

### 1.2 External systems

| ID | System | Interaction with the finder | Status / what is known | Source |
|---|---|---|---|---|
| E1 | Grafana (self-hosted) | No query execution required. Possible uses: data-source/label discovery via Grafana MCP server; deeplinks for engineers on escalation | Self-hosted confirmed. MCP server behavior and service-account RBAC on our instance: verify in Stage 4. Grafana Assistant likely unavailable (Cloud feature per brief): verify in Stage 4 | §1.3, User 2026-11-25 |
| E2 | Loki | Read-only, templated LogQL queries over a bounded time range (`query_range`, labels, label values per brief) | Access path (gateway/proxy, auth) **unknown, verify in Stage 4**. One Loki per project: working assumption. Shipping via Fluent Bit, at least UAT; RC shipping: **TBD** | §1.3, §1.5, User 2026-11-25 |
| E3 | Database | Read-only, templated, limited queries | Engine, schemas, tables: **TBD**. No read-replica: working assumption, treated as risk | §1.4, §3 Stage 2 |
| E4 | Kubernetes pod logs | Read-only pod log reading, if needed | Whether Loki already holds the same logs: **TBD** (`K-01`) | §1.4 |
| E5 | LLM (NL-to-filters) | Turns a sentence into structured filters only (C2, proposed) | Location (internal vs external) **TBD**; linked to hosting (`H-01`, `L-01`) | §1.4 |
| E6 | Identity provider / SSO | User login, role and project membership | Which IdP, if any: **TBD** (`H-03`) | §4 |
| E7 | Team chat tool | Source of the 3-month ping baseline; possible escalation channel | Tool name and export method: **TBD** (`MET-01`, `O-03`) | §1.1 |

---

## 2. Constraints

| # | Constraint | Status | Source |
|---|---|---|---|
| C1 | Security and compliance first (banking clients). Logs may contain CIF, account numbers, hashcards, tokens, document content | Given | §1.4 |
| C2 | LLM translates a sentence into structured filters only; finder runs predefined query templates; no free-form LogQL/SQL | **Proposed; confirm** (`L-03`) | §1.4 |
| C3 | Multi-project architecture from day one; only the pilot project onboarded first | **Proposed; confirm** (`S-01`, contradiction X-001) | §1.4 |
| C4 | Must not replace Grafana, the database or k9s for engineers | Given | §1.4 |
| C5 | Must not become a backdoor with broader raw access than intended | Given | §1.4 |
| C6 | Environments in scope: UAT and RC only; production and others out of MVP | **User-confirmed 2026-11-25** | User 2026-11-25 |
| C7 | Hosting "local first" | **Direction confirmed 2026-11-25; meaning open** (`H-01`) | User 2026-11-25 |
| C8 | No direct Grafana / database / k9s access for BA/QA | Decided by team (T-002) | §1.2 |
| C9 | Next phase (images, more input sources) must not be blocked by the MVP design | Given | §1.4 |

AI observation on C6: limiting scope to UAT and RC lowers exposure compared with production, but UAT/RC logs can still contain sensitive values [§1.5], so C1 and C5 apply in full. Whether UAT/RC data is real client data or synthetic test data: **TBD – needs confirmation** (`SEC-02`). Update 2026-11-25: **synthetic**, confirmed by Yosia Alvin (`C-04`, A-062); masking stays on by default (A-051, A-054).

---

## 3. Assumptions register

### 3.1 The four DevOps assumptions [§3 Stage 2]

The DevOps owner was unavailable; these are never treated as facts.

#### A1. Grafana is Grafana Cloud
- **Status:** **Resolved: FALSE** [User 2026-11-25]. Grafana is self-hosted (audit A-007, P-004a, X-002).
- **If it had been true:** Grafana Assistant could have been an option; Grafana would be reached over the internet; Grafana-managed auth.
- **Now that it is false (consequences, verify in Stage 4):**
  - Grafana Assistant is described in the brief as built into Grafana Cloud, so it is likely unavailable (`G-03`).
  - Grafana MCP server, if used, would connect to our internal Grafana URL with a service account token; RBAC scope must be checked on our instance (`G-02`).
  - Grafana deeplinks resolve to an internal URL that only Developers and DevOps can use; fine for escalation, useless for BA/QA (`O-02`).
- **Robust design (AI proposal):** the finder does not depend on Grafana to run queries. Grafana is optional (discovery and engineer deeplinks only), so its availability or licensing does not block the MVP.

#### A2. Loki is accessed through a gateway
- **Status (update 2026-11-25):** **Answered FALSE** in `G-04` (option 2: directly to Loki without authentication, network-restricted only; audit A-063, P-004c). The answer came before the planned verification; its source is asked in `C-08`, so it stays a working assumption until C-08 is answered. See risk RK-002. The "If false" case below applies.
- **Original status:** Unknown. **Deferred to Stage 4 technical verification** [User 2026-11-25]. Owner DevOps (Hadi Septian), backup System Architect (William Tanardi). Can-defer, but **blocks the Loki connector spike/build**. Checklist in §6 (`G-04`).
- **If true:** the finder calls the gateway URL with the gateway's credential type (TBD); the gateway may set or require the tenant header. Network route from the finder host to the gateway is needed.
- **If false (direct Loki):** per the brief, Loki does not handle authentication, so anyone with a network route can query it. The finder's connector becomes the main control point, and the Loki endpoint must not be reachable from user browsers. DevOps may need to add a proxy or network restriction (TBD).
- **Robust design (AI proposal):** per-project connector configuration with base URL, auth type (none / basic / bearer, as found), optional custom headers (e.g. `X-Scope-OrgID`), and secrets held server-side only. All Loki calls happen server-side; the browser never receives Loki credentials or URLs.

#### A3. There is one Loki per project
- **Status (update 2026-11-25):** **Confirmed TRUE** in `G-05` (option 1; audit A-064). With no authentication at Loki (A2), project isolation for Loki relies on separate Loki endpoints plus the finder's server-side checks (RK-002).
- **Original status:** Unconfirmed working assumption; verify in Stage 4 (`G-05`).
- **If true:** project isolation comes from separate endpoints/credentials. Onboarding a project means adding an endpoint and credential.
- **If false (shared Loki):** isolation relies either on tenants (`X-Scope-OrgID`, when multi-tenancy is enabled) or on label selectors in every query. Label-only isolation fails if a template omits the project selector, which would breach C5.
- **Robust design (AI proposal):** a per-project "source binding" holding endpoint, credential, optional tenant ID and mandatory label selectors. Templates always inject the binding server-side; users cannot edit or remove it. The same model works for both answers. A test per template checks that the project selector is present (Stage 3/4).

#### A4. The database has no read-replica
- **Status (update 2026-11-25):** **Confirmed TRUE** in `D-02` (option 1; audit A-067). Risk stays high. Limits are proposed in Stage 4 and approved by DevOps and the database owner Hadi Septian before the database source is enabled (`D-03`, A-068).
- **Original status:** Unconfirmed working assumption; **treated as a risk**; verify in Stage 4 (`D-02`).
- **If true (no replica):** finder queries hit the primary UAT/RC database. Risk: slow queries affect testers during UAT/regression. Needs query timeout, row cap, mandatory time range, concurrency limit, and explicit approval from the database owner before the connector is enabled (`D-03`). Values: **TBD** (proposed in Stage 4, approved by DevOps).
- **If false (replica exists):** connector points to the replica; the same limits still apply, with values possibly relaxed.
- **Robust design (AI proposal):** limits are mandatory in both cases; the connector target is configuration; the database source is switched on per project only after recorded approval; queries come only from predefined, parameterized templates (C2).

### 3.2 Other working assumptions (AI proposals, need confirmation)

| ID | Assumption | Why it is needed | Confirm via |
|---|---|---|---|
| AS-05 | Fluent Bit ships both UAT and RC logs to Loki (brief only says "UAT at least" for the sample) | RC appears in real requests [§2]; if RC is not in Loki, another source is needed | `K-01`, Stage 4 |
| AS-06 | Identifiers (CIF, hashcard, account number) live inside log lines, not as Loki labels | Drives the mandatory time range and result limits [§3 Stage 4] | `M-02`, Stage 4 |
| AS-07 | Users state times in WIB (+0700) [§4] | Timezone conversion rules | `M-05` |
| AS-08 | Pilot users can reach the finder host over the internal network | Depends on the meaning of "local first" | `H-01` |
| AS-09 | One repository holds the finder (single-repo, provisional) | Workflow state; affects Stage 5 plan only | `R-01` |
| AS-10 | (Added 2026-11-25) An environment does not map to a single Loki label: the screenshot shows payload `project_name` = `bcas-uat-1b` with label `cluster` = `xip-development` [Ideation p.17]. Environment-to-label mapping is configured per project and environment in the source binding | Wrong mapping returns the wrong environment or nothing | `M-08`, Stage 4 |
| AS-11 | (Added 2026-11-25) `X-Request-Id` / `traceparent` headers inside `log_data` can link lines across services when `correlation_id` is empty [Ideation p.17] | Cross-service view (`M-07`) | `M-09`, Stage 4 |

### 3.3 Risks (added 2026-11-25)

| ID | Risk | Status | Question |
|---|---|---|---|
| RK-001 | The committed reference PDFs (commit `ac124f7`) contain what appear to be real CIFs, account numbers, hashcards, a phone number, request IDs, S3 bucket names and base64 document content, now in git history. Relevant to C1 because the repository is used on several laptops and may be pushed to a remote | **Flagged; no action taken; decision pending user**. Update 2026-11-25: accepted (SEC-06 opt 3, repository stays private); severity lowered because UAT/RC data is confirmed synthetic (`C-04`, A-062) | `SEC-06`, `C-04` |
| RK-002 | (Added 2026-11-25) Loki is reachable without authentication (`G-04`): anyone with a network route can query it; no per-project credential at Loki, so isolation relies on one Loki per project (`G-05`) plus the finder's server-side checks (`G-07`) | **Open; source of the G-04 answer pending** | `C-08` |

---

## 4. Ambiguities and gaps found

| Gap | Effect if left open | Question |
|---|---|---|
| "Local first" can mean a laptop, an internal server, or inside the UAT/RC network | Decides whether a team pilot is possible, how the finder reaches Loki/DB/Kubernetes, where secrets live, and whether an external LLM is acceptable | `H-01`, `L-01` |
| MVP "applied across projects" vs one-team pilot (X-001) | Scope and onboarding effort of the MVP | `S-01` |
| Loki access path and topology unknown | Connector design, project isolation | `G-04`, `G-05` |
| Single service identity vs per-project credentials | Project isolation (C5) | `G-06`, `G-07` |
| Masking and "Copy raw" behavior undefined | C1, C5 | `SEC-02`, `SEC-03` |
| Simplified view by parser or LLM | Risk of wrong content forwarded to clients | `L-04` |
| Ping counting and "self-served" not defined | Pilot hypothesis cannot be measured | `MET-01` to `MET-03` |
| Source documents (decks, full 24 samples, log sample) not in workspace | **Mostly closed 2026-11-25** (`reference-docs-review.md`): all 24 requests available. Still missing: masked raw-log text and full label list; confirmation of the AIDLC deck | `SRC-01`, `SRC-02` |
| About half of sampled requests carry no identifier or time (added 2026-11-25) | Unbounded or failed searches | `M-10` |
| Logs forwarded to the client (added 2026-11-25) | "Copy raw" masking is client-facing | `SEC-05`, `SEC-03` |

---

## 5. Build vs reuse challenge [§1.3]

Updated for self-hosted Grafana, Grafana used only by Developers/DevOps, UAT/RC only, and "local first" hosting.

| Option | What it means | Fit with constraints | Trade-offs |
|---|---|---|---|
| O1 Reuse as-is | BA/QA use Grafana (shared dashboards, saved queries, Grafana Assistant) | Conflicts with T-002 (direct access rejected) and T-003 (dashboards are support only). Grafana Assistant likely unavailable on self-hosted. Logs stay dense JSON | Lowest build effort; does not solve the root cause; security risk the team already rejected |
| O2 Hybrid (P-001, the team's proposal) | Finder is a thin orchestration layer: calls Loki HTTP API (via gateway if present) with predefined templates; builds only the differentiators (dictionary, identifier mapping, parser, masking/audit, DB connector, UI, pilot instrumentation) | Fits C1–C5 if masking/audit and source bindings are enforced server-side. Keeps Grafana, DB and k9s for engineers (C4). Does not depend on Grafana Cloud features | Depends on the unknown Loki access path (A2) and topology (A3). Identifier search inside log lines can be slow over wide ranges (needs mandatory time range and limits). Local hosting needs a network route to Loki/DB/Kubernetes |
| O2b Hybrid via Grafana MCP server | As O2, but query execution goes through the Grafana MCP server using a service account | Query execution through MCP tools may not be restricted to our templates (C2) unless constrained; RBAC on self-hosted Grafana unverified | Reuses an existing tool; adds a dependency and a second permission model. Safer use: discovery only (`G-02`) |
| O3 Replace stack (e.g. OpenObserve) | Replace Loki + Grafana with another platform | Conflicts with C4 and with "not a new observability platform" [§1.1]. Claims come from a vendor blog | High migration effort; changes engineers' tools; unverified claims |
| O4 Build from scratch | Own log store/index (copy logs into the finder) | Contradicts "not a new observability platform"; duplicates sensitive data (C1, C5) | Highest effort; second copy of banking data to secure and retain |

**Recommendation (AI proposal only, pending user approval of P-001, question `G-01`):** O2 Hybrid. Query Loki directly through a server-side connector with per-project source bindings, so the design works with or without a gateway and with one or many Loki instances. Use the Grafana MCP server at most for label/data-source discovery (O2b limited to discovery), subject to Stage 4 verification. Do not plan on Grafana Assistant for the MVP.

---

## 6. How to check Loki access (for DevOps, Stage 4)

Owner: DevOps (Hadi Septian); backup: System Architect (William Tanardi). These are **suggested checks, not verified behavior**. Record each result in `stage2-questions.md` `G-04` / `G-05` or in the Stage 4 design notes. Do not share credentials in the documents.

| Step | Check | What to record | How to read the result |
|---|---|---|---|
| a | In self-hosted Grafana (requires a Grafana admin), open **Connections > Data sources > Loki** | Data source URL; auth method (basic auth, bearer, forward OAuth, none); custom HTTP headers such as `X-Scope-OrgID` | A URL pointing to a service like `loki-gateway`, nginx or an ingress suggests a gateway. An `X-Scope-OrgID` header suggests multi-tenancy |
| b | In the cluster, list services in the Loki namespace (e.g. `kubectl get svc -n <loki-namespace>`) | Service names and ports | A gateway-type service suggests requests go through a gateway |
| c | If Loki was installed by Helm, check the values for `gateway.enabled` and the Loki config for `auth_enabled` | Both values | `auth_enabled: true` means multi-tenant mode, so `X-Scope-OrgID` is required |
| d | From the network location where the finder would run (depends on `H-01`), call Loki `/ready` and `/loki/api/v1/labels`, with and without credentials | HTTP status for each call | Success without credentials means no auth in front of Loki (risk for C5). A denied response without credentials and success with them means auth is enforced. No response means no network route from that host |

Also record: the Loki version (for checking API behavior, item 17), whether RC logs are present (AS-05), and whether any identifier appears as a label (AS-06).

---

## 7. Stage 2 self-check [§0.2]

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Every acceptance criterion is testable | **N/A at Stage 2** | No acceptance criteria yet; Stage 3 writes them. Questions that feed testable ACs (time windows `M-06`, timezone `M-05`, masking `SEC-02`, metrics `MET-01`–`MET-03`) are raised |
| 2 | Every rule traces to a source | **Pass** | Actors, constraints and assumptions cite `[§x.y]`, `[User 2026-11-25]` or (from 2026-11-25) `[Ideation p.N]` / `[Presenter p.N]`. Screenshot facts (AS-10, AS-11) are marked for Stage 4 verification. Items without a source are labelled "AI proposal", "AI observation" or "TBD – needs confirmation". No numbers for limits, timeouts or windows are stated as facts. No real identifiers are copied from the reference PDFs |
| 3 | Design respects stack, security and scope limits | **Pass (direction level)** | Options in §5 are checked against C1–C9; O1, O3 and O4 are flagged as conflicting. Production excluded (C6). Robust-design proposals in §3.1 keep credentials server-side and enforce project bindings (C1, C5) |
| 4 | Reuses existing systems, roles and data where possible | **Pass** | Recommended O2 reuses Loki, Fluent Bit ingestion, Grafana (optional), DB and Kubernetes; no new log store. Existing roles mapped to product roles R1–R5 |
| 5 | Evidence of the check is shown | **Pass** | This table; assumption-by-assumption true/false analysis (§3.1); options table (§5); gap-to-question map (§4) |
| 6 | Open questions flagged, not silently decided | **Pass** | All open items are in `stage2-questions.md` with Blocker/Can-defer tags and owners. P-001, P-002, P-003 remain Pending; A2–A4 remain unconfirmed; the build-vs-reuse recommendation is a proposal only. Reference-review findings (2026-11-25) added as questions `U-01`, `M-08`, `M-09`, `M-10`, `SEC-05`, `SRC-02`, `SEC-06`, `O-04`; risk RK-001 flagged with no action taken |
