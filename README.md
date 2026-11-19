
# LedgerGuard --- Backend Development Roadmap

> **Project:** LedgerGuard --- Dynamic Multi-Tenant Zero-Knowledge
> Billing Engine\
> **Role:** Backend Developer\
> **Backend branch:** `backend`\
> **Stack:** Node.js, Express.js, MongoDB, Mongoose, Redis, JWT,
> RSA-256\
> **Project duration:** 4-week implementation plan

------------------------------------------------------------------------

## 1. Project Overview

LedgerGuard is a multi-tenant billing backend designed to keep each
organization's data isolated and to make billing transactions reliable,
even when requests are repeated or a network failure occurs.

The backend must:

1.  Authenticate incoming requests and identify the organization
    (tenant).
2.  Route each request to the correct tenant database connection.
3.  Keep tenant data isolated from every other tenant.
4.  Prevent duplicate billing operations using idempotency controls.
5.  Use Redis distributed locks and MongoDB transactions to protect
    ledger updates.
6.  Provide usage and cost analytics through MongoDB aggregation
    queries.
7.  Validate request authenticity using RSA-256 asymmetric cryptography.
8.  Expose documented REST APIs and support the frontend dashboard.

### Main use case

Company A and Company B use LedgerGuard at the same time. The API
gateway identifies the organization from a verified JWT and routes the
request to that organization's isolated database. If a billing request
is retried or a network timeout occurs, idempotency controls, Redis
locks, and MongoDB transactions help prevent duplicate charges and
inconsistent ledger records.

------------------------------------------------------------------------

## 2. My Role and Ownership

I am responsible for the server-side implementation. My work includes
the following modules.

  -----------------------------------------------------------------------
  Module                              My responsibility
  ----------------------------------- -----------------------------------
  API Gateway                         Build Express middleware and routes
                                      that authenticate requests and
                                      identify tenants.

  Authentication                      Verify JWTs and enforce
                                      authorization for protected
                                      endpoints.

  Tenant Database Pool                Create, cache, select, and safely
                                      manage tenant-specific Mongoose
                                      connections.

  Billing Ledger                      Store billing events and ledger
                                      entries with reliable transaction
                                      handling.

  Idempotency                         Detect repeated requests using
                                      unique event/request IDs.

  Redis Locks                         Prevent conflicting operations from
                                      executing simultaneously where
                                      required.

  MongoDB Transactions                Make related ledger writes atomic
                                      and roll back failed operations.

  Analytics                           Build aggregation pipelines for
                                      usage and cost metrics over
                                      historical time ranges.

  Cryptographic Security              Validate signed requests/payloads
                                      using RSA-256 as required by the
                                      project.

  API Integration                     Agree on request/response contracts
                                      with the frontend developer.

  Testing                             Write unit, integration, security,
                                      and failure-case tests.

  Documentation                       Maintain setup instructions,
                                      environment variable documentation,
                                      API documentation, and progress
                                      notes.
  -----------------------------------------------------------------------

**Important:** The frontend dashboard is outside my primary ownership. I
will provide stable APIs and coordinate with the frontend developer on
payload formats, authentication, errors, and integration.

------------------------------------------------------------------------

## 3. Technology Stack

  -----------------------------------------------------------------------
  Technology                          How I will use it
  ----------------------------------- -----------------------------------
  Node.js                             JavaScript runtime for the backend.

  Express.js                          REST API server, routing, and
                                      middleware.

  MongoDB                             Store tenant data, billing events,
                                      ledger entries, and usage records.

  Mongoose                            Define schemas, indexes,
                                      validation, and tenant-specific
                                      connections.

  Redis                               Distributed locks and supporting
                                      idempotency coordination.

  JWT                                 Authenticate requests and identify
                                      the organization/tenant.

  RSA-256                             Asymmetric signature validation for
                                      protected API requests.

  Jest                                Unit and integration test
                                      framework.

  Supertest                           Test Express endpoints.

  dotenv                              Load local environment
                                      configuration.

  Git and GitHub                      Branch-based collaboration and
                                      version control.
  -----------------------------------------------------------------------

