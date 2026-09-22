# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Vietnamese-language capstone report (đồ án tốt nghiệp, HCMUT — Computer Engineering) written in LaTeX. Subject: an AI assistant robot for elderly care and monitoring, built from two Android apps (RobotApp on a tablet, CaregiverApp on a phone), on-device YOLO-Pose + LSTM fall detection, Firebase RTDB, WebRTC, and ESP32/BLE firmware.

There is no application source code here — the report *describes* a system that lives elsewhere (the sibling `../do_an_v2` folder listed in `Capstone_Project_Report-.code-workspace`).

## Build

```bash
latexmk -xelatex -interaction=nonstopmode main.tex   # full build (resolves toc/lof/lot cross-refs)
latexmk -c                                            # clean aux files, keep main.pdf
```

**XeLaTeX is mandatory** — `main.tex` declares `% !TeX program = xelatex` and uses `fontspec` with system fonts (Times New Roman, Arial, Courier New) plus `\usepackage[vietnamese]{babel}`. pdflatex will fail.

There is no bibtex/biber step: the bibliography in `Outro/TaiLieuThamKhao.tex` is a hand-numbered `enumerate` list, and citations use the custom `\reportcite{ref:key}` macro resolving against `\label{ref:key}` inside that list.

### Verifying a change
`latexmk` exits 0 even with unresolved references, so check explicitly:

```bash
grep -nE "^! |LaTeX Error|Undefined control" main.log   # hard errors
grep -c "LaTeX Warning: Reference" main.log             # must be 0
```

To eyeball a specific page (the repo has ghostscript, but no pdftoppm/ImageMagick):

```bash
gs -q -dNOPAUSE -dBATCH -sDEVICE=jpeg -r80 -dFirstPage=N -dLastPage=N -sOutputFile=/tmp/pN.jpg main.pdf
```

Note: **printed page numbers ≠ PDF page numbers.** Front matter is numbered in roman, so the arabic page `N` cited in `main.toc`/`main.lof` sits at physical page `N + 22`.

## Structure

`main.tex` is preamble + title page only; all prose is `\input`. Order of inclusion:

- `Intro/` — LoiCamDoan, LoiCamOn, TomTat, DanhMucTuVietTat, DanhMucThuatNgu (unnumbered front matter)
- `Contents/` — the six numbered chapters: GioiThieu, CoSoLyThuyet, PhanTichThietKeHeThong, HienThucHeThong, KetQuaVaThaoLuan, KetLuan
- `Outro/` — TaiLieuThamKhao, PhuLuc

`Contents/OpenBotTomLuoc.tex` is not in `main.tex`; it is `\input` from inside `CoSoLyThuyet.tex`. `Outro/Image.tex` is empty and unused.

### Sectioning convention

This is `\documentclass{article}`, but **`\section` is retitled as "Chương N"** via `titlesec`. So inside `Contents/*.tex`, a chapter is `\section{...}` and its children are `\subsection`/`\subsubsection`. Do not introduce `\chapter`. Figures, tables and equations are numbered per section (`\counterwithin`), giving `4.3`, `Bảng 3.9`, etc.

Unnumbered front-matter headings use the custom `\frontsection{Tiêu đề}`, which handles `\clearpage`, TOC entry and running header in one call.

## Figures

`\graphicspath{{Images/}}` is set, so `\includegraphics{foo.png}` resolves inside `Images/` — pass the bare filename, not `Images/foo.png` (some older lines in `CoSoLyThuyet.tex` do the redundant thing; don't copy that).

`subcaption` is loaded. Multi-panel figures in `HienThucHeThong.tex` follow this shape — always `[t]` alignment, because `[b]` misaligns panels whose subcaptions wrap to a different number of lines:

```latex
\begin{subfigure}[t]{0.30\textwidth}
    \centering
    \includegraphics[width=\linewidth]{name.jpg}
    \caption{...}\label{fig:...}
\end{subfigure}
```

Working widths, given a ~16 cm text block:
- portrait phone screenshots (630×1400): `0.30\textwidth` for a row of three, `0.38` for two
- landscape tablet screenshots (1400×875): `0.72`–`0.80\textwidth`, **stacked vertically**, one per row — two side by side renders the on-screen overlay text too small to read in print

### Adding device screenshots

`device_recent_images_20260917_2032/` holds raw, opaquely named captures pulled off the devices. Don't reference it from LaTeX. Copy into `Images/` under a descriptive name and downscale first (raw tablet captures are ~1.7 MB each):

```bash
sips -Z 1400 -s formatOptions 82 "device_recent_images_.../Screenshot_....jpg" --out Images/robotapp_tracking_front.jpg
```

Existing UI screenshots use the `robotapp_*` / `caregiver_*` prefixes.

## Writing conventions

- Body text is Vietnamese. Captions are Vietnamese; code identifiers, Android/Firebase API names and protocol fields stay in English.
- Identifiers go in `\texttt{}`; identifiers containing underscores go in `\path|...|` to avoid escaping (e.g. `\path|robot_follow_enabled|`).
- Decimals use the Vietnamese comma form in prose (`0,15`, `20\,\mathrm{rad/s}`).
- Chapter 4 (HienThucHeThong) states *how* things are built; Chapter 5 (KetQuaVaThaoLuan) states *what was measured*. Chapter 5 deliberately distinguishes bằng chứng tĩnh / tự động / tích hợp and refuses to report numbers without an artifact behind them — keep that discipline when adding results, and don't invent metrics.

## Generated files

`main.pdf` and the aux files (`.aux .log .toc .lof .lot .out .fls .fdb_latexmk .xdv`) are committed and will show as modified after every build. That is expected in this repo; commit or discard them deliberately rather than treating the churn as a problem.
