# Vaultline – Requirements v3 (consolidated)

> **Fictional company for training purposes.** Vaultline, Kestrel Groundworks LLC and Saffron Lane Hospitality Group LLC are not real. All data is simulated.

| | |
|---|---|
| **Version** | v3, consolidated: product owner requirements v2 + enterprise architect review + secure design review v1. Supersedes v1 and v2 |
| **Sources** | `docs/architecture/2026-11-24_ea-review-requirements-v1.md` · `docs/security/2026-11-24_vaultline-security-review_v1.md` |
| **Date** | 24 November 2026 |
| **Owner** | Product owner |
| **Scope** | Release one of the Vaultline client portal, admin portal and AI assistant |
| **Sign-off** | **Not signed off.** Product owner: draft · Enterprise architect: reviewed · Secure design: **not signed off** (see section 12) |
| **Language** | British English. Insurance terms follow US usage because the product serves US businesses (certificate of insurance, deductible, general liability). |

**Status key:** **Agreed** = follows a user decision (D-xx) · **Draft** = proposed, needs the user's OK · **Open (AS-xx)** = depends on an open assumption.
**Priority:** MoSCoW for release one. M = Must, S = Should, C = Could, W = Won't.

---

## How to use this document

**For builders and coding agents (including Claude Code):**
- This is the single source of requirements. Every item has an ID; quote the ID in branches, commits, pull requests and tests.
- **Only build items whose status is Agreed *and* that appear in an approved backlog ticket.** Draft and Open items are not ready; don't build them or fill the gaps with assumptions.
- **Security requirements (SR-xx) are mandatory** for every item they apply to. The release gates (RG-xx) must pass before release.
- The company-ending failure is one client seeing another client's data (D-05). If an item seems to conflict with that, stop and ask.
- Products are chosen by the architect in ADRs within constraint D-07 (AWS); requirements state needs, not products.

**Status key:** **Agreed** = follows a user decision (D-xx) · **Draft** = proposed, needs the user's OK · **Open (AS-xx)** = depends on an open assumption.

## 0. Inputs checked

