# WebGPT Provider — Experiment Archive

> **Status: STOPPED / ARCHIVED AS A REFERENCE**
>
> This repository records the WebGPT Provider experiment carried out in September 2026. The project proved that ChatGPT Web could be bridged into a Codex/OpenCodex workflow for ordinary model turns and a bounded v1 subagent flow, but the custom browser-to-Responses tool bridge became too brittle once arbitrary agentic tool calls were introduced. Further incremental prompt/protocol patching was intentionally stopped. The next direction is to adopt an existing upstream repository and reuse only the verified lessons from this experiment.

## What was proven

- `webgpt/gpt-6` could be exposed to Codex/OpenCodex as a routed model.
- ChatGPT Web `Latest` mapped successfully to the GPT-6 family readback.
- Five local effort levels were modeled as `low`, `medium`, `high`, `xhigh`, `max`.
- Main-turn end-to-end execution succeeded at XHigh.
- Function-tool round-trip succeeded in the provider's own smoke test.
- Codex v1 subagent spawn completed end-to-end with `webgpt/gpt-6 xhigh`, `fork_context:false`, and a successful child final result.
- Browser concurrency remained one owner at a time, while queued parent/child requests were serialized instead of failing immediately with HTTP 409.

## What did not reach a robust state

Arbitrary agentic tool use through Codex remained unreliable because the browser bridge asked ChatGPT Web to serialize tool calls as text and then parsed that text back into Responses-style tool call objects. Once nested shell/code payloads appeared, model-generated JSON escaping became a recurring failure mode.

The experiment therefore stopped before treating ChatGPT Web as a general-purpose structured tool-call transport.

## Important scope boundary

This was an unofficial browser automation experiment. It did not use a documented OpenAI API path for ChatGPT Web, and nothing here should be interpreted as an OpenAI-supported integration.

## Repository map

- [`HANDOFF.md`](HANDOFF.md) — concise continuation handoff and current decision.
- [`docs/CURRENT_STATE.md`](docs/CURRENT_STATE.md) — final technical state and verified capabilities.
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — architecture used during the experiment.
- [`docs/TEST_EVIDENCE.md`](docs/TEST_EVIDENCE.md) — successful and failed runtime evidence.
- [`docs/DECISIONS.md`](docs/DECISIONS.md) — key decisions and why the project was stopped.
- [`docs/PATCH_HISTORY.md`](docs/PATCH_HISTORY.md) — patch/revision history and lessons.
- [`docs/MIGRATION_GUIDE.md`](docs/MIGRATION_GUIDE.md) — what to carry into an upstream-repository adoption.
- [`docs/ARTIFACT_INDEX.md`](docs/ARTIFACT_INDEX.md) — local experimental scripts/artifacts and supersession notes.

## Final project decision

Do **not** continue the v15-style prompt-hardening path from this snapshot. When evaluating a replacement repository, prefer an implementation that already has a deterministic structured tool-call adapter or a native transport for tool calls, rather than depending on free-form model text to recreate Responses API objects.
