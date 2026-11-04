# ChatGPT Exporter: Export button disappears after page load

> **Status (4 November 2026): LOCALLY VERIFIED WORKAROUND — Continuity local.5.**
>
> Applies to the locally customised Tampermonkey userscript **ChatGPT Exporter (Continuity All Formats — Shared Images)**, previously **2.38.0-local.4**, then updated to **local.5**. This is a record of the observed fix, not a claim that upstream or every ChatGPT layout is affected.

## Observed symptom

- The **Export** button appeared briefly when reloading `chatgpt.com`, then disappeared after the page initialised.
- The browser settings had not been changed.
- Before the fix, the user could briefly catch the export button during reload, but it did not remain accessible.

## Explanation and scope

**Inferred:** ChatGPT's dynamically rendered navigation/sidebar can change during startup. The local script's preferred insertion point may be present in the DOM but subsequently hidden or replaced, leaving the export control inaccessible. The exact browser-side DOM transition was not independently captured; do not describe this as a proven upstream root cause.

The targeted local.5 adjustment was designed to choose a **visible** sidebar insertion point and recover with a fallback floating Export control when no suitable visible sidebar anchor is available. The intended scope was the UI attachment/recovery behaviour, **not** export parsing, shared images, bulk export or saved preferences.

## Verified results (4 November 2026)

1. The user replaced the installed local.4 script with the supplied **local.5** build through Tampermonkey.
2. On a fresh page reload, the Export button appeared, briefly disappeared, **then returned**.
3. The restored Export button opened and produced a successful **Markdown export**, shared back into the conversation as `ChatGPT-Qwen_Code_Catchup_Review.md`.
4. The workaround therefore **restored a usable Export button** across the observed page initialisation. The short flicker remains; it was not considered worth an additional change.

**Not verified:** HTML, JSON, bulk exports and shared-image packaging were not individually re-tested after local.5; their code was intended to remain unchanged. The exact compatibility of local.5 with later ChatGPT layouts and Tampermonkey releases is unknown.

## Reproduction / safe diagnosis

1. Confirm the active userscript and its version in Tampermonkey; check for duplicate enabled exporters before editing.
2. On `chatgpt.com`, reload and observe whether the Export control appears transiently or remains available.
3. If transient, inspect the **existing** UI insertion/recovery code before changing the export pipeline. Check whether its chosen DOM anchor is connected **and visible**.
4. Retain the existing script as a backup. Prefer a narrow fix which reattaches the UI or uses a fallback only when its normal location is unsuitable.
5. Reload, verify that the button returns, click Export, and successfully export one test conversation. Test other formats separately before declaring them verified.

## Relationship to upstream

The local exporter is derived from **[pionxzh's ChatGPT Exporter](https://github.com/pionxzh/chatgpt-exporter)** (MIT-licensed). The user reported that the upstream maintainer accepted a separately contributed **HTML-link conversion fix**. That improvement is distinct from the local.5 disappearing-button workaround; **do not assume the latter is upstream**.

## Archived working source

- [ChatGPT Exporter (Continuity All Formats — Shared Images) **2.38.0-local.5**](chatgpt-exporter/ChatGPT-Exporter-Continuity-Shared-Images-local.5.user.js) — the exact tested userscript, retained with upstream author attribution and MIT licence. Git blob SHA-1: `468532da6346191a4f6e4588c8bcc571cdce9254`; file SHA-256: `5efd7f2851884af3616ff601db975769d03bf78b3bc89b4e9d12f28881881d9b`. This script was confirmed to restore the Export button and export Markdown on 4 November 2026.

## Preservation

Keep working Tampermonkey settings and export functionality intact. Do not replace the customised shared-image exporter with a stock version merely to fix its button position. Avoid committing personal conversation exports, private images, credentials or browser-profile data.
