# Enterprise Systems Pattern Guide

Concise references for reviewing enterprise application architecture. Each note is standalone: start with the architectural concern, then follow the relevant pattern catalog. Pattern names describe recurring trade-offs, not mandatory components.

## Architectural concerns

- [Layering](layering.md) — dependency direction and logical boundaries
- [Organizing domain logic](organizing-domain-logic.md) — choosing a model for business rules
- [Relational mapping](mapping-to-relational-databases.md) — identity, relationships, loading, and inheritance
- [Web presentation](web-presentation.md) — requests, controllers, views, and security
- [Concurrency](concurrency.md) — invariants, conflicts, and locking
- [Session state](session-state.md) — state placement, lifetime, and trust
- [Distribution](distribution-strategies.md) — remote contracts and failure boundaries

## Pattern catalogs

- [Domain logic](domain-logic-patterns.md)
- [Data source](data-source-architectural-patterns.md)
- [Object–relational behavior](object-relational-behavioral-patterns.md)
- [Object–relational structure](object-relational-structural-patterns.md)
- [Metadata and queries](object-relational-metadata-mapping-patterns.md)
- [Web presentation](web-presentation-patterns.md)
- [Distribution](distribution-patterns.md)
- [Offline concurrency](offline-concurrency-patterns.md)
- [Session state](session-state-patterns.md)
- [Supporting patterns](base-patterns.md)

## Review sequence

1. Identify business invariants and transaction boundaries.
2. Check dependency direction between presentation, domain, and data source.
3. Trace one read and one write through mapping, loading, and failure paths.
4. Examine concurrent updates, retries, and partial failure.
5. Verify trust boundaries: validation, authorization, output encoding, session integrity, and remote authentication.
6. Remove layers or patterns that do not address an observed force.
