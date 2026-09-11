# OCR Skill — Native C++ (No Python) for Claude Code

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that recognizes text from images with a **self-contained native C++ engine** — **no Python, no pip, nothing to install**. Windows x64 only. Chinese + English + multilingual.

Powered by the [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) engine (ONNX Runtime + PaddleOCR PP-OCRv3), rebuilt as `bin\RapidOcrDll.dll` (ONNX Runtime 1.15.1 + OpenCV 4.8.1, statically linked /MT, VS2022) and driven by the bundled CLI wrapper `bin\OcrTest.exe`. Recognized text is identical to the previous single-exe build, ~5× faster end-to-end.

> Sibling skill: [`OCR_Skill_Python`](https://github.com/wosens/OCR_Skill_Python) uses Python + RapidOCR (cross-platform). **This** one is zero-dependency native on Windows — faster startup, nothing to install.

## Requirements

- **Windows x64** (64-bit binaries; will not run on macOS/Linux/ARM).
- **VC++ Redistributable** (most dev machines already have it). If you see `VCRUNTIME140_1.dll missing`, install the Microsoft Visual C++ Redistributable 2015-2022 (x64).
- `bin\OcrTest.exe` and `bin\RapidOcrDll.dll` must stay side by side in `bin\` (the DLL resolves models from `bin\..\models`).
- No Python, no GPU required (runs on CPU).

## Install

```bat
git clone https://github.com/wosens/OCR_Skill_CPP.git "%USERPROFILE%\.claude\skills\ocr-cpp"
```

That's it — the repo already bundles the binaries and models. **No install/build step.**

## Usage

Ask Claude Code in natural language ("OCR this image"), or run the driver directly (image paths may be relative or absolute):

```bat
cd %USERPROFILE%\.claude\skills\ocr-cpp
bin\OcrTest.exe --image "your-image.jpg"
```

- **Batch:** `bin\OcrTest.exe --dir <folder>` recognizes every image in a folder with a single model load (~0.4 s/image). It appends a run log under `logs\` (git-ignored, safe to delete).
- **Tuning:** the driver uses proven defaults and exposes no tuning flags (`--numThread`, `--padding`, `--maxSideLen`, `--boxScoreThresh`, `--doAngle` are fixed). If you need custom engine parameters, write a small driver against `bin\RapidOcrDll.dll` or use the upstream [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) project.

**Output:** exit code 0 on success; per line `#N score=... box=... text=<recognized text>` after the `[OK ] <path> lines=N` header — the text is everything after `text=`.

## Benchmark (same Windows x64 machine, CPU)

Test image `benchmark/sample.jpg` — 564×533, 93 KB, 12 lines of Chinese + English:

![benchmark sample](benchmark/sample.jpg)

| Metric | This skill (DLL engine) | old exe build | Python skill (ocr) |
|---|---|---|---|
| End-to-end wall time | **~0.9 s** | 3.6 s | 6.3 s |
| Inference | ~0.38 s | 3.28 s | 5.48 s |
| Batch (one model load) | ~0.4 s/image | — | — |
| English word spacing | preserved | preserved | dropped (RapidOCR behavior) |

Recognized text is byte-identical to the old build (verified line-by-line on the benchmark image); ~7× faster than the Python skill end-to-end.

## Limitations

- Windows x64 only. For macOS/Linux use the Python sibling skill.
- Ships PP-OCRv3 models; can be swapped for v4 (interface-compatible) if higher accuracy is needed.
- Text recognition only (no layout/table reconstruction).
- The DLL driver exposes no engine tuning flags (proven defaults).

## License

MIT — see [LICENSE](LICENSE). The bundled engine and models originate from [RapidOcrOnnx](https://github.com/RapidAI/RapidOcrOnnx) (Apache-2.0); this repo only packages them for convenient skill distribution.

## Author

**wosens** — [https://github.com/wosens](https://github.com/wosens)

Issues/PRs welcome at [https://github.com/wosens/OCR_Skill_CPP/issues](https://github.com/wosens/OCR_Skill_CPP/issues).

2026.9.12