# Requirements Document

## Introduction

AI-DLC Inception, Stage 3 (Specify). Status: **Approved 2026-11-25 (audit A-085)** (drafted 2026-11-25, after Stage 2 approval, audit A-078).

The Simplified Log Finder ("the Finder") is a greenfield, internal, self-serve web application that sits in front of the log sources Soluix teams already use (self-hosted Grafana / Loki, a project database, Kubernetes pod logs). Non-technical teammates find and read a log themselves by time, identifier, service/feature or error code, see a simplified view with the raw log one click away, and escalate to an engineer only when the lookup cannot be resolved. The Finder is a thin, input/output-agnostic layer (G-01, A-042); it is not a new observability platform and it does not replace Grafana, the database or k9s for engineers (C4).

**Problem being solved.** 24 of 24 sampled chat captures are the same ask: an engineer is pinged to pull a log; only the time, the identifier and the feature change [brief §1.1; Ideation p.4–11]. Root cause (5 Whys) [brief §1.1; Presenter p.6]:

| ID | Root-cause statement |
|---|---|
| RC-1 | There is no standard, self-serve path from a non-technical question to a readable log |
| RC-2 | Requesters know what they need (time, CIF, error) but cannot fetch it; nothing turns time + CIF + feature into a query |
| RC-3 | Logs live in Grafana, the database and k9s; each needs technical skill and access, which sit with a few people |
| RC-4 | Raw logs are dense JSON (stdout prefixes, ANSI codes, nested escaped JSON) that is hard to read |
| RC-5 | Lookup is treated as a favor, not as a capability with an owner, so it is neither measured nor improved |

**Pilot hypothesis** [brief §1.1]: if one real project team (BCAS, S-02) uses the Finder for everyday lookups, engineer log-request pings fall by 65% and requesters get a readable log in under 2 minutes.

**Sources and citation keys.**
- Brief: `prompt-inception.md`, cited `§x.y`.
- Decisions: question IDs in `aidlc-docs/inception/stage2-questions.md` (the requirements decision file, approved 2026-11-25) and decision IDs in `aidlc-docs/audit.md` (A-xxx, T-xxx).
- Constraints C1–C9: `aidlc-docs/inception/stage2-actors-constraints-assumptions.md` §2. C1 security first; C2 LLM produces structured filters only; C3 multi-project architecture, pilot project onboarded first; C4 must not replace Grafana / DB / k9s; C5 must not become a backdoor; C6 UAT and RC only; C7 internal "local first" hosting; C8 no direct Grafana / DB / k9s access for BA/QA (T-002); C9 next phase (images, more sources) must not be blocked.
- Real requests RQ-01 to RQ-24: the 24 chat captures [Ideation p.4–11], numbered in image order, three per page; listed in the Traceability Matrix section. Identifiers are placeholders (`<CIF>`, `<ACCOUNT_NO>`, `<SHA256_HASH>`, `<PROGRAM_ID>`, `<LICENSE_ID>`). People are referred to by role.

**Scope of this document.** MVP requirements only. Values that depend on Loki performance (time windows, result caps, per-search latency) are parameters whose defaults are proposed in Stage 4 (M-06, N-02). Technical choices (stack, repository layout, connector internals) are recorded as Stage 4 constraints, not as requirements.

## Glossary

### System and component terms

- **Finder**: the Simplified Log Finder web application (UI and server) specified here.
- **Project**: a client project onboarded to the Finder (pilot: BCAS). Each Project has its own configuration, sources, members and audit scope.
- **Environment**: a test environment of a Project. MVP values: `UAT`, `RC`. Production is excluded.
- **Source**: an existing log or data store the Finder reads: Loki (self-hosted, reached server-side), Kubernetes pod logs (served through Loki in the MVP), and the Project database.
- **Source_Binding**: per-Project, per-Environment, per-Source configuration held server-side: endpoint, service identity, environment-to-label mapping (label selectors plus optional payload filter such as `project_name`), and limits.
- **Service_Identity**: a dedicated read-only credential per Project and per Source, approved by DevOps and the Project lead (G-06).
- **Query_Template**: a predefined, parameterized LogQL or SQL query owned by the Finder. Filter values are bound as parameters; the Source_Binding selectors are injected server-side.
- **Filter_Set**: the structured set of filters for one search: Time_Window, Environment, identifiers, service/feature, error code.
- **Filter_Chip**: the UI element that shows one filter of the Filter_Set; the Searcher can edit or remove it before running the search.
- **Time_Window**: the bounded start and end time of a search, entered and displayed in WIB.
- **Max_Range**: the largest Time_Window a Project allows for one search (parameter per Project; default proposed in Stage 4).
- **Result_Cap**: the maximum number of log lines (or rows) returned by one search (parameter per Project; default proposed in Stage 4).
- **NL_Translator**: the internal/local LLM component that turns a user sentence into a Filter_Set. It receives only the sentence and the Project's dictionary and identifier vocabulary; it never receives log content (L-01, L-03).
- **Business_Term_Dictionary**: per-Project mapping of business terms (often Indonesian) to services, modules, endpoints or message patterns.
- **Identifier_Mapping**: per-Project mapping of each identifier type (CIF, account number, hashcard, program ID, license ID, request ID, endpoint path) to the log field(s) or DB column(s) where it appears.
- **Parser**: the deterministic component that turns a raw log line into structured fields (L-04).
- **Simplified_View**: the readable view of a log line produced by the Parser: selected fields only, masked, times in WIB.
- **Raw_View**: the pretty-printed original log line with every field kept, masked by default, original timestamps unchanged.
- **True_Raw**: the original log line byte-for-byte, unmasked, available only to holders of the Unmask_Right.
- **Masking_Rules**: per-Project patterns that hide or partially mask sensitive values (tokens, document content, CIF, account numbers, hashcards).
- **Unmask_Right**: a grant to one user, for one Project and one Environment, to switch masking off and to view or copy True_Raw (C-01, C-05).
- **Audit_Log**: the append-only record of security-relevant user actions (SEC-04).
- **Escalation**: the "Ask an engineer" action for a lookup the Searcher cannot resolve.
- **Escalation_Link**: the only shareable link in the MVP; it opens the escalated search after re-checking the viewer's access (C-07).
- **Search_History**: the list of a user's own previous searches in a Project.
- **Session**: one sign-in period of a user in the Finder; used for pilot metrics (MET-02, MET-03).
- **Pilot_Metrics**: the measurements the PM reads to evaluate the pilot (MET-01 to MET-04).
- **WIB**: Western Indonesia Time, UTC+07:00. Users state and read times in WIB.
- **UTC**: Coordinated Universal Time; Loki line timestamps are UTC.
- **correlation_id**: application payload field meant to link lines of one request; often empty [Ideation p.17].
- **X-Request-Id / traceparent**: request headers seen inside some `log_data` payloads [Ideation p.17]; fallback linking keys (M-09).

### Business terms (kept verbatim; they are test data)

| Term | Meaning (as used in the requests) | Seen in |
|---|---|---|
| ganti rek sumber | change of source account | RQ-01, RQ-05; §2 |
| penutupan kartu | card closure | RQ-10; §2 |
| buka blokir | unblock (card/account) | RQ-11; §2 |
| saldo / buka saldo | balance / open balance | RQ-20; §2 |
| informasi rekening | account information | RQ-20 |
| norek | account number (nomor rekening) | RQ-04, RQ-20 |
| ganti sof | change source of funds | RQ-12 |
| CWLD (`external_opening_account`) | a submission flow named in the request; maps to `external_opening_account` per the requester | RQ-07; §2 |
| refund | refund transaction | RQ-09; §2 |
| closed amount | a feature named in the request | RQ-13 |
| laporan kartu debit | debit card report (filter defect) | RQ-14 |
| upgrade kartu | card upgrade | RQ-17; §2 |
| export excel | export to Excel | RQ-18 |
| liveness / get token | liveness check and the token request | RQ-21; §2 |
| simplify auth | a flow named in the request | RQ-21 |
| license_id | license identifier | RQ-22 |
| VKYC | video KYC (in the UI mockup chips) | Presenter p.1; M-03 |
| H2H | host-to-host integration | RQ-24; §2 |
| hashcard | SHA-256 hex hash of a card | RQ-11, RQ-12; §2 |
| trx | transaction | RQ-06 |
| minta log / minta lognya | "please give me the log" | all 24 requests |

## Personas (product user roles)

These are roles inside the Finder, not project-team roles [§3 Stage 3]. A person may hold several roles and belong to several Projects (SEC-01).

