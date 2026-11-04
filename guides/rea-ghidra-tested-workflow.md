# REA + Ghidra: tested read-only workflow

> **Status:** Observed with REA 6.8.0, Ghidra 12.1.4, Qwen Code 0.26.6 and LM Studio on Nobara Linux. Not a general-purpose installer.

## Purpose

This guide documents a **working workflow** for selecting and inspecting a native function without flooding an agent's context or losing the provenance of its evidence.

The example used a Windows PE library associated with the FocalTech fingerprint matching engine. Do not share real biometric captures or templates when reproducing these steps.

## Tool order that worked

1. Use deferred-tool discovery with the *exact names*:

   `select:mcp__rea__open_binary,mcp__rea__analyze_function`

2. Call `mcp__rea__open_binary` with the **absolute path** of the selected native binary. The tool returned format, resolved path, architecture and SHA-256 in the local test.
3. Compare the returned SHA-256 with a known independently recorded digest.
4. Call `mcp__rea__analyze_function` with a known entry address and `detail: "summary"`.
5. For deeper inspection, first discover `mcp__rea__inspect_analysis_view`. The analysis summary identifies the **original** evidence through `normalized_result.parent_evidence_id`.
6. Page the **original parent evidence**, not an evidence ID returned by a derived view:

   ```json
   {
     "source": {
       "kind": "retained-evidence",
       "evidence_id": "<parent_evidence_id>"
     },
     "view": {
       "kind": "native",
       "facet": "pseudocode",
       "offset": 0,
       "limit": 4000
     }
   }
   ```

7. Read further bounded pages as needed. For native pseudocode, offsets and limits count **UTF-16 code units**; other native facets use rows.
8. Cross-check indirect calls with relevant assembly, and label inferred types separately from observed bytes and instructions.

### Why avoid `detail: "complete"` initially?

One earlier call produced approximately **552,180 characters** of JSON on a single line. Qwen Code truncated the visible output and wrote the full result to a temporary file. The smaller summary-plus-pages pattern was later shown to return the necessary identity and evidence provenance without that huge inline payload.

## Case-study evidence (sanitised)

- Binary: `ftWbioEngineAdapter.dll`
- SHA-256: `a4f037c260e8c86718675ddb6540467a9006ab26184858eb5b8726137185d57c`
- Target entry: `0x18012a350`
- Ghidra's recovered name: `FUN_18012a350`; diagnostic strings identify `EngineAdapterVerifyFeatureSet`
- Body range: `0x18012a350`–`0x18012a87e`, 1,327 bytes
- Summary: 8,841 UTF-16 pseudocode code units, 289 assembly instructions, 49 basic blocks, 659 value-flow operations

The recovered function used **different interfaces** for storage access and matching:

- The storage-interface method at offset `+0xA0` was associated with obtaining the current record.
- The matching-engine method at offset `+0x40` preceded a diagnostic log recording `algo->identify_image` duration.
- In that decompilation, `local_68._4_4_` was tested as positive in the match branch; this does **not** establish the exact public ABI or true C structure layout.

These details are *static analysis observations*. Ghidra's inferred variable names and types are not original source declarations.

## Evidence identity

The four-model baseline test returned a summary view associated with the same original parent evidence ID across models. That consistency is **not proof** each run independently recreated the entire underlying Ghidra database or that runtime matching was executed.

## Stop conditions

- Unexpected SHA-256: **stop**.
- Invalid/deferred tool schema: review the exact schema before retrying; don't call guessed tools.
- A REA error says no app is open: don't label it a transport disconnection without log evidence.
- An evidence ID fails integrity validation: stop and investigate its provenance rather than repeatedly opening and closing the target.

See [the MCP investigation](../fixes/mcp-progress-notification-race.md) for a different, genuine connection problem.

## Bionic + Gemma E4B REA/Ghidra test — 4 November 2026

**Observed, user-reported (not independently re-run by this document):**

- LM Studio Bionic connected immediately to the **existing** REA MCP server and reported **229 tools ready**. The pre-existing Qwen Code and Ghidra setups were preserved.
- The agent initially stopped because the DLL was not found in its searched workspace. After the user supplied the existing absolute path, it progressed to tool schema discovery.
- The `analyze_function` schema requires `procedure` as a **string**, not an `address` property. E4B initially assumed a symbol was mandatory and stopped. Correcting it to `{"procedure":"0x18012a350","detail":"summary"}` allowed the analysis to complete.
- The agent reported **`FUN_18012a350`**, **49 basic blocks**, **7 callees**, **162 outgoing references**, **659 value-flow operations**, and **673 def-use tracking items**. The name, basic-block count and value-flow count agree with earlier recorded REA findings; the remaining counts were reported in this run and have not been independently compared against retained raw evidence.
- User reported **8.1 GB GPU RAM in use** while using `google/gemma-4-e4b` (LM Studio Q6_K default, 7.21 GB). Treat this as a user-observed snapshot, **not** a controlled benchmark or peak-memory measurement.
- The user's quoted final message described structural statistics, **not a verified semantic explanation** of the function's algorithm. Binary SHA-256 verification and full tool outputs from this Bionic attempt were not included in the quoted final response; do not claim independently verified hash from that alone.

**Outcome:** Bionic agent connection, tool discovery, and one read-only function-summary request completed; no basis yet to claim a correct high-level reverse-engineering explanation, a controlled comparison against Qwen Code, or general REA reliability.

**Next test if useful:** With the already-open verified target, use one bounded, read-only REA evidence view to explain a specific call or branch and compare the explanation with the existing prior analysis. Record tool results, model and elapsed time without reopening/re-analysing unnecessarily.
