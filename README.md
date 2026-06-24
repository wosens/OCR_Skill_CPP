# OCR Skill — Native C++ (No Python) for Claude Code

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that recognizes text from images using a **self-contained native C++ executable** — **no Python, no pip, no extra DLLs to install**. Windows x64 only. Chinese + English + multilingual.

Powered by [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) (ONNX Runtime + PaddleOCR PP-OCRv3, statically linked into a single exe).

> Sibling skill: [`OCR_Skill_Python`](https://github.com/wosens/OCR_Skill_Python) uses Python + RapidOCR (cross-platform). **This** one is zero-dependency native on Windows — faster startup, nothing to install.

## Requirements

- **Windows x64** (64-bit Windows binary; will not run on macOS/Linux/ARM).
- **VC++ Redistributable** (most dev machines already have it). If you see `VCRUNTIME140_1.dll missing`, install the Microsoft Visual C++ Redistributable 2015-2022 (x64).
- No Python, no GPU required (runs on CPU).

## Install

```bat
git clone https://github.com/wosens/OCR_Skill_CPP.git "%USERPROFILE%\.claude\skills\ocr-cpp"
```

That's it — the repo already bundles the exe and models. **No install/build step.**

## Usage

Ask Claude Code in natural language ("OCR this image"), or run the exe directly:

```bat
cd %USERPROFILE%\.claude\skills\ocr-cpp
bin\RapidOcrOnnx.exe --models models ^
  --det  ch_PP-OCRv3_det_infer.onnx ^
  --cls  ch_ppocr_mobile_v2.0_cls_infer.onnx ^
  --rec  ch_PP-OCRv3_rec_infer.onnx ^
  --keys ppocr_keys_v1.txt ^
  --image "your-image.jpg"
```

Key options: `--numThread N` · `--padding N` · `--maxSideLen N` · `--boxScoreThresh F` · `--doAngle 1` (for upside-down images).

**Output:** per line `textLine[N](text)` + per-char scores, then a clean plain-text block after `=====End detect=====`.

## Benchmark (same Windows x64 machine, CPU)

Test image `benchmark/sample.jpg` — 564×533, 93 KB, 12 lines of Chinese + English:

![benchmark sample](benchmark/sample.jpg)

| Metric | This skill (ocr-cpp) | Python skill (ocr) |
|---|---|---|
| End-to-end wall time | **3.6 s** | 6.3 s |
| Inference | 3.28 s | 5.48 s |
| English word spacing | preserved | dropped (RapidOCR behavior) |

`ocr-cpp` is ~45% faster end-to-end (no Python startup) and on this sample keeps English spacing better.

## Limitations

- Windows x64 only. For macOS/Linux use the Python sibling skill.
- Ships PP-OCRv3 models; can be swapped for v4 (interface-compatible) if higher accuracy is needed.
- Text recognition only (no layout/table reconstruction).

## License

MIT — see [LICENSE](LICENSE). The bundled exe and models originate from [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) (Apache-2.0); this repo only packages them for convenient skill distribution.

## Author

**wosens** — [https://github.com/wosens](https://github.com/wosens)

Issues/PRs welcome at [https://github.com/wosens/OCR_Skill_CPP/issues](https://github.com/wosens/OCR_Skill_CPP/issues).