| ID | Product role | What the role does in the Finder | Rights in the MVP | Source |
|---|---|---|---|---|
| P1 | Business Analyst (Searcher) | Primary searcher: finds and reads logs, copies simplified or raw output, often to forward to the client | Search in granted Projects/Environments; masked views; Unmask_Right only if granted | §1.1, §3; RQ-21, RQ-22 |
| P2 | QA Engineer (Searcher) | Primary searcher, heavy during UAT/regression | Same as P1 | §1.1, §3; RQ-01, RQ-05, RQ-11 |
| P3 | FE / Mobile Developer as Searcher | Sends the same requests as BA/QA | Same as P1 (U-01, A-036). Whether these developers use Grafana today is unknown (open item OI-07) | U-01; RQ-06, RQ-13 to RQ-17, RQ-23 |
| P4 | Developer / Engineer (escalation target) | Receives Escalations; keeps using Grafana, the database and k9s directly (C4) | Opens Escalation_Links in Projects where the engineer is a member; may also search | O-03, C-07, G-08 |
| P5a | Project Admin: business configuration | Maintains the Business_Term_Dictionary, Identifier_Mapping and Simplified_View fields | Held by a BA per Project (M-04). Dictionary owner: BA, Developers confirm mappings (M-03) | M-03, M-04 |
| P5b | Project Admin: access manager | Manages local accounts, Project membership, Environment access, Unmask_Right grants, Source_Bindings and limits | Held by DevOps (M-04) | M-04, H-03, C-05 |
| P6 | Project Manager | Reads Pilot_Metrics; reads the Audit_Log | Read-only metrics and audit | §1.1, MET-01 to MET-04, SEC-04 |

**Named owners recorded for the pilot Project (BCAS):** read-only access approver Yosia Alvin together with DevOps (G-06, C-02, A-060); database limits approver Hadi Septian, DevOps and database owner (D-03, A-068); Business_Term_Dictionary owned by a BA (M-03); Audit_Log readers PM and System Architect (SEC-04).

## Requirements

### Requirement 1: Sign in with a local account

**User Story:** As a Searcher (P1, P2, P3), I want to sign in to the Finder with my own account, so that my access, my searches and my audit trail are tied to me.

#### Acceptance Criteria

1. WHEN a user submits a valid username and password, THE Finder SHALL start a Session for that user and show only the Projects the user is a member of.
2. IF a user submits an invalid username or password, THEN THE Finder SHALL refuse the sign-in, show a message that does not reveal which of the two was wrong, and record the failed attempt in the Audit_Log.
3. WHILE a user account is disabled, THE Finder SHALL refuse every sign-in attempt for that account.
4. WHEN a Session has been inactive for the Project-independent idle timeout (value proposed in Stage 4), THE Finder SHALL end the Session and require a new sign-in.
5. THE Finder SHALL store passwords only as salted one-way hashes.

**Example (Given/When/Then):** Given a QA Engineer with a local account who is a member of BCAS UAT, When the QA Engineer signs in, Then the Finder shows BCAS with Environment UAT only and starts a Session.

**Traceability:** Real request: none directly (prerequisite for all RQ-01..RQ-24). Root cause: RC-1. Constraints: C1, C5. Decisions: H-03 (A-020), SEC-01, SEC-04.

### Requirement 2: Manage local accounts

**User Story:** As a Project Admin access manager (P5b), I want to create, disable and remove local accounts, so that only current team members can use the Finder.

#### Acceptance Criteria

1. WHEN the access manager creates an account, THE Finder SHALL create the account with no Project membership, no Environment access and no Unmask_Right.
2. WHEN the access manager disables or removes an account, THE Finder SHALL end every active Session of that account before serving its next request and refuse further sign-ins.
3. WHEN an account is removed, THE Finder SHALL keep the Audit_Log entries of that account unchanged.
4. THE Finder SHALL record every account creation, disabling, removal and password reset in the Audit_Log with the acting access manager.
5. THE Finder SHALL show the access manager a list of accounts with last sign-in time, so that unused accounts can be found and removed.

**Example (Given/When/Then):** Given a QA Engineer leaves the BCAS team, When the access manager removes the QA Engineer's account, Then the QA Engineer's open Session ends and the QA Engineer's earlier searches remain in the Audit_Log.

**Traceability:** Real request: none directly. Root cause: RC-5 (lookup becomes a capability with an owner). Constraints: C1, C5. Decisions: H-03 (A-020), M-04 (A-026), audit note N-003.

### Requirement 3: Access per Project and Environment

**User Story:** As a Project Admin access manager (P5b), I want to grant access per Project and per Environment using least privilege, so that each user can search only where the user works, including users who work on several Projects.

#### Acceptance Criteria

1. THE Finder SHALL deny access to every Project and Environment for which a user holds no explicit grant.
2. WHEN the access manager grants a user access to a Project and Environment, THE Finder SHALL allow that user to search that Project and Environment only.
3. WHERE a user is a member of several Projects, THE Finder SHALL keep each Project's searches, results, Search_History and configuration separate and show the active Project on every screen.
4. IF a user requests a Project or Environment without a grant (through the UI, an Escalation_Link or a direct server call), THEN THE Finder SHALL return an access-denied response that reveals no log content or result count, and record the attempt in the Audit_Log.
5. THE Finder SHALL record every grant and revocation in the Audit_Log with the acting access manager.
6. WHEN a grant is revoked, THE Finder SHALL apply the revocation to the user's next request.

**Example (Given/When/Then):** Given a BA granted BCAS UAT only, When the BA asks for "log ganti rek sumber di RC" (RQ-01 wording), Then the Finder states that the BA has no access to RC for BCAS and runs no query.

**Traceability:** Real requests: RQ-01, RQ-07 (RC), RQ-11, RQ-20, RQ-21 (UAT). Root cause: RC-3. Constraints: C1, C5, C6, C8. Decisions: SEC-01 (A-050), M-04, S-03 (A-017).

### Requirement 4: Project isolation and read-only Source access

**User Story:** As a Project Admin access manager (P5b), I want every query to run with that Project's read-only Service_Identity and Source_Binding, so that a user of one Project can never read another Project's logs and the Finder never writes to a Source.

#### Acceptance Criteria

1. THE Finder SHALL execute every Source query server-side with the Service_Identity of the active Project and Source.
2. THE Finder SHALL inject the Source_Binding (endpoint, label selectors, payload filter) of the active Project and Environment into every Query_Template, and SHALL check the user's grant (Requirement 3) before execution.
3. IF a Query_Template rendered for execution lacks the Source_Binding selectors of the active Project and Environment, THEN THE Finder SHALL refuse to execute the query and record the refusal in the Audit_Log.
4. THE Finder SHALL keep Source endpoints and credentials on the server; the browser SHALL receive only rendered results.
5. WHEN a Source_Binding is created or enabled, THE Finder SHALL require a recorded approval by DevOps and the Project lead (BCAS: Yosia Alvin) before the binding can be used.
6. WHILE Loki is reachable without authentication (working assumption G-04, unverified per C-08), THE Finder SHALL rely on the per-Project Loki endpoint (one Loki per Project, G-05) plus the server-side checks in criteria 2 and 3 for Loki isolation.

**Example (Given/When/Then):** Given a QA Engineer who is a member of BCAS only, When the QA Engineer searches "penutupan kartu 17:23:26 CIF `<CIF>`" (RQ-10), Then the query runs against the BCAS UAT Loki endpoint with the BCAS UAT selectors and no other Project's Source is contacted.

**Traceability:** Real request: RQ-10 (example); all requests. Root cause: RC-3. Constraints: C1, C3, C5. Decisions: G-06 (A-045, A-060), G-07 (A-046), G-05 (A-064), G-04 / C-08 (A-063, A-081), risk RK-002.

### Requirement 5: Per-Project configuration and onboarding

**User Story:** As a Project Admin (P5a, P5b), I want to onboard a Project with its own Environments, Sources, dictionary, mappings and parameters, so that the Finder serves several Projects from day one while only the pilot Project is onboarded first.

#### Acceptance Criteria

1. THE Finder SHALL hold Environments, Source_Bindings, Business_Term_Dictionary, Identifier_Mapping, Masking_Rules, Simplified_View fields, Max_Range, Result_Cap and default Time_Windows as separate configuration per Project.
2. WHEN a new Project is onboarded, THE Finder SHALL require no code change, only configuration.
3. THE Finder SHALL keep configuration and secrets outside the application code, so that the host can change without rebuilding (H-02).
4. WHEN a configuration value changes, THE Finder SHALL record the change, the previous value and the acting Project Admin in the Audit_Log.
5. THE Finder SHALL let P5a edit only business configuration (dictionary, mappings, view fields) and P5b edit only access and Source configuration (accounts, grants, Source_Bindings, limits, Masking_Rules).
6. WHERE an Environment has no approved Source_Binding (for example RC before its Loki data is confirmed), THE Finder SHALL show that Environment as "not yet available" and run no search there.

