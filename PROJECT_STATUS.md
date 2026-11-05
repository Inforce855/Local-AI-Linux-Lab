# Project status

**Last updated:** 5 November 2026  
**Repository:** `Inforce855/Local-AI-Linux-Lab`  
**Scope:** Public, reproducible notes for local AI tools on Linux.

## 5 NOVEMBER 2026 — Bionic/REA long investigation and model comparison

**COMPLETED (observed):** A read-only ~44-minute Bionic/REA/Ghidra investigation of the FocalTech matching DLL identified candidate descriptor, RANSAC and verification procedures, retrieved call/pseudocode evidence, and executed a no-argument `close_binary` successfully. Two automatic Bionic context compactions were observed. Q8_0 and Q4_K_M activity overlapped; **the precise reason Q4 was selected is unknown**, so this is not a single-quantisation benchmark. A separate bounded test found Q8_0 disciplined, while Q4_K_M incorrectly decoded interface version 6.0 as 6.2. The earlier Gemma 31B QAT test overwrote the original DLL through a dangerous snapshot-path close; the owner restored and SHA-256-verified the original.

**REMAINING:** Independently validate the proposed matching semantics; determine Bionic helper/compaction model selection from direct request logs before making claims; preserve the verified REA bridge, Krita bridge and current local configuration.

**NEXT ACTION:** Retain the results and safety requirements in [the 5 November Bionic/REA research record](benchmarks/bionic-rea-moh-investigation-2026-11-05.md). No software or security-configuration changes are required by this documentation result.

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

- **Bionic + Krita MCP (4 November):** existing isolated Krita 6 bridge connected to LM Studio Bionic with **57 tools ready**, without server rebuild. Bionic/Gemma E4B drew visible magenta and green rectangles on new layers. Early tests erroneously used `opacity=1` (nearly transparent) and `index=1` (below Background) and falsely claimed full success; one 29-second test using `opacity=255` and `index=0` was **visually verified** successful. Qwen Code E4B also completed a 178-ms nonmutating `warp_layer` preview, but its intended *visual effect* was not confirmed. See [expanded Krita guide](guides/krita-mcp-appimage-nobara.md#9-bionic--gemma-e4b-and-real-image-editing-tests-4-november-2026). Preserve working server and uncommitted local changes.\n\n- **Tiger visual deformation (4 November):** **User visually confirmed** a Bionic/Gemma E4B `warp_layer` **apply** result by supplying before/after views from Krita. Using the same `Background` source as the successful preview: `center={x:1050,y:680}`, `offset={x:0,y:-30}`, `radius=160`, `mode="apply"`, new layer `TIGER - VISIBLE WARP`. The preceding preview reported **73,038 changed pixels**, 320×320 affected region (890,490), **471 ms**, and `mutated=false`; that preview alone was **not** visible on Krita's canvas. Apply created a visible change to Tiger's back/torso; adjacent sofa upholstery/stitching also visibly warped. **Original objective not yet complete:** isolate deformation to Tiger with a selection/mask, then establish believable subtle breathing and animation. Same MCP bridge unchanged. See the [Krita guide](guides/krita-mcp-appimage-nobara.md#10-first-user-confirmed-visible-tiger-warp-4-november-2026).

- **Bionic + REA (4 November):** Bionic connected to the existing REA MCP server (**229 tools ready**), and Gemma 4 E4B completed a read-only `analyze_function` summary with `procedure: "0x18012a350"`. User-reported results `FUN_18012a350`, 49 basic blocks and 659 value-flow operations match earlier records; 7 callees, 162 outgoing references and 673 def-use items were also reported. User observed **8.1 GB GPU RAM used** (snapshot, not a peak benchmark). Symbol/address confusion was resolved by supplying the address as a string. This verifies tool access and summary retrieval, **not** semantic correctness or an independently rechecked binary hash. See [REA workflow](guides/rea-ghidra-tested-workflow.md#bionic--gemma-e4b-reaghidra-test--4-november-2026).

## REMAINING / UNKNOWN

- Krita MCP: Bionic's large Tiger warp has now been visually verified, but it also deforms nearby sofa pixels. Next isolate Tiger with an appropriate selection/mask and determine a natural breathing motion. Consider a Bionic Krita skill grounded in upstream docs before editing agent-facing MCP schemas. Qt ABI compatibility outside tested operations remains unverified. Preserve the uncommitted local `krita-mcp` version-check correction.
- Determine whether upstream PR #3023 is merged, released, or included in any particular Qwen Code release before recommending removal of the local patch.
- Complete a *successful*, controlled multi-step REA benchmark comparing coding-agent reliability, evidence tracking, call-argument accuracy and total time. Bionic/E4B now completes a read-only function summary, but it has not yet produced a verified semantic explanation of the target indirect call.
- Independently verify installation instructions on a clean machine before describing any end-to-end setup as universally reproducible.
- Continue distinguishing model/tool misuse from SDK/transport failures using actual debug logs; the 3 November logs separate tool-schema refusals, a provider-selection error and context-compression failure from successful REA operations.
- Determine why one English Qwen prompt produced a Chinese final response and follow-up suggestion; no cause established, and no justification for changing the existing language preference file.
- Add NVIDIA-specific guidance only after someone verifies it on NVIDIA hardware.

## NEXT ACTION

1. For Krita MCP, preserve the working bridge/source fix and both agent connections. The first large warp is now visually confirmed; next constrain the deformation to Tiger and test smaller breathing motion, preserving the background. Follow upstream selection/warp examples.
2. Preserve confirmed details and links to primary sources in this repository.
3. Preserve the 3 November follow-up results and record future benchmarks with model build, quantisation, context size, prompts and tool traces; control host/tool state and avoid treating cumulative token totals as context usage.
4. Verify upstream MCP SDK resolution and publish a version-qualified workaround or migration note.

## Operating rules

- `main` is the canonical documentation branch; don't create branches or overwrite existing work without a clear reason.
- Label statements **Observed**, **Inferred**, **Not verified** or **Upstream documented**.
- Never commit fingerprints, biometric templates, secrets, personal paths, private logs, or locally patched vendor/third-party binaries.
- Do not assume `PROJECT_STATUS.md` describes a user's currently installed packages: check actual versions before giving a destructive instruction.