Use asynchronous code (`async`/`await`) for database, Redis, and network
operations. Keep modules separated by responsibility. Use JSON for REST
API communication.

------------------------------------------------------------------------

## 4. Suggested Backend Architecture

``` text
Client / Frontend
       |
       v
Express API Gateway
       |
       +--> Request validation
       |
       +--> RSA-256 signature verification (where required)
       |
       +--> JWT authentication and authorization
       |
       +--> Tenant identification
       |
       v
Tenant Database Manager
       |
       +--> Tenant A MongoDB connection
       +--> Tenant B MongoDB connection
       +--> Tenant C MongoDB connection
       |
       v
Billing / Ledger Services
       |
       +--> Idempotency check
       +--> Redis distributed lock
       +--> MongoDB transaction
       +--> Ledger and usage updates
       |
       v
Analytics / Aggregation Services
       |
       v
JSON API Response to Frontend
```

The exact order of signature verification and authentication should be
finalized with the API security contract. Never trust a tenant ID
supplied only in an unverified request body or query parameter.

------------------------------------------------------------------------

## 5. Suggested Folder Structure

This is a proposed structure; adapt it to the team's existing repository
conventions before creating duplicate folders.

``` text
backend/
├── src/
│   ├── config/
│   │   ├── env.js
│   │   ├── redis.js
│   │   └── database.js
│   │
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── ledger.controller.js
│   │   └── analytics.controller.js
│   │
│   ├── middleware/
│   │   ├── authenticate.js
│   │   ├── authorize.js
│   │   ├── identifyTenant.js
│   │   ├── verifySignature.js
│   │   ├── validateRequest.js
│   │   └── errorHandler.js
│   │
│   ├── models/
│   │   ├── ledger.model.js
│   │   ├── billingEvent.model.js
│   │   └── usageRecord.model.js
│   │
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── ledger.routes.js
│   │   └── analytics.routes.js
│   │
│   ├── services/
│   │   ├── tenantConnection.service.js
│   │   ├── ledger.service.js
│   │   ├── idempotency.service.js
│   │   ├── lock.service.js
│   │   ├── transaction.service.js
│   │   └── analytics.service.js
│   │
│   ├── utils/
│   │   ├── ApiError.js
│   │   └── asyncHandler.js
│   │
│   ├── app.js
│   └── server.js
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
│
├── scripts/
├── docs/
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

If the repository is a monorepo, keep the backend inside the agreed
backend directory and do not move or rename shared project files without
team agreement.

------------------------------------------------------------------------

## 6. Implementation Roadmap

The project document defines a four-week backend plan. The tasks below
translate those milestones into practical development steps.

### Week 1 --- API Gateway and Dynamic Tenant Provisioning

**Project milestone:** Build the API gateway for JWT decoding and
implement runtime Mongoose connection switching for isolated tenant
databases.

#### Tasks

-   [ ] Inspect the existing repository, package manager, coding
    conventions, and frontend integration needs.
-   [ ] Set up the Node.js and Express server.
-   [ ] Add environment configuration using `.env` and provide
    `.env.example`.
-   [ ] Configure the base MongoDB/Mongoose setup.
-   [ ] Define how tenant identity is represented and validated.
-   [ ] Implement JWT verification middleware.
-   [ ] Implement tenant identification middleware using verified
    identity/claims.
-   [ ] Build a tenant connection manager that selects the correct
    database connection.
-   [ ] Implement connection caching and safe connection lifecycle
    management.
-   [ ] Create initial tenant-aware schemas/models.
-   [ ] Add health-check and basic error-handling endpoints.
-   [ ] Test that Tenant A cannot access Tenant B's data.

#### How I will implement it

1.  Start with a small Express application and a central configuration
    module.
2.  Define the authentication contract with the team: token issuer,
    signing/verification method, required claims, expiry, and tenant
    claim.
3.  Verify the token before using its tenant information.
4.  Create a tenant connection service that maps an authenticated tenant
    identifier to an approved database configuration.
5.  Ensure every tenant-scoped query uses models/collections from the
    selected tenant connection.
6.  Add tests using at least two separate tenants and deliberately
    attempt cross-tenant access.

**Week 1 output:** A running Express API with JWT-protected routes and
verified tenant-specific database selection.

**Do not:** Select a database using an arbitrary client-supplied
database name or tenant ID without validating it against authenticated
identity and server-side configuration.

### Week 2 --- Idempotent Ledger and Transaction Safety

**Project milestone:** Structure Redis distributed locks and unique
event IDs. Wrap ledger logic in MongoDB multi-document transactions.

#### Tasks

-   [ ] Configure Redis connectivity.
-   [ ] Design billing event and ledger data models.
-   [ ] Decide the idempotency key/event ID contract with the team.
-   [ ] Add a unique database index for the appropriate tenant-scoped
    event identifier.
-   [ ] Implement idempotency checks for repeated requests.
-   [ ] Implement Redis lock acquisition, expiry, and safe release.
-   [ ] Define the critical section for each billing operation.
-   [ ] Use MongoDB sessions and multi-document transactions for related
    writes.
-   [ ] Handle transaction failure and rollback paths.
-   [ ] Return consistent success, duplicate, conflict, and failure
    responses.
-   [ ] Test simultaneous requests with the same idempotency key.

#### How I will implement it

1.  Require a stable idempotency key or unique event ID for operations
    that must not be applied twice.
2.  Scope uniqueness correctly to the tenant and operation as required
    by the data model.
3.  Check/record idempotency in a way that remains safe under concurrent
    requests; a unique database constraint is an important final
    safeguard.
4.  Use a Redis lock with a bounded expiry and a unique lock-owner
    token. Release only a lock owned by the current operation.
5.  Perform all related MongoDB writes in one transaction/session where
    supported by the deployment.
6.  On failure, abort the transaction and release the lock safely.
7.  Test retries, concurrent duplicates, timeouts, and failures between
    writes.

**Week 2 output:** A ledger flow that prevents duplicate processing and
keeps related database writes consistent.

**Important implementation note:** Redis locks alone do not guarantee
exactly-once billing. Use database uniqueness and transaction-safe
idempotency as well. MongoDB multi-document transactions require a
transaction-capable deployment, such as a replica set or supported
sharded cluster; verify the development environment early.

### Mid-Project Review --- Isolation Audit

The project document requires an isolation audit and mock trace tests
showing fast database switching under load.

-   [ ] Present the tenant routing and database connection architecture.
-   [ ] Demonstrate that two tenants remain isolated.
-   [ ] Show connection selection/switching traces under load.
-   [ ] Demonstrate duplicate-request protection.
-   [ ] Demonstrate a failed transaction and rollback.
-   [ ] Record test setup, results, and issues.
-   [ ] Review findings with the mentor/team and update the plan.

### Week 3 --- Analytics Pipeline

**Project milestone:** Design MongoDB aggregation queries to return
real-time usage cost metrics over historical time scopes.

#### Tasks

-   [ ] Agree on the usage, cost, and time-range definitions with the
    team.
-   [ ] Confirm which billing/usage records are the source of truth.
-   [ ] Design tenant-scoped aggregation pipelines.
-   [ ] Add appropriate indexes for tenant and time-based queries.
-   [ ] Implement analytics service functions.
-   [ ] Create API endpoints for dashboard metrics.
-   [ ] Support agreed historical ranges and filters.
-   [ ] Validate aggregation output against known sample records.
-   [ ] Test performance and ensure tenant filtering is applied
    throughout.

#### How I will implement it

1.  Confirm the metric definitions and response fields with the frontend
    developer.
2.  Filter by the authenticated tenant before calculating metrics.
3.  Use MongoDB aggregation stages such as `$match`, `$group`,
    `$project`, and date grouping where appropriate.
4.  Add indexes based on actual query patterns, for example tenant
    identifier plus event timestamp.
5.  Compare API results with manually calculated sample data.
6.  Measure query performance and tune indexes/pipelines based on
    explain plans and realistic data.

**Week 3 output:** Tenant-safe analytics APIs that return usage and cost
metrics for the dashboard.

### Week 4 --- RSA-256 Security, Trace Logging, Testing, and Deployment

**Project milestone:** Lock endpoints using RSA-256 validation. Conduct
end-to-end trace logging and secure deployments.

#### Tasks

-   [ ] Agree on the RSA signature format and canonical payload format.
-   [ ] Configure public-key verification and secure key management.
-   [ ] Implement signature validation middleware for the endpoints that
    require it.
-   [ ] Reject invalid, malformed, or tampered signatures.
-   [ ] Define timestamp/nonce handling if replay protection is part of
    the agreed protocol.
-   [ ] Review JWT authorization and tenant-isolation rules.
-   [ ] Avoid logging secrets, private keys, tokens, or sensitive
    billing data.
-   [ ] Add request/trace identifiers for debugging.
-   [ ] Complete unit and integration tests using Jest and Supertest.
-   [ ] Test transaction rollback, duplicate requests, and cross-tenant
    access.
-   [ ] Prepare environment variables and deployment configuration.
-   [ ] Document setup, APIs, test commands, and known limitations.
-   [ ] Prepare the final backend demonstration.

#### How I will implement it

1.  Obtain the public-key and signing requirements from the project/team
    contract. Never invent a signature format.
2.  Use Node.js cryptographic APIs or an approved library to verify
    signatures using the public key.
3.  Ensure the exact bytes/canonical representation being verified are
    agreed by both client and server.
4.  Keep private keys and secrets out of source control.
5.  Add negative tests for invalid signatures, expired/replayed requests
    where applicable, and tampered payloads.
6.  Run the complete test suite and document the results.

**Week 4 output:** A tested, documented, and securely configured backend
ready for final review and integration.

------------------------------------------------------------------------

## 7. Proposed API Contract

The project brief does not prescribe exact endpoint names, request
fields, or response schemas. The following is a **starting proposal** to
discuss with the team and finalize before implementation.

  ---------------------------------------------------------------------------
  Method                  Proposed endpoint           Purpose
  ----------------------- --------------------------- -----------------------
  GET                     `/api/health`               Check server health.

  GET                     `/api/auth/me`              Return authenticated
                                                      user and tenant
                                                      context.

  POST                    `/api/ledger/entries`       Submit a billing/ledger
                                                      operation with an
                                                      idempotency key.

  GET                     `/api/ledger/entries`       Retrieve the
                                                      authenticated tenant's
                                                      ledger entries.

  GET                     `/api/ledger/entries/:id`   Retrieve one
                                                      tenant-scoped ledger
                                                      entry.

  GET                     `/api/analytics/usage`      Return usage metrics
                                                      for an agreed time
                                                      range.

  GET                     `/api/analytics/costs`      Return cost metrics for
                                                      an agreed time range.
  ---------------------------------------------------------------------------

### API design rules

-   All protected routes must authenticate the caller.
-   Tenant identity must come from verified authentication context, not
    an untrusted client field.
-   Validate request bodies, parameters, and query strings.
-   Use consistent HTTP status codes and JSON response shapes.
-   Do not expose database connection details, secrets, stack traces, or
    another tenant's records.
-   Define pagination, sorting, date range, and filtering behavior with
    the frontend developer.
-   Document required headers, including authorization, signature, and
    idempotency headers where applicable.

### Example response shape (proposal)

``` json
{
  "success": true,
  "message": "Ledger entry processed",
  "data": {
    "entryId": "example-entry-id",
    "status": "processed"
  }
}
```

This is illustrative only. Final field names and statuses must be agreed
with the frontend and project team.

------------------------------------------------------------------------

## 8. Data Model Planning

The project document requires tenant isolation and a reliable billing
ledger, but it does not define final schemas. The following are planning
considerations, not finalized models.

### Billing event

Potential fields:

-   Tenant-scoped event identifier
-   Idempotency key
-   Event type
-   Amount and currency
-   Event timestamp
-   Processing status
-   Reference/metadata needed for audit and reconciliation

### Ledger entry

Potential fields:

-   Entry identifier
-   Tenant context
-   Related billing event identifier
-   Debit/credit or transaction direction, as defined by the billing
    rules
-   Amount and currency
-   Created timestamp
-   Transaction/reference status

### Usage record

Potential fields:

-   Tenant context
-   Resource or usage category
-   Quantity and unit
-   Recorded timestamp
-   Pricing reference or calculated cost, according to agreed business
    rules

### Database rules

-   Add indexes for real query patterns.
-   Enforce uniqueness for idempotency/event identifiers at the database
    level.
-   Keep tenant scoping consistent in reads, writes, updates, and
    deletes.
-   Avoid trusting tenant identifiers from request payloads.
-   Decide retention, audit, and reconciliation requirements with the
    mentor/team.

------------------------------------------------------------------------

## 9. Security Checklist

-   [ ] Verify JWT signatures and expiry.
-   [ ] Validate authorization and tenant claims.
-   [ ] Apply tenant isolation to every tenant-scoped operation.
-   [ ] Never accept arbitrary database names from clients.
-   [ ] Validate RSA-256 signatures on the agreed protected routes.
-   [ ] Store secrets and keys in environment/secret-management systems.
-   [ ] Commit `.env.example`, never the real `.env`.
-   [ ] Validate and sanitize all incoming data.
-   [ ] Use safe Redis lock ownership, expiry, and release behavior.
-   [ ] Enforce database-level uniqueness for idempotency.
-   [ ] Use MongoDB transactions for related ledger writes.
-   [ ] Avoid logging tokens, keys, passwords, or sensitive billing
    details.
-   [ ] Return safe error messages without internal implementation
    details.
-   [ ] Test cross-tenant access attempts and invalid authentication.

------------------------------------------------------------------------

## 10. Testing and Performance Validation

The project brief calls for Jest and Supertest and sets performance
expectations, including sub-5ms database index queries. Treat the
performance target as a benchmark to measure and report under a
documented test setup, not as an assumed result.

### Unit tests

-   JWT and tenant middleware behavior.
-   Tenant connection selection.
-   Idempotency service behavior.
-   Redis lock ownership and release logic.
-   Signature verification.
-   Analytics aggregation transformations.

### Integration tests

-   API authentication and authorization.
-   Tenant A and Tenant B data isolation.
-   Ledger creation and retrieval.
-   Duplicate event submission.
-   Concurrent requests with the same idempotency key.
-   MongoDB transaction commit and rollback.
-   Redis unavailable/timeout behavior.
-   Invalid or tampered RSA-signed requests.
-   Analytics results for known sample data.

### Example test scenarios

  -----------------------------------------------------------------------
  Scenario                            Expected result
  ----------------------------------- -----------------------------------
  Valid token for Tenant A            Request uses Tenant A context.

  Tenant A requests Tenant B record   Access is denied or record is not
                                      found, according to the API policy.

  Same billing event submitted twice  It is not applied twice.

  Two simultaneous requests use the   Only one logical operation is
  same key                            committed.

  A transaction fails midway          Related MongoDB writes are rolled
                                      back.

  Invalid signature                   Protected request is rejected.

  Analytics request omits tenant      Request is rejected.
  context                             
  -----------------------------------------------------------------------

### Performance evidence to collect

-   Query latency and explain-plan/index evidence.
-   API throughput and latency under concurrent requests.
-   Redis lock contention and timeout behavior.
-   Tenant connection switching behavior under load.
-   Test data volume, environment, and benchmark commands.

Do not claim a benchmark has passed until it has actually been run and
its result recorded.

------------------------------------------------------------------------

## 11. Git Workflow

I have already created the `backend` branch. I will use it for backend
implementation and avoid committing directly to the shared main branch.

### Initial checks

``` bash
git status
git branch
git switch backend
git pull origin backend
```

If the remote branch does not exist yet, coordinate with the team before
pushing it.

### Commit workflow

``` bash
git add .
git commit -m "feat(backend): add Express server setup"
git push origin backend
```

Use small, meaningful commits. Suggested commit prefixes:

-   `feat:` new backend functionality
-   `fix:` bug fixes
-   `test:` tests
-   `refactor:` code restructuring without intended behavior change
-   `docs:` documentation
-   `chore:` tooling/configuration

### Before opening a pull request

-   [ ] Pull/rebase the latest agreed base branch.
-   [ ] Run linting and tests.
-   [ ] Check that `.env` and private keys are not staged.
-   [ ] Review the diff for unrelated changes.
-   [ ] Explain the feature, test evidence, and any known limitations.
-   [ ] Request review from the assigned teammate/mentor.

Coordinate the target branch and merge strategy with the repository
owner. Do not force-push shared branches or overwrite teammates' work.

------------------------------------------------------------------------

## 12. Local Setup --- Planned Instructions

The exact commands depend on the repository's existing package manager
and scripts. Once the backend package is created, document the verified
commands here.

Typical setup sequence:

``` bash
# Move into the backend directory
cd backend

