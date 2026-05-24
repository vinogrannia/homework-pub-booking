# Ex7 — Handoff bridge

## Your answer

In Ex7 I ran the handoff bridge successfully in a persisted session: `sess_5da94d49e14e`.

The bridge completed in two rounds. In round 1, the bridge started the loop half, the planner produced one subgoal, and the loop called `venue_search` with `near='Haymarket'` and `party_size=12`. That search returned `0 result(s)`. The loop then called `handoff_to_structured`, passing the booking attempt to the structured half. The trace then recorded the state transition `loop → structured` for round 1.

The important part is that structured did not complete the booking in round 1. The trace shows the reverse transition `structured → loop`, which means the bridge returned control to the loop side instead of failing the whole scenario. This is the core Ex7 behaviour: a rejection from the structured half becomes a new loop-side attempt.

In round 2, the bridge started the loop half again. This time the loop called `venue_search` with `near='Old Town'` and `party_size=6`, which returned `1 result(s)`. The loop again called `handoff_to_structured`, with the summary “retry after reverse handoff — scaled down to fit policy”. The trace then recorded `loop → structured` for round 2, followed by `structured → complete`. The final run output was `Bridge outcome: completed`, `rounds: 2`, and `summary: structured confirmed in round 2`.

This run demonstrates the bridge pattern: the loop half can propose, the structured half can reject, and the bridge can route control back to the loop half for a corrected attempt.

## Citations

- Ex7 session: `sess_5da94d49e14e`
- Trace path: `C:\Users\const\AppData\Local\sovereign-agent\examples\ex7-handoff-bridge\sess_5da94d49e14e\logs\trace.jsonl`
- Narration lines: `State: loop → structured (round 1)`, `State: structured → loop (round 1)`, `State: loop → structured (round 2)`, `State: structured → complete (round 2)`
- Run output: `Bridge outcome: completed`, `rounds: 2`, `structured confirmed in round 2`