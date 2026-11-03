# Standalone REA baseline prompt

This is the *short*, tool-orchestration-only test used successfully by four local models on 3 November 2026. Replace the example DLL path and SHA-256 with a binary that you may legally inspect. **Do not reuse the host-specific sample path on another machine.**

## Prompt template

```text
Use REA with Ghidra to analyse one function in a native binary.
This is a standalone, read-only test. No previous session is required.

Binary path: <ABSOLUTE_PATH_TO_BINARY>
Expected SHA-256: <EXPECTED_SHA256>
Function address: <KNOWN_FUNCTION_ENTRY_ADDRESS>

Do these steps in order:
1. Use ToolSearch with this exact query:
   select:mcp__rea__open_binary,mcp__rea__analyze_function
2. Call mcp__rea__open_binary with the exact path above.
   Check the returned SHA-256. If different, report and stop.
3. Call mcp__rea__analyze_function with:
   procedure: "<KNOWN_FUNCTION_ENTRY_ADDRESS>"
   detail: "summary"
4. Report the returned name, address, summary status, original
   parent evidence ID and whether the operation succeeded.

Do not use any other REA tools, search the filesystem,
request complete output, modify files, or analyse other functions.
On any failure, report the error and stop.
After reporting, stop.
```

## Caveats

The response to `analyze_function` with `detail:"summary"` may itself be an `inspect_analysis_view` projection; the top-level `evidence_id` identifies that **view**, whereas `normalized_result.parent_evidence_id` identifies the original analysis. Do not conflate them.

For a genuinely new session, do not use 'the previously opened DLL' or 'the last evidence ID'. Explicit inputs make cross-model comparisons much fairer.

Read [model benchmark results](rea-model-comparison.md) for the observed outcomes, not performance promises.
