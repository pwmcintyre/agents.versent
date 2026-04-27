---
name: lm-studio
description: Run tasks on local LM Studio models using the CLI
---

## context

- LM Studio server status: !`/Users/peter/.lmstudio/bin/lms server status`
- Available models: !`/Users/peter/.lmstudio/bin/lms ls --format json 2>/dev/null | jq -r '.[0].model' 2>/dev/null || /Users/peter/.lmstudio/bin/lms ls | head -5`

## your task

Execute a prompt against a local LM Studio model and return the result.

### usage

Use this skill to offload tasks to your local LM Studio models. Useful for:
- Testing prompts locally before using with Anthropic models
- Running private/sensitive operations on local models
- Batch processing with local models
- Exploring different model capabilities

### examples

**Simple query:**
```
/lm-studio What is the capital of France?
```

**With specific model:**
```
/lm-studio --model qwen2.5-7b-instruct-1m "Write a short poem about rain"
```

**Complex task:**
```
/lm-studio --model mistral-nemo-instruct-2407 "Analyze this code for bugs: [code here]"
```

### instructions

1. Parse the skill input to extract the prompt and optional model name
2. If no model is specified, use the most recently loaded model or the smallest suitable model from the available list
3. Run: `/Users/peter/.lmstudio/bin/lms chat --prompt "[prompt]"` (or with `--model` flag if specified)
4. Return the model's response as-is, without modification
5. If LM Studio is not running, return an error asking the user to start it with `/Users/peter/.lmstudio/bin/lms server start`
6. Do not output anything else

### format

Input format: `[--model MODEL_NAME] <prompt>`

Output: The raw response from the LM Studio model
