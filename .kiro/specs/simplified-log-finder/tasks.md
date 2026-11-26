# Implementation Plan: Simplified Log Finder

AI-DLC Inception, Stage 5 (Plan). Status: **Approved 2026-11-26 (audit A-136)** (generated 2026-11-26 from `_decisions-tasks.md`, all 13 decisions answered; audit A-121 to A-135).

This plan turns the approved `requirements.md` (31 requirements) and `design.md` v1.0 into units of work (U1 to U6) and 15 coarse tasks, ordered in waves. Each wave is one unit delivered by two workers in parallel, one in `/backend` and one in `/frontend` plus `/deploy`. No application code is written in Inception; this plan is executed in Construction after the Human Review Gate.

## Planning Decisions Applied

| Decision | Answer | Effect on this plan |
|---|---|---|
| TD-01 Units | 1 | U1 to U6 as proposed |
| TD-02 Pilot start | 1 | Pilot starts after U1, U2, U3 and U6; U4 and U5 join during the pilot. Wave order puts U6 before U4 and U5 |
| TD-03 Slice style | 1 | Vertical slices: each wave delivers one unit end to end, split across two workers by directory |
| TD-04 Test timing | 1 | Test-first for pure logic, properties P-a..P-f and Pact contracts; tests alongside code for UI and integration |
| TD-05 Granularity | 2 | Coarse: 15 tasks, each with sub-steps that the Bolt's plan step breaks down |
| TD-06 Parallel workers | 2 | Two workers in parallel: Group A/C/E/G/I (backend) and Group B/D/F/H/J (frontend and deploy) |
| TD-07 Owner per unit | 2 | Roles only in this plan; the PM assigns names after Inception |
| TD-08 Dates | 2 | Relative order only; no durations or dates; dates are set at the review gate |
| TD-09 Deployment | 1 | Deploy to the internal server at the end of each unit with a smoke test; U2 deploys before its review so the finder can reach BCAS UAT Loki |
| TD-10 Rollback | 1 | Previous image tag through Compose; migrations forward-only and backward compatible for one release; the audit table is never rolled back |
| TD-11 CI/CD | 2 | Fully automatic deploy on main (build, tests, Pact publish and can-i-deploy, image, deploy). See proposal D5-02 |
| TD-12 Definition of Done | 1 | See "Definition of Done" below |
| TD-13 Open items | 3 | Owners as proposed; due dates are decided later at task execution. See note under "Open Decisions" |

## Execution Plan

| Wave | Unit | Groups (run in parallel) | Depends on |
|---|---|---|---|
| 1 | U1 Foundation and access | Group A (backend), Group B (frontend and deploy) | H-04 stack confirmed |
| 2 | U2 Loki spike and manual filters | Group C (backend), Group D (frontend) | Wave 1; V-01 done; read-only Loki access approved |
| 3 | U3 Simplified and raw view | Group E (backend), Group F (frontend) | Wave 2; V-10 masked log sample |
| 4 | U6 Pilot instrumentation and escalation | Group I (backend), Group J (frontend) | Wave 2 (U1 and U2 outputs) |
| | **Pilot-start gate** (after Wave 4) | U1, U2, U3 and U6 done; open items below closed | Waves 1 to 4 |
| 5 | U4 Natural-language search and U5 Database and Kubernetes sources | Group G (backend), Group H (frontend) | Wave 3 (U4 needs U3), Wave 2 (U5), inference VM, V-06, V-08, D-03 |
| 6 | Verification and pilot readiness | Group K (both workers, separate directories) | Wave 5 |

> **How to run:** one worker per group within a wave. Wait for all groups in a wave to finish and for the unit review (Definition of Done) before starting the next wave. If a group finishes early, its worker waits; next-wave tasks are not started early.

**Rule for files.** Groups in the same wave never touch the same files. Backend groups own `/backend` including `backend/openapi/finder.yaml` and the router registration; frontend and deploy groups own `/frontend`, `/deploy`, `/fixtures` and the CI configuration. The frontend group develops against Pact consumer tests derived from the API Contract table in `design.md`; the backend group's provider verification catches any mismatch inside the wave. Each group adds new files in its own packages and avoids edits to shared files owned by the other group. Two specific edits cross directories and are assigned to one group: `design.md` amendments (Group C, step C.1) and fixtures in `/fixtures` (Group F creates them from V-10; Group E reads them).

## Units of Work

