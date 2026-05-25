# Voice Pipeline (Fork Customizations)

This fork replaces the upstream local-MLX LLM with an Ollama HTTP backend and
gates responses on the wake word "hey assistant". Conversation history is kept
across turns.

## Prerequisites

- Ollama running locally with the `lfm2` model pulled:
  ```bash
  ollama serve              # if not already running
  ollama pull lfm2
  ```
- A working input device (built-in mic is fine; Bluetooth headsets often fail
  to negotiate HFP mode and feed silence — see Troubleshooting).

## Run

List your audio devices first if you've never run this before:

```bash
uv run python -m sounddevice
```

Then start the pipeline. Pass the input device by name so it's stable across
device reconnects:

```bash
HF_HUB_DISABLE_PROGRESS_BARS=1 PYTHONWARNINGS=ignore \
  uv run python -m mlx_audio.sts.voice_pipeline \
  --input_device "MacBook Pro Microphone"
```

Once you see the model-loading lines finish, say:

> **Hey assistant**, what's the date today?

The terminal shows the finalized transcript, calls Ollama, and prints the
reply:

```
🎙️  You: Hey assistant, what's the date today?
     ...thinking
🤖 Assistant: Today is May 24, 2026.
```

Turns without "hey assistant" are ignored (logged as `(ignored)`).

## Useful flags

| Flag | Default | Notes |
|---|---|---|
| `--input_device` | system default | Name or index from `python -m sounddevice` |
| `--llm_model` | `lfm2` | Any model name pulled in Ollama |
| `--llm_base_url` | `http://localhost:11434` | Ollama endpoint |
| `--wake_word` | `hey assistant` | Empty string disables the gate |
| `--no_play` | off | Skip TTS audio playback (still logs the reply) |
| `--verbose` | off | Full structured `event=...` debug logs |

## Troubleshooting

- **Silent input (RMS=0 even though the mic works in other apps).** Usually
  zombie audio streams from a previous `^C`-killed run, or a Bluetooth headset
  still in A2DP-only mode. Fully quit the terminal app (cmd-Q) and reopen.
- **No `event=response_ready` after a transcript.** Check Ollama:
  `curl -s http://localhost:11434/api/tags` should list `lfm2`.
- **Wrong device picked.** macOS reshuffles device indices when devices are
  added or removed. Always pass `--input_device` by name, not index.
