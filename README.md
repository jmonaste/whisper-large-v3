---
language:
  - en
  - zh
  - de
  - es
  - ru
  - ko
  - fr
  - ja
  - pt
  - tr
  - pl
  - ca
  - nl
  - ar
  - sv
  - it
  - id
  - hi
  - fi
  - vi
  - he
  - uk
  - el
  - ms
  - cs
  - ro
  - da
  - hu
  - ta
  - 'no'
  - th
  - ur
  - hr
  - bg
  - lt
  - la
  - mi
  - ml
  - cy
  - sk
  - te
  - fa
  - lv
  - bn
  - sr
  - az
  - sl
  - kn
  - et
  - mk
  - br
  - eu
  - is
  - hy
  - ne
  - mn
  - bs
  - kk
  - sq
  - sw
  - gl
  - mr
  - pa
  - si
  - km
  - sn
  - yo
  - so
  - af
  - oc
  - ka
  - be
  - tg
  - sd
  - gu
  - am
  - yi
  - lo
  - uz
  - fo
  - ht
  - ps
  - tk
  - nn
  - mt
  - sa
  - lb
  - my
  - bo
  - tl
  - mg
  - as
  - tt
  - haw
  - ln
  - ha
  - ba
  - jw
  - su
  - yue
tags:
  - audio
  - automatic-speech-recognition
license: mit
library_name: ctranslate2
---

# Whisper large-v3 model for CTranslate2

This repository contains the conversion of [openai/whisper-large-v3](https://huggingface.co/openai/whisper-large-v3) to the [CTranslate2](https://github.com/OpenNMT/CTranslate2) model format.

This model can be used in CTranslate2 or projects based on CTranslate2 such as [faster-whisper](https://github.com/systran/faster-whisper).

## Example

```python
from faster_whisper import WhisperModel

model = WhisperModel("large-v3")

segments, info = model.transcribe("audio.mp3")
for segment in segments:
    print("[%.2fs -> %.2fs] %s" % (segment.start, segment.end, segment.text))
```

## Conversion details

The original model was converted with the following command:

```
ct2-transformers-converter --model openai/whisper-large-v3 --output_dir faster-whisper-large-v3 \
    --copy_files tokenizer.json preprocessor_config.json --quantization float16
```

Note that the model weights are saved in FP16. This type can be changed when the model is loaded using the [`compute_type` option in CTranslate2](https://opennmt.net/CTranslate2/quantization.html).

## More information

**For more information about the original model, see its [model card](https://huggingface.co/openai/whisper-large-v3).**

---

## Hosting on GitHub - Setup notes

### Problem

The original `model.bin` file weighs **2.9 GB**, which exceeds GitHub's limits:

- **GitHub standard push limit**: 2 GB per pack.
- **GitHub LFS limit per file**: 2 GB (2,147,483,648 bytes).

Attempting to push the file directly or even via Git LFS results in rejection by GitHub.

### Solution: Split model + Git LFS

The model file was split into two parts, each under the 2 GB limit, and tracked with Git LFS.

#### Steps performed

1. **Initialize Git LFS**

   ```bash
   git lfs install
   ```

2. **Configure `.gitattributes`**

   The `.gitattributes` file (originally from Hugging Face as `gitattributes` without the dot) was renamed to `.gitattributes` so Git recognizes it. A new rule was added for the split parts:

   ```
   *.bin.part_* filter=lfs diff=lfs merge=lfs -text
   ```

3. **Split `model.bin` into parts under 2 GB**

   ```bash
   split -b 1900M model.bin model.bin.part_
   ```

   This produces:
   - `model.bin.part_aa` — 1.9 GB
   - `model.bin.part_ab` — 1.1 GB

4. **Stage and commit**

   The split parts are tracked by LFS (verified with `git lfs status`), while the original `model.bin` is excluded from the repository.

5. **Push to GitHub**

   The first push uploaded the LFS objects (3.1 GB total) but the connection was reset by the remote. A second `git push` succeeded immediately since the LFS objects were already uploaded.

### Reassembling the model after cloning

After cloning the repository, reconstruct `model.bin` from the parts:

**Linux / macOS / Git Bash:**

```bash
cat model.bin.part_* > model.bin
```

**Windows (PowerShell):**

> **Warning:** Do NOT use PowerShell's `cat` (`Get-Content`) for binary files — it corrupts the output by converting to UTF-16 (resulting in ~6 GB instead of 2.9 GB).

Use `cmd` instead:

```powershell
cmd /c "copy /b model.bin.part_aa+model.bin.part_ab model.bin"
```

### Verification

The reassembled `model.bin` should be approximately **2.9 GB** (3,094,236,160 bytes). If the file is significantly larger (~6 GB), it was likely concatenated incorrectly using PowerShell's `cat`.

### Repository structure

| File | Size | Tracked by |
|---|---|---|
| `config.json` | ~2 KB | Git |
| `preprocessor_config.json` | ~1 KB | Git |
| `tokenizer.json` | ~2.2 MB | Git |
| `vocabulary.json` | ~1 MB | Git |
| `README.md` | — | Git |
| `.gitattributes` | ~1.5 KB | Git |
| `model.bin.part_aa` | 1.9 GB | Git LFS |
| `model.bin.part_ab` | 1.1 GB | Git LFS |