| Unit | Scope | Requirements | Roles | Acceptance criteria (testable) | Verification evidence |
|---|---|---|---|---|---|
| U1 Foundation and access | Repository layout, schema and migrations, config model, local accounts, sessions, CSRF, RBAC, audit with hash chain, admin screens, Compose deployment, CI/CD | 1, 2, 3, 5, 26, 30 | Developers (backend lead and frontend), DevOps | A user signs in with a local account and sees only granted Projects and Environments; disabling an account ends its session on the next request; every grant, revocation and sign-in failure is in the Audit_Log; the audit verification command passes on a clean chain and reports the first broken row after a test edit; `docker compose up` starts finder, PostgreSQL and the proxy; a merge to main is built, tested and deployed to the internal server | CI run with unit tests and Pact verification; audit verify output (clean and tampered); deploy log and smoke test result |
| U2 Loki spike and manual filters | V-01 to V-04 and V-07 checks, Loki connector, templates, Filter_Set validation, time handling, windows and caps, identifier mapping, manual filter chips, results states | 4, 6, 7, 8, 9, 12, 14, 21 | Developers, DevOps | With V-01 done and read-only access approved, a QA searches the RQ-05 request (time, identifier, feature) against BCAS UAT Loki and sees editable chips, a bounded window in WIB, and results or "nothing found"; no Source is queried without a confirmed window; a window above Max_Range is refused; a query without the binding selectors is refused and audited; spike results are recorded and `design.md` is amended | Spike report; properties P-a, P-d, P-e, P-f at 100 iterations or more; Pact verification; E2E run of the RQ-05 path; audit rows |
| U3 Simplified and raw view | Parser rules registry and default BCAS rule set, masking module, Raw_View, copies, unmask and True_Raw, dictionary | 11, 15, 16, 17, 18, 19, 20 | Developers, BA (dictionary) | A fixture line from V-10 produces a Simplified_View with no ANSI codes or escape characters and the Raw_View keeps every field path; masking is on by default in every view and copy; the unmask toggle and True_Raw appear only with an Unmask_Right and each use is audited; an unparseable line shows "could not be simplified" | Properties P-b and P-c at 100 iterations or more; fixture set from V-10; E2E for the Unmask toggle; audit rows |
| U6 Pilot instrumentation and escalation | Metric events, PM page, ping entry, MET-04 classification, CSV, "Ask an engineer", Escalation_Link, outage records | 25, 27, 28, 29 | Developers, PM, QA | The PM page shows weekly searches, self-served and escalated lookups, NL share, median and p90 time to readable log, and classifies the result as met, partly met, no change or "unclassified"; an Escalation_Link opens for a Project member and shows access denied with no content for a non-member; Source outages are recorded | Unit tests for classification; E2E for the Escalation_Link (member and non-member); CSV export sample from test data |
| U4 Natural-language search | Pre-processor, inference client on the inference VM, schema validation, fallback, Search_History and guidance | 10, 13 | Developers, System Architect, DevOps | The RQ-01 sentence produces chips and runs nothing until confirmed; unknown items are dropped and listed; with the inference server down, manual filters and pre-processor chips still work; "minta log lagii" without identifier or time shows guidance and the user's own history | NL accuracy report on the RQ-01..RQ-24 test set; property P-d with NL-like output; E2E for the fallback |
| U5 Database and Kubernetes sources | Pod logs through Loki, database connector with limits, cross-service grouping and fallback linking | 22, 23, 24 | Developers, DevOps | Pod logs of an Environment are searchable only after DevOps confirms Loki coverage; the database Source stays disabled until DevOps and the database owner record approval of its limits; a query that reaches the timeout is cancelled at the database; lines from several services with empty correlation_id are grouped and labelled "not linked by correlation_id" | Integration tests with limits; recorded approvals; grouping tests on fixtures |

## Definition of Done (every unit, TD-12)

A unit is done when all of these hold:

1. All unit acceptance criteria above pass.
2. Unit tests, properties and Pact provider verification are green in CI.
3. E2E tests for the unit's flows are green.
4. The unit is deployed to the internal server and smoke-tested.
5. The log-hygiene test passes: no filter value, sentence or log content appears in the finder's own logs.
6. Audit entries for the unit's actions are checked.
7. The unit owner and QA have reviewed the result.
8. `design.md` is amended through a versioned change entry if a verification changed an assumption.

## Tasks

### Wave 1: U1 Foundation and access (starts after H-04 stack confirmation)

#### Group A: backend

