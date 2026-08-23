# Qwen3.8 27B quant vibe bench

**TL;DR:** In one from-scratch Pi coding run per quant, Q4, Q6, and Q8 all
produced complete projects and scored 35/36 on the same independent test suite.
The observed quality difference was minimal; run time and token use varied more.

- **Date:** 2026-08-22 to 2026-08-23
- **Distro:** Ubuntu Server 26.04 minimal
- **Linux kernel:** 7.0.0-29
- **Mesa RADV:** 26.0.3
- **llama.cpp:** b10449 (`0d9ceae1e`)
- **Models:** Unsloth Qwen3.8 27B UD-Q4_K_XL, UD-Q6_K_XL, and UD-Q8_K_XL
- **Harness:** Pi coding agent
- **Sample size:** one project per quant

This is a small "vibe bench," not a statistically significant evaluation. Each
model received the same prompt, seed, sampling settings, tools, 128K context
limit, and empty project directory.

## Results

| Quant | GGUF size | Self-authored tests | Independent tests | Elapsed | Generation | MTP acceptance |
|---|---:|---:|---:|---:|---:|---:|
| UD-Q4_K_XL | 17.56 GB | 157/157 | 35/36 | 4:43:47 | 16.75 t/s | 76.47% |
| UD-Q6_K_XL | 25.30 GB | 143/143 | 35/36 | 4:33:32 | 13.85 t/s | 78.94% |
| UD-Q8_K_XL | 31.46 GB | 170/170 | 35/36 | 6:28:17 | 10.73 t/s | 82.02% |

| Quant | Input tokens | Cached prompt tokens | Output tokens | Peak context | Compactions | Tool calls / errors |
|---|---:|---:|---:|---:|---:|---:|
| UD-Q4_K_XL | 408,760 | 10,339,115 | 212,134 | 127,178 | 2 | 167 / 9 |
| UD-Q6_K_XL | 344,959 | 10,226,996 | 166,296 | 127,413 | 2 | 155 / 5 |
| UD-Q8_K_XL | 393,282 | 9,373,990 | 197,277 | 127,060 | 2 | 121 / 15 |

Output tokens include reasoning, text, and serialized tool calls. Cached prompt
tokens are counted again on each request and are not unique context tokens.

All three missed the same independent edge case: after `flush()`, an event at
`max_seen_ts` must be late even when `allowed_lateness_ms` is nonzero. Each
implementation continued to compare it only against
`max_seen_ts - allowed_lateness_ms`. The other 35 independent tests passed.

## Task and method

- The project was SignalWeave, a dependency-free Python event-time correlation
  engine with validation, a predicate parser, sequence matching, deterministic
  snapshot/restore, a CLI, documentation, and tests.
- Pi started in an empty directory with only `read`, `bash`, `edit`, and `write`.
- Extensions, skills, prompt templates, context files, telemetry, and network
  access were disabled.
- The independent 36-test evaluator was run only after Pi stopped and was not
  visible to the model.
- Public tests, the independent evaluator, `compileall`, server metrics, Pi
  events, and the complete session were captured for each run.
- All runs used two Pi context compactions and reached approximately 127K
  tokens before compaction.

## llama-server command used

`$MODEL` was changed for each quant. All other server arguments were identical.

```bash
VK_DRIVER_FILES=/usr/share/vulkan/icd.d/radeon_icd.json \
  llama-server \
  --model "$MODEL" \
  --alias "$ALIAS" \
  --device Vulkan0 \
  --n-gpu-layers all \
  --flash-attn on \
  --ctx-size 131072 \
  --parallel 1 \
  --cache-type-k f16 \
  --cache-type-v f16 \
  --batch-size 2048 \
  --ubatch-size 256 \
  --threads 16 \
  --threads-batch 32 \
  --load-mode auto \
  --fit off \
  --mmproj /path/to/mmproj-Qwen3.8-27B-F16.gguf \
  --no-context-shift \
  --reasoning on \
  --reasoning-preserve \
  --reasoning-format deepseek \
  --reasoning-budget 2048 \
  --reasoning-budget-message "I have enough analysis; I will now use the tools and complete the project." \
  --spec-type draft-mtp \
  --spec-draft-n-max 2 \
  --temp 1.0 \
  --top-p 0.95 \
  --top-k 20 \
  --min-p 0 \
  --presence-penalty 0 \
  --repeat-penalty 1 \
  --cache-prompt \
  --cache-ram 8192 \
  --ctx-checkpoints 32 \
  --checkpoint-min-step 8192 \
  --metrics \
  --no-ui \
  --host 127.0.0.1 \
  --port 8088
```

## Pi settings

Pi used the corresponding llama-server alias through its OpenAI-compatible
endpoint:

```json
{
  "providers": {
    "local": {
      "baseUrl": "http://127.0.0.1:8088/v1",
      "api": "openai-completions",
      "apiKey": "unused",
      "models": [{
        "id": "$ALIAS",
        "name": "Qwen3.8 27B quant vibe bench",
        "reasoning": true,
        "contextWindow": 131072,
        "maxTokens": 32768,
        "cost": {
          "input": 0,
          "output": 0,
          "cacheRead": 0,
          "cacheWrite": 0
        },
        "samplingParams": {
          "temperature": 1.0,
          "top_p": 0.95,
          "top_k": 20,
          "min_p": 0.0,
          "presence_penalty": 0.0,
          "repeat_penalty": 1.0,
          "seed": 424242
        },
        "compat": {
          "supportsReasoningEffort": false,
          "maxTokensField": "max_tokens",
          "thinkingFormat": "chat-template",
          "chatTemplateKwargs": {
            "enable_thinking": {"$var": "thinking.enabled"},
            "preserve_thinking": true,
            "reasoning_effort": {
              "$var": "thinking.effort",
              "omitWhenOff": true
            }
          }
        }
      }]
    }
  }
}
```

Pi was invoked as:

```bash
PI_TELEMETRY=0 pi \
  --mode json \
  --print \
  --approve \
  --offline \
  --provider local \
  --model "$ALIAS" \
  --thinking xhigh \
  --tools read,bash,edit,write \
  --no-extensions \
  --no-skills \
  --no-prompt-templates \
  --no-context-files \
  < PROMPT.md
```

The saved Pi sessions recorded the thinking level as `high` for all three runs.
