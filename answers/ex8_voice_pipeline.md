# Ex8 — Voice pipeline

## Your answer

In Ex8 I tested both the primary text mode and the real voice mode.

First I ran text mode with `uv run python -m starter.voice_pipeline.run --text`. The successful text session was `sess_6d37f91a65ba`. I asked Alasdair to book Haymarket Tap for 6 people on `2026-04-25` at `19:30` with a `£200` deposit. The manager accepted the booking, asked for a contact number, and confirmed after I provided `00000000000`. The narration for that session showed matching `voice.utterance_in` and `voice.utterance_out` events for every turn.

Then I tested real voice mode with Speechmatics STT in session `sess_12c9a188f4ba`. I ran `uv run python -m starter.voice_pipeline.run --voice`. Rime was not configured, so the system degraded gracefully: it printed manager replies instead of speaking them aloud. Speechmatics still captured my spoken input and produced audio artifacts in the workspace. The narration shows two real voice turns: I said “Hi, I would love some help with booking...” and then “Um 12.” Alasdair responded by asking how many people were in the party, then rejected the party of 12 as too large and suggested The Royal Oak for larger groups.

This confirms the Ex8 trace contract in both modes: each user utterance is recorded as `voice.utterance_in`, and each manager reply is recorded as `voice.utterance_out`. Voice mode also wrote recorded input files, including `workspace\turn_0_input.wav` and `workspace\turn_1_input.wav`.

## Citations

- Text session: `sess_6d37f91a65ba`
- Text trace: `C:\Users\const\AppData\Local\sovereign-agent\homework\ex8\sess_6d37f91a65ba\logs\trace.jsonl`
- Voice session: `sess_12c9a188f4ba`
- Voice trace: `C:\Users\const\AppData\Local\sovereign-agent\homework\ex8\sess_12c9a188f4ba\logs\trace.jsonl`
- Voice artifacts: `workspace\turn_0_input.wav`, `workspace\turn_1_input.wav`
- Voice narration: `You said: Hi ! I would love some help with booking a book .`, `Agent said: Too many, I'm afraid. Try The Royal Oak, they can handle larger groups.`