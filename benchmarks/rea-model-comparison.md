# REA agent benchmark — initial controlled test

**Date:** 3 November 2026  
**Status:** Four observed successful baseline sessions; no controlled deep-reasoning winner yet.

## Environment and task

- Nobara Linux, local LM Studio endpoint `http://127.0.0.1:1234/v1`
- Qwen Code 0.26.6, REA 6.8.0, Ghidra 12.1.4
- Context configured as **65,536** tokens for each run
- One `ToolSearch` for `mcp__rea__open_binary` and `mcp__rea__analyze_function`
- Open explicit `ftWbioEngineAdapter.dll` path; verify SHA-256
- Analyse only address `0x18012a350` with `detail: "summary"`, report and stop

**Binary SHA-256:** `a4f037c260e8c86718675ddb6540467a9006ab26184858eb5b8726137185d57c`

## Results from conversation exports

| Model ID | Session start–export elapsed | Tools and hash | Function summary | Reported cumulative tokens |
| --- | ---: | --- | --- | ---: |
| `google/gemma-4-31b-qat` | 6m 51s | Passed | Passed | 109,666 |
| `qwen/qwen3-coder-next` | 2m 32s | Passed | Passed | 107,650 |
| `meta/muse-glimmer` | 5m 16s | Passed | Passed | 107,083 |
| `recursivesi/flint` | 2m 48s | Passed | Passed | 106,129 |

Elapsed values are from the export header timestamps; they are **not pure inference times**. Token figures are **reported cumulative session counters** and should not be interpreted as model-generated tokens or directly comparable throughput.

All four reached the correct digest and same recovered function summary. Counts included 8,841 pseudocode code units, 289 assembly instructions and 49 basic blocks.

All four returned a summary-view evidence ID `ev_4aa359d5cc8094f78cfab2f1aeb3644f60b97a97527f17ff63698722a5a07034` with original parent `ev_217b5d98255fa4f30571263074a5ef74aceb6ce718f9b89ecd55b6bc25d60096`. The repeatable evidence IDs show consistent summaries **not necessarily independent recomputation**.

## Interpretation

- The earlier large/ambiguous prompts triggered repeated tool searches, confusion over previous sessions, and excessive output. Simplifying the task dramatically improved observed reliability for all four candidates.
- Qwen completed this particular tool-orchestration baseline fastest; Flint was close. Neither advantage establishes superiority in complex static analysis.
- Prior Gemma runs also encountered repeated truncation and an empty-response error. The simple successful test does not prove those failures are eliminated.
- A useful next benchmark must score **evidence fidelity**, *not merely successful tool calls*: correct matching-interface offset, storage-versus-algorithm distinction, input/output parameter uncertainties, correct parent evidence use and absence of recovery loops.

## Suggested scoring for a second-stage test

| Category | Maximum |
| --- | ---: |
| Correct and bounded tool calls | 20 |
| Original-evidence provenance tracking | 20 |
| Technical interpretation | 30 |
| Assembly cross-check and justified claims | 20 |
| Efficiency | 10 |

The next-stage results are **pending verification** and should not be invented or inferred from the first test. Retain redacted timestamps, full model IDs and exact prompts for new runs.

## Reproducibility warning

Use the same quantisation, server settings, sampling parameters, session state and exact prompt whenever practical. Keep model architecture/marketing claims clearly separate from locally observed benchmark facts.
