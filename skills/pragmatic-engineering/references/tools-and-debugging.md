# Tools and Debugging

## Plain text and automation

Prefer documented, versionable text formats for durable data and configuration when practical. They remain inspectable across tool changes and support search, diff, recovery, and automation. Choose a binary or proprietary format only when its concrete benefit outweighs those operational advantages.

Turn recurring commands into repository scripts or task definitions. The aim is a repeatable team outcome, not dependence on one person's command history.

## Shell and editor fluency

Know the working environment well enough to navigate, search, transform, compose commands, and repeat routine changes without manual friction. Keep essential setup reproducible and portable across the project's relevant languages.

Fluency is outcome-focused: use whichever shell and editor capabilities make investigation and change reliable. Do not prescribe a product when ordinary, shared interfaces are sufficient.

## Version control

Version every meaningful artifact needed to understand, verify, deliver, or operate a change, including documentation, configuration, and operational scripts. Keep changes coherent and reviewable, with history that explains intent and trade-offs.

Practice restoration, comparison, and branching workflows before an incident. Version control is both collaboration infrastructure and a recovery tool.

## Systematic debugging

Use this sequence:

1. **Reproduce** the failure reliably enough to observe it, preserving the original evidence.
2. **Narrow** the failing boundary with logs, controlled experiments, recent changes, and the actual error.
3. **Identify root cause** by challenging assumptions and distinguishing the initiating defect from downstream symptoms.
4. **Demonstrate the explanation** by showing that it accounts for every material observation and predicts a confirming result.
5. **Fix** the cause rather than masking the symptom, adding proportionate protection against recurrence.
6. **Verify** the original scenario, relevant neighboring behavior, and operational result.

Persistent intuition that something is wrong is a prompt to pause and gather evidence, not a substitute for evidence. Write down what is known and what result would falsify the current theory.

## Text manipulation

Delegate repetitive transformations to search tools, structured queries, scripts, or small one-off programs instead of error-prone manual editing. Preview broad changes, keep them reviewable, and verify the result through diffs and relevant system checks.

Choose the simplest tool that handles the actual data safely. Preserve a reusable command only when the task is likely to recur.

## Engineering journal

Keep a dated working record of commands, observations, experiments, diagrams, decisions, and open questions. It can be rough; its purpose is to preserve investigative context and prevent repeated dead ends.

Move durable decisions and operational knowledge into shared project documentation. A private journal supports thinking but must not become the team's only source of truth.
