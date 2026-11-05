# Bionic + REA agent investigation — 5 November 2026

**Status:** Observed local benchmark and read-only DLL analysis. This is a research record, not a native-matcher implementation or a clean single-model performance comparison.

## Setup and evidence

- Nobara Linux, AMD Ryzen AI Max+ 395 / 128 GB unified RAM; LM Studio Bionic, REA 6.8.0, Ghidra 12.1.4.
- Investigated the original FocalTech `ftWbioEngineAdapter.dll`, SHA-256 `a4f037c260e8c86718675ddb6540467a9006ab26184858eb5b8726137185d57c`.
- Main hard-task prompt: read-only investigation of `FtVerifyTwoTemplate`, `FtComputeDescriptors`, `FtRansacNew`, verified call references, pseudocode, thresholds and outstanding native Linux requirements; **no snapshot/file changes**, close using `close_binary` with empty arguments; final report at most 600 words.
- Original user-run local Bionic transcript timestamps: **11:16:48 to approximately 12:01:05 GMT (about 44 min 17 sec)**. The exported history includes overlapping pre-compaction records, not independent full runs. Seven supplied exports included **three populated overlapping records and four empty sessions**.
- The completed session reported **37 tool calls**, no failed calls in its final report, and a successful `close_binary` invoked with `params_json:"{}"` / null tool return. Search-by-symbol returned no matches because symbols were stripped; this was not a tool error.
- **Observed:** Two Bionic context reductions (approximately 82K → 40.1K, and 71.6K → 23.4K in the UI); exported history includes Bionic `lmstudio/context-compaction-v1` state. Evidence IDs and function addresses survived sufficient to complete the task.
- **Observed in LM Studio UI:** Q8_0 and Q4_K_M instances were loaded; Q4_K_M processed prompts while the run continued. **Not verified:** which precise inference/model request performed each step, why Bionic selected Q4, or whether it was exclusively a compactor. Indexed session identifier `qwen/qwen3.8-27b` does not distinguish quantisation. **Do not label this 44-minute run a pure Q8_0 benchmark.**

## Agent behaviour and limitations

- The model found procedures through self-logging string searches and xrefs after direct name lookup was empty; it retrieved summary/callers/callees/pseudocode via REA retained evidence.
- After initially calling `FUN_18003cab0` a dispatcher, the report revised this to a score-producing matching-core candidate. **Label semantics as inferred**, because stripped symbols and unresolved indirect calls prevent complete identification.
- `WbioQueryEngineInterface` was incorrectly hypothesised during intermediate reasoning to accept a function name; earlier direct decompilation establishes that it returns an interface-structure pointer instead. This is a clear reasoning error, not a verified architecture fact.
- The agent reached a final report under a requested word limit and visibly reconsidered its draft length; this demonstrates instruction-following behaviour, **not** verified final word-count accuracy.
- No live fingerprint tests, repository-source edits or native matching implementation were performed.

## Distinct short benchmark observations (same day)

The preceding bounded tests on the `WbioQueryEngineInterface` function were **not** equivalent to the complex 44-minute run:

| Model/quantisation | Observed result | Caution |
| --- | --- | --- |
| Qwen3.8-27B Q8_0 | Correct interface analysis, no failed calls, safe closure; ~5m46s | Most reliable bounded test here |
| Qwen3.8-27B Q4_K_M | Completed safely, ~6m19s | Incorrectly interpreted interface version as 6.2 (verified version is **6.0**) |
| Qwen3-Coder-Next | Fast (~1m40s) but 5 invalid calls out of 12 | Unsafe snapshot-path attempt was rejected |
| Qwen3-Coder-30B | Fast (~1m20s) but 5 failed calls out of 11 | Inaccurate claim of zero errors |
| Gemma 4 E4B | Could not open target in this test | Misattributed tool-argument failures to REA |
| Gemma 4 31B QAT | ~7m18s; matching analysis partly useful | **Destructively replaced the original DLL** by using `close_binary(snapshot_path=<original>, overwrite=true)`; owner restored verified original hash from backup |
| RecursiveSI Flint | Explored extensively; failed to finish/close within test | Queried unrelated Krita MCP tools; possible poor fit to tightly constrained prompt, not established general incapacity |

These are individual local sessions under differing conditions, not statistically controlled rankings. Keep tool errors, content correctness, closure safety and quantisation separate.

## Reusable lessons / next action

1. Require `close_binary` **with no arguments**, no analysis snapshots, and ideally analyse a disposable copy before granting an agent destructive tool access. Preserve the original DLL.
2. Evaluate agent conclusions against decompiler evidence. Distinguish a tool's successful response from a correct interpretation.
3. Capture actual model instance, quantisation, compaction events, configured context, source hash and elapsed timestamps in subsequent tests. Do not infer compactor model selection from simultaneous GPU residency alone.
4. For a follow-up Flint experiment, allow a more exploratory discovery prompt **without relaxing read-only safety or completion requirements**; this is a hypothesis to test, not an approved new benchmark.

**Related implementation evidence:** [FocalTech 5 November matching-engine note](https://github.com/Inforce855/focaltech-5955-linux/blob/main/docs/moh-matching-engine-rea-2026-11-05.md).
