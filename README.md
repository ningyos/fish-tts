# fish-tts

Inference-focused package for [Fish-Speech](https://github.com/fishaudio/fish-speech).

Built with [Fish Audio](https://fish.audio).

## Highlights

Compared to the upstream Fish-Speech, this package adds:

- **Standalone package**: Extracted pure inference code, no fish_speech dependency needed
- **Mandatory torch.compile + Inductor**: coordinate descent tuning, persistent cache, ~120 tokens/sec (RTF ~0.26). Without compile, inference is ~10x slower
- **Singleton pattern**: Thread-safe model loading, loaded once per process
- **Prefilled references**: Pre-encode voice profiles and reuse across calls, avoiding redundant encoding
- **Pipeline streaming**: Token generation and vocoder decoding run in parallel (separate thread + queue), ~18% faster end-to-end
- **Dynamic references**: Add/remove voice profiles at runtime without reloading
- **bf16 vocoder**: DAC vocoder auto-cast to bf16 on CUDA for faster decode

## Installation

```bash
uv add git+https://github.com/smolGura/fish-tts
```

## Usage

### Basic Synthesis

```python
from fish_tts import get_instance

# Get singleton instance (first call loads and compiles model)
synth = get_instance()

# Basic synthesis
audio = synth.synthesize("Hello world")
```

### Voice Cloning

```python
from fish_tts import get_instance, VoiceProfile

synth = get_instance()

# Load voice profile
profile = VoiceProfile.load("voice.npy", text="reference transcript")

# Set reference (prefilled for efficiency)
synth.set_references([profile])

# Synthesize with cloned voice
audio = synth.synthesize("Text to speak")
```

### Streaming

```python
# Pipeline streaming for low latency
for chunk in synth.synthesize_stream("Long text to speak..."):
    play_audio(chunk)
```

### Dynamic Reference Management

```python
# Add/remove voice profiles at runtime
synth.add_reference(another_profile)
synth.clear_references()
```

## Performance

| Mode | Speed | RTF |
|------|-------|-----|
| torch.compile | ~120 tokens/sec | ~0.26 |
| Pipeline streaming | +18% faster | - |

## Built by

This project was built with [Claude Code](https://claude.ai/claude-code) (Claude Opus 4.5 / 4.6).

## License

Licensed under the [Fish Audio Research License](LICENSE).
Research and non-commercial use is free. Commercial use requires a separate
license from [Fish Audio](https://fish.audio).

See [NOTICE](NOTICE) for attribution details.

## References

- [Fish-Speech](https://github.com/fishaudio/fish-speech)
- [Fish-Speech Paper](https://arxiv.org/html/2411.01156v1)
