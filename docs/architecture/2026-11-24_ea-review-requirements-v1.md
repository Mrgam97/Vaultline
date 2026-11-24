# Vaultline – Enterprise Architect Review of Requirements v1

> **Fictional company for training purposes.** All data is simulated.

| | |
|---|---|
| **Reviewed** | `docs/requirements/2026-11-24_vaultline-requirements_v1.md` |
| **Reviewer** | Enterprise architect (TOGAF ADM: Requirements Management, Phase A and inputs to B–D) |
| **Tailoring** | Small product: principles and vision on one page, findings as a list, no diagrams yet |
| **Date** | 24 November 2026 |
| **Verdict** | **Not ready for design.** 6 high findings must be resolved first; the rest can be resolved in parallel with design |

---

## 1. What's good

- Every row has an ID, priority and status, and stories trace to personas and FRs. This is a solid requirements baseline.
- The company-ending risk (D-05) runs through FR, NFR and INF rather than being one line.
- Infrastructure needs describe needs, not products, leaving design choices open.
- Edge cases (cancelled policies, time zones, last admin) are already listed.

---

## 2. Findings

**Severity:** High = blocks design or creates legal or company-ending risk · Medium = must be resolved before the backlog · Low = tidy up.

| ID | Severity | Refs | Challenge | Recommendation | Decides |
|---|---|---|---|---|---|
| EA-01 | **High** | D-08 | **There is no baseline.** D-08 says Build 1 exists, but its code isn't in the repository. I can't describe what exists, so any target design risks contradicting what's built. | Push Build 1 to the repository. The architect records the baseline before the target. | User |
| EA-02 | **High** | FR-08 – FR-14, NFR-09 | **Who is the source of truth for policy data?** Certificates are generated from "the policy record", but that record is typed in by a broker (AS-08), not received from the insurer. NFR-09 only proves the certificate matches *our* record, not the real policy. A keying error becomes a wrong certificate issued automatically. | Add a requirement: a policy is only usable for certificates once a broker has **verified** it against the insurer's declarations page (a verified flag, who verified it and when). Restate NFR-09 to cover verified data. | Product owner |
| EA-03 | **High** | FR-11, AS-09 | **Who signs an automatic certificate?** An industry-standard certificate carries an authorised representative of the broker. Issuing automatically means applying a licensed person's authorisation with no human in the loop. This is a business and legal decision, not a technical one. | New assumption: which licensed producer's authorisation appears on auto-issued certificates, under what written authority, and with what audit. Without a yes, FR-11 can't be Must. | User / Founder |
| EA-04 | **High** | NFR-25, INF-03 – INF-09 | **The cost ceiling likely clashes with the requirements.** A managed relational database with row protection, private networking, per-client keys, malware scanning and an AI service together are likely to exceed $20 a month even when idle, because several of these carry fixed monthly charges. | Either raise the development ceiling, or allow development-only compromises (for example a local database container and fewer always-on components) while keeping the isolation design identical. The architect will cost the options in an ADR. | User |
| EA-05 | **High** | NFR-02, INF-09, INF-16, AS-07 | **One requirement prescribes the solution and conflicts with scale.** NFR-02 says "separate AI knowledge per client". At about 500 clients (AS-07), one AI knowledge store per client may hit platform limits and cost. | Turn the separation rule into an **architecture principle** (outcome: no cross-client retrieval, provable by test). The *how* (per-client stores or a shared store with enforced client filters) becomes ADR-001, decided with secure design. | Architect + secure design |
| EA-06 | **High** | P-05, NFR-01, NFR-02 | **Staff cross the client boundary on purpose.** The Compliance Officer reads all clients, and brokers and claims handlers read several. NFR-01 and NFR-02 are written only for clients. If the data-layer lock is designed only for clients, staff access either breaks it or bypasses it. | Add an NFR: staff access to more than one client goes through a separate, audited path with its own role checks. Name the staff "break-glass" rules. Hand to secure design. | Product owner + secure design |
| EA-07 | Medium | FR-08, FR-11, FR-14, FR-21 | **Policy lifecycle isn't defined.** Certificates, reminders and flags depend on states (draft, verified, active, endorsed, cancelled, expired, renewed) that aren't listed. | Add a policy lifecycle requirement: states, allowed transitions, who can make each change. | Product owner |
| EA-08 | Medium | FR-27, US-13, P-04 | **Assignment rules are missing.** How are brokers and claims handlers assigned to clients? One broker per client, a team, cover during leave? The edge case "broker leaves" has no rule. | Add FRs for assigning, reassigning and covering clients, and for assigning claims. | Product owner |
| EA-09 | Medium | — | **There's no data model.** Requirements don't name the key entities, so isolation and audit can't be traced to data. | Agree the entity list: client, user, policy, coverage line, certificate holder, certificate request, certificate, document, claim, renewal, AI conversation, audit event. Each is owned by exactly one client, except staff users and audit events (owned by Vaultline). The architect then produces the data model and classification. | Product owner confirms, architect models |
| EA-10 | Medium | P-08, FR-10, US-04 | **Certificate delivery to the holder is undefined.** P-08 receives certificates, but no FR says how. If the system emails holders, that's a new external data flow, a new trust boundary and third-party personal data. | Decide: the client downloads and forwards, or the system sends to the holder. If the system sends, add an FR and an NFR (link not attachment, expiry). | User |
| EA-11 | Medium | FR-12 | **"Changes cover" needs explicit rules.** Whether a request auto-issues or goes to a broker must be a fixed, testable rule list, not a judgement (and never decided by the AI assistant). | List the triggers (additional insured, waiver of subrogation, primary and non-contributory, project-specific wording, any limit or date change). Everything else auto-issues. | Product owner |
| EA-12 | Medium | — | **Client lifecycle is missing.** There's no requirement for onboarding a client (only INF-16) or offboarding one: export, access removal, retention, deletion. | Add FRs for client onboarding and offboarding, and reconcile offboarding with AS-10 retention. | Product owner |
| EA-13 | Medium | FR-17, FR-30, NFR-13 | **AI transcripts are client data.** They contain the client's questions and quoted clauses but aren't covered by the separation rules, and their retention (2 years) differs from the records they quote (7 years). | State that transcripts are per-client data under NFR-01 and NFR-02. Confirm their retention. Hand to secure design. | Product owner + secure design |
| EA-14 | Medium | FR-14, US-06, out of scope | **Holder notification on cancellation is unclear.** US-06 wants holders "not misled", but holders have no account and the FR only notifies the client and broker. | Decide whether the system tells holders when cover is cancelled (often a policy or contract obligation), or state that it's out of scope. | User |
| EA-15 | Medium | NFR-03 | **"US business hours" is ambiguous.** Contractors request certificates early in the morning and across time zones. | Make it 99.5% around the clock, or name the window (for example 06:00–20:00 Eastern). | User |
| EA-16 | Low | AS-07 | **Volumes are missing for the key event.** Scale is given in clients and users but not certificates per month, which drives the certificate service and queue. | Add an estimate (for example about 20 certificates per contractor client per month). | Product owner |
| EA-17 | Low | FR-21, INF-08 | **Notifications may leak content.** Emails could include policy details. | Notifications contain no policy, claim or financial details, only a link that requires sign-in. Hand to secure design. | Secure design |
| EA-18 | Low | NFR-18 | **Policy dates have a convention.** US policies usually start and end at 12:01 a.m. standard time at the insured's address. | Use that rule, not just "local time zone". | Product owner |
| EA-19 | Low | All | **Traceability ends at the requirements.** NFRs don't link to stories and nothing links to tests yet. | Add a test reference column when the backlog is drafted. | Product owner |

