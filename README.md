# portfolio-360
Portfolio360 — Exact Project Implementation Plan
Version 1.0 · Database-first development · Solo developer · India-first pilot
Based on everything discussed in this chat, Portfolio360 should be built as a secure, multi-tenant student portfolio platform that becomes the foundation for a future education ERP.
The immediate goal is not to build the entire ERP or prove five-million-student capacity. It is to build a working system for 1–3 pilot organizations with more than 10,000 students, validate the complete workflow, and scale based on measured results.
1. Final project scope

Multi-organization management
Support schools, colleges, preschools, coaching centres, and other education providers. Each organization can have multiple campuses, academic years, classes, sections, and students.




Excel processing
Accept standard and custom Excel formats, map columns, validate records, detect duplicates, preview changes, obtain administrator approval, and process imports in batches.




Student portfolios
Generate PDFs, printable reports, authenticated web portfolios, and downloadable ZIP packages. Support custom templates, organization branding, multilingual content, academic history, attendance, achievements, photos, certificates, and permitted guardian details.




Security and governance
Enforce tenant isolation, role-based access, administrator MFA, audit logging, private file storage, encryption, controlled downloads, backups, and configurable retention policies.



2. Technology and database decision
Your requirement is a NoSQL database with table-style access, but you also require full SQL and relational transactions. We must resolve that conflict before locking the database.
The concrete plan is:
- Primary candidate: PostgreSQL with JSONB for flexible organization-specific fields and portfolio data.
- Alternative if a dedicated NoSQL database is mandatory: PostgreSQL + MongoDB, with PostgreSQL authoritative for student identity, enrollments, permissions, approvals, and other transactional records.
- Evaluation alternatives: CockroachDB and YugabyteDB if distributed SQL becomes necessary.
- Backend: Python, Django, and Django REST Framework, subject to the database integration decision.
- Frontend: React and TypeScript.
- Background processing: Celery and Redis or an equivalent task-queue design.
- File storage: Private object storage for Excel files, photos, certificates, PDFs, and ZIP exports.
- Deployment: Docker-based development and an India-region pilot deployment, subject to hosting and data-residency requirements.
Do not implement two database systems until the comparison establishes that the second database is genuinely required. The logical data model can be designed now, while the physical schema and migration framework are finalized after the comparison.
3. Exact development phases
The order below is the implementation sequence. The durations are planning estimates for one developer, not delivery guarantees.
01

Requirements and database selection
Start here
- Document the organization, student, enrollment, class, section, and academic-year relationships.
- Compare PostgreSQL + JSONB, PostgreSQL + MongoDB, and distributed SQL alternatives.
- Define student identity and duplicate-resolution rules.
- Prepare synthetic test data and representative import workloads.
- Record the decision in docs/decisions/ADR-001-database-selection.md.
Deliverable: approved database decision, logical data model, and initial schema design.



02

Database and security foundation
- Implement organizations, campuses, academic years, classes, sections, students, enrollments, users, memberships, and audit events.
- Add foreign keys, unique constraints, indexes, and schema migrations.
- Implement organization-scoped authorization.
- Add automated tests for cross-tenant reads and writes.
- Establish private file storage, secrets management, and backup procedures.
Deliverable: reproducible schema with verified organization isolation.



03

Excel import engine
- Upload and privately store Excel files.
- Inspect sheets, map columns, and validate data types.
- Detect missing identifiers, duplicate admissions, invalid class references, and conflicting updates.
- Stage data before committing it.
- Provide an import preview and administrator approval.
- Write records in batches, track row-level failures, and support safe retries.
Deliverable: approved imports create or update the correct student records without duplicate side effects.



04

Portfolio template and generation engine
- Build the first portfolio template from scratch.
- Support configurable fields, branding, locale, and template versions.
- Fetch authorized student and academic data.
- Render HTML/CSS into PDFs using background workers.
- Add generation status, progress, retry handling, and stale-version detection.
- Implement administrator preview and publication approval.
Deliverable: a correctly generated, versioned PDF portfolio for each eligible student.



