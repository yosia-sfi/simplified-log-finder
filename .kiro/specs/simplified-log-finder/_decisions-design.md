# Decisions: Design (Stage 4)

> **Instructions:** Review each decision point below. AI recommendations are provided for guidance. Fill in your decisions in the "Answer" sections, then confirm when ready to proceed.

AI-DLC Inception, Stage 4 (Design). Created 2026-11-25, after Stage 3 approval (audit A-085). Status: **all 26 answered (2026-11-26): DD-01..DD-09 in commit `ec616d7`; DD-10..DD-13, DD-15..DD-19, DD-25, DD-26 by the System Architect; DD-24 in commit `cd09c79`; DD-14, DD-20..DD-23 on 2026-11-26. `design.md` generated 2026-11-26 (audit A-114)**. `design.md` is generated only after the Blocker decisions below are answered.

**How to answer.** On each `**Answer:**` line write an option number (for example `1`), your own text, or an owner and due date if someone else must answer (for example `Defer - DevOps, 2026-12-05`). For a whole group you can write once, in chat or here, "use recommendations for section N" or "use recommendations for all except DD-xx: option 2". When done, say **"decisions filled"** and `design.md` is generated from these answers.

**Tags.** *Blocker for design.md* = `design.md` cannot be written without the answer. *Can-defer* = `design.md` records the recommended option as a working assumption, marked open, until answered. **Owner** = role(s) expected to answer.

**Sources.** Brief `prompt-inception.md` §3 Stage 4; Stage 2 decisions in `aidlc-docs/inception/stage2-questions.md` and `aidlc-docs/audit.md` (A-xxx); requirements in `.kiro/specs/simplified-log-finder/requirements.md` (Req N.M, open items OI-xx, constraints SC-x). Values marked "proposal" are AI proposals, not facts, and stay parameterizable. No real identifiers appear in this file.

---

## Already decided (inputs, not re-asked)

| Topic | Decision | Source |
|---|---|---|
| Build vs reuse | Hybrid thin layer over existing sources; input/output-agnostic engine; approach chosen per source in its own discussion | G-01 (A-042) |
| Hosting | Internal Soluix server/VM on the UAT/RC network ("local first"); config and secrets outside the code | H-01 (A-018), H-02 (A-019) |
| Identity | Local accounts managed by the Project Admin (access manager, DevOps half); no SSO in the MVP | H-03 (A-020), M-04 (A-026) |
| LLM location and input | Internal/local LLM; only the user sentence (plus Project dictionary and identifier types) is sent, never log content | L-01 (A-029), Req 10.3–10.4 |
| LLM output | Structured filters only; finder runs predefined Query_Templates; no free-form LogQL/SQL | L-03 (A-048), Req 6.4, 10.1 |
| Simplified view | Deterministic parsing only, no LLM | L-04 (A-049), Req 15 |
| Stack | Backend Go + Echo; frontend TypeScript, React, Vite, TanStack Router, TanStack Query, TanStack Virtual, Tailwind, shadcn/ui, lucide-react; PactFlow contract testing in backend, frontend and CI/CD. **To confirm with Developers** | H-04 (A-040), SC-1 |
| Repository | Single repository, separate top-level directories per part, so it can be split later | R-01 (A-041), SC-2 |
| Loki topology | One Loki per project | G-05 (A-064) |
| Loki access | "Direct, without authentication, network-restricted" is an **unverified working assumption** until the Loki spike (RK-002 open) | G-04, C-08 (A-063, A-081), SC-4 |
| Database | No read-replica on UAT/RC; Stage 4 proposes limits; DevOps and DB owner (Hadi Septian) approve before enabling | D-02 (A-067), D-03 (A-068), Req 23 |
| Kubernetes | Pod logs served through Loki only, once DevOps confirms coverage; deleted pods rely on Loki retention | K-01 (A-069), K-02 (A-070), Req 22 |
| Identifiers | Inside log lines, not Loki labels | M-02 (A-071) |
| Environment mapping | Environment-to-label mapping per project and environment in the Source_Binding | M-08 (A-073), Req 21.2 |
| Masking and grants | Masking on by default; patterns parameterizable; Unmask_Right per user, project, environment, off by default, grantable to any role, audited | SEC-02 (A-051), C-01 (A-059), C-05 (A-079), Req 19–20 |
| Audit | SEC-04 option 1 scope; readable by PM and System Architect; append-only; retention set by compliance (TBD) | SEC-04 (A-053), Req 26 |
| Sharing | No general sharing; the "Ask an engineer" Escalation_Link is the only link and re-checks access | O-02 (A-075), C-07 (A-080), Req 25 |
| Rollout | UAT first, RC added within the pilot once its Loki data is confirmed | S-03 (A-017), Req 30.3 |
| Time handling | Users enter and read WIB; queries in UTC; raw keeps original timestamps | M-05 (A-027), Req 8 |
| Empty correlation_id | Group by service, label "not linked by correlation_id"; fallback link by X-Request-Id/traceparent once propagation is confirmed | M-07 (A-072), M-09 (A-074), Req 24 |
| Stage 3 behaviors | Idle timeout (Req 1.4), salted one-way password hashes (1.5), sessions end on account removal (2.2), masking back on per new search (20.5), median/p90 reporting (27.4), "unclassified" pilot result (27.7): accepted by Stage 3 approval; values and mechanisms are chosen below | A-084, A-086 |

