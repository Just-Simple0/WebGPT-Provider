# Decisions and Rationale

## D1. Use a visible browser only

Decision:

- ChatGPT Web automation remained visible.
- No browser cookie/token extraction was used as an integration shortcut.

Reason:

The experiment was intended to remain a user-visible browser workflow rather than turning the browser profile into an undocumented authentication source.

## D2. Freeze phase 1 on Latest / GPT-6

Decision:

Phase 1 targeted only:

```text
webgpt/gpt-6
```

with ChatGPT Web `Latest` selected and GPT-6 numeric readback verified.

Reason:

Family switching to GPT-5.6 Sol and GPT-5.5 introduced a separate UI-control problem. It was intentionally deferred until the core main/subagent transport was proven.

## D3. Keep five local effort values

Decision:

```text
low, medium, high, xhigh, max
```

were preserved as the OpenCodex-facing ladder.

Reason:

The user wanted a stable five-level interface even though the live ChatGPT Web UI could temporarily expose only four active slider positions when Pro was unavailable.

## D4. Never silently degrade Max/Pro to XHigh

Decision:

When the live UI showed `aria-valuemax=3`, `max`/Pro requests failed rather than being mapped to XHigh.

Reason:

Exact effort semantics mattered more than graceful fallback.

## D5. Keep `multiAgentMode=v1`

Decision:

Use Codex/OpenCodex v1 collaboration tools for the routed WebGPT model.

Reason:

The v1 routed heterogeneous path was demonstrably workable. It successfully spawned and completed a `webgpt/gpt-6 xhigh` child.

## D6. Expose only the v1 collaboration namespace directly during focused tests

Decision:

Use the Codex override:

```text
features.code_mode.direct_only_tool_namespaces=["multi_agent_v1"]
```

Reason:

Broadly disabling code-mode wrapping caused the entire direct tool surface to expand and triggered context-pressure problems. Keeping only `multi_agent_v1` direct preserved compactness while still allowing `spawn_agent`/`wait_agent` to be callable.

## D7. Serialize browser ownership instead of increasing concurrency

Decision:

Keep exactly one browser owner and queue overlapping requests.

Reason:

The browser/profile/UI state was not designed for multiple simultaneous workers. The actual subagent failure was not lack of parallel browsers; it was the immediate 409 rejection of the second request.

## D8. Fail closed after uncertain submission

Decision:

A queued request re-checks ledger uncertainty after acquiring the browser lock and does not continue if the prior owner ended in an uncertain post-submit state.

Reason:

Avoid duplicate or mis-bound browser submissions.

## D9. Do not normalize malformed arbitrary tool JSON silently

Decision:

The strict parser continued rejecting syntactically invalid model output.

Reason:

Repairing malformed shell/code payloads heuristically risks changing tool semantics and executing something the model did not actually specify.

## D10. Stop prompt-hardening as the primary solution

Decision:

The project was stopped after strict-JSON prompt hardening still failed to make arbitrary agentic tool calls reliable.

Reason:

At that point the work had become repeated prompt engineering around a structural transport weakness. This no longer matched the user's goal of obtaining a dependable Codex/OpenCodex provider.

## D11. Adopt an upstream repository instead

Decision:

Freeze this experiment as an evidence archive and move to an existing project that already solves more of the transport/tool-call problem.

Reason:

The most valuable output of this project is now the compatibility knowledge and test matrix, not the accumulated custom patches.

## D12. Do not treat successful subagent final-only tasks as proof of arbitrary tool robustness

Decision:

Keep the capability boundary explicit:

```text
subagent no-tool child: proven
arbitrary agentic tool child: not proven
```

Reason:

The transport failure only appeared once nested tool arguments became complex. Conflating those cases would overstate the implementation's maturity.
