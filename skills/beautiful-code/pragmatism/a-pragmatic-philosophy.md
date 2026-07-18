# Engineering Mindset

## Agency

**Review question:** Are contributors empowered to improve the code, process, and environment?

Treat career and technical constraints as choices to revisit, not permanent facts. Ask for change, run a small experiment, learn the missing skill, or move on when a situation cannot improve.

## Responsibility

**Review question:** Does the work make ownership, risks, and failures explicit?

Own commitments and outcomes. When something cannot be delivered, explain the cause plainly, offer options, and avoid excuses. Do not promise what evidence cannot support.

### Don't: offer excuses instead of options

A migration misses its date and the engineer reports "the vendor API was flaky, nothing we could
do," leaving the team with no path forward and no earlier warning that the risk existed.

### Do: own the outcome and present choices

The engineer flags the flaky vendor API the day it appears, and at the deadline reports: "We can
ship with a retry queue now, or wait two weeks for the vendor fix. Here is the risk of each."

## Software entropy

**Review question:** Does this change repair disorder or normalize it?

Visible neglect invites more neglect. Fix small defects when practical; otherwise isolate and track them. Do not use existing mess as permission to lower the standard nearby.

### Don't: normalize the existing mess

While fixing a bug, an engineer notices the module has three dead feature flags and a misleading
function name, and copies the same style into the fix because "that's how this file works."

### Do: repair small disorder as you pass

The same engineer deletes the dead flags in a separate commit, renames the misleading function,
and files a tracked issue for the larger cleanup that would not fit in this change.

## Catalyzing change

**Review question:** Can the improvement prove its value in a small, concrete slice?

Build enough of a useful result that others can see and extend it. Invite participation instead of demanding wholesale adoption. At the same time, watch gradual degradation: regularly reassess accumulated compromises.

## Good-enough software

**Review question:** Is the quality target explicit and appropriate to the risk?

Quality is a product decision, not a pursuit of perfection. Agree on acceptance criteria and trade-offs with stakeholders. Ship when the result meets them; never compromise safety, integrity, or essential correctness.

## Continuous learning

**Review question:** Is the team expanding and refreshing the knowledge needed to maintain this system?

Invest steadily rather than cramming during a crisis. Diversify technical and domain knowledge, test claims against primary sources, and turn learning into experiments. Revisit assumptions as evidence changes.

## Communication

**Review question:** Is the message shaped for its audience and easy to act on?

Know the goal, choose the right medium and timing, and lead with what matters. Make documents and code legible, invite feedback, and answer requests. Clear communication includes listening and closing the loop.
