# Patch / Revision History

This is a historical map of the experiment. It is not a recommendation to replay these patches.

## Baseline provider

The original local provider implemented:

- `/v1/models`;
- `/v1/responses` JSON + buffered SSE;
- function/custom/freeform/namespace tool metadata;
- tool-result continuation;
- local SQLite receipts/idempotency;
- dedicated visible browser profile;
- localhost binding and origin restrictions;
- single browser owner.

## macOS login fix

Problem:

Google blocked sign-in in the automated Chrome profile because the launch arguments included browser settings such as mock/basic keychain behavior.

Result:

A macOS-specific launch adjustment allowed the dedicated visible Chrome profile to sign in normally without exporting cookies or tokens.

Historical artifact name:

```text
webgpt_macos_login_fix.py
```

## UI menu / slider revisions v1-v3

Purpose:

Adapt browser selectors to the newer ChatGPT reasoning/model menu.

Key discoveries:

- one visible reasoning slider;
- model family rows were `menuitemradio`;
- `Latest`, `GPT-5.6 Sol`, and `GPT-5.5` were nested in the same menu;
- earlier selectors assumed a different menu structure.

Historical artifacts included:

```text
webgpt_ui_menu_fix.py
webgpt_ui_menu_fix_v2.py
webgpt_ui_menu_fix_v3.py
```

## Latest-only v4

Decision:

Freeze phase 1 on Latest/GPT-6 and stop clicking GPT-5.6/GPT-5.5 family rows.

Behavior:

- verify `Latest` checked;
- numeric readback had to indicate GPT-6 when available;
- expose only `webgpt/gpt-6`;
- preserve five local effort values.

Historical artifact:

```text
webgpt_latest_only_fix.py
```

## Draft/readback v5

Problem:

Pre-send prompt verification could disagree with the visible editor because semantic whitespace/readback normalization was too strict.

Result:

Logical editor text comparison was separated from superficial formatting differences.

Historical artifacts:

```text
webgpt_presend_check.py
webgpt_draft_readback_fix.py
```

## Codex 0.150 compatibility v6

Problem:

Codex sent `client_metadata`, which the provider's strict request allowlist rejected as an unsupported semantic option.

Result:

- bounded string-only `client_metadata` accepted;
- metadata validated but not exposed to the browser model prompt;
- malformed metadata still failed closed.

Historical artifact:

```text
webgpt_codex0150_compat_fix.py
```

## Hosted-tool experiments v7-v8

### v7 — ineffective

Attempt:

Remove `web_search_tool_type` from the catalog.

Why it failed:

Codex defaulted the missing metadata, so hosted web search was still emitted.

### v8 — effective policy seam

Fix:

Use the OpenCodex hosted-tool policy path to strip `web_search` / `web_search_preview` for the exact WebGPT destination while preserving client-executed tools.

Result:

The previous `unsupported_tool` failure disappeared and requests reached browser automation.

Historical artifacts:

```text
webgpt_hosted_tool_fix.py
webgpt_hosted_tool_fix_v8_fixed.py
```

## Navigation readiness v9

Problem:

Browser selection sometimes ran before the ChatGPT composer/menu had stabilized.

Fix:

Wait for:

- same-origin ChatGPT page;
- `document.readyState == complete`;
- one visible editable composer;
- short stability interval;
- then resolve the model/effort controls.

Historical artifact:

```text
webgpt_navigation_readiness_fix_v9.py
```

## Dynamic effort range v10 / v10.1

Raw DOM evidence during Pro unavailability:

```text
aria-valuemin=0
aria-valuemax=3
aria-valuenow=3
label=매우 높음
Pro disabled
```

Problem:

Provider hard-required `max=4`, so an already-correct XHigh state was rejected.

Fix:

- accept active slider ranges `0..3` or `0..4`;
- XHigh stays index 3;
- Max/Pro remains index 4 and is unavailable when the live max is 3;
- no silent fallback.

`v10` helper itself had an appended-marker newline bug; `v10.1` corrected the helper and successfully applied the source change.

Historical artifacts:

```text
webgpt_dynamic_effort_range_fix_v10.py
webgpt_dynamic_effort_range_fix_v10_1.py
```

## Direct tool surface v11

Problem:

Codex code-mode/deferred-tool behavior hid v1 collaboration tools behind wrappers.

Attempt:

Broadly switch the WebGPT model away from code-mode-only and disable search-tool metadata so tools were directly visible.

Result:

It exposed too much of the tool surface and contributed to context pressure. This was not the right long-term shape.

Historical artifacts:

```text
webgpt_direct_tool_surface_fix_v11.py
webgpt_direct_tool_runtime_check_v11.py
```

## Request-data framing v12

Problem:

A child request such as `return exactly WEBGPT_SUBAGENT_OK` could be interpreted as an instruction to emit raw text rather than preserve the provider's outer request-bound envelope.

Fix:

Re-state the wire-format requirement after `REQUEST_DATA_JSON`, clarifying that `reply exactly X` constrains the semantic final text, not the transport envelope.

Historical artifact:

```text
webgpt_request_data_framing_fix_v12.py
```

## Targeted v1 surface v13

Correction to v11:

Restore compact code-mode behavior and directly expose only the v1 collaboration namespace during focused Codex testing.

The correct Codex override was:

```text
features.code_mode.direct_only_tool_namespaces=["multi_agent_v1"]
```

This led to the first fully successful real `webgpt/gpt-6 xhigh` v1 subagent run.

Historical artifact:

```text
webgpt_targeted_v1_surface_fix_v13.py
```

## Browser queue v14

Problem:

Parent and child could overlap and one request received:

```text
409 Conflict: One WebGPT request already owns the browser.
```

Fix:

Use the existing in-process async lock as a bounded queue while preserving one active browser owner.

Offline verification eventually passed after correcting a verifier-only macOS `/var` symlink issue:

```text
normal_serialization = true
uncertain_recheck_fail_closed = true
```

Historical artifacts:

```text
webgpt_serial_browser_queue_fix_v14.py
webgpt_serial_browser_queue_verify_v14_1.py
webgpt_serial_browser_queue_verify_v14_2.py
```

## Strict JSON prompt hardening v15 / v15.1

Problem:

Arbitrary parent tool use produced an object-looking reply that contained `request_id`, `type`, and `calls`, but strict JSON parsing failed at a quote boundary in a nested argument payload.

Attempt:

Strengthen the post-request wire reminder so the model explicitly serializes nested tool arguments as strict JSON.

`v15` helper had an anchor mismatch; `v15.1` corrected the helper and successfully applied the prompt-only hardening.

Result:

The same class of parent tool failure still occurred. This was the point at which the user concluded the project had become prompt engineering rather than the desired robust integration.

Historical artifacts:

```text
webgpt_strict_json_escaping_fix_v15.py
webgpt_strict_json_escaping_fix_v15_1.py
```

## Final conclusion from the patch history

The sequence demonstrates a useful distinction:

```text
UI compatibility issues           -> fixable with deterministic selectors/readback
Codex metadata/tool exposure      -> fixable with deterministic adapter/catalog policy
browser overlap                   -> fixable with deterministic queueing
arbitrary model-authored tool JSON -> structural weakness of this transport design
```

Do not replay the patch series as a migration strategy. Use it as a list of requirements and failure modes when evaluating an upstream replacement.
