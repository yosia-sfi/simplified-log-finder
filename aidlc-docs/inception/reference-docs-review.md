# Stage 1 Reference Documents Review

AI-DLC Inception, supports Stage 1 (Context) and Stage 2 (Discover). Date: 2026-11-25. Status: **Draft, for user review**.
Sources: three files committed at the repo root (commit `ac124f7`). Citations use `[Ideation p.N]`, `[Presenter p.N]`, `[Workshop]`. The brief is `prompt-inception.md` `[§x.y]`.

> **Placeholder rule:** this file never contains real identifiers. CIFs, account numbers, hashcards, phone numbers, request IDs, bucket names, document numbers and document content are written as `<CIF>`, `<ACCOUNT_NO>`, `<SHA256_HASH>`, `<PHONE>`, `<REQUEST_ID>`, `<BUCKET>`, `<DOC_NO>`, `<BASE64_DOC>`. People are referred to by role.

All facts below come from reading the files, including screenshots. Facts about logs, labels and infrastructure are **observations from screenshots, to be re-verified in Stage 4**; they are not verified system behavior.

---

## 1. What each file contains

| File | Size | Contents |
|---|---|---|
| `Ideation - Simple Log Finder.pdf` | 19 pages | The idea sheet. Problem and context; the 24 sampled chat requests as screenshots (p.4–11, 3 per page); sources of logs (p.12, all three marked "Technical"); must / must-not list (p.14); simplified vs raw view (p.16); a Grafana Explore screenshot of real UAT logs (p.17) |
| `Presenter's Deck - Investor's Team - Simplified Log Finder.pdf` | 11 pages | Cover "Simplified Log Finder, Built with AIDLC, AIDLC – The Investor's Team" with a UI mockup (p.1); MVP scope (p.2); the same chat captures (p.4–5); 5-Whys with intermediate steps (p.6); time target (p.7); option ratings (p.9); pilot plan (p.10) |
| `Reference AI DLC Workshop v3.pptx` | 32 slides | The AI-DLC workshop deck: staged flow, deliverables, principles (including Bounded Autonomy) and the six retrospective metrics |

---

## 2. Coverage of `stage1-unverified.md` section A

| # | Item | Status | Evidence | What is still missing |
|---|---|---|---|---|
| 1 | Idea sheet | **Resolved** | Ideation PDF | — |
| 2 | "Investor's Team – Simplified Log Finder (AIDLC)" deck | **Unclear** | The Presenter's Deck cover reads "Simplified Log Finder, Built with AIDLC, AIDLC – The Investor's Team" [Presenter p.1]. It may be the same deck | Confirmation from the PM (question `SRC-02`) |
| 3 | Presenter deck | **Resolved** | Presenter PDF | — |
| 4 | AI-DLC workshop deck | **Resolved** | Workshop PPTX matches the brief's flow, deliverables, principles (incl. Bounded Autonomy) and six retrospective metrics. Wording differs on two metrics: the deck defines Speed as "brief to first working demo" and Human effort as "share of code humans wrote"; the brief [§5] adapts both for Inception. **The brief governs** | — |
| 5 | Real Grafana/Loki UAT log sample | **Partly resolved** | One Grafana Explore screenshot [Ideation p.17] | A masked **text** export of a few raw lines (parser test fixtures) and the full list of common labels (3 are hidden behind "+3") |
| 6 | Full 24 sampled chat requests | **Resolved** | [Ideation p.4–11]; same captures in [Presenter p.4–5] | — |
| 7 | 5-Whys intermediate steps | **Resolved** | [Presenter p.6], summarized in §5 below | — |

---

## 3. Findings from the log screenshot [Ideation p.17]

All to be re-verified in Stage 4.

| # | Observation | Effect on later stages |
|---|---|---|
| B1 | Grafana Explore with a data source named `loki`. Consistent with self-hosted Grafana (A-007) | None new |
| B2 | Common labels visible: `cluster=xip-development`, `job=fluentbit`, `kubernetes_container_name=ces-d…`, plus 3 more not visible ("+3") | Full label list needed for templates and source bindings |
| B3 | Payload `project_name` reads `bcas-uat-1b` while the `cluster` label reads `xip-development` | An environment may not map to a single label. Environment-to-label mapping must be configured per project (question `M-08`, assumption AS-10) |
| B4 | Line format: `{"log":"<UTC ts>Z stdout F <ts>+0700 \t <ANSI-colored level INFO/DEBUG> \t <source file/function> \t <message> \t {<escaped JSON: project_name, project_module, module_version, correlation_id, log_data>}"}`. Grafana shows local time; the Loki line carries UTC; the app timestamp carries +0700 | Parser must strip the CRI prefix and ANSI codes and unwrap escaped JSON; timezone rules (`M-05`) |
| B5 | `correlation_id` is empty in every visible line | Confirms `M-07` concern |
| B6 | Some `log_data` payloads contain Header fields `X-Request-Id` and `traceparent` | Possible linking key when `correlation_id` is empty (question `M-09`, assumption AS-11) |
| B7 | Sensitive content visible: base64 document body (`BodyRaw` = `<BASE64_DOC>`), document numbers (`<DOC_NO>`), document template IDs, S3 bucket and object keys (`<BUCKET>`), ETag | Adds to the sensitive field list for `SEC-02` |
| B8 | Messages are partly in Indonesian | Dictionary and simplified view must handle Indonesian text |
| B9 | Services seen: a document service and a RabbitMQ consumer, written in Go | Parser fixtures should cover Go log formats from these services |

