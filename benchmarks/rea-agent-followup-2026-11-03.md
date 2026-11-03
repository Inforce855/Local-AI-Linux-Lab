# REA agent follow-up benchmarks — 3 November 2026

**Status:** Observed local tests, exploratory; not an agent/model ranking.  
**Environment:** Nobara Linux; LM Studio OpenAI-compatible local endpoint; Qwen Code 0.26.6; REA 6.8.0; Ghidra 12.1.4; configured model context 65,536 tokens.  
**Models:** `recursivesi/flint` and `qwen/qwen3-coder-next`.  
**Source:** User-supplied Qwen Code Markdown session exports and corresponding debug logs, inspected on 3 November 2026. Private complete logs and personal absolute paths are intentionally not committed.

## 1. Focused indirect-call investigation (earlier follow-up)

**Task:** Analyse the indirect call at `0x18012a676` within `ftWbioEngineAdapter.dll` function `0x18012a350`; use REA assembly and pseudocode evidence to explain the vtable offset and arguments. Both models were given the same focused task. Binary digest was verified as `a4f037c260e8c86718675ddb6540467a9006ab26184858eb5b8726137185d57c`.

| Model | Session start–export elapsed | Exported cumulative tokens | Observed outcome |
| --- | ---: | ---: | --- |
| Qwen3-Coder-Next | 6m 04s | 246,338 | Produced a report, but analysed a *different* call, `0x18012a482`, rather than the target `0x18012a676` |
| Flint | 15m 47s | 194,773 | Retrieved relevant tool evidence but failed to finish a report after Qwen Code context-compression failure |

**Observed:** Flint's host reported `COMPRESSION_FAILED_EMPTY_SUMMARY` after an estimated 42,802-token prompt exceeded its 42,536-token hard safety limit. This is a host/context-compaction failure, not evidence that REA failed. Qwen's completed answer was technically off-target. Both models incurred large cumulative token counts for a narrowly scoped task.

**Interpretation (inferred):** Qwen tended to launch tool work promptly but lost the requested target; Flint did more preliminary interpretation and was vulnerable to harness/context overhead. These are behavioural observations in these runs, not general model capability claims.

## 2. Positive versus negative wording: narrow hash task

Each of the four tests used a fresh Qwen Code session (one user prompt). The same binary was opened, and all returned the correct SHA-256 above. Two prompts asked for the same hash and differed in positive versus restrictive wording.

- **Positive:** Use REA's `open_binary` to open the DLL, report its SHA-256 hash, and stop.
- **Negative:** Do not use other tools to open the DLL, analyse functions or search files; report only its SHA-256 hash.

| Model | Positive: cumulative tokens | Negative: cumulative tokens | Observed difference |
| --- | ---: | ---: | --- |
| Flint | 173,203 | 75,623 | Positive: initially attempted a deferred tool without reviewed schema, then searched, opened and unnecessarily closed the binary. Negative: searched first, opened the binary, reported only hash and stopped |
| Qwen3-Coder-Next | 178,221 | 210,775 | Positive: initial schema-review refusal, recovered, then unnecessarily closed binary and answered in Chinese. Negative: tried unregistered tool name, requested unavailable `hopper` provider, recovered, unnecessarily closed binary and answered in English |

**Observed:** All four correct SHA-256 outputs. Flint's negative run was notably narrower. Qwen's negative run spent more cumulative tokens and included an avoidable unavailable-provider attempt. These counts are *cumulative session counters*, not instantaneous context usage, generated output size or precise measures of inference throughput.

**Limitations:** Single run per model/wording pair, varied tool errors and host approval behaviour; cannot attribute the quantitative differences solely to positive/negative phrasing. The REA `open_binary` tool documentation recommends `close_binary` on completion, which can conflict with a prompt that asks to report and stop. The observed cleanup calls should not automatically be labelled a REA defect.

## 3. Debug-log findings

**Observed:**
- Each run started as a distinct fresh Qwen Code session; no reused conversational context was demonstrated.
- Initial estimated active context was roughly 15.2K tokens, from the harness/system instructions/tools before useful analysis.
- The four hash tests did **not** trigger automatic compression; active estimated context stayed well below the 65,536 configured window, despite much larger *cumulative* session-token counters.
- Each startup loaded 17 bundled skills, one user skill and 37 built-in Qwen Code tools.
- Qwen Code loaded an 841-character user language instruction file with preference `auto`, advising responses in the user's language. Its English user prompt was present in the export, but the positive run's final reply was in Chinese, contrary to that preference.
- A follow-up suggestion in the positive Qwen run was also generated in Chinese; this does **not** establish that the typed English prompt was converted in the terminal before submission.
- Qwen Code's positive run included a tool-approval/classifier wait/abort of approximately 31.7 seconds, so elapsed session time must not be interpreted as pure model inference time.
- Logs displayed `[TOOL_USE_SUMMARY] No fast model configured — skipping summary generation`; this is a configuration observation, **not** proof that it caused high token use.
- Qwen Code's tool bridge refused calls when a deferred tool schema had not been reviewed. A separate negative run selected `hopper`, which was unavailable, before succeeding with automatic provider selection.

**Inference:** Agent orchestration, tool-schema handling, approval workflow, instruction interpretation and stop conditions can materially affect local coding-agent efficiency even when the underlying REA operation succeeds. No new changes to the working MCP progress-notification patch or REA service were indicated by these tests.

## 4. Clipboard and language notes

**Observed:** Both models saw Qwen Code's system/harness text indicating that marked blocks had been pasted from the system clipboard and might not have been displayed fully in the editor. Flint spent visible pre-tool deliberation interpreting that instruction. Its references to the clipboard are therefore not evidence of independent detection of Ctrl+V.

**Observed:** The language instruction file specified responding in the language of the user's input. Qwen Code's recorded positive user prompt was in English while the final response was Chinese. The root cause of the language switch and the user's observation of in-editor conversion remain **not verified**.

## 5. Operational implications / next benchmark

1. Preserve the verified REA tool connection and existing MCP progress-notification fix. Do not reinstall or change providers based solely on this comparison.
2. Prefer concise, bounded tasks and explicit success/stop conditions as a *testable hypothesis*, not an established universal optimisation.
3. Score target fidelity and correct ABI/assembly interpretation, not just whether tools were called and a report was produced.
4. For follow-up comparisons, control the prompt, model revision/quantisation, decoding settings, context, host permissions and tool state; record model-output versus host/tool waiting separately.
5. Avoid publishing raw session/debug logs, local absolute paths, proprietary binary contents or biometric material.

Related previous record: [Initial four-model baseline](rea-model-comparison.md). Existing project status: [PROJECT_STATUS.md](../PROJECT_STATUS.md).
