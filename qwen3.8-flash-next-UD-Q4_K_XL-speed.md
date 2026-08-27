# Qwen3.8 Flash Next UD-Q4_K_XL

- **Date:** 2026-08-27
- **System:** AMD Ryzen AI MAX+ 395, Radeon 8060S, 128 GB unified memory
- **llama.cpp:** early PR [#27742](https://github.com/ggml-org/llama.cpp/pull/27742), `d807f04ea`
- **Model:** `Qwen3.8-Flash-Next-UD-Q4_K_XL.gguf` (111.3 GB), no MTP layers
- **Settings:** Vulkan RADV, full offload, F16 KV, flash attention, batch 2048, ubatch 256

| Used context | Prompt processing | Generation | Peak RAM increase |
|---:|---:|---:|---:|
| 512 | 321.99 t/s | 25.22 t/s | 104.77 GiB |
| 4,096 | 258.46 t/s | 21.49 t/s | 104.76 GiB |
| 16,384 | 214.12 t/s | 17.41 t/s | 105.77 GiB |
| 32,768 | 192.49 t/s | 14.44 t/s | 106.81 GiB |
| 65,536 | 161.41 t/s | 10.59 t/s | 109.07 GiB |
| 131,040 | 122.71 t/s | 7.71 t/s | 109.28 GiB |

Generation was measured after filling each context. The 128K generation result is a shorter 14-token sample because the model emitted EOS. MTP startup fails because this early GGUF contains no MTP/NextN layers.

## Commands used

The 512–64K prompt-processing and generation measurements used:

```bash
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-bench -m "$PATH_TO_GGUF" -ngl 999 -b 2048 -ub 256 -t 16 \
  -fa on -ctk f16 -ctv f16 -lm auto -p "$CONTEXT" -n 0

VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-bench -m "$PATH_TO_GGUF" -ngl 999 -b 2048 -ub 256 -t 16 \
  -fa on -ctk f16 -ctv f16 -lm auto -p 0 -n 32 -d "$CONTEXT"
```

The 128K combined request used:

```bash
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-server \
  --model "$PATH_TO_GGUF" \
  --device Vulkan0 \
  --n-gpu-layers 999 \
  --flash-attn on \
  --ctx-size 131072 \
  --parallel 1 \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --batch-size 2048 \
  --ubatch-size 256 \
  --threads 16 \
  --load-mode auto \
  --tensor-read-lazy auto \
  --host 127.0.0.1 \
  --port "$PORT"
```