---

## 1. Technical verifications to run first

These are checks, not technology choices. `design.md` lists every row as an explicit assumption with its current working value. "Blocks" names the work that cannot finish until the check is done.

| ID | Verification | Current working value | Owner | Blocks | Source |
|---|---|---|---|---|---|
| V-01 | Loki access path and auth from the finder host: run the `stage2-actors-constraints-assumptions.md` §6 checklist (steps a–d: Grafana data source URL/auth/headers, Loki services, `auth_enabled`/gateway values, `/ready` and `/loki/api/v1/labels` with and without credentials) | Direct, no auth, network-restricted (unverified) | DevOps (Hadi Septian); backup System Architect | Loki connector auth settings (Req 21.3); decision on an authenticating proxy (RK-002); U2 | G-04, C-08 (A-081), OI-01 |
| V-02 | Loki version and behavior of `query_range`, `labels`, `label values`, `limit`/`direction` and `X-Scope-OrgID` in that version | Unknown | DevOps, Developers | DD-04, DD-05 final query shape; U2 | `stage1-unverified.md` item 17 |
| V-03 | Identifiers appear only inside log lines (not labels) for BCAS UAT/RC | Inside lines | DevOps | DD-04 (line filters vs label selectors) | M-02 (A-071), AS-06 |
| V-04 | Real environment-to-label mapping for BCAS UAT and RC (labels such as `cluster`, payload `project_name`), plus the full common label list | Mapping per binding; values unknown | DevOps | Source_Binding content for BCAS (Req 21.2) | M-08 (A-073), AS-10, OI-09 |
| V-05 | `X-Request-Id` / `traceparent` propagated between BCAS services | Unconfirmed | Developers | Req 24.3 linking (fallback stays off until confirmed) | M-09 (A-074), AS-11, OI-12 |
| V-06 | Loki holds UAT and RC pod logs (same as k9s shows) | Unconfirmed | DevOps | Req 22 Kubernetes source; RC enablement | K-01 (A-069), AS-05, OI-11 |
| V-07 | Loki retention period per environment | TBD | DevOps | Req 22.3 message; Max_Range upper bound | K-02 (A-070), OI-02 |
| V-08 | Database engine, schemas and the tables Developers open for RQ-01..RQ-24 | Engine TBD; tables TBD | Developers, DevOps | DB connector driver and templates (Req 23); U5 | D-01 (A-066), OI-08 |
| V-09 | Whether FE/Mobile developers have Grafana access today | Unknown | PM, BA | Persona P3 rights review only (no design block) | U-01, N-006, OI-07 |
| V-10 | Masked text export of a few raw BCAS log lines (with the full label list) for parser fixtures | Screenshot only [Ideation p.17] | DevOps or a Developer | Parser build and parser property tests (Req 15); U3 | C-03 (A-061), SC-5, OI-09 |
| V-11 | Grafana MCP server / Grafana Assistant availability on self-hosted Grafana | Optional; likely no Assistant | System Architect, DevOps | Nothing in the MVP (Req 31.3: finder runs without them) | G-02, G-03 (A-043, A-044), X-003 |

### DD-01 When the Loki spike runs relative to design.md approval

**Tag:** Blocker for design.md · **Owner:** System Architect, DevOps
**Builds on:** C-08 (A-081), OI-01, brief §3 Stage 5 (Loki spike unit), V-01..V-08

**Question:** Should `design.md` wait for the verification results above, or be approved with them as explicit assumptions?

**Why this matters:** Waiting delays the whole Inception package; not waiting means parts of the Loki connector design may change after the spike.

**Options:**
1. **Approve design.md with V-01..V-11 as explicit assumptions — Recommended:** `design.md` states each working value, marks the sections that depend on it, and keeps limits and connection settings as parameters. The spike is unit U2 (brief §3 Stage 5); its results amend `design.md` through a versioned change entry (change log + audit row). *Rationale:* the brief places the spike in the MVP plan, no access is available during Inception (A-005), and the parameterized design absorbs most outcomes.
2. **Run V-01..V-08 before generating design.md:** design waits for DevOps and Developers. Lowest rework; longest delay.
3. **Generate design.md now, but hold approval of the Loki connector and DB connector sections until V-01, V-02 and V-08 are done:** the rest is approved now.
4. Other (please specify): _______________________

**Answer:**1

---

## 2. Architecture and components

