# Krita 6.0.4 AppImage + MCP native bridge on Nobara Linux

**Tested:** 4 November 2026  
**Evidence:** local terminal output and Krita MCP `--selftest` supplied by the tester  
**Upstream:** [thuanlm215/krita-mcp](https://github.com/thuanlm215/krita-mcp)  
**Checkout commit:** `443e22c` · **Bridge plugin:** 1.7.0 · **Reported operations:** 66  
**Result:** **10/10 self-tests passed** using an isolated Krita 6.0.4 AppImage environment.  
**Scope:** documented working local configuration, **not** a generally validated installer or Qt ABI guarantee.

## Goal and preservation rules

Make an existing LM Studio/Qwen Code MCP setup capable of controlling Krita for restrained image deformations (eventually a small sleeping-toy breathing animation). Keep the existing Krita **5.3.4.1 Flatpak** and its normal configuration untouched. Do not install the bridge into the ordinary user Krita resource directories during this experiment.

Only the bridge/self-test was confirmed here; **Qwen Code connection and actual warp animation were not yet tested**.

## Verified environment

| Component | Observed |
| --- | --- |
| OS | Nobara Linux, Fedora 44 base |
| Existing Krita | 5.3.4.1 Flatpak (preserved) |
| Experimental Krita | 6.0.4 AppImage, `git 57db7cd` |
| AppImage path on test machine | `~/Applications/Krita/krita-6.0.4-x86_64.AppImage` |
| MCP checkout | `~/Applications/Krita/krita-mcp` at `443e22c` |
| C++ toolchain | GCC 16.2.1, CMake |
| System development Qt | 6.11.2 |
| AppImage bundled Qt | Exported Qt symbol versions through `Qt_6.8` |
| ELF patching | `patchelf 0.18.0` |
| Bridge health | `Krita 6.0.4 (git 57db7cd), plugin 1.7.0, 66 operations` |

The paths below reflect this installation; adapt them for another system. **Never apply the temporary AppImage mount path literally:** it changes each time.

## 1. Check the already-installed applications

The Krita 6 AppImage was downloaded from the [official Krita downloads](https://krita.org/en/download/), placed in `~/Applications/Krita`, marked executable and launched. It reported:

```text
krita 6.0.4 (git 57db7cd)
```

The AppImage logged non-fatal `QFont::fromString` warnings; it still launched. The original Krita Flatpak was not replaced.

The MCP repository was cloned into the existing Krita application directory:

```bash
cd "$HOME/Applications/Krita"
git clone https://github.com/thuanlm215/krita-mcp.git
cd krita-mcp
git rev-parse --short HEAD
```

The observed checkout was `443e22c`. Re-check upstream changes before following these commands on a later commit.

## 2. Build dependencies and Krita headers

The system already had `cmake`, `g++`, `pkg-config`, and Qt6 Core/Gui/Widgets/Xml development files. The following were installed during this work:

```bash
sudo dnf install kf6-ki18n-devel boost-devel eigen3-devel kf6-kconfig-devel patchelf
```

DNF also pulled in `libicu-devel`. The `kf6-kconfig-devel` package supplies `/usr/include/KF6/KConfigCore/kconfiggroup.h`. The first compilation failed on that header, because KConfig development files had not yet been installed; reconfiguring CMake after installation refreshed the include paths and resolved the failure.

The repository's own pinned downloader cached **141 Krita 6.0.4 header dependencies**:

```bash
cd "$HOME/Applications/Krita/krita-mcp"
python3 native/fetch_headers.py
```

The script fetched the `v6.0.4` tree/headers from the KDE Krita GitHub source. Its tag was **not independently checked** against the AppImage's `57db7cd` commit.

## 3. Configure with the AppImage's Krita libraries

An AppImage contains runtime libraries but not the matching Krita development headers. The four required libraries existed in its mounted `usr/lib`: `libkritacommand`, `libkritaui`, `libkritalibkis`, `libkritaimage`.

In a separate terminal, mount the AppImage and **leave the mount command running**:

```bash
"$HOME/Applications/Krita/krita-6.0.4-x86_64.AppImage" --appimage-mount
```

It prints a path of the form `/tmp/.mount_krita-XXXXXX`. Use **that actual path** as `MOUNT` in a second terminal (not an old mount path):

```bash
cd "$HOME/Applications/Krita/krita-mcp"
MOUNT="/tmp/.mount_krita-XXXXXX"  # Replace with actual current mount
LIB="$MOUNT/usr/lib"
test -f "$LIB/libkritaui.so.21.0.0" || { echo "Wrong/unmounted AppImage path"; exit 1; }

cmake -S native -B native/build \
  -DCMAKE_BUILD_TYPE=Release \
  -DKRITA_COMMAND="$LIB/libkritacommand.so.21.0.0" \
  -DKRITA_UI="$LIB/libkritaui.so.21.0.0" \
  -DKRITA_LIBKIS="$LIB/libkritalibkis.so.21.0.0" \
  -DKRITA_IMAGE="$LIB/libkritaimage.so.21.0.0"
```

CMake warned that bundled Qt/KF6 libraries might shadow system libraries. This warning is real: the first compiled helper required `qt_version_tag@Qt_6.11`, but Krita's bundled Qt exported versions only through `Qt_6.8`.

## 4. The narrowly scoped Qt compatibility experiment

**Observed:** the only `Qt_6.11` requirement reported in the helper was `qt_version_tag`. Rebuilding with Qt version tagging disabled removed it. This **bypasses a deliberate Qt compatibility guard** and is not proof of full Qt 6.11-header/6.8-runtime ABI compatibility. Do not generalise this workaround to other binaries.

```bash
cd "$HOME/Applications/Krita/krita-mcp"
cmake -S native -B native/build \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_CXX_FLAGS="-DQT_NO_VERSION_TAGGING"
cmake --build native/build -j2
objdump -T native/build/libkrita_mcp_native.so | grep 'Qt_6\.11' || echo "No Qt 6.11 symbols found"
```

Observed build: `[100%] Built target krita_mcp_native`. The Qt 6.11 symbol check then reported **no Qt 6.11 symbols**.

While the original AppImage mount was still present, `ldd -r` reported **no unresolved runtime dependencies**. This check is not equivalent to loading the library inside Krita.

## 5. Remove the temporary mount RUNPATH and prepare manifest

The compiled library initially embedded a changing directory:

```text
RUNPATH: /tmp/.mount_krita-EeiKME/usr/lib
```

It was patched with `patchelf --remove-rpath`, then the matching SHA256-named helper and manifest were created in `pykrita/krita_mcp`. The resulting observed package was:

```text
libkrita_mcp_native_17d8e3e7d908b1f4.so
SHA256: 17d8e3e7d908b1f4fb9d8abf6805df620ccd383e15cc2a2c83b5840406b06aec
RUNPATH: removed
```

For reproducing the packaging with a different build, run the following **only after the helper exists**, and check that its existing RPATH consists solely of temporary Krita AppImage mounts:

```bash
cd "$HOME/Applications/Krita/krita-mcp"
python3 - <<'PY'
from pathlib import Path
import hashlib, json, shutil, subprocess

root = Path.cwd()
source = root / "native/build/libkrita_mcp_native.so"
target = root / "pykrita/krita_mcp"
manifest_path = target / "native_manifest.json"
if not source.is_file() or not target.is_dir():
    raise SystemExit("Expected existing build/plugin not found")

rpath = subprocess.check_output(["patchelf", "--print-rpath", str(source)], text=True).strip()
if rpath and not all(p.startswith("/tmp/.mount_krita-") for p in rpath.split(":")):
    raise SystemExit(f"Unexpected RPATH: {rpath}")
if rpath:
    subprocess.run(["patchelf", "--remove-rpath", str(source)], check=True)
if subprocess.check_output(["patchelf", "--print-rpath", str(source)], text=True).strip():
    raise SystemExit("Could not remove temporary RPATH")

digest = hashlib.sha256(source.read_bytes()).hexdigest()
name = f"libkrita_mcp_native_{digest[:16]}.so"
shutil.copy2(source, target / name)
manifest_path.write_text(json.dumps({
    "file": name, "sha256": digest, "abi": "krita-6.0.4-qt6"
}, indent=2) + "\n")
print(name, digest)
PY
```

**Why this matters:** the helper is intended to load inside Krita's own runtime environment. Removing RPATH did not by itself prove it would load correctly; the subsequent self-test did.

## 6. Isolated plugin installation

**Do not run the installer without isolated environment variables** in this workflow: the default installer writes to `~/.local/share/krita/pykrita` and `~/.config/kritarc`, removing/replacing an existing `krita_mcp` plugin directory. Check for any existing plugin before installation.

With Krita closed:

```bash
cd "$HOME/Applications/Krita/krita-mcp"
export XDG_DATA_HOME="$PWD/native/build/xdg-data"
export XDG_CONFIG_HOME="$PWD/native/build/xdg-config"
python3 mcp_server.py install
```

Observed:

```text
Installed plugin to .../native/build/xdg-data/krita/pykrita/krita_mcp
Enabled enable_krita_mcp in .../native/build/xdg-config/kritarc
```

Launch **this AppImage**, using the **same** isolated paths:

```bash
cd "$HOME/Applications/Krita/krita-mcp"
XDG_DATA_HOME="$PWD/native/build/xdg-data" \
XDG_CONFIG_HOME="$PWD/native/build/xdg-config" \
"$HOME/Applications/Krita/krita-6.0.4-x86_64.AppImage"
```

**Observed:** Settings → Configure Krita → Python Plugin Manager displayed **MCP Bridge** with its checkbox enabled. No files were installed into the checked default plugin path. The normal user `~/.config/kritarc` existed and was deliberately left alone.

## 7. Strict version guard rejected the correct Krita build

First self-test connected and performed pixel operations, but yielded **5/6**:

```text
[FAIL] exception OpError: Native helper requires Linux/Krita 6.0.4/Qt6/little-endian.
```

Inspection of `pykrita/krita_mcp/native_commands.py` found:

```python
b._krita().version() != "6.0.4"
```

The running Krita reported `6.0.4 (git 57db7cd)`; the condition rejected that valid version *before attempting to load the native library*. The same guard was present in the installed isolated copy.

**Local source correction applied to both copies** (repository source **and** isolated installed plugin):

```python
not b._krita().version().startswith("6.0.4")
```

This is a one-expression change; the Qt6/little-endian checks were preserved. Krita was then restarted using the isolated XDG environment.

**Important:** this correction was observed as an **uncommitted local change** in the `krita-mcp` checkout after testing. Do not claim it was committed to the upstream repository or overwrite it during later steps.

## 8. Confirmed 10/10 self-test

With the isolated Krita 6 AppImage running:

```bash
cd "$HOME/Applications/Krita/krita-mcp"
XDG_DATA_HOME="$PWD/native/build/xdg-data" \
XDG_CONFIG_HOME="$PWD/native/build/xdg-config" \
python3 mcp_server.py --selftest
```

**Observed result:**

```text
health: Krita 6.0.4 (git 57db7cd), plugin 1.7.0, 66 operations
[ok] krita_instance         6.0.4 (git 57db7cd)
[ok] create_document        64x48
[ok] root_layer             Background
[ok] pixel_channel_order    read back r=255 g=0 b=0 on a little-endian host
[ok] pixel_roundtrip        read #0080ff
[ok] draw                   pixel at 10,10 is #ff0000
[ok] native_draw_history
[ok] png_encode             516 bytes
[ok] create_group_layer     grouplayer
[ok] layer_tree             self test group, Background
10/10 checks passed
```

This verifies bridge communication, image round-tripping, native drawing/history, PNG encoding and layer operations in the tested session. It does **not** yet verify warp operation correctness, rendering performance, long-run stability or arbitrary Qt ABI compatibility.

## COMPLETED — REMAINING — NEXT ACTION

**COMPLETED:** Separate Krita 6.0.4 AppImage preserved alongside existing 5.3.4.1 Flatpak; native helper built, version-tag requirement suppressed, temporary RUNPATH removed, isolated MCP bridge enabled; 10/10 self-tests passed.

**REMAINING:** Validate native warp/deformation operations with an expendable test image, then connect Qwen Code through existing LM Studio setup. Confirm stability and avoid modifying original artwork. Long-term, rebuild against version-matched Qt 6.8 development libraries if the workaround proves brittle.

**NEXT ACTION:** Preserve the working uncommitted fix in the local `krita-mcp` checkout, establish the Qwen Code MCP configuration, then test a minimal masked deformation on a disposable image. Do not repeat the already-successful build tests unless a regression appears.

## Caveats

- **Observed:** one system ran 10/10 tests. **Not verified:** a fresh-machine install or universal compatibility.
- The version-tag workaround is narrowly scoped; don't treat it as a safe general Qt compatibility technique.
- The temporary `/tmp/.mount_krita-*` path must never be committed into a persistent RPATH.
- No original Tiger images, private material or third-party compiled libraries are published here.
- The project source and local modification belong to the upstream checkout; this repository contains **documentation only**.

## 9. Bionic + Gemma E4B and real image-editing tests (4 November 2026)

**Source and status:** User-observed Krita UI and Bionic results, plus Qwen Code session exports; these are local observations, not upstream guarantees. The documented checkout is still `443e22c`. The existing native-helper build, isolated XDG plugin and uncommitted `native_commands.py` version-check fix remain intact.

### Agent and bridge comparison

- **Observed:** Qwen Code 0.26.6 connected to the existing `krita-mcp` bridge via LM Studio using `google/gemma-4-e4b` (Q6_K default, 7.21 GB; Q4_K_M also present). It correctly retrieved the `warp_layer` schema and, after Krita was restarted with the isolated XDG launcher, executed a **preview** in 178 ms. The tool response reported centre (994,722), offset (0,-2), radius 40, `changed_pixels=4114`, `mutated=false`, and `source_preserved=true`. **Not visually verified:** whether the preview made the intended visible Tiger movement. Treat it as an executed preview, not a successful animation.
- **Observed:** A more autonomous Qwen Code magenta-Tiger task failed to create its layer: repeated `create_layer` calls were refused by Qwen Code's **deferred-tool schema-review gate**; Gemma retried malformed calls until retry-loop protection intervened. This is an agent/tool-call issue, not evidence of a Krita MCP failure.
- **Observed:** LM Studio **Bionic** accepted the *existing* Krita MCP stdio server under Settings → Integrations → MCP → Add custom MCP, reporting **connected, 57 tools ready**. Working setup: command `/usr/bin/python3`; argument `/home/tim/Applications/Krita/krita-mcp/mcp_server.py`; working directory `/home/tim/Applications/Krita/krita-mcp`; `XDG_DATA_HOME=/home/tim/Applications/Krita/krita-mcp/native/build/xdg-data`; `XDG_CONFIG_HOME=/home/tim/Applications/Krita/krita-mcp/native/build/xdg-config`; request timeout 60 seconds. Krita AppImage must launch with those same XDG paths. The default KDE menu launch omitted them and left the MCP Bridge unavailable.
- **Observed:** Bionic + Gemma E4B drew a large magenta rectangle on a new layer. It did **not** fulfil the visibility requirement: layer was almost transparent and below Background, although the agent's final message claimed full success. The user manually repaired order and opacity and saw the rectangle.
- **Observed:** A second green rectangle attempt reported new layer `index=1`, `opacity=1`, `visible=true`, and one drawn pixel command; again behind Background and approximately 0.39% opaque, regardless of the agent's success claim.
- **Observed:** After supplying exact parameter values `index: 0`, `opacity: 255`, `visible: true`, `blending_mode: "normal"`, a fresh Bionic test created `BIONIC - FIXED VALUES TEST` above `Background`, painted a visible green 400×200 rectangle near (760,440), and inspected resulting layer properties: test index 0, Background index 1, opacity 255, visible true. **User visually confirmed the green box over the original image.** Bionic reported 29 seconds. No need to repeat rectangle tests absent a regression.
- **Inference:** Bionic currently completes this limited layered-drawing workflow with fewer agent/tool retries than the Qwen Code sessions tested, but neither agent's verbal claim alone establishes successful visual editing. The explicit 0/255 conventions resolved the two observed Bionic layer-configuration errors; the underlying Krita layer/draw methods worked when correctly invoked.

### Krita MCP parameter conventions and upstream references

At `443e22c`, `mcp_server.py` describes `create_layer` as **index 0 is the top**, with negative indices counting from the bottom, but does not explain the `opacity` property in the `create_layer` schema. Its `set_layer` tool description accepts **opacity 0–255 or 0–1**, which is ambiguous for agents. In the successful explicit test, **255** means fully opaque and **1** in the 0–255 scale is nearly transparent; do not assume bare `1` means 100% without documenting the conversion. Layer inspection returns opacity on the 0–255 scale. Larger nonnegative layer indices go below smaller indices. **Proposed but not implemented:** improve descriptions of `create_layer` and `set_layer` to make layer order and opacity scale unmistakable; do not change actual conversion or semantics without tests.

Upstream material to consult *before making code changes*:

- [Krita MCP usage](https://github.com/thuanlm215/krita-mcp/blob/443e22c/docs/usage.md) — layer tree, indices, operation behaviour and responses.
- [Workflow documentation](https://github.com/thuanlm215/krita-mcp/blob/443e22c/docs/workflows.md) — warp, drawing, source preservation and undo.
- [Deformation example](https://github.com/thuanlm215/krita-mcp/blob/443e22c/examples/example_deformation_workflow.py) — preview `warp_layer`, then apply, inspect images.
- [Artist workflow](https://github.com/thuanlm215/krita-mcp/blob/443e22c/examples/example_artist_workflow.py) — preserve original, duplicate working layers, edit and inspect.
- [MCP tool definitions](https://github.com/thuanlm215/krita-mcp/blob/443e22c/mcp_server.py) — live agent-facing schemas.
  
`warp_layer` **apply** already inserts a new edited copy and hides the source (see upstream workflows); do not duplicate the original pre-emptively unless required for a distinct purpose.

### Continuation

**COMPLETED:** 10/10 bridge self-tests, Qwen Code warp-preview operation, Bionic 57-tool connection, visibly verified layer creation and green drawing with explicit values.

**REMAINING:** Verify where a warp lands on Tiger and whether it creates a visibly useful deformation. Consider a Bionic **Krita skill** grounded in the upstream usage/workflow examples and explicit opacity/index rules **before** patching working MCP source. Independently verify any future schema change. Never treat tool metadata or agent claims as a substitute for visible inspection.

**NEXT ACTION:** Return to an unchanged disposable original image and perform one controlled, visually observable deformation using the upstream warp workflow, then compare against the original. Do not repeat completed builds, installations or green-rectangle tests.

## 10. First user-confirmed visible Tiger warp (4 November 2026)

**Observed:** Bionic with Gemma 4 E4B successfully called the existing `warp_layer` tool after an initial parameter error. The model mistakenly passed `center: [1050,680]` three times; the MCP rejected it with `params.center is not of a type(s) object`. The accepted format is **`center: {"x":1050,"y":680}`**, as used for `offset`. The agent initially attributed this to a server typing problem, but the evidence shows invalid caller arguments.

**Observed preview:**
```json
{
  "document": 0,
  "layer": "Background",
  "mode": "preview",
  "center": {"x": 1050, "y": 680},
  "offset": {"x": 0, "y": -30},
  "radius": 160,
  "max_size": 1024
}
```
The bridge returned `changed_pixels=73038`, region `{"x":890,"y":490,"width":320,"height":320}`, `source_preserved=true`, `mutated=false`, `elapsed_ms=471`, and a PNG preview response. **Preview mode does not visibly update the Krita document**; the user saw no change on the canvas. Preview metadata is not proof of visually satisfactory deformation.

**Observed applied test:** Repeated those geometry parameters with `mode: "apply"` and `name: "TIGER - VISIBLE WARP"`. **User-reported elapsed time for this Bionic test: 24 seconds** (end-to-end Bionic work duration, distinct from the 471 ms MCP preview execution). The user supplied **before and after Krita images showing a visible change**, with Tiger's rounded back/torso distorted and background sofa stitching/upholstery also visibly shifted. The user's visual evidence establishes that a deformation was rendered in Krita; it does **not** establish that the warp is suitable as breathing. The background distortion is the outstanding defect for this visual task.

**Next action:** Use the upstream `select_content`/selection-mask workflow to constrain the existing working warp to Tiger's torso; verify the sofa and plate remain stationary, then reduce warp strength for natural breathing. Do not repeat earlier infrastructure setup or rectangle tests. Keep the original unchanged, and inspect both the layer stack and actual before/after pixels.

**REA follow-up idea:** Bionic's 57-tool Krita MCP discovery and subsequent operations suggest comparing its agent tool handling with the **already installed** REA server. This is an untested proposal, not a confirmed Bionic–REA connection. Prefer a small read-only REA/Ghidra task first, preserve Qwen Code MCP configuration and existing analysis files, and use the established REA docs/status before setting connection commands.
