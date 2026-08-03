# Qwen3.6 27B MTP

| Environment | Version |
|---|---|
| Ubuntu Server | 26.04 minimal |
| Linux kernel | 7.0.0-27 |
| Mesa RADV | 26.0.3 |
| llama.cpp | b10238 (`4ed2b13f7`) |

| Model | Base generation | MTP coding | Draft acceptance | KLD vs Q8 ↓ |
|---|---:|---:|---:|---:|
| Bartowski Q4 + sidecar | 12.09 t/s | **25.80 t/s** | 85.31% | 0.007081 |
| Unsloth UD-Q4_K_XL | 12.03 t/s | 24.09 t/s | 77.80% | 0.006072 |
| Unsloth Q5_K_M | 11.06 t/s | **23.09 t/s** | 82.18% | 0.003076 |
| Unsloth UD-Q5_K_XL | 10.72 t/s | 22.60 t/s | 82.18% | 0.003232 |
| Unsloth Q6_K | 9.61 t/s | **20.95 t/s** | 84.27% | 0.001967 |
| Unsloth Q8_0 | 7.74 t/s | 16.77 t/s | 81.76% | reference |
