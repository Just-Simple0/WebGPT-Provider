# Architecture Used in the Experiment

## Overview

The experiment used three major layers:

```text
Codex CLI
  -> OpenCodex routing/control plane
    -> local WebGPT provider (OpenAI Responses-compatible facade)
      -> Playwright-driven visible ChatGPT Web browser
```

The goal was to make ChatGPT Web behave like a routed provider while keeping the browser visible and avoiding hidden browser-session extraction.

## 1. Codex / OpenCodex layer

OpenCodex acted as the routing and model-catalog layer.

Target checkout used during the experiment:

```text
repository: lidge-jun/opencodex
version:    2.56.0
commit:     e4a8539b957b7ae7cd278666f0364eb0f82d4ac3
```

The local patched checkout was launched from source with the pinned Bun runtime rather than the older globally installed `ocx` binary.

Important routing concepts:

- `webgpt/gpt-6` was added as a routed custom model.
- Exact five-level reasoning metadata was preserved for this model.
- `multiAgentMode` remained `v1` because the routed v1 subagent path was successfully exercised.
- A focused Codex smoke used `features.code_mode.direct_only_tool_namespaces=["multi_agent_v1"]` so the v1 collaboration namespace stayed directly visible while the rest of the larger tool surface remained compact.

## 2. Local WebGPT provider

The provider exposed a local OpenAI-compatible surface on:

```text
http://127.0.0.1:8791/v1
```

Major responsibilities:

- `/v1/models` model metadata;
- `/v1/responses` request handling;
- JSON and buffered SSE response modes;
- declared function/custom/namespace tool metadata;
- local request receipts / idempotency tracking;
- dedicated browser profile;
- one-browser-owner concurrency control;
- browser selection and exact readback verification.

The provider never intentionally relied on cookie/token extraction from the browser profile.

## 3. Browser adapter

The browser adapter automated visible ChatGPT Web.

Phase-1 selection target:

```text
Family: Latest
Numeric readback: GPT-6
```

Observed UI properties included:

- one reasoning slider;
- Latest/GPT-5.6/GPT-5.5 family options;
- Pro as a separate disabled/available state depending on quota;
- Korean labels such as `매우 높음` for XHigh.

The adapter treated UI state as a contract that had to be read back before prompt submission.

## Effort selection

Original hard assumption:

```text
slider min=0, max=4
```

Live quota state disproved that assumption. When Pro was unavailable, ChatGPT exposed:

```text
min=0
max=3
now=3
label=매우 높음
Pro row disabled
```

The corrected contract became:

```text
0 Instant
1 Medium
2 High
3 XHigh
4 Pro, only when actually available
```

`max` never silently fell back to index 3.

## Tool-call bridge design

The provider transformed each sampling request into a browser-visible prompt containing:

- a request identifier;
- the model-visible request data;
- declared tools;
- a required outer tool/final reply envelope.

The intended model reply shapes were conceptually:

```json
{"request_id":"req_...","type":"final","text":"..."}
```

or

```json
{
  "request_id":"req_...",
  "type":"calls",
  "calls":[...]
}
```

The provider then parsed and validated the returned text before translating it back into Responses-compatible output items.

This architecture was sufficient for bounded final replies and some simple function calls, but it became the core reliability problem once arbitrary nested shell/code arguments appeared.

## Browser concurrency

The provider intentionally supported one browser owner at a time.

Initial behavior:

```text
second concurrent request -> HTTP 409 browser_busy
```

This broke v1 subagents because the parent and child could overlap. The final experimental behavior serialized requests on the existing async lock:

```text
request A owns browser
request B waits
A completes
B proceeds
```

If A left the ledger in an uncertain post-submit state, B re-checked the ledger after acquiring the lock and failed closed rather than proceeding.

## Context-pressure mitigation during testing

Focused Codex tests disabled nonessential skills/apps/plugins to isolate provider behavior:

```text
orchestrator.skills.enabled=false
skills.include_instructions=false
features.apps=false
features.plugins=false
features.tool_suggest=false
```

This was diagnostic isolation only, not a proposed production default.

## Architectural weakness that ended the project

The browser bridge relied on ChatGPT Web to produce text that was itself syntactically valid structured tool-call JSON. That made nested payload correctness depend on model-authored escaping.

For arbitrary code/shell arguments, this is a poor transport boundary:

```text
semantic tool intent
   + JSON syntax
   + nested payload escaping
all authored by the model
```

The project stopped instead of continuing to strengthen prompts around that weakness.

## Preferred architecture for a replacement repository

Prefer an upstream implementation where at least one of the following is true:

1. tool calls arrive through a native structured channel;
2. the model-facing grammar is simple and the adapter performs deterministic JSON serialization;
3. Codex/OpenAI Responses compatibility is already an explicit design target with tests for nested tool arguments.
