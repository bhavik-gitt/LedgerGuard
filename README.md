# LedgerGuard: Backend Development Roadmap

| | |
|---|---|
| **Project** | LedgerGuard: Dynamic Multi-Tenant Zero-Knowledge Billing Engine |
| **Role** | Backend Developer |
| **Working branch** | `backend` |
| **Stack** | Node.js, Express.js, MongoDB, Mongoose, Redis, JWT, RSA-256 |
| **Duration** | 4-week implementation plan with a mid-project and a final review |
| **Document status** | Living roadmap. Update as implementation decisions are finalized. |

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Ownership and Responsibilities](#2-ownership-and-responsibilities)
3. [Technology Stack](#3-technology-stack)
4. [Backend Architecture](#4-backend-architecture)
5. [Folder Structure](#5-folder-structure)
6. [Implementation Roadmap](#6-implementation-roadmap)
7. [Proposed API Contract](#7-proposed-api-contract)
8. [Data Model Planning](#8-data-model-planning)
9. [Security Checklist](#9-security-checklist)
10. [Testing and Performance Validation](#10-testing-and-performance-validation)
11. [Git Workflow](#11-git-workflow)
12. [Local Setup](#12-local-setup)
13. [Frontend Integration](#13-frontend-integration)
14. [Deliverables and Review Preparation](#14-deliverables-and-review-preparation)
15. [Working Principles](#15-working-principles)
16. [Progress Tracker](#16-progress-tracker)
17. [Scope Notes and Open Decisions](#17-scope-notes-and-open-decisions)
18. [Reference Documents](#18-reference-documents)

---

## 1. Project Overview

LedgerGuard is a multi-tenant billing backend. It keeps each organization's data isolated and makes billing transactions reliable, even when requests are repeated or a network failure occurs.

### Backend objectives

1. Authenticate incoming requests and identify the organization (tenant).
2. Route each request to the correct tenant database connection.
3. Keep tenant data isolated from every other tenant.
4. Prevent duplicate billing operations through idempotency controls.
5. Protect ledger updates with Redis distributed locks and MongoDB transactions.
6. Provide usage and cost analytics through MongoDB aggregation queries.
7. Validate request authenticity using RSA-256 asymmetric cryptography.
8. Expose documented REST APIs that support the frontend dashboard.

### Primary use case

Company A and Company B use LedgerGuard at the same time. The API gateway identifies each organization from a verified JWT and routes the request to that organization's isolated database. If a billing request is retried or a network timeout occurs, idempotency controls, Redis locks, and MongoDB transactions prevent duplicate charges and inconsistent ledger records.

---

## 2. Ownership and Responsibilities

The backend developer owns the server-side implementation.

| Module | Responsibility |
|---|---|
| **API Gateway** | Build Express middleware and routes that authenticate requests and identify tenants |
| **Authentication** | Verify JWTs and enforce authorization on protected endpoints |
| **Tenant Database Pool** | Create, cache, select, and safely manage tenant-specific Mongoose connections |
| **Billing Ledger** | Store billing events and ledger entries with reliable transaction handling |
| **Idempotency** | Detect repeated requests using unique event or request IDs |
| **Redis Locks** | Prevent conflicting operations from running simultaneously where required |
| **MongoDB Transactions** | Make related ledger writes atomic and roll back failed operations |
| **Analytics** | Build aggregation pipelines for usage and cost metrics over historical ranges |
| **Cryptographic Security** | Validate signed requests and payloads using RSA-256 |
| **API Integration** | Agree on request and response contracts with the frontend developer |
| **Testing** | Write unit, integration, security, and failure-case tests |
| **Documentation** | Maintain setup instructions, environment variables, API docs, and progress notes |

> **Boundary:** The frontend dashboard is outside primary backend ownership. The backend provides stable APIs and coordinates with the frontend developer on payload formats, authentication, and error handling.

---

## 3. Technology Stack

| Technology | Usage |
|---|---|
| Node.js | Backend runtime |
| Express.js | REST API server, routing, and middleware |
| MongoDB | Tenant data, billing events, ledger entries, and usage records |
| Mongoose | Schemas, indexes, validation, and tenant-specific connections |
| Redis | Distributed locks and supporting idempotency coordination |
| JWT | Request authentication and tenant identification |
| RSA-256 | Asymmetric signature validation for protected requests |
| Jest | Unit and integration testing |
| Supertest | Express endpoint testing |
| dotenv | Local environment configuration |
| Git and GitHub | Branch-based collaboration and version control |

**Conventions:** use `async`/`await` for all database, Redis, and network operations; separate modules by responsibility; use JSON for all REST communication.

---

## 4. Backend Architecture

```text
Client / Frontend
       |
       v
Express API Gateway
       |-- Request validation
       |-- RSA-256 signature verification (where required)
       |-- JWT authentication and authorization
       |-- Tenant identification
       v
Tenant Database Manager
       |-- Tenant A MongoDB connection
       |-- Tenant B MongoDB connection
       |-- Tenant C MongoDB connection
       v
Billing / Ledger Services
       |-- Idempotency check
       |-- Redis distributed lock
       |-- MongoDB transaction
       |-- Ledger and usage updates
       v
Analytics / Aggregation Services
       |
       v
JSON API Response
```

> The order of signature verification and authentication must be finalized in the API security contract. Never trust a tenant ID that arrives only in an unverified request body or query parameter.

---

## 5. Folder Structure

A proposed layout. Adapt it to the team's existing repository conventions before creating folders.

```text
backend/
├── src/
│   ├── config/
│   │   ├── env.js
│   │   ├── redis.js
│   │   └── database.js
│   ├── controllers/
│   │   ├── auth.controller.js
│   │   ├── ledger.controller.js
│   │   └── analytics.controller.js
│   ├── middleware/
│   │   ├── authenticate.js
│   │   ├── authorize.js
│   │   ├── identifyTenant.js
│   │   ├── verifySignature.js
│   │   ├── validateRequest.js
│   │   └── errorHandler.js
│   ├── models/
│   │   ├── ledger.model.js
│   │   ├── billingEvent.model.js
│   │   └── usageRecord.model.js
│   ├── routes/
│   │   ├── auth.routes.js
│   │   ├── ledger.routes.js
│   │   └── analytics.routes.js
│   ├── services/
│   │   ├── tenantConnection.service.js
│   │   ├── ledger.service.js
│   │   ├── idempotency.service.js
│   │   ├── lock.service.js
│   │   ├── transaction.service.js
│   │   └── analytics.service.js
│   ├── utils/
│   │   ├── ApiError.js
│   │   └── asyncHandler.js
│   ├── app.js
│   └── server.js
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── scripts/
├── docs/
├── .env.example
├── .gitignore
├── package.json
└── README.md
```

In a monorepo, keep the backend inside the agreed directory and do not move or rename shared files without team agreement.

---

## 6. Implementation Roadmap

The tasks below translate the four-week project milestones into practical development steps.

### Week 1: API Gateway and Dynamic Tenant Provisioning

**Milestone:** Build the API gateway for JWT decoding and implement runtime Mongoose connection switching for isolated tenant databases.

**Tasks**

- [ ] Inspect the repository, package manager, coding conventions, and frontend needs
- [ ] Set up the Node.js and Express server
- [ ] Add environment configuration with `.env` and `.env.example`
- [ ] Configure the base MongoDB and Mongoose setup
- [ ] Define how tenant identity is represented and validated
- [ ] Implement JWT verification middleware
- [ ] Implement tenant identification middleware using verified claims
- [ ] Build a tenant connection manager that selects the correct connection
- [ ] Add connection caching and safe lifecycle management
- [ ] Create initial tenant-aware schemas and models
- [ ] Add a health-check endpoint and basic error handling
- [ ] Test that Tenant A cannot access Tenant B's data

**Approach**

1. Start with a small Express app and a central configuration module.
2. Agree the authentication contract with the team: issuer, verification method, required claims, expiry, and tenant claim.
3. Verify the token before using any tenant information from it.
4. Map the authenticated tenant to an approved, server-side database configuration.
5. Ensure every tenant-scoped query uses models from the selected tenant connection.
6. Test with at least two tenants and deliberately attempt cross-tenant access.

**Output:** a running Express API with JWT-protected routes and verified tenant-specific database selection.

**Avoid:** selecting a database from a client-supplied database name or tenant ID without validating it against authenticated identity and server-side configuration.

---

### Week 2: Idempotent Ledger and Transaction Safety

**Milestone:** Structure Redis distributed locks and unique event IDs, and wrap ledger logic in MongoDB multi-document transactions.

**Tasks**

- [ ] Configure Redis connectivity
- [ ] Design billing event and ledger data models
- [ ] Agree the idempotency key / event ID contract with the team
- [ ] Add a unique, tenant-scoped database index for the event identifier
- [ ] Implement idempotency checks for repeated requests
- [ ] Implement Redis lock acquisition, expiry, and safe release
- [ ] Define the critical section for each billing operation
- [ ] Use MongoDB sessions and transactions for related writes
- [ ] Handle transaction failure and rollback paths
- [ ] Return consistent success, duplicate, conflict, and failure responses
- [ ] Test simultaneous requests that share an idempotency key

**Approach**

1. Require a stable idempotency key or event ID for operations that must not be applied twice.
2. Scope uniqueness to the tenant and operation as the data model requires.
3. Keep idempotency checks safe under concurrency. The unique database constraint is the final safeguard.
4. Use a Redis lock with a bounded expiry and a unique owner token, and release only a lock the current operation owns.
5. Perform all related writes in a single transaction where the deployment supports it.
6. On failure, abort the transaction and release the lock safely.
7. Test retries, concurrent duplicates, timeouts, and failures between writes.

**Output:** a ledger flow that prevents duplicate processing and keeps related writes consistent.

> **Important:** Redis locks alone do not guarantee exactly-once billing. Combine them with database uniqueness and transaction-safe idempotency. MongoDB multi-document transactions require a replica set or supported sharded cluster, so verify the development environment early.

---

### Mid-Project Review: Isolation Audit

The project requires an isolation audit and mock trace tests showing fast database switching under load.

- [ ] Present the tenant routing and connection architecture
- [ ] Demonstrate that two tenants remain isolated
- [ ] Show connection selection and switching traces under load
- [ ] Demonstrate duplicate-request protection
- [ ] Demonstrate a failed transaction and its rollback
- [ ] Record test setup, results, and open issues
- [ ] Review findings with the mentor and team, then update the plan

---

### Week 3: Analytics Pipeline

**Milestone:** Design MongoDB aggregation queries that return usage and cost metrics over historical time scopes.

**Tasks**

- [ ] Agree usage, cost, and time-range definitions with the team
- [ ] Confirm which billing and usage records are the source of truth
- [ ] Design tenant-scoped aggregation pipelines
- [ ] Add indexes for tenant and time-based queries
- [ ] Implement analytics service functions
- [ ] Create API endpoints for dashboard metrics
- [ ] Support the agreed historical ranges and filters
- [ ] Validate aggregation output against known sample records
- [ ] Test performance and confirm tenant filtering throughout

**Approach**

1. Confirm metric definitions and response fields with the frontend developer.
2. Filter by the authenticated tenant before calculating any metric.
3. Use aggregation stages such as `$match`, `$group`, `$project`, and date grouping as appropriate.
4. Add indexes based on real query patterns, for example tenant identifier plus event timestamp.
5. Compare API results with manually calculated sample data.
6. Measure performance and tune indexes and pipelines using explain plans and realistic data.

**Output:** tenant-safe analytics APIs that return usage and cost metrics for the dashboard.

---

### Week 4: RSA-256 Security, Trace Logging, Testing, and Deployment

**Milestone:** Lock endpoints with RSA-256 validation, conduct end-to-end trace logging, and prepare a secure deployment.

**Tasks**

- [ ] Agree the RSA signature format and canonical payload format
- [ ] Configure public-key verification and secure key management
- [ ] Implement signature validation middleware for the protected endpoints
- [ ] Reject invalid, malformed, or tampered signatures
- [ ] Define timestamp and nonce handling if replay protection is part of the protocol
- [ ] Review JWT authorization and tenant-isolation rules
- [ ] Avoid logging secrets, private keys, tokens, or sensitive billing data
- [ ] Add request and trace identifiers for debugging
- [ ] Complete unit and integration tests with Jest and Supertest
- [ ] Test rollback, duplicate requests, and cross-tenant access
- [ ] Prepare environment variables and deployment configuration
- [ ] Document setup, APIs, test commands, and known limitations
- [ ] Prepare the final backend demonstration

**Approach**

1. Obtain public-key and signing requirements from the project contract. Do not invent a signature format.
2. Verify signatures with Node.js cryptographic APIs or an approved library.
3. Ensure client and server agree on the exact bytes or canonical form being signed.
4. Keep private keys and secrets out of source control.
5. Add negative tests for invalid signatures, expired or replayed requests where applicable, and tampered payloads.
6. Run the full test suite and document the results.

**Output:** a tested, documented, securely configured backend ready for final review and integration.

---

## 7. Proposed API Contract

The project brief does not prescribe endpoint names or schemas. The following is a **starting proposal** to agree with the team before implementation.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/health` | Check server health |
| GET | `/api/auth/me` | Return the authenticated user and tenant context |
| POST | `/api/ledger/entries` | Submit a billing or ledger operation with an idempotency key |
| GET | `/api/ledger/entries` | List the authenticated tenant's ledger entries |
| GET | `/api/ledger/entries/:id` | Retrieve one tenant-scoped ledger entry |
| GET | `/api/analytics/usage` | Return usage metrics for an agreed time range |
| GET | `/api/analytics/costs` | Return cost metrics for an agreed time range |

### Design rules

- Authenticate every protected route.
- Take tenant identity from verified authentication context, never from an untrusted client field.
- Validate request bodies, parameters, and query strings.
- Use consistent HTTP status codes and JSON response shapes.
- Never expose connection details, secrets, stack traces, or another tenant's records.
- Define pagination, sorting, date ranges, and filtering with the frontend developer.
- Document required headers: authorization, signature, and idempotency where applicable.

### Example response (illustrative)

```json
{
  "success": true,
  "message": "Ledger entry processed",
  "data": {
    "entryId": "example-entry-id",
    "status": "processed"
  }
}
```

Final field names and statuses must be agreed with the frontend and project team.

---

## 8. Data Model Planning

The project requires tenant isolation and a reliable billing ledger but does not define final schemas. These are planning considerations only.

| Model | Potential fields |
|---|---|
| **Billing event** | Tenant-scoped event ID, idempotency key, event type, amount and currency, event timestamp, processing status, audit and reconciliation metadata |
| **Ledger entry** | Entry ID, tenant context, related billing event ID, debit/credit direction per billing rules, amount and currency, created timestamp, transaction status |
| **Usage record** | Tenant context, resource or usage category, quantity and unit, recorded timestamp, pricing reference or calculated cost per agreed business rules |

### Database rules

- Add indexes for real query patterns.
- Enforce uniqueness of idempotency and event identifiers at the database level.
- Apply tenant scoping consistently to reads, writes, updates, and deletes.
- Never trust tenant identifiers from request payloads.
- Decide retention, audit, and reconciliation requirements with the mentor and team.

---

## 9. Security Checklist

- [ ] Verify JWT signatures and expiry
- [ ] Validate authorization and tenant claims
- [ ] Apply tenant isolation to every tenant-scoped operation
- [ ] Never accept arbitrary database names from clients
- [ ] Validate RSA-256 signatures on the agreed protected routes
- [ ] Store secrets and keys in environment variables or a secret manager
- [ ] Commit `.env.example`, never the real `.env`
- [ ] Validate and sanitize all incoming data
- [ ] Use safe Redis lock ownership, expiry, and release
- [ ] Enforce database-level uniqueness for idempotency
- [ ] Use MongoDB transactions for related ledger writes
- [ ] Avoid logging tokens, keys, passwords, or sensitive billing details
- [ ] Return safe errors without internal implementation details
- [ ] Test cross-tenant access attempts and invalid authentication

---

## 10. Testing and Performance Validation

The project calls for Jest and Supertest and sets performance expectations, including sub-5 ms indexed database queries. Treat these as benchmarks to measure and report under a documented setup, not as assumed results.

### Unit tests

- JWT and tenant middleware behavior
- Tenant connection selection
- Idempotency service behavior
- Redis lock ownership and release
- Signature verification
- Analytics aggregation transformations

### Integration tests

- API authentication and authorization
- Tenant A and Tenant B data isolation
- Ledger creation and retrieval
- Duplicate event submission
- Concurrent requests with the same idempotency key
- MongoDB transaction commit and rollback
- Redis unavailable or timeout behavior
- Invalid or tampered RSA-signed requests
- Analytics results against known sample data

### Key scenarios

| Scenario | Expected result |
|---|---|
| Valid token for Tenant A | Request uses Tenant A context |
| Tenant A requests a Tenant B record | Access denied or record not found, per API policy |
| Same billing event submitted twice | Applied only once |
| Two simultaneous requests share a key | Only one logical operation is committed |
| A transaction fails midway | All related writes are rolled back |
| Invalid signature | Protected request is rejected |
| Analytics request lacks tenant context | Request is rejected |

### Performance evidence to collect

- Query latency with explain-plan and index evidence
- API throughput and latency under concurrent load
- Redis lock contention and timeout behavior
- Tenant connection switching under load
- Test data volume, environment, and benchmark commands

> Do not report a benchmark as passed until it has been run and the result recorded.

---

## 11. Git Workflow

Backend work happens on the `backend` branch. Avoid committing directly to the shared main branch.

### Initial checks

```bash
git status
git branch
git switch backend
git pull origin backend
```

If the remote branch does not exist yet, coordinate with the team before pushing it.

### Commit workflow

```bash
git add .
git commit -m "feat(backend): add Express server setup"
git push origin backend
```

Use small, meaningful commits with these prefixes:

| Prefix | Use for |
|---|---|
| `feat:` | New backend functionality |
| `fix:` | Bug fixes |
| `test:` | Tests |
| `refactor:` | Restructuring without behavior change |
| `docs:` | Documentation |
| `chore:` | Tooling and configuration |

### Before opening a pull request

- [ ] Pull or rebase the latest agreed base branch
- [ ] Run linting and tests
- [ ] Confirm `.env` and private keys are not staged
- [ ] Review the diff for unrelated changes
- [ ] Describe the feature, test evidence, and known limitations
- [ ] Request review from the assigned teammate or mentor

Agree the target branch and merge strategy with the repository owner. Never force-push shared branches or overwrite teammates' work.

---

## 12. Local Setup

Exact commands depend on the repository's package manager and scripts. Replace this section with verified commands once the backend package exists.

```bash
cd backend
npm install
cp .env.example .env
npm run dev
```

### Environment variables

Commit `.env.example` with names and safe placeholders only. Confirm exact names and formats during implementation.

```dotenv
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

# Optional: only if the tenant architecture uses a registry or configuration store
TENANT_REGISTRY_URI=
```

Never commit real credentials, tokens, private keys, production database URIs, or populated secret files. If the JWT design uses a shared secret instead of an asymmetric key, document the actual configuration and remove unused variables.

---

## 13. Frontend Integration

Coordinate with the frontend developer before finalizing APIs. Agree on:

- Authentication method and token format
- How tenant context is established
- Required request headers
- RSA signature requirements, if the client signs requests
- Ledger request and response fields
- Idempotency key generation and retry behavior
- Analytics filters, date ranges, and response formats
- Loading, validation, authorization, conflict, and server error responses
- CORS and local development URLs

Share API examples and endpoint changes early, and never change response fields without notifying the frontend developer.

---

## 14. Deliverables and Review Preparation

### Backend deliverables

- [ ] Working backend source code
- [ ] Express API and middleware
- [ ] Tenant-aware database connection management
- [ ] Billing, ledger, and idempotency implementation
- [ ] Redis locking and MongoDB transaction handling
- [ ] Analytics aggregation APIs
- [ ] RSA-256 request validation as agreed
- [ ] Jest and Supertest test suite
- [ ] `.env.example` and setup documentation
- [ ] API documentation and frontend integration notes
- [ ] Test and performance evidence
- [ ] Backend architecture explanation for the reviews
- [ ] Final demonstration of tenant isolation and safe billing behavior

### Mid-project review: be ready to explain

1. How the API identifies a tenant
2. How requests select the correct database connection
3. How isolation is enforced
4. How duplicate requests are detected
5. How Redis locks and MongoDB transactions work together
6. What happens when a transaction fails

### Final review: be ready to demonstrate

1. Secure JWT and RSA-256 validation
2. Tenant isolation
3. Idempotent ledger operations
4. Transaction rollback
5. Usage and cost analytics
6. Test coverage and measured performance
7. Setup and API documentation

---

## 15. Working Principles

- Read the existing repository before changing its structure.
- Confirm requirements and API contracts with the team and mentor.
- Implement one module at a time and test it before integration.
- Keep controllers thin and business logic in services.
- Centralize database access and tenant connection selection.
- Use clear validation and centralized error handling.
- Write tests for both success and failure paths.
- Update documentation whenever behavior or API contracts change.
- Send periodic progress updates to the assigned mentor.
- Keep confidential project information and credentials secure.

---

## 16. Progress Tracker

Update as work is completed.

| Milestone | Status | Notes / Evidence |
|---|---|---|
| Repository and backend setup | ⬜ Not started | |
| Express server and configuration | ⬜ Not started | |
| JWT authentication | ⬜ Not started | |
| Tenant identification and isolation | ⬜ Not started | |
| Dynamic Mongoose connection manager | ⬜ Not started | |
| Billing and ledger schemas | ⬜ Not started | |
| Idempotency and unique constraints | ⬜ Not started | |
| Redis distributed locks | ⬜ Not started | |
| MongoDB transactions and rollback | ⬜ Not started | |
| Mid-project isolation review | ⬜ Not started | |
| Analytics aggregation APIs | ⬜ Not started | |
| RSA-256 signature validation | ⬜ Not started | |
| Unit and integration tests | ⬜ Not started | |
| Performance testing and evidence | ⬜ Not started | |
| Frontend integration | ⬜ Not started | |
| Documentation and final demo | ⬜ Not started | |

---

## 17. Scope Notes and Open Decisions

The project specification defines the main backend modules and four-week milestones. It does **not** specify the items below, so each is an open decision to settle with the mentor and team.

| Open decision | Notes |
|---|---|
| API routes and schemas | Section 7 is a proposal only |
| JWT claims | Issuer, audience, tenant claim, expiry |
| Database schemas and pricing rules | Section 8 lists planning considerations only |
| RSA-256 details | Exact algorithm, padding, key format, and payload canonicalization |
| Deployment provider | Not specified |
| Tenant provisioning | Must fit the actual database hosting arrangement and tenant registry |
| MongoDB transaction support | Depends on deployment topology; confirm before building the ledger flow |

**Other notes**

- **"Zero-knowledge" terminology:** it appears in the project title, but the brief defines no zero-knowledge proof protocol or privacy design. Do not claim such a protocol is implemented unless it is separately specified and built.
- **Performance goals:** must be measured and documented with reproducible tests.

---


