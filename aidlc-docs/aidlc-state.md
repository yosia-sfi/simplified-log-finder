# AI-DLC Workflow State

## Project Info
- **Project Type**: Greenfield
- **Repo Model**: Single-repo with separate top-level directories per part (e.g. backend, frontend), so it can be split later (R-01, decided 2026-11-25)
- **Spec Placement**: `.kiro/specs/simplified-log-finder/` (config exists; requirements-first, feature)
- **Existing Stack**: Not applicable (greenfield). Existing log sources the finder will read: self-hosted Grafana / Loki, a database (engine TBD), Kubernetes pod logs. Environments UAT and RC only.
- **Target Architecture**: `design.md` drafted 2026-11-26: Go modular monolith + separate inference VM, PostgreSQL, Docker Compose on the internal server, REST/JSON + OpenAPI + Pact. Decided in Stage 2: hybrid thin layer over existing sources, input/output-agnostic, approach decided per source (G-01); hosted on an internal Soluix server/VM on the UAT/RC network (H-01); internal/local LLM producing structured filters only (L-01, L-03); deterministic simplified view (L-04); local accounts (H-03).
- **Tech Stack (H-04, to confirm with Developers)**: Backend Go + Echo. Frontend TypeScript, React, Vite, TanStack Router, TanStack Query, TanStack Virtual, Tailwind, shadcn/ui, lucide-react. Contract testing with PactFlow in backend, frontend and CI/CD.
- **Active Spec**: simplified-log-finder
- **Deck**: template `Simplified_Log_Finder_Inception_Update_Nov_2026.pptx` (unchanged); newest filled copy `Simplified_Log_Finder_Inception_Update_Nov_2026_filled_v2.pptx` (A-139, 2026-11-26; older `_filled.pptx` is stale); inputs in `aidlc-docs/inception/deck-update-inputs.md`; refresh after gate approval
- **Session handoff**: `aidlc-docs/session-handoff.md` (verbatim prompts/responses and "Resume here"; commit and push before switching laptops)

## Reference Documents (available since 2026-11-25, commit `ac124f7`)
- `Ideation - Simple Log Finder.pdf` (19 pages; idea sheet, 24 chat requests, log screenshot)
- `Presenter's Deck - Investor's Team - Simplified Log Finder.pdf` (11 pages; whether it is the "AIDLC deck": `SRC-02`)
- `Reference AI DLC Workshop v3.pptx` (32 slides)
- Review and remaining gaps: `aidlc-docs/inception/reference-docs-review.md`
- Risk RK-001: PDFs appear to contain real identifiers; decided 2026-11-25 (`SEC-06` option 3): keep as is, repository stays private to the team (risk accepted); severity lowered 2026-11-25 because UAT/RC data is synthetic (`C-04`)
- Risk RK-002 (2026-11-25): Loki reachable without authentication (`G-04`); not verified (`C-08` = 2), working assumption until the Loki spike; open

## Inception Stage Mapping (brief §0.1 → AI-DLC workflow)

| Brief stage | Workflow equivalent | Status |
|---|---|---|
| 1 Context | Context building | Approved with corrections 2026-11-25 |
| 2 Discover | Requirements clarification (`stage2-questions.md` = requirements decision file) | **Approved 2026-11-25** (A-078) |
| 3 Specify | Phase 1 Requirements → `.kiro/specs/simplified-log-finder/requirements.md` | **Approved 2026-11-25** (A-085; A-084 behaviors accepted by approval, A-086) |
| 4 Design | Phase 2 Design → `_decisions-design.md`, then `design.md` | **Approved 2026-11-26** (A-116) |
| 5 Plan | Phase 3 Tasks / units of work | **Approved 2026-11-26** (A-136) |
| Human Review Gate | Full package approval before Construction | **Awaiting approval**: `inception/human-review-gate.md` (A-138) |

