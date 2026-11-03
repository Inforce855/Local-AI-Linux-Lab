# LM Studio + Qwen Code on Linux

> **Status:** Observed local integration, not yet an independently reproduced fresh-install walkthrough. Checked against Qwen Code documentation on 3 November 2026.

## Tested environment

- Linux: Nobara (Fedora-based), KDE desktop
- Inference: LM Studio, OpenAI-compatible API at `http://127.0.0.1:1234/v1`
- Coding agent: Qwen Code 0.26.6, with locally configured `openai` model provider
- Models tested: `qwen/qwen3-coder-next`, `recursivesi/flint`, `google/gemma-4-31b-qat`, `meta/muse-glimmer`
- Tools: REA MCP server with Ghidra 12.1.4

These are **observed components**, not an exhaustive package or installation manifest.

## General connection workflow

1. Install and launch compatible LM Studio and Qwen Code builds using their current upstream instructions.
2. Start LM Studio's local OpenAI-compatible server and load the intended model.
3. In Qwen Code settings, define the model under `modelProviders.openai`, with an appropriate local `baseUrl`, and configure any needed key through the method supported by your Qwen Code version.
4. Select the same model in Qwen Code and confirm the selected model ID and API endpoint in its startup output.
5. Run a harmless single-tool test before starting a long MCP investigation.

Primary source: [Qwen Code model-provider documentation](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/model-providers/).

## Why context settings matter

LM Studio's loaded-model context and Qwen Code's *assumed* context are distinct controls. In a local test, LM Studio was set to **65,536** while the Qwen Code provider entry explicitly contained:

```json
"generationConfig": {
  "contextWindowSize": 32768,
  "timeout": 300000,
  "streamIdleTimeoutMs": 300000,
  "maxRetries": 1,
  "samplingParams": {
    "max_tokens": 4096
  }
}
```

The configured `32768` was changed to `65536` to match the local test. This was a **configuration mismatch**, not proof that it alone caused an empty-response error. `samplingParams.max_tokens` is a generation-output setting, not the total conversation context length.

For entries under `modelProviders`, put `contextWindowSize` under **that model's own** `generationConfig`; the top-level `model.generationConfig` does not fill model-provider settings. Qwen Code documents using its model-specific defaults and a fallback when context is not explicitly configured; don't assume the local server's loaded context is automatically discovered.

Source: [Qwen Code settings reference](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/).

## Symptoms to distinguish

| Symptom | First thing to verify |
| --- | --- |
| 'Context near hard limit' unexpectedly early | Both LM Studio loaded-model context and Qwen Code provider `contextWindowSize` |
| 'Response truncated due to token limits' | Generated response length and configured output tokens; inspect logs |
| 'Model stream ended with empty response text' | Actual LM Studio response / Qwen Code debug log; do not immediately blame MCP |
| Tool call refused: schema not reviewed | Use `ToolSearch` to review the exact deferred tool schema |
| REA reports no app open | Open the target binary in the **current** REA session; don't assume another session is shared |
| REA progress-token error and repeated MCP reconnection | See [MCP progress-notification investigation](../fixes/mcp-progress-notification-race.md) |

## Controlled testing approach

Give every model the **same complete prompt**, explicit binary path, expected hash, permitted tools, context budget and stop condition. Use a new agent conversation for each model. The task should not refer to a 'previous test' when the new model cannot access it. See [benchmark methodology](../benchmarks/rea-model-comparison.md).

## Limitations

The local tests did **not** establish an automatic context synchronisation feature in Qwen Code. Do not publish access keys, settings files with secrets or unredacted debug logs.
