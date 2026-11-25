# Business Context: Simplified Log Finder

AI-DLC Inception, Stage 1 (Context). Status: **Approved with corrections 2026-11-25**.
Source for every statement: `prompt-inception.md` (the brief), sections cited as `[§x.y]`, and the user's Stage 1 corrections, cited as `[User 2026-11-25]` (see `audit.md` A-006 to A-012). Anything not in either source is marked **TBD – needs confirmation**.

---

## 1. What exists

- **Organization:** Soluix, an IT solution provider whose clients include banks, e.g. BCAS. [§1.1]
- **Team:** "The Investor's Team", 11 people. [§1.1]

| Team role | People |
|---|---|
| Project Advisor | Yosia Alvin |
| Project Managers | Ingwen Tannaris, Laurence Benedicta |
| System Architect | William Tanardi |
| Business Analysts | Ayuningtyas, Bernhard Praditya |
| Developers | Albert Rafer, Daniel Riyanto, Michael Christian |
| QA | Bimas Nugraha |
| DevOps | Hadi Septian |

- **Existing log sources:** Grafana/Loki, a database, and Kubernetes pod logs (read via k9s). Today engineers open these to pull logs. [§1.1, §1.4] Grafana is **self-hosted** and only **Developers and DevOps** use it. [User 2026-11-25] Details are in `system-context.md`.
- **Product to be built:** a greenfield, internal, self-serve web app in front of the log sources that already exist. It is **not** a new observability platform. [§1.1]

Note: these are the team members who run the project. Product user roles (personas) are defined later in Stage 3 Specify. [§3 Stage 3]

## 2. Who is served

| Group | How they are served | Source |
|---|---|---|
| BA, QA and other non-technical teammates | Find and read logs themselves instead of asking an engineer | §1.1 |
| Engineers (who can open Grafana, the database or k9s; for Grafana this is Developers and DevOps only) | Fewer interruptions; step in only when a lookup cannot be resolved | §1.1; User 2026-11-25 |
| Project Managers | Read pilot metrics (persona confirmed in Stage 3) | §1.1, §3 Stage 3 |
| Banking clients (e.g. BCAS) | Indirect: faster issue analysis on their projects; their data must stay protected | §1.1, §1.4 |

Environments served: **UAT and RC only**; production and any other environment are out of MVP scope. [User 2026-11-25]
Which projects and teams beyond the pilot are served: **TBD – needs confirmation** (raised in Stage 2).

## 3. Why it matters: the validated problem

### 3.1 Symptom [§1.1]
- BA, QA and other non-technical teammates keep pinging engineers to pull logs.
- 24 of 24 sampled chat captures are the same ask. Only the time, the identifier and the feature change.
- Each ping interrupts an engineer who can open Grafana, the database or k9s.
- Raw logs are dense JSON that is hard to read.

### 3.2 Root cause (5 Whys, as stated by the team) [§1.1]
- There is no standard, self-serve path from a non-technical question to a readable log.
- Log lookup is treated as a favor, not as a capability with an owner.
- Access and query skill sit with a few people.

The intermediate "why" steps of the 5 Whys are not in the brief; they are in the team decks, which are not in the workspace. **TBD – needs confirmation.**

### 3.3 Impact (estimates, **to be baselined**) [§1.1]
| Metric | Estimate | Status |
|---|---|---|
| Log-request pings in high-load periods (UAT/regression) | more than 10 per day | Estimate, to be baselined |
| Engineer time per lookup (excluding troubleshooting) | 5–10 minutes | Estimate, to be baselined |

No cost-per-hour, team-wide totals or other derived numbers are claimed here.

## 4. Expected behavior [§1.1]

A non-technical teammate:
1. finds the log themselves by **time, identifier, service or error code**;
2. sees a **simplified view**, with the **raw log one click away**;
3. engineers step in **only when the lookup cannot be resolved**.

## 5. Pilot hypothesis and decision rule [§1.1]

**Hypothesis:** if one real project team uses the finder for everyday lookups, engineer log-request pings fall by **65%** and requesters get a readable log in **under 2 minutes**.

**Pilot setup:**
- One real team (not a demo). Which team and project: **TBD – needs confirmation**.
- Baseline from the last 3 months of log-request chats. How pings are counted: **TBD – needs confirmation**.
- Behavior measurement: did they search themselves first; did they use filters or natural language.

**Decision after the pilot:**
| Outcome | Decision |
|---|---|
| Target met | Roll out |
| Partly met | Adjust and rerun |
| No change | Revisit the root cause |

