---
name: ocr-cpp
description: DEFAULT OCR skill on Windows x64 — native C++ engine (RapidOcrDll.dll + OcrTest.exe CLI driver: ONNX Runtime + PP-OCRv3, statically linked, NO Python, fast startup, zero dependencies). Use this BY DEFAULT to OCR / recognize text from images (jpg/png/webp/bmp/tiff), screenshots, scanned docs, photos of text; Chinese/English/multilingual. Use this UNLESS the user needs cross-platform (macOS/Linux) or is not on Windows x64 (then use the 'ocr' Python skill). Triggers on "OCR", "文字识别", "识别图片", "提取文字", "图片转文字", "图片识字", "扫描件转文本", "scan to text", "image to text".
---

# OCR Skill (Native C++ DLL Engine, No Python)

Recognizes text from images with a **native C++ engine** (`bin\RapidOcrDll.dll` — ONNX Runtime 1.15.1 + OpenCV 4.8.1 + PaddleOCR PP-OCRv3, statically linked /MT), driven by the small CLI wrapper `bin\OcrTest.exe`. **No Python, no dependencies to install, no GPU required** (runs on CPU). Chinese + English + multilingual.

Compared with the previous single-exe build: same models, **identical recognized text**, ~5× faster end-to-end (rebuilt with ONNX Runtime 1.15.1 + OpenCV 4.8.1, /MT, VS2022 toolchain), and images on any drive letter (e.g. `E:\...`) are handled correctly.

> The sibling skill `ocr` uses Python + RapidOCR. This `ocr-cpp` skill needs **nothing installed** on the target machine. Prefer this one when you want zero-dependency native OCR on Windows.

## Requirements

- **Windows x64 only** (64-bit binaries; will not run on macOS/Linux/ARM).
- **VC++ Redistributable** present (most dev machines have it). If you see "`VCRUNTIME140_1.dll` missing", install the "Microsoft Visual C++ Redistributable 2015-2022 (x64)".
- `bin\OcrTest.exe` and `bin\RapidOcrDll.dll` **must stay side by side in `bin\`**; the DLL auto-loads models from `<dll-dir>\..\models`, which is exactly this skill's layout.

## Files in this skill

```
bin/OcrTest.exe        # CLI driver that hosts the DLL (~244 KB)
bin/RapidOcrDll.dll    # OCR engine: ONNX Runtime + OpenCV, statically linked /MT (~17 MB)
models/
  ch_PP-OCRv3_det_infer.onnx           # text detection model
  ch_ppocr_mobile_v2.0_cls_infer.onnx  # orientation classifier
  ch_PP-OCRv3_rec_infer.onnx           # text recognition model
  ppocr_keys_v1.txt                    # character dictionary
```

## How to run

Works from **any working directory** (models resolve from the DLL's own location, not the CWD; image paths may be relative or absolute):

```bat
C:\Users\<user>\.claude\skills\ocr-cpp\bin\OcrTest.exe --image "<path-to-image>"
```

| Mode | Command | Use when |
|---|---|---|
| Single image | `--image <path>` | default, one or a few images |
| Batch | `--dir <dir>` | many images in one directory — model loaded **once**, ~0.4 s/image |

Notes:

- `--models <dir>` overrides the default model directory if you relocate models.
- `--dir` appends a run log under `<skill-root>\logs\` (git-ignored) — harmless and safe to delete.

### Engine tuning

`OcrTest.exe` runs the engine with proven defaults and exposes **no tuning flags** (`--padding`, `--maxSideLen`, `--boxScoreThresh`, `--doAngle`, `--numThread` are fixed inside the driver). If you ever need custom engine parameters, write a small driver against `bin\RapidOcrDll.dll` (Init once + Recognize loop) or use the upstream RapidOcrOnnx project.

## Reading the output

Exit code 0 on success. For each recognized line the driver prints:

```
[INIT] ok
[OK ] <image-path> lines=12 elapsedMs=384.8
     #0 score=0.9603 box=(6,0)(518,3)(518,27)(5,24) text=If the past has taught us anything, it is that ...
     #1 score=0.9363 box=... text=...
total keys size(6625)
```

**To get the text:** take everything after `text=` on each `#N` line, in printed order (top-to-bottom). `score` = line confidence; `box` = four corner points of the text box. There is no trailing plain-text block — the `#N` lines are the result.

## Limitations

- **Windows x64 only.** Other platforms need a separate build (use the `ocr` Python skill there).
- Ships **PP-OCRv3** models (2022) — good accuracy, slightly behind PP-OCRv4. Models are interface-compatible and can be swapped for v4 if higher accuracy is needed.
- Text recognition only (no layout/table reconstruction). For layout-preserving document parsing, use a different engine.
- The DLL driver exposes no engine tuning flags (proven defaults; see *Engine tuning*).

## Benchmark (reference, same Windows x64 machine, CPU)

Test image `benchmark/sample.jpg` — **564×533, 93 KB, 12 lines of Chinese + English**:

![benchmark sample — 12 lines of Chinese + English](benchmark/sample.jpg)

| Metric | DLL engine (current) | old single-exe build | Python `ocr` skill |
|---|---|---|---|
| End-to-end wall time (one image) | **~0.9 s** | 3.6 s | 6.3 s |
| Pure inference | ~0.38 s | 3.28 s | 5.48 s |
| Batch (`--dir`, model loaded once) | ~0.4 s/image | n/a (reloads per image) | — |
| Lines recognized | 12 | 12 | 12 |

Verified: the 12 recognized lines are **byte-identical** to the old build's reference (`benchmark/sample.jpg-result.txt`), punctuation and English spacing preserved.

On the same image the Python `ocr` skill (PP-OCRv4) takes **6.3 s** end-to-end — `ocr-cpp` with the DLL engine is ~7× faster.

---

版权声明：作者：梅文海
