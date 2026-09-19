---
name: image-to-layout-spec
description: Convert a reference page image plus declared physical page format (for example A4 portrait) into a deterministic layout contract for exact print-form/MXL reproduction, extracting only geometry, table structure, text style cues, colors, and comparison metrics.
---

# image-to-layout-spec

## Purpose

Turn a reference PNG/JPG plus a declared physical page size into a machine-readable layout specification for print forms and 1C MXL. The skill must extract only the data required for accurate reproduction and review. It must not copy entire external projects or introduce unrelated UI/code-generation stacks.

## Target outputs

Given a reference image and page format (for example A4 portrait = 210×297 mm), produce:

1. `LAYOUT_SPEC.json`
2. `LAYOUT_SPEC.md`
3. `overlay.png` with numbered boxes and measurements
4. Optional `COMPARE_REPORT.json` / `COMPARE_REPORT.md` for reference vs generated page

## Required extracted data

- Page geometry and margins.
- Major visual blocks with `x/y/w/h` in px and mm.
- Table geometry: table bounds, columns, column widths, header height, row heights, cell bounds where deterministically detectable.
- Text geometry and style cues only: bbox, approximate font size, weight class, alignment, wrapping, text color. Do not dump unstructured OCR text as a primary output.
- Fill colors, line colors, borders, separators.
- Image/icon/logo bounds.
- Reference-vs-generated deltas in px, mm and percent.

## Coordinate conversion

When the physical page size is declared, pixel-to-mm conversion is deterministic:

`x_mm = x_px / image_width_px * page_width_mm`

`y_mm = y_px / image_height_px * page_height_mm`

Equivalent formulas apply to width and height. Do not let a vision model guess physical scale when page dimensions are known.

## Scope control

Do NOT import or vendor entire third-party repositories.
Do NOT add React/HTML code generation.
Do NOT add general-purpose UI screenshot parsing unrelated to print documents.
Do NOT make heavyweight ML models mandatory for the base path.
Do NOT directly generate MXL without the intermediate layout contract.

## Architecture

### Phase 1 — deterministic core

Implement without heavyweight ML:
- page geometry;
- px↔mm conversion;
- line/border detection;
- dominant fill/color detection;
- rectangular block detection;
- table/grid candidate detection;
- overlay generation;
- reference-vs-generated geometric comparison.

### Phase 2 — optional OCR/layout backend

Add an optional backend informed by PaddleOCR PP-Structure / LayoutParser concepts for text and document-region detection. Keep dependency optional and isolated.

### Phase 3 — optional table backend

Add an optional backend informed by Microsoft Table Transformer concepts for hard table cases. Keep dependency optional and isolated.

## External sources policy

Study only the targeted concepts needed for this skill from:
- PaddlePaddle/PaddleOCR — PP-Structure layout and bbox concepts;
- microsoft/table-transformer — table/cell structure concepts;
- Layout-Parser/layout-parser — document layout abstractions;
- abi/screenshot-to-code — image-to-layout workflow concepts only.

Record source URLs, license notes, and exactly which concepts were adopted in `reference/SOURCES.md`. Do not copy unrelated code or assets.

## Acceptance

The base skill is acceptable only when a test image with declared A4 dimensions produces:
- deterministic px→mm geometry;
- valid JSON + readable MD;
- overlay image;
- table/block measurements where present;
- compare report against a second image;
- tests for conversion math and output schema.

The skill must remain reusable and independent of `agent-settlements-1c`.
