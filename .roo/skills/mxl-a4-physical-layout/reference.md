# Research reference — physical A4 layout for 1C MXL/VPF

Purpose: source-backed notes for `mxl-a4-physical-layout`. This document separates what 1C officially documents from empirical/community findings and from Teplovik project policy.

## 1. OFFICIAL 1C — what is safe to treat as platform semantics

### 1.1 Print margins and page settings are physical

1C Standard Subsystems Library documentation describes SpreadsheetDocument print parameters:
- TopMargin — millimeters;
- LeftMargin — millimeters;
- BottomMargin — millimeters;
- RightMargin — millimeters;
- orientation;
- FitToPage;
- PrintScale in percent.

The same SSL guide gives a concrete print-template example: add a picture of **40 mm width** for signature/seal output. This confirms that physical-size reasoning is normal in 1C print-form design.

Source:
https://kb.1ci.com/1C_Enterprise_Platform/Guides/Developer_Guides/1C_Enterprise_Standard_Subsystems_Library_Developer_Guide/Chapter_3._Setting_and_using_subsystems_upon_configuration_development/Chapter_3._Setting_and_using_subsystems_upon_configuration_development/

### 1.2 SpreadsheetDocument page size is physical; margins are mm

1C:Шина / 1C platform library documentation describes SpreadsheetDocument properties:
- `ПолеСверху`, `ПолеСлева`, `ПолеСнизу`, `ПолеСправа` — millimeters;
- `РазмерСтраницы` — physical print page size;
- `ОриентацияСтраницы` — print orientation.

Source:
https://1cmycloud.com/console/help/esb/6.1/docs/stdlib/esb/Std/Spreadsheet/Spreadsheet_ru/

### 1.3 Millimeter page coordinates are a native 1C concept

Official PDF documentation for `PDFRepresentationObjectDescription` defines:
- `Left`, `Top` as millimeter coordinates from page top-left;
- `Width`, `Height` as millimeters.

This is **not** evidence that MXL cell width itself is measured in mm. It is evidence that page-relative physical geometry is a first-class concept in 1C and is suitable as an intermediate design coordinate system.

Source:
https://kb.1ci.com/1C_Enterprise_Platform/Guides/Developer_Guides/1C_Enterprise_8.3.23_Developer_Guide/Chapter_16._Operations_with_various_data_formats/16.4._PDF_format/16.4.3._Digital_signature/

### 1.4 Spreadsheet complexity matters

Official spreadsheet guidance recommends avoiding excessive numbers of columns/merged cells where possible and notes that different formats/export contexts can affect appearance. A physical grid should therefore be systematic, not an excuse to create uncontrolled complexity.

Source:
https://kb.1ci.com/1C_Enterprise_Platform/FAQ/Development/Spreadsheets/Spreadsheet_documents/

### 1.5 Pictures and DPI

Official platform documentation exposes picture width/height and horizontal/vertical density (DPI). If a format lacks density metadata, the platform may use a default density value. Therefore DPI must be treated as image metadata, not automatically as the coordinate system of the A4 page reference.

Source:
https://kb.1ci.com/1C_Enterprise_Platform/Guides/Developer_Guides/1C_Enterprise_8.3.23_Developer_Guide/Chapter_21._Temporary_storage_functionality__handling_files_and_pictures/21.4._Picture_management/21.4.1._Picture_parameters_and_conversion_operations/

## 2. EMPIRICAL RESEARCH — MXL units measured in real print

### 2.1 Native-api / Infostart, 27.02.2025

Article: `Точные значения единиц измерения размера ячеек в табличном документе в мм`
https://infostart.ru/1c/articles/2290559/

The author starts from the Syntax Assistant semantics:
- row height is described as points;
- column width is described as units of an average font character.

The author then prints a large bordered cell **without scaling**, measures the physical result, and derives approximately:
- horizontal: `(279.5 ± 0.5) mm / 151 = (1.851 ± 0.003) mm/unit`;
- vertical: `(200 ± 0.5) mm / 567 = (0.3527 ± 0.0008) mm/unit`.

The article then identifies the vertical value with the Adobe/PostScript typographic point and summarizes:
- vertical: `1 = 0.3528 mm`;
- horizontal: `1 = 5.25 pt ≈ 1.8522 mm`.

Important classification:
- `0.3528 mm` is consistent with the standard typographic point and the documented word “points” for row height;
- `1.8522 mm` for horizontal MXL width is an empirical finding / inferred historical unit, not an official API guarantee;
- both are extremely useful for initial calculation, but a target environment can still introduce scaling, clipping, font and renderer behavior. Real runtime PDF remains authoritative.

