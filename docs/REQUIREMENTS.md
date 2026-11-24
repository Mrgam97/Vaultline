> **Superseded.** Replaced by `docs/requirements/2026-11-24_vaultline-requirements_v1.md`. Kept for history only.

# Vaultline – Functional and Non-Functional Requirements

> **Fictional company created for a course project.** Vaultline, Kestrel Groundworks LLC and Saffron Lane Hospitality Group LLC are not real. All data is sample data.

**Owner:** Product Owner · **Version:** 0.2 (draft for review) · **Date:** 24 November 2026 · **Cloud:** AWS (us-east-1)

**Roles:**
- **CA** = Client Admin
- **CU** = Client User (new in v0.2)
- **AB** = Account Broker
- **CH** = Claims Handler
- **CO** = Compliance Officer
- **PA** = Platform Admin
- **All** = every role

**Priority (MoSCoW):**
- **M** = Must
- **S** = Should
- **C** = Could
- **W** = Won't (this release)

## What changed in v0.2

- **New role: Client User (CU).** Site managers and restaurant managers who need certificates but must not see premiums, report claims or manage users.
- **Certificates are now the core feature.** Clients request a certificate of insurance naming a certificate holder. Standard requests are issued automatically; anything that changes cover goes to the broker.
- **Getting data in.** Brokers bulk-upload policies at launch; insurer feeds are a Later item.
- **Renewals** go beyond reminders: brokers can send a renewal questionnaire.
- **New section 3: future infrastructure requirements** (Now, Next, Later).
- New requirements are FR-31 to FR-39 and NFR-28 to NFR-30. v0.1 IDs are unchanged so Jira links still work.

## Product Owner assumptions (overrule any of these)

| ID | Assumption | Why |
|---|---|---|
| A-01 | Success metric is % of certificate requests self-served, target 70% | Certificates are the most frequent, most urgent client pain, especially for contractors |
| A-02 | A Client User role is needed | The people who need certificates are usually not the Client Admin |
| A-03 | At launch, brokers bulk-upload PDFs and key policy data; insurer feeds come Later | Insurer integrations are slow to build and not needed to prove value |
| A-04 | Certificates are generated on an ACORD 25-style template naming the holder; standard requests auto-issue, others need broker approval | Matches how US contractors actually use certificates, while keeping changes to cover with a licensed broker |
| A-05 | Scale: 2 clients now, about 50 clients and 500 users in year 1, about 500 clients and 5,000 users by year 2 | Gives the architect a realistic target without over-building |
| A-06 | Vaultline is a commission-based broker and the portal is free to clients | Working assumption from the Founder questions |

**Working success metric (assumption A-01):** **% of certificate of insurance requests self-served through the portal, without broker involvement.** Target: 70% within 3 months of a client going live.

---

## 1. Functional requirements

Every requirement has a **Must NOT** column: what a user must not be able to do. A requirement isn't done until its Must NOT case is tested.

### 1.1 Identity and access

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-01 | Clients sign in with email and password plus MFA through the client Cognito pool | CA | M | A client account must not sign in to the admin portal | KAN-12 |
| FR-02 | Staff sign in through a separate Cognito pool with MFA, and reach the admin portal only through AWS Verified Access | AB, CH, CO, PA | M | Staff must not reach the admin portal from an untrusted device or network | KAN-13 |
| FR-03 | Every API call is checked against the role permission matrix; anything not explicitly allowed is denied | All | M | No user can call an endpoint their role doesn't allow, even by crafting the request by hand | KAN-14 |
| FR-04 | The tenant is taken only from the signed-in user's token | All | M | A user must not switch tenant by changing an ID, parameter, header or chat message | KAN-28 |
| FR-05 | Platform Admin can create and disable staff accounts and assign roles | PA | S | PA must not read client documents or claims, or change their own role | KAN-15 |
| FR-31 | Client Admin invites, edits and removes Client Users for their own company | CA | M | A Client Admin must not add users to another company or grant any staff role | – |
| FR-32 | Client User can view policies (without premiums) and download or request certificates | CU | M | A Client User must not see premiums or financial details, report claims, upload documents or manage users | – |
| FR-06 | Users can reset their password and re-enrol MFA | All | S | A reset link must not work after 30 minutes or be reused | – |

