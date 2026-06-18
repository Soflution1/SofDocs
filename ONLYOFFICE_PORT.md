# OnlyOffice AGPL Port Tracker

Alto is moving from a clean-room office engine to an AGPL-compatible port that
can derive behavior and structure from ONLYOFFICE. This tracker is the audit
trail for source attribution and port status.

## Rules

- Every derived Rust module must retain AGPL attribution in its file header.
- Source references must point to the exact ONLYOFFICE file used as the port
  source or behavioral anchor.
- Branded ONLYOFFICE assets, logos, product names, icons, and UI artwork are
  excluded from the port.
- A module is not considered complete until parity tests or fixture notes are
  attached.

## Word / Docs

| ONLYOFFICE source | Alto Rust target | Status | Notes |
| --- | --- | --- | --- |
| `References/OnlyOffice/sdkjs/word/Editor/Document.js` | `crates/core/src/document/onlyoffice_model.rs` | Started | Document root, page constants, selection/search enums, conversion from current `Document`. |
| `References/OnlyOffice/sdkjs/word/Editor/Paragraph.js` | `crates/core/src/document/onlyoffice_paragraph.rs` | Started | Paragraph/run container and recalc result flags. |
| `References/OnlyOffice/sdkjs/word/Editor/Run.js` | `crates/core/src/document/onlyoffice_run.rs` | Started | Run style projection from the current Rust model. |
| `References/OnlyOffice/sdkjs/word/Editor/Serialize2.js` | `crates/core/src/document/onlyoffice_serialize.rs` | Started | JSON serialization facade; binary `doct` parity is not implemented yet. |
| `References/OnlyOffice/core/OOXML/Binary/Document/DocWrapper/DocxSerializer.h` | `crates/core/src/document/onlyoffice_serialize.rs` | Planned | Future `docx <-> doct_bin` compatible bridge. |

## Sheets

| ONLYOFFICE source | Alto Rust target | Status | Notes |
| --- | --- | --- | --- |
| `References/OnlyOffice/sdkjs/cell/model/Workbook.js` | `crates/core/src/spreadsheet/mod.rs` | Started | Workbook, worksheet, cells, defined names, base metrics. |
| `References/OnlyOffice/sdkjs/cell/model/FormulaObjects/parserFormula.js` | `crates/core/src/spreadsheet/mod.rs` | Planned | Formula AST/evaluator not ported yet. |
| `References/OnlyOffice/sdkjs/cell/view/WorkbookView.js` | `crates/core/src/spreadsheet/mod.rs` | Planned | View state and grid metrics only sketched. |

## Slides

| ONLYOFFICE source | Alto Rust target | Status | Notes |
| --- | --- | --- | --- |
| `References/OnlyOffice/sdkjs/slide/Editor/Format/Presentation.js` | `crates/core/src/presentation/mod.rs` | Started | Presentation root, slides, selected-content shape. |
| `References/OnlyOffice/sdkjs/slide/Editor/Format/Slide.js` | `crates/core/src/presentation/mod.rs` | Started | Slide and basic shape records. |

## PDF

| ONLYOFFICE source | Alto Rust target | Status | Notes |
| --- | --- | --- | --- |
| `References/OnlyOffice/sdkjs/pdf/src/viewer.js` | `crates/core/src/pdf/mod.rs` | Started | Viewer state and page model. |
| `References/OnlyOffice/sdkjs/pdf/src/document.js` | `crates/core/src/pdf/mod.rs` | Started | Document metadata/page placeholders. |
