# Artifact Index

This index records the notable local scripts/artifacts created during the experiment. They are listed for traceability only; they were **not** copied into this archive repository because the project was stopped and the next direction is upstream adoption rather than replaying the patch chain.

## Provider package / baseline artifacts

```text
opencodex-webgpt-provider-0.1.0.zip
webgpt-provider/README.md
webgpt-provider/BUILD_REPORT.md
opencodex_webgpt_provider-0.1.0-py3-none-any.whl
opencodex-webgpt-provider-0.1.0.SHA256SUMS
```

## Browser/login/UI artifacts

```text
webgpt_macos_login_fix.py
webgpt_ui_menu_fix.py
webgpt_ui_menu_fix_v2.py
webgpt_ui_menu_fix_v3.py
webgpt_latest_only_fix.py
webgpt_presend_check.py
webgpt_draft_readback_fix.py
webgpt_navigation_readiness_fix_v9.py
webgpt_effort_raw_dom_inspect_v3.py
webgpt_dynamic_effort_range_fix_v10.py
webgpt_dynamic_effort_range_fix_v10_1.py
```

## Codex/OpenCodex compatibility artifacts

```text
webgpt_opencodex_preflight.py
webgpt_opencodex_256_repair_v3.py
webgpt_opencodex_private_network_fix.py
webgpt_opencodex_runtime_check.py
webgpt_codex0150_compat_fix.py
webgpt_hosted_tool_fix.py
webgpt_hosted_tool_fix_v8_fixed.py
webgpt_direct_tool_surface_fix_v11.py
webgpt_direct_tool_runtime_check_v11.py
webgpt_targeted_v1_surface_fix_v13.py
```

## Tool/subagent diagnostics

```text
webgpt_tool_smoke.py
webgpt_namespace_tool_smoke.py
webgpt_protocol_structure_serve.py
webgpt_subagent_protocol_diag_v2.py
webgpt_subagent_request_trace_v3.py
webgpt_subagent_json_diag_v4.py
webgpt_malformed_json_diag_v5.py
```

## Framing / queue experiments

```text
webgpt_request_data_framing_fix_v12.py
webgpt_serial_browser_queue_fix_v14.py
webgpt_serial_browser_queue_verify_v14_1.py
webgpt_serial_browser_queue_verify_v14_2.py
webgpt_strict_json_escaping_fix_v15.py
webgpt_strict_json_escaping_fix_v15_1.py
```

## UI screenshots referenced during debugging

```text
29fb41dc-edc6-4b7a-a6f2-b083de056fc2.png
23d14ef4-c293-493e-8f08-ee7860926739.png
```

These captured the ChatGPT Web model/reasoning menu during the experiment and were used only as visual debugging evidence.

## Important artifact-status notes

### Retired / do not rerun blindly

The early OpenCodex setup helper was retired after it could remove a corrected `allowPrivateNetwork:true` setting:

```text
webgpt_opencodex_256_setup.py
```

Do not use an old helper merely because it is listed in historical logs.

### Helper bugs encountered

Several patch helpers themselves needed correction:

- v10 had a literal newline/marker generation bug; v10.1 superseded it.
- v15 had a source-anchor mismatch; v15.1 superseded it.
- v14's first verifier used macOS's default `/var/folders` temp path and triggered the provider's symlink-safety check; v14.2 corrected the verifier path without changing the queue implementation.

These incidents are another reason not to treat the patch series as a reusable installation bundle.

## Final local-source note

At the time the project was stopped, the user's local provider checkout had accumulated multiple experimental source patches. This repository intentionally records the **behavioral state and evidence**, not a claim that those local source files constitute a clean distributable release.

If the old checkout must ever be revisited, compare it against the original package/baseline first and treat every local source difference as experimental until independently revalidated.