### 1.2 Client portal: policies and renewals

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-07 | The dashboard lists all of the client's policies: type, insurer, limits, deductible, premium, renewal date | CA | M | A client must not see any policy belonging to another client | KAN-16 |
| FR-08 | Policy detail page shows the declarations page and policy wording as downloadable PDFs | CA | M | A client must not open another client's PDF, even with a guessed or copied link | – |
| FR-09 | Renewal reminders by email and in-app at 60 and 30 days before expiry, copied to the assigned broker | CA, AB | S | Reminders must not be sent for cancelled policies or to anyone outside the client and their broker | KAN-18 |
| FR-38 | Broker sends a renewal questionnaire (revenue, payroll, headcount, vehicles, locations); the Client Admin completes it online and the broker is notified | AB, CA | S | A Client User must not see or submit renewal financials | – |
| FR-39 | Users choose how they receive notifications (email, in-app) | All | C | A user must not turn off security notifications such as new sign-ins | – |
| FR-10 | Client can see who their Account Broker is and message them | CA | C | A client must not message or see other clients' brokers' caseloads | – |

### 1.3 Certificates of insurance (COIs)

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-11 | Client downloads a current general evidence-of-cover certificate instantly as a PDF | CA, CU | M | A client must not download a certificate for a policy that's expired, cancelled or not theirs | KAN-17 |
| FR-12 | Client requests a certificate of insurance naming a certificate holder (for example a general contractor), generated on an ACORD 25-style template | CA, CU, AB | **M** | A client must not change policy terms or add additional insureds themselves | – |
| FR-13 | Download links expire after 15 minutes | CA, CU | M | An expired or forwarded link must not work | KAN-17 |
| FR-33 | A request that only names a holder (name and address) is issued automatically; a request asking for additional insured status, waiver of subrogation, primary and non-contributory wording or any change to cover goes to the broker | CA, CU, AB | M | A client must not self-issue a certificate that adds an additional insured or changes cover | – |
| FR-34 | Broker sees a queue of certificate requests for their clients and can approve, edit or reject each one; the client is told the outcome | AB | M | A broker must not issue certificates for unassigned clients; an approved certificate must not show cover the policy doesn't have | – |
| FR-35 | Clients save frequent certificate holders and reuse them | CA, CU | S | A client must not see another client's saved holders | – |
| FR-36 | Every issued certificate is kept in a register (holder, date, policy, who issued it) and can be re-downloaded | CA, CU, AB, CO | S | A certificate must not be changed after issue; corrections create a new version | – |

### 1.4 Documents

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-14 | Client uploads documents (PDF, JPG, PNG, up to 25 MB) into their own storage | CA | M | Unsupported, oversized or malware-infected files must not be accepted or made available | KAN-19, KAN-31 |
| FR-15 | Broker uploads policy documents for an assigned client; the client sees them within 5 minutes, and they are added to that client's chatbot knowledge base | AB | M | A broker must not upload to, or view, an unassigned client | KAN-21 |
| FR-37 | Broker bulk-uploads policy PDFs and enters key policy data (by form or CSV import) for one selected client at a time; the import is checked before it's saved | AB | M | An import must not write to any client other than the one selected; a CSV row naming a different client is rejected | KAN-21 |
| FR-16 | Every document is tagged with its client, type and upload date, and can be searched by these | CA, AB | S | Search must not return documents from another client | – |

### 1.5 Claims

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-17 | Client reports a claim with date, type, description, estimated value and up to 10 evidence files, and gets a claim reference | CA | M | A client must not report a claim against another client's policy | KAN-22 |
| FR-18 | Claims Handler updates claim status (Reported, With insurer, Info needed, Settled, Declined), and the client sees each change with its history | CH, CA | M | A client must not change a claim's status; a Claims Handler must not see claims for clients they aren't working on | KAN-23 |
| FR-19 | Claims Handler can request more information; the client is notified and can reply with files | CH, CA | S | Replies must not be attached to a different claim or client | – |

