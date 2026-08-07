# Qwen3.6 27B Q6_K MTP batch and ubatch

- **Date:** 2026-08-07
- **Distro:** Ubuntu Server 26.04 minimal
- **Linux kernel:** 7.0.0-27
- **Mesa RADV:** 26.0.3
- **llama.cpp:** b10238 (`4ed2b13f7`)
- **Model:** Unsloth Qwen3.6 27B Q6_K, embedded MTP

Results are calculated from a three-run median on a 16,384-token prompt for each configuration.

| Batch | Ubatch | 16K prompt | vs 2048/512 | MTP generation | Peak GTT |
|---:|---:|---:|---:|---:|---:|
| 512 | 256 | 216.91 t/s | +4.1% | 19.11 t/s | 22.886 GiB |
| 1024 | 128 | 199.40 t/s | -4.3% | 19.12 t/s | 22.864 GiB |
| 1024 | 256 | 218.71 t/s | +5.0% | 19.10 t/s | 22.896 GiB |
| 2048 | 128 | 199.72 t/s | -4.2% | 19.12 t/s | 22.884 GiB |
| **2048** | **256** | **218.93 t/s** | **+5.1%** | 19.09 t/s | 22.915 GiB |
| 2048 | 512 | 208.38 t/s | baseline | 19.08 t/s | 23.097 GiB |
| 2048 | 1024 | 194.92 t/s | -6.5% | 19.02 t/s | 23.452 GiB |
| 2048 | 2048 | 152.39 t/s | -26.9% | 19.04 t/s | 24.434 GiB |
| 4096 | 256 | 217.99 t/s | +4.6% | 19.09 t/s | 22.954 GiB |
| 4096 | 512 | 208.40 t/s | +0.0% | 19.09 t/s | 23.136 GiB |
| 8192 | 256 | 217.06 t/s | +4.2% | 19.03 t/s | 23.032 GiB |
| 8192 | 1024 | 193.26 t/s | -7.3% | 18.96 t/s | 23.569 GiB |
| 16384 | 256 | 217.51 t/s | +4.4% | 18.97 t/s | 23.189 GiB |
| 16384 | 512 | 207.41 t/s | -0.5% | 19.00 t/s | 23.371 GiB |
| 16384 | 2048 | 151.32 t/s | -27.4% | 18.92 t/s | 24.708 GiB |
| 32768 | 256 | 216.88 t/s | +4.1% | 18.83 t/s | 23.501 GiB |

## Prompt-size check

The final head-to-head pass used exact token prefixes from the same corpus and
disabled prompt-cache reuse on every request.

| Prompt tokens | 2048/512 | 2048/256 | Improvement |
|---:|---:|---:|---:|
| 512 | 240.42 t/s | 240.77 t/s | +0.1% |
| 2048 | 228.11 t/s | 237.33 t/s | +4.0% |
| 8192 | 219.81 t/s | 226.49 t/s | +3.0% |
| 16384 | 206.19 t/s | 216.46 t/s | +5.0% |

## Method

- One server slot, 32,768-token context, F16 KV, full Vulkan offload, flash
  attention, and MTP depth 2.
- Fixed synthetic technical corpus tokenized by the server. Each screening
  request evaluated exactly 16,384 uncached tokens with `n_predict=0`.
- A 512-token prompt-processing warm-up preceded every screening run.
- MTP generation used the same deterministic coding prompt, seed, sampler, and
  128-token limit for every configuration.
- Peak swap stayed at the pre-existing 47.1 MiB. The runs produced no server
  error, allocation failure, GPU fault, timeout, or reset, and GTT returned to
  0.017 GiB after shutdown.

## llama-server command used

```bash
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-server \
  --model /path/to/unsloth-Qwen3.6-27B-MTP-GGUF/Qwen3.6-27B-Q6_K.gguf \
  --alias qwen3.6-27b-q6k-mtp \
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
  --spec-draft-n-max 2 \
  --metrics \
  --no-ui \
  --host 127.0.0.1 \
  --port 8088
```