### DD-02 Component boundaries

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** G-01 (A-042), R-01 (A-041), brief §3 Stage 4 component list, Req 31.5 (one connector interface)

**Question:** How are the components (UI, API/BFF, NL-to-filter, query builder + template registry, source connectors, parser/renderer, masking, audit, config/admin, metrics) packaged?

**Why this matters:** It decides the number of deployables, where security checks live, and how easily the repository can be split later.

**Options:**
1. **Modular monolith: one Go (Echo) service with internal packages per component, plus the LLM inference server as a separate process — Recommended:** packages such as `access`, `config`, `nl`, `query` (templates), `connector/loki`, `connector/db`, `parser`, `masking`, `audit`, `metrics`; one connector interface (Req 31.5); one place where grant checks, binding injection (Req 4.2–4.3) and masking run. *Rationale:* fewer than 10 users (A-021), best-effort availability (A-023), one internal server (A-018); package boundaries keep a later split possible (R-01).
2. **Two services: finder API + separate connector/query service:** the API never reaches Sources directly. More isolation, more deployment and contract work.
3. **Service per component:** highest operational cost for a pilot of this size.
4. Other (please specify): _______________________

**Answer:**1

### DD-03 Frontend–backend interface

**Tag:** Blocker for design.md · **Owner:** System Architect, Developers
**Builds on:** H-04 (A-040, PactFlow), workspace regression-testing strategy (Tier 1 API contract tests)

**Question:** How does the React frontend talk to the Go backend?

**Why this matters:** It fixes the API contract format that `design.md` documents and that Pact tests pin down.

**Options:**
1. **REST/JSON, documented in an OpenAPI file in the repository, with Pact consumer tests (frontend) and provider verification (backend) through PactFlow — Recommended:** *Rationale:* matches the chosen stack and PactFlow decision; TanStack Query works naturally with REST; OpenAPI gives one reviewable contract.
2. **REST/JSON with Pact only (no OpenAPI file):** less to maintain; contract only visible in Pact files.
3. **RPC style (for example gRPC-web or Connect) with generated clients:** stronger typing; adds tooling outside the chosen stack.
4. Other (please specify): _______________________

**Answer:**1

---

## 3. Loki connector

### DD-04 Query construction from templates

**Tag:** Blocker for design.md · **Owner:** System Architect, Developers
**Builds on:** L-03 (A-048), M-02 (A-071), M-08 (A-073), Req 4.2–4.3, 6.2, 6.4, 21; brief note that Loki indexes labels only; V-02, V-03

**Question:** How does a Query_Template turn a Filter_Set into a Loki query?

**Why this matters:** Identifier search inside lines can be slow over wide ranges; the query shape decides cost, correctness and how injection is prevented.

**Options:**
1. **Stream selector from the Source_Binding (injected server-side) + escaped line filters for identifiers and message text; parsing done in the finder — Recommended:** templates render `{<binding selectors>} |= "<value>"` (several values of one type as an escaped regex alternation or one query per value, decided in the spike); feature/service from the dictionary maps to selector labels where they exist, otherwise to line filters; every value is escaped by the template engine, never concatenated. *Rationale:* simplest query, no dependency on log format at query time, parser stays deterministic in one place (L-04).
2. **Line filters plus query-time parsing (`| json` / `| pattern` with label filters on parsed fields):** more precise matches on known fields; more Loki CPU and depends on stable payload format.
3. **Hybrid: option 1 by default; a template may add query-time parsing for a named field when the spike shows it is needed:** flexible; two template styles to test.
4. Other (please specify): _______________________

**Answer:**1

### DD-05 Pagination and Result_Cap strategy

**Tag:** Can-defer · **Owner:** Developers, System Architect
**Builds on:** Req 7.5, 14.3, 28.2; M-06 (A-047); V-02

**Question:** How does the connector honor the Result_Cap and report "more matches exist"?

**Why this matters:** It decides Loki load per search and whether users see the earliest or latest lines first.

**Options:**
1. **One `query_range` call with `limit = Result_Cap + 1` and a fixed direction; if more than Result_Cap lines come back, show Result_Cap lines and the "narrow your search" message — Recommended (proposal, to validate in the Loki spike):** *Rationale:* one call per search, predictable load, directly implements Req 7.5.
2. **Page through the window in successive calls until Result_Cap:** more complete within the cap; more calls per search.
3. **Split the window into fixed chunks run sequentially, stopping at Result_Cap:** smoother load on wide windows; most complex.
4. Other (please specify): _______________________

**Answer:**1

### DD-06 Default values for time windows, caps and limits

**Tag:** Can-defer · **Owner:** System Architect, BA (DB limits: DevOps and Hadi Septian approve)
**Builds on:** M-06 (A-047), D-03 (A-068), Req 7, 9, 23.2, 28.2, OI-03

**Question:** Which starting defaults go into the Project configuration?

