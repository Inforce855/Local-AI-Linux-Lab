# Contributing

Contributions are welcome from people running local AI tools on Linux, including AMD, NVIDIA and CPU-only systems.

## What makes a useful report?

A reproducible finding contains:

- **What you wanted to do** and the exact task/prompt
- **Environment:** distribution, kernel, hardware, software and model versions, quantisation and inference backend
- **Observed behaviour:** exact, redacted error message and relevant timestamp
- **Reproduction:** smallest set of operations that demonstrates the problem
- **Diagnosis:** clear distinction between observed facts and hypotheses
- **Change:** exact scope, upstream reference where relevant
- **Verification:** before/after outcome and what *was not* tested
- **Status:** fixed locally, confirmed upstream, unresolved or superseded

Avoid claims that a model 'supports' a workflow merely because it ran one successful call. For a tool-agent issue, distinguish transport failures, tool schema refusal, context settings, model errors and source-analysis accuracy.

## Safety and privacy

Do not upload full unredacted agent conversations, API keys, credentials, home-directory paths that identify people, biometric images/templates, private DLLs, copyrighted binaries or proprietary model files. Link upstream PRs and issues; do not redistribute patched dependency bundles.

## Repository workflow

Use existing files wherever possible. Check `PROJECT_STATUS.md` and earlier decisions before modifying guidance. Write focused commits to the existing repository structure and do not add new branches without prior agreement from the maintainer.

Suggested labels in documentation: **Observed**, **Inferred**, **Upstream documented**, **Not verified**.
