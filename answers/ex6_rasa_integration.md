# Ex6 — Rasa structured half

## Your answer

In Ex6 I verified the Rasa structured half in both mock mode and real three-terminal mode.

The Python side starts in `starter/rasa_half/validator.py`. `normalise_booking_payload` takes loose booking data and converts it into the shape Rasa expects at its REST webhook. In my run, the raw booking contained values like `venue_id="Haymarket Tap"`, `date="25th April 2026"`, `time="7:30pm"`, `party_size="6"`, and `deposit="£200"`. The validator normalised these into `venue_id="haymarket_tap"`, `date="2026-04-25"`, `time="19:30"`, `party_size=6`, and `deposit_gbp=200`.

`starter/rasa_half/structured_half.py` then sends that normalised payload to Rasa with an HTTP POST to `/webhooks/rest/webhook`. It parses the Rasa response and returns a `HalfResult`: confirmed bookings become `next_action="complete"`, while rejected or failed bookings become `next_action="escalate"`.

The real Rasa validation happens in `rasa_project/actions/actions.py`. `ActionValidateBooking` reads `tracker.latest_message.metadata.booking`, sets Rasa slots, and applies the policy rules: party size must be no more than 8 and deposit must be no more than £300. If the booking passes, it clears `validation_error` and creates a booking reference.

I first ran Ex6 mock mode successfully. Then I ran Ex6 real mode with three processes: the action server on port `5055`, the Rasa server on port `5005`, and the scenario in a third terminal. The real run confirmed the booking and returned reference `BK-7D401E9E`.

## Citations

- `starter/rasa_half/validator.py`
- `starter/rasa_half/structured_half.py`
- `rasa_project/actions/actions.py`
- Ex6 real session: `sess_0ed8a81100b6`
- Ex6 real output: `Structured half outcome: complete`, `booking confirmed by rasa (ref=BK-7D401E9E)`