## 3. EMPIRICAL DESIGN METHOD — uniform MXL grid

### 3.1 DanilKrav4 / Infostart

Article: `Универсальный размер ячейки макета ПФ`
https://infostart.ru/1c/articles/2315214/

Problem described by the author: arbitrary row heights, column widths and row formats make tiny visual edits require rebuilding large parts of a print form.

Proposed method:
- build a template from homogeneous small cells;
- make wider/higher logical areas by combining cells instead of introducing arbitrary sizes everywhere;
- derive the proportions using the empirical unit conversion above.

Practical variants reported:

### Grid A — practical
- column width: `2` MXL units;
- row height: `10.5` units;
- approximately `80 × 55` cells for A4;
- author reports this is practical for forms mainly using fonts around 6–9 pt.

Because horizontal unit is about 5.25 times the vertical unit, `2 × 1.8522 ≈ 3.7044 mm` and `10.5 × 0.3528 ≈ 3.7044 mm`, producing roughly square physical cells.

### Grid B — finer
- column width: `1`;
- row height: `5.25`;
- approximately `160 × 110` cells on A4.

The author reports that after switching to the uniform-grid approach:
- print boundaries moved little or not at all during migration;
- later visual edits became much faster;
- cut-line layouts became easier;
- fewer repeated printer checks were needed;
- fewer custom row formats were necessary.

Project interpretation: use these grids as a reusable **calibrated coordinate lattice**, not as a mandate that every print form must have exactly 80×55 or 160×110 cells.

## 4. COMMUNITY PRACTICE — image-driven print form

Example publication:
`Разработка печатной формы на основе картинки`
https://infostart.ru/1c/reports/621687/

Known technique: prepare a designer image at A4 proportions, use it as a visual/background template in a SpreadsheetDocument, and position live information over it.

Benefits:
- preserves complex designer geometry;
- useful for certificates, forms, tickets, branded static layouts.

Risks:
- large background images increase size/memory;
- searchable/selectable text can be lost if everything is rasterized;
- dynamic tables, item counts, prices and variable-height text do not fit a fully flattened-image architecture well;
- page breaking and multi-page output become harder.

Teplovik policy: a background/full-page picture is acceptable as a reference technique or for genuinely static artwork. Commercial proposal business data must remain native 1C text/table structures; do not fake item rows or render business facts into the background image.

## 5. COMMUNITY PRACTICE — pixels/DPI → millimeters

Example publication:
`Вставка печати и подписи в табличный документ с учетом полей печати`
https://infostart.ru/1c/reports/414750/

The technique derives physical image size from pixel dimensions and image resolution:

```text
width_mm  = width_px  / horizontal_DPI * 25.4
height_mm = height_px / vertical_DPI   * 25.4
```

Then the picture is positioned/scaled considering the printable page area/margins while preserving proportions.

This is correct when the task is: “What physical size should this raster image have given its own DPI?”

It is **not automatically the same problem** as converting a screenshot of an A4 page to page coordinates. For a known full-page A4 raster, page-derived scale is normally clearer:

```text
sx = 210/Wpx
sy = 297/Hpx
```

DPI and page-derived systems may coincide, but do not assume that without checking.

## 6. Recommended canonical model: IMAGE/SVG → PHYSICAL A4 → MXL

### 6.1 Why millimeters are the best intermediate representation

Pixel coordinates depend on raster resolution. MXL horizontal cell units are abstract. Millimeters represent the real target: the physical printed page.

Therefore use:

```text
DESIGN PX       PHYSICAL PAGE         MXL/RUNTIME
(x,y,w,h)  ->   (x,y,w,h in mm)  ->   rows/columns/drawings
                                           |
                                           v
                                     real PDF in 1C
                                           |
                                           v
                                    measured geometry
```

This lets the design be rerendered at another resolution without changing its physical geometry contract.

### 6.2 A4 formulas

For full-page A4 portrait raster `Wpx × Hpx`:

```text
paper_w_mm = 210
paper_h_mm = 297
sx = paper_w_mm / Wpx
sy = paper_h_mm / Hpx

x_mm = x_px * sx
y_mm = y_px * sy
w_mm = w_px * sx
h_mm = h_px * sy
```

For landscape swap physical width/height.

Before using formulas verify:
1. image really represents the entire physical paper;
2. no viewer screenshot chrome/padding is included;
3. no crop has removed page edges;
4. aspect ratio is consistent with selected paper format within declared tolerance.

### 6.3 Worked Teplovik example — 1191×1684 raster

If `1191×1684` is the complete A4 page:

```text
sx = 210 / 1191 ≈ 0.176322 mm/px
sy = 297 / 1684 ≈ 0.176366 mm/px
```