| Input | Status |
|---|---|
| Company pitch (Richmond, Virginia; problem; app; AI assistant; biggest risk) | Read |
| Business model and pricing | **Unchecked:** Founder pages not written. See AS-06 |
| Competitors | **Unchecked:** not written |
| Risk appetite | **Partly checked:** company-ending failure is stated in the pitch (one client seeing another's data); the rest is not written |
| Success metric | **Unchecked:** see AS-01 |
| Organisation and roles | Read (Confluence *Team and roles*) |
| Jira backlog KAN (8 epics, 25 tasks) | Read. Mapped later, after secure design sign-off |
| Requirements v0.2, architecture page (HLD v0.2, AWS) | Read |
| Enterprise architect review of v1 (EA-01 to EA-19, AP-01 to AP-08) | Read and applied (section 10) |
| Secure design review v1 (SD-01 to SD-23, SR-01 to SR-32, AC-01 to AC-18, RG-01 to RG-07) | Read and applied (sections 7a, 12, 13) |
| **Baseline: Build 1 code** | **Unchecked:** not in the repository (EA-01). Requirements marked "already built" are unverified until it's pushed |

---

## 1. Decisions log

| ID | Decision | Source |
|---|---|---|
| D-01 | Vaultline is a commercial insurance broker in Richmond, Virginia, serving owners and finance managers of small and mid-sized businesses | Pitch |
| D-02 | Clients see policies, certificates, renewal dates and claims in one secure portal; they can download certificates, upload documents and report claims | Pitch |
| D-03 | Staff use a separate admin portal | Pitch |
| D-04 | An AI assistant answers each client's questions about their own cover by quoting the exact clause from their own documents, and hands anything about whether a claim will be paid to a human broker | Pitch |
| D-05 | The company-ending failure is one client seeing another client's data, including through the AI assistant or via a hidden instruction in an uploaded document | Pitch |
| D-06 | The two simulated clients are Kestrel Groundworks LLC (construction) and Saffron Lane Hospitality Group LLC (restaurants) | User, 22 Nov |
| D-07 | Cloud platform is AWS (a constraint for the architect, not a product choice made here) | User, 22 Nov |
| D-08 | Roles already built: Client Admin, Account Broker, Claims Handler, Compliance Officer | Build 1 |
| D-09 | Requirements follow the TOGAF-aligned flow: product owner → enterprise architect → secure design → backlog → build | User, 24 Nov |

---

## 2. Assumptions log

| ID | Assumption | Why it matters | Status |
|---|---|---|---|
| AS-01 | Success metric: % of certificate of insurance requests clients complete in the portal without a broker; target 70% within 3 months of a client going live | Sets priorities for every row | Open |
| AS-02 | Top three customer jobs: (1) get a certificate fast, (2) know what I'm covered for, (3) never miss a renewal | Defines release-one scope | Open |
| AS-03 | A **Client User** role is needed (site or restaurant managers) | The people who need certificates are often not the Client Admin | Open |
| AS-04 | A **Platform Admin** role exists to manage staff accounts and cannot read client documents | Separation of duties | Open |
| AS-05 | The AI assistant is treated as a **system actor** with its own limits | Its permissions must be designed, not inherited | Open |
| AS-06 | Vaultline earns commission from insurers; the portal is free to clients | Rules out paid tiers in release one | Open |
| AS-07 | Scale: 2 clients now; about 50 clients and 500 users in year one; about 500 clients and 5,000 users by year two | Sets scalability targets | Open |
| AS-08 | Brokers load policies at launch (documents plus key data); automatic insurer feeds come later | Defines how data gets in | Open |
| AS-09 | Certificates name a certificate holder on an industry-standard form (ACORD 25 style); requests that only name a holder are issued automatically; requests that change cover go to the broker | Core of AS-01 | Open |
| AS-10 | Retention: policy and claim records 7 years after expiry; AI assistant transcripts 2 years | Retention and deletion NFRs | Open |
| AS-11 | Cost ceiling: no more than $20 a month for the development environment | Constrains design choices | Open |
| AS-12 | Claims reporting is in release one but is not a top-three job (priority Should) | Changes the build order from Build 1's "next step" | Open |
| AS-13 | All data stays in the United States | Residency NFR | Open |
| AS-14 | Auto-issued certificates carry the authorisation of a named licensed producer at Vaultline, under a recorded written authority, only for broker-verified policies, with every issue audited | A licensed person's authorisation is applied with no human in the loop (EA-03) | Open |
| AS-15 | The development cost ceiling stays at $20 a month; development may use cheaper stand-ins (for example a local database) **only** if the isolation design is identical to production | Cost likely exceeds $20 otherwise (EA-04) | Open |
| AS-16 | In release one the client downloads the certificate and forwards it to the holder; the system doesn't email holders or tell them about cancellations | Avoids a new external data flow and third-party personal data (EA-10, EA-14) | Open |
| AS-17 | Availability target applies around the clock, not only business hours | Contractors request certificates early and across time zones (EA-15) | Open |
| AS-18 | About 20 certificate requests per contractor client per month; about 10,000 a month at year two | Sizes the certificate service and queues (EA-16) | Open |
| AS-19 | Each client has one primary Account Broker and one named cover broker; the Platform Admin assigns them, seeing client names only | Assignment rules were missing (EA-08) | Open |
| AS-20 | Each claim is assigned to one Claims Handler; handlers see only their assigned claims | Claims scope was undefined (EA-08) | Open |
| AS-21 | Vaultline is subject to the state insurance data security law (written security programme, risk assessment, incident notification) and federal financial privacy notice rules | Drives SR-27 and SR-28 (SQ-01) | Open |
| AS-22 | Security verification uses OWASP ASVS Level 2 overall, with Level 3 depth for access control and data protection | Sets the depth of security testing (SQ-02) | Open |
| AS-23 | Four-eyes (two different people) for policy verification, staff account creation, role grants, broker assignment and exports | Separation of duties (SQ-03, SD-05, SD-09, SD-10) | Open |
| AS-24 | AI assistant transcripts are kept 1 year, not 2 | Privacy minimisation (SQ-04); changes AS-10 for transcripts | Open |
| AS-25 | Risk R-05 (holders relying on certificates after cancellation) is accepted for release one, with certificate wording stating it confers no rights | Only the user can accept a risk | Open |

---

## 2a. Architecture principles (proposed by the enterprise architect)

These guide every requirement and design choice. Status **Draft** until the user agrees them.

| ID | Principle | Status |
|---|---|---|
| AP-01 | Client isolation by design, in at least two independent layers | Draft |
| AP-02 | Identity is the only source of client context | Draft |
| AP-03 | The broker-verified policy record is the single source of truth for certificates | Draft |
| AP-04 | AI assists; licensed humans decide on cover | Draft (follows D-04) |
| AP-05 | Managed and scale-to-zero first, within the cost ceiling | Draft |
| AP-06 | Everything as code, everything audited | Draft |
| AP-07 | US data residency | Open (AS-13) |
| AP-08 | Simulated data only | Agreed (D-06) |

## 2b. Key data entities (EA-09)

For the architect to model. **Owner** says whose data it is for isolation purposes.

| Entity | Owner | Classification | Notes |
|---|---|---|---|
| Client (company) | Vaultline | Confidential | One row per client; the unit of isolation |
| Client user | Client | Personal | Client Admins and Client Users |
| Staff user | Vaultline | Personal | Brokers, claims handlers, compliance, platform admins |
| Broker assignment | Vaultline | Internal | Primary and cover broker per client (AS-19) |
| Policy | Client | Confidential, financial | Has a lifecycle state (FR-34) and a verified flag (FR-33) |
| Coverage line | Client | Confidential, financial | Limits, deductibles per policy |
| Certificate holder | Client | Personal (third party) | Name and address of the holder |
| Certificate request | Client | Confidential | Standard or needs approval (FR-12) |
| Certificate | Client | Confidential | Immutable; versions on correction (FR-13) |
| Document | Client | Confidential, may hold personal data | Policy wordings, uploads, evidence |
| Claim | Client | Confidential, personal, financial | Assigned to one claims handler (AS-20) |
| Renewal questionnaire | Client | Financial | Revenue, payroll, headcount |
| AI conversation | Client | Confidential | Transcripts are client data (NFR-01) |
| Audit event | Vaultline | Internal, tamper-evident | Records client and user for every sensitive action |

## 3. Personas

| ID | Persona | Goal | Pain today (and its cost) | Can see | Can't see |
|---|---|---|---|---|---|
| P-01 | **Client Admin**: owner or finance manager (for example Kestrel's finance manager) | Keep the business properly insured with minimal admin | Policies scattered across emails and PDFs; renewal paperwork chased by email; **a lapse in cover or a lost contract** | Own company's policies with premiums, certificates, claims, documents, users | Any other client's data |
| P-02 | **Client User**: site or restaurant manager (AS-03) | Get a certificate or check cover on the spot | Waits a day for a certificate; **can't start work on site, crew stood down** | Own company's policies without premiums; certificates | Premiums, financials, claims, user management, other clients |
| P-03 | **Account Broker**: Vaultline staff | Keep assigned clients covered and renewing | Same certificate and "am I covered?" emails every week; **hours lost, slower service** | Assigned clients only | Unassigned clients |
| P-04 | **Claims Handler**: Vaultline staff | Move claims to settlement quickly | Evidence arrives piecemeal by email; **claims stall** | Claims assigned to them (AS-20) | Other claims, client financials beyond the claim |
| P-05 | **Compliance Officer**: Vaultline staff | Prove the firm handles data properly | No single audit trail; **failed audit or regulator complaint** | Audit logs, AI transcripts (read-only), all clients read-only | Nothing editable |
| P-06 | **Platform Admin**: Vaultline IT (AS-04) | Keep staff access correct | Manual, error-prone access changes | Staff accounts and roles; client names for broker assignment (AS-19) | Client documents, claims, policies |
| P-07 | **AI assistant**: system actor (AS-05) | Answer a client's cover questions with citations | Not applicable | Only the signed-in client's documents | Any other client's documents; the right to judge whether a claim is covered |
| P-08 | **Certificate holder**: external party, for example a general contractor | Receive a valid certificate quickly | Delayed or wrong certificates hold up their site | The certificate the client forwards to them (AS-16) | Anything in the portal (no account, no system emails in release one) |

---

## 4. User stories

Every story meets INVEST. Each has at least one unhappy path (marked ✗).

### US-01 Sign in securely · P-01, P-02 · M · Draft
As a Client Admin or Client User, I want to sign in with a second factor so that only I can reach my company's insurance data.
- **Given** I have an active account, **when** I sign in with my password and second factor, **then** I see only my company's dashboard.
- ✗ **Given** I'm a client user, **when** I try to open the admin portal, **then** access is refused.
- ✗ **Given** my session has been idle past the limit, **when** I act, **then** I must sign in again.
- ✗ **Given** a sign-in from a new device succeeds, **when** I then try to invite a user, export data or change a role, **then** I must re-authenticate, and I'm notified of the new device (AC-01, SR-16).

### US-02 Invite and remove my staff · P-01 · M · Open (AS-03)
As a Client Admin, I want to invite and remove Client Users so that my site managers can help themselves and leavers lose access.
- **Given** I'm a Kestrel Client Admin, **when** I invite a site manager, **then** they receive a single-use invitation and get Client User access.
- **Given** a Client User leaves Kestrel, **when** I remove them, **then** their access ends immediately.
- ✗ **Given** I'm the last Client Admin, **when** I try to remove myself, **then** I'm stopped until another Client Admin exists.
- ✗ **Given** I'm a Kestrel Client Admin, **when** I try to add a user to another company or give a staff role, **then** it's refused and logged.
- ✗ **Given** I invite a Client User, **when** the invite is sent, **then** it's bound to that email, single-use, expires in 72 hours, and every other Client Admin is notified (AC-02, SR-20).

### US-03 See all my policies · P-01, P-02 · M · Draft
As a Client Admin, I want every policy in one place so that I know what we have and when it renews.
- **Given** Kestrel has five policies, **when** I open the dashboard, **then** I see type, insurer, limits, deductible, premium and renewal date for each.
- **Given** I'm a Client User, **when** I open the dashboard, **then** premiums are hidden.
- ✗ **Given** I'm signed in as Kestrel, **when** I change a policy ID in the address bar to a Saffron Lane policy, **then** I get "not found" and no data.

### US-04 Get a certificate naming a holder · P-02, P-01, P-08 · M · Open (AS-09)
As a Client User, I want a certificate naming the job's general contractor so that my crew can start today.
- **Given** an active, broker-verified general liability policy, **when** I request a certificate naming a holder with only a name and address, **then** it's ready to download within the target time, matches the verified policy exactly and carries the authorised producer's authorisation (AS-14).
- ✗ **Given** the policy hasn't been verified by a broker, **when** I request a certificate, **then** it goes to the broker instead of auto-issuing.
- **Given** I ask for the holder to be an additional insured, **when** I submit, **then** it goes to the broker and I'm told it's awaiting approval.
- ✗ **Given** the policy has expired or been cancelled, **when** I request a certificate, **then** none is issued and I'm told why.
- ✗ **Given** I edit limits or dates in the request, **when** it's processed, **then** my edits are ignored and only the policy record is used.
- ✗ **Given** someone alters a downloaded certificate, **when** a broker checks its reference against the register, **then** it fails verification (AC-04, SR-13).

### US-05 Approve certificate requests · P-03 · M · Open (AS-09)
As an Account Broker, I want a queue of non-standard certificate requests so that I can answer the same day.
- **Given** a request for additional insured status, **when** I approve it, **then** the certificate is issued, the client told and my approval logged.
- ✗ **Given** a request from an unassigned client, **when** I try to open or approve it, **then** it's refused and logged.

### US-06 Certificates stay correct after a policy changes · P-01, P-08 · S · Draft
As a Client Admin, I want to know when a certificate I issued no longer matches the policy so that holders aren't misled.
- **Given** a policy is cancelled or its limits change, **when** that happens, **then** certificates already issued on it are flagged and the broker and Client Admin are told.
- ✗ **Given** a certificate was issued, **when** anyone tries to edit it, **then** it can't be changed; a new version must be issued.

### US-07 Ask about my cover · P-01, P-02, P-07 · M · Draft
As a Client Admin, I want to ask a question about my cover and see the exact clause so that I can trust the answer.
- **Given** my policy wording is loaded, **when** I ask "what's my deductible for glass breakage?", **then** I get an answer citing the document and section.
- **Given** nothing relevant exists, **when** I ask, **then** the assistant says it can't find it rather than guessing.
- ✗ **Given** I ask "will my claim for the flood be paid?", **when** the assistant answers, **then** it doesn't judge cover and creates a task for my broker.
- ✗ **Given** I ask about another client, or my uploaded document contains a hidden instruction, **when** the assistant answers, **then** it reveals nothing about any other client and ignores the instruction.
- ✗ **Given** a document tries to make the answer include a link or image to an outside address, **when** the answer is shown, **then** it contains no active links or remote images (AC-07, SR-10).

### US-08 Never miss a renewal · P-01, P-03 · S · Draft
As a Client Admin, I want reminders before renewal and an online renewal questionnaire so that cover never lapses and I don't trade emails.
- **Given** a policy renews in 60 days, **when** that date arrives, **then** the Client Admin and broker are reminded, and again at 30 days.
- **Given** my broker sends the renewal questionnaire, **when** I complete it online, **then** the broker is notified.
- ✗ **Given** I'm a Client User, **when** I try to open the questionnaire, **then** access is refused (it holds financials).
- ✗ **Given** a policy is cancelled, **when** its renewal date nears, **then** no reminder is sent.

### US-09 Upload documents · P-01 · M · Draft
As a Client Admin, I want to upload documents so that my broker has what they need.
- **Given** a PDF under the size limit, **when** I upload it, **then** it's stored against my company only and available after a safety scan.
- ✗ **Given** an unsupported, oversized or infected file, **when** I upload it, **then** it's rejected or quarantined and never reaches staff or the AI assistant.
- ✗ **Given** a PDF with active content, **when** it's processed, **then** the active content is stripped or the file rejected before anyone opens or indexes it (AC-09, SR-21).

### US-10 Load a new client's policies · P-03 · M · Open (AS-08)
As an Account Broker, I want to load a client's policy documents and key data in one go so that they're ready the same day.
- **Given** I select Kestrel, **when** I upload documents and a data file, **then** the data is checked and saved to Kestrel only, and appears in the portal and AI assistant within the target time.
- ✗ **Given** the data file names a different client, **when** it's processed, **then** those rows are rejected and nothing is written to the other client.
- ✗ **Given** a cell in the data file starts with a formula, **when** it's imported or later exported, **then** the formula is neutralised (AC-10, SR-21).

### US-11 Report a claim · P-01, P-04 · S · Open (AS-12)
As a Client Admin, I want to report a claim with evidence so that it starts quickly.
- **Given** an active policy, **when** I submit date, type, description, estimated value and evidence, **then** I receive a claim reference and the Claims Handler is notified.
- ✗ **Given** I'm a Client User, **when** I try to report a claim, **then** access is refused.
- ✗ **Given** I choose another client's policy by changing its ID, **when** I submit, **then** it's refused.

### US-12 Track and update a claim · P-04, P-01 · S · Open (AS-12)
As a Claims Handler, I want to update a claim's status and ask for more information so that the client always knows where it stands.
- **Given** a claim is with the insurer, **when** I change its status, **then** the client sees it with the history.
- ✗ **Given** I'm a Client Admin, **when** I try to change a claim's status, **then** it's refused.

### US-13 Work only my book · P-03 · M · Draft
As an Account Broker, I want to see only my assigned clients, with upcoming renewals highlighted, so that I focus on the right work.
- **Given** I'm assigned Kestrel, **when** I open my list, **then** I see Kestrel and its renewals due within 60 days.
- ✗ **Given** Saffron Lane isn't assigned to me, **when** I open it by ID, **then** it's refused and logged.

### US-14 Manage staff access · P-06 · S · Open (AS-04)
As a Platform Admin, I want to create, disable and change staff roles so that access stays correct.
- **Given** a new broker joins, **when** I create their account, **then** they get the Account Broker role.
- ✗ **Given** I'm a Platform Admin, **when** I try to open a client document or change my own role, **then** it's refused and logged.
- ✗ **Given** I create a staff account or grant a role, **when** I submit, **then** it doesn't take effect until a second authorised person approves, and an alert fires (AC-13, SR-17, AS-23).

### US-15 Audit everything sensitive · P-05 · M · Draft
As a Compliance Officer, I want a tamper-evident record of sign-ins, document access, role changes, exports and AI assistant conversations so that I can prove good practice.
- **Given** any sensitive action, **when** it happens, **then** it's recorded with user, client, action and time.
- **Given** I filter by client and date, **when** I search, **then** I see matching events read-only.
- ✗ **Given** anyone, including a Platform Admin, **when** they try to edit or delete an audit entry, **then** it's refused.

### US-16 Prove isolation automatically · P-05 (on behalf of all clients) · M · Draft
As the Compliance Officer, I want automated tests that try to reach one client's data as the other so that a leak is caught before release.
- **Given** a change is proposed, **when** tests run, **then** every attempt to read Saffron Lane data as Kestrel (and the reverse) through the portal, documents, data and AI assistant fails.
- ✗ **Given** any isolation test fails, **when** someone tries to release, **then** the release is blocked.

### US-17 Verify a policy before it's used · P-03 · M · Draft
As an Account Broker, I want to verify a loaded policy against the insurer's declarations page so that certificates and answers are based on correct data.
- **Given** I've loaded Kestrel's general liability policy, **when** I check it against the declarations page and mark it verified, **then** it becomes usable for certificates and my name and the time are recorded.
- ✗ **Given** a verified policy's limits or dates are changed, **when** the change is saved, **then** it returns to unverified and certificates stop auto-issuing until re-verified.
- ✗ **Given** I'm the broker who keyed the policy, **when** I try to verify it, **then** it's refused; a different broker must verify (AC-16, SR-12, AS-23).

### US-18 Onboard and offboard a client · P-03, P-06 · M (onboard) / S (offboard) · Draft
As an Account Broker, I want to set up a new client in one step, and close a leaving client cleanly, so that clients start fast and leave without their data lingering in reach.
- **Given** a new client signs, **when** I create it, **then** its separated storage is prepared, its first Client Admin is invited and it's assigned to me as primary broker.
- **Given** a client leaves, **when** offboarding completes, **then** all its users lose access the same day, the Client Admin can request an export, and data is kept only for the retention period, then deleted.
- ✗ **Given** a client has been offboarded, **when** a former user tries to sign in or use an existing session, **then** access is refused; all sessions and tokens are revoked within 15 minutes (AC-17, SR-16).

### US-19 Assign brokers and cover · P-06, P-03 · M · Open (AS-19)
As a Platform Admin, I want to assign a primary and a cover broker to each client so that every client always has someone responsible.
- **Given** Kestrel has a primary broker, **when** I assign a cover broker, **then** both see Kestrel and the change is audited.
- ✗ **Given** I'm a Platform Admin assigning brokers, **when** I view a client, **then** I see its name only, never its policies or documents.
- ✗ **Given** a broker leaves Vaultline, **when** their account is disabled, **then** their clients move to the cover broker and nobody is left unassigned.
- ✗ **Given** I assign a broker to a client, **when** I submit, **then** it doesn't take effect until a second authorised person approves (AC-13, SR-17, AS-23).

### US-20 Staff use the AI assistant for one client at a time · P-03, P-07 · M · Draft
As an Account Broker, I want to use the AI assistant for a client I'm assigned to so that I can answer their questions quickly, without any risk of mixing clients.
- **Given** I'm assigned Kestrel, **when** I start an assistant conversation, **then** I must choose Kestrel and the assistant uses only Kestrel's documents.
- ✗ **Given** I'm in a Kestrel conversation, **when** I ask about Saffron Lane, **then** the assistant uses nothing from Saffron Lane; switching client starts a new conversation (AC-18, SR-06).
- ✗ **Given** Saffron Lane isn't assigned to me, **when** I try to start a conversation for it, **then** it's refused and logged.

---

## 5. Functional requirements

| ID | Area | Requirement | Users | Stories | Priority | Status |
|---|---|---|---|---|---|---|
| FR-01 | Access | Client sign-in with a second factor | P-01, P-02 | US-01 | M | Draft |
| FR-02 | Access | Staff sign-in with a second factor through a separate entry point; client accounts cannot use it | P-03 – P-06 | US-01 | M | Agreed (D-03) |
| FR-03 | Access | Every action is checked against a role permission matrix; anything not explicitly allowed is refused | All | All | M | Draft |
| FR-04 | Access | The client a user belongs to is taken only from their verified identity, never from anything they send | All | US-03, US-07 | M | Agreed (D-05) |
| FR-05 | Access | Client Admin invites, edits and removes Client Users for their own company; the last Client Admin can't be removed | P-01 | US-02 | M | Open (AS-03) |
| FR-06 | Access | Platform Admin manages staff accounts and roles; cannot read client data or change own role | P-06 | US-14 | S | Open (AS-04) |
| FR-07 | Access | Users can reset their password and second factor | All | US-01 | S | Draft |
| FR-08 | Policies | Dashboard lists the company's policies: type, insurer, limits, deductible, premium (hidden from Client Users), renewal date | P-01, P-02 | US-03 | M | Draft |
| FR-09 | Policies | Policy documents (declarations page, wording) can be viewed and downloaded | P-01, P-02 | US-03 | M | Draft |
| FR-10 | Certificates | Request a certificate naming a certificate holder, produced on an industry-standard certificate form; the client downloads it to forward to the holder | P-01, P-02 | US-04 | M | Open (AS-09, AS-16) |
| FR-11 | Certificates | Requests that only name a holder are issued automatically, **only** for active, broker-verified policies, from the verified policy record | P-01, P-02 | US-04 | M | Open (AS-09, AS-14) |
| FR-12 | Certificates | A fixed, testable rule list sends a request to the broker's approval queue if it asks for: additional insured status, waiver of subrogation, primary and non-contributory wording, project-specific wording, or any change to limits or dates; or if the policy is unverified. Everything else auto-issues. The AI assistant never makes this decision | P-03 | US-04, US-05 | M | Open (AS-09) |
| FR-13 | Certificates | Every issued certificate is kept in a register and can't be edited; corrections create a new version | P-01 – P-05 | US-06 | S | Draft |
| FR-14 | Certificates | Issued certificates are flagged when their policy is cancelled or changed; Client Admin and broker are told (holders are not, see AS-16) | P-01, P-03 | US-06 | S | Open (AS-16) |
| FR-15 | Certificates | Clients save frequent certificate holders | P-01, P-02 | US-04 | C | Draft |
| FR-16 | Certificates | Download links expire after a short time | P-01, P-02 | US-04 | M | Draft |
| FR-17 | AI assistant | Answers questions using only the signed-in client's documents | P-01, P-02, P-07 | US-07 | M | Agreed (D-04) |
| FR-18 | AI assistant | Every answer cites document and section; says so when nothing is found | P-07 | US-07 | M | Agreed (D-04) |
| FR-19 | AI assistant | Questions about whether a claim is or will be covered or paid get a standard hand-off and create a broker task | P-07, P-03 | US-07 | M | Agreed (D-04) |
| FR-20 | AI assistant | Prompts, retrieved text and answers are screened for injection and off-limits topics | P-07 | US-07 | M | Agreed (D-05) |
| FR-21 | Renewals | Reminders at 60 and 30 days before renewal to Client Admin and broker; none for cancelled policies | P-01, P-03 | US-08 | S | Draft |
| FR-22 | Renewals | Broker sends an online renewal questionnaire; Client Admin completes it | P-01, P-03 | US-08 | S | Draft |
| FR-23 | Documents | Client Admin uploads documents within type and size limits; files are scanned before use | P-01 | US-09 | M | Draft |
| FR-24 | Documents | Broker loads a client's documents and key policy data in one go for one selected client, with validation | P-03 | US-10 | M | Open (AS-08) |
| FR-25 | Claims | Client Admin reports a claim with evidence and gets a reference | P-01, P-04 | US-11 | S | Open (AS-12) |
| FR-26 | Claims | Claims Handler updates status and requests information on claims assigned to them; client sees history | P-04, P-01 | US-12 | S | Open (AS-12, AS-20) |
| FR-27 | Staff | Broker sees only clients where they're primary or cover broker, with renewals due in 60 days highlighted | P-03 | US-13 | M | Open (AS-19) |
| FR-28 | Staff | Broker sees a queue of AI assistant hand-offs for their clients | P-03 | US-07, US-20 | M | Agreed (D-04) |
| FR-29 | Audit | Tamper-evident audit of sign-ins, document access, certificate issue, role changes, exports and AI assistant turns | P-05 | US-15 | M | Draft |
| FR-30 | Audit | Compliance Officer reads audit events and AI assistant transcripts read-only, filtered by client, user and date | P-05 | US-15 | M | Draft |
| FR-31 | Audit | Only the Compliance Officer can export data; every export is logged | P-05 | US-15 | S | Draft |
| FR-32 | Assurance | Automated isolation tests run on every change and block release on failure | P-05 | US-16 | M | Agreed (D-05) |
| FR-33 | Policies | A broker verifies each policy against the insurer's declarations page before it can be used for certificates; verifier and time are recorded; any later change to limits or dates resets it to unverified | P-03 | US-17 | M | Draft |
| FR-34 | Policies | Policies follow a defined lifecycle: draft → verified → active → (endorsed) → cancelled, expired or renewed; each transition has a permitted role and is audited | P-03 | US-17, US-06, US-08 | M | Draft |
| FR-35 | Certificates | Auto-issued certificates carry the authorisation of a named licensed producer under a recorded authority; the producer and authority appear in the certificate register | P-03, P-05 | US-04 | M | Open (AS-14) |
| FR-36 | Staff | Platform Admin assigns a primary and a cover broker to each client, seeing client names only; disabling a broker moves their clients to the cover broker | P-06 | US-19 | M | Open (AS-19) |
| FR-37 | Claims | Each claim is assigned to one Claims Handler, who sees only assigned claims | P-04 | US-12 | S | Open (AS-20) |
| FR-38 | Clients | Onboarding a client prepares its separated storage, invites its first Client Admin and assigns brokers in one step | P-03 | US-18 | M | Draft |
| FR-39 | Clients | Offboarding a client removes all its users' access the same day, offers the Client Admin an export, and keeps data only for the retention period before deletion | P-03, P-01 | US-18 | S | Draft |
| FR-40 | AI assistant | Every assistant conversation, including staff ones, is bound to exactly one client; switching client starts a new conversation | P-03, P-07 | US-20 | M | Draft |
| FR-41 | Certificates | Brokers can verify any certificate's authenticity by its unique reference against the register | P-03 | US-04 | M | Draft |

---

## 6. Non-functional requirements

| ID | Category | Requirement | Target | Test | Priority | Status |
|---|---|---|---|---|---|---|
| NFR-01 | Customer separation | No client can reach another client's data by any route, including AI assistant transcripts and certificate holders | 0 successful cross-client reads | Isolation suite on every change (US-16) | M | Agreed (D-05) |
| NFR-02 | Customer separation | Separation enforced in at least two independent layers for every store: structured data, documents, encryption keys and AI retrieval. *How* is decided in ADR-001 and ADR-002, not here | With the application check disabled in a test, 0 cross-client reads from any store | Bypass test per store; design review | M | Draft |
| NFR-03 | Availability | Portal and certificate service available around the clock | 99.5% monthly | Synthetic monitoring | S | Open (AS-17) |
| NFR-04 | Speed of key events | Standard certificate ready after request | 95% within 60 s | Timed end-to-end test | M | Open (AS-09) |
| NFR-05 | Speed of key events | Broker answers non-standard certificate requests | Within 4 business hours | Queue report | S | Open (AS-09) |
| NFR-06 | Performance | Portal pages and API calls | 95% of pages < 2 s; 95% of API calls < 500 ms | Load test | S | Draft |
| NFR-07 | Performance | AI assistant answers | First words < 2 s; 95% complete < 8 s | Timed test | S | Draft |
| NFR-08 | Data freshness | Documents a broker loads are usable in the portal and AI assistant | Within 5 min | Timed test | M | Draft |
| NFR-09 | Data freshness | Certificates reflect the current **verified** policy record | 0 certificates differing from the verified record at issue time; 0 auto-issued certificates from unverified policies | Automated comparison on every certificate; test with an unverified policy | M | Draft |
| NFR-10 | Scalability | Growth without redesign | 500 clients, 5,000 users, 100 concurrent AI assistant sessions | Load test at target | S | Open (AS-07) |
| NFR-11 | Audit | All sensitive actions recorded, tamper-evident | 100% of FR-29 events captured; entries can't be altered | Audit test cases | M | Draft |
| NFR-12 | Data residency | All data and processing in the United States | 100% of stores and processing in US regions | Configuration check | S | Open (AS-13) |
| NFR-13 | Retention and deletion | Records kept, then deleted, including after a client is offboarded | Policies, claims and certificates 7 years after expiry; AI transcripts 1 year; audit events 7 years; deletion within 30 days of retention ending | Lifecycle test, including an offboarded client | S | Open (AS-10, AS-24) |
| NFR-14 | Recovery | Data can be restored after loss | RPO ≤ 24 h; RTO ≤ 4 h | Restore test each release | S | Draft |
| NFR-15 | Accessibility | Usable by people with disabilities | WCAG 2.2 AA | Automated checks plus manual keyboard and screen-reader test | S | Draft |
| NFR-16 | Mobile use | Certificate request works on a phone on site | Completed one-handed in ≤ 3 steps on a 375 px screen | Usability test (5 users) | M | Open (AS-03) |
| NFR-17 | Mobile use | Poor signal on site | Request survives a dropped connection without duplicate certificates | Network-throttling test | S | Draft |
| NFR-18 | Time handling | Dates shown and calculated correctly | Policy periods start and end at 12:01 a.m. standard time at the insured's address; stored in UTC; correct across daylight-saving changes | Unit tests across time zones and clock changes | M | Draft |
| NFR-19 | Browser and device support | Current browsers and phones | Latest 2 versions of Chrome, Edge, Safari, Firefox | Cross-browser tests | S | Draft |
| NFR-20 | Observability | Problems and attacks noticed quickly | Alerts within 5 min for repeated cross-client denials, AI screening blocks, error spikes, bulk downloads | Alert test | M | Draft |
| NFR-21 | AI safety | AI assistant resists manipulation | 100% pass on a red-team set of ≥ 30 prompts, including instructions hidden in documents | Evaluation before each release | M | Agreed (D-05) |
| NFR-22 | AI quality | Answers are grounded | ≥ 95% of answers carry a valid citation; 0 answers judging cover | Evaluation set of ≥ 50 questions per client | M | Agreed (D-04) |
| NFR-23 | Security | Second factor and session limits | 100% of users; phishing-resistant MFA for staff and Client Admins; idle timeout 15 min (staff), 30 min (client); access tokens ≤ 1 h; revocation ≤ 15 min | Configuration and end-to-end test | M | Draft |
| NFR-24 | Privacy | Logs hold identifiers, not content | No document text, chat text or personal data in application logs | Log review | M | Draft |
| NFR-25 | Cost ceiling | Development environment cost | ≤ $20 a month, alert at 80%; any development-only stand-in keeps the isolation design identical to production | Cost alert; design review of stand-ins | M | Open (AS-11, AS-15) |
| NFR-26 | Supportability | Problems can be traced | Every request has a correlation ID; runbook for top 5 incidents | Runbook review | S | Draft |
| NFR-27 | Maintainability | Changes are safe and repeatable | All changes via reviewed pull request and automated pipeline; no manual changes to environments | Pipeline and branch rules | M | Draft |
| NFR-29 | Staff access | Staff access to more than one client goes through a separate, audited path with its own role checks; it never weakens client-to-client isolation | 100% of cross-client staff reads audited with staff ID, client and reason; Compliance access read-only | Audit test; isolation suite run as each staff role | M | Draft |
| NFR-30 | Notifications | Emails and in-app notifications contain no policy, claim or financial details | 0 notifications containing such details; every link requires sign-in | Content check on notification templates | M | Draft |
| NFR-31 | Scalability | Certificate volume | 10,000 certificate requests a month; peak 50 an hour | Load test | S | Open (AS-18) |
| NFR-28 | Simulated data | Training data is clearly fictional | 100% of seed data, sample documents and pages marked "Fictional company for training purposes" | Review checklist | M | Agreed (D-06) |

---

## 7. Infrastructure needs

Needs, not products: the enterprise architect chooses products within constraint D-07 (AWS).

| ID | Need | Driven by | Priority | Status |
|---|---|---|---|---|
| INF-01 | Two separate identity populations (clients and staff) with second factor, roles and a client identifier carried in the verified identity | FR-01 – FR-06, NFR-23 | M | Draft |
| INF-02 | Two separate web front ends (client portal, staff portal), the staff one reachable only from trusted devices or networks | FR-02, D-03 | M | Agreed (D-03) |
| INF-03 | Request-driven, scale-to-zero compute for the API, to meet the cost ceiling | NFR-25, NFR-10 | S | Open (AS-11) |
| INF-04 | Structured data store with per-client row protection at the data layer | FR-04, NFR-02 | M | Draft |
| INF-05 | Unstructured document storage separated per client, with per-client encryption keys and malware scanning on arrival | FR-23, FR-24, NFR-02 | M | Draft |
| INF-06 | Document generation service producing certificates from the verified policy record, applying the recorded producer authorisation | FR-10 – FR-14, FR-35, NFR-04, NFR-09 | M | Open (AS-09, AS-14) |
| INF-07 | Queue and scheduler for certificate approvals, renewal reminders and AI assistant hand-offs | FR-12, FR-21, FR-28 | M | Draft |
| INF-08 | Outbound email for invitations, reminders and notifications | FR-05, FR-21, FR-22 | M | Draft |
| INF-09 | AI service with retrieval limited to one client's documents, input and output screening, and citation support | FR-17 – FR-20, NFR-21, NFR-22 | M | Agreed (D-04) |
| INF-10 | Text extraction from scanned documents for the AI assistant | FR-17, NFR-08 | S | Draft |
| INF-11 | Tamper-evident audit storage separate from application data | FR-29, NFR-11 | M | Draft |
| INF-12 | Backups, versioned document storage and restore within RPO and RTO | NFR-14 | S | Draft |
| INF-13 | Monitoring, alerting and correlation IDs across all components | NFR-20, NFR-26 | M | Draft |
| INF-14 | Environments: development and test now; staging and production later, isolated from each other | NFR-27 | M | Draft |
| INF-15 | Automated pipeline that runs the isolation suite and blocks release on failure | FR-32, NFR-27 | M | Agreed (D-05) |
| INF-16 | Automated onboarding of a new client's separated storage, keys and AI knowledge | NFR-02, NFR-10 | S | Open (AS-07) |
| INF-17 | Later: integration layer for insurer data feeds | AS-08 | W | Open (AS-08) |
| INF-18 | Later: single sign-on for larger clients' own identity systems | NFR-10 | W | Draft |
| INF-19 | A separate, audited access path for staff who work across clients | NFR-29, FR-30 | M | Draft |
| INF-20 | A rules component for certificate routing that is deterministic and testable, separate from the AI service | FR-12 | M | Draft |

**Decisions handed to the architect (ADRs):** ADR-001 AI retrieval isolation at 500 clients · ADR-002 data-layer isolation · ADR-003 certificate generation and signing · ADR-004 development environment within the cost ceiling · ADR-005 staff cross-client access path.

---

## 7a. Security requirements

From the secure design review. Mandatory for every item listed under "Applies to". ASVS references are by chapter name. CSF codes are NIST CSF 2.0 categories.

| ID | Requirement | Threats | Control | Second lock | Standards | Verify | Applies to | Pri. | Status |
|---|---|---|---|---|---|---|---|---|---|
| SR-01 | Client context comes only from the verified identity | TM-04 | API derives client from token claims; any client value in the request that differs is refused and logged | Data-layer lock (SR-03) | ASVS Authorization; CSF PR.AA | Isolation suite tampering tests | FR-04 | M | Agreed (D-05) |
| SR-02 | Every endpoint checks object-level and function-level authorisation, deny by default | TM-03 | Central authorisation on every request against the permission matrix | SR-03 | ASVS Authorization; OWASP API1, API5 | Automated tests per endpoint and role | FR-03 | M | Draft |
| SR-03 | Structured data is locked to the client at the data layer | TM-05 | Data-layer policy enforces client on every table holding client data; application roles can't bypass it | SR-01, SR-02 | ASVS Authorization; CSF PR.DS | Bypass test: application check disabled returns 0 rows | NFR-02, INF-04 | M | Draft |
| SR-04 | Every store holding client data is scoped to one client and listed in a store register | TM-06 | Store register (queues, caches, files, extracted text, indexes, conversations, logs, backups, exports) with isolation method per store | Per-client encryption keys where data is at rest | CSF ID.AM, PR.DS | Security review of register each release; isolation suite per store | NFR-01, NFR-02 | M | Draft |
| SR-05 | Per-client encryption keys for documents and certificates | TM-06, TM-07 | Separate key per client; key policy limited to that client's context | Store-level separation | ASVS Data Protection; CSF PR.DS | Configuration check | INF-05 | M | Draft |
| SR-06 | Every AI assistant conversation, including staff ones, is bound to exactly one client | TM-08 | Client chosen at start; retrieval, history and hand-offs limited to it; switching starts a new conversation | SR-07 | OWASP LLM sensitive information disclosure | Isolation suite with staff accounts | FR-17, FR-28, NFR-29 | M | Draft |
| SR-07 | Retrieval can only return the conversation's client's content | TM-07 | Retrieval scope set on the server from the bound client; never from prompt text | Separate knowledge per client or enforced filter tested by ADR-001 | OWASP LLM vector and embedding weaknesses | Cross-client retrieval tests | FR-17, NFR-02 | M | Agreed (D-05) |
| SR-08 | Prompts, retrieved text and answers are screened for injection and off-limits topics | TM-09, TM-11 | Input and output screening; denied topic for cover judgements | System prompt rules plus SR-10 | OWASP LLM prompt injection; NIST AI RMF Measure | Red-team set (NFR-21) before each release | FR-19, FR-20 | M | Agreed (D-04, D-05) |
| SR-09 | Retrieved documents are passed as data, never as instructions | TM-09 | Documents placed only in a clearly delimited context section; no document text in system instructions | SR-08 | OWASP LLM prompt injection | Hidden-instruction tests | FR-20 | M | Draft |
| SR-10 | Answers can't leak data through rendering | TM-10 | Plain text plus citations; links only to Vaultline document references; no remote images or scripts | Content security policy (SR-31) | OWASP LLM improper output handling | Rendering tests with crafted documents | FR-18 | M | Draft |
| SR-11 | AI use is limited and its instructions aren't revealed | TM-12 | Per-user and per-client rate and size limits; cost alarms; system instructions hold no secrets | Global spend cap | OWASP LLM unbounded consumption, system prompt leakage | Load test; red-team prompts | NFR-25 | S | Draft |
| SR-12 | Policies used for auto-issue are verified by a second person | TM-14 | Keyer and verifier must differ; both recorded | Certificate matches verified record (NFR-09) | ASVS Business Logic; separation of duties | Test: same broker can't verify | FR-33, US-17 | M | Open (SQ-03) |
| SR-13 | Certificates can be verified as genuine | TM-13 | Unique reference and integrity protection on each certificate; register lookup for brokers | Immutable register (FR-13) | ASVS Data Protection | Altered certificate fails verification | FR-13, FR-35 | M | Draft |
| SR-14 | Download links are unguessable, short-lived and tied to the signed-in user's client | TM-15 | Random, expiring links; access re-checked on use | SR-05 | ASVS Data Protection | Expired, forwarded and guessed link tests | FR-16 | M | Draft |
| SR-15 | Phishing-resistant MFA for all staff and Client Admins | TM-01, TM-02 | Phishing-resistant factors required; weaker factors allowed for Client Users only | Conditional access for staff (trusted devices) | ASVS Authentication; CSF PR.AA | Configuration check; sign-in tests | FR-01, FR-02, NFR-23 | M | Draft |
| SR-16 | Sessions are short, revocable, and sensitive actions need re-authentication | TM-01, TM-21 | Access tokens ≤ 1 h; revocation within 15 min on offboarding, role change or disable; re-authentication for invite, export and role change; new-device alerts | Idle timeouts (NFR-23) | ASVS Session Management | Revocation and re-authentication tests | FR-05, FR-39 | M | Draft |
| SR-17 | Staff creation, role grants and broker assignments need a second approver | TM-18 | Four-eyes approval; alert on every grant | Audit (SR-22) | CSF PR.AA; separation of duties | Test: single admin can't complete a grant | FR-06, FR-36 | M | Open (SQ-03) |
| SR-18 | Staff cross-client access is just-in-time, time-limited and reasoned | TM-19 | Access requested with a reason, granted for a set period, logged | Alerts on unusual volume | CSF PR.AA, DE.CM | Audit tests | NFR-29, INF-19 | M | Draft |
| SR-19 | Exports need a second approver and are limited | TM-19 | Second approval; size limits; alert on every export | Audit | CSF PR.DS, DE.CM | Export tests | FR-31 | M | Draft |
| SR-20 | Invitations are bound, single-use and visible | TM-16 | Bound to email, single-use, 72 h expiry; all Client Admins notified | Re-authentication to invite (SR-16) | ASVS Authentication | Invite tests | FR-05 | M | Draft |
| SR-21 | Uploaded and imported files are made safe before use | TM-17 | Type and size checks, malware scan, active content stripped, formulas neutralised, processing isolated | Per-client stores | ASVS File Handling; OWASP Top 10 injection | Malicious file test set | FR-23, FR-24 | M | Draft |
| SR-22 | Audit is append-only and outside every application or admin's control | TM-20, TM-25 | Separate store; write-only for applications; integrity verification | Copy to a separate account | ASVS Security Logging; CSF DE.AE | Tamper tests; integrity check | FR-29, NFR-11, INF-11 | M | Draft |
| SR-23 | No sensitive content in logs, telemetry or AI tracing | TM-29 | Identifiers only; scrubbing at source | Log review | ASVS Security Logging | Automated log scan | NFR-24 | M | Draft |
| SR-24 | Code, dependencies, secrets, infrastructure and containers are scanned on every change | TM-22 | Automated scanning in the pipeline; high findings block release | Software bill of materials | NIST SSDF; CSF PR.PS | Pipeline evidence | NFR-27 | M | Draft |
| SR-25 | The pipeline uses short-lived credentials and protected branches | TM-22 | No long-lived keys; reviewed pull requests; signed build artefacts | Production deploy approval | SLSA; NIST SSDF | Configuration check | NFR-27, INF-15 | M | Draft |
| SR-26 | Environments are isolated at account level; production changes only via pipeline | TM-22, TM-31 | Separate accounts per environment; no human write access to production | Audit | CSF PR.IR | Configuration check | INF-14 | S | Draft |
| SR-27 | Incident response and regulator notification are planned and practised | TM-25 | Written plan, notification decision tree with deadlines, annual tabletop | — | CSF RS.MA, RS.CO | Tabletop record | — | S | Open (SQ-01) |
| SR-28 | Privacy notices cover certificate holders and AI conversations; holder data is minimal | TM-26, TM-27 | Notice in the portal; holder name and address only | Retention limits (NFR-13) | LINDDUN; CSF GV.OC | Content review | FR-10, FR-17 | S | Draft |
| SR-29 | The AI service doesn't retain or train on Vaultline data and processes in the US | TM-30 | Supplier terms and configuration | Contract review | NIST AI RMF Govern; CSF GV.SC | Supplier evidence | INF-09 | M | Draft |
| SR-30 | Outbound email is authenticated and never asks for credentials | TM-23 | Authenticated sending domain; template rules | NFR-30 | CSF PR.DS | Template and domain check | INF-08 | S | Draft |
| SR-31 | Browser protections are on | TM-10 | Security headers and content security policy | — | ASVS Configuration | Automated header scan | INF-02 | S | Draft |
| SR-32 | Isolation and security tests can't be skipped without security approval | AC-15 | Test changes require security review; release blocked on failure | Audit | NIST SSDF | Pipeline rule | FR-32 | M | Draft |

## 7b. Risk register (summary)

| ID | Risk | Inherent L×I | Key controls | Residual L×I | Owner | Decision | Review |
|---|---|---|---|---|---|---|---|
| R-01 | One client sees another's data (any route) | 4×5 = 20 | SR-01 – SR-07, SR-32 | 1×5 = 5 | User (Founder) | Mitigate | Each release |
| R-02 | Cross-client leak through AI (including staff and injection) | 4×5 = 20 | SR-06 – SR-10 | 2×5 = 10 | User | Mitigate | Each release |
| R-03 | Wrong or forged certificate issued under Vaultline's authorisation | 3×5 = 15 | SR-12, SR-13, NFR-09 | 1×5 = 5 | User | Mitigate | Each release |
| R-04 | Staff account takeover or insider exfiltration | 3×5 = 15 | SR-15 – SR-19, SR-22 | 2×4 = 8 | User | Mitigate | Quarterly |
| R-05 | Holders rely on certificates after cancellation (AS-16) | 3×3 = 9 | FR-14 flags; certificate wording | 2×3 = 6 | User | **Accept?** Needs the user's decision | Release one |
| R-06 | AI cost or capacity exhaustion | 3×3 = 9 | SR-11 | 1×3 = 3 | User | Mitigate | Monthly |
| R-07 | Supply-chain compromise | 2×5 = 10 | SR-24 – SR-26 | 1×5 = 5 | User | Mitigate | Quarterly |
| R-08 | Missed regulator notification after an incident | 2×4 = 8 | SR-27 | 1×4 = 4 | User | Mitigate | Annually |
| R-09 | AI gives cover advice relied on by a client | 3×4 = 12 | SR-08, NFR-22 | 1×4 = 4 | User | Mitigate | Each release |
| R-10 | Unknown weaknesses in Build 1 | 4×4 = 16 | Review against SR-01 – SR-10 (SD-23) | Unknown | User | Mitigate | Before next build |

## 8. Edge cases to design for

| Edge | Requirement it affects |
|---|---|
| A policy is cancelled after certificates were issued on it | FR-14 |
| Two people request certificates for the same holder at once | FR-11, NFR-17 |
| Signal drops mid-request on a job site | NFR-17 |
| Renewal date falls on a daylight-saving change or in another time zone | NFR-18 |
| A Client User leaves the client company | FR-05 |
| The last Client Admin tries to leave | FR-05 |
| A broker leaves Vaultline with clients still assigned | FR-06, FR-27 |
| An uploaded document contains hidden instructions for the AI assistant | FR-20, NFR-21 |
| A data file loaded by a broker contains rows for a different client | FR-24 |
| An endorsement changes a verified policy's limits | FR-33, FR-34 |
| A primary broker is on leave or leaves | FR-36 |
| A former user of an offboarded client tries to sign in | FR-39 |

---

## 9. Out of scope for release one

| Item | Reason |
|---|---|
| Online premium payment | Handled by insurers; adds payment scope |
| Quoting and binding new policies | Needs insurer integrations |
| Insurer data feeds | Later (AS-08, INF-17) |
| Native mobile apps | Responsive web meets NFR-16 |
| AI assistant judging cover or advising on claims | Decision D-04 |
| Accounts for certificate holders | They receive certificates only (P-08) |
| System emailing certificates to holders, or telling holders about cancellations | AS-16 |
| Paid tiers | AS-06 |

---

## 9a. Release gates

| Gate | Must pass before |
|---|---|
| RG-01 | Data-layer lock (SR-03) and store register (SR-04) in place **before** any new feature that stores client data |
| RG-02 | Isolation suite green for every store and every role, including staff (SR-01 – SR-07, SR-32), on every release |
| RG-03 | AI red-team set (≥ 30 prompts, including document-borne injection and rendering tests) and grounding evaluation pass before any AI change ships (NFR-21, NFR-22, SR-08 – SR-10) |
| RG-04 | Certificate integrity and four-eyes verification tests pass before auto-issue is switched on (SR-12, SR-13) |
| RG-05 | Pipeline scanning clean of high findings (SR-24) |
| RG-06 | Security review of Build 1 completed (SD-23) before the next build |
| RG-07 | Penetration test and AI red-team exercise before any real client is onboarded (not needed for the course) |

## 10. Response to the enterprise architect's review

| Finding | How v2 responds |
|---|---|
| EA-01 Baseline | Recorded as unchecked in section 0; needs Build 1 pushed |
| EA-02 Source of truth | FR-33, US-17, NFR-09 rewritten, AP-03 |
| EA-03 Signing authority | AS-14, FR-35, US-04 |
| EA-04 Cost ceiling | AS-15, NFR-25 |
| EA-05 Prescriptive NFR | NFR-02 rewritten as an outcome; AP-01; ADR-001 |
| EA-06 Staff crossing clients | NFR-29, INF-19, ADR-005 |
| EA-07 Policy lifecycle | FR-34 |
| EA-08 Assignment | AS-19, AS-20, FR-36, FR-37, US-19 |
| EA-09 Data model | Section 2b |
| EA-10 Holder delivery | AS-16, FR-10, out of scope |
| EA-11 Routing rules | FR-12 rewritten, INF-20 |
| EA-12 Client lifecycle | FR-38, FR-39, US-18 |
| EA-13 Transcripts | NFR-01, NFR-13, section 2b |
| EA-14 Holder notification | AS-16, FR-14, out of scope |
| EA-15 Availability window | AS-17, NFR-03 |
| EA-16 Volumes | AS-18, NFR-31 |
| EA-17 Notification content | NFR-30 |
| EA-18 Policy times | NFR-18 |
| EA-19 Test traceability | Test column added when the backlog is drafted |

## 11. Response to the secure design review

| Finding | How v3 responds |
|---|---|
| SD-01 Derived stores | SR-04 (store register), RG-01 |
| SD-02 Staff AI across clients | US-20, FR-40, SR-06 |
| SD-03 AI output leakage | US-07 must-not, SR-10 |
| SD-04 Certificate forgery | US-04 must-not, FR-41, SR-13 |
| SD-05 Self-verification | US-17 must-not, SR-12, AS-23 |
| SD-06 Weak MFA | NFR-23, SR-15, SR-16, US-01 must-not |
| SD-07 Invitation abuse | US-02 must-not, SR-20 |
| SD-08 Malicious files | US-09, US-10 must-nots, SR-21 |
| SD-09 Admin grants by proxy | US-14, US-19 must-nots, SR-17, AS-23 |
| SD-10 Insider export | SR-18, SR-19, AS-23 |
| SD-11 Audit tamper-evidence | SR-22 |
| SD-12 Telemetry content | SR-23 |
| SD-13 AI consumption | SR-11 |
| SD-14 Supply chain | SR-24, SR-25 |
| SD-15 Environment isolation | SR-26 |
| SD-16 Incident response | SR-27, AS-21 |
| SD-17 Token revocation | NFR-23, SR-16, US-18 must-not |
| SD-18 Privacy notices | SR-28 |
| SD-19 AI supplier terms | SR-29 |
| SD-20 Email spoofing | SR-30 |
| SD-21 Browser protections | SR-31 |
| SD-22 Control ordering | RG-01 |
| SD-23 Build 1 unreviewed | RG-06 |

## 12. Sign-off status

| Role | Status | What's needed |
|---|---|---|
| Product owner | Draft | User answers AS-01 – AS-25 and agrees AP-01 – AP-08 |
| Enterprise architect | Reviewed v1 | Baseline from Build 1 (EA-01); ADR-001 – ADR-005 |
| Secure design | **Not signed off** | v3 re-reviewed after the user's answers; expected outcome "signed off with conditions" (RG-01, RG-06) |

## 13. Next steps

1. User confirms or rejects AS-01 to AS-25 and principles AP-01 to AP-08, and decides R-05 (AS-25). Rows move from Open or Draft to Agreed.
2. User pushes Build 1 so the baseline can be recorded (EA-01).
3. **Secure design** re-reviews v3 and gives its sign-off decision.
4. Enterprise architect starts journeys, data model, DFDs and ADR-001 to ADR-005.
5. Only after secure design signs off, the product owner designs the backlog (including draft security items SEC-01 – SEC-14): draft the backlog, mapping each item against the existing KAN epics and tasks as keep, change, add or retire, for one approval.

**Note:** `docs/BACKLOG.md` (24 Nov) was drafted before review and is **withdrawn**. It will be replaced at step 3.
