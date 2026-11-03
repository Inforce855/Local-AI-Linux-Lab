# Project status

**Last updated:** 3 November 2026  
**Repository:** `Inforce855/Local-AI-Linux-Lab`  
**Scope:** Public, reproducible notes for local AI tools on Linux.

## COMPLETED

- Repository created on `main`.
- Local test environment documented: Nobara Linux, Qwen Code 0.26.6, LM Studio, REA 6.8.0, Ghidra 12.1.4.
- An REA/Qwen Code MCP disconnection was traced to an **unknown progress token** error followed by MCP health-check failures and a server restart.
- A local modification based on upstream TypeScript MCP SDK [PR #3023](https://github.com/modelcontextprotocol/typescript-sdk/pull/3023) was applied to two bundled JavaScript chunks; both passed `node --check`. A subsequent controlled REA session-persistence test successfully retained and revisited function evidence.
- A verified, read-only test against `ftWbioEngineAdapter.dll` reached function `0x18012a350`; later four-model tests opened the same SHA-256-verified binary and returned the same function summary.
- A Qwen Code model-provider context-window mismatch (32,768 vs 65,536) was identified from the local settings.
- The first set of public guides and troubleshooting records was prepared from observed results. Documentation is **not** an assertion that every combination works.

## REMAINING / UNKNOWN

- Determine whether upstream PR #3023 is merged, released, or included in any particular Qwen Code release before recommending removal of the local patch.
- Perform a controlled multi-step REA benchmark comparing coding-agent reliability, evidence tracking, call-argument accuracy and total time.
- Independently verify installation instructions on a clean machine before describing any end-to-end setup as universally reproducible.
- Distinguish model/tool misuse from SDK/transport failures using actual debug logs.
- Add NVIDIA-specific guidance only after someone verifies it on NVIDIA hardware.

## NEXT ACTION

1. Preserve confirmed details and links to primary sources in this repository.
2. Record new benchmarks with model build, quantisation, context size, prompts and tool traces.
3. Verify upstream MCP SDK resolution and publish a version-qualified workaround or migration note.

## Operating rules

- `main` is the canonical documentation branch; don't create branches or overwrite existing work without a clear reason.
- Label statements **Observed**, **Inferred**, **Not verified** or **Upstream documented**.
- Never commit fingerprints, biometric templates, secrets, personal paths, private logs, or locally patched vendor/third-party binaries.
- Do not assume `PROJECT_STATUS.md` describes a user's currently installed packages: check actual versions before giving a destructive instruction.
