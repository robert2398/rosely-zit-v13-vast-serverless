# Verification record

Prepared: 2026-09-12

- Existing deployment repository head audited: `c3c698f3f901f52b38fa614ea232722c54c9915e`.
- Existing repo uses Vast's standard PyWorker and contains no custom root `worker.py` or `model_server.py`.
- Upstream Vast PyWorker pinned here: `2207a3f94b55a0921c1641520eeb83de5a0c1611`.
- Upstream `workers/comfyui-json/worker.py` serves `/generate/sync` through ai-dock's ComfyUI API wrapper on port 18288.
- ComfyUI image updated from old v0.25.1 to `vastai/comfy:v0.35.0-cuda-12.9-py312`.
- Current stock ComfyUI supports MXFP8 checkpoints; pre-Blackwell NVIDIA GPUs use the compatible fallback/dequant path rather than native MXFP8 tensor-core math.
- Official Z-Image pattern verified: `CLIPLoader(type=lumina2)`, Qwen3-4B encoder, Z-Image VAE, zeroed negative conditioning.
- ComfyUI v0.35.0 core contains `LoraLoaderModelOnly`; LoRA modes use it so the Qwen encoder is not patched.

Pinned model bundle:
` s3://rosely-infrastructure/serverless/zimage/zenith13/zenith13-mxfp8-unified-test.tar.zst `
Size `10703686769`; SHA-256 `70062f128985c4e187e50d914ebc4bc073b3941ece4221e4bd8c7e6f8e51308a`.
Provisioning validates remote size, full archive SHA-256, and the exact 8 safetensors before install.

Deliberate worker decision: this repo contains **no custom `worker.py` and no `model_server.py`**.
The supported Vast `comfyui-json` PyWorker already accepts complete workflow JSON, so adding another inference-server layer would only add failure modes.

Checks run before ZIP creation:
- `bash -n provision_vast_zit_unified.sh`
- Python compileall
- JSON parse of all workflows
- `python scripts/validate_repo.py`
- `python scripts/test_builder.py`

These are static/integration-contract checks. A real GPU/Vast smoke test still must be run after deployment because this build environment cannot instantiate a Vast GPU worker.

Additional package checks:
- ComfyUI v0.35.0 core node contract checked for `LoraLoaderModelOnly`.
- `dpmpp_sde` confirmed present in current KSampler names; `simple` confirmed as a scheduler.
- Vast base image contract checked for `BACKEND=comfyui-json`.
- Final ZIP is re-extracted and the repository validator/builder tests are run again against the extracted copy before delivery.
