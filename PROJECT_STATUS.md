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

- **ChatGPT Exporter (Continuity local.5, 4 November):** the Tampermonkey Export button previously appeared briefly on reload and disappeared. A narrow UI anchor/visibility fallback restored the button after ChatGPT initialisation; the user confirmed it opened and successfully exported Markdown. A short flicker remains. Other formats were not re-tested. See [`fixes/chatgpt-exporter-disappearing-button.md`](fixes/chatgpt-exporter-disappearing-button.md).

- **Bionic + Krita MCP (4 November):** existing isolated Krita 6 bridge connected to LM Studio Bionic with **57 tools ready**, without server rebuild. Bionic/Gemma E4B drew visible magenta and green rectangles on new layers. Early tests erroneously used `opacity=1` (nearly transparent) and `index=1` (below Background) and falsely claimed full success; one 29-second test using `opacity=255` and `index=0` was **visually verified** successful. Qwen Code E4B also completed a 178-ms nonmutating `warp_layer` preview, but its intended *visual effect* was not confirmed. See [expanded Krita guide](guides/krita-mcp-appimage-nobara.md#9-bionic--gemma-e4b-and-real-image-editing-tests-4-november-2026). Preserve working server and uncommitted local changes.\n\n## REMAINING / UNKNOWN

- Krita MCP: Qwen Code and Bionic connections are established; next visually verify a useful Tiger deformation (the 2-pixel preview was executed but the effect was not established). Consider a small Bionic Krita skill informed by upstream docs, and only then consider improving ambiguous opacity/index descriptions in MCP schemas. Qt ABI compatibility outside tested operations remains unverified. Preserve the uncommitted local `krita-mcp` version-check correction.
- Determine whether upstream PR #3023 is merged, released, or included in any particular Qwen Code release before recommending removal of the local patch.
- Complete a *successful*, controlled multi-step REA benchmark comparing coding-agent reliability, evidence tracking, call-argument accuracy and total time. Earlier focused trials were performed and documented, but neither produced a correct, completed explanation of the target indirect call.
- Independently verify installation instructions on a clean machine before describing any end-to-end setup as universally reproducible.
- Continue distinguishing model/tool misuse from SDK/transport failures using actual debug logs; the 3 November logs separate tool-schema refusals, a provider-selection error and context-compression failure from successful REA operations.
- Determine why one English Qwen prompt produced a Chinese final response and follow-up suggestion; no cause established, and no justification for changing the existing language preference file.
- Add NVIDIA-specific guidance only after someone verifies it on NVIDIA hardware.

## NEXT ACTION

1. For Krita MCP, preserve the working bridge/source fix and both agent connections. Next visually compare a controlled, useful deformation on an expendable Tiger image; follow the upstream warp/artist examples and use explicit layer opacity/index conventions.
2. Preserve confirmed details and links to primary sources in this repository.
3. Preserve the 3 November follow-up results and record future benchmarks with model build, quantisation, context size, prompts and tool traces; control host/tool state and avoid treating cumulative token totals as context usage.
4. Verify upstream MCP SDK resolution and publish a version-qualified workaround or migration note.

## Operating rules

- `main` is the canonical documentation branch; don't create branches or overwrite existing work without a clear reason.
- Label statements **Observed**, **Inferred**, **Not verified** or **Upstream documented**.
- Never commit fingerprints, biometric templates, secrets, personal paths, private logs, or locally patched vendor/third-party binaries.
- Do not assume `PROJECT_STATUS.md` describes a user's currently installed packages: check actual versions before giving a destructive instruction.
