# Deck Update Inputs: Simplified_Log_Finder_Inception_Update_Nov_2026.pptx

AI-DLC Inception. Created 2026-11-25. Status: **Living tracker; Stage 3 approved 2026-11-25; Stage 4 in progress (decision file awaiting answers)** (last update 2026-11-25).

## Purpose

This file holds the data needed to complete the deck `Simplified_Log_Finder_Inception_Update_Nov_2026.pptx` (repository root; 10 slides with speaker notes; template with placeholders). For each placeholder it records the current value, its source (file or decision ID in `audit.md`), its status and its owner.

- **Update rule:** update this file at the end of each stage (Stage 2 approval, Stage 3, Stage 4, Stage 5) and when the pilot baseline is counted.
- **Used at:** the Human Review Gate, when the deck is completed with the Inception results.
- **Source of slide content:** text extracted from the deck by the orchestrator on 2026-11-25 (slide titles, placeholders and notes summarized below). Re-check the deck itself before editing it.
- **Security:** no real identifiers here. Screenshots for the deck must be masked (see "Screenshot needs").

**Status values:** Ready = value final for the gate unless a later stage changes it · Partly = value known, something still open · Waiting for Stage N · Waiting for pilot baseline.

## File name mapping (deck notes vs this repository)

| Name used in the deck / its notes | File in this repository |
|---|---|
| `requirement-verification-questions.md` | `aidlc-docs/inception/stage2-questions.md` |
| `stories.md` | `.kiro/specs/simplified-log-finder/requirements.md` (Stage 3, approved 2026-11-25, A-085) |
| `application-design/` | `.kiro/specs/simplified-log-finder/design.md` (Stage 4, not created yet; decision file `_decisions-design.md` drafted 2026-11-25) |
| Units of work / plan | `.kiro/specs/simplified-log-finder/tasks.md` (Stage 5, not created yet) |
| `audit.md` | `aidlc-docs/audit.md` |
| "First deck" (logo, recap numbers) | `Presenter's Deck - Investor's Team - Simplified Log Finder.pdf` (confirmed as the AIDLC deck, A-038) |

---

## Slide 1: Title

| Placeholder | Current value | Source | Status | Owner |
|---|---|---|---|---|
| Presenter names | TBD | Not in workspace | Waiting for gate | PM |
| Date | Date of the Human Review Gate presentation, TBD | — | Waiting for gate | PM |
| One-line Inception status | "Stages 1–3 approved; Stage 4 design decisions open" (update at each stage) | `aidlc-state.md`; A-078, A-085 | Partly | AI updates; PM confirms |
| Team logo (notes: from first deck) | Take from the Presenter's Deck PDF cover | A-038 | Ready (asset to extract) | PM |
| Stage strip "Inception [status]" | "In progress: Stage 4 of 5" (update at each stage) | `aidlc-state.md` | Partly | AI |
| Construction / Operations | "Next" / "Later" as in the template | Template | Ready | — |

## Slide 2: Recap

| Placeholder | Current value | Source | Status | Owner |
|---|---|---|---|---|
| 24/24 chats, −65% pings, <2 min | Unchanged: 24 sampled requests; target −65% log-request pings; readable log in under 2 minutes | Brief §1.1; A-022, A-033, A-034 | Ready | PM |
| Decision thresholds (optional addition) | Met = ≥65% reduction and under 2 min; partly met = 35–64%; no change = 0–34% | MET-04 (A-076) | Ready (see note N-008 in `audit.md`) | PM |
| Root cause | Unchanged from first deck | Presenter's Deck p.6 | Ready | — |
| Mechanism wording | Draft "thin layer over existing Loki API; direct access for BA/QA rejected". Suggested update: "Thin, input/output-agnostic layer over existing sources (Loki first); engineers keep Grafana/DB/k9s; direct Grafana/DB/k9s access for BA/QA rejected" | A-042 (G-01), T-002, C4 | Ready: C-05 = 1 confirms "direct Grafana/DB/k9s access for BA/QA rejected" (unmask inside the finder is a separate, audited grant) | System Architect, PM |

## Slide 3: Inception progress

