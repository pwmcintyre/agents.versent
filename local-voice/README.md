# Local Voice — STT and TTS on Apple Silicon

Exploration of running speech-to-text (STT) and text-to-speech (TTS) locally on an M1 Pro
32 GB MacBook, and wiring them into [OpenCode](https://opencode.ai) via MCP.

See [PLAN.md](./PLAN.md) for status, decisions, and next steps.

---

## Goal

1. Run Whisper STT locally on Metal (no cloud, no latency)
2. Run Kokoro TTS locally on Metal
3. Expose both as MCP tools in OpenCode so the LLM can transcribe audio files and speak responses
4. Provide a standalone voice loop demo (mic → STT → OpenCode → TTS → speaker)

---

## Hardware Context

| Property | Value |
|---|---|
| Machine | MacBook M1 Pro |
| Unified memory | 32 GB |
| Framework | Apple Metal (via MLX or whisper.cpp) |
| OS | macOS (darwin) |

---

## Architecture

```
[OpenCode session — via MCP]

  User drops an audio file or records a snippet
       ↓
  LLM calls transcribe_audio(file_path)
       ↓ MCP tool call
  TypeScript MCP server → POST /v1/audio/transcriptions (mlx-audio, port 8000)
       ↓
  Whisper large-v3-turbo (Metal) → transcribed text → returned to LLM

  LLM calls speak_text(text)
       ↓ MCP tool call
  TypeScript MCP server → POST /v1/audio/speech (mlx-audio, port 8000)
       ↓
  Kokoro 82M (Metal) → audio file → afplay → speaker


[Standalone voice loop — outside OpenCode]

  sox (mic capture) → POST /v1/audio/transcriptions
       ↓
  opencode run --print (LLM response)
       ↓
  POST /v1/audio/speech → afplay
```

Two layers:

| Layer | Tool | Language | Notes |
|---|---|---|---|
| Inference server | `mlx-audio` | Python (runs once, background) | Metal-native; single process for both STT and TTS; OpenAI-compatible endpoints |
| MCP server | TypeScript | Node.js (spawned by OpenCode) | Thin HTTP client; exposes `transcribe_audio` + `speak_text` tools |
| Standalone demo | `voice_loop.mjs` | Node.js | Full mic→LLM→speaker loop, no MCP needed |

---

## STT Options

### mlx-audio (recommended)

- **Repo:** [Blaizzy/mlx-audio](https://github.com/Blaizzy/mlx-audio) (6.6k stars)
- **Install:** `pip install mlx-audio` into `~/.venvs/mlx`
- **Model:** `mlx-community/whisper-large-v3-turbo-asr-fp16` (~1.5 GB, best speed/accuracy)
- **Endpoint:** `POST /v1/audio/transcriptions` — fully OpenAI-compatible
- **Acceleration:** Metal (Apple GPU via MLX framework)
- **Also handles TTS** on the same server process (port 8000)

```sh
mlx_audio.server --host 127.0.0.1 --port 8000
```

```sh
# Smoke test
curl -X POST http://localhost:8000/v1/audio/transcriptions \
  -F "file=@audio.wav" \
  -F "model=mlx-community/whisper-large-v3-turbo-asr-fp16"
```

### whisper.cpp (alternative — Core ML / ANE)

- **Repo:** [ggml-org/whisper.cpp](https://github.com/ggml-org/whisper.cpp) (48k stars, v1.8.4 Mar 2026)
- **Build:** C++, build from source with `cmake -B build -DWHISPER_COREML=1` for ANE support
- **Acceleration:** Metal by default; Core ML (Apple Neural Engine) encoder with build flag — ~3× faster encoder
- **Endpoint:** `POST /inference` — **not** OpenAI-compatible (different path and format)
- **Trade-off:** more control, ANE support, but requires manual build and non-standard endpoint

> Use whisper.cpp if you need ANE encoder acceleration or a compiled binary with no Python dep.
> Use mlx-audio for everything else — simpler, single server, standard endpoints.

---

## TTS Options

### mlx-audio + Kokoro (recommended)

- **Model:** `mlx-community/Kokoro-82M-bf16` — 82M params, Metal-native, Apache 2.0
- **Quality:** ranked #1 in TTS Arena; exceptional for its size
- **Endpoint:** `POST /v1/audio/speech` — OpenAI-compatible (same server as STT above)
- **Voices:** 54+ presets (`af_heart`, `af_bella`, `af_nova`, `af_sky`, `am_adam`, `bf_alice`, …)
- **Languages:** English (US/UK), Spanish, French, Hindi, Italian, Japanese, Portuguese, Mandarin

```sh
curl -X POST http://localhost:8000/v1/audio/speech \
  -H "Content-Type: application/json" \
  -d '{"model": "mlx-community/Kokoro-82M-bf16", "input": "Hello!", "voice": "af_heart"}' \
  --output speech.wav
```

### Other MLX TTS models available via mlx-audio

| Model | Notes |
|---|---|
| `mlx-community/csm-1b` | Voice cloning with reference audio |
| `mlx-community/Qwen3-TTS-12Hz-1.7B-VoiceDesign-bf16` | Multi-language, voice design |
| `mlx-community/Voxtral-4B-TTS-2603-mlx-bf16` | Mistral, 9 languages, 20 voices |
| `mlx-community/Dia-1.6B-fp16` | Dialogue-focused |

### Coqui TTS — do not use

Coqui.ai shut down in early 2024. The `TTS` package last released v0.22.0 in December 2023
and is effectively abandoned. Superseded by Kokoro.

---

## MCP Integration

### Why MCP

OpenCode fully supports MCP servers (`type: local` via stdio). Exposing STT/TTS as MCP tools
means the LLM can invoke them during any session without any special prompt or CLI flag.

### Node.js MCP SDK

- **Package:** `@modelcontextprotocol/sdk` v1.29.0 (24.5M weekly downloads, MIT)
- **Pattern:** `McpServer` + `StdioServerTransport` + `zod` for schema validation
- Stdio servers: never use `console.log` (corrupts JSON-RPC stream) — use `console.error`

### MCP audio spec

Audio is a first-class content type in the MCP spec (2025-11-25):

```json
{ "type": "audio", "data": "<base64>", "mimeType": "audio/wav" }
```

However: no microphone capture primitive exists in MCP. Audio *output* from tools is in-spec;
audio *input* requires passing a file path or base64 blob.

### Existing MCP audio servers (for reference)

| Repo | Language | Notes |
|---|---|---|
| [kachiO/mlx-whisper-mcp](https://github.com/kachiO/mlx-whisper-mcp) | Python | mlx-whisper directly; tools: `transcribe_file`, `transcribe_audio` (base64), `transcribe_youtube` |
| [SmartLittleApps/local-stt-mcp](https://github.com/SmartLittleApps/local-stt-mcp) | TypeScript | whisper.cpp; `transcribe`, `transcribe_long`, `transcribe_with_speakers` |
| [johnmatthewtennant/mcp-voice-hooks](https://github.com/johnmatthewtennant/mcp-voice-hooks) | JS | Claude Code hooks system — does **not** work in OpenCode (no hooks API) |

### Limitations in OpenCode today

| Capability | Status |
|---|---|
| LLM calls `transcribe_audio(file_path)` | Works — practical, well-supported |
| LLM calls `speak_text(text)` | Works, but LLM must be prompted to call it |
| Automatic "read every response aloud" | Not possible — OpenCode has no hooks/event system yet |
| True hands-free bidirectional voice | Not practical without a hooks API |

---

## Calling Local Endpoints from Node.js

Use the `openai` npm package with `baseURL` pointing at the local server.
It handles multipart STT upload and TTS streaming transparently:

```typescript
import OpenAI, { toFile } from 'openai';
import fs from 'fs';

const client = new OpenAI({
  apiKey: 'not-needed',
  baseURL: 'http://localhost:8000/v1',
});

// STT
const transcription = await client.audio.transcriptions.create({
  file: fs.createReadStream('audio.wav'),
  model: 'whisper-1',
});
console.log(transcription.text);

// TTS
const speech = await client.audio.speech.create({
  model: 'mlx-community/Kokoro-82M-bf16',
  voice: 'af_heart',
  input: 'Hello from local TTS',
});
const buffer = Buffer.from(await speech.arrayBuffer());
fs.writeFileSync('output.wav', buffer);
```