### 1.6 Admin portal

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-20 | Broker sees only their assigned clients, with renewals due in 60 days highlighted | AB | M | A broker must not see or open unassigned clients | KAN-20 |
| FR-21 | Broker creates client accounts and invites the Client Admin | AB | S | A broker must not create an account under another client or grant staff roles | – |
| FR-22 | Broker sees a queue of chatbot hand-offs ("will this be paid?" questions) for their clients | AB | M | Hand-offs must not appear in another broker's queue | KAN-26 |

### 1.7 AI chatbot (Amazon Bedrock)

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-23 | Client asks questions about their cover; answers come only from their own knowledge base | CA, CU | M | The chatbot must not quote, summarise or hint at another client's documents or claims | KAN-24, KAN-25 |
| FR-24 | Every answer cites the document and section it came from; if nothing relevant is found, the bot says so | CA | M | The bot must not invent a clause or answer without a citation | KAN-25 |
| FR-25 | Questions about whether a claim is covered or will be paid get a standard hand-off reply and create a broker task | CA, AB | M | The bot must not say a claim is or isn't covered or will be paid | KAN-26 |
| FR-26 | Prompts, retrieved text and responses pass through Bedrock Guardrails (prompt-attack filter, denied topic, grounding check) | CA | M | Instructions hidden in an uploaded document or chat message must not change the bot's rules or reveal data | KAN-27 |
| FR-27 | Clients can rate an answer as helpful or not helpful | CA | C | Ratings must not expose the conversation to other clients | – |

### 1.8 Audit and compliance

| ID | Requirement | Role | Priority | Must NOT | Jira |
|---|---|---|---|---|---|
| FR-28 | Audit log of sign-ins, document views and downloads, role changes, exports and chat turns, with user, client and time | CO | M | Nobody, including PA, can edit or delete audit entries | KAN-32 |
| FR-29 | Compliance Officer reads chat transcripts and audit logs (read-only) and can filter by client, user and date | CO | M | CO must not edit data or use the chatbot as a client | KAN-33 |
| FR-30 | Compliance Officer can export audit data; every export is logged | CO | S | No other role can bulk-export data | – |

### 1.9 Out of scope for this release (Won't)

| ID | Item | Reason |
|---|---|---|
| W-01 | Online payment of premiums | Handled by insurers; adds payment security scope |
| W-02 | Quoting and binding new policies | Needs insurer integrations; not core to the success metric |
| W-03 | Native mobile apps | Responsive web is enough for the course |
| W-04 | Chatbot interpreting cover or advising on claims | Licensed advice and liability risk; hand off to brokers |

---

## 2. Non-functional requirements

