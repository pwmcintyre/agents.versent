---
name: local-ai
description: Run tasks on local AI models using LM Studio
---

## context

- LM Studio server status: !`/Users/peter/.lmstudio/bin/lms server status`
- Available models: !`/Users/peter/.lmstudio/bin/lms ls 2>/dev/null | grep -E "^[a-z]" | head -5 || echo "No models loaded"`

## your task

Execute a prompt against a local AI model and return the result.

### usage

Use this skill to offload tasks to your local models. Useful for:
- Running private/sensitive operations locally
- Testing prompts before sending to API
- Batch processing with local models
- Exploring capabilities of open models without API costs

### examples

**Simple query:**
```
/local-ai What is the capital of France?
```

**With specific model:**
```
/local-ai --model qwen3.6-35b-a3b@iq2_xxs "Write a Python function for..."
```

**Complex task:**
```
/local-ai "Analyze this code for performance issues: [code]"
```

### instructions

1. Parse the skill input to extract the prompt and optional `--model MODEL_NAME`
2. If no model specified, use the currently loaded model
3. Run: `/Users/peter/.lmstudio/bin/lms chat --prompt "[prompt]"` (add `--model` if specified)
4. Return the model's response as-is
5. If LM Studio is not running, instruct user to run: `/Users/peter/.lmstudio/bin/lms server start`
6. Do not modify or summarize the output

### notes

- Current setup: Qwen 35B XXS (heavily quantized, 32K context, 13.88 GB VRAM)
- Good for code generation, analysis, and reasoning tasks
- Private by default (runs locally, no external API calls)
