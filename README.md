# Rosely Zenith 13 MXFP8 — Vast Serverless Unified Deployment

Zenith 13 replacement for the previous ZiTC 9.2 deployment. It keeps the proven architecture:
**Vast ComfyUI Serverless + official `comfyui-json` PyWorker + backend-selected workflow JSON**.
There is no custom inference server in this repo.

## Core stack
- `Zenith_13.0_MXFP8_E4M3.safetensors`
- `qwen_3_4b_fp8_mixed.safetensors`
- `flux_vae.safetensors`
- ComfyUI `vastai/comfy:v0.35.0-cuda-12.9-py312`
- PyWorker pinned to `2207a3f94b55a0921c1641520eeb83de5a0c1611`
- 12 steps, CFG 1.0, `dpmpp_sde`, `simple`
- `ConditioningZeroOut` negative conditioning

## Model bundle
```text
s3://rosely-infrastructure/serverless/zimage/zenith13/zenith13-mxfp8-unified-test.tar.zst
size: 10703686769 bytes
sha256: 70062f128985c4e187e50d914ebc4bc073b3941ece4221e4bd8c7e6f8e51308a
```
Provisioning validates size + SHA-256 before extraction and validates the exact eight model files.

## Modes
- `realistic` — no LoRA
- `realistic_snapshot` — RealisticSnapshot V5 @ 0.60
- `realistic_amateur` — Amateur Photography @ 0.60
- `anime_illustria` — Illustria 01 @ 0.70
- `anime_modern` — Z-Image Anime 01 @ 0.70
- `anime_elusarca` — Elusarca @ 0.90

LoRAs use ComfyUI core `LoraLoaderModelOnly`, so they patch only the diffusion model and never touch the Qwen text encoder. Amateur Photography is included for A/B testing; its author recommends a specialized workflow for best results.

## Vast template
```text
Docker image: vastai/comfy:v0.35.0-cuda-12.9-py312
SERVERLESS=true
BACKEND=comfyui-json
PYWORKER_REPO=https://github.com/vast-ai/pyworker
PYWORKER_REF=2207a3f94b55a0921c1641520eeb83de5a0c1611
BENCHMARK_JSON_PATH=/workspace/zenith13_benchmark.json
PROVISIONING_SCRIPT=https://raw.githubusercontent.com/robert2398/rosely_zit_vast_realistic_anime_unified/main/provision_vast_zit_unified.sh
```
Copy remaining variables from `endpoint-env.example`. Never commit real AWS keys.

MXFP8 note: the checkpoint loads on current stock ComfyUI. Native MXFP8 tensor-core matmul is Blackwell-only; RTX 3090/4090 use the compatible fallback/dequant path, while RTX 5090 is the direct consumer choice for native MXFP8 testing.

## No worker.py / model_server.py
The official Vast `comfyui-json` worker already exposes `/generate/sync` and forwards complete workflow JSON to ai-dock's ComfyUI API wrapper. This package deliberately does not replace that layer.

## Validate
```bash
bash -n provision_vast_zit_unified.sh
python scripts/validate_repo.py
python scripts/test_builder.py
python -m compileall -q .
```

## Smoke tests
```bash
pip install vastai
export VAST_API_KEY=...
python test_vast_unified.py --mode realistic
python test_vast_unified.py --mode realistic_snapshot
python test_vast_unified.py --mode realistic_amateur
python test_vast_unified.py --mode anime_illustria
python test_vast_unified.py --mode anime_modern
python test_vast_unified.py --mode anime_elusarca
```
Use the same seed/prompt/resolution across modes for a controlled A/B test. See `VERIFICATION.md` for the audit record.
# rosely-zit-v13-vast-serverless