Examples:
- `x=33 px` → about `5.82 mm`;
- `y=35 px` → about `6.17 mm`;
- `x=953 px` → about `168.04 mm`;
- `w=513 px` → about `90.45 mm`.

A 7 mm physical margin corresponds to approximately:

```text
7 / 0.176322 ≈ 39.7 px horizontally
7 / 0.176366 ≈ 39.7 px vertically
```

Therefore an exact anchor at 33–35 px from the physical page edge conflicts with an exact 7 mm page margin if both use the same full-page coordinate frame. The implementation must not hide that conflict by inventing MXL offsets. First decide which contract is authoritative or identify that the raster coordinates were measured from a different frame.

## 7. Physical geometry map

Store a map before editing MXL. Example schema:

```json
{
  "paper": {"format":"A4","orientation":"portrait","width_mm":210,"height_mm":297},
  "reference": {"width_px":1191,"height_px":1684,"frame":"full_page"},
  "print": {"left_mm":7,"right_mm":7,"top_mm":7,"bottom_mm":8,"scale_pct":100,"fit_to_page":false},
  "anchors": {
    "logo": {"x_mm":0,"y_mm":0,"w_mm":0,"h_mm":0,"clip":"none"},
    "slogan": {"x_mm":0,"y_mm":0,"w_mm":0,"h_mm":0,"clip":"right_page_edge"}
  }
}
```

Use meaningful anchors rather than only a global bounding box. A global bbox can match while individual items are badly misplaced.

## 8. MXL implementation strategy

### 8.1 Vertical

Row height is suitable for point/mm conversion:

```text
1 pt = 25.4/72 = 0.352777... mm
height_pt = height_mm * 72/25.4
```

Rounding and actual text metrics still require runtime validation.

### 8.2 Horizontal

Do not write code based on `column_width = desired_mm`.

Possible approach:

```text
initial_column_units = desired_mm / 1.8522
```

Use it only as the initial calibrated estimate. Then render and measure actual geometry. If the environment shows a stable different scale, store the calibration explicitly rather than scattering magic numbers through Template.xml.

### 8.3 Drawings/images

Where the SpreadsheetDocument drawing API exposes physical position/size, use explicit geometry and preserve aspect ratio as intended. If a drawing remains cell-relative in Template.xml, derive its cell/offset placement from the physical map and then calibrate via runtime PDF.

### 8.4 Dynamic table

Keep:
- real item rows;
- article/name/qty/unit/price/sum cells;
- real pagination;
- real totals;
- named areas and BSP print flow.

Do not manufacture rows to make a screenshot look full.

## 9. Closed-loop calibration is mandatory

Even a correct physical map is not sufficient by itself because final rendering can be affected by:
- horizontal overflow causing page scaling;
- FitToPage;
- automatic row height;
- font substitution;
- font metrics/wrapping;
- proportional picture scaling;
- PDF rasterization settings;
- platform/runtime differences;
- clipping at printable/page boundaries.

Required cycle:

```text
measure SoT
→ calculate mm
→ implement MXL
→ load into test IB
→ generate REAL 1C PDF
→ rasterize at fixed declared scale
→ detect anchors
→ compare X/Y/W/H/clip
→ fix causal geometry
→ repeat
```

## 10. Visual comparator requirements

A valid comparator must not only check vertical position or one broad content bbox.

For each required anchor emit at least:
- `dx`;
- `dy`;
- `dw`;
- `dh`;
- edge/clip metrics where applicable.

Also include a residual/perceptual image comparison that tolerates antialiasing noise but does not automatically translate/register the runtime image onto the SoT, because doing so can conceal layout shifts.

Regression contract:
- canonical vs canonical → PASS;
- known-bad shifted/scaled/clipped examples → FAIL;
- missing/MANUAL required visual criteria → BLOCKED, never overall PASS.

## 11. Source classification rule

Every implementation note should distinguish:

**OFFICIAL** — documented by 1C; can be treated as platform semantics within the documented scope.

**EMPIRICAL** — measured/reported by practitioners; useful for engineering calculations but must be validated on our runtime.

**PROJECT POLICY** — Teplovik acceptance architecture (SoT, runtime artifact, visual gate, Owner handoff).

Do not collapse these categories into one “1C standard”.

## 12. Final recommendation

For designer VPF/MXL work use millimeters as the canonical intermediate geometry layer:

**A4 SoT pixels → millimeters → calibrated MXL grid/drawings → real 1C PDF → machine visual gate.**

This is more deterministic than iterative eyeballing, while retaining the real 1C table/text/business-data model. The runtime loop remains essential because MXL horizontal units and rendering behavior are not a pure millimeter-coordinate canvas.