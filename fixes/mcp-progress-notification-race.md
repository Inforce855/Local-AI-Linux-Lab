# MCP progress notifications: observed disconnect and local mitigation

> **Status (3 November 2026): LOCALLY TESTED WORKAROUND, NOT A RELEASED UPSTREAM FIX.**
>
> The steps below describe a targeted investigation in one Qwen Code build. **Do not paste or run a blind patch against a different build.** Check the upstream SDK issue/PR and your installed bundle first.

## Observable failure

A `qwen --debug` log showed:

```text
[MCP] MCP ERROR (rea): Received a progress notification for an unknown token:
{"method":"notifications/progress","params":{"progress":2,"message":"list_documents: failed","progressToken":4}}
```

After the SDK error, Qwen Code marked the REA MCP connection unhealthy. In the observed run, health checks and an automatic REA subprocess restart followed, discarding process-local Ghidra state. Later symptoms included `No app is open`, `evidence_integrity_mismatch` and repetitive attempts to reopen or reanalyse the binary.

**Do not conflate** this genuine disconnect chain with a normal `target_unavailable` error immediately after starting a fresh REA session, or with an agent choosing incorrect tools.

## Root-cause hypothesis supported by the SDK analysis

In the relevant TypeScript MCP SDK path, a progress notification and its matching response could arrive together in a transport read. Notification handlers were deferred to a microtask while response handling synchronously removed the request's progress handler. The progress handler then ran too late and an unknown-token error was raised.

The matching upstream work is:

- [modelcontextprotocol/typescript-sdk PR #3023 — deliver progress notifications that share a read with their response](https://github.com/modelcontextprotocol/typescript-sdk/pull/3023)
- [Related issue #3018](https://github.com/modelcontextprotocol/typescript-sdk/issues/3018)

The PR narrowed synchronous delivery specifically to `notifications/progress` and preserved existing deferred handling for other notifications. This distinction matters: making every notification synchronous would be a broader behavioural change.

## What was changed locally

The locally installed Qwen Code version was **0.26.6** under its managed npm-update installation. Two bundled chunks containing notification dispatch were modified:

- `chunks/chunk-AFSLFBFT.js`
- `chunks/chunk-YK7NDQKM.js`

The original files were backed up separately. The modification applied a narrowly scoped synchronous dispatch path to progress notifications; other notifications retained deferred dispatch. Both modified chunks passed `node --check`.

**No prebuilt patch, replacement JS files, copied proprietary code or automatic patcher is distributed here.** File names and function shapes are build-specific. A Qwen Code update can replace those files; disabling updates was a temporary way to preserve the local experiment, not recommended ongoing maintenance.

## What testing actually proved

- A controlled `list_documents` call in a fresh session returned a normal `target_unavailable` structured tool error without causing the previous progress-notification/MCP disconnection sequence in the observed debug log.
- A subsequent session opened the target library, ran `analyze_function`, inspected the returned evidence and listed the same open document again. The earlier evidence remained retrievable.
- This is evidence that the local fix mitigated the observed failure **in those tests**. It is not proof that every progress event ordering, every REA function or every SDK version is safe.
- A separate second-session debug-log audit would be needed to conclusively rule out *every* transient restart in that longer test.

## Reproduction and safe diagnosis

1. Save your Qwen Code version and REA/Ghidra versions.
2. Start Qwen Code with `qwen --debug`.
3. Reproduce using a narrow, read-only REA operation; save timestamps and logs.
4. Look for the **exact** `unknown token` progress-notification error and the following health-check/restart events.
5. Compare your bundled SDK behaviour with the upstream PR **before** considering local modification.
6. Prefer an official upstream fix when it becomes available; do not apply a build-specific patch blindly.

## Security / data hygiene

Keep original source backups and file hashes for any local experiment. Inspect uncommitted changes before touching installed code. Never publish full unredacted logs, personal paths, access tokens or biometric data. Preserve working driver source separately; this investigation did not require modifying it.
