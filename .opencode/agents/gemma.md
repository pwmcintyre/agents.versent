# Agent: Gemma

Local Gemma 4 26B MoE (MLX) agent. Fast, offline, bounded tasks.

**Model:** `mlx/mlx-community/gemma-4-26b-a4b-it-4bit`  
**Speed:** ~40 t/s, 1.2s TTFT  
**Max output:** 1000 tokens (before compaction risk)

---

## How to Use

Call this agent with any prompt. It'll hit the local MLX server (no cloud roundtrip).

Examples:
- "Explain what this function does: [code]"
- "Generate a commit message for: [diff summary]"
- "Suggest 3 better names for this variable: [context]"
- "Fix grammar: [text]"

Keep inputs under 500 tokens for best performance.

---

## Implementation

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  baseURL: "http://localhost:8080/v1",
  apiKey: "not-used",
});

export async function callGemma(prompt: string): Promise<string> {
  const response = await client.messages.create({
    model: "mlx-community/gemma-4-26b-a4b-it-4bit",
    max_tokens: 1000,
    messages: [
      {
        role: "user",
        content: prompt,
      },
    ],
  });

  return response.content[0].type === "text" ? response.content[0].text : "";
}
```

---

## Caveats

- **Server must be running:** Start with the command in `findings.md`
- **Compaction risk:** Very large prompts (>2000 tokens) trigger KV cache reorganization
- **No context:** Each call is fresh; no conversation history
- **Offline only:** Requires MLX server on localhost:8080