| ID | Category | Requirement | Target / measure | How we verify | Priority |
|---|---|---|---|---|---|
| NFR-01 | Security – tenant isolation | No client can access another client's data through any route: API, storage, search, chatbot or exports | 0 cross-tenant reads in the automated isolation suite | Isolation suite (KAN-30) runs on every pull request and blocks merge on failure | M |
| NFR-02 | Security – defence in depth | Isolation is enforced in at least two independent layers | App-layer tenant check **and** Aurora Row-Level Security **and** a separate S3 bucket, KMS key and knowledge base per client | Code review plus a test that bypasses the app layer and still gets 0 rows | M |
| NFR-03 | Security – authentication | MFA for every user; sessions expire | MFA 100%; idle timeout 15 min (admin), 30 min (client); tokens ≤ 1 h | Cognito configuration review, end-to-end test | M |
| NFR-04 | Security – encryption | Data encrypted in transit and at rest | TLS 1.2+ everywhere; KMS encryption on Aurora, S3 and logs | AWS Config rules, Security Hub | M |
| NFR-05 | Security – secrets | No secrets in code, tickets, pages or logs | 0 findings from secret scanning | GitHub secret scanning in CI | M |
| NFR-06 | Security – least privilege | Each Lambda and role has only the permissions it needs | No wildcard (`*`) actions on data resources | IAM Access Analyzer, CDK review | M |
| NFR-07 | AI safety | Chatbot resists prompt injection and stays on its own client's data | 100% pass on the red-team prompt set (≥ 30 prompts, including document-borne injection) | Evaluation run before each chatbot release; AI Security Engineer sign-off | M |
| NFR-08 | AI quality | Answers are grounded and cited | ≥ 95% of answers carry a valid citation; 0 answers judging cover in the evaluation set | Evaluation set of ≥ 50 questions per client | M |
| NFR-09 | Privacy | Minimal personal data; logs hold IDs, not content | No document text, chat text or personal data in application logs | Log review, automated log lint | M |
| NFR-10 | Auditability | All security-relevant actions are logged and tamper-evident | 100% of FR-28 events captured; CloudTrail on with log file validation | Audit test cases, CloudTrail configuration | M |
| NFR-11 | Data retention | Records kept for a defined period, then deleted | Policies and claims: 7 years after expiry (assumed; to confirm with Compliance); chat transcripts: 2 years | Lifecycle rules in S3 and the database; Compliance page | S |
| NFR-12 | Performance – portal | Pages load quickly | 95% of page loads < 2 s; 95% of API calls < 500 ms | CloudWatch metrics, load test | S |
| NFR-13 | Performance – chatbot | Answers arrive quickly | 95% of answers < 8 s; streaming starts < 2 s | CloudWatch metrics | S |
| NFR-14 | Performance – certificates | Certificates download instantly | 95% of certificate downloads < 3 s | End-to-end test timing | M |
| NFR-15 | Availability | Portal is available during US business hours | 99.5% monthly (course target) | CloudWatch Synthetics canary | S |
| NFR-16 | Backup and recovery | Data can be restored after loss | RPO ≤ 24 h, RTO ≤ 4 h; Aurora backups and S3 versioning on | Restore test once per release | S |
| NFR-17 | Scalability | Handles growth without redesign | 500 clients, 5,000 users, 100 concurrent chat sessions (assumption A-05) | Load test | S |
| NFR-18 | Accessibility | Usable by people with disabilities | WCAG 2.2 AA | Automated accessibility tests plus a manual keyboard and screen-reader check | S |
| NFR-19 | Usability | A new Client Admin can find and download a certificate without help | ≤ 3 clicks from sign-in; task success ≥ 90% in user testing | Moderated usability test (5 users) | S |
| NFR-20 | Browser support | Works on current browsers and mobile | Latest 2 versions of Chrome, Edge, Safari, Firefox; responsive down to 375 px | Playwright cross-browser tests | S |
| NFR-21 | Observability | Problems and attacks are noticed quickly | Alerts within 5 min for repeated cross-tenant 403s, Guardrail blocks, error spikes and bulk downloads | GuardDuty, Security Hub, CloudWatch alarms to SNS | M |
| NFR-22 | Maintainability | Code is consistent and tested | TypeScript strict mode; lint clean; ≥ 80% unit coverage on services; all infrastructure in AWS CDK | CI quality gates | S |
| NFR-23 | Deployability | Repeatable, approved releases | All changes via pull request and pipeline; no manual console changes; Product Owner approval to deploy | GitHub branch protection, CDK diff in PR | M |
| NFR-24 | Cost | Stays within course budget | ≤ $20 a month in dev; AWS Budget alert at 80% | AWS Budgets | M |
| NFR-25 | Compliance | Aligns with US state insurance data security expectations (Virginia Bureau of Insurance) | Documented controls mapping; incident response steps written down | Compliance page in Confluence | S |
| NFR-26 | Data residency | All data stays in the US | Every resource in us-east-1; Bedrock model invoked in a US region | AWS Config rule, CDK review | S |
| NFR-28 | Performance – certificates | Standard certificate requests are fast | Auto-issued certificate ready ≤ 60 s from request; broker-approved requests answered within 4 business hours | Timing in end-to-end tests; broker queue SLA report | M |
| NFR-29 | Tenant onboarding | Adding a client is automated and repeatable | New client (bucket, key, knowledge base, database records) provisioned ≤ 15 min with no manual console steps | Provisioning runbook test | S |
| NFR-30 | Correctness – certificates | Certificates match the policy | 0 certificates showing limits, dates or insurers that differ from the policy record | Automated comparison test on every generated certificate | M |
| NFR-27 | Honesty (course) | Clearly fictional | Every page, seed file and sample PDF is labelled "SAMPLE – FICTIONAL" | Review checklist | M |

