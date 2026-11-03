# AMD Strix Halo Linux workstation notes

> **Status:** Local observations, not an official compatibility matrix or general installation guide.

## Tested hardware

- AMD Ryzen AI Max+ 395 (Strix Halo)
- 128 GB unified system memory
- Nobara Linux (Fedora-based) with KDE
- Local inference through LM Studio and Qwen Code
- Ghidra and REA in the same workstation environment

This setup ran large local coding models and a connected reverse-engineering workflow. The size of the model file alone does not establish runtime memory use, performance or reliable context length.

## Lessons that transferred to local coding agents

- Long context budgets and large tool outputs can consume considerable memory and tokens even when inference appears fast.
- A low-parameter-active MoE model can be useful for repeated tool calls, but benchmark **actual tool performance and accuracy**, not just active parameter counts.
- Verify the *loaded* context length separately from the agent's own configured context.
- When benchmarking, record model identifier, quantisation, model-file size, inference backend, context window, output-token limit, wall-clock time, and failures.

## ROCm note from a separate local experiment

A separate Podman-hosted ComfyUI/LTX experiment observed a persistent post-render CPU busy-loop in ROCm `AsyncEventsLoop`, with elevated temperatures. A local launcher calling `hipSetDeviceFlags(4)` before ComfyUI initialisation was tested and returned idle CPU use to approximately zero in that configuration. A later launcher that bypassed the workaround reproduced the high-load state.

**Do not generalise this to all ROCm systems or blindly set HIP flags:** this is a configuration-specific observation, not an upstream verified universal fix. The working scripts remain part of that project's own setup.

## NVIDIA

No NVIDIA hardware has been tested as part of the documented REA/Ghidra/Qwen Code findings. Contributions covering new NVIDIA Linux machines should include driver, CUDA version, GPU, model, runtime, commands and reproducible results. Do not present AMD-specific ROCm mitigations as NVIDIA advice.