**Example (Given/When/Then):** Given BCAS is onboarded with UAT only, When the access manager adds RC with an approved Source_Binding, Then BCAS users with an RC grant can search RC, as needed for RQ-07 ("CWLD ... yang gagal di RC").

**Traceability:** Real requests: RQ-01, RQ-07 (RC), all BCAS requests. Root cause: RC-5. Constraints: C3, C6, C9. Decisions: S-01 (A-015), S-02 (A-056), S-03 (A-017), M-04 (A-026), H-02 (A-019), M-06 (A-047).

### Requirement 6: Manual filters with editable chips

**User Story:** As a Searcher (P1, P2, P3), I want to search by time, one or more identifiers, service/feature, error code and Environment using filters shown as editable chips, so that I can find the log myself without asking an engineer.

#### Acceptance Criteria

1. THE Finder SHALL offer manual filters for Time_Window, Environment (UAT, RC), identifier type and value, service or business feature, and error code or message text.
2. WHEN a Searcher enters several values for one identifier type (for example three CIFs), THE Finder SHALL search for lines matching any of the values and label each result with the value it matched.
3. WHEN a filter is added, THE Finder SHALL show the filter as a Filter_Chip that the Searcher can edit or remove before running the search.
4. WHEN the Searcher runs a search, THE Finder SHALL execute only Query_Templates with the Filter_Set values bound as parameters.
5. IF a filter value contains characters outside the allowed pattern for its identifier type (per Identifier_Mapping), THEN THE Finder SHALL reject the value with a message naming the expected format and run no query.
6. THE Finder SHALL keep the Filter_Chips of the last search visible with the results, so that the Searcher can adjust and re-run.

**Example (Given/When/Then):** Given a QA Engineer with BCAS RC access, When the QA Engineer enters time "around 15:11", Environment RC, feature "CWLD", and CIFs `<CIF1>`, `<CIF2>`, `<CIF3>` (RQ-07), Then the Finder shows five editable chips and, after running, results labelled by the CIF each line matched.

**Traceability:** Real requests: RQ-01, RQ-02, RQ-05, RQ-07, RQ-10, RQ-11, RQ-12, RQ-17, RQ-20, RQ-24; brief §2 patterns (a)–(d). Root causes: RC-1, RC-2. Constraints: C2, C5, C8. Decisions: T-001, M-01 (A-024); UI mockup [Presenter p.1].

### Requirement 7: Mandatory bounded Time_Window and Result_Cap

**User Story:** As a Project Admin access manager (P5b), I want every search to have a bounded Time_Window and a Result_Cap set per Project, so that searches inside log lines stay fast and cannot overload a Source.

#### Acceptance Criteria

1. THE Finder SHALL require a Time_Window on every search before any Source is queried.
2. WHEN a Searcher enters an exact time (for example "16:04:44"), THE Finder SHALL propose a Time_Window around that time using the Project's exact-time default and show the Time_Window as an editable chip.
3. WHEN a Searcher enters an approximate time or range (for example "around 15.11" or "11:22 - 11:23"), THE Finder SHALL propose a Time_Window using the Project's approximate-time default and show the Time_Window as an editable chip.
4. IF a requested Time_Window exceeds the Project's Max_Range, THEN THE Finder SHALL refuse the search and state the Max_Range.
5. WHEN a search matches more lines than the Result_Cap, THE Finder SHALL return the first lines up to the Result_Cap and state that more matches exist and how to narrow the search.
6. THE Finder SHALL read the exact-time default, approximate-time default, Max_Range and Result_Cap from the Project configuration (defaults proposed in Stage 4).

**Example (Given/When/Then):** Given BCAS defaults are configured, When a QA Engineer searches "log perubahan ganti rek sumber di jam 16:04:44 cif `<CIF>`" (RQ-05), Then the Finder shows a Time_Window chip around 16:04:44 WIB that the QA Engineer can widen up to the Max_Range.

**Traceability:** Real requests: RQ-05, RQ-07, RQ-20, RQ-24. Root cause: RC-2. Constraints: C5, A4 (no read-replica). Decisions: M-06 (A-047), M-02 (A-071, identifiers inside log lines), N-02 (A-022); brief §3 Stage 4.

### Requirement 8: Timezone handling (WIB in, WIB out, UTC to Sources)

**User Story:** As a Searcher (P1, P2, P3), I want to enter and read times in WIB, so that the time I quote from the app or from a tester matches the log I find.

#### Acceptance Criteria

1. THE Finder SHALL interpret every time a Searcher enters as WIB (UTC+07:00).
2. WHEN a search is executed, THE Finder SHALL convert the Time_Window from WIB to UTC for Source queries.
3. THE Finder SHALL display times in the Simplified_View and on Filter_Chips in WIB with the label "WIB".
4. THE Finder SHALL keep the original timestamps unchanged in the Raw_View and True_Raw.
5. FOR ALL times t entered in WIB, converting t to UTC and back to WIB SHALL return t (round-trip property).

**Example (Given/When/Then):** Given a QA Engineer reports "penutupan kartu lagi 17:23:26" (RQ-10), When the QA Engineer searches with 17:23:26, Then the Loki query uses a window around 10:23:26 UTC and the Simplified_View shows 17:23:26 WIB, while the Raw_View keeps both original timestamps (UTC `Z` and `+0700`).

**Traceability:** Real requests: RQ-01, RQ-05, RQ-10, RQ-11, RQ-12, RQ-20, RQ-24. Root cause: RC-2, RC-4. Constraints: C1. Decisions: M-05 (A-027); log screenshot finding B4 [Ideation p.17].

### Requirement 9: Confirmation of vague times

**User Story:** As a Searcher (P1, P2, P3), I want the Finder to ask me to confirm the window when my time is vague, so that I do not get an empty or misleading result.

#### Acceptance Criteria

1. WHEN a Searcher gives a vague time (for example "yesterday afternoon", "tadi", "last hit tadi"), THE Finder SHALL propose a concrete Time_Window in WIB and ask the Searcher to confirm or edit it before running the search.
2. WHEN a Searcher gives a time without a date, THE Finder SHALL assume today's date in WIB and show the date on the Time_Window chip.
3. IF the Searcher does not confirm the proposed Time_Window, THEN THE Finder SHALL run no query.

**Example (Given/When/Then):** Given a QA Engineer types "saldo muncul titik di UAT, norek `<ACCOUNT_NO>`, last hit abt jam 11:22 - 11:23 tadi" (RQ-20), When the search is prepared, Then the Finder shows a Time_Window chip "today 11:22–11:23 WIB" with the approximate-time default applied and waits for confirmation.

**Traceability:** Real requests: RQ-20, RQ-07 ("around 15.11"). Root cause: RC-2. Constraints: C5. Decisions: M-06 (A-047, option 1 behavior "vague times always ask the user to confirm"), M-05.

### Requirement 10: Natural-language search producing structured filters

**User Story:** As a Searcher (P1, P2, P3), I want to type my request as a sentence, like I would in chat, so that the Finder fills in the filters for me.

#### Acceptance Criteria

1. WHEN a Searcher submits a sentence, THE NL_Translator SHALL return a Filter_Set only (Time_Window, Environment, identifiers, service/feature, error code); THE Finder SHALL accept no query text from the NL_Translator.
2. WHEN the NL_Translator returns a Filter_Set, THE Finder SHALL show every filter as an editable Filter_Chip and run the search only after the Searcher confirms.
3. THE Finder SHALL send the NL_Translator only the sentence, the active Project's Business_Term_Dictionary and identifier types; THE Finder SHALL send no log content to the NL_Translator.
4. THE NL_Translator SHALL run on infrastructure inside the Soluix environment.
5. IF the NL_Translator returns a filter that is not defined for the active Project (unknown field, unknown term or invalid value), THEN THE Finder SHALL drop that filter, show which part of the sentence was not understood, and keep the remaining Filter_Chips.
6. THE Finder SHALL record the search mode (manual filters or natural language) for every search (Requirement 26).
7. FOR ALL sentences in the NL test set built from RQ-01..RQ-24 and brief §2, THE NL_Translator SHALL produce Filter_Sets that pass Filter_Set validation (well-formed property).

**Example (Given/When/Then):** Given BCAS has "ganti rek sumber" in its dictionary, When a QA Engineer types "`<CIF>` di jam 15:00:33 ini ya, minta tolong ambilin log ganti rek sumber di RC" (RQ-01), Then the Finder shows chips Time 15:00:33 WIB, Environment RC, CIF `<CIF>`, Feature "ganti rek sumber", and runs nothing until the QA Engineer confirms.