- [ ] A.1 Backend foundation and schema
  - Go module and Echo service under `/backend` with the package layout from `design.md`; PostgreSQL migrations for every table in the ER diagram; versioned per-Project configuration with audit of changes; `/healthz`, `/readyz`, `/metrics`; structured JSON logs without sensitive values; OpenAPI skeleton `backend/openapi/finder.yaml` for the U1 endpoints; the `Connector` interface (Req 31.5).
  - Test-first: configuration versioning and the log-hygiene test.
  - _Requirements: 5, 30, 31 (design: DD-02, DD-03, DD-15, DD-22)_
- [ ] A.2 Access and audit
  - Local accounts with argon2id; server-side sessions, CSRF, idle timeout parameter; RBAC tables and the single `Authorize` function; account and grant admin API; append-only `audit_event` with permission-restricted role, hash chain and the `audit verify` command; sign-in failure delay.
  - Test-first: `Authorize` cases (default deny, revocation on next request), session end on account removal, hash-chain tamper detection; Pact provider verification for the U1 endpoints.
  - _Requirements: 1, 2, 3, 26 (design: DD-16, DD-17, DD-18, DD-19)_

#### Group B: frontend and deploy

- [ ] B.1 Frontend shell and admin screens
  - Vite and React app under `/frontend` with TanStack Router and Query, Tailwind, shadcn/ui, lucide-react; sign-in, Project and Environment switcher, account and grant admin screens, Source_Binding screens (non-secret fields and approval record); Pact consumer tests for the U1 endpoints.
  - _Requirements: 1, 2, 3, 5_
- [ ] B.2 Deployment and CI/CD
  - `deploy/` with Docker Compose (reverse proxy with TLS from an internal CA, finder serving the built SPA, PostgreSQL), `finder.env.example` with names only, runbooks (start, roll back, rotate secrets, verify the audit chain, back up and restore PostgreSQL); CI pipeline: build, unit and property tests, Pact publish and can-i-deploy, image build, automatic deploy of main to the internal server (TD-11); request to DevOps for the outbound rules to each Project's Loki, database and the inference VM.
  - _Requirements: 30 (design: DD-20, DD-21)_

### Wave 2: U2 Loki spike and manual filters (needs Wave 1, V-01 and approved read-only access)

#### Group C: backend

- [ ] C.1 Loki spike
  - Run the §6 checklist (V-01) and the API checks V-02, V-03, V-04 and V-07 from the finder host against the BCAS UAT Loki with the approved read-only access; decide the multi-value query form and `loki_direction`; record the results; amend `design.md` through a versioned change entry and an audit row; update the parameter proposals.
  - _Requirements: 21 (design: DD-01, DD-04, DD-05, DD-06; V-01..V-04, V-07)_
- [ ] C.2 Query builder, Loki connector and search API
  - Filter_Set model and validator; WIB and UTC conversion; window rules (exact, approximate, vague, no date, Max_Range); template registry and renderer with binding injection and escaped values; binding guard; Loki connector (read-only, timeout, concurrency, retention message); Source_Binding approval flow; Identifier_Mapping API; search API with confirmation, Result_Cap, "nothing found", partial results, multi-value labels.
  - Test-first: properties P-a, P-d, P-e, P-f.
  - _Requirements: 4, 6, 7, 8, 9, 12, 14, 21_

#### Group D: frontend

- [ ] D.1 Manual filter UI
  - Filter chips (time, Environment, identifier type and value including several values, feature, error code); confirmation of vague windows; Identifier_Mapping screen for the business-configuration role; results list with TanStack Virtual; "nothing found", partial and over-limit states; Pact consumer tests for the search endpoints; E2E for the RQ-05 path against a Pact-mocked backend.
  - _Requirements: 6, 7, 8, 9, 12, 14_

### Wave 3: U3 Simplified and raw view (needs Wave 2 and V-10)

#### Group E: backend

- [ ] E.1 Parser, masking, views and copies
  - Step 0: first real-request check of U2: run the RQ-05 path end to end on the deployed finder against BCAS UAT and record the evidence.
  - Parser rules registry with the fixed step types, rule-set validation, default BCAS rule set from the V-10 fixtures; Simplified_View and Raw_View endpoints; masking module as the single response choke point (key rules, value rules, searched identifier shown to its searcher); True_Raw endpoint with grant check and audit before response; copy endpoint with audit; unmask toggle state reset per search and Session; Business_Term_Dictionary API with "unconfirmed" state; Simplified_View field configuration.
  - Test-first: properties P-b and P-c.
  - _Requirements: 11, 15, 16, 17, 18, 19, 20_

#### Group F: frontend

- [ ] F.1 Views, copies and business configuration screens
  - Create the masked fixture set in `/fixtures` from V-10; Simplified and Raw view with toggle; copy buttons that show masked or unmasked; Unmask toggle and True_Raw shown only with the right; dictionary and view-field screens for the business-configuration role; Pact consumer tests; E2E for the Unmask toggle (with and without the right).
  - _Requirements: 11, 16, 17, 18, 20_

