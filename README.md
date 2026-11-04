# Local-AI-Linux-Lab

Practical, evidence-led notes for running **local AI coding agents, inference servers, MCP tools and development workflows on Linux**.

Initial work is based on **Nobara Linux + AMD Strix Halo (Ryzen AI Max+ 395, 128 GB)**, **LM Studio**, **Qwen Code**, **REA** and **Ghidra**. NVIDIA and other platforms are welcome, but they are **not claimed to have been tested** until real results are contributed.

> **Status:** Started 3 November 2026. This is a growing collection of observations and reproducible tests, not a universal plug-and-play installation script.

## Start here

| Topic | Documentation |
| --- | --- |
| Current confirmed work and outstanding questions | [PROJECT_STATUS.md](PROJECT_STATUS.md) |
| LM Studio + Qwen Code model configuration | [LM Studio/Qwen Code guide](guides/lm-studio-qwen-code.md) |
| Krita 6.0.4 AppImage + MCP (verified 10/10 self-test) | [Krita MCP tested installation](guides/krita-mcp-appimage-nobara.md) |
| REA/Ghidra read-only analysis and evidence IDs | [REA/Ghidra tested workflow](guides/rea-ghidra-tested-workflow.md) |
| Confirmed MCP progress-token disconnect investigation | [Progress-notification race](fixes/mcp-progress-notification-race.md) |
| Distinguish SDK failures from model mistakes | [Troubleshooting decision guide](troubleshooting/agent-vs-mcp-failures.md) |
| Four-model basic REA comparison | [Benchmark results](benchmarks/rea-model-comparison.md) |
| Reuse the bounded standalone test | [Benchmark prompt template](benchmarks/standalone-rea-baseline-prompt.md) |
| Hardware/ROCm observations | [AMD Strix Halo notes](hardware/amd-strix-halo.md) |
| Add verified results | [CONTRIBUTING.md](CONTRIBUTING.md) |

## Why this exists

A locally hosted coding agent can fail for very different reasons: an MCP SDK race, a tool schema it never reviewed, context settings that differ from the inference server, an enormous tool response, or a model misunderstanding an API. Those failures can look similar if all you see is 'No app is open' or a stalled agent.

The first documented case followed an REA/Ghidra session that vanished after an MCP unknown-progress-token error. A narrow local mitigation based on [TypeScript SDK PR #3023](https://github.com/modelcontextprotocol/typescript-sdk/pull/3023) was tested; the subsequent controlled session test retained its function evidence. The workaround is **not** presented as a universally applicable upstream release.

Four local models were also run through a simplified two-tool REA baseline. All completed it; the results should not be mistaken for a comprehensive coding-model ranking.

## Evidence standard

Every guide aims to distinguish:

- **Observed:** something demonstrated by a log, tool output or local test
- **Inferred:** a plausible explanation supported by evidence, not directly proven
- **Upstream documented:** linked official documentation or project discussion
- **Not verified:** untested hardware, versions, assumptions or generalisations

No secrets, personal debugging dumps, biometric material or third-party binaries are distributed. Hardware and software may change; check the original sources and `PROJECT_STATUS.md` before applying changes.

## Primary upstream links

- [Qwen Code model providers](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/model-providers/)
- [Qwen Code settings](https://qwenlm.github.io/qwen-code-docs/en/users/configuration/settings/)
- [MCP TypeScript SDK PR #3023](https://github.com/modelcontextprotocol/typescript-sdk/pull/3023)
- [MCP SDK issue #3018](https://github.com/modelcontextprotocol/typescript-sdk/issues/3018)

## Licence and contributions

Contributions are welcome via GitHub. Before reuse or redistribution, check the licence and permissions applying to each contribution; no licence has been asserted for this repository in these initial notes. See [CONTRIBUTING.md](CONTRIBUTING.md).

**Maintainer:** [Inforce855](https://github.com/Inforce855).
