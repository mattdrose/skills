# Discovery and Requirements

## Find the need behind the request

Treat a requested feature as one proposed solution, not the requirement itself. Ask who needs what outcome, why it matters, what happens today, and how success will be observed. Concrete examples often expose a cheaper or more direct solution than the original request.

Requirements emerge through conversation and evidence. Record the need, representative examples, decisions, and unresolved questions so later implementation does not silently replace them with assumptions.

## Separate constraints from assumptions

List constraints explicitly and identify their source. Verify legal, safety, physical, policy, budget, compatibility, and product limits with the people or evidence that establish them. Do not evade genuine constraints through clever framing.

Label everything else as an assumption and test it. When work appears impossible, identify which dimension can move and solve the smallest core problem first. A familiar process, architecture, or interpretation is not a constraint merely because it already exists.

## Prototypes and tracer evidence

Use a prototype to answer a named uncertainty cheaply: a risky interaction, algorithm, integration, or workflow. Declare what the experiment must demonstrate, which realities it intentionally ignores, and that exploratory shortcuts are disposable. Do not let prototype code become production by accident.

Use a thin production-shaped path when the uncertainty is whether the real system can work end to end. It should touch the relevant boundaries and produce evidence about feasibility while leaving room to grow. Choose between prototype and tracer evidence according to the question, not according to which artifact looks more complete.

## Domain language

Use the vocabulary and representations of the people whose problem is being solved. Shared terms reduce translation errors in requirements, decisions, interfaces, and user-facing behavior. Capture examples when a term has ambiguous boundaries.

Prefer an existing notation, data format, or small purpose-shaped interface. A custom language creates documentation, tooling, migration, and support obligations; accept those costs only when the domain benefit is clear.

## Estimation as uncertainty management

Start with the decision the estimate must support and choose precision appropriate to that horizon. Expose assumptions, dependencies, unknowns, and a range rather than presenting an unsupported date as certainty.

Derive estimates from a simple model, known quantities, comparable work, and measured progress. Refine them as evidence arrives, and communicate what changed. Estimates are instruments for planning and risk response, not promises detached from conditions.

## Collaboration and agility

Bring the people with domain, delivery, and operational context together where doing so removes handoffs or hidden assumptions. Pairing, ensemble work, design sessions, and direct user conversations are useful only when participation is active and psychologically safe.

Agility is the ability to observe and respond, not compliance with a branded process. Use short learning cycles, revise plans when evidence changes, and retain only practices that improve outcomes in the current context.