**Why this matters:** Req 7 and 23 need concrete defaults to test; wrong values make searches empty or slow.

**Options:**
1. **Start with labelled proposal values, all per-Project parameters, re-set after the spike from measured latency — Recommended:** exact time ± 5 min; approximate time ± 30 min; Max_Range 24 h (never beyond Loki retention, V-07); Result_Cap 500 lines; Loki connector timeout 30 s; concurrent searches per Source 2; DB (once enabled) statement timeout 5 s, row cap 200, max range 7 days, concurrency 1. **All values are proposals, to validate in the Loki spike** (DB values approved by DevOps and Hadi Septian before enabling). *Rationale:* gives testable defaults now without claiming they are right.
2. **Leave all values blank until after the spike:** no wrong numbers; Req 7 tests wait for U2.
3. **BA proposes windows from the 24 requests; System Architect sets caps; no AI values:** owner-driven; needs a meeting.
4. Other (please specify): _______________________

**Answer:**1

---

## 4. NL-to-filter layer

### DD-07 Internal model serving approach

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** L-01 (A-029), L-03 (A-048), T-001 (manual first, NL on top), Req 10, SC-6

**Question:** How is the NL_Translator built?

**Why this matters:** It decides hardware needs, accuracy on Indonesian chat wording, and how much works without an LLM.

**Options:**
1. **Hybrid: deterministic pre-processing (dictionary terms, identifier patterns per Identifier_Mapping, time expressions such as "jam 16:04:44", "around 15.11", "tadi") plus a self-hosted open-weights LLM behind an internal inference server for what remains — Recommended:** the LLM gets the sentence, dictionary and identifier types only. Candidate inference servers and models are **evaluated, not chosen here**, against an NL test set built from RQ-01..RQ-24 and brief §2 (Req 10.7). *Rationale:* identifiers and times are the most common filters and are handled exactly by rules; the LLM covers free wording.
2. **Rules/dictionary-only parser first, LLM added later:** no model hosting in the pilot; weaker on free wording; NL share metric (Req 27.4) measures the rules parser only.
3. **LLM only (sentence in, Filter_Set out):** least code; most dependent on model quality and hardware.
4. Other (please specify): _______________________

**Answer:**1

### DD-08 Validation of NL output

**Tag:** Blocker for design.md · **Owner:** System Architect, Developers
**Builds on:** L-03 (A-048), Req 10.1, 10.5, 10.7, 6.5

**Question:** How is the NL_Translator output checked before it becomes Filter_Chips?

**Why this matters:** This is the control that keeps the LLM from producing anything other than known filters (C2, C5).

**Options:**
1. **JSON Schema for the Filter_Set (fixed field names, enums for Environment and identifier types) used for constrained/structured output where the inference server supports it, then always validated server-side against the schema and the active Project's vocabulary; unknown items dropped and reported (Req 10.5) — Recommended:** *Rationale:* schema validation does not trust the model; the same validator serves manual filters (Req 6.5).
2. **Free-text JSON from the model, validated server-side only:** works with any server; more dropped outputs.
3. **Tool/function-calling interface of the inference server, validated server-side:** depends on server support.
4. Other (please specify): _______________________

**Answer:**1

### DD-09 Behavior when the LLM is unavailable

**Tag:** Can-defer · **Owner:** System Architect, BA
**Builds on:** T-001, N-03 (A-023), Req 10, 29

**Question:** What does the Searcher see when the inference server is down or slow?

**Why this matters:** Manual filters must keep working so the pilot is not blocked by the model.

**Options:**
1. **Manual filters keep working; the NL box shows "natural-language search unavailable, use filters"; deterministic pre-processing (if DD-07 = 1) still fills the chips it can; an NL timeout is a parameter — Recommended:** *Rationale:* degrades gracefully and keeps metrics honest (search mode recorded, Req 10.6).
2. **NL box hidden while the server is down:** simpler UI; Searchers may not know why.
3. **Queue the sentence and notify when filters are ready:** adds state for little benefit with fewer than 10 users.
4. Other (please specify): _______________________

**Answer:**1

### DD-10 Where the inference server runs

**Tag:** Can-defer · **Owner:** DevOps, System Architect
**Builds on:** L-01 (A-029), H-01 (A-018), SC-6, deck slide 10 ask "internal host for the LLM"

**Question:** On which internal machine does the model run?

**Why this matters:** Model size and response time depend on available CPU/GPU; nothing may leave Soluix.

**Options:**
1. **A separate internal VM sized after the candidate evaluation in DD-07, reached by the finder over the internal network — Recommended:** *Rationale:* model load does not affect the finder; sizing is based on measured results, not guesses.
2. **Same internal server as the finder (CPU only):** one machine; smallest models only.
3. **An existing internal inference service, if Soluix has one:** reuse; availability unknown.
4. Other (please specify): _______________________

**Answer:** 1

---

## 5. Parser and renderer

### DD-11 Parser rules: code vs per-project configuration

