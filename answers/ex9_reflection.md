# Ex9 — Reflection

## Q1 — Planner handoff decision

### Your answer

In my Ex7 run, session `sess_5da94d49e14e`, the clearest handoff point appears in the trace when the loop half calls `handoff_to_structured` and the bridge records the state transition `loop → structured` in round 1. In this run the planner produced one loop-side subgoal first, then the executor searched for a Haymarket venue with `party_size=12`. That search returned `0 result(s)`, so the loop could not complete the booking by itself.

The signal that caused the handoff was that the booking needed a structured policy decision after the loop-side research attempt. The narration shows the tool call summary: `handoff to structured half: loop half identified a candidate venue; passing to structured half...`, followed immediately by `State: loop → structured (round 1)`. The structured half then returned control rather than completing, shown by `State: structured → loop (round 1)`. That reverse transition is the important Ex7 behaviour: the bridge treats rejection as a reason to re-enter the loop instead of ending the session.

In round 2, the loop retried with `near='Old Town'` and `party_size=6`, found `1 result(s)`, handed off again, and the trace ended with `State: structured → complete (round 2)`. So the handoff was not just a static planner label; it became a real bridge state transition.

### Citation

- Ex7 session: `sess_5da94d49e14e`
- Trace: `C:\Users\const\AppData\Local\sovereign-agent\examples\ex7-handoff-bridge\sess_5da94d49e14e\logs\trace.jsonl`
- Narration lines: `venue_search near='Haymarket', party=12 → 0 result(s)`, `State: loop → structured (round 1)`, `State: structured → loop (round 1)`, `State: structured → complete (round 2)`

---

## Q2 — Dataflow integrity catch

### Your answer

I did not see `verify_dataflow` fail during my successful offline Ex5 run: that run produced `flyer.html` and ended with `dataflow OK: verified 4 fact(s) against tool outputs`. To test the failure path deliberately, I constructed the exact fabrication case described in the assignment.

I created a small flyer-like file containing `Total: £9999. Weather: cloudy, 12C.` Then I repopulated `_TOOL_CALL_LOG` by calling the real tools: `venue_search('Haymarket', 6, 800)`, `get_weather('Edinburgh', '2026-04-25')`, and `calculate_cost('haymarket_tap', 6, 3, 'bar_snacks')`. Those calls produced real venue, weather, and cost facts, but none of them produced `£9999`.

When I ran `verify_dataflow` against the fabricated file, it returned: `dataflow FAIL: 1 unverified fact(s): ['£9999']`. That is exactly the failure mode the check is meant to catch. A human reviewer might skim a flyer and miss a plausible-looking price, especially if the rest of the flyer is coherent. The integrity check does not ask whether a value looks reasonable; it asks whether that exact value appeared in a previous tool output. That is stronger than manual review.

### Citation

- Ex5 successful offline run output: `sess_b89e2cb12279`, `dataflow OK: verified 4 fact(s) against tool outputs`
- Manual fabrication test output: `dataflow FAIL: 1 unverified fact(s): ['£9999']`
- Relevant implementation: `starter/edinburgh_research/integrity.py`, `verify_dataflow`
- Tool logging implementation: `starter/edinburgh_research/tools.py`, calls to `record_tool_call`

---

## Q3 — Production failure and primitive

### Your answer

The first production failure I would expect is: the live LLM exits the required workflow before producing the required user-facing artifact. I saw this in my real Ex5 run, session `sess_8ef7efd582cd`. The live model called `venue_search` three times with `party_size=50` and increasingly broad locations: `Old Town`, `City Centre`, and `Edinburgh`. All three searches returned `0 result(s)`. Instead of recovering by changing strategy or completing the required Ex5 pipeline, it requested `handoff_to_structured`. The run ended with `No flyer written to workspace/. Ex5 failed.`

The one sovereign-agent primitive I would rely on to surface this is the ticket state machine. The tickets showed that the planner and executor tickets themselves completed successfully, but the overall loop outcome was `handoff_to_structured`, not a completed flyer workflow. That distinction matters in production: a tool call or ticket can succeed locally while the business task still fails. The ticket state machine gives a concrete audit trail of what ran, what succeeded, and where the scenario stopped.

This is the kind of failure I would expect in a real pub-booking business: the agent performs valid individual actions, but never delivers the promised artifact or final booking state. The ticket state machine makes that visible instead of hiding it behind a fluent LLM answer.

### Citation

- Ex5 real session: `sess_8ef7efd582cd`
- Trace: `C:\Users\const\AppData\Local\sovereign-agent\examples\ex5-edinburgh-research\sess_8ef7efd582cd\logs\trace.jsonl`
- Run output: `Loop half outcome: handoff_to_structured`, `No flyer written to workspace/. Ex5 failed.`
- Narration: `venue_search near='Old Town', party=50`, `venue_search near='City Centre', party=50`, `venue_search near='Edinburgh', party=50`, `handoff_to_structured`