### Wave 4: U6 Pilot instrumentation and escalation

#### Group I: backend

- [ ] I.1 Metrics, escalation and outages
  - Metric events (search, view open, raw open, copy, escalation, session); weekly aggregates; ping count entry; MET-04 classification including "unclassified"; CSV export; escalation record and Escalation_Link with access re-check; Source outage recording; audit and metrics endpoints for the PM and System Architect.
  - Test-first: classification table from MET-04 and requirement 27.6 and 27.7.
  - _Requirements: 25, 26, 27, 29_

#### Group J: frontend

- [ ] J.1 PM page and escalation UI
  - PM weekly metrics page and CSV button; ping entry form; "Ask an engineer" on result screens and "nothing found"; Escalation_Link page for members and the access-denied state; Audit_Log viewer for the PM and System Architect roles; Pact consumer tests; E2E for the Escalation_Link with a member and a non-member.
  - _Requirements: 25, 26, 27_

### Pilot-start gate (after Wave 4)

Pilot users (BCAS UAT, fewer than 10) start when U1, U2, U3 and U6 are done and these open items are closed: read-only access approvals, MET-01 baseline count, audit retention value, Loki retention value. See "Open Decisions".

### Wave 5: U4 Natural-language search and U5 Database and Kubernetes sources (join during the pilot)

#### Group G: backend

- [ ] G.1 Natural-language layer, history and guidance
  - Deterministic pre-processor (dictionary terms, identifier patterns, time expressions, environment words); inference client with timeout; Filter_Set JSON Schema validation against the Project vocabulary; fallback when the inference server is unavailable; Search_History per user and the guidance for searches without identifier or time; NL test set from RQ-01..RQ-24 (placeholders only) and an accuracy report used to choose the model on the inference VM.
  - Test-first: property P-d with NL-like output.
  - _Requirements: 10, 13_
- [ ] G.2 Kubernetes through Loki, database connector and cross-service grouping
  - Enable the Kubernetes source per Environment after DevOps confirms coverage (V-06); database connector for the tables listed for the Project (V-08) with timeout, row cap, range and concurrency limits, disabled until approval is recorded; cancel at the database on timeout; grouping by service when correlation_id is empty, linking by shared identifier and, once V-05 confirms propagation, by X-Request-Id or traceparent.
  - _Requirements: 22, 23, 24_

#### Group H: frontend

- [ ] H.1 NL box, history, database and grouped results
  - Natural-language box with chips and "not understood" list; fallback message; history and guidance panel; grouped cross-service view with link labels; database results and "retry later" state; Pact consumer tests; E2E for the RQ-01 sentence path and the fallback.
  - _Requirements: 10, 13, 22, 23, 24_

### Wave 6: Verification and pilot readiness

#### Group K: both workers, separate directories

- [ ] K.1 End-to-end verification and pilot readiness
  - Backend worker: performance check of time to a readable log against the 2-minute target, set the per-search target from spike data (N-02), outage and fallback behavior (Req 29), audit verification run, log-hygiene test across all endpoints, coexistence check that no Source configuration was changed (Req 31).
  - Frontend and deploy worker: full Playwright suite for the sampled request flows (RQ-01, RQ-05, RQ-07, RQ-10, RQ-11 patterns), README with run and teardown instructions, runbook review with DevOps, deck tracker update.
  - _Requirements: 28, 29, 30, 31_

## Open Decisions and Gates

TD-13 = 3: owners are accepted as proposed; due dates are decided later at task execution. **The brief (section 1.7) asks for an owner and a due date for every open blocker. This plan leaves the due dates open by the user's choice; confirm at the Human Review Gate.**

| Item | Gates | Owner | Due date |
|---|---|---|---|
| H-04 stack confirmation | Wave 1 (U1) | Developers | Set at task execution |
| V-01 Loki access path (§6 checklist) | Wave 2 (U2) | DevOps (Hadi Septian) | Set at task execution |
| Read-only Loki access approval for BCAS UAT | Wave 2 (U2) | Yosia Alvin and DevOps | Set at task execution |
| V-10 masked raw-log sample | Wave 3 (U3 parser) | DevOps or a Developer | Set at task execution |
| Inference VM (DD-10) | Wave 5 (U4) | DevOps | Set at task execution |
| V-06 pod logs in Loki; V-08 database engine and tables | Wave 5 (U5) | DevOps, Developers | Set at task execution |
| DB limits approval (D-03) | Wave 5 (U5 database source) | DevOps, Hadi Septian | Set at task execution |
| 3-month chat export and ping count (MET-01) | Pilot start | PM, BA | Set at task execution |
| Audit retention (OI-05) | Pilot start | PM, compliance | Set at task execution |
| Loki retention (V-07) | Wave 2 (U2) | DevOps | Set at task execution |

