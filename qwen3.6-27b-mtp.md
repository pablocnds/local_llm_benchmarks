# Qwen3.6 27B MTP

- **Distro:** Ubuntu Server 26.04 minimal
- **Linux kernel:** 7.0.0-27
- **Mesa RADV:** 26.0.3
- **llama.cpp:** b10238 (`4ed2b13f7`)

| Model | Base generation | MTP coding | Draft acceptance | KLD vs Q8 ↓ |
|---|---:|---:|---:|---:|
| Bartowski Q4 + sidecar | 12.09 t/s | **25.80 t/s** | 85.31% | 0.007081 |
| Unsloth UD-Q4_K_XL | 12.03 t/s | 24.09 t/s | 77.80% | 0.006072 |
| Unsloth Q5_K_M | 11.06 t/s | **23.09 t/s** | 82.18% | 0.003076 |
| Unsloth UD-Q5_K_XL | 10.72 t/s | 22.60 t/s | 82.18% | 0.003232 |
| Unsloth Q6_K | 9.61 t/s | **20.95 t/s** | 84.27% | 0.001967 |
| Unsloth Q8_0 | 7.74 t/s | 16.77 t/s | 81.76% | reference |

## llama-server command

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

Bartowski GGUFs have the MTP draft model separate. Add:

```bash
--spec-draft-model ${MTP_DIR}/mtp-Qwen_Qwen3.6-27B-Q4_0.gguf \
--spec-draft-device Vulkan0 \
--spec-draft-ngl all
```
