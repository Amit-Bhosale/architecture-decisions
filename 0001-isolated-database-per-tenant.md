# ADR-001: Isolated database per tenant over shared schema with tenant IDs

**Status:** Accepted
**Date:** 2024-03
**Owner:** Amit Bhosale

---

## Context

We were building a multi-tenant SaaS platform on Django serving institutional
clients. Each client held data they considered commercially sensitive, and
several were contractually required to demonstrate that their records could not
be reached by any other customer of the platform.

The team was small. There was no dedicated infrastructure engineer, so whatever
we chose had to be operable by the application developers who built it.

Two options were on the table:

**Option A: Shared schema with a tenant_id column.** All tenants live in one
database. Every query filters on tenant_id, usually through a middleware-injected
default manager.

**Option B: Database per tenant.** A shared authentication service resolves
identity and routes each request to that tenant's own database.

The deciding constraint was not performance. It was the failure mode. Under
Option A, a single missing filter in a single query leaks one customer's data to
another. That failure is silent, easy to introduce during ordinary feature work,
and difficult to detect in review. Under Option B, the same mistake returns
nothing at all, because the data is not in the connection.

## Decision

We chose Option B, a separate database per tenant, with a shared authentication
service handling identity and a custom routing layer resolving the correct
connection per request.

The routing layer and tenant-aware middleware were built in-house rather than
adopting an existing multi-tenancy package, because we needed the resolution
logic to sit before the ORM rather than around it, and the available packages
assumed a shared-schema model.

## Consequences

**What this bought us**

Cross-tenant leakage moved from a code-discipline problem to a structural
impossibility. A developer cannot accidentally query another tenant's data
because the connection does not reach it.

Per-tenant operations became straightforward: backup, restore, export on
offboarding, and migration of a single tenant without touching others.

Onboarding a client with a data isolation clause stopped requiring a
conversation about how our query layer works.

**What it cost us**

Migrations became fan-out operations. A schema change runs N times rather than
once, and partial failure across tenants is a real state that has to be handled.

Cross-tenant analytics became genuinely hard. Any aggregate query spanning all
customers now requires a separate path, which is one of the reasons we later
built a CDC pipeline into a warehouse rather than querying production.

Connection pool pressure grows with tenant count. This was acceptable at our
scale and would need revisiting well before we reached hundreds of tenants.

**What would change this decision**

If tenant count grew by an order of magnitude, or if the product moved toward
self-serve signup with many small tenants rather than fewer institutional ones,
the operational cost would likely outweigh the isolation benefit. At that point
a hybrid model, isolation for enterprise tiers and shared schema for self-serve,
would be worth evaluating.