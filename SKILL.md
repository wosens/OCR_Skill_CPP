---
name: ocr-cpp
description: DEFAULT OCR skill on Windows x64 — native C++ (RapidOcrOnnx = ONNX Runtime + PP-OCRv3, statically linked, NO Python, fast startup, zero dependencies). Use this BY DEFAULT to OCR / recognize text from images (jpg/png/webp/bmp/tiff), screenshots, scanned docs, photos of text; Chinese/English/multilingual. Use this UNLESS the user needs cross-platform (macOS/Linux) or is not on Windows x64 (then use the 'ocr' Python skill). Triggers on "OCR", "文字识别", "识别图片", "提取文字", "图片转文字", "图片识字", "扫描件转文本", "scan to text", "image to text".
---

# OCR Skill (Native C++, No Python)

Recognizes text from images using a **self-contained native C++ executable** (`RapidOcrOnnx.exe` — ONNX Runtime + PaddleOCR PP-OCRv3, statically linked). **No Python, no DLLs to install, no GPU required** (runs on CPU). Chinese + English + multilingual.

> The sibling skill `ocr` uses Python + RapidOCR. This `ocr-cpp` skill needs **nothing installed** on the target machine except the VC++ runtime. Prefer this one when you want zero-dependency native OCR on Windows.

## Requirements

- **Windows x64 only** (the bundled exe is a 64-bit Windows binary; it will not run on macOS/Linux/ARM).
- **VC++ Redistributable** present (most dev machines have it). If you see "`VCRUNTIME140_1.dll` missing", install the "Microsoft Visual C++ Redistributable 2015-2022 (x64)".

## Files in this skill

```
bin/RapidOcrOnnx.exe                   # self-contained native OCR (statically linked, ~16 MB)
models/
  ch_PP-OCRv3_det_infer.onnx           # text detection model
  ch_ppocr_mobile_v2.0_cls_infer.onnx  # orientation classifier
  ch_PP-OCRv3_rec_infer.onnx           # text recognition model
  ppocr_keys_v1.txt                    # character dictionary
```

## How to run

Run from this skill's root directory (where `SKILL.md` lives):

```bash
./bin/RapidOcrOnnx.exe \
  --models ./models \
  --det  ch_PP-OCRv3_det_infer.onnx \
  --cls  ch_ppocr_mobile_v2.0_cls_infer.onnx \
  --rec  ch_PP-OCRv3_rec_infer.onnx \
  --keys ppocr_keys_v1.txt \
  --image "<path-to-image>"
```

If the current directory is not the skill root, use absolute paths for `./bin/RapidOcrOnnx.exe` and `--models`. The skill lives at `C:\Users\<user>\.claude\skills\ocr-cpp`.

### Useful options

| Option | Meaning |
|---|---|
| `--numThread N` | inference threads (default 4) |
| `--padding N` | add a white border around the image (helps when text boxes are clipped) |
| `--maxSideLen N` | scale the long side down to N (e.g. 1024) before detection; `0` = no scaling |
| `--boxScoreThresh F` | text-box confidence threshold (lower it if boxes miss text) |
| `--doAngle 1` | enable 180° detection (only for upside-down images) |

## Reading the output

For each detected line the exe prints:
```
textLine[N](the recognized text)
textScores[N]{... per-char confidence ...}
crnnTime[N](...ms)
```
then `=====End detect=====`, a total-time line, and finally a **clean plain-text block** of all recognized lines.

**To get the text:** either collect the content inside every `textLine[N]( ... )`, or take the plain-text block printed after `=====End detect=====`.

## Limitations

- **Windows x64 only.** Other platforms need a separate build (not bundled).
- Ships **PP-OCRv3** models (2022) — good accuracy, slightly behind PP-OCRv4. Models are interface-compatible and can be swapped for v4 if higher accuracy is needed.
- Text recognition only (no layout/table reconstruction). For layout-preserving document parsing, use a different engine.

## Benchmark (reference, same Windows x64 machine, CPU)

Test image `benchmark/sample.jpg` — **564×533, 93 KB, 12 lines of Chinese + English**:

![benchmark sample — 12 lines of Chinese + English](benchmark/sample.jpg)

| Metric | This skill (ocr-cpp, PP-OCRv3) |
|---|---|
| End-to-end wall time | **3.6 s** |
| Pure inference (FullDetectTime) | 3.28 s |
| Lines recognized | 12 |

Recognized text (excerpt — punctuation and English spacing preserved):
```
人生活的真实写照：善有善报，恶有恶报。
我们中国人有一句俗语说："种瓜得瓜，种豆得豆。"而这就是每个
every man's life: good begets good, and evil leads to evil.
We Chinese have a saying: "If a man plants melons, he will reap
```
On the same image the Python `ocr` skill (PP-OCRv4) takes **6.3 s** end-to-end — `ocr-cpp` is ~45% faster and on this sample also keeps English word spacing better.