| Placeholder | Current value | Source | Status | Owner |
|---|---|---|---|---|
| "[X] of 5 stages done" | 3 of 5 (Stage 3 approved 2026-11-25) | A-006, A-078, A-085 | Partly (update at each stage) | AI |
| Stage 1 Context | Approved with corrections 2026-11-25 | A-006 to A-012 | Ready | — |
| Stage 2 Discover | Approved 2026-11-25; all questions answered (C-05 = 1, C-07 = 1, C-08 = 2) | A-078 to A-082 | Ready | — |
| Stage 3 Specify | Approved 2026-11-25: 31 requirements with EARS acceptance criteria; six AI-proposed behaviors accepted by approval | `requirements.md`; A-083, A-085, A-086 | Ready | — |
| Stage 4 Design | In progress: `_decisions-design.md` drafted 2026-11-25 (26 decisions, 17 Blockers; verification checklist V-01..V-11); 20 of 26 answered (A-088 to A-107); waiting for DD-20, DD-21 (DevOps), DD-24 (QA, Developers) | `_decisions-design.md`; P-008, P-009, P-008a | Partly | DevOps, QA, Developers |
| Stage 5 Plan | Not started | — | Waiting for Stage 5 | — |
| Human review gate | Pending | — | Waiting for gate | PM |

**"What changed since last month"** (real changes; each with one reason). Replace the template drafts with these, choosing the most relevant 4–6 for the slide:

| Change | Reason | Source |
|---|---|---|
| Grafana is self-hosted, so Grafana Assistant is likely unavailable; treated as optional | Assistant is a Grafana Cloud feature per the brief | A-007, X-002, X-003, A-044 (G-03) |
| Reuse analysis done (reuse as-is, hybrid, hybrid via Grafana MCP, replace with OpenObserve, build from scratch); hybrid chosen | Keeps engineers' tools; only differentiators are built | `stage2-actors-constraints-assumptions.md` §5; A-042 |
| Engine is input/output-agnostic; approach decided per source in its own session | Must not break or majorly modify existing tooling | A-042 (G-01) |
| Grafana MCP server and Grafana Assistant optional | Availability on each source unknown | A-043, A-044 (G-02, G-03) |
| "All projects" = multi-project design, one-project pilot (BCAS) | Resolves the "applied across projects" vs one-team pilot contradiction | A-015, A-056 (S-01, S-02), X-001 |
| UAT first, RC added within the pilot | The known log sample is from UAT; RC Loki data to be confirmed | A-010, A-017 (S-03) |
| Hosted on an internal Soluix server/VM on the UAT/RC network ("local first") | Pilot users can reach it; nothing leaves the internal network | A-018 (H-01) |
| Local accounts managed by the Project Admin instead of SSO | Decided by DevOps/System Architect for the MVP | A-020 (H-03) |
| LLM runs internally; only the user sentence is sent, never log content; LLM outputs structured filters only | Banking data stays inside Soluix; templates prevent free-form queries | A-029, A-048 (L-01, L-03) |
| Simplified view by deterministic parsing only | Repeatable and testable; no invented content in logs forwarded to clients | A-049 (L-04) |
| No result sharing in the MVP; the "Ask an engineer" link is the only link and re-checks access | Reduces exposure; keeps escalations measurable | A-075 (O-02), A-080 (C-07) |
| Unmask / true-raw right can be granted to any user (incl. BA/QA) per user, project, environment; off by default; audited. T-002 covers direct Grafana/DB/k9s access only | Keeps C-01 within C5 | A-079 (C-05) |
| "Loki without authentication" is an unverified working assumption until the Loki spike | Answer had no verification source | A-081 (C-08), RK-002 |
| UAT/RC data confirmed synthetic; masking stays on by default | Confirmed by the BCAS lead | A-062 (C-04) |
| Stack chosen: Go + Echo; TypeScript, React, Vite, TanStack Router/Query/Virtual, Tailwind, shadcn/ui, lucide-react; PactFlow contract tests | Team standard; to be confirmed with Developers | A-040 (H-04) |
| Deferred: Grafana Assistant trial; image input (next phase) | Assistant optional; images are next phase (constraint C9) | A-044; C9 |

## Slide 4: "AI proposes, humans decide"