---

## 3. Proposed architecture principles (for agreement)

| ID | Principle | Rationale | Implication |
|---|---|---|---|
| AP-01 | **Client isolation by design, in at least two layers** | Company-ending risk (D-05) | Every store, index, key and query is scoped to one client; one layer failing doesn't leak data |
| AP-02 | **Identity is the only source of client context** | Tampering is the easiest attack | No client ID from request bodies, URLs or AI prompts is trusted |
| AP-03 | **Verified policy record is the single source of truth for certificates** | EA-02 | Certificates and AI answers never invent or override policy data |
| AP-04 | **AI assists; licensed humans decide on cover** | D-04 | The AI assistant has no authority to issue, approve or judge |
| AP-05 | **Managed and scale-to-zero first, within the cost ceiling** | NFR-25 | Prefer services with no idle cost; justify any always-on component in an ADR |
| AP-06 | **Everything as code, everything audited** | NFR-11, NFR-27 | No manual environment changes; every sensitive action leaves an audit event |
| AP-07 | **US data residency** | AS-13 | All stores and processing in US regions |
| AP-08 | **Simulated data only** | D-06, NFR-28 | No real personal or policy data in any environment |

---

## 4. Decisions the architect will take later (ADRs)

| ADR | Question | Depends on |
|---|---|---|
| ADR-001 | Per-client AI knowledge stores, or a shared store with enforced client filters | EA-05, secure design |
| ADR-002 | Data-layer isolation method for structured data | AP-01, EA-06 |
| ADR-003 | Certificate generation and signing approach | EA-02, EA-03 |
| ADR-004 | Development environment shape within the cost ceiling | EA-04 |
| ADR-005 | Staff cross-client access path | EA-06 |

---

## 5. Questions for the user

1. **Baseline (EA-01):** can you push Build 1's code to the repository so the architect can record what exists? *Recommended: yes, before any design.*
2. **Signing (EA-03):** may auto-issued certificates carry a licensed producer's authorisation, as long as the policy was broker-verified (EA-02) and every issue is audited? *Recommended: yes, with both conditions.*
3. **Cost (EA-04):** raise the development cost ceiling, or keep $20 with development-only compromises? *Recommended: keep $20 with compromises; the isolation design stays identical.*
4. **Holders (EA-10, EA-14):** does the system send certificates to holders and tell them about cancellations in release one? *Recommended: the client downloads and forwards in release one; holder notification is out of scope and stated as such.*
5. **Availability (EA-15):** around the clock or a named window? *Recommended: 99.5% around the clock.*

---

## 6. Handovers

- **Product owner:** EA-02, EA-06 – EA-13, EA-16, EA-18, EA-19, to update requirements to v2.
- **Secure design:** EA-05, EA-06, EA-13 and EA-17, plus principles AP-01, AP-02 and AP-04, for challenge.
- **User:** the five questions in section 5.