05

Student and parent access
- Build authenticated student access.
- Add parent/guardian relationships with a verified linking process.
- Apply resource-level authorization to portfolio pages and documents.
- Support private downloads and authorized printable reports.
- Generate ZIP exports asynchronously, with access controls and expiry.
Deliverable: students and verified parents can view only the portfolios they are authorized to access.



06

Pilot and performance validation
- Test with 1–3 organizations and more than 10,000 synthetic or appropriately authorized student records.
- Test large imports, simultaneous generation jobs, database queries, retries, and export workflows.
- Resolve critical security issues.
- Validate backups and restoration.
- Measure costs and establish performance targets from real workloads.
Deliverable: a limited pilot with documented capacity and known limitations.



4. Database entities to implement first
Use these as the initial data-model scope.
Domain	Entities
Organization	Organization, campus, academic year
Academic structure	Class, section, enrollment
Identity	User, organization membership, role, student
Student information	Academic record, attendance, activity, guardian relationship
Files	Document metadata, private storage reference
Import	Import job, row staging, validation error, approval
Portfolios	Template, template version, portfolio, publication approval
Background work	Generation job, export job, retry state
Security	Audit event, access event, administrative action
Keep growing data—such as attendance, exam attempts, and document metadata—in separate records. Use flexible JSON/document fields only where the structure varies, rather than storing every student attribute and history in one unbounded document.
5. Repository structure
portfolio360/
├── docs/
│   ├── requirements.md
│   ├── architecture.md
│   ├── database-design.md
│   ├── security-threat-model.md
│   └── decisions/
│       └── ADR-001-database-selection.md
├── database/
│   ├── migrations/
│   ├── seeds/
│   └── tests/
├── backend/
│   ├── config/
│   ├── apps/
│   │   ├── organizations/
│   │   ├── identity/
│   │   ├── academics/
│   │   ├── students/
│   │   ├── imports/
│   │   ├── templates/
│   │   ├── portfolios/
│   │   ├── documents/
│   │   └── audit/
│   └── tests/
├── frontend/
│   └── src/
│       ├── features/
│       ├── components/
│       └── services/
├── deployment/
│   ├── Dockerfile.backend
│   └── backup-restore.md
├── docker-compose.yml
├── .env.example
└── README.md


If Django is selected with PostgreSQL, use Django migrations as the authoritative schema migration mechanism. Do not maintain a second, conflicting migration system.
6. Security requirements before real data
The pilot must enforce these controls before student data is uploaded:
- Every query and file access is scoped to the authenticated user's organization and permitted records.
- Admission identifiers and enrollment rules are validated by the database and application.
- Excel files are validated and processed in bounded batches.
- Student portfolios and uploaded documents are stored privately.
- Administrator MFA, secure sessions, least-privilege permissions, and audit logging are enabled.
- Database credentials and storage credentials are never committed to the repository.
- Backups are protected and restoration is tested.
- Retention and deletion behavior is documented before production launch.
Because this system may process children's information, confirm the applicable Indian data-protection requirements and any organization-specific obligations before the pilot uses real records.
7. Performance and acceptance criteria
The pilot target is more than 10,000 students across 1–3 organizations. This does not establish production capacity for five million students or tens of thousands of concurrent users.
The initial tests must measure:
Area	Acceptance requirement
Data integrity	Invalid relationships and duplicate identities are rejected or flagged
Tenant security	Cross-organization access tests show no unauthorized reads or writes
Imports	Repeated jobs do not duplicate committed records
Background work	Failed jobs can be retried without duplicating side effects
Portfolios	PDF content matches the approved source data and template version
Private access	Unauthorized users cannot view or download portfolio files
Performance	Query latency and import throughput are measured and documented
Recovery	Database and file restoration are tested
Production readiness	Critical security findings are resolved