**Traceability:** Real requests: RQ-01, RQ-05, RQ-07, RQ-10, RQ-11, RQ-12, RQ-20, RQ-24; brief §2. Root causes: RC-1, RC-2. Constraints: C1, C2, C5, C7. Decisions: T-001, L-01 (A-029), L-03 (A-048), L-02 (A-030), H-01 (A-018).

### Requirement 11: Business-term dictionary per Project

**User Story:** As a Project Admin for business configuration (P5a, a BA), I want to maintain a dictionary of business terms per Project, so that "ganti rek sumber" or "buka blokir" finds the right service or endpoint.

#### Acceptance Criteria

1. THE Finder SHALL let P5a create, edit and retire dictionary entries per Project, each mapping a term (and its synonyms, Indonesian or English) to one or more services, modules, endpoints or message patterns.
2. WHEN a dictionary entry is created or changed, THE Finder SHALL mark the entry "unconfirmed" until a Developer confirms the mapping (M-03).
3. WHEN a Searcher uses an unconfirmed term, THE Finder SHALL show the chip with the label "unconfirmed mapping".
4. WHEN a Searcher uses a term that is not in the dictionary, THE Finder SHALL show the term as free text in a message-text chip and offer to report the missing term to P5a.
5. THE Finder SHALL keep each entry's change history with author and date.

**Example (Given/When/Then):** Given BCAS has the confirmed entry "buka blokir" mapped to its unblock service, When a QA Engineer types "tim UAT udah reject buka blokir di jam 10:28:32 untuk hashcard `<SHA256_HASH>`" (RQ-11), Then the Feature chip reads "buka blokir" and the query is limited to the mapped service.

**Traceability:** Real requests: RQ-01, RQ-05, RQ-07, RQ-09, RQ-10, RQ-11, RQ-12, RQ-13, RQ-14, RQ-17, RQ-18, RQ-20, RQ-21, RQ-24. Root cause: RC-2. Constraints: C3. Decisions: M-03 (A-025), M-04 (A-026); reference finding C5.

### Requirement 12: Identifier-to-field mapping per Project

**User Story:** As a Project Admin for business configuration (P5a), I want to map each identifier type to the log fields or DB columns where it appears, so that a CIF, norek or hashcard search looks in the right place.

#### Acceptance Criteria

1. THE Finder SHALL let P5a define per Project each identifier type with: name, synonyms (for example "norek" for account number), allowed value pattern, and the log field(s) and DB column(s) where it appears.
2. THE Finder SHALL support at least these identifier types for the pilot Project: CIF, account number (norek), hashcard, program ID, license ID, request ID, endpoint path (M-01).
3. WHEN a Searcher enters an identifier, THE Finder SHALL search only the fields and columns mapped for that type in the active Project.
4. IF an identifier type has no mapping for the selected Source, THEN THE Finder SHALL tell the Searcher which Sources can be searched for that type.

**Example (Given/When/Then):** Given BCAS maps "norek" to the account-number field, When a BA types "`<ACCOUNT_NO>` minta log lagi untuk norek ini" (RQ-04), Then the Finder shows an Account number chip with `<ACCOUNT_NO>` and asks for a Time_Window (Requirement 13).

**Traceability:** Real requests: RQ-04, RQ-05, RQ-07, RQ-11, RQ-12, RQ-17, RQ-20, RQ-22, RQ-23. Root cause: RC-2. Constraints: C3. Decisions: M-01 (A-024), M-02 (A-071), D-01 (A-066).

### Requirement 13: Searches without identifier or time

**User Story:** As a Searcher (P1, P2, P3), I want the Finder to tell me what is missing and let me reopen an earlier search, so that "minta log lagi" requests still lead to a log.

#### Acceptance Criteria

1. IF a search has neither an identifier nor a Time_Window, THEN THE Finder SHALL run no query and show guidance naming the missing filters (for example "add a CIF, an account number or a time").
2. WHEN guidance is shown, THE Finder SHALL offer the Searcher's own Search_History in the active Project and let the Searcher reopen a previous search with its Filter_Chips.
3. WHEN a Searcher reopens a previous search, THE Finder SHALL run the search again only after the Searcher confirms the Filter_Chips, including a new Time_Window if needed.
4. THE Finder SHALL keep each user's Search_History visible to that user only.

**Example (Given/When/Then):** Given a FE developer searched BCAS UAT earlier today, When the FE developer types "minta log lagii" (RQ-13), Then the Finder asks for an identifier or a time and lists the FE developer's earlier searches to reopen.

**Traceability:** Real requests: RQ-03, RQ-04, RQ-06, RQ-08, RQ-09, RQ-13, RQ-14, RQ-15, RQ-16, RQ-18, RQ-19, RQ-21, RQ-22 (12 of 24 carry no identifier and no time; RQ-04 carries no time). Root cause: RC-1. Constraints: C5, A4. Decisions: M-10 (A-037).

### Requirement 14: Clear result or clear "nothing found"

**User Story:** As a Searcher (P1, P2, P3), I want either a clear result or a clear "nothing found", so that I know whether to change my search or escalate.

#### Acceptance Criteria

1. WHEN a search returns lines, THE Finder SHALL show the number of lines, the Time_Window in WIB, the Environment and the Sources searched.
2. WHEN a search returns no lines, THE Finder SHALL state "nothing found" with the Filter_Set, Time_Window and Sources searched, and offer to widen the Time_Window (within Max_Range) or to escalate (Requirement 24).
3. IF a Source fails or times out, THEN THE Finder SHALL state which Source failed and show results from the other Sources, labelled as partial.
4. THE Finder SHALL distinguish "nothing found" from "Source failed" in the message text.

**Example (Given/When/Then):** Given no log line exists in the window, When a QA Engineer searches "H2H error lagi jam 9.58, connection timeout" (RQ-24), Then the Finder states "nothing found in 09:5x–10:0x WIB, UAT, Loki" and offers "widen window" and "Ask an engineer".

**Traceability:** Real requests: RQ-02, RQ-23 ("pernah ke hit ngga?"), RQ-24. Root cause: RC-1. Constraints: C4. Decisions: O-03 (A-032); "Must" rule [Ideation p.14].

### Requirement 15: Simplified view by deterministic parsing

**User Story:** As a Searcher (P1, P2, P3), I want a readable simplified view of each log line, so that I can understand it and forward it without asking an engineer.

#### Acceptance Criteria

1. WHEN a log line is shown, THE Parser SHALL produce the Simplified_View deterministically: strip the stdout/CRI prefix, strip ANSI color codes, unwrap the JSON `log` field and nested escaped JSON, and select the configured fields.
2. THE Parser SHALL produce the same Simplified_View for the same raw line and the same Project configuration (determinism property).
3. THE Finder SHALL produce the Simplified_View without an LLM.
4. IF the Parser cannot parse a line, THEN THE Finder SHALL show the line in the Raw_View with the label "could not be simplified".
5. THE Parser SHALL keep message text, including Indonesian text, unchanged apart from the removals in criterion 1.
6. FOR ALL raw lines in the parser fixture set, parsing then printing the parsed structure then parsing again SHALL produce an equivalent structure (round-trip property).

**Example (Given/When/Then):** Given a raw Loki line from the BCAS UAT document service with a CRI prefix, ANSI-colored level and escaped JSON payload [Ideation p.17], When a QA Engineer opens the result for "penutupan kartu 17:23:26 CIF `<CIF>`" (RQ-10), Then the Simplified_View shows time (WIB), level, Environment, `project_name`, `project_module`, `module_version`, `correlation_id` and the parsed, masked `log_data`, with no ANSI codes or escape characters.

**Traceability:** Real requests: RQ-05, RQ-10, RQ-11, RQ-12, RQ-20 (logs read and forwarded). Root cause: RC-4. Constraints: C1. Decisions: L-04 (A-049), C-03 (A-061: screenshot is the source now; masked text sample required before the parser is built); findings B4, B8, B9 [Ideation p.17].

### Requirement 16: Configurable Simplified_View fields

**User Story:** As a Project Admin for business configuration (P5a), I want to choose the fields shown in the Simplified_View per Project, so that the view shows what that Project's team reads and forwards.

#### Acceptance Criteria

1. THE Finder SHALL use, for a Project without its own field list, the default field list: time (WIB), Environment, `project_name`, `project_module`, `module_version`, `correlation_id`, and parsed `log_data` (masked).
2. WHEN P5a changes the field list of a Project, THE Finder SHALL apply the new list to every Simplified_View of that Project from the next search on.
3. THE Finder SHALL apply Masking_Rules to every configured field.

