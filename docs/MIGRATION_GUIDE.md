# Migration Guide for an Upstream Replacement

The next project direction is to adopt an existing repository rather than continue the bespoke WebGPT Provider patch series.

This guide defines what to evaluate and what knowledge is worth carrying forward.

## 1. First rule: reproduce upstream unchanged

Before porting anything from this archive:

1. clone the candidate repository;
2. install it exactly as documented;
3. run its own tests/examples;
4. prove its browser/login/model flow without local patches;
5. record the exact upstream commit/version used.

Do not begin by copying browser selectors or WebGPT patches into the new project.

## 2. Capability checklist

Evaluate the replacement repository against these requirements.

### Browser/session

- visible browser operation is possible;
- dedicated profile/session storage is supported;
- login can be completed manually without exporting browser cookies/tokens;
- UI readiness and redirect/load states are handled explicitly;
- concurrency behavior is documented.

### Model selection

Phase-1 requirement:

```text
ChatGPT Web Latest / GPT-6
```

Need to determine whether the upstream project can:

- select Latest;
- verify actual selected family/state;
- distinguish stale UI state from successful selection.

Phase-2 optional requirement:

```text
GPT-5.6 Sol
GPT-5.5
```

Do not start phase 2 until the phase-1 transport is stable.

### Effort selection

Target local ladder:

```text
low
medium
high
xhigh
max
```

Important live-UI behavior to preserve:

- XHigh must remain exact when Pro is unavailable;
- Pro/Max must not silently fall back to XHigh;
- the active slider may expose `0..3` rather than `0..4` when Pro is disabled.

### OpenAI/Codex compatibility

Prefer an upstream project that already supports:

- `/v1/responses` or an equivalent Responses adapter;
- `client_metadata` or transparent passthrough/validation for current Codex clients;
- function/custom/freeform/namespace tools;
- tool-result continuation;
- strict request/response identity binding;
- streaming semantics compatible with Codex/OpenCode clients.

### Tool-call transport

This is the highest-priority evaluation item.

Good signs:

- tool calls come from a structured browser/native channel;
- tool arguments are deterministically serialized by adapter code;
- nested shell/code payloads are covered by tests;
- malformed tool data fails closed without changing semantics;
- model text is not expected to recreate the full API JSON envelope manually.

Red flag:

```text
"Tell the model to output the exact tool-call JSON and parse the text"
```

That was the principal failure mode of this experiment.

### Multi-agent behavior

Need to test:

- parent spawn;
- explicit child model selection;
- child reasoning effort;
- no-history / bounded-context child mode;
- wait/result propagation;
- overlapping parent/child requests with a single browser session.

The known-good historical child case was:

```text
webgpt/gpt-6
xhigh
fork_context=false
```

### Concurrency

If the replacement uses one browser/profile, it should queue work rather than immediately reject a second legitimate worker.

Required invariants:

```text
one active browser owner
bounded queue/wait
fail closed after uncertain submission
no duplicate prompt submission
```

## 3. Compatibility tests to run in order

Do not jump directly to a large agent task.

### T1 — plain main turn

Expected exact response:

```text
CODEX_WEBGPT_MAIN_OK
```

### T2 — simple function call

Use a function with short scalar arguments and a deterministic local result.

Verify:

- call identity;
- argument parsing;
- result continuation;
- final response.

### T3 — no-tool subagent

Spawn one child:

```text
model = webgpt/gpt-6
reasoning = xhigh
no inherited long history
```

Expected child result:

```text
WEBGPT_SUBAGENT_OK
```

### T4 — overlap behavior

Verify that parent/child overlap does not produce immediate browser-busy failure.

### T5 — nested tool argument

Use a tool argument that contains:

- spaces;
- quotes;
- backslashes;
- newline;
- code/shell text.

This test is critical because it is where the bespoke provider failed.

### T6 — disposable agentic harness

Use only a temp directory and verify:

- parent tool use;
- child tool use;
- write/readback;
- parent verification;
- no access outside the fixture directory.

Only after T6 passes should the replacement be considered a real substitute for arbitrary Codex agent work.

## 4. Knowledge worth porting

These ideas are worth reusing if the upstream project lacks them:

- exact effort readback rather than assuming click success;
- quota-dependent slider-domain handling;
- explicit Latest/GPT-6 verification;
- dedicated visible browser profile;
- fail-closed post-submit receipts;
- bounded single-browser queue;
- strict no-fallback effort semantics;
- focused compatibility tests in the order listed above.

## 5. Knowledge not worth porting directly

Do **not** copy these by default:

- accumulated v1-v15 browser patch files;
- selectors from old UI snapshots without fresh readback;
- prompt text that asks the model to hand-author full Responses tool JSON;
- malformed-JSON repair heuristics;
- assumptions tied specifically to Codex 0.150.x without checking the current version.

## 6. OpenCodex-specific historical notes

Historical OpenCodex environment:

```text
repo:    lidge-jun/opencodex
version: 2.56.0
commit:  e4a8539b957b7ae7cd278666f0364eb0f82d4ac3
```

Important lessons:

- routed model metadata can materially change which tools Codex exposes;
- hosted search removal must occur at an actual tool-policy seam rather than by deleting a catalog field that Codex re-defaults;
- direct collaboration-tool exposure should be targeted, not broad;
- current Codex versions should be treated as a new compatibility target rather than assuming old behavior.

## 7. Adoption decision template

For each candidate repository, record:

```text
Repository:
Commit/tag:
Maintenance activity:
License:
Browser mechanism:
Auth/session mechanism:
Responses API support:
Structured tool-call mechanism:
Nested argument handling:
Concurrency model:
GPT-6/Latest selection:
Effort selection/readback:
Subagent compatibility:
T1 result:
T2 result:
T3 result:
T4 result:
T5 result:
T6 result:
Required local patches:
Decision: ADOPT / REVISE / REJECT
```

## 8. Final migration rule

The replacement should reduce custom protocol code, not simply relocate it.

A repository that still requires large amounts of prompt engineering to emulate arbitrary structured tool calls should be rejected even if its basic browser automation is convenient.
