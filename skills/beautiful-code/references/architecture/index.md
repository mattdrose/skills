# Architecture Guide

Choose architecture from observed forces, not from a desired pattern count.

## Start by concern

- [Layering and domain logic](layering-and-domain-logic.md)
- [Data access](data-access.md)
- [Relational mapping](relational-mapping.md)
- [Relational behavior](relational-behavior.md)
- [Relational structure](relational-structure.md)
- [Metadata and queries](metadata-and-queries.md)
- [Web presentation](web-presentation.md)
- [Session state](session-state.md)
- [Distribution](distribution.md)
- [Concurrency control](concurrency-control.md)
- [Supporting patterns](supporting-patterns.md)

## Review sequence

1. Identify business invariants and transaction boundaries.
2. Check dependency direction among presentation, domain, and data access.
3. Trace one read and one write through mapping, loading, and failure paths.
4. Examine concurrent updates, retries, and partial failure.
5. Verify validation, authorization, output encoding, session integrity, and remote authentication.
6. Remove layers or patterns that do not answer an observed force.
