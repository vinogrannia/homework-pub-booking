# Ex5 — Edinburgh research loop scenario

## Your answer

In Ex5 I implemented the Edinburgh research tools in `starter/edinburgh_research/tools.py`.

The `venue_search` tool loads `venues.json` from `sample_data`, checks that the fixture exists, and filters venues by four conditions: the pub must be open, its area must match the requested location case-insensitively, it must have enough evening seats for the party size, and `hire_fee_gbp + min_spend_gbp` must fit within `budget_max_gbp`. Matching venues are returned in `results`, and the tool records its output with `record_tool_call`.

The `get_weather` tool loads `weather.json`, normalises the city name, checks that both the city and requested date exist, and returns structured weather data. If the city or date is missing, it returns `ToolResult(success=False)` with `SA_TOOL_INVALID_INPUT` rather than crashing.

The `calculate_cost` tool loads `catering.json` and `venues.json`, finds the venue by `venue_id`, validates the catering tier, and computes subtotal, service charge, total cost, and required deposit using the provided pricing formula and deposit thresholds.

The `generate_flyer` tool validates required event fields, builds a self-contained `flyer.html` with inline CSS, tags key facts with `data-testid`, writes it to `workspace/flyer.html`, logs the call, and returns a `ToolResult`.

I ran Ex5 offline successfully. The run created `flyer.html` and ended with `dataflow OK: verified 4 fact(s) against tool outputs`.

I also tested the fabrication check by creating a flyer-like file containing the fake value `£9999`, then calling `verify_dataflow` after repopulating `_TOOL_CALL_LOG` with real tool outputs. The check correctly failed with: `dataflow FAIL: 1 unverified fact(s): ['£9999']`.

Finally, I ran Ex5 in real Nebius mode in session `sess_8ef7efd582cd`. That run did not complete the flyer pipeline: the live LLM called `venue_search` three times with `party_size=50` and increasingly broad locations, then requested `handoff_to_structured` before calling `get_weather`, `calculate_cost`, or `generate_flyer`. This matched the README warning that real models may spiral or hand off too early.

## Citations

- `starter/edinburgh_research/tools.py`
- Offline Ex5 run: `sess_b89e2cb12279`, `dataflow OK: verified 4 fact(s) against tool outputs`
- Fabrication test output: `dataflow FAIL: 1 unverified fact(s): ['£9999']`
- Real Nebius Ex5 run: `sess_8ef7efd582cd`
- Real run trace: `C:\Users\const\AppData\Local\sovereign-agent\examples\ex5-edinburgh-research\sess_8ef7efd582cd\logs\trace.jsonl`