| Placeholder | Current value | Source / method | Status | Owner |
|---|---|---|---|---|
| [##] requirement gaps AI raised (Clarity) | 88 clarification questions so far: 62 in Stage 2 + 26 Stage 4 design decisions (see tracker below) | `stage2-questions.md` sections 1–13; `_decisions-design.md` DD-01..DD-26 | Partly (Stage 4 follow-ups and Stage 5 may add) | AI |
| [##] corrections / re-prompts (Rework) | 1 correction turn with 5 corrections; 0 re-prompts to redo AI work | `audit.md`, `session-handoff.md` | Partly | AI |
| [## days] brief to approved package (Speed) | Start 2026-11-25; end = Human Review Gate approval date, TBD | See tracker | Waiting for gate | PM |
| [#]/5 confidence | TBD at gate | Team vote at gate | Waiting for gate | PM |
| "Overrode [##] AI proposals" | 19 of 57 answered questions that had a recommended option (C-08 added) | See tracker | Partly | AI |
| Screenshot of questions file | See "Screenshot needs" | — | Ready to capture | PM |

## Slide 5: "Six decisions shape the build"

All dated 2026-11-25. Owner = owner of the source question.

| Row | Outcome | Why | Owner | Source | Status |
|---|---|---|---|---|---|
| Build vs reuse | Hybrid: thin, input/output-agnostic layer over existing sources; approach per source | Keeps Grafana/DB/k9s for engineers (C4); builds only what is missing | System Architect | A-042 (G-01) | Ready |
| NL search scope | LLM turns a sentence into structured filters only; finder runs predefined LogQL/SQL templates; no free-form queries | Main control against injection and over-broad queries (C5) | System Architect | A-048 (L-03) | Ready |
| Environments in scope | UAT and RC only; UAT first, RC within the pilot; production excluded | Lower exposure; known sample is UAT | User (PM, DevOps) | A-010, A-017 (S-03) | Ready |
| Masking in Copy raw | Masked by default in all views and copies; users with the right may toggle masking off or copy true raw; the Project Admin can grant the right to any user, audited | Users must confirm the right log; banking data protected (C1) | System Architect, PM | A-051, A-052, A-054, A-059, A-079 (SEC-02, SEC-03, SEC-05, C-01, C-05) | Ready (X-004 resolved by C-05: T-002 covers direct Grafana/DB/k9s access only) |
| LLM hosting | Internal/local LLM; nothing leaves the Soluix environment; only the user sentence is sent, never log content | Sentences contain identifiers; logs contain banking data | PM, System Architect | A-029 (L-01), A-018 (H-01) | Ready (model and host chosen in Stage 4) |
| k9s / Kubernetes source | Served through Loki; no direct pod-log reading in the MVP; deleted pods covered by Loki retention (period TBD) | Avoids cluster RBAC; Loki already ships pod logs (DevOps to confirm) | DevOps | A-069, A-070 (K-01, K-02) | Partly (DevOps confirmation in Stage 4) |

## Slide 6: Requirements

| Placeholder | Needed | Source | Status | Owner |
|---|---|---|---|---|
| Personas list | P1 Business Analyst and P2 QA Engineer (primary searchers); P3 FE / Mobile Developer as searcher (same rights as BA/QA); P4 Developer / Engineer (escalation); P5a Project Admin, business configuration (BA: dictionary, identifier mapping, view fields); P5b Project Admin, access manager (DevOps: accounts, access, unmask grants, source bindings); P6 Project Manager (pilot metrics, audit) | `requirements.md` "Personas"; A-026, A-036 | Ready (Stage 3 approved, A-085) | BA |
| "[##] stories total" | **31** (one user story per requirement: 27 functional, 4 non-functional) | `requirements.md` | Ready (Stage 3 approved, A-085) | AI |
| "[##] traced to real chats" | **28 of 31** stories cite at least one of the 24 sampled requests (RQ-01..RQ-24, Ideation p.4–11); Requirements 1, 2, 31 trace to root causes and constraints only. All 24 requests map to requirements; only RQ-23 (a) "example endpoint response" is not answerable by design (O-04) | `requirements.md` "Traceability Matrix" | Ready (Stage 3 approved, A-085) | AI |
| Example story ("ganti rek sumber" request) | **Requirement 10** (natural-language search): "As a Searcher, I want to type my request as a sentence, like I would in chat, so that the Finder fills in the filters for me." Example: Given BCAS has "ganti rek sumber" in its dictionary, When a QA Engineer types "`<CIF>` di jam 15:00:33 ini ya, minta tolong ambilin log ganti rek sumber di RC" (RQ-01), Then the Finder shows chips Time 15:00:33 WIB, Environment RC, CIF `<CIF>`, Feature "ganti rek sumber", and runs nothing until the QA Engineer confirms. Alternative for the brief's wording (16:04:44): Requirement 7 example (RQ-05) | `requirements.md` Req 10, Req 7 | Ready (Stage 3 approved, A-085) | BA |
| Screenshot of stories file | `requirements.md`, section "Requirement 10: Natural-language search producing structured filters" (user story, acceptance criteria 1–3, Given/When/Then example) | See "Screenshot needs" | Ready to capture | PM |

## Slide 7: Architecture

| Placeholder | Current value / correction needed | Source | Status | Owner |
|---|---|---|---|---|
| Components (Finder UI, NL-to-filter layer, Query builder with predefined LogQL/SQL templates, Source connectors, Result renderer) | Consistent with decisions so far; final names from `design.md` | A-042, A-048, A-049 | Waiting for Stage 4 | System Architect |
| Sources (Loki API reused, Database, Kubernetes logs) | Kubernetes logs served through Loki (A-069); database enabled only after limits are approved (D-03, approver Hadi Septian) and the table list is compiled (D-01) | A-066, A-068, A-069 | Partly | DevOps |
| Per-project dictionary / mapping / environments | Consistent: dictionary owned by BA; mapping and environments per project (M-08 source binding) | A-025, A-026, A-073 | Ready | BA, DevOps |
| Cross-cutting "SSO and role-based access" | **Correct to** "Local accounts managed by the Project Admin; access per project and environment, least privilege" | A-020 (H-03), A-050 (SEC-01) | Ready (correction) | System Architect |
| Cross-cutting masking, audit | Consistent: masking on by default; audit per SEC-04 option 1 (retention TBD by compliance) | A-051, A-053 | Partly (grants pending C-05) | System Architect |
| Placeholder list of four DevOps assumptions (incl. "Grafana Cloud") | **Replace with current state:** Grafana self-hosted (A-007); Loki reached directly without authentication, network-restricted (A-063, **pending C-08**); one Loki per project (A-064); no database read-replica (A-067); Kubernetes logs via Loki (A-069) | P-004c | Partly (C-08) | DevOps |
| Final diagram (notes: `application-design/`) | Final diagram comes from `design.md` after Stage 4 decisions (component diagram; scope asked in DD-26) | Stage 4; `_decisions-design.md` DD-02, DD-26 | Waiting for Stage 4 | System Architect |

## Slide 8: MVP slice (units U1–U6)

Owner, target date and dependencies per unit, and the Construction start date, come from Stage 5. Known dependencies now:

| Unit | Known dependencies / inputs so far | Source | Status | Owner |
|---|---|---|---|---|
| U1 Foundation: access and config | Local accounts (A-020); access per project and environment (A-050); unmask/true-raw grants (A-059, **C-05**); account removal requirement (N-003) | `audit.md` | Waiting for Stage 5 | TBD |
| U2 Loki spike + manual filters | Source of G-04 answer (**C-08**); read-only access approved by DevOps and the BCAS lead Yosia Alvin (A-045, A-060); network route from the finder host (A-018) | `audit.md` | Waiting for Stage 5 | TBD |
| U3 Simplified + raw view | Masked raw-log text export and full label list before the parser is built (A-061, C-03) | `audit.md` | Waiting for Stage 5 | TBD |
| U4 NL search | Internal LLM host and model (A-029; chosen in Stage 4); dictionary (A-025) | `audit.md` | Waiting for Stage 5 | TBD |
| U5 DB and K8s sources | Table list (A-066, D-01); limits approved by Hadi Septian (A-068, D-03); no read-replica (A-067); K8s via Loki (A-069) | `audit.md` | Waiting for Stage 5 | TBD |
| U6 Pilot instrumentation | Ping counting (A-033), timer (A-034), self-serve events (A-035), thresholds (A-076); escalation link rule (**C-07**) | `audit.md` | Waiting for Stage 5 | TBD |
| Construction start date | After the Human Review Gate | — | Waiting for Stage 5 | PM |

## Slide 9: Baseline

| Row | Baseline needed | Target | Measurement | Status | Owner |
|---|---|---|---|---|---|
| Pings per week | Average from the 3-month chat export, keyword tagging, BA review | −65% | Chat keyword count (A-033) | Waiting for pilot baseline | PM, BA |
| Time to get a log | Today's time from request to readable log; **baseline method not defined yet** (MET-02 covers in-tool timing only) | <2 min | In-tool timer, first search to first simplified-view open (A-034) | Waiting for pilot baseline; method gap to raise in Stage 3 | PM, QA |
| Engineer minutes per lookup | 5–10 min (estimate from the first deck) | [##] from pilot sampling | Sampling | Waiting for pilot baseline | PM |
| Self-serve rate | n/a today | [##%] | In-tool events (A-035) | Waiting for pilot baseline | PM |
| Weekly chart, last 3 months | Weekly ping counts from the chat export, sanitized (no names, no identifiers) | — | — | Waiting for pilot baseline | PM |

## Slide 10: Lessons, risks, asks

**Lessons (drafts for the gate; team to confirm):**

| Item | Draft | Source |
|---|---|---|
| What worked | Decision file with a recommended option per question; reference review found new requesters (FE/Mobile) and client-facing copies; contradictions flagged and asked instead of decided (X-001 to X-006) | `audit.md` |
| What failed / needed rework | Stage 1 assumed Grafana Cloud and an unknown hosting meaning (corrected by the user, A-006 to A-012); the SEC-06 recommendation came after the PDFs were already pushed | `audit.md`; `session-handoff.md` turn 3 |
| Where human judgment was essential | Unmask / true-raw rights vs T-002 (C-01, C-05); audit kept after being questioned (SEC-04); input/output-agnostic direction (G-01); confirming that UAT/RC data is synthetic (C-04) | `audit.md` |

**Top risks (replace template drafts):**

| Risk | Current state | Owner | Due | Source |
|---|---|---|---|---|
| LLM hosting not approved (template draft) | **Decided:** internal/local LLM, sentence only; remaining work is choosing host and model in Stage 4 | System Architect | Stage 4 | A-029 |
| Read-only access to Loki and DB not granted | Still open; approvers: DevOps + Yosia Alvin (Loki/DB access), Hadi Septian (DB limits) | DevOps | Before U2 / U5 | A-045, A-060, A-068 |
| **New: Loki reachable without authentication (RK-002)** | Anyone with a network route can query Loki; isolation relies on one Loki per project plus finder checks; source of the answer pending C-08 | DevOps (Hadi Septian), System Architect | Before the Loki spike | A-063, RK-002 |
| No DB read-replica | **Confirmed** (D-02); risk stays high: finder queries hit the primary UAT/RC database; limits mandatory before enabling | DevOps | Before U5 | A-067, A4 |
| Masking rules undefined (template draft) | **Defined:** masked by default, parameterizable, toggle by rights; who may be granted unmask/true-raw pending C-05 | System Architect, PM | Before Stage 3 approval | A-051, A-059 |
| Sensitive data in reference PDFs (RK-001) | Lowered: UAT/RC data confirmed synthetic (C-04); repository still stays private | PM | — | A-055, A-062 |

**Next / Asks (replace template drafts):**

| Ask | Current state | Owner |
|---|---|---|
| Run U1 after the gate | Unchanged | PM |
| Approve LLM hosting | Replace with: "Provide an internal host for the LLM (model chosen in Stage 4)" | DevOps, System Architect |
| Read-only access to Loki gateway and DB | Replace "gateway" (Loki has no gateway per G-04, pending C-08) with: "Read-only access to BCAS UAT/RC Loki and DB; decide whether Loki needs an authenticating proxy (RK-002)" | DevOps, Yosia Alvin |
| Confirm pilot team and window | Pilot project BCAS (A-056), fewer than 10 users (A-021); window TBD | PM |

---

## Retrospective metrics tracker

Recompute at the end of each stage. **Update 2026-11-25 (Stage 3 approved; Stage 4 decision file drafted):** Clarity 88 (26 new design decisions DD-01..DD-26); Rework unchanged (Stage 3 approved without corrections); Overridden unchanged until the Stage 4 answers are in. Earlier counts below are as of 2026-11-25 (Stage 2 approved; Stage 3 drafted). Stage 3 raised no new clarification questions: open points went to `requirements.md` "Open Items Carried to Stage 4" (OI-01..OI-13) and AI proposals to confirm are logged as A-084, so Clarity stays 62. Stage 2 approval was given without corrections, so Rework is unchanged. Overrides: section 13 answers C-05 = 1, C-07 = 1 and C-08 = 2 (C-08 had a recommended option 1), so 57 answered questions now had a recommended option and 19 of them were overridden.

### Clarity: clarification questions raised before code

- **Count:** 88 (updated 2026-11-25 after Stage 3 approval): 62 from Stage 2 + 26 from Stage 4.
- **Stage 4 method:** count decision headings `### DD-xx` in `.kiro/specs/simplified-log-finder/_decisions-design.md` (DD-01..DD-26 = 26). The verification checklist rows V-01..V-11 are not counted: they are checks of facts already asked in Stage 2 (G-04, K-01, K-02, M-02, M-08, M-09, D-01, U-01, C-03, G-02/G-03, item 17), not new questions. The "Already decided (inputs)" table is not counted.
- **Stage 2 method:** count question IDs (headings `### <ID>`) in `stage2-questions.md` sections 1–13. Per section: 1 Scope 3, 2 Hosting 5, 3 Non-functional 3, 4 Reuse and access 8, 5 Database 3, 6 Kubernetes 2, 7 Search and mapping 8, 8 LLM and output 7, 9 Security 4, 10 Metrics 4, 11 Reference review 8, 12 Blocker review 4, 13 Section 12 review 3 (C-05, C-07, C-08; C-06 not raised). Items in "Already answered (not re-asked)" are not counted. Stage 3–5 questions are added when raised.

### Rework: user corrections and re-prompts

- **Count:** 1 correction turn containing 5 corrections to AI output; 0 re-prompts asking the AI to redo work.
- **List:** Turn 1 (2026-11-25), corrections to the Stage 1 proposal: (1) Grafana is self-hosted, not Cloud; (2) Loki access path unknown, defer to technical team; (3) only Developers and DevOps use Grafana; (4) environments UAT and RC only; (5) hosting "local first" (A-006 to A-012).
- **Not counted (method):** turns 2, 3, 6, 8 (new material, resume, status); turns 4, 5, 7 (users fixing their own answers after AI review: SEC-04 changed to option 1, S-02 name, G-06 name, SEC-04 leftover text). An AI self-correction (turn 3: the SEC-06 recommendation was too late because the commit was already pushed) is a lesson, not user rework.

### Overridden AI proposals

- **Count:** 19 of 57 answered questions that had a recommended option (updated 2026-11-25 after section 13 answers).
- **Method:** a question counts when the answer chose a non-recommended option number or wrote its own text on the answer line. Excluded: 5 questions with no recommended option (G-04, G-05, G-08, D-02, M-02). Filling a blank inside the recommended option (S-02, G-06, D-03, MET-04 values) is not an override.
- **Non-recommended option chosen (14):** H-03 (2), N-01 (2), M-06 (3, own text), L-02 (2), O-02 (2), SEC-01 (3, own text), SEC-02 (4, own text), SEC-03 (3, own text), SEC-05 (4, own text), SEC-06 (3), C-01 (3), C-02 (2), C-04 (2), C-08 (2). C-05 and C-07 took the recommended option 1.
- **Own text on the answer line (5):** H-04, R-01, G-01, G-02, G-03.
- **Note for the slide:** 4 of the 19 agree in substance with the recommendation and extend it (H-04 names the stack, R-01 adds directories, G-01 adds input/output-agnostic, SEC-02 adds parameterizable). Substantive overrides: 15.

### Speed

- **Start:** 2026-11-25 (first prompt commit `399e472`, 09:18 +0700; initial files dated 2026-11-24).
- **End:** Human Review Gate approval date, TBD. Speed = end − start in days.

### Human effort and confidence

- TBD at the gate (team estimate of hours spent; confidence vote 1–5).

## Screenshot needs

| Slide | File | Section to capture | Masking reminder |
|---|---|---|---|
| 4 | `aidlc-docs/inception/stage2-questions.md` | Section 12 (C-01 to C-04 with answers) or section 4 `G-06`/`G-07`; shows question, recommended option and the human answer | Blur client and project names (e.g. BCA/BCAS) and any example label values (e.g. the `project_name` value in `M-08`); person names only if the team agrees |
| 6 | `.kiro/specs/simplified-log-finder/requirements.md` (Stage 3) | "Requirement 10: Natural-language search producing structured filters" (the "ganti rek sumber" story, RQ-01, with acceptance criteria and Given/When/Then) | Identifiers must appear only as placeholders (`<CIF>`, `<ACCOUNT_NO>`) |
| 7 | `.kiro/specs/simplified-log-finder/design.md` (Stage 4) | Component diagram | No internal hostnames, URLs or credentials |
| 9 | Chat export summary (pilot baseline) | Weekly ping chart | No names, chat text or identifiers |

## Update 2026-11-26: Stage 4 design.md drafted

| Slide | Placeholder | Value now | Source | Status |
|---|---|---|---|---|
| 1 | Inception status | "Stages 1–3 approved; Stage 4 design drafted, awaiting approval" | A-114 | Ready (update after approval) |
| 3 | Stage 4 status | Design drafted 2026-11-26, awaiting approval (26 decisions answered) | A-108..A-114 | Ready |
| 5 | Build vs reuse / NL scope rows | Confirmed by design.md (hybrid over Loki API; Filter_Set schema has no query field) | design.md Overview, NL layer | Ready |
| 7 | Final diagram | Use the component diagram in `design.md` → Architecture. Replace "SSO and role-based access" with "Local accounts and role-based access"; replace the DevOps assumptions placeholder with: self-hosted Grafana; Loki direct without auth (unverified, V-01); one Loki per project; no DB read-replica; K8s logs via Loki. Gold pill = Loki API and existing DB (reused). Add the separate inference VM | design.md Architecture, Deployment, Assumptions | Ready (after Stage 4 approval) |
| 10 | Risks | Replace drafts with design.md Risks table top items: Loki label-index cost; RK-002 Loki without auth; no read-replica (DB limits); LLM accuracy on Indonesian wording; parser default from screenshot (V-10) | design.md Risks | Ready |
| 10 | Asks | Run V-01 (Loki access) before the spike; provide V-10 masked log sample; inference VM (DD-10); Developers confirm stack (H-04) | design.md Assumptions | Ready |

Retrospective tracker update: Clarity stays 88 (no new questions). Rework unchanged. Overridden AI proposals in design decisions: 1 of 26 (DD-11 option 2). Screenshot for slide 7: `design.md` → "Architecture → Component view" diagram (no identifiers to blur).

## Update 2026-11-26: Stage 4 approved, Stage 5 started

| Slide | Placeholder | Value now | Source | Status |
|---|---|---|---|---|
| 1 | Inception status | "4 of 5 stages approved; Plan in progress" | A-116 | Ready |
| 3 | Title / statuses | "Inception progress: 4 of 5 stages done"; Stage 4 Done; Stage 5 In progress; review gate Not started | A-116, A-119 | Ready |
| 7 | Final diagram | Approved (design.md v1.0) | A-116 | Ready |
| 8 | Units U1–U6 | Proposed in `_decisions-tasks.md`; owners, dates and dependencies after TD-01, TD-07, TD-08 | A-119 | Waiting for Stage 5 |

Deck filling deferred (A-118): produce `Simplified_Log_Finder_Inception_Update_Nov_2026_filled.pptx` when the user asks. Retrospective tracker: Clarity 88 + 13 task decisions = 101 (method: count `### TD-xx` headings). Rework unchanged.

## Update 2026-11-26: Deck filled (A-120)

File: `Simplified_Log_Finder_Inception_Update_Nov_2026_filled.pptx` (original unchanged). Still open in the filled deck: slide 1 presenter names and date; slide 4 days to approval and confidence, plus the screenshot; slide 6 screenshot; slide 7 optional diagram picture; slide 8 unit owners and target dates (PM, TD-07/TD-08) and Construction start; slide 9 baseline values, time-to-log baseline method and weekly chart; slide 10 lessons need team confirmation and risk due dates (TD-13). Re-run after Stage 5 approval and after the baseline count. Slide 4 numbers used: Clarity 101, Rework 1, overrides 20 of 83.

## Update 2026-11-26: Stage 5 tasks.md drafted

| Slide | Placeholder | Value now | Source | Status |
|---|---|---|---|---|
| 1, 3 | Stage 5 status | "Plan drafted, awaiting approval" (still 4 of 5 done until approval) | A-134 | Ready |
| 4 | Clarity | 101 (62 + 26 + 13); unchanged | `_decisions-tasks.md` | Ready |
| 4 | Overrides | 26 of 96 recommendations (20 of 83 + 6 of 13 Stage 5: TD-05, 06, 07, 08, 11, 13) | N-020 | Ready (replaces 20 of 83) |
| 8 | Unit order and dependencies | Waves: U1, U2, U3, U6, then U4 and U5 together (pilot starts after U1, U2, U3, U6) | tasks.md | Ready |
| 8 | Owners and target dates | Roles only (TD-07); relative plan, no dates (TD-08); dates at task execution (TD-13) | A-127, A-128, A-133 | Stays "TBD" |
| 10 | Risk due dates | Left open by the user's answer (TD-13 = 3) | A-133 | Stays open |

Re-run the deck fill after Stage 5 approval: slide 4 overrides become 26 of 96; slide 8 changes the unit order (U6 before U4/U5).

## Update 2026-11-26: Stage 5 approved, gate prepared

| Slide | Placeholder | Value now | Source | Status |
|---|---|---|---|---|
| 1 | Inception status | "5 of 5 stages done; review gate pending" | A-136, A-138 | Ready |
| 3 | Title / statuses | "Inception progress: 5 of 5 stages done"; Plan Done · 26 Nov; review gate "Awaiting approval" | A-136 | Ready |
| 4 | Overrides | 26 of 96 | N-020 | Ready |
| 4 | Days / confidence | Speed ~2 working days so far; confidence from team vote at the gate | N-021 | Waiting for gate |
| 8 | Unit order | U1, U2, U3, U6 (pilot gate), then U4 and U5; roles only, no dates | `tasks.md` | Ready |
| 10 | Lessons | Use `human-review-gate.md` §6 (worked, failed, human judgment) | N-021 | Ready (team to confirm) |

Refill the deck when the user asks (previous fill A-120 is now out of date for slides 1, 3, 4, 8 and 10).

## Update 2026-11-26: Deck refreshed (A-139)

Newest deck: `Simplified_Log_Finder_Inception_Update_Nov_2026_filled_v2.pptx` (the older `_filled.pptx` was open in PowerPoint, so it could not be overwritten). Values used: 5 of 5 stages done, gate in review; clarity 101; rework 1; overrides 26 of 96; speed ~2 days (gate pending); wave order U1, U2, U3, U6, then U4 + U5. Still open: presenter names and date, confidence score (gate vote), unit owners and dates (task execution), baseline numbers and chart, screenshots.

## Update 2026-11-26: Slide 4 changes (A-140, A-141)

Slide 4 now shows: 101 gaps raised; 26 decisions where we did not take Option 1 (26 of 96); ~2 days actual (gate pending); ~30 days if done with a legacy SDLC (simulation). Bullet: "Did not take the AI's Option 1 in 26 of 96 decisions".

### Legacy SDLC estimate (simulation, working days)

Assumed legacy way: sequential document-driven phases; one BA, one System Architect, one PM, with DevOps and QA consulted; clarification by meetings and email in rounds; a formal review and sign-off after each document. Scope taken from this project: 3 reference documents read; 24 real requests analysed; 101 questions and decisions (62 + 26 + 13); 31 requirements with EARS criteria and a traceability matrix; a design with component, sequence and ER diagrams, about 20 API endpoints and a security model; a plan with 6 units and 15 tasks.

| Step | Assumption | Days |
|---|---|---|
| Context gathering and reading the reference material, analysing the 24 requests | BA | 3 |
| Clarifying 62 open questions | About 3 rounds of meetings and email across PM, BA, Architect, DevOps, QA, with waiting time between rounds | 6 |
| Requirements document: 31 requirements with criteria and traceability matrix | About 0.3 day per requirement | 9 |
| Requirements review and sign-off | One cycle | 2 |
| Design document: components, API, data model, security, 26 decisions | Architect, with decision meetings | 9 |
| Design review and sign-off | One cycle | 2 |
| Implementation plan: units, tasks, dependencies | PM and Architect | 3 |
| Plan review, sign-off and the gate | One cycle | 2 |
| **Sequential total** | | **36** |
| Overlap between phases (design starts during requirement review, planning during design review) | Subtract about 4 to 6 days | **about 30** |
| Range | 25 (more parallel work) to 36 (fully sequential) | |

Caveats: this is an AI simulation from assumed effort, not a measured figure; the actual AI-DLC run took about 2 calendar days with the team answering questions in between, and the team's human-effort hours are not yet recorded, so the two numbers measure different things (elapsed time versus effort). Ask the BA and PM to adjust the assumptions if they have real numbers from past projects.

## Update 2026-11-26: Deck v3 is the working deck (A-142)

Working deck: `Simplified_Log_Finder_Inception_Update_Nov_2026_filled_v3.pptx` (edited by the user in PowerPoint). New slide 9 "Technology stack at a glance" (table: Backend, Frontend, Data, Quality, Deploy and CI/CD, AI and sources; source `design.md`, H-04, TD-11). Note that the user removed the baseline slide and added "Our experience with AI-DLC Inception"; slide numbers in earlier entries of this file refer to the older layout.

## Update 2026-11-26: Screenshots added (A-143)

Slide "AI proposes, humans decide": image of `stage2-questions.md` C-01 and C-04 (recommended option vs the human answer). Slide "Every requirement traces back to a real chat": image of `requirements.md` Requirement 10. Both are rendered excerpts with the client name and the approver name masked. The pilot ping chart is not in v3 (slide removed); add it when the baseline count exists.

## Update 2026-11-26: Gate approved (A-144)

Inception closed. For the deck: slide 1 and the progress slide can say "5 of 5 stages done; gate approved" (currently "gate pending" / "In review"). Speed is final at about 2 working days (25 to 26 Nov). The deck still has no human-effort figure or confidence score (N-028). Refresh the status lines when the user asks.