**Tag:** Blocker for design.md · **Owner:** System Architect, Developers
**Builds on:** L-04 (A-049), O-01 (A-031), C-03 (A-061), Req 15, 16, 17.5; findings B4, B8, B9 [Ideation p.17]

**Question:** Where do parsing rules live, and what is shown when a line is only partly parsed?

**Why this matters:** Per-project formats differ; the choice decides whether onboarding a project needs code (Req 5.2).

**Options:**
1. **Fixed parsing pipeline in code (strip CRI/stdout prefix, strip ANSI, unwrap `log` JSON and nested escaped JSON) + per-project configuration as data (field list, field paths, display names); a line that fails any step is shown raw with "could not be simplified" (Req 15.4) — Recommended:** *Rationale:* the pipeline steps are the same for the known format, tests stay deterministic, and onboarding needs configuration only.
2. **Per-project rules registry (ordered regex/pattern steps in configuration):** handles new formats without code; harder to test and to keep deterministic.
3. **Option 1, plus a "partly simplified" state that shows the stripped message when JSON unwrap fails:** more readable for odd lines; one more state to test.
4. Other (please specify): _______________________

**Answer:** 2

---

## 6. Masking

### DD-12 Where masking runs

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** SEC-02 (A-051), SEC-05 (A-054), C-05 (A-079), Req 19, 20, 26.1. Masking never runs in the browser.

**Question:** At which point in the server is masking applied?

**Why this matters:** Every path that returns data (views, copies, escalation views, audit) must be masked unless the user holds the Unmask_Right (Req 19.5 invariant).

**Options:**
1. **Server-side, in one masking module applied to every response DTO before serialization; True_Raw served only by a separate, grant-checked and audited endpoint; unmasked data kept in memory for the request only — Recommended:** *Rationale:* one choke point to test against the masking invariant; the browser never holds unmasked data unless the user asked for it with the right.
2. **Server-side, immediately after fetch (before parsing); True_Raw re-fetched from the Source on demand:** strongest separation; True_Raw costs a second query.
3. **Server-side at both points (after fetch for audit/metrics paths, at response for views):** belt and braces; two code paths to keep in sync.
4. Other (please specify): _______________________

**Answer:** 1

### DD-13 Masking pattern format and default partial pattern

**Tag:** Can-defer · **Owner:** System Architect, PM
**Builds on:** SEC-02 (A-051), Req 19.2, 19.4, OI-10; finding B7 [Ideation p.17]

**Question:** How are Masking_Rules written, and what is the default partial pattern for CIF, account number and hashcard?

**Why this matters:** P5b edits these per Project (Req 19.4); the format decides what can be masked reliably.

**Options:**
1. **Two rule kinds in Project config: key rules (JSON field paths/names such as token, document body → full mask) and value rules (named regex per identifier type → partial mask); default partial pattern keeps the last 4 characters (proposal) — Recommended:** *Rationale:* key rules catch values with no fixed shape (tokens, base64 bodies); value rules catch identifiers wherever they appear in text.
2. **Value regex rules only:** one rule kind; misses tokens and document bodies without a stable pattern.
3. **Key rules only:** misses identifiers inside free message text.
4. Other (please specify): _______________________

**Answer:** 1

### DD-14 How the searched identifier is shown

**Tag:** Can-defer · **Owner:** PM, BA
**Builds on:** Req 19.3

**Question:** Is the identifier the Searcher typed shown in full in that Searcher's results?

**Why this matters:** Searchers must confirm they found the right log; others' identifiers stay masked.

**Options:**
1. **Show in full only exact matches of the value(s) the Searcher typed, only in that Searcher's response, and still masked in copies unless the copy type is unmasked — Recommended:** *Rationale:* confirms the match without exposing other values in forwarded copies.
2. **Show in full in both views and copies:** easier for forwarding; copies carry the identifier.
3. **Keep it partially masked but highlighted:** strictest; Searchers cannot verify the full value.
4. Other (please specify): _______________________

**Answer:**1

---

## 7. Finder's own data storage

### DD-15 Store for accounts, grants, config, dictionary, mapping, audit, metrics events, search history

**Tag:** Blocker for design.md · **Owner:** System Architect, DevOps
**Builds on:** H-01 (A-018), H-02 (A-019), Req 2, 5, 11–13, 26, 27

**Question:** Where does the finder keep its own data?

**Why this matters:** It shapes the ER model in `design.md`, backups, and how append-only audit is enforced.

**Options:**
1. **PostgreSQL on the internal server (container or package), one schema for the finder — Recommended:** relational tables plus JSONB for per-project configuration. *Rationale:* concurrent users, database roles can deny UPDATE/DELETE on audit tables, standard backup tooling; Go drivers are mature.
2. **SQLite file next to the service:** simplest for fewer than 10 users; weaker for permission-based append-only and concurrent writes.
3. **New schema on an existing Soluix database server:** reuses operations; couples the finder to a server it does not own.
4. Other (please specify): _______________________