**Example (Given/When/Then):** Given P5a adds the `X-Request-Id` header to the BCAS field list, When a BA opens the "liveness gagal / get token" result (RQ-21), Then the Simplified_View shows `X-Request-Id` next to the default fields.

**Traceability:** Real requests: RQ-21, RQ-22. Root cause: RC-4. Constraints: C1, C3. Decisions: O-01 (A-031), M-04.

### Requirement 17: Raw view one click away

**User Story:** As a Searcher (P1, P2, P3), I want the raw log one click away, pretty-printed and complete, so that I can paste it into a ticket or forward it to the client.

#### Acceptance Criteria

1. WHEN a Searcher selects "Raw" on a result, THE Finder SHALL show the Raw_View of that line within the same screen.
2. THE Finder SHALL pretty-print the Raw_View and keep every field of the original line, including fields not in the Simplified_View.
3. THE Finder SHALL apply Masking_Rules to the Raw_View by default (Requirement 19).
4. THE Finder SHALL keep the original timestamps in the Raw_View (Requirement 8).
5. FOR ALL lines, the set of field paths in the Raw_View SHALL equal the set of field paths in the original line (no field dropped).
6. THE Finder SHALL record each Raw_View open in the Audit_Log.

**Example (Given/When/Then):** Given a BA found the "liveness gagal" line (RQ-21), When the BA selects "Raw", Then the Finder shows the complete pretty-printed line with tokens hidden and both original timestamps.

**Traceability:** Real requests: RQ-08, RQ-18, RQ-21, RQ-22 (logs forwarded to the client). Root cause: RC-4. Constraints: C1, C5. Decisions: SEC-03 (A-052), M-05; raw side "pretty-printed without dropping fields" [Ideation p.16]; UI mockup SIMPLIFIED/RAW toggle [Presenter p.1].

### Requirement 18: Copy simplified and copy raw

**User Story:** As a Searcher (P1, P2, P3), I want to copy the simplified or the raw output, so that I can paste it into a ticket, chat or a message to the client.

#### Acceptance Criteria

1. WHEN a Searcher selects "Copy simplified", THE Finder SHALL copy the Simplified_View text, as shown on screen and masked, to the clipboard.
2. WHEN a Searcher selects "Copy raw", THE Finder SHALL copy the Raw_View text with the masking state shown on screen.
3. WHERE the Searcher holds the Unmask_Right for the Project and Environment and has chosen True_Raw, THE Finder SHALL copy True_Raw.
4. THE Finder SHALL record every copy action in the Audit_Log with the copy type (simplified, raw masked, raw unmasked, True_Raw).
5. THE Finder SHALL show, next to the copy buttons, whether the copy is masked or unmasked.

**Example (Given/When/Then):** Given a requester found the "export excel" log (RQ-18: "tar aku kirim ke bcas"), When the requester selects "Copy raw" without the Unmask_Right, Then the clipboard holds the masked Raw_View and the Audit_Log records "raw masked" copy.

**Traceability:** Real requests: RQ-08, RQ-12 ("buat dokumentasi"), RQ-18, RQ-21, RQ-22. Root cause: RC-1. Constraints: C1, C5. Decisions: SEC-03 (A-052), SEC-05 (A-054), SEC-04 (A-053); copy buttons [Presenter p.1], [Ideation p.16].

### Requirement 19: Masking on by default

**User Story:** As a Project Admin access manager (P5b), I want sensitive values masked in every view and copy by default, using patterns set per Project, so that banking data is protected even when logs are forwarded.

#### Acceptance Criteria

1. THE Finder SHALL apply the Project's Masking_Rules to the Simplified_View, Raw_View, copies, Escalation_Link views and Audit_Log entries by default.
2. THE Finder SHALL ship default Masking_Rules that hide tokens and document content (including base64 document bodies) fully and mask CIF, account numbers and hashcards partially (exact partial pattern set in Stage 4).
3. WHERE the Masking_Rules allow it, THE Finder SHALL show the identifier the Searcher searched for in full to that Searcher only.
4. THE Finder SHALL let P5b add, edit and disable masking patterns per Project, and record each change in the Audit_Log.
5. FOR ALL values matched by an active masking pattern, THE Finder SHALL emit no unmasked occurrence of the value in any masked view or masked copy (masking invariant).
6. THE Finder SHALL treat copies that leave Soluix as out of scope: THE Finder SHALL record each copy (Requirement 18) and SHALL attempt no detection of where the copy is sent.

**Example (Given/When/Then):** Given the default BCAS Masking_Rules, When a QA Engineer opens the result for "hashcard `<SHA256_HASH>` di jam 14:12:10 ... ganti sof" (RQ-12), Then the searched hashcard is shown in full to that QA Engineer, other hashcards and CIFs are partially masked, and tokens and document bodies are hidden.

**Traceability:** Real requests: RQ-08, RQ-12, RQ-18, RQ-21, RQ-22 (forwarding). Root cause: RC-4. Constraints: C1, C5. Decisions: SEC-02 (A-051), SEC-05 (A-054), C-04 (A-062: data synthetic, masking still on); finding B7 [Ideation p.17].

### Requirement 20: Unmask and True_Raw right

**User Story:** As a Project Admin access manager (P5b), I want to grant the right to unmask and to copy True_Raw per user, Project and Environment, so that people who need full values can get them, explicitly and on record.

#### Acceptance Criteria

1. THE Finder SHALL give no user the Unmask_Right by default.
2. WHEN P5b grants the Unmask_Right, THE Finder SHALL scope the grant to one user, one Project and one Environment, and allow any user role (including BA, QA, FE/Mobile developer) to receive the grant.
3. WHILE a Searcher holds the Unmask_Right for the active Project and Environment, THE Finder SHALL offer a masking toggle and a True_Raw option; WHILE the Searcher does not hold the right, THE Finder SHALL hide both.
4. WHEN a Searcher switches masking off or opens or copies True_Raw, THE Finder SHALL record the action in the Audit_Log with user, time, Project, Environment, result reference and action.
5. WHEN a Searcher starts a new search or a new Session, THE Finder SHALL switch masking back on.
6. THE Finder SHALL record every Unmask_Right grant and revocation in the Audit_Log with the acting P5b.

**Example (Given/When/Then):** Given a BA holds the Unmask_Right for BCAS UAT only, When the BA opens the "liveness gagal / get token" result (RQ-21) to check the token with the client, Then the BA can switch masking off for that result in UAT, the action is audited, and the toggle is absent in BCAS RC.

**Traceability:** Real requests: RQ-21, RQ-22 ("yg token itu perlu cek ke BCA kalo salah"). Root cause: RC-3. Constraints: C1, C5, C8 (T-002 covers direct Grafana / DB / k9s access only, per C-05). Decisions: C-01 (A-059), C-05 (A-079), SEC-02, SEC-03, SEC-04, M-04.

### Requirement 21: Loki Source connector

**User Story:** As a Searcher (P1, P2, P3), I want the Finder to read the Project's Loki logs through templates, so that I get the same logs engineers see in Grafana without needing Grafana access.

#### Acceptance Criteria

1. THE Finder SHALL query Loki server-side through Query_Templates over the bounded Time_Window, using the HTTP API of the Project's Loki endpoint in the Source_Binding.
2. THE Finder SHALL map each Environment to Loki label selectors and, where configured, a payload filter (for example `project_name`), per Project and Environment in the Source_Binding.
3. THE Finder SHALL support per-binding connection settings: base URL, authentication type (none, basic, bearer), optional headers (for example `X-Scope-OrgID`), held server-side.
4. IF Loki returns an error or exceeds the connector timeout, THEN THE Finder SHALL report "Source failed" for Loki (Requirement 14) and record the failure.
5. THE Finder SHALL use read-only Loki API operations only.

**Example (Given/When/Then):** Given the BCAS UAT binding maps UAT to its label selectors and `project_name` filter, When a QA Engineer searches "buka blokir 10:28:32 hashcard `<SHA256_HASH>`" (RQ-11), Then the Finder queries the BCAS Loki with those selectors and the hashcard line filter for the UTC window.

**Traceability:** Real requests: RQ-01, RQ-05, RQ-07, RQ-10, RQ-11, RQ-12, RQ-24. Root cause: RC-3. Constraints: C2, C4, C5. Decisions: G-01 (A-042), G-04 (A-063, A-081), G-05 (A-064), M-08 (A-073), G-06; finding B3 [Ideation p.17].

### Requirement 22: Kubernetes pod logs through Loki

**User Story:** As a Searcher (P1, P2, P3), I want Kubernetes pod logs to be searchable, so that I do not need k9s.

#### Acceptance Criteria

