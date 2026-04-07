# Local Voice — Plan

> Persistent plan and progress tracker. Update the status checklist as work progresses.
> Last updated: 2026-04-07

---

## Status

### Phase 1 — Documentation
- [x] `README.md` — architecture, STT/TTS options, MCP integration research, Node.js client snippets
- [x] `PLAN.md` — this file
- [ ] `setup/mlx-audio.md` — install, serve, model pull, STT + TTS curl smoke tests
- [ ] `setup/whisper-cpp.md` — build from source, Core ML ANE option, endpoint caveat

### Phase 2 — MCP Server
- [ ] `mcp/package.json` — deps: `@modelcontextprotocol/sdk`, `zod`, `openai`
- [ ] `mcp/tsconfig.json`
- [ ] `mcp/src/index.ts` — `transcribe_audio(file_path)` + `speak_text(text)` tools
- [ ] `mcp/README.md` — build steps, registration in `opencode.json`
- [ ] `opencode/voice-mcp-config.json` — drop-in `mcp` block for `opencode.json`

### Phase 3 — Standalone Demo
- [ ] `scripts/voice_loop.mjs` — mic (sox) → STT API → opencode subprocess → TTS API → afplay
- [ ] `opencode/voice-notes.md` — prompt tips, future hooks ideas, caveats

---

## Goals

1. Run Whisper STT locally on Metal — fast, private, no cloud dependency
2. Run Kokoro TTS locally on Metal — high quality, Apache 2.0
3. Expose both as MCP tools in OpenCode so the LLM can use them during any session
4. Provide a standalone voice loop outside OpenCode for fully autonomous voice sessions
5. Document setup so peers can replicate on Apple Silicon

**Non-goals (for now):**
- Real-time streaming VAD (voice activity detection)
- Wake-word detection
- Automatic read-aloud of every LLM response (needs OpenCode hooks API — not available yet)
- Windows / Linux support (Apple Silicon is the primary target)

---

## Hardware

| Property | Value |
|---|---|
| Machine | MacBook M1 Pro |
| Unified memory | 32 GB |
| GPU cores | 16 (M1 Pro) |
| OS | macOS (darwin) |
| Inference framework | MLX (via mlx-audio) |
| Existing venv | `~/.venvs/mlx` (shared with local-llm) |

---

## Stack Decisions

| Layer | Choice | Rationale |
|---|---|---|
| Inference server | `mlx-audio` (Python) | Metal-native, single server for STT + TTS, OpenAI-compatible endpoints, installs into existing `~/.venvs/mlx` |
| STT model | `whisper-large-v3-turbo` | Best speed/accuracy trade-off; 1.5 GB; well-supported in mlx-community |
| TTS model | `Kokoro-82M-bf16` | #1 TTS Arena ranking; 82M params; Apache 2.0; 54+ voices |
| MCP server | TypeScript (`@modelcontextprotocol/sdk`) | Consistent with Node.js preference; thin HTTP client; strongly typed |
| Standalone client | Node.js ESM (`openai` npm, `voice_loop.mjs`) | `openai` package handles multipart STT + TTS streaming cleanly |
| Audio capture (standalone) | `sox` CLI | Available via `brew install sox`; zero-dep mic recording from shell |
| Audio playback | `afplay` (built-in macOS) | No install needed |

---

## Decisions Log

| Date | Decision | Rationale |
|---|---|---|
| 2026-04-07 | Use `mlx-audio` as inference server | Single server for STT + TTS, Metal-native, OpenAI-compatible `/v1/audio/transcriptions` and `/v1/audio/speech`, installs into existing venv |
| 2026-04-07 | Document `whisper.cpp` as alternative | Core ML / ANE encoder option (3× faster encoder); worth knowing; but non-standard endpoint |
| 2026-04-07 | TypeScript MCP server (not Python) | Consistent with Node.js preference across this project; `@modelcontextprotocol/sdk` is well-maintained (24.5M weekly downloads); thin HTTP client is trivial in TS |
| 2026-04-07 | Skip Coqui TTS | Company shut down early 2024; last release Dec 2023; superseded by Kokoro |
| 2026-04-07 | Don't attempt auto read-aloud in OpenCode | OpenCode has no hooks/event system; would require prompt engineering to make LLM call `speak_text` — fragile. Document as future work. |
| 2026-04-07 | MCP STT tool is file-based (not streaming) | MCP spec has no mic capture primitive; file path is the practical input type |