---

## 4. Findings from the 24 chat captures [Ideation p.4–11]

| # | Observation | Effect on later stages |
|---|---|---|
| C1 | Requesters include BAs, QAs, an FE developer and a Mobile developer, not only BA/QA. Engineers pinged include the System Architect and backend developers | Persona scope (question `U-01`) |
| C2 | Environments named in requests: RC and UAT | Consistent with A-010 |
| C3 | By the orchestrator's count, 12 of 24 requests give no identifier and no time (e.g. "minta log lagi", "log untuk trx ini", "minta lognya waktu refund"), relying on an earlier photo or message | Handling of under-specified requests (question `M-10`) |
| C4 | Several requests ask for the log in order to forward it to the client (BCA/BCAS) | "Copy raw" masking is client-facing (question `SEC-05`, linked to `SEC-03`) |
| C5 | Features/terms beyond the brief: ganti sof (source of funds), CWLD / `external_opening_account`, closed amount, laporan kartu debit (filter defect), export excel, `license_id`, liveness / get token, Verihubs generate-key endpoint (`/api/registration/ibridge/v1/verihubs/encryption/generate-key`), H2H connection timeout, 403 on a URL path | Input to the business-term dictionary (`M-03`) and identifier list (`M-01`) |
| C6 | One request asks for an example endpoint response, not a log | Scope question `O-04` |
| C7 | Identifier types seen: CIF (`<CIF>`, several per request), norek (`<ACCOUNT_NO>`), hashcard (`<SHA256_HASH>`, SHA-256 hex), program ID, `license_id`, endpoint path | Confirms and extends `M-01` |
| C8 | The brief's 10 representative requests [§2] match these captures | Stage 3 can trace to all 24 requests |

---

## 5. Findings from the idea sheet and presenter deck

| # | Observation | Source |
|---|---|---|
| D1 | All three log sources (Grafana, database, k9s) are marked "Technical" only | [Ideation p.12] |
| D2 | MVP "Applied to all projects — one finder reused by every project team" vs one team pilots. **Confirms contradiction X-001** (still open, `S-01`) | [Presenter p.2], [Presenter p.10] |
| D3 | UI mockup: editable chips ("Yesterday", "VKYC", "CIF ●●●●●", "Failed"), SIMPLIFIED/RAW toggle, "Copy simplified" and "Copy raw" buttons | [Presenter p.1] |
| D4 | "Must": a clear result or a clear "nothing found". "Must not": require a ticket or chat ping for common cases | [Ideation p.14] |
| D5 | Raw side is also pretty-printed without dropping fields; raw is used to "forward to BCA, paste into a ticket, debug" | [Ideation p.16] |
| D6 | Time for a requester to get a log: "no longer than 2 minutes" (matches the brief [§1.1]) | [Presenter p.7] |
| D7 | Option ratings are team assessments "to be confirmed in design" | [Presenter p.9] |
| D8 | 5-Whys: (1) requesters know what they need (time, CIF, error) but cannot fetch it; (2) logs live in Grafana, the database and k9s, each needing technical skill and access; (3) tools are built for engineers, and nothing turns time + CIF + feature into a query; (4) lookup is treated as a favor, not a capability with an owner. Root cause: no standard self-serve path from a non-technical question to a readable log | [Presenter p.6] |

No new contradictions with the brief or with the user's corrections were found. D2 confirms the existing X-001. The workshop-deck metric wording (item 4) differs from the brief, and the brief governs.

---

## 6. Reference documents still needed

| # | Document | Needed for | Owner | When |
|---|---|---|---|---|
| N1 | Masked **text** export of a few raw Loki log lines (identifiers replaced by placeholders), plus the full list of common labels | Parser test fixtures; templates; label mapping (`M-08`) | DevOps or a Developer | Before Stage 3 approval preferred; Stage 4 at the latest |
| N2 | Confirmation that the Presenter's Deck is the "Investor's Team – Simplified Log Finder (AIDLC)" deck | Closing item 2 | PM | Can-defer (`SRC-02`) |
| N3 | 3-month chat export for the ping baseline | Pilot baseline (`MET-01`) | PM | Later, before pilot start; not needed for Inception |
| N4 | Database schema / table list for the pilot project | Database templates (`D-01`) | DevOps / System Architect | Stage 4 |

---

## 7. Sensitive-data risk in the committed files

**Risk RK-001 (flagged 2026-11-25, decision pending user):** the committed PDFs contain what appear to be real CIFs, account numbers, hashcards, a phone number, request IDs, S3 bucket names and base64 document content (in chat and log screenshots). They are now in git history at commit `ac124f7`.

- If the repository is pushed to a shared or external remote, these values travel with it.
- Replacing the files with masked copies does not remove them from history; only a history rewrite (and re-cloning on every laptop) does.
- **No action taken.** The decision belongs to the user (question `SEC-06`, owners PM and System Architect).
- None of these values are copied into `aidlc-docs/`.