## Phase Progress
- [x] Phase 0: Reverse Engineering (skipped: greenfield)
- [x] Stage 1: Context (approved with corrections 2026-11-25)
- [x] Stage 2: Discover (approved 2026-11-25, A-078)
  - [x] 2a. Actors, constraints, assumptions drafted (`inception/stage2-actors-constraints-assumptions.md`)
  - [x] 2b. Question file drafted (`inception/stage2-questions.md`)
  - [x] 2b+. Reference docs reviewed; questions added in section 11 (2026-11-25)
  - [x] 2c. Blocker questions answered by owners (2026-11-25: PM/BA in commit `5806546`, System Architect after it)
    - Fixed 2026-11-25: S-02 = BCAS; G-06 approver named; SEC-04 leftover text removed; due dates apply only to unanswered Blockers (A-058, user to confirm at approval)
    - Section 12 (C-01 to C-04) and all 14 Can-defer questions answered 2026-11-25 (audit A-059 to A-076). BCAS approver = Yosia Alvin (A-060); UAT/RC data synthetic (A-062)
    - Section 13 answered 2026-11-25: C-05 = 1 (X-004 resolved, A-079), C-07 = 1 (X-006 resolved, A-080), C-08 = 2 (G-04 is an unverified working assumption until the Loki spike; RK-002 open, A-081)
  - [x] 2d. Stage 2 approved (2026-11-25, A-078; A-058 due-date rule confirmed, A-082)
- [x] Phase 1 / Stage 3: Requirements (Specify)
  - [x] 1a. Decision input = answered `stage2-questions.md`
  - [x] 1b. requirements.md generated (2026-11-25: 31 requirements, 31 stories, 28 traced to real chats; validator: no format issues) and approved 2026-11-25 (A-085)
- [x] Phase 2 / Stage 4: Design (approved 2026-11-26, A-116)
  - [x] 2a. _decisions-design.md created (2026-11-25: DD-01..DD-26, 17 Blockers, 9 Can-defer; verification checklist V-01..V-11)
  - [x] 2b. _decisions-design.md completed by user (26 of 26 answered: A-088 to A-107, and A-108 to A-113 for DD-14, DD-20..DD-24 on 2026-11-26)
    - Answered: DD-01..DD-09 (commit `ec616d7`); DD-10..DD-13, DD-15..DD-19, DD-25, DD-26 (System Architect, 2026-11-25). All System Architect decisions are answered
    - Still blocking design.md: DD-20 secrets and Source_Binding storage (DevOps), DD-21 runtime/TLS/network (DevOps), DD-24 test layers (QA, Developers)
    - Can-defer open (recommended option used as an open working assumption if left empty): DD-14 (PM, BA), DD-22 (DevOps, Developers), DD-23 (PM, Developers)
    - Follow-ups carried into design.md: N-013 (DD-11 rules registry constraints and editor role), N-014 (DD-18 idle timeout 30 min is a proposal)
  - [x] 2c. design.md generated (2026-11-26, A-114) and approved (A-116; D4-01..D4-07 accepted, A-117)
  - [ ] 2d. Stage 4 technical verifications done: checklist V-01..V-11 in `_decisions-design.md` section 1 (Loki access path, Loki version/API, identifiers in lines, env-to-label mapping, X-Request-Id/traceparent, pod logs in Loki, Loki retention, DB engine/tables, FE/Mobile Grafana access, masked raw-log sample, Grafana MCP/Assistant). DB read-replica already confirmed absent (D-02); timing per DD-01
- [x] Phase 3 / Stage 5: Tasks (Plan) (approved 2026-11-26, A-136)
  - [x] 3a. _decisions-tasks.md created (2026-11-26: TD-01..TD-13, 7 Blockers; units U1–U6 proposed)
  - [x] 3b. _decisions-tasks.md completed by user (13 of 13, A-121 to A-133)
  - [x] 3c. tasks.md generated (2026-11-26, A-134; 6 waves, 15 tasks) and approved (A-136; D5-01..D5-06 accepted, A-137)
- [ ] Human Review Gate (package prepared 2026-11-26: `aidlc-docs/inception/human-review-gate.md`; awaiting approval)
- [ ] Construction (not started; no application code during Inception)

## Current Status
**Phase**: Human Review Gate
**Status**: Awaiting approval of the full inception package (`inception/human-review-gate.md`). Team to give human-effort hours and confidence (1–5). Approval accepts deferred due dates (N-018). Construction blocked until approved
**Last Updated**: 2026-11-26 (Stage 5 approved; gate prepared)
