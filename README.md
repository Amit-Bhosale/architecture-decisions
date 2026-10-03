# Architecture Decisions

These are decisions I've made while building production systems, written down properly.

Most engineering choices disappear into Slack threads and meeting notes. Six months later nobody remembers why we picked one database over another, or why we didn't just rewrite the old system. I want to keep that reasoning, partly for myself and partly because the "why" is usually more interesting than the "what".

Each record covers one decision:

- what we were building, and what constraints we were under
- the options we actually considered
- what we picked, and the one thing that decided it
- what it cost us, because every decision costs something
- what would make me choose differently today

Client names, code and internal numbers are left out on purpose. The reasoning is mine to share. The systems aren't.

## Decisions

| # | Decision | Area |
|---|----------|------|
| 0001 | [Isolated database per tenant over a shared schema](0001-isolated-database-per-tenant.md) | Multi-tenant SaaS |

## Coming up


If a decision later turns out wrong, I won't delete it. I'll write a new one that replaces it, and link the two. Changing your mind for good reasons is part of the job.