1. WHERE DevOps confirms that Loki holds the pod logs of an Environment, THE Finder SHALL serve the Kubernetes source of that Environment through the Loki connector (Requirement 21).
2. THE Finder SHALL label results from pod logs with the pod and container labels available in Loki (for example `kubernetes_container_name`).
3. WHEN a search covers a time before the Loki retention period of the Project, THE Finder SHALL state that logs older than the retention period are unavailable.
4. WHERE DevOps has not confirmed Loki coverage for an Environment, THE Finder SHALL show the Kubernetes source of that Environment as "not yet available".

**Example (Given/When/Then):** Given DevOps confirmed BCAS UAT pod logs are in Loki, When a QA Engineer searches "upgrade kartu ... program not found, program ID `<PROGRAM_ID>`" (RQ-17), Then results from the card service pods are shown with their container label, including pods that no longer exist, within the retention period.

**Traceability:** Real requests: RQ-17, RQ-24. Root cause: RC-3. Constraints: C4, C8. Decisions: K-01 (A-069), K-02 (A-070, retention TBD), AS-05.

### Requirement 23: Database Source with limits

**User Story:** As a Searcher (P1, P2, P3), I want to look up records in the Project database through the Finder, so that I can check data such as account information without database access.

#### Acceptance Criteria

1. THE Finder SHALL query the database only through read-only, parameterized Query_Templates for the tables listed for the Project (D-01).
2. THE Finder SHALL enforce, for every database query, a query timeout, a row cap, a maximum time range and a concurrency limit, read from the Source_Binding.
3. WHILE the database limits of a Project and Environment have no recorded approval by DevOps and the database owner (BCAS: Hadi Septian), THE Finder SHALL keep the database Source disabled for that Project and Environment.
4. IF a database query reaches the timeout, THEN THE Finder SHALL cancel the query at the database and report "Source failed" for the database.
5. WHEN the concurrency limit is reached, THE Finder SHALL queue or refuse further database searches and tell the Searcher to retry later.

**Example (Given/When/Then):** Given the BCAS UAT database Source is approved with limits, When a QA Engineer checks "saldo muncul titik dan informasi rekening kosong, norek `<ACCOUNT_NO>`" (RQ-20), Then the Finder runs the account template with the account number as a parameter, within the timeout and row cap.

**Traceability:** Real requests: RQ-20, RQ-04. Root cause: RC-3. Constraints: C2, C4, C5, A4 (no read-replica, primary protected). Decisions: D-01 (A-066), D-02 (A-067), D-03 (A-068), G-06, G-08 (A-065).

### Requirement 24: Cross-service view when correlation_id is empty

**User Story:** As a Searcher (P1, P2, P3), I want to see the lines of one request across services even when `correlation_id` is empty, so that I see the whole flow, not one part of it.

#### Acceptance Criteria

1. WHEN matching lines come from several services and `correlation_id` is empty, THE Finder SHALL group the lines by service within the Time_Window and label the groups "not linked by correlation_id".
2. WHEN matching lines share a non-empty `correlation_id`, THE Finder SHALL show them as one linked flow ordered by time.
3. WHERE Developers have confirmed that `X-Request-Id` or `traceparent` is propagated between the services of a Project, THE Finder SHALL link lines that share that value and label the link "linked by X-Request-Id/traceparent".
4. WHEN lines share a searched business identifier (for example CIF) but no linking key, THE Finder SHALL show them in the same group labelled "linked by `<identifier type>`".

**Example (Given/When/Then):** Given the CWLD flow spans several services with empty `correlation_id` (RQ-07, "mungkin pas external_opening_account"), When a QA Engineer searches the three CIFs around 15:11 in RC, Then the Finder shows the lines grouped by service and per CIF, labelled "not linked by correlation_id".

**Traceability:** Real requests: RQ-07, RQ-21 (liveness → get token). Root cause: RC-4. Constraints: C4. Decisions: M-07 (A-072), M-09 (A-074); findings B5, B6 [Ideation p.17].

### Requirement 25: Ask an engineer (escalation)

**User Story:** As a Searcher (P1, P2, P3), I want to escalate a lookup I cannot resolve, so that an engineer (P4) starts from my search instead of from scratch.

#### Acceptance Criteria

1. WHEN a Searcher selects "Ask an engineer", THE Finder SHALL record the Escalation (user, time, Project, Environment, Filter_Set, result count) and produce an Escalation_Link to paste into the team chat.
2. WHEN someone opens an Escalation_Link, THE Finder SHALL require sign-in, re-check the viewer's grant for the Project and Environment, and show the escalated Filter_Set and results under the viewer's own masking rights.
3. IF the viewer of an Escalation_Link has no grant for the Project and Environment, THEN THE Finder SHALL show access denied with no content (Requirement 3).
4. THE Finder SHALL offer the Escalation_Link as the only shareable link in the MVP.
5. THE Finder SHALL offer "Ask an engineer" on the "nothing found" message and on every result screen.

**Example (Given/When/Then):** Given a Mobile developer cannot tell whether `/api/registration/ibridge/v1/verihubs/encryption/generate-key` was ever hit (RQ-23), When the Mobile developer selects "Ask an engineer", Then an Escalation is recorded and the Escalation_Link opens the same search for a Developer who is a BCAS member.

**Traceability:** Real requests: RQ-02, RQ-03, RQ-15, RQ-23, RQ-24. Root cause: RC-5. Constraints: C4, C5. Decisions: O-03 (A-032), O-02 (A-075), C-07 (A-080), MET-03 (A-035).

### Requirement 26: Audit log

**User Story:** As a Project Manager (P6) or System Architect, I want an audit log of searches and sensitive actions, so that access to banking data can be reviewed and the pilot has evidence.

#### Acceptance Criteria

1. THE Finder SHALL record in the Audit_Log, for every search: user, time, Project, Environment, Filter_Set or sentence (masked), search mode and result count.
2. THE Finder SHALL record in the Audit_Log every Raw_View open, copy action, unmask action, True_Raw view or copy, Escalation, Escalation_Link open, access-denied attempt, sign-in failure, and configuration or grant change.
3. THE Finder SHALL let only the PM and System Architect roles read the Audit_Log.
4. THE Finder SHALL keep the Audit_Log append-only for all roles, including Project Admins.
5. THE Finder SHALL keep Audit_Log entries for the retention period set by compliance (value TBD, open item OI-05) as a configuration parameter.

**Example (Given/When/Then):** Given a QA Engineer searched "penutupan kartu 17:23:26 CIF `<CIF>`" (RQ-10) and copied the raw log, When the PM opens the Audit_Log, Then the PM sees the search with the CIF masked, the result count, and the "raw masked" copy entry.

**Traceability:** Real requests: RQ-10 (example); all requests. Root cause: RC-5. Constraints: C1, C5. Decisions: SEC-04 (A-053), G-07 (A-046), C-05 (A-079).

### Requirement 27: Pilot instrumentation and metrics

**User Story:** As a Project Manager (P6), I want the Finder to measure self-service use and time to a readable log, so that the pilot decision (roll out, adjust, revisit) uses data.

#### Acceptance Criteria

1. THE Finder SHALL record, per Session, the time from the first search to the first Simplified_View open (time to readable log).
2. THE Finder SHALL record the search mode (manual filters or natural language) of every search.
3. THE Finder SHALL classify a lookup as self-served WHEN the user opened a result and used no "Ask an engineer" in the same Session, and as escalated WHEN "Ask an engineer" was used.
4. THE Finder SHALL show the PM, per Project and per week: number of searches, self-served lookups, escalated lookups, share of natural-language searches, and the median and 90th percentile of time to readable log.
5. THE Finder SHALL let the PM enter the weekly ping counts from the chat-export baseline and the pilot period (counted outside the Finder by keyword tagging and BA review), and show the reduction against the baseline.
6. WHEN the PM views the pilot result, THE Finder SHALL classify it with the MET-04 thresholds: "met" at ≥ 65% ping reduction and time to readable log under 2 minutes; "partly met" at 35%–64%; "no change" at 0%–34%.
7. IF a result falls outside the MET-04 thresholds (≥ 65% reduction but time not under 2 minutes, or a negative reduction), THEN THE Finder SHALL show the result as "unclassified: PM decision needed" (rule open, OI-06).

**Example (Given/When/Then):** Given a QA Engineer types "minta log penutupan kartu lagi 17:23:26 CIF `<CIF>`" (RQ-10) in natural language and opens the Simplified_View 70 seconds later without escalating, When the PM opens the weekly metrics, Then the lookup counts as self-served, natural language, 70 seconds.

