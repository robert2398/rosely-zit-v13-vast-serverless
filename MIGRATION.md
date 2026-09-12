# Migration from ZiTC 9.2 to Zenith 13 MXFP8

This package is intended to replace the contents of the existing
`robert2398/rosely_zit_vast_realistic_anime_unified` deployment repository.
The provisioning filename is intentionally unchanged so the existing raw-GitHub
`PROVISIONING_SCRIPT` URL can remain stable after the new files are pushed to `main`.

## What changed

- Diffusion model: `ZiTC_9.2_BF16` -> `Zenith_13.0_MXFP8_E4M3.safetensors`
- Text encoder: old abliterated Qwen -> `qwen_3_4b_fp8_mixed.safetensors`
- VAE remains the Z-Image/Flux-compatible VAE stored as `Flux/flux_vae.safetensors`
- Generation defaults: 12 steps, CFG 1.0, `dpmpp_sde`, `simple`
- Negative conditioning uses `ConditioningZeroOut`
- Six selectable workflow modes are included
- LoRAs use ComfyUI core `LoraLoaderModelOnly`
- Model bundle is a single verified `.tar.zst` object in `rosely-infrastructure`
- Vast ComfyUI image updated to `vastai/comfy:v0.35.0-cuda-12.9-py312`
- Vast PyWorker is pinned to commit `2207a3f94b55a0921c1641520eeb83de5a0c1611`

## Deliberately NOT included

There is no custom `worker.py` and no `model_server.py`. The supported Vast
`comfyui-json` worker already exposes `/generate/sync` and passes complete API-format
workflow JSON to the ComfyUI API wrapper. This avoids duplicating Vast's worker/model
server lifecycle and readiness logic.

## Deploy

1. Replace/push this package into the existing deployment repository.
2. In the Vast template, use `vastai/comfy:v0.35.0-cuda-12.9-py312`.
3. Copy the variables from `endpoint-env.example` and insert the model-download AWS credentials.
4. Keep `PROVISIONING_SCRIPT` pointing to this repo's `provision_vast_zit_unified.sh`.
5. Start one worker and inspect provisioning logs until the archive SHA-256 validates and all eight assets are installed.
6. Run the baseline smoke test first, then all LoRA modes with the same seed/prompt.

Do not commit real AWS keys to GitHub.
