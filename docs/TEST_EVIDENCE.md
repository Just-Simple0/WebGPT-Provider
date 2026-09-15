# Test Evidence

This document records the key runtime observations used to decide what was actually working at the time the project was stopped.

## 1. Main-turn end-to-end — PASS

Codex model selection:

```text
Model changed to webgpt/gpt-6 xhigh
```

Prompt:

```text
Reply with exactly CODEX_WEBGPT_MAIN_OK. Do not use tools.
```

Observed final answer:

```text
CODEX_WEBGPT_MAIN_OK
```

Interpretation:

```text
Codex -> OpenCodex -> WebGPT provider -> ChatGPT Web -> Codex
```

worked for a plain no-tool turn.

## 2. Provider function-tool smoke — PASS

A provider-level two-turn smoke verified:

- function call produced;
- local tool execution performed;
- tool result returned to provider;
- continuation response completed;
- both request receipts verified;
- final answer reflected the tool result.

Representative status:

```text
status = function_tool_roundtrip_verified
tool_call_verified = true
tool_result_returned = true
tool_result_reflected = true
both_receipts_verified = true
```

## 3. Latest / GPT-6 family selection — PASS

The browser adapter successfully verified:

```text
selected_option = latest
numeric_family_readback = gpt-6
```

The experiment intentionally stayed on Latest/GPT-6 for phase 1 rather than continuing family switching work.

## 4. Quota-dependent XHigh selection — PASS

Raw DOM inspection during Pro unavailability showed:

```text
aria-valuemin = 0
aria-valuemax = 3
aria-valuenow = 3
label = 매우 높음
Pro = disabled
```

This proved that Pro quota state could shrink the active slider from five positions to four while keeping XHigh at index 3.

The corrected selection logic then allowed exact XHigh verification in this state while keeping Max/Pro unavailable.

## 5. Codex v1 subagent — PASS

Focused test request:

```text
Spawn exactly one subagent.
model: webgpt/gpt-6
reasoning_effort: xhigh
fork_context: false
Ask it to return exactly WEBGPT_SUBAGENT_OK.
```

Observed result:

```text
Spawned ... (webgpt/gpt-6 xhigh)
Waiting for ...
Finished waiting
Completed - WEBGPT_SUBAGENT_OK
WEBGPT_SUBAGENT_OK
```

Interpretation:

- parent could call the v1 collaboration surface;
- routed child model selection worked;
- XHigh override reached the child;
- child completed through WebGPT;
- parent wait/result propagation worked.

This is the definitive subagent success evidence.

## 6. Browser overlap before queue fix — FAIL, then isolated

Observed failure:

```text
409 Conflict
One WebGPT request already owns the browser.
Queue the next worker in the caller.
```

The child had already spawned, which proved the failure was no longer the spawn protocol itself. The failure was browser ownership contention between parent and child requests.

## 7. Serialized browser queue — OFFLINE PASS

After the queue change, an offline verifier reported:

```text
normal_serialization = true
uncertain_recheck_fail_closed = true
```

The test covered:

1. request B waits while request A owns the browser, then proceeds after A completes;
2. if A becomes uncertain after submission, B does not proceed and fails closed.

This preserved one-browser-owner semantics.

## 8. Context-budget issue — observed, then isolated

Several early v1 subagent attempts failed with:

```text
Codex ran out of room in the model's context window.
```

The successful subagent smoke used a fresh thread with nonessential skills/apps/plugins disabled and direct exposure of the v1 collaboration namespace.

This proved that some failures attributed to WebGPT were actually Codex context/tool-surface pressure rather than provider transport errors.

## 9. Agentic parent tool smoke — FAIL

A parent task attempted to create/read a disposable file under `/tmp` using Codex tools.

Observed provider failure:

```text
The completed browser reply did not match the request-bound tool/final protocol.
```

Sanitized reply diagnostics showed:

```text
starts_left_brace = true
ends_right_brace = true
request_id key present
type key present
calls key present
stdlib_json_parse = failed
json_error = Expecting ',' delimiter
```

The parser error occurred at a quote boundary inside the generated payload, consistent with a nested argument string whose internal quote was not escaped correctly.

## 10. Prompt-level strict JSON hardening — insufficient

The provider added stronger post-request framing telling the model to emit strict JSON and properly escape nested argument strings.

The same class of parent-tool failure still occurred.

Interpretation:

Prompt hardening was not a reliable substitute for a deterministic structured transport.

## Final test matrix

| Capability | Result |
|---|---|
| Plain GPT-6 main turn | PASS |
| Latest/GPT-6 selection | PASS |
| Exact XHigh with Pro unavailable | PASS |
| Provider-level function round-trip | PASS |
| Codex v1 child spawn | PASS |
| Child `webgpt/gpt-6 xhigh` | PASS |
| Parent wait / child final propagation | PASS |
| Single-browser queued serialization | PASS offline |
| Arbitrary parent shell/code tool call | FAIL |
| General agentic harness | NOT CLOSED |
| GPT-5.6/GPT-5.5 family switching | NOT COMPLETED |

## Evidence rule for future adoption

When comparing an upstream replacement repository, reproduce these tests in the same order:

```text
main turn
-> simple structured tool
-> subagent no-tool child
-> overlapping parent/child behavior
-> nested shell/code argument tool call
-> full agentic harness
```

Do not treat success in the first three as proof that the arbitrary tool bridge is robust.