**Traceability:** Real requests: all RQ-01..RQ-24 (baseline of pings). Root cause: RC-5. Constraints: C3. Decisions: MET-01 (A-033), MET-02 (A-034), MET-03 (A-035), MET-04 (A-076), audit notes N-008, N-011; pilot hypothesis §1.1.

### Requirement 28: Performance and capacity (non-functional)

**User Story:** As a Searcher (P1, P2, P3), I want to reach a readable log fast, so that self-service is quicker than pinging an engineer.

#### Acceptance Criteria

1. WHEN a pilot Searcher looks up a log that exists in an available Source, THE Finder SHALL let the Searcher reach a readable log (first Simplified_View open) in under 2 minutes end to end, measured by Requirement 27 criterion 1 (pilot target, §1.1).
2. THE Finder SHALL support the pilot load of fewer than 10 users; the concurrent-search limit per Source is a parameter (value proposed in Stage 4).
3. THE Finder SHALL meet the per-search response-time target that Stage 4 sets after the Loki spike (N-02); until then, the end-to-end target in criterion 1 is the only latency requirement.

**Traceability:** Real requests: all (each ping costs 5–10 engineer minutes, §1.1). Root cause: RC-1. Constraints: A4, C5. Decisions: N-01 (A-021), N-02 (A-022), MET-02; time target [Presenter p.7].

### Requirement 29: Availability and fallback (non-functional)

**User Story:** As a Searcher (P1, P2, P3), I want to know when the Finder is unavailable, so that I can fall back to asking an engineer.

#### Acceptance Criteria

1. THE Finder SHALL be available on working days during working hours on a best-effort basis during the pilot.
2. IF the Finder cannot reach any Source of the active Project, THEN THE Finder SHALL tell the Searcher to ask an engineer through the usual team chat.
3. THE Finder SHALL record each Source outage it detects (start time, end time, Source) for the PM.