Other carried items without a gate: per-search time target (N-02, set in K.1), MET-04 gaps (OI-06), masking partial pattern (PM, DD-13), proposal values in `design.md` Parameters.

## AI Proposals to Confirm at Stage 5 Review

| ID | Proposal | Reason |
|---|---|---|
| D5-01 | Each wave is one vertical slice (TD-03) split between two workers by directory (TD-06); the contract between them is Pact plus the OpenAPI file owned by the backend group | Reconciles TD-03 with TD-06 without shared files |
| D5-02 | Automatic deploy on main (TD-11 = 2) comes with: protected main branch; a deploy credential and a runner with a network route to the internal server (DevOps); the database Source and the RC binding stay disabled by configuration until their approvals, so a deploy cannot enable them; a pause switch in the pipeline during the pilot; rollback by previous image tag (TD-10). Server-side sessions in the database survive a restart | TD-11 removes the manual gate that the recommendation had; these controls replace it |
| D5-03 | U6 runs in Wave 4, before U4 and U5, to meet the pilot-start rule (TD-02) | TD-02 = 1 |
| D5-04 | Relative order only: no durations or dates in this plan (TD-08 = 2); roles only (TD-07 = 2); due dates at task execution (TD-13 = 3) | User answers; deviation from brief section 1.7 flagged above |
| D5-05 | 15 coarse tasks (TD-05 = 2); each Bolt's plan step splits a task into sub-tasks | User answer |
| D5-06 | The first real-request check of U2 is step 0 of E.1, because it needs the frontend of Wave 2 | Wave 2 groups cannot depend on each other |

## Traceability: Requirements to Tasks

| Requirement | Tasks | Requirement | Tasks |
|---|---|---|---|
| 1 Local sign-in | A.2, B.1 | 17 Raw view | E.1, F.1 |
| 2 Manage accounts | A.2, B.1 | 18 Copies | E.1, F.1 |
| 3 Access per Project and Environment | A.2, B.1 | 19 Masking | E.1 |
| 4 Isolation and read-only | C.2 | 20 Unmask and True_Raw | E.1, F.1 |
| 5 Per-Project configuration | A.1, B.1 | 21 Loki connector | C.1, C.2 |
| 6 Manual filters | C.2, D.1 | 22 Kubernetes through Loki | G.2, H.1 |
| 7 Window and cap | C.2, D.1 | 23 Database Source | G.2, H.1 |
| 8 Timezone | C.2, D.1 | 24 Cross-service view | G.2, H.1 |
| 9 Vague times | C.2, D.1 | 25 Escalation | I.1, J.1 |
| 10 Natural-language search | G.1, H.1 | 26 Audit log | A.2, I.1, J.1 |
| 11 Dictionary | E.1, F.1 | 27 Pilot metrics | I.1, J.1 |
| 12 Identifier mapping | C.2, D.1 | 28 Performance | K.1 |
| 13 No identifier or time | G.1, H.1 | 29 Availability | I.1, K.1 |
| 14 Nothing found | C.2, D.1 | 30 Hosting | A.1, B.2, K.1 |
| 15 Deterministic parsing | E.1 | 31 Coexistence | A.1, K.1 |
| 16 View fields | E.1, F.1 | | |

All 31 requirements have at least one task.

## Stage 5 Self-check (brief section 0.2)

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Every acceptance criterion is testable | Pass | Each unit lists testable criteria and evidence; tasks reference requirements whose criteria are EARS |
| 2 | Every rule traces to a source | Pass | Planning Decisions table cites TD answers; tasks cite requirements and design decisions; new proposals are D5-01 to D5-06 |
| 3 | Respects stack, security and scope limits | Pass | Stack and layout per `design.md`; production excluded; DB source and RC stay disabled until approvals; no Source configuration changes |
| 4 | Reuses existing systems, roles and data | Pass | Loki, Kubernetes logs through Loki, existing database used read-only; roles from M-04 |
| 5 | Evidence of the check is shown | Pass | This table, the Traceability table and the Definition of Done |
| 6 | Open questions flagged, not silently decided | Partly | Due dates for open items are left open by the user's answer (TD-13 = 3); flagged for the gate |
