# Qwen3.6 and Qwen3.8 27B MTP draft depth

- **Date:** 2026-08-14
- **Distro:** Ubuntu Server 26.04 minimal
- **Linux kernel:** 7.0.0-29
- **Mesa RADV:** 26.0.3
- **llama.cpp:** b10327 (`69bf64379`)

Results are calculated from a three-run median with a 128-token output for
each workload and configuration.

## Qwen3.6 27B Q6_K

| MTP drafts | Coding generation | Coding acceptance | General text generation | Text acceptance |
|---:|---:|---:|---:|---:|
| 0 | 9.61 t/s | — | 9.59 t/s | — |
| 1 | 16.12 t/s | 88.06% | 15.91 t/s | 85.29% |
| 2 | 20.44 t/s | 79.59% | **19.25 t/s** | 72.12% |
| 3 | **21.22 t/s** | 69.11% | 18.89 t/s | 57.55% |
| 4 | 16.63 t/s | 44.24% | 17.84 t/s | 48.83% |

## Qwen3.8 27B UD-Q6_K_XL

| MTP drafts | Coding generation | Coding acceptance | General text generation | Text acceptance |
|---:|---:|---:|---:|---:|
| 0 | 8.52 t/s | — | 8.53 t/s | — |
| 1 | 14.35 t/s | 86.27% | 13.38 t/s | 73.97% |
| 2 | 16.54 t/s | 68.75% | 15.68 t/s | 61.95% |
| 3 | **17.59 t/s** | 60.64% | 15.85 t/s | 51.12% |
| 4 | 17.11 t/s | 51.01% | **16.66 t/s** | 48.92% |

## Method

- One server slot, 32,768-token context, F16 KV, full Vulkan offload, flash
  attention, batch 2048, and ubatch 256.
- Identical deterministic coding and general-text prompts, greedy sampling,
  seed 42, disabled thinking, and disabled prompt-cache reuse.
- A 32-token warm-up for each workload preceded three measured runs. Depth 0
  disabled speculative decoding.
- Qwen3.8 configurations used a 30-second recovery interval between depths.

## llama-server command used

```bash
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-server \
  --model $PATH_TO_GGUF \
  --alias $ALIAS \
  --device Vulkan0 \
  --n-gpu-layers all \
  --flash-attn on \
  --ctx-size 32768 \
  --parallel 1 \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --batch-size 2048 \
  --ubatch-size 256 \
  --threads 16 \
  --threads-batch 32 \
  --load-mode mmap \
  --fit off \
  --no-mmproj \
  --no-context-shift \
  --reasoning on \
  --reasoning-preserve \
  --reasoning-format deepseek \
  --spec-type draft-mtp \
  --spec-draft-n-max $N_DRAFTS \
  --metrics \
  --no-ui \
  --host 127.0.0.1 \
  --port 8088
```

The two speculative-decoding arguments were omitted for depth 0.