---

## 3. Future infrastructure requirements

Horizon: **Now** = needed for the course build · **Next** = year 1 (about 50 clients) · **Later** = year 2 (about 500 clients).

| ID | Need | Horizon | Driver | What it means for the architecture |
|---|---|---|---|---|
| FI-01 | Automated tenant provisioning | Now | NFR-29, A-05 | Adding a client creates its bucket, KMS key, knowledge base and database records from one workflow, not by editing code |
| FI-02 | Per-tenant AI retrieval at scale | Next | FR-23, NFR-01 | One knowledge base per client may hit AWS account quotas at hundreds of clients. Architect and security must choose between per-tenant knowledge bases (strong isolation) and a shared index with enforced tenant filters (cheaper, riskier) before year 1 |
| FI-03 | Certificate generation service | Now | FR-12, FR-33, NFR-28 | Server-side PDF generation from policy data on an ACORD 25-style template, with a certificate register |
| FI-04 | Email delivery | Now | FR-09, FR-38 | Transactional email (for example Amazon SES) with verified sending domain |
| FI-05 | Environments | Now: dev, test · Next: staging · Later: production | NFR-23 | Separate AWS accounts per environment under AWS Organizations from Next |
| FI-06 | Insurer integrations | Later | A-03 | An integration layer (queue plus adapters) for insurer data feeds, kept separate from the portal |
| FI-07 | E-signature | Later | FR-38 | Signing renewal questionnaires and broker forms |
| FI-08 | Single sign-on for larger clients | Later | FR-01 | Federate client identity providers into Cognito per tenant |
| FI-09 | Data growth and archiving | Next | NFR-11 | About 200 documents per client per year (about 100,000 a year at 500 clients); S3 lifecycle to archive storage, database partitioning by tenant |
| FI-10 | Disaster recovery in the US | Next | NFR-16, NFR-26 | Backups and S3 replication to a second US region |
| FI-11 | Broker reporting | Next | FR-20 | Book-of-business reports from a read replica, not the live database |
| FI-12 | Mobile use on job sites | Now: responsive · Later: installable web app | FR-32 | Certificate request flow must work one-handed on a phone |
| FI-13 | Central security logging | Next | NFR-10, NFR-21 | A separate log-archive account that application roles can't write over |

## 4. Traceability

| Area | Functional | Non-functional | Jira epic |
|---|---|---|---|
| Identity and roles | FR-01 – FR-06, FR-31, FR-32 | NFR-03, NFR-06 | KAN-4 |
| Client portal | FR-07 – FR-16, FR-31 – FR-39 | NFR-12, NFR-14, NFR-18 – NFR-20 | KAN-5 |
| Admin portal | FR-20 – FR-22 | NFR-12, NFR-18 | KAN-6 |
| Claims | FR-17 – FR-19 | NFR-12 | KAN-7 |
| AI chatbot | FR-23 – FR-27 | NFR-07, NFR-08, NFR-13 | KAN-8 |
| Tenant isolation and security | FR-04, FR-14, FR-26 | NFR-01, NFR-02, NFR-04, NFR-05, NFR-09, NFR-21 | KAN-9 |
| Compliance and audit | FR-28 – FR-30 | NFR-10, NFR-11, NFR-25, NFR-26 | KAN-10 |
| Platform and DevOps | – | NFR-15 – NFR-17, NFR-22 – NFR-24 | KAN-11 |

**Requirements with no Jira ticket yet:** FR-06, FR-08, FR-10, FR-12, FR-16, FR-19, FR-21, FR-27, FR-30, FR-31 – FR-36, FR-38, FR-39. Each will be raised as a story with Given/When/Then acceptance criteria once priorities are confirmed.

---

## 5. Open questions

1. Confirm or overrule assumptions A-01 to A-06.
2. Confirm retention periods (NFR-11) with the Compliance Lead.
3. FI-02: per-tenant knowledge bases versus a shared index with tenant filters. Decision needed from the architect and security before year 1.