The thresholds that separate "partly met" from "no change" are not defined in the brief: **TBD – needs confirmation**.

## 6. Decisions already made by the team [§1.2]

These are recorded as team decisions and are not reopened without strong reason.

| # | Decision | Status |
|---|---|---|
| D-T1 | Mechanism: Simplified Log Finder; manual filters first, natural-language search on top | Decided by team |
| D-T2 | Rejected: giving BA/QA direct Grafana / database / k9s access (security risk; logs stay unreadable) | Decided by team |
| D-T3 | Support only (not the main solution): runbooks/training; shared dashboards and saved queries | Decided by team |

## 7. Proposed direction awaiting approval [§1.3]

- **Hybrid "build vs reuse":** the finder is a thin orchestration layer that calls the existing Loki HTTP API and builds only what is missing.
- Status: **Proposed, needs approval.** Stage 2 Discover will challenge it with options (reuse as-is, hybrid, replace stack, build from scratch).

## 8. Constraints [§1.4]

| # | Constraint | Status |
|---|---|---|
| C1 | Security and compliance come first (banking clients). Logs may contain CIF, account numbers, hashcards, tokens and document content. | Given |
| C2 | The LLM translates a sentence into structured filters only; the finder then executes predefined query templates. Free-form generated LogQL/SQL is not allowed. | **Proposed; needs confirmation** |
| C3 | Multi-project from day one in architecture; only the pilot project is onboarded first. | **Proposed; needs confirmation** |
| C4 | Must NOT replace Grafana, the database or k9s for engineers. | Given |
| C5 | Must NOT become a backdoor with broader raw access than intended. | Given |
| C6 | Environments in scope: UAT and RC only; production excluded from MVP. | User-confirmed 2026-11-25 |
| C7 | Hosting: "local first". Exact meaning (laptop, internal server, inside UAT/RC network) is open. | User-confirmed direction 2026-11-25; meaning TBD |

## 9. Scope [§1.4]

| MVP | Next phase (design must not block it) |
|---|---|
| Manual filters | Reading images |
| Natural-language search on top | More input sources |
| Formatted logs | |
| Three sources: Grafana/Loki, database, Kubernetes pod logs | |
| Environments: UAT and RC only (production out of MVP) [User 2026-11-25] | |
| Applied across projects (see C3: pilot project onboarded first, pending confirmation) | |

Note a tension to resolve in Stage 2: §1.4 says the MVP is "applied across projects", while §1.1 and C3 describe a one-team pilot. §4 of the brief lists this as a mandatory clarification.

## 10. Request patterns seen in the 24 samples [§2]

Kept here as business evidence only; they become acceptance-criteria seeds in Stage 3. Requests combine:
- (a) an exact time or window;
- (b) one or more identifiers (CIF, account number, hashcard, request ID, program ID, license ID, endpoint path);
- (c) a business feature name, often in Indonesian (e.g. "ganti rek sumber", "penutupan kartu", "buka blokir", "saldo");
- (d) an environment (UAT, RC...), sometimes implicit.

The brief lists 10 representative requests out of the 24. The other 14 are not in the workspace: **TBD – needs confirmation**.

---

## 11. Stage 1 self-check [§0.2]

| # | Check | Result | Evidence |
|---|---|---|---|
| 1 | Every acceptance criterion is testable | **N/A at Stage 1** | No acceptance criteria are written yet (Stage 3). |
| 2 | Every rule traces to a source | **Pass** | Every section and table row cites a `[§x.y]` of the brief. Items without a source are marked "TBD – needs confirmation" (5-Whys steps, pilot team, ping counting, decision thresholds, the 14 unseen requests). |
| 3 | Design respects stack, security and scope limits | **N/A at Stage 1** (no design). Constraints C1–C5 and the MVP/next-phase split are captured for Stage 4. | §8, §9 of this file. |
| 4 | Reuses existing systems, roles and data where possible | **Pass (context level)** | Existing sources (Grafana/Loki, database, k9s) and existing team roles are recorded; the reuse proposal is captured as "Proposed, needs approval" (§7). No new systems introduced. |
| 5 | Evidence of the check is shown | **Pass** | This table; source citations throughout; `stage1-unverified.md` lists what was not verified. |
| 6 | Open questions flagged, not silently decided | **Pass** | C2, C3 and the hybrid direction are marked Proposed; scope tension in §9 is flagged; DevOps assumptions are left for Stage 2; full list in `stage1-unverified.md`. |
