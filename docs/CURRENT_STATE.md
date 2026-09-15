# Current State at Project Stop

## Project phase

The custom provider experiment stopped after the following boundary was reached:

- constrained main-model use: **working**;
- constrained v1 subagent use: **working**;
- arbitrary agentic tool bridge: **not robust**.

## Provider surface that worked

### Model

Primary phase-1 model:

```text
webgpt/gpt-6
```

Display intent:

```text
ChatGPT Web · Latest
```

Later family switching to GPT-5.6 Sol and GPT-5.5 was explicitly deferred and never completed as a stable phase.

### Effort mapping

Local/OpenCodex effort values:

```text
low     -> Instant
medium  -> Medium
high    -> High
xhigh   -> XHigh / 매우 높음
max     -> Pro
```

The live ChatGPT UI changed when Pro was unavailable:

```text
Normal / Pro available:
aria-valuemin = 0
aria-valuemax = 4

Pro unavailable / quota-disabled:
aria-valuemin = 0
aria-valuemax = 3
aria-valuenow = 3  # XHigh
```

The final logic treated XHigh as exact index 3 in both states and refused to silently map `max`/Pro to XHigh when Pro was unavailable.

## Main-turn status

Verified real Codex run:

```text
Model changed to webgpt/gpt-6 xhigh
```

Prompt:

```text
Reply with exactly CODEX_WEBGPT_MAIN_OK. Do not use tools.
```

Observed result:

```text
CODEX_WEBGPT_MAIN_OK
```

## Function-tool status

A provider-level two-turn function-tool smoke succeeded with:

- one declared function call;
- local execution;
- tool result returned in the continuation turn;
- final response reflecting the tool result;
- request receipts verified for both turns.

This proved the provider's own declared-function protocol in a bounded case.

## v1 subagent status

A real Codex v1 subagent smoke completed successfully.

Successful child configuration:

```text
model: webgpt/gpt-6
reasoning_effort: xhigh
fork_context: false
```

Observed UI/runtime sequence:

```text
Spawned ... (webgpt/gpt-6 xhigh)
Waiting for ...
Finished waiting
Completed - WEBGPT_SUBAGENT_OK
WEBGPT_SUBAGENT_OK
```

This is the strongest positive result from the experiment: the routed WebGPT model was usable as a real Codex v1 child agent for a no-tool child task.

## Browser concurrency status

Original behavior:

```text
HTTP 409 Conflict
One WebGPT request already owns the browser.
```

A later service change preserved single-browser ownership but serialized overlapping requests on the in-process lock instead of immediately failing the second request.

Offline verification covered:

```text
normal_serialization = true
uncertain_recheck_fail_closed = true
```

The design still intentionally allowed only one active browser owner at a time.

## Remaining blocker at stop

The first deterministic agentic harness attempted to exercise arbitrary Codex tool calls. It failed before completing the parent tool step.

Sanitized diagnostics showed:

```text
reply starts with {
reply ends with }
request_id present
type present
calls present
strict JSON parse failed
error: Expecting ',' delimiter
```

The error occurred at a quote boundary within the payload. This was consistent with malformed JSON caused by nested shell/code content being emitted without correct escaping.

The project stopped rather than continuing prompt-level attempts to make the model hand-author strict Responses-compatible JSON.

## Final operating conclusion

Treat the following as proven:

```text
ChatGPT Web -> routed main model        YES
GPT-6 Latest selection                 YES
XHigh selection                        YES
v1 subagent no-tool child              YES
single-browser queued serialization     YES
arbitrary structured agentic tools      NO
```

The repository should therefore be used as an evidence archive and migration reference, not as a production-ready provider implementation.
