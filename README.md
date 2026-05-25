# Whisper Models for CTranslate2

This repository contains CTranslate2-converted Whisper models for use with [faster-whisper](https://github.com/systran/faster-whisper).

## Models included

| Model | Directory | Size | Source |
|---|---|---|---|
| Whisper large-v3 | `whisper-large-v3/` | ~3.1 GB (split) | [openai/whisper-large-v3](https://huggingface.co/openai/whisper-large-v3) |
| Whisper medium | `faster-whisper-medium/` | ~1.5 GB | [openai/whisper-medium](https://huggingface.co/openai/whisper-medium) |

## Usage

```python
from faster_whisper import WhisperModel

# Use the medium model
model = WhisperModel("faster-whisper-medium/")

# Or the large-v3 model (after reassembly, see below)
model = WhisperModel("whisper-large-v3/")

segments, info = model.transcribe("audio.mp3")
for segment in segments:
    print("[%.2fs -> %.2fs] %s" % (segment.start, segment.end, segment.text))
```

## Cloning and setup

### faster-whisper-medium

No special steps needed. The `model.bin` (1.5 GB) is stored via Git LFS and downloaded automatically on clone.

### whisper-large-v3

The `model.bin` (2.9 GB) exceeds GitHub's 2 GB per-file LFS limit, so it was split into two parts. After cloning, reassemble it:

**Linux / macOS / Git Bash:**

```bash
cd whisper-large-v3
cat model.bin.part_* > model.bin
```

**Windows (cmd):**

```cmd
cd whisper-large-v3
copy /b model.bin.part_aa+model.bin.part_ab model.bin
```

> **Warning:** Do NOT use PowerShell's `cat` (`Get-Content`) for binary files — it corrupts the output by converting to UTF-16.

### Verification

The reassembled `model.bin` should be approximately **2.9 GB** (3,094,236,160 bytes).

## Repository structure

```
├── faster-whisper-medium/
│   ├── .gitattributes
│   ├── README.md
│   ├── config.json
│   ├── model.bin            (1.5 GB, Git LFS)
│   ├── tokenizer.json
│   └── vocabulary.txt
├── whisper-large-v3/
│   ├── .gitattributes
│   ├── config.json
│   ├── model.bin.part_aa    (1.9 GB, Git LFS)
│   ├── model.bin.part_ab    (1.1 GB, Git LFS)
│   ├── preprocessor_config.json
│   ├── tokenizer.json
│   └── vocabulary.json
├── .gitignore
└── README.md
```
