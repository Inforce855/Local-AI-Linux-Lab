# Project status

**Last updated:** 4 November 2026  
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
- Follow-up REA tests on 3 November 2026 are documented in [`benchmarks/rea-agent-followup-2026-11-03.md`](benchmarks/rea-agent-followup-2026-11-03.md): focused indirect-call failures, four fresh Flint/Qwen positive-versus-negative prompt runs, verified hashes, cumulative token counts and debug-log observations. No changes to the verified REA/MCP patch were required.

- **Krita MCP (4 November):** isolated Krita 6.0.4 AppImage with MCP Bridge 1.7.0 and native helper passed **10/10** local self-tests. Documented the build from checkout `443e22c`, Qt 6.11-to-6.8 compatibility workaround, temporary RUNPATH removal, isolated XDG installation, and local version-prefix correction in [`guides/krita-mcp-appimage-nobara.md`](guides/krita-mcp-appimage-nobara.md). Existing Krita 5.3.4.1 Flatpak was preserved. This does not establish general ABI safety.

## REMAINING / UNKNOWN

- Krita MCP: test actual native warp/deformation on a disposable image, then connect Qwen Code through LM Studio; Qt ABI compatibility outside the passing self-test remains unverified. Preserve the uncommitted local `krita-mcp` version-check correction before updating that checkout.
- Determine whether upstream PR #3023 is merged, released, or included in any particular Qwen Code release before recommending removal of the local patch.
- Complete a *successful*, controlled multi-step REA benchmark comparing coding-agent reliability, evidence tracking, call-argument accuracy and total time. Earlier focused trials were performed and documented, but neither produced a correct, completed explanation of the target indirect call.
- Independently verify installation instructions on a clean machine before describing any end-to-end setup as universally reproducible.
- Continue distinguishing model/tool misuse from SDK/transport failures using actual debug logs; the 3 November logs separate tool-schema refusals, a provider-selection error and context-compression failure from successful REA operations.
- Determine why one English Qwen prompt produced a Chinese final response and follow-up suggestion; no cause established, and no justification for changing the existing language preference file.
- Add NVIDIA-specific guidance only after someone verifies it on NVIDIA hardware.

## NEXT ACTION

1. For Krita MCP, keep the verified 10/10 build configuration and local source fix intact; next verify a small warp on a disposable image before the Qwen Code connection.
2. Preserve confirmed details and links to primary sources in this repository.
3. Preserve the 3 November follow-up results and record future benchmarks with model build, quantisation, context size, prompts and tool traces; control host/tool state and avoid treating cumulative token totals as context usage.
4. Verify upstream MCP SDK resolution and publish a version-qualified workaround or migration note.

## Operating rules

- `main` is the canonical documentation branch; don't create branches or overwrite existing work without a clear reason.
- Label statements **Observed**, **Inferred**, **Not verified** or **Upstream documented**.
- Never commit fingerprints, biometric templates, secrets, personal paths, private logs, or locally patched vendor/third-party binaries.
- Do not assume `PROJECT_STATUS.md` describes a user's currently installed packages: check actual versions before giving a destructive instruction.