# Install dependencies using the repository's chosen package manager
npm install

# Create a local environment file from the example
cp .env.example .env

# Start the development server (after the script is configured)
npm run dev
```

### Environment variables

Create `.env.example` with variable names and safe placeholders only.
Confirm exact names and formats during implementation.

``` dotenv
NODE_ENV=development
PORT=5000

# MongoDB
MONGODB_URI=

# Redis
REDIS_URL=

# Authentication
JWT_PUBLIC_KEY=
JWT_ISSUER=
JWT_AUDIENCE=

# RSA request-signature verification
RSA_PUBLIC_KEY=

# Optional: only if the agreed tenant architecture uses a registry/configuration store
TENANT_REGISTRY_URI=
```

Do not commit real credentials, tokens, private keys, production
database URIs, or populated secret files. If the JWT design uses a
shared secret rather than an asymmetric key, document the actual agreed
configuration instead of keeping unused variables.

------------------------------------------------------------------------

## 13. Frontend Integration Responsibilities

I will coordinate with the frontend developer before finalizing APIs.

We need to agree on:

-   Authentication method and token format.
-   How tenant context is established.
-   Required request headers.
-   RSA signature requirements, if the frontend/client signs requests.
-   Ledger request and response fields.
-   Idempotency key generation and retry behavior.
-   Analytics filters, date ranges, and response formats.
-   Loading, validation, authorization, conflict, and server error
    responses.
-   CORS and local development URLs.

Share API examples and endpoint changes early. Avoid changing response
fields without notifying the frontend developer.

------------------------------------------------------------------------

## 14. Deliverables and Review Preparation

The assignment letter asks for source code, project documentation,
project report, presentation, model evaluation results if applicable,
and a final project demonstration if applicable.

### Backend deliverables

-   [ ] Working backend source code.
-   [ ] Express API and middleware.
-   [ ] Tenant-aware database connection management.
-   [ ] Billing/ledger and idempotency implementation.
-   [ ] Redis locking and MongoDB transaction handling.
-   [ ] Analytics aggregation APIs.
-   [ ] RSA-256 request validation as agreed.
-   [ ] Jest/Supertest test suite.
-   [ ] `.env.example` and setup documentation.
-   [ ] API documentation and frontend integration notes.
-   [ ] Test and performance evidence.
-   [ ] Backend architecture explanation for the review.
-   [ ] Final demonstration of tenant isolation and safe billing
    behavior.

### Mid-project review

Be ready to demonstrate:

1.  How the API identifies a tenant.
2.  How requests select the correct database connection.
3.  How isolation is enforced.
4.  How duplicate requests are detected.
5.  How Redis locks and MongoDB transactions work together.
6.  What happens when a transaction fails.

### Final review

Be ready to demonstrate:

1.  Secure JWT and RSA-256 validation.
2.  Tenant isolation.
3.  Idempotent ledger operations.
4.  Transaction rollback.
5.  Usage and cost analytics.
6.  Test coverage and measured performance.
7.  Setup and API documentation.

------------------------------------------------------------------------

## 15. Working Principles

-   Read the existing repository before changing its structure.
-   Confirm requirements and API contracts with the team/mentor.
-   Implement one module at a time and test it before integration.
-   Keep controllers thin; put business logic in services.
-   Keep database access and tenant connection selection centralized.
-   Use clear validation and centralized error handling.
-   Do not use `any` as a shortcut if the repository uses strict
    TypeScript.
-   Keep code modular and maintainable.
-   Write tests for both successful and failure paths.
-   Update documentation when behavior or API contracts change.
-   Send periodic progress updates to the assigned mentor.
-   Keep confidential project information and credentials secure.

------------------------------------------------------------------------

## 16. Progress Tracker

Update this section as work is completed.

  Milestone                             Status        Notes / evidence
  ------------------------------------- ------------- ------------------
  Repository and backend setup          Not started   
  Express server and configuration      Not started   
  JWT authentication                    Not started   
  Tenant identification and isolation   Not started   
  Dynamic Mongoose connection manager   Not started   
  Billing and ledger schemas            Not started   
  Idempotency and unique constraints    Not started   
  Redis distributed locks               Not started   
  MongoDB transactions and rollback     Not started   
  Mid-project isolation review          Not started   
  Analytics aggregation APIs            Not started   
  RSA-256 signature validation          Not started   
  Unit and integration tests            Not started   
  Performance testing and evidence      Not started   
  Frontend integration                  Not started   
  Documentation and final demo          Not started   

------------------------------------------------------------------------

## 17. Important Scope Notes

-   The project specification defines the main backend modules and
    four-week milestones. It does **not** provide final API routes,
    exact JWT claims, database schemas, pricing rules, RSA signature
    format, or deployment provider. These must be agreed with the
    mentor/team.
-   "Zero-knowledge" appears in the project title, but the supplied
    project brief does not define a specific zero-knowledge proof
    protocol or cryptographic data-privacy design. Do not claim that
    such a protocol has been implemented unless it is separately
    specified and built.
-   RSA-256 is the terminology used in the project brief. Confirm the
    precise signature algorithm, padding, key format, and payload
    canonicalization with the team before implementation.
-   Dynamic tenant database provisioning and connection switching must
    be designed around the actual database hosting arrangement and
    tenant registry.
-   MongoDB transaction support depends on deployment topology. Confirm
    this before implementing the ledger transaction flow.
-   Performance goals must be measured and documented using reproducible
    tests.

------------------------------------------------------------------------

## 18. Reference Documents

This roadmap is based on the supplied project and assignment documents:

1.  **PROJECT.pdf** --- Project 3, "LedgerGuard: Dynamic Multi-Tenant
    Zero-Knowledge Billing Engine," especially pages 4--6 for modules,
    weekly milestones, technology stack, and evaluation standards.
2.  **Assignment letter** --- Confirms the assigned project, internship
    scope, expected deliverables, and performance evaluation criteria.

------------------------------------------------------------------------

**Owner:** Backend Developer\
**Working branch:** `backend`\
**Project:** LedgerGuard\
**Status:** Development roadmap --- update as implementation decisions
are finalized.
