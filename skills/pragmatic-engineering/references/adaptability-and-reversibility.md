# Adaptability and Reversibility

## Orthogonality

Arrange responsibilities so one concern can change without unrelated parts moving with it. Focused components, narrow interfaces, and explicit dependencies reduce coordination cost, regression risk, and the scope of testing needed after a change.

Assess independence by likely change scenarios, not by diagram neatness. Global state, hidden dependencies, and exposed internals make operational impact harder to predict.

## Decoupling

A component should know only the capabilities it needs from collaborators. Avoid dependency chains that reveal internal organization or require callers to coordinate another component's work.

Couple to stable behavior rather than current structure. This contains vendor, storage, workflow, and organizational changes, making replacement and incident isolation less disruptive.

## Events and transformation pipelines

Use events when a state change has independent reactions and the producer should not orchestrate each one. Choose callbacks, streams, queues, or logs according to delivery, ordering, replay, latency, and observability needs. Account explicitly for duplicate handling, retries, and failed consumers as operational costs.

Use a transformation pipeline when work is naturally understood as data moving through stages. Keep intermediate representations and state transitions visible so stages can be inspected, replaced, and recovered. Do not hide essential side effects behind a deceptively simple pipeline.

## Composition over inheritance

Assemble capabilities through interfaces, delegation, and components when behavior must vary independently. Composition limits the commitments inherited by each participant and makes replacement boundaries clearer.

Use inheritance only for a stable substitutability relationship, not merely to reuse implementation. A rigid hierarchy turns local changes into broad compatibility and migration risks.

## Configuration

Externalize policy that legitimately varies by environment or deployment so operators can change it without rebuilding the product. Make effective configuration inspectable, validated before use, versioned where appropriate, and handled securely when sensitive.

Do not convert every constant into a setting. Each option adds states to test, document, migrate, and support; configuration should represent real operational variation.

## Reversible decisions

Identify choices whose reversal would require broad rewrites, coordinated migration, downtime, or vendor negotiation. Under material uncertainty, defer those commitments or isolate them behind a boundary while evidence develops.

Reversibility has a cost, so preserve options selectively. State the uncertainty, the cost of changing course, the boundary or rollout that limits commitment, and the signal that will trigger a decision. Practice reversal for operationally critical paths rather than assuming it will work.