**Answer:** 1

### DD-16 Audit log tamper evidence

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** SEC-04 (A-053), Req 26.4, 2.3

**Question:** How is "append-only for all roles" enforced?

**Why this matters:** Audit is the main evidence for unmask and true-raw use (C-05).

**Options:**
1. **Database permissions (application role may INSERT and SELECT audit rows only) plus a hash chain (each row stores the hash of the previous row) with a verification check — Recommended:** *Rationale:* permissions stop changes through the app; the chain shows if someone with database admin rights altered rows.
2. **Database permissions only:** simpler; changes by a database admin are not detectable.
3. **Option 1 plus a periodic export to write-once storage:** strongest; needs storage that is not known yet.
4. Other (please specify): _______________________

**Answer:** 1

---

## 8. Identity and access

### DD-17 Password hashing

**Tag:** Can-defer · **Owner:** System Architect, Developers
**Builds on:** H-03 (A-020), Req 1.5 (accepted A-086)

**Question:** Which password hashing algorithm is used?

**Why this matters:** Local accounts make the finder responsible for password storage.

**Options:**
1. **argon2id (Go `golang.org/x/crypto/argon2`), parameters held in configuration and set from current OWASP Password Storage guidance at build time — Recommended:** *Rationale:* memory-hard, the OWASP first choice; no input-length limit issue.
2. **bcrypt (Go `golang.org/x/crypto/bcrypt`):** well known; inputs beyond 72 bytes are truncated.
3. **scrypt:** acceptable; less common in Go web code.
4. Other (please specify): _______________________

**Answer:** 1

### DD-18 Session mechanism and idle timeout default

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** Req 1.4, 2.2, 3.6 (accepted A-086), H-03

**Question:** How are Sessions held, and what is the default idle timeout?

**Why this matters:** Req 2.2 and 3.6 require ending sessions and applying revocations on the next request.

**Options:**
1. **Server-side sessions stored in the finder database; opaque session ID in an HttpOnly, Secure, SameSite cookie; CSRF token on state-changing calls; idle timeout parameter, default 30 minutes (proposal) — Recommended:** *Rationale:* deleting the session row ends it immediately (Req 2.2); every request re-reads grants (Req 3.6).
2. **Short-lived JWT access token + refresh token with a server-side revocation list:** stateless reads; revocation logic still needed.
3. **Stateless JWT only:** cannot meet Req 2.2 without waiting for expiry.
4. Other (please specify): _______________________

**Answer:** 1

### DD-19 Grants model

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** SEC-01 (A-050), M-04 (A-026), C-01 (A-059), C-05 (A-079), SEC-04 (audit readers), Req 3, 5.5, 20, 26.3

**Question:** How are roles and grants modeled?

**Why this matters:** It is the core of the security model in `design.md` and of every access test.

**Options:**
1. **RBAC tables: `membership(user, project, environment)` for search; `project_role(user, project, role ∈ {business_admin, access_admin})` for P5a/P5b; `unmask_grant(user, project, environment)`; `global_role(user, role ∈ {pm, system_architect})` for metrics and audit reading; one authorization function checked on every request — Recommended:** *Rationale:* maps one-to-one to Req 3, 5.5, 20.2 and 26.3; easy to audit and test.
2. **Permission strings per user (for example `search:BCAS:UAT`, `unmask:BCAS:UAT`):** flexible; harder to list and review.
3. **Policy engine (candidates such as Casbin or OPA, to evaluate):** expressive; more moving parts than the pilot needs.
4. Other (please specify): _______________________

**Answer:** 1

---

## 9. Secrets and configuration

### DD-20 Where Service_Identity credentials and Source_Bindings live

**Tag:** Blocker for design.md · **Owner:** DevOps
**Builds on:** H-02 (A-019), G-06 (A-045), Req 4.4, 5.3, 30.5

**Question:** Where are source credentials and connection settings stored, and how are they rotated?

**Why this matters:** Credentials must stay on the server and out of the code; Loki may need none today (V-01) but the DB will.

**Options:**
1. **Non-secret binding data (endpoints, selectors, limits) in the finder database; secrets in root-readable files or environment variables on the server, referenced by name from the binding; manual rotation with a documented runbook — Recommended for the pilot:** *Rationale:* works on one internal server with no extra service; keeps secrets out of the database and the repository.
2. **An internal secret vault, if Soluix runs one:** better rotation and audit; availability unknown.
3. **Secrets encrypted in the finder database with a master key from the environment:** editable in the admin UI; the master key becomes the single secret to protect.
4. Other (please specify): _______________________

**Answer:**1

---

## 10. Deployment on the internal server (includes IaC scope)

### DD-21 Runtime, frontend serving, TLS and network path

**Tag:** Blocker for design.md · **Owner:** DevOps
**Builds on:** H-01 (A-018), H-02 (A-019), N-03 (A-023), R-01 (A-041), Req 30

**Question:** How are the finder, its database and the inference server run on the internal server?

**Why this matters:** It fixes the deployment diagram, the files kept in the repository, and the firewall rules DevOps opens.

**Options:**
1. **Docker Compose: finder (Go binary also serving the built SPA on the same origin), PostgreSQL, reverse proxy with TLS from an internal CA; inference server on its own host per DD-10; Compose files and scripts in a `deploy/` directory of the repository; outbound rules from the server only to each Project's Loki and DB endpoints — Recommended:** *Rationale:* one command to start and roll back, same-origin cookies (DD-18), no cluster dependency; minimal IaC.
2. **systemd services with binaries on the VM, configured by scripts or Ansible:** no container runtime; more host setup per change.
3. **Deploy into an existing Kubernetes cluster:** reuses platform tooling; ties the finder to the cluster it reads logs from.
4. Other (please specify): _______________________

**Answer:**1

---

## 11. Observability of the finder itself

### DD-22 Logs, metrics and health of the finder

**Tag:** Can-defer · **Owner:** DevOps, Developers
**Builds on:** Req 29.3 (source outages), Req 19 (no unmasked values), N-03 (A-023)

**Question:** How is the finder itself observed?

**Why this matters:** The finder handles banking data; its own logs must not become a new leak.

**Options:**
1. **Structured JSON logs (Go `log/slog`) to stdout, with request ID, user ID, project, environment and outcome only, never filter values, sentences or log content; `/healthz` and `/readyz` endpoints; a metrics endpoint in Prometheus format for searches, latency and Source errors — Recommended:** *Rationale:* standard and cheap; sensitive values are excluded by design and checked by a test.
2. **Option 1 and ship the finder's logs to an existing internal Loki:** central view; needs a target Loki that is not a project Loki.
3. **Plain logs and health endpoints only:** least effort; no latency data for Req 28.
4. Other (please specify): _______________________

**Answer:**1

---

## 12. Pilot instrumentation

### DD-23 Storage and reporting of pilot metrics

**Tag:** Can-defer · **Owner:** PM, Developers
**Builds on:** MET-01..MET-04 (A-033 to A-035, A-076), Req 27, accepted A-086 (median/p90, "unclassified")

**Question:** How are pilot events stored and shown to the PM?

**Why this matters:** The pilot decision (roll out, adjust, revisit) depends on these numbers.

**Options:**
1. **Events table in the finder database (search, view open, escalation, session), weekly aggregates computed on read, a PM page in the finder plus CSV export; ping counts entered by the PM (Req 27.5) — Recommended:** *Rationale:* meets Req 27.4 inside the finder; CSV supports the deck and offline review.
2. **PM page only, no export:** less work; numbers must be retyped into the deck.
3. **Export only, analysis outside the finder:** least code; does not meet Req 27.4 as written.
4. Other (please specify): _______________________

**Answer:**1

---

## 13. Testing strategy at design level

### DD-24 Test layers

**Tag:** Blocker for design.md · **Owner:** QA, Developers
**Builds on:** H-04 (PactFlow), workspace regression-testing strategy (Tier 1 contracts + E2E, Tier 2 pure logic, Tier 3 integration)

**Question:** Which test layers does `design.md` commit to?

**Why this matters:** It sets what evidence each unit's Bolt must produce in Stage 5.

**Options:**
1. **Tier 1: Pact consumer tests (frontend) and provider verification (Go) via PactFlow in CI; browser E2E tests (Playwright, candidate) for the RQ-01 happy path, "nothing found", access denied, Unmask toggle and Escalation_Link. Tier 2: unit tests for pure logic (time conversion, Filter_Set validation, templates, parser, masking). Tier 3: Loki connector integration tests against a local Loki container loaded with masked fixture lines (V-10) — Recommended:** *Rationale:* follows the workspace strategy; tests external behavior; no real Loki needed in CI.
2. **Option 1 without browser E2E (contract + unit + integration only):** faster; user flows untested end to end.
3. **Option 1 plus a nightly smoke test against BCAS UAT Loki with the read-only identity:** catches drift in real labels; needs network access from CI.
4. Other (please specify): _______________________

**Answer:** 1

### DD-25 Correctness properties (property-based tests)

**Tag:** Blocker for design.md · **Owner:** QA, System Architect
**Builds on:** Req 4.3, 6.2, 6.5, 7.1, 7.4, 8.5, 10.5, 10.7, 15.2, 15.6, 17.5, 19.5

**Question:** How many correctness properties does `design.md` define for property-based testing?

**Why this matters:** Properties cover the pure logic where one wrong edge case leaks data or finds the wrong log.

**Options:**
1. **Essential only — Recommended:** candidate properties: (a) WIB→UTC→WIB round trip (8.5); (b) masking invariant: no matched value appears unmasked in any masked output (19.5); (c) parser determinism, round trip and no field dropped (15.2, 15.6, 17.5); (d) Filter_Set validation rejects every out-of-pattern value and accepts every valid one, including NL output (6.5, 10.5, 10.7); (e) time-window bounding: no query runs without a window or beyond Max_Range (7.1, 7.4); (f) every rendered template contains the active binding's selectors and escaped values (4.3). Libraries are candidates to confirm with Developers (Go: `pgregory.net/rapid`; TypeScript: `fast-check`), at least 100 iterations each. *Rationale:* covers the security and correctness core without testing I/O.
2. **Skip properties; example-based tests only:** less effort; Req 8.5, 15.6, 17.5 and 19.5 are written as properties and would be checked by examples only.
3. **Comprehensive: option 1 plus multi-value labelling (6.2), grouping by correlation (24), metrics classification (27.6–27.7):** widest coverage; more test code in the pilot.
4. Other (please specify): _______________________

**Answer:** 1

---

## 14. Content of design.md

### DD-26 Scope of design.md

**Tag:** Blocker for design.md · **Owner:** System Architect
**Builds on:** brief §3 Stage 4; workspace design rules (component, sequence and ER diagrams)

**Question:** Confirm what `design.md` contains.

**Why this matters:** It is the Stage 4 deliverable reviewed at the Human Review Gate and the source of deck slide 7.

**Options:**
1. **Full scope — Recommended:** overview with build-vs-reuse recap; component diagram; sequence diagram for the RQ-01 happy path (sentence → chips → confirm → Loki query → parse → mask → view); sequence for Escalation_Link open; ER diagram of the finder data; per-project configuration model; API contract (paths, methods, payloads, status codes); security model (grants, masking, audit, secrets, RK-002); how Loki, Grafana and the database are used; deployment view; verification assumptions V-01..V-11; risks (Loki label-index cost over wide ranges, timezone mismatch UTC vs +0700, empty `correlation_id`, no DB read-replica, Loki without auth, LLM accuracy on Indonesian wording); correctness properties per DD-25; decision log back to this file. *Rationale:* covers every item the brief lists for Stage 4.
2. **Option 1 without the API contract (written in Construction from OpenAPI):** shorter; Pact work starts without a reviewed contract.
3. **Architecture and security only; data model and API in Stage 5:** fastest; less to review at the gate.
4. Other (please specify): _______________________

**Answer:** 1

---

## Decision index

| ID | Title | Tag | Owner |
|---|---|---|---|
| DD-01 | When the Loki spike runs relative to design.md approval | Blocker | System Architect, DevOps |
| DD-02 | Component boundaries | Blocker | System Architect |
| DD-03 | Frontend–backend interface | Blocker | System Architect, Developers |
| DD-04 | Loki query construction from templates | Blocker | System Architect, Developers |
| DD-05 | Pagination and Result_Cap strategy | Can-defer | Developers, System Architect |
| DD-06 | Default values for time windows, caps and limits | Can-defer | System Architect, BA (DB: DevOps, Hadi Septian) |
| DD-07 | Internal model serving approach | Blocker | System Architect |
| DD-08 | Validation of NL output | Blocker | System Architect, Developers |
| DD-09 | Behavior when the LLM is unavailable | Can-defer | System Architect, BA |
| DD-10 | Where the inference server runs | Can-defer | DevOps, System Architect |
| DD-11 | Parser rules: code vs per-project configuration | Blocker | System Architect, Developers |
| DD-12 | Where masking runs | Blocker | System Architect |
| DD-13 | Masking pattern format and default partial pattern | Can-defer | System Architect, PM |
| DD-14 | How the searched identifier is shown | Can-defer | PM, BA |
| DD-15 | Finder data store | Blocker | System Architect, DevOps |
| DD-16 | Audit log tamper evidence | Blocker | System Architect |
| DD-17 | Password hashing | Can-defer | System Architect, Developers |
| DD-18 | Session mechanism and idle timeout default | Blocker | System Architect |
| DD-19 | Grants model | Blocker | System Architect |
| DD-20 | Where Service_Identity credentials and Source_Bindings live | Blocker | DevOps |
| DD-21 | Runtime, frontend serving, TLS and network path | Blocker | DevOps |
| DD-22 | Logs, metrics and health of the finder | Can-defer | DevOps, Developers |
| DD-23 | Storage and reporting of pilot metrics | Can-defer | PM, Developers |
| DD-24 | Test layers | Blocker | QA, Developers |
| DD-25 | Correctness properties | Blocker | QA, System Architect |
| DD-26 | Scope of design.md | Blocker | System Architect |

Blockers: 17 (DD-01, 02, 03, 04, 07, 08, 11, 12, 15, 16, 18, 19, 20, 21, 24, 25, 26). Can-defer: 9 (DD-05, 06, 09, 10, 13, 14, 17, 22, 23).
