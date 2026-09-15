# HANDOFF — WebGPT Provider Experiment

## Status

**STOPPED. Do not continue incremental protocol/prompt patches from this snapshot.**

This handoff records the state reached before the project direction changed from building a bespoke ChatGPT Web provider to adopting an existing upstream repository and carrying over only the verified lessons.

## Original goal

Expose ChatGPT Web as a usable Codex/OpenCodex provider with:

- `webgpt/gpt-6` as the primary phase-1 model;
- five reasoning levels: `low`, `medium`, `high`, `xhigh`, `max`;
- visible-browser execution only;
- OpenAI Responses-compatible main turns;
- function/custom/namespace tool support;
- Codex v1 subagents;
- later expansion to GPT-5.6 Sol and GPT-5.5 family switching.

## Final verified capabilities

The following were demonstrated in real end-to-end runs:

1. **Main turn** — Codex selected `webgpt/gpt-6 xhigh` and received the exact requested response `CODEX_WEBGPT_MAIN_OK`.
2. **Latest/GPT-6 UI selection** — the provider verified the ChatGPT Web `Latest` family and GPT-6 numeric readback.
3. **Quota-dependent effort UI** — when Pro was unavailable, the live slider changed from `0..4` to `0..3`; XHigh remained index 3 and was verified correctly without silently mapping Max/Pro down to XHigh.
4. **Provider function-tool smoke** — a two-turn function call/result round-trip completed successfully.
5. **Codex v1 subagent** — a child was spawned as `webgpt/gpt-6 xhigh` with `fork_context:false`, completed with `WEBGPT_SUBAGENT_OK`, and the parent successfully waited for and returned the child result.
6. **Browser serialization queue** — parent/child requests no longer had to fail immediately with HTTP 409 when they overlapped; the single-browser-owner design was preserved while requests were queued.

## Final unresolved blocker

A general agentic harness remained unreliable when Codex required arbitrary tool calls containing shell/code-like payloads.

The bridge represented tool calls by asking ChatGPT Web to emit a textual JSON envelope, then parsed that text back into Responses-style tool call objects. A representative failure had all of the expected outer markers (`request_id`, `type`, `calls`) but failed strict JSON parsing at a quote boundary inside the payload, consistent with an unescaped quote in a nested tool argument string.

Prompt-level JSON-escaping hardening was attempted, but continuing that path became prompt engineering rather than a robust transport design. The user explicitly stopped the project at that point.

## Architectural conclusion

The main lesson is not that browser automation is impossible. The proven main/subagent flows show that it can work for constrained cases. The problem is treating free-form ChatGPT Web text as if it were a native arbitrary structured-tool channel.

A replacement implementation should preferably provide one of these:

- a native structured tool-call transport;
- a deterministic adapter whose model-facing grammar does not require the model to hand-author JSON escaping for arbitrary nested payloads;
- an already-tested Codex/OpenAI Responses compatibility layer.

## Important compatibility notes

- The OpenCodex experiment used a patched `2.56.0` checkout pinned at commit `e4a8539b957b7ae7cd278666f0364eb0f82d4ac3`.
- During later runtime testing, the local Codex CLI reported `v0.154.0`; some earlier analysis had been performed against `0.150.x`, so exact behavior must be revalidated against the version used by any replacement repository.
- `multiAgentMode` was kept at `v1` because that path successfully supported routed heterogeneous providers.
- The working Codex smoke environment used a one-shot direct-namespace override for `multi_agent_v1` plus disabled skills/apps/plugins to reduce context pressure during focused verification.

## Known-good focused Codex launch pattern

The successful v1 subagent smoke was performed from a fresh thread with a configuration equivalent to:

```bash
codex \
  -c 'features.code_mode.direct_only_tool_namespaces=["multi_agent_v1"]' \
  -c 'orchestrator.skills.enabled=false' \
  -c 'skills.include_instructions=false' \
  -c 'features.apps=false' \
  -c 'features.plugins=false' \
  -c 'features.tool_suggest=false'
```

Then the child was explicitly requested with:

- model: `webgpt/gpt-6`
- reasoning effort: `xhigh`
- `fork_context:false`

This was a focused diagnostic setup, not a recommended permanent user configuration.

## Security / safety constraints retained throughout

- visible browser only;
- no cookie/token extraction from the browser profile;
- localhost provider binding;
- dedicated browser profile;
- fail-closed handling for uncertain post-submit state;
- no silent effort fallback;
- no raw secret/config dumping in diagnostics.

## What to do next

1. Select an existing upstream repository.
2. Reproduce the upstream project **without modification** first.
3. Compare it against [`docs/MIGRATION_GUIDE.md`](docs/MIGRATION_GUIDE.md).
4. Port only the verified requirements that the upstream project lacks.
5. Keep this repository as historical evidence, not as the base for another round of incremental v16/v17 prompt patches.

## Final handoff statement

**Project status: STOPPED / superseded by upstream-repository adoption.**

Do not resume incremental prompt/protocol patches from this snapshot unless the explicit goal changes back to experimental browser-protocol research.
