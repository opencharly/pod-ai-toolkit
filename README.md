# pod-ai-toolkit

The OpenCharly **training** pod: [Ostris AI Toolkit](https://github.com/ostris/ai-toolkit) — a
diffusion-model fine-tuning suite with a web GUI and a CLI trainer — packaged as one candy so a
CUDA host can train a LoRA and hand it to [`pod-comfyui`](https://github.com/opencharly/pod-comfyui)
for inference.

This repo owns the **candy**. The **image** that composes it lives in the distro repos
(`distro-cachyos`, `distro-fedora`), exactly as `pod-comfyui`'s does.

## What you get

```bash
charly box build ai-toolkit
charly config ai-toolkit
charly start ai-toolkit
# GUI: http://localhost:8675
```

- **GUI on 8675** — queue, monitor and stop training jobs; the worker spawns each job's trainer.
- **CLI** — `ai-toolkit-train <config.yml>` runs on the pod's own pinned venv regardless of PATH.
- **Pinned upstream** — ai-toolkit publishes no tags, so the candy pins a **commit** and the build
  asserts the checkout matches it.
- **State in a volume** — `~/ai-toolkit/{config,output,datasets}`; the application stays in the
  image at `/opt/ai-toolkit`.
- **A Z-Image LoRA example** — `/opt/ai-toolkit/config.examples/z-image-lora-de-turbo.yml`.

## Z-Image: train on De-Turbo, infer on Turbo

`Tongyi-MAI/Z-Image-Turbo` is a distilled 6B S3-DiT (8 NFEs, `guidance 0`, CFG disabled) whose own
model card rates fine-tunability **N/A**. Ostris publishes `ostris/Z-Image-De-Turbo` — the same
model de-distilled back to a CFG-using base — which is what the shipped example trains. The LoRA
then applies to Turbo at inference.

Two fields make or break it, and neither is guessable:

| Field | Value | Why |
|---|---|---|
| `model.arch` | `zimage` | selects the Z-Image handler |
| `model.extras_name_or_path` | `Tongyi-MAI/Z-Image-Turbo` | De-Turbo ships `transformer/` + a single-file and **no `text_encoder/`, no `vae/`** — the Qwen3 text encoder and the VAE must come from the complete Turbo diffusers repo |

## The pair: one host directory, two pods

Train in one pod, infer in the other, with no copy step:

| Pod | Same host directory mounts at |
|---|---|
| `ai-toolkit` | `/workspace/shared-loras` |
| `comfyui` | `~/ComfyUI/models/loras` |

The example config already writes to `/workspace/shared-loras`. Declare the bind on both deploys:

```yaml
volume:
    - name: shared-loras
      type: bind
      host: /home/<user>/charly-shared/zimage-loras
      path: /workspace/shared-loras
```

Then in ComfyUI: load **Z-Image-Turbo** (quantized weights), add a **LoraLoader** naming the
trained file, and generate at **8 steps / cfg 0** — Turbo's sampler configuration, deliberately
different from the training samples, which run on the CFG-capable base.

## Hardware

Validated against a **16 GB** card: a 6B DiT trains with `quantize: true`, `qtype: qfloat8`,
`quantize_te: true` and `gradient_checkpointing: true`. Drop `resolution` to `[512, 768]` if a
1024 bucket still OOMs.

## Layout

| Path | What |
|---|---|
| `charly.yml` | the `ai-toolkit` candy and its two skill nodes |
| `pixi.toml` | pins the Python **minor version** the runtime venv is built from |
| `configs/` | the shipped Z-Image LoRA example config |
| `.agents/skills/` | `ai-toolkit` (operator) and `ai-toolkit-layer` (candy internals) |

## Skills

- `/charly-ai-toolkit:ai-toolkit` — operating the pod, the Z-Image split, pairing with ComfyUI.
- `/charly-ai-toolkit:ai-toolkit-layer` — the candy's build contracts (commit pin, venv, symlinked
  state, the two services, the `ZImagePipeline` canary).

## Upstream

- ai-toolkit: <https://github.com/ostris/ai-toolkit> (no tags; commit-pinned)
- Z-Image-Turbo: <https://huggingface.co/Tongyi-MAI/Z-Image-Turbo> (Apache-2.0)
- Z-Image-De-Turbo: <https://huggingface.co/ostris/Z-Image-De-Turbo>
