# Troubleshooting: MCP failure or model tool-selection failure?

> Based on observed local REA, LM Studio and Qwen Code tests in November 2026.

## Start with the symptom, not a reinstall

| What the log shows | Likely category | Evidence needed / next action |
| --- | --- | --- |
| `Received a progress notification for an unknown token` followed by MCP health failures | Transport/SDK issue | Debug log, exact SDK build, restart timestamps; see [progress-notification case](../fixes/mcp-progress-notification-race.md) |
| `[tool_call bridge refused] ... no verified schema review` | Tool-discovery protocol | Run `ToolSearch` for the exact fully qualified tool name before calling it |
| `No app is open` in a new agent session | Target state unavailable | Provide an explicit absolute path and open once; not necessarily an error in MCP connectivity |
| `evidence_integrity_mismatch` | Evidence provenance/session issue | Check original `parent_evidence_id` and whether session state changed; stop automatic recovery |
| Repeated ToolSearch queries for the same tools | Model/tool-planning loop | Reduce to two known tools and one objective |
| `Response truncated due to token limits` | Output limit or oversized response | Prefer `detail: "summary"` and bounded native evidence views; inspect actual settings |
| `Context near hard limit` earlier than expected | Context configuration mismatch, or large history | Compare local model context with Qwen Code's configured `contextWindowSize` |
| `Model stream ended with empty response text` | Model/server/client response handling | Retain exact LM Studio and Qwen Code logs; cause not established by the message alone |

## Prompt design matters

A vague prompt said: 'If not open, use the same DLL as the previous test.' A new model session had **no knowledge** of the previous DLL path and wasted calls locating multiple candidates. A later prompt supplied the absolute path, target SHA-256, address, permitted tools and stop condition. Four different models then completed the same two-tool baseline.

A reliable initial prompt should:

1. State a standalone goal.
2. Supply the absolute file path (including spaces) and optional expected digest.
3. Require exactly `open_binary` and `analyze_function` first.
4. Use `detail: "summary"` and avoid broad inventory calls.
5. Define a stop condition and a concise requested report.
6. For a second-stage analysis, explain how the **parent evidence ID** relates to inspected views.

## The crucial distinction

A *working tool invocation* does not mean the model understood the binary. A *correct technical interpretation* requires evidence-linked addresses, arguments, and known unknowns. Equally, a model's wrong vtable offset is not automatically an MCP or Ghidra defect.

Our case study has two different offsets that were confused by one model: storage-record retrieval `+0xA0` and matching algorithm `+0x40`. Verify them from assembly and pseudocode rather than trusting an agent's summary.
