# LedgerGuard

## Dynamic Multi-Tenant Zero-Knowledge Billing Engine

LedgerGuard is an enterprise SaaS billing-engine project designed to
address two critical challenges: **strict data isolation between
organizations** and **reliable financial transaction processing**.

The system is designed to route authenticated organizations to isolated
MongoDB database connections, protect billing operations against
duplicate requests, and provide usage and cost analytics through a
client dashboard.

> **Project focus:** Multi-tenant architecture · Secure API gateway ·
> Idempotent billing · Redis distributed locks · MongoDB ACID
> transactions · RSA-256 request validation

------------------------------------------------------------------------

## Table of Contents

-   [The Problem](#the-problem)
-   [The Proposed Solution](#the-proposed-solution)
-   [Example Scenario](#example-scenario)
-   [Core Features](#core-features)
-   [System Architecture](#system-architecture)
-   [Technology Stack](#technology-stack)
-   [Billing Transaction Workflow](#billing-transaction-workflow)
-   [Security and Data Isolation](#security-and-data-isolation)
-   [Analytics Dashboard](#analytics-dashboard)
-   [Development Roadmap](#development-roadmap)
-   [Quality and Evaluation Goals](#quality-and-evaluation-goals)
-   [Getting Started](#getting-started)
-   [Repository Structure](#repository-structure)
-   [Project Status](#project-status)
-   [Important Scope Notes](#important-scope-notes)

------------------------------------------------------------------------

## The Problem

Enterprise SaaS platforms often serve many organizations through a
shared application. This creates important engineering challenges.

### 1. Cross-Tenant Data Exposure

If tenant data is not properly isolated, one organization may
accidentally or maliciously access another organization's records. A
data leak can expose sensitive business and billing information.

### 2. Duplicate Billing Operations

Network failures, client retries, or repeated API requests can cause the
same billing event to be processed more than once. This can result in
duplicate charges or duplicate ledger entries.

### 3. Inconsistent Financial Records

A billing operation may involve multiple database writes. If the process
fails after some writes have succeeded, the ledger can be left in an
inconsistent state unless the operation is handled atomically.

### 4. Limited Usage Visibility

SaaS organizations need to understand their resource consumption and
expenditure over time. The platform therefore needs tenant-scoped
analytics that can report usage and cost metrics across historical time
ranges.

------------------------------------------------------------------------

## The Proposed Solution

LedgerGuard proposes a secure, multi-tenant billing architecture built
around authenticated tenant routing and reliable transaction processing.

The API gateway verifies incoming requests and identifies the
organization associated with the authenticated identity. A tenant-aware
database manager selects the appropriate isolated MongoDB connection.

For billing operations, the system combines:

-   **Unique event IDs and database constraints** to identify repeated
    billing requests.
-   **Redis distributed locks** to coordinate concurrent operations.
-   **MongoDB multi-document ACID transactions** to commit related
    writes together or roll them back on failure.
-   **RSA-256 asymmetric cryptography** to validate request/payload
    authenticity on protected API routes, according to the agreed
    signature protocol.
-   **MongoDB aggregation pipelines** to calculate tenant-specific usage
    and cost metrics.

The intended result is a billing platform architecture that protects
tenant boundaries and maintains consistent ledger state during retries
and failures.

------------------------------------------------------------------------

## Example Scenario

Imagine Company A and Company B using LedgerGuard at the same time.

1.  Both companies send requests to the LedgerGuard API.
2.  The API gateway verifies each request and identifies the
    authenticated organization.
3.  The tenant database manager selects the corresponding isolated
    MongoDB connection.
4.  A billing request is checked using its event ID or idempotency key.
5.  Redis locking coordinates concurrent processing where required.
6.  MongoDB performs the related ledger writes inside a transaction.
7.  If the operation succeeds, the transaction is committed.
8.  If a failure occurs, the transaction is aborted so partial writes
    are not retained.
9.  The organization's dashboard can retrieve its own usage and cost
    analytics.

A repeated request for the same logical billing event should not create
a second billing operation.

------------------------------------------------------------------------

## Core Features

### 1. Secure API Gateway

**Technologies:** Node.js, Express.js, JWT

-   Intercepts incoming API requests.
-   Verifies authentication tokens and required claims.
-   Identifies the organization associated with the authenticated
    request.
-   Routes authorized requests to the correct tenant context.
-   Protects application endpoints with authentication and authorization
    middleware.

### 2. Dynamic Multi-Tenant Database Pool

**Technologies:** MongoDB, Mongoose

-   Manages tenant-specific MongoDB connections at runtime.
-   Selects the appropriate connection for an authenticated tenant.
-   Supports connection provisioning, reuse, and lifecycle management.
-   Keeps tenant-scoped database operations within the correct tenant
    boundary.

### 3. Idempotent Billing Ledger

**Technologies:** Redis, MongoDB

-   Uses unique event IDs or idempotency keys to identify repeated
    requests.
-   Enforces appropriate unique database constraints.
-   Uses Redis distributed locks to coordinate concurrent billing
    operations.
-   Wraps related ledger writes in MongoDB multi-document transactions.
-   Handles failures by aborting incomplete transactions.

### 4. Usage and Cost Analytics

**Technologies:** MongoDB Aggregation, React, Chart.js

-   Calculates usage metrics for an organization.
-   Returns cost metrics across historical time ranges.
-   Supports a client dashboard for viewing resource expenditure and
    limits.
-   Keeps analytics scoped to the authenticated tenant.

### 5. Cryptographic Request Validation

**Technology:** RSA-256 asymmetric cryptography

-   Uses public/private key cryptography as specified by the project.
-   Validates payload authenticity on protected API routes.
-   Rejects requests that fail the agreed signature-verification
    process.
-   Keeps cryptographic keys and secrets out of source control.

------------------------------------------------------------------------

## System Architecture

``` text
                  Organization A
                        |
                  Organization B
                        |
                        v
                +------------------+
                |   API Gateway    |
                | Express + JWT    |
                +------------------+
                        |
                 Request validation
                        |
                 Tenant identification
                        |
                        v
              +----------------------+
              | Tenant DB Manager    |
              | Mongoose Connections |
              +----------------------+
                    /         \
                   /           \
                  v             v
          +-------------+  +-------------+
          | Tenant A DB |  | Tenant B DB |
          +-------------+  +-------------+
                  |
                  v
          +----------------------+
          | Billing / Ledger     |
          +----------------------+
                  |
          +-------+--------+
          |                |
          v                v
     +---------+    +-------------+
     |  Redis  |    |  MongoDB    |
     |  Locks  |    | Transactions|
     +---------+    +-------------+
                           |
                           v
                 +------------------+
                 | Analytics APIs   |
                 +------------------+
                           |
                           v
                 +------------------+
                 | React Dashboard  |
                 | Usage & Cost     |
                 +------------------+
```

This diagram illustrates the intended logical flow. The final connection
topology, tenant registry, and deployment arrangement are implementation
decisions to be confirmed during development.

------------------------------------------------------------------------

## Technology Stack

  -----------------------------------------------------------------------
  Layer                   Technologies            Purpose
  ----------------------- ----------------------- -----------------------
  Frontend                React, TypeScript       Enterprise client
                                                  dashboard

  Charts                  Chart.js                Usage and expenditure
                                                  visualizations

  Backend                 Node.js, Express.js     REST API and
                                                  application services

  Database                MongoDB, Mongoose       Tenant data, billing
                                                  events, and ledger
                                                  records

  Distributed             Redis                   Distributed locks
  coordination                                    

  Authentication          JWT                     Request authentication
                                                  and tenant identity

  Cryptography            RSA-256                 Asymmetric
                                                  request/payload
                                                  validation

  Testing                 Jest, Supertest         Unit and API
                                                  integration testing
  -----------------------------------------------------------------------

### Common Engineering Practices

-   Use asynchronous programming and `async`/`await` for non-blocking
    operations.
-   Use JSON for REST API communication.
-   Keep routes, controllers, services, middleware, and database models
    modular.
-   Manage configuration and secrets through environment variables.
-   Maintain clear API contracts between frontend and backend.

------------------------------------------------------------------------

## Billing Transaction Workflow

The billing workflow is designed to prevent duplicate processing and
partial ledger updates.

``` text
Incoming Billing Request
          |
          v
Authenticate and Validate
          |
          v
Identify Tenant
          |
          v
Select Tenant Database
          |
          v
Check Event ID / Idempotency Key
          |
          v
Coordinate with Redis Lock
          |
          v
Start MongoDB Transaction
          |
          v
Perform Related Ledger Writes
          |
          v
     Transaction Result
       /          \
      /            \
   Success         Failure
     |                |
     v                v
   Commit           Abort
     |                |
     v                v
 Return Result    Return Safe Error
```

The database uniqueness constraint and transaction are important
safeguards alongside Redis locking. A Redis lock by itself does not
guarantee exactly-once processing.

MongoDB multi-document transactions require a transaction-capable
deployment, such as a replica set or supported sharded cluster.

------------------------------------------------------------------------

## Security and Data Isolation

Security is a central part of LedgerGuard's design.

### Tenant Isolation

-   Derive tenant identity from verified authentication context.
-   Do not trust an arbitrary tenant ID or database name supplied by a
    client.
-   Ensure every tenant-scoped read and write uses the correct tenant
    connection.
-   Test that one tenant cannot access another tenant's records.

### Authentication and Authorization

-   Verify JWT signatures and expiry.
-   Validate required claims and permissions.
-   Protect private endpoints with middleware.
-   Return safe errors for invalid or unauthorized requests.

### Billing Integrity

-   Use unique event identifiers and database constraints.
-   Coordinate concurrent operations with Redis locks.
-   Use MongoDB transactions for related writes.
-   Test retry, timeout, duplicate, and rollback scenarios.

### Cryptographic Validation

-   Verify signatures using the public key and agreed RSA signature
    format.
-   Keep private keys and secrets secure.
-   Define payload canonicalization and replay protection requirements
    as part of the API security contract.

------------------------------------------------------------------------

## Analytics Dashboard

The React dashboard is intended to help each organization understand its
own resource usage and expenditure.

The backend analytics pipeline is designed to:

1.  Identify the authenticated tenant.
2.  Filter records to that tenant.
3.  Apply the requested historical time range.
4.  Aggregate usage and cost metrics in MongoDB.
5.  Return structured JSON for dashboard visualizations.

The frontend project specification identifies Chart.js for visualizing
active resource expenditures and limits.

------------------------------------------------------------------------

## Development Roadmap

The supplied project specification defines a four-week implementation
plan.

  -----------------------------------------------------------------------
  Timeline                Backend milestones      Frontend milestones
  ----------------------- ----------------------- -----------------------
  Week 1                  Build Express API       Build enterprise login
                          gateway, JWT decoding,  and multi-tenant
                          and runtime Mongoose    dashboard layout.
                          connection switching    
                          for isolated tenant     
                          clusters.               

  Week 2                  Implement Redis         Build billing ledger
                          distributed locks,      views and transaction
                          unique event IDs, and   success/failure state
                          MongoDB multi-document  management.
                          transactions for the    
                          ledger.                 

  Mid-project review      Audit tenant isolation  Demonstrate correct UI
                          and demonstrate         handling of transaction
                          database switching      rollback states.
                          under load.             

  Week 3                  Build MongoDB           Build Chart.js
                          aggregation queries for visualizations for
                          usage and cost metrics  resource expenditure
                          over historical time    and limits.
                          ranges.                 

  Week 4                  Implement RSA-256       Refine loading states,
                          validation, end-to-end  chart rendering, and
                          trace logging, and      end-to-end workflows.
                          secure deployment.      

  Final review            Demonstrate tenant      Demonstrate accurate
                          isolation, payload      multi-tenant usage
                          security, and           metering in the
                          transaction atomicity.  dashboard.
  -----------------------------------------------------------------------

### Mid-Project Review

The project calls for an isolation audit, code reviews, and mock trace
tests demonstrating database switching under load.

### Final Project Review

The final review is intended to demonstrate:

-   Tenant data isolation.
-   Payload authenticity validation.
-   Reliable ledger transaction handling.
-   Usage metering and analytics in the client dashboard.

------------------------------------------------------------------------

## Quality and Evaluation Goals

The project specification identifies performance and delivery goals.
These are targets to validate through testing, not claims that have
already been achieved.

  -----------------------------------------------------------------------
  Area                                Project goal
  ----------------------------------- -----------------------------------
  Database performance                Evaluate indexed query performance
                                      against the specified sub-5ms
                                      target.

  Tenant isolation                    Demonstrate isolation through code
                                      review and cross-tenant access
                                      tests.

  Transaction integrity               Demonstrate atomic commit and
                                      rollback behavior.

  Request security                    Demonstrate RSA-256 validation on
                                      protected API routes.

  Code quality                        Maintain a clean GitHub repository,
                                      modular code, and comprehensive
                                      README documentation.

  Project delivery                    Complete the mid-project review in
                                      Week 2 and final review in Week 4.
  -----------------------------------------------------------------------

Performance reports should include the test environment, dataset size,
workload, methodology, and measured results.

------------------------------------------------------------------------

## Getting Started

The repository's exact package structure, scripts, and deployment
configuration should be documented as implementation is finalized.

### Prerequisites

-   Node.js and npm.
-   MongoDB configured with transaction support.
-   Redis.
-   Git.
-   A modern web browser for the dashboard.

### Clone the Repository

``` bash
git clone https://github.com/bhavik-gitt/LedgerGuard.git
cd LedgerGuard
```

### Configure Environment Variables

Create a local `.env` file based on the project's `.env.example`.

The following names are illustrative; confirm the actual configuration
names used by the implementation.

``` dotenv
NODE_ENV=development
PORT=5000

MONGODB_URI=
REDIS_URL=

JWT_PUBLIC_KEY=
JWT_ISSUER=
JWT_AUDIENCE=

RSA_PUBLIC_KEY=
```

Never commit real database credentials, tokens, private keys, or
populated `.env` files.

### Install and Run

Install dependencies and use the development scripts defined in the
relevant frontend and backend `package.json` files. Exact commands will
be added once the repository setup is finalized.

------------------------------------------------------------------------

## Suggested Repository Structure

The following is an illustrative structure. Adapt it to the repository's
existing layout.

``` text
LedgerGuard/
├── README.md
├── frontend/
│   ├── src/
│   └── package.json
│
├── backend/
│   ├── src/
│   │   ├── config/
│   │   ├── controllers/
│   │   ├── middleware/
│   │   ├── models/
│   │   ├── routes/
│   │   ├── services/
│   │   └── server.js
│   ├── tests/
│   └── package.json
│
├── docs/
│   ├── architecture/
│   └── api/
│
└── .gitignore
```

------------------------------------------------------------------------

## Project Status

**Development status:** Update this section as features are implemented
and tested.

  Component                            Status
  ------------------------------------ ---------
  API Gateway and JWT authentication   Planned
  Dynamic tenant database management   Planned
  Tenant isolation tests               Planned
  Idempotent billing ledger            Planned
  Redis distributed locks              Planned
  MongoDB transactions and rollback    Planned
  Usage and cost analytics APIs        Planned
  React analytics dashboard            Planned
  RSA-256 request validation           Planned
  End-to-end testing and deployment    Planned

Update each status only when implementation and relevant verification
are complete.

------------------------------------------------------------------------

## Important Scope Notes

1.  **Zero-Knowledge terminology:** "Zero-Knowledge" is part of the
    project title in the supplied specification. The specification does
    not define a particular zero-knowledge proof protocol or describe
    its implementation. This README therefore does not claim that a
    zero-knowledge proof system has been implemented.
2.  **RSA-256 details:** The project specification names RSA-256
    validation. The exact signature scheme, padding, key format, and
    payload canonicalization must be defined in the implementation's
    security contract.
3.  **Tenant architecture:** The specification calls for runtime
    provisioning and switching of isolated MongoDB clusters. The actual
    tenant registry, connection lifecycle, and infrastructure topology
    must be finalized during implementation.
4.  **Exactly-once behavior:** Redis locks, unique constraints, and
    MongoDB transactions serve different purposes. The final
    implementation must combine them carefully and test concurrent
    retries and failure cases.
5.  **Performance:** The stated query-time goal must be measured in the
    target environment. Do not describe it as achieved until benchmark
    evidence exists.

------------------------------------------------------------------------

## Project Reference

Project scope and milestones are based on the supplied **PROJECT.pdf**,
Project 3: "LedgerGuard: Dynamic Multi-Tenant Zero-Knowledge Billing
Engine" (pages 4--6).

------------------------------------------------------------------------

## License

No license has been specified for this project. Add a license after
confirming the repository owner's intended terms.

