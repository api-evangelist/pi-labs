---
name: Score an LLM output with Pi Labs
description: >-
  Use the Pi Labs (Pi Client) scoring API to evaluate an LLM output against a
  rubric of natural-language questions and read back a total_score, so an agent
  can gate, rank, or retry generations deterministically.
api: https://api.withpi.ai
docs: https://code.withpi.ai/quickstart
method: generated
source: https://code.withpi.ai/quickstart
operations:
  - scoring_system.score
---

# Score an LLM output with Pi Labs

Pi Labs (`withpi`) exposes a deterministic ~200ms foundation-model scorer that
judges any text against a rubric of natural-language questions. Use it to grade,
gate, or rank model outputs instead of an LLM-as-a-judge.

## Prerequisites

1. Get a free API key from your Pi Labs account and export it:
   `export WITHPI_API_KEY=...`
2. Install the official SDK:
   - Python: `pip install withpi`
   - TypeScript: `npm install withpi`

Authentication is a single API key sent as an `Authorization` bearer header; the
SDK reads it from `WITHPI_API_KEY`. See `authentication/pi-labs-authentication.yml`.

## Steps

1. Construct a `scoring_spec` — an array of `{ "question": "..." }` rubric items,
   each a yes/no-style natural-language criterion (e.g. "Is there a strong call
   to action?", "Is the response truthful?", "Is the response relevant?").
2. Call `scoring_system.score` with the input, the candidate output, and the spec:

   ```python
   from withpi import PiClient
   import os

   pi = PiClient(api_key=os.environ["WITHPI_API_KEY"])

   scores = pi.scoring_system.score(
       llm_input="Pi Labs",
       llm_output="Score anything with Pi Labs today!",
       scoring_spec=[{"question": "Is there a strong call to action?"}],
   )
   ```

3. Read `scores.total_score` (0..1) plus the per-question breakdown. Apply your
   threshold to gate/accept the output, or feed the score back into a retry or
   optimization loop.

## Conventions and error handling

- The API is OpenAPI-compatible JSON over HTTPS at `https://api.withpi.ai`.
- No idempotency key or pagination is documented for the scoring surface — see
  `conventions/pi-labs-conventions.yml`.
- Treat non-2xx responses as transient/auth failures and surface the HTTP status;
  Pi Labs does not publish an RFC 9457 problem+json error catalog.

> Grounding note: only `scoring_system.score` is used here because it is the one
> operation Pi Labs documents verbatim in its quickstart. Do not invent
> additional operationIds — extend this skill only against the published spec.