**Traceability:** Real requests: all (fallback is today's chat ping). Root cause: RC-5. Constraints: C4. Decisions: N-03 (A-023).

### Requirement 30: Hosting, environments and data location (non-functional)

**User Story:** As a System Architect, I want the Finder hosted inside Soluix and limited to UAT and RC, so that banking data stays inside the internal network.

#### Acceptance Criteria

1. THE Finder SHALL run on an internal Soluix server or VM on the same network as UAT and RC.
2. THE Finder SHALL connect only to Sources of the UAT and RC Environments in the MVP; production Sources are excluded.
3. THE Finder SHALL onboard UAT of the pilot Project first and add RC within the pilot once RC's Loki data is confirmed.
4. THE Finder SHALL keep all log content, the NL_Translator and the Audit_Log inside the Soluix environment.
5. THE Finder SHALL read its configuration and secrets from outside the code (environment or secret store), so that a later host can be used without code changes.

**Traceability:** Real requests: RQ-01, RQ-07 (RC), RQ-11, RQ-20, RQ-21 (UAT). Root cause: RC-3. Constraints: C1, C6, C7, C9. Decisions: H-01 (A-018), H-02 (A-019), A-010, S-03 (A-017), L-01 (A-029).

### Requirement 31: Coexistence with existing tools (non-functional)

**User Story:** As a Developer/Engineer (P4), I want the Finder to leave Grafana, Loki, the database and k9s as they are, so that my tools keep working and the Finder adds no new risk to them.

#### Acceptance Criteria

1. THE Finder SHALL connect to each Source only through read-only access granted to its Service_Identity, with no change to the Source's configuration other than that access grant (and any network route DevOps approves).
2. THE Finder SHALL leave engineers' use of Grafana, the database and k9s unchanged.
3. THE Finder SHALL run with Grafana MCP server and Grafana Assistant absent; WHERE either is available and approved in Stage 4, THE Finder SHALL use it for discovery only, with query execution limited to Query_Templates.
4. THE Finder SHALL offer no Grafana deeplinks in the MVP.
5. THE Finder SHALL keep Source connectors behind one connector interface, so that new input sources (and image input) can be added in the next phase without changing existing connectors.

**Traceability:** Real requests: none directly. Root cause: RC-3. Constraints: C4, C5, C8, C9. Decisions: G-01 (A-042), G-02 (A-043), G-03 (A-044), O-02 (A-075), T-002, T-003.

## Constraints for Stage 4 (not functional requirements)

| ID | Constraint | Source | Status |
|---|---|---|---|
| SC-1 | Backend Go + Echo; frontend TypeScript, React, Vite, TanStack Router, TanStack Query, TanStack Virtual, Tailwind, shadcn/ui, lucide-react; PactFlow contract testing in backend, frontend and CI/CD | H-04 (A-040) | **To confirm with Developers**, including any other libraries |
| SC-2 | Single repository with separate top-level directories per part (for example backend, frontend), so it can be split later | R-01 (A-041) | Decided |
| SC-3 | Approach per Source decided in its own session (input/output-agnostic engine) | G-01 (A-042) | Decided |
| SC-4 | Loki access path: working assumption "direct, without authentication, network-restricted" (A2 unverified); the Loki spike verifies it with the §6 checklist; Stage 4 decides whether an authenticating proxy is needed (RK-002) | G-04, C-08 (A-081) | Open |
| SC-5 | Parser fixtures require a masked text export of raw lines and the full label list before the parser is built | C-03 (A-061) | Open; owner DevOps or a Developer |
| SC-6 | Internal LLM host and model for the NL_Translator | L-01 (A-029), N-010 | Open; Stage 4 |

## Out of Scope (MVP)

| Item | Reason | Source |
|---|---|---|
| Production environment | MVP is UAT and RC only | A-010, C6 |
| Image input (reading screenshots) and more input sources | Next phase; the design must not block them (Requirement 31.5) | §1.4, C9 |
| General result sharing (share links other than the Escalation_Link) | Not in the MVP | O-02 (A-075), C-07 (A-080) |
| Grafana deeplinks | Not in the MVP | O-02 (A-075) |
| Detecting copies that leave Soluix | No reliable way to detect; copies are audited only | SEC-05 (A-054) |
| Example API / endpoint responses (as documentation) | The Finder reads logs; a response is shown only if it appears in a log line found by a normal search | O-04 (A-039); RQ-23 |
| LLM-generated LogQL or SQL | LLM returns structured filters only | L-03 (A-048), C2 |
| LLM-produced simplified views or summaries | Deterministic parsing only | L-04 (A-049) |
| Direct Grafana / database / k9s access for BA/QA | Rejected by the team | T-002, C8 |
| Direct Kubernetes pod-log reading | Pod logs served through Loki | K-01 (A-069) |
| SSO / external identity provider | Local accounts in the MVP | H-03 (A-020) |
| Onboarding Projects other than BCAS | After the pilot decision | S-01 (A-015) |

## Open Items Carried to Stage 4

| ID | Open item | Blocks | Owner | Source |
|---|---|---|---|---|
| OI-01 | Loki access unverified: "direct, without authentication" is a working assumption (A2); verify with the §6 checklist; RK-002 open | Loki spike and connector (Req 4, 21) | DevOps (Hadi Septian), System Architect | G-04, C-08 (A-081) |
| OI-02 | Loki retention period per Environment | Req 22.3 | DevOps | K-02 (A-070) |
| OI-03 | Default values: exact-time window, approximate-time window, Max_Range, Result_Cap, concurrency, idle timeout | Req 1.4, 7, 9, 28.2 | System Architect, BA | M-06 (A-047) |
| OI-04 | Per-search response-time target, after the Loki spike | Req 28.3 | PM, System Architect | N-02 (A-022) |
| OI-05 | Audit_Log retention period, set by compliance | Req 26.5 | PM, compliance | SEC-04, N-005 |
| OI-06 | MET-04 gaps: how to classify ≥ 65% reduction with time not under 2 minutes, and a negative reduction; baseline method for today's time to get a log | Req 27.6–27.7 | PM, QA | N-008, N-011 |
| OI-07 | Whether FE/Mobile developers use Grafana today | Persona P3 rights review | PM, BA | U-01, N-006 |
| OI-08 | Database table list for the pilot (tables Developers open for RQ-01..RQ-24) | Req 23.1 | Developers, DevOps | D-01 (A-066) |
| OI-09 | Masked raw-log text sample plus full label list | Parser build (Req 15) | DevOps or a Developer | C-03 (A-061) |
| OI-10 | Partial-masking pattern for CIF, account number, hashcard | Req 19.2 | System Architect, PM | SEC-02 (A-051) |
| OI-11 | DevOps confirmation that Loki holds UAT and RC pod logs (AS-05) | Req 22 | DevOps | K-01 (A-069) |
| OI-12 | Developers confirm `X-Request-Id` / `traceparent` propagation | Req 24.3 | Developers | M-09 (A-074) |
| OI-13 | Pilot window dates; L-02 "no limits" has no named confirmer | Pilot start | PM | N-010, N-007 |

## Stage 3 Self-Check

| # | Check (brief §0.2) | Result | Evidence |
|---|---|---|---|
| 1 | Every acceptance criterion is testable | **Pass, with parameters** | Each criterion names an observable output (chip shown, query refused, audit entry, label text, count). Criteria that depend on values (Req 1.4, 7, 9, 22.3, 26.5, 28.2–28.3) reference a named Project parameter whose default is set in Stage 4 (OI-03 to OI-05), so the test checks behavior against the configured value. Property criteria: 8.5 (timezone round trip), 10.7 (well-formed Filter_Sets), 15.2 (determinism), 15.6 (parser round trip), 17.5 (no field dropped), 19.5 (masking invariant) |
| 2 | Every rule traces to a source | **Pass** | Every requirement has a Traceability line citing real requests (RQ-xx, brief §2), root causes RC-1..RC-5, constraints C1–C9 and decision IDs. Requirements 1, 2 and 31 cite no single request and trace to root causes, constraints and decisions instead |
| 3 | Design respects stack, security and scope limits | **Pass (requirement level)** | C2 enforced (Req 6.4, 10.1); C5 (Req 3, 4, 19, 20); C6 production excluded (Req 30.2); stack and repository recorded as Stage 4 constraints only (SC-1, SC-2); out-of-scope list matches decisions |
| 4 | Reuses existing systems, roles and data | **Pass** | Loki, Fluent Bit labels, existing pod logs in Loki, the existing database and existing team roles are reused (Req 21–23, 31); no new log store |
| 5 | Evidence of the check is shown | **Pass** | This table; Traceability lines; Traceability Matrix below |
| 6 | Open questions flagged, not silently decided | **Pass** | OI-01..OI-13 listed with owners; G-04 kept as an unverified assumption (Req 4.6, SC-4); MET-04 gaps not decided (Req 27.7); stack marked "to confirm" (SC-1). AI proposals to confirm at approval (not asked in Stage 2): Session idle timeout (Req 1.4), salted password hashing (Req 1.5), session end on account removal (Req 2.2), masking switched back on per new search (Req 20.5), median and 90th-percentile reporting (Req 27.4) |

## Traceability Matrix: the 24 Sampled Requests

Numbering: image order in [Ideation p.4–11], three captures per page. Wording is shortened; identifiers are placeholders; people are named by role. "Id" = identifier value given; "Time" = time given.

| RQ | Page | Requester role (as labelled) | Request (short, placeholders) | Id | Time | Requirements |
|---|---|---|---|---|---|---|
| RQ-01 | p.4 | requester (role not shown) | "`<CIF>` di jam 15:00:33 ... minta tolong ambilin log ganti rek sumber di RC" | yes | yes | 3, 5, 6, 8, 10, 11, 21, 30 |
| RQ-02 | p.4 | requester (role not shown) | "url path yang mana yang kena 403 ya? minta lognya aja sekalian" | path asked | no | 6, 14, 25 |
| RQ-03 | p.4 | requester (role not shown) | "minta lognya, ini ada beberapa yg kena error" | no | no | 13, 25 |
| RQ-04 | p.5 | requester (role not shown) | "`<ACCOUNT_NO>` minta log lagi untuk norek ini" (reply to a photo) | yes | no | 12, 13, 23 |
| RQ-05 | p.5 | requester (role not shown) | "minta ambilin log perubahan ganti rek sumber ... di jam 16:04:44 cif `<CIF>`" | yes | yes | 6, 7, 8, 10, 11, 12, 15, 21 |
| RQ-06 | p.5 | requester, FE developer in cc | "minta log untuk trx ini" | no | no | 13 |
| RQ-07 | p.6 | requester (role not shown) | "cek log untuk pengajuan CWLD (mungkin pas external_opening_account) yang gagal di RC; around 15.11; CIF `<CIF1>`, `<CIF2>`, `<CIF3>`" | yes (3) | approx | 3, 5, 6, 7, 9, 10, 11, 12, 21, 24, 30 |
| RQ-08 | p.6 | requester, replying to FE developer | "minta lognya boleh kah ... biar aku kasi ke `<client contact>`" | no | no | 13, 17, 18, 19 |
| RQ-09 | p.6 | requester (role not shown) | "sama minta lognya waktu refund" | no | no | 11, 13 |
| RQ-10 | p.7 | requester (role not shown) | "minta log penutupan kartu lagi 17:23:26 CIF `<CIF>`" | yes | yes | 4, 6, 8, 10, 11, 15, 21, 26, 27 |
| RQ-11 | p.7 | requester (role not shown) | "minta log kaya tadi lagi, tim UAT udah reject buka blokir di jam 10:28:32 ... hashcard `<SHA256_HASH>`" | yes | yes | 3, 6, 8, 10, 11, 12, 15, 21, 30 |
| RQ-12 | p.7 | requester (role not shown) | "minta log untuk hashcard `<SHA256_HASH>` di jam 14:12:10. Buat dokumentasi, ... abis test ganti sof" | yes | yes | 6, 8, 10, 11, 12, 15, 18, 19, 21 |
| RQ-13 | p.8 | FE developer | "minta log lagii" (reply to a photo); "kalo yg closed amount gagal ga?" | no | no | 11, 13 |
| RQ-14 | p.8 | FE developer | "laporan kartu debit ... semua filter nampilin hal yang ga sesuai" → "minta log ... mgkn salah satu aja kalo yg mirip (kyk filter)" | no | no | 11, 13 |
| RQ-15 | p.8 | FE developer | "ini belum bisa ..., masih sama errornya" → "minta log nya lagi" | no | no | 13, 25 |
| RQ-16 | p.9 | FE developer | "gagall" (photo) → "minta lognya lagii" | no | no | 13 |
| RQ-17 | p.9 | FE developer | "yang upgrade kartu kena program not found, program idnya `<PROGRAM_ID>`" → "minta log nya" | yes | no | 6, 11, 12, 22 |
| RQ-18 | p.9 | requester (role not shown) | "export excel ... coba minta log nya, ... tar aku kirim ke bcas" | no | no | 11, 13, 17, 18, 19 |
| RQ-19 | p.10 | requester (role not shown) | "di lognya begitu" → "bolehh minta lognya kahh" | no | no | 13 |
| RQ-20 | p.10 | requester, FE developer in cc | "cek di log kenapa waktu buka saldo muncul titik dan buka informasi rekening jadi kosong di UAT; norek `<ACCOUNT_NO>`; last hit abt jam 11:22 - 11:23 tadi" | yes | approx | 3, 6, 7, 8, 9, 10, 11, 12, 15, 23, 30 |
| RQ-21 | p.10 | BA (quoted), requester | "liveness gagal ... UAT BCAS masih gagal, apa boleh minta log get tokennya? biar kulaporin ke IT BCAS"; "stopper untuk simplify auth" | no | no | 3, 11, 13, 16, 17, 18, 19, 20, 24, 30 |
| RQ-22 | p.11 | BA | "Atau cuma mau license_id nya" → "minta log nya ..., soalnya yg token itu perlu cek ke BCA kalo salah" (`<LICENSE_ID>` not given) | type only | no | 12, 13, 16, 17, 18, 19, 20 |
| RQ-23 | p.11 | Mobile developer | (a) "boleh minta contoh response dari endpoint ini kah? `/api/registration/ibridge/v1/verihubs/encryption/generate-key`"; (b) "bisa bantu check ini ada log pernah ke hit ngga?" | path | no | (a) **not answerable by design**: example endpoint response is out of MVP scope (O-04); shown only if visible in a found log line. (b) 12, 14, 25 |
| RQ-24 | p.11 | requester (role not shown) | "cek log kah H2H kena error lagi jam 9.58 kayanya sama ya kena connection timeout? mau minta lognyaa" | no | yes | 6, 7, 8, 10, 11, 14, 21, 22, 25 |

**Brief §2 representative requests:** all ten map to captures above (ganti rek sumber → RQ-05; penutupan kartu → RQ-10; CWLD → RQ-07; buka blokir → RQ-11; program not found → RQ-17; saldo → RQ-20; 403 → RQ-02; generate-key → RQ-23; H2H → RQ-24; refund / liveness / trx → RQ-09, RQ-21, RQ-06).

**Counts:** 31 user stories (one per requirement). 28 of 31 cite at least one sampled request (Requirements 1, 2 and 31 trace to root causes and constraints only). Every one of RQ-01..RQ-24 maps to at least one requirement; only RQ-23 (a) is not answerable by design.
