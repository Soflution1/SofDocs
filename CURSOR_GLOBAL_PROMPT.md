# SofDocs: Global Cursor Prompt
# Project: Open-source office suite (Google Docs / Microsoft 365 competitor)
# Owner: SOFLUTION LTD / Antoine Pinelli
# License: AGPL v3 (forked from OnlyOffice, full rewrite)
# Reference repos: /Users/antoinepinelli/Cursor/App/OnlyOffice-* (14 repos, read-only reference)

## IDENTITY

You are building **SofDocs**, a full office suite (documents, spreadsheets, presentations, PDF) that will compete with Google Docs and Microsoft 365. This is a complete rewrite from scratch. The OnlyOffice forks in ../OnlyOffice-* are kept as functional reference only to understand document format specifications and editor behavior. Do NOT copy OnlyOffice code. Do NOT use any C++ or JavaScript from the reference repos. Everything is written from scratch.

## TARGET STACK

### Frontend UI: SvelteKit + Svelte 5 + Tailwind CSS 4
- All UI components (toolbar, sidebar, panels, modals, menus, ribbons) in Svelte 5 runes syntax
- $state(), $derived(), $effect() exclusively, NO legacy reactive stores
- Tailwind CSS 4 for all styling, NO separate CSS files
- SvelteKit for routing, SSR, API routes
- shadcn-svelte for base UI components (buttons, dialogs, dropdowns, tooltips, tabs)
- The frontend is IDENTICAL between web (SaaS) and desktop (Tauri webview)
- The frontend communicates with the Rust/WASM engine via typed bindings

### Engine: Rust compiled to WebAssembly (WASM)
- Rust crate compiled via wasm-pack with wasm-bindgen for JS interop
- The WASM module handles ALL heavy operations:
  - Document parsing (OOXML: .docx, .xlsx, .pptx)
  - Document rendering to HTML5 Canvas (text layout, pagination, cell grid, slide rendering)
  - Format conversion (docx <-> pdf, xlsx <-> csv, pptx <-> pdf, odt <-> docx, etc.)
  - Undo/redo command stack (in Rust memory, not JS)
  - Spell checking engine
  - Search and replace across documents
  - Formula engine for spreadsheets
  - PDF parsing, rendering, and form filling
- Key Rust crates to use:
  - `quick-xml` for XML parsing (OOXML is XML-based)
  - `zip` for OOXML container handling (.docx/.xlsx/.pptx are ZIP archives)
  - `wasm-bindgen` + `js-sys` + `web-sys` for browser interop
  - `serde` + `serde_json` for serialization
  - `tiny-skia` for 2D software rendering (CPU baseline)
  - `wgpu` for GPU-accelerated rendering when WebGPU is available
  - `lopdf` or custom PDF engine for PDF read/write
  - `icu4x` for internationalization and text shaping
  - `rustybuzz` for text shaping (HarfBuzz port)
  - `ttf-parser` + `ab_glyph` for font handling
  - `image` crate for image processing (embedded images in docs)
  - `comemo` for incremental computation / memoization
- Svelte communicates with WASM via typed bindings (wasm-bindgen)
- The canvas HTML element is owned by Svelte, pixel rendering is done by WASM

### Server: Axum (Rust)
- Axum web framework for HTTP API and WebSocket
- Real-time collaboration via WebSocket (OT or CRDT-based)
- File conversion service (calls the same Rust engine, no separate process)
- Authentication via Better Auth (not Supabase Auth)
- Database: Supabase (PostgreSQL) for user data, document metadata
- Storage: Supabase Storage or S3-compatible for document files
- Rate limiting, CORS, compression built-in
- Horizontal scaling ready (stateless API, shared state via Redis/PostgreSQL)

### Desktop App: Tauri v2 (Rust)
- Tauri v2 for native desktop app (macOS, Windows, Linux)
- The Svelte frontend runs in the native webview (WebKit on macOS, WebView2 on Windows)
- Rust backend in Tauri handles:
  - Local file system access (open/save documents)
  - Native OS dialogs (file picker, print dialog)
  - System tray, notifications, auto-update
  - Offline mode (full editing without internet)
  - Local document conversion (no server needed)
- The WASM engine runs inside the webview, same as web
- Tauri commands (invoke) bridge Svelte <-> Rust for native features
- App size target: under 30 MB (vs 500+ MB for OnlyOffice/Electron)
- RAM target: under 100 MB at idle (vs 300+ MB for OnlyOffice)

### Mobile (Future): Tauri v2 Mobile
- Tauri v2 mobile support (iOS, Android) when stable
- Same Svelte frontend + Rust/WASM engine
- Fallback: Capacitor wrapper if Tauri mobile is not ready

## PROJECT STRUCTURE

```
SofDocs/
  apps/
    web/                    # SvelteKit app (SaaS frontend)
      src/
        lib/
          components/       # Svelte 5 UI components
            toolbar/        # Document toolbar, ribbon
            sidebar/        # Side panels (styles, properties)
            editor/         # Editor canvas wrapper
            sheets/         # Spreadsheet grid UI
            slides/         # Presentation slide UI
            pdf/            # PDF viewer UI
          stores/           # Svelte stores for app state
          wasm/             # WASM bindings and loader
        routes/             # SvelteKit routes
          /                 # Landing page
          /app/             # Main editor app
          /app/document/    # Document editor
          /app/spreadsheet/ # Spreadsheet editor
          /app/presentation/# Presentation editor
          /app/pdf/         # PDF viewer/editor
      static/
      svelte.config.js
      tailwind.config.ts
      vite.config.ts
    desktop/                # Tauri v2 app
      src-tauri/
        src/
          main.rs           # Tauri entry point
          commands.rs       # Tauri commands (file I/O, native dialogs)
          menu.rs           # Native menu bar
        tauri.conf.json
        Cargo.toml
  crates/
    sofdocs-core/           # Core engine (Rust, compiles to WASM + native)
      src/
        lib.rs
        document/           # DOCX parser/renderer
          parser.rs         # OOXML document parsing
          renderer.rs       # Document layout and rendering
          model.rs          # Document object model
        spreadsheet/        # XLSX parser/renderer
          parser.rs
          renderer.rs
          model.rs
          formulas.rs       # Formula engine (SUM, VLOOKUP, etc.)
        presentation/       # PPTX parser/renderer
          parser.rs
          renderer.rs
          model.rs
        pdf/                # PDF engine
          parser.rs
          renderer.rs
          forms.rs
        converter/          # Format conversion
          mod.rs
          docx_to_pdf.rs
          xlsx_to_csv.rs
          odt_to_docx.rs
        canvas/             # Canvas rendering abstraction
          mod.rs
          cpu.rs            # tiny-skia CPU renderer
          gpu.rs            # wgpu GPU renderer
        text/               # Text layout engine
          shaping.rs        # Text shaping (rustybuzz)
          layout.rs         # Line breaking, paragraph layout
          fonts.rs          # Font loading and management
        collab/             # Collaboration primitives
          crdt.rs           # CRDT for real-time sync
          operations.rs     # Operational transforms
      Cargo.toml
    sofdocs-wasm/           # WASM bindings (thin wrapper around core)
      src/
        lib.rs              # wasm-bindgen exports
      Cargo.toml
    sofdocs-server/         # Axum server
      src/
        main.rs
        routes/
          documents.rs      # Document CRUD API
          convert.rs        # File conversion API
          collab.rs         # WebSocket collaboration
          auth.rs           # Authentication routes
        middleware/
        config.rs
      Cargo.toml
  Cargo.toml                # Workspace root
  package.json              # Frontend dependencies
  README.md
```

## PHASE 1: MVP (Start Here)

Build a minimal but functional document editor as proof of concept.

### Step 1: Rust Workspace Setup
1. Initialize Cargo workspace at project root
2. Create crates: sofdocs-core, sofdocs-wasm, sofdocs-server
3. Configure wasm-pack for sofdocs-wasm
4. Add all Rust dependencies listed above
5. Verify `cargo build` and `wasm-pack build` work

### Step 2: DOCX Parser (Rust)
1. Parse .docx files (they are ZIP archives containing XML)
2. Extract document.xml, styles.xml, [Content_Types].xml
3. Build a Rust document model (paragraphs, runs, text, styles, images)
4. Support: headings, bold, italic, underline, font size, font family, colors
5. Support: lists (ordered, unordered), tables, images
6. Reference: read ../OnlyOffice-core/OOXML/DocxFormat/ to understand the OOXML spec
7. Reference: read ../OnlyOffice-core/X2tConverter/src/lib/docx.h for conversion logic

### Step 3: Document Renderer (Rust/WASM)
1. Take the parsed document model and render to HTML5 Canvas
2. Implement text layout: line breaking, word wrap, paragraph spacing
3. Implement pagination (page breaks, headers, footers)
4. Use rustybuzz for text shaping, ttf-parser for fonts
5. Use tiny-skia for CPU rendering to canvas
6. Export the renderer via wasm-bindgen so Svelte can call it

### Step 4: SvelteKit Frontend
1. Create SvelteKit app with Svelte 5 + Tailwind CSS 4 + shadcn-svelte
2. Build document editor UI:
   - Toolbar: bold, italic, underline, font picker, size, color, alignment
   - Canvas area: renders the document via WASM
   - Sidebar: document properties, styles panel
3. Load WASM module on mount
4. File upload: user uploads .docx, WASM parses and renders it
5. Editing: user clicks on text, types, WASM updates the model and re-renders
6. Export: download as .docx (WASM serializes the model back to OOXML)

### Step 5: Axum Server
1. Basic Axum server with health check
2. File upload endpoint (accepts .docx, stores in Supabase Storage)
3. Document list endpoint (fetch user's documents from Supabase)
4. WebSocket stub for future collaboration
5. Auth via Better Auth

### Step 6: Tauri Desktop
1. Wrap the SvelteKit app in Tauri v2
2. Add file open/save commands (native file dialogs)
3. Add native menu bar (File, Edit, View, Format, Help)
4. Build for macOS (.dmg) and Windows (.msi)

## CODING RULES (MANDATORY)

### Rust Rules
- Use Rust 2021 edition minimum
- All public APIs must have doc comments (///)
- Use `thiserror` for error types, `anyhow` for application errors
- Use `tracing` for logging (not println! or log)
- All async code uses `tokio` runtime
- No `unwrap()` in production code, use `?` operator or explicit error handling
- No `unsafe` blocks unless absolutely necessary and documented why
- All WASM-exposed functions must be #[wasm_bindgen]
- Keep WASM binary size minimal: use `wasm-opt -Oz`, enable LTO, strip debug info
- Target WASM binary size: under 2 MB for initial load (lazy-load modules after)
- Use feature flags to compile different targets:
  - `wasm` feature: WASM-specific code (web-sys, wasm-bindgen)
  - `native` feature: native-specific code (file I/O, Tauri commands)
  - `server` feature: server-specific code (Axum, database)

### Svelte Rules
- Svelte 5 runes syntax ONLY: $state(), $derived(), $effect()
- NO legacy syntax: NO $:, NO let x; export let, NO writable/readable stores
- Component props use $props() destructuring
- Event handling: use callback props or Svelte 5 event syntax
- All components are TypeScript (.svelte with <script lang="ts">)
- Tailwind CSS 4 for ALL styling, no inline styles, no CSS modules
- shadcn-svelte components as base, customize with Tailwind
- NO em-dashes or en-dashes anywhere in UI text or code comments

### Architecture Rules
- The Svelte frontend NEVER directly manipulates document data
- ALL document operations go through the WASM engine
- The WASM engine is the single source of truth for document state
- Svelte only handles: user input events, UI state, calling WASM, rendering canvas
- The Axum server NEVER processes document content directly
- Server only handles: auth, file storage, collaboration relay, metadata
- Tauri only handles: native OS features (file dialogs, menus, system tray)
- ALL business logic lives in sofdocs-core (shared between WASM, server, desktop)

### Performance Targets
- Open a 100-page .docx: under 500ms
- Render first page: under 100ms
- Typing latency: under 16ms (60fps)
- Memory for a 100-page doc: under 50 MB
- WASM initial load: under 2 MB
- Desktop app cold start: under 2 seconds
- Server API response: under 50ms for metadata, under 2s for conversion

## SUPPORTED FORMATS (Full List)

### Read + Write (Full editing)
- DOCX (Office Open XML Document)
- XLSX (Office Open XML Spreadsheet)
- PPTX (Office Open XML Presentation)
- PDF (view, annotate, fill forms, export)
- ODF: ODT, ODS, ODP (OpenDocument formats)
- RTF (Rich Text Format)
- TXT, CSV, TSV (plain text)
- HTML (import/export)
- Markdown (.md)

### Read Only (View + convert to editable)
- DOC (legacy Word binary format)
- XLS (legacy Excel binary format)
- PPT (legacy PowerPoint binary format)
- EPUB (e-books)
- DjVu
- XPS
- FB2 (fiction books)

### Export Only
- PDF (from any document type)
- PNG, JPEG (page/slide as image)
- SVG (vector export)

## BRANDING

- Product name: **SofDocs**
- Company: **Soflution LTD**
- Primary color: #2563EB (blue-600)
- Accent color: #7C3AED (violet-600)
- Logo: to be designed (use text "SofDocs" as placeholder)
- Tagline: "Your documents. Your rules."
- All UI text in English by default, i18n ready (French, German, Italian, Spanish)

## REFERENCE MATERIAL

The OnlyOffice forks are available at:
- ../OnlyOffice-core/ (C++ core, format parsers, converters)
- ../OnlyOffice-sdkjs/ (JS editor engines: word/, cell/, slide/, pdf/)
- ../OnlyOffice-web-apps/ (React frontend)
- ../OnlyOffice-server/ (Node.js backend)
- ../OnlyOffice-desktop-sdk/ (Desktop SDK)
- ../OnlyOffice-DesktopEditors/ (Desktop app)

Use these ONLY to understand:
- How OOXML formats are structured
- How document rendering works (pagination, text layout, cell grid)
- How collaboration synchronization works
- How format conversion pipelines are built

Do NOT copy code from these repos. Write everything from scratch in Rust + Svelte.

## IMPORTANT NOTES

- AGPL v3 license: all code must remain open source
- The goal is to build a product that can be sold as SaaS AND as desktop app
- Desktop app must work fully offline (no server dependency for local editing)
- Web app requires server for collaboration and storage
- Mobile apps are Phase 5 (future), do not build for mobile now
- Focus on QUALITY over QUANTITY: a perfect document editor is worth more than a mediocre suite with all 4 editors
- Start with the document editor (Word equivalent), then spreadsheet, then presentation, then PDF


## AI / LLM INTEGRATION

### Layer 1: In-Document AI Assistant (User-Facing)

An AI assistant is embedded directly in each editor, accessible via:
- Right-click context menu on selected text/cells/slides
- Floating AI toolbar (cmd+J or /)
- Sidebar AI chat panel for complex requests

**Document Editor AI features:**
- Rewrite (change tone: formal, casual, technical, creative)
- Summarize (selection or full document)
- Translate (any language pair, powered by LLM)
- Expand / Shorten text
- Grammar and style correction (beyond basic spellcheck)
- Generate content from a brief
- Extract action items, key points, TODOs from meeting notes
- Smart templates: generate contracts, reports, emails from prompts

**Spreadsheet AI features:**
- Generate formulas from natural language
- Explain existing formulas in plain language
- Auto-detect and suggest data cleaning
- Generate charts and pivot tables from natural language
- SQL-like queries on spreadsheet data in natural language

**Presentation AI features:**
- Generate slide deck from a text brief or document
- Suggest layouts and design improvements
- Auto-summarize and generate speaker notes

**PDF AI features:**
- Summarize PDF content
- Extract structured data (tables, forms, key-value pairs)
- Q&A over PDF content ("What is the total amount on this invoice?")
- OCR + AI for scanned documents

**Technical implementation:**
- LLM calls go through the Axum server (sofdocs-server)
- Server proxies to configurable LLM backends:
  - Anthropic Claude API (default SaaS offering)
  - OpenAI-compatible API (any provider)
  - Ollama / local models (self-hosted, privacy-first)
  - AURA network (Soflution's own decentralized LLM, when ready)
- Users can bring their own API key or use Soflution's bundled credits
- Streaming responses via SSE for real-time text generation
- Context window management: only send relevant sections, not full documents
- Rate limiting per user/org to control costs

### Layer 2: Self-Improving Product Intelligence (Backend)

Anonymous telemetry to continuously improve the product. No document content collected.

**What is collected (anonymous, opt-in):**
- Format statistics: which file types are opened most
- Performance metrics: time to open, render, memory usage per doc type
- Feature usage heatmap: toolbar clicks, AI feature usage, shortcuts
- Error events: parsing failures, rendering glitches, conversion errors
- AI acceptance rate: did the user keep or discard the AI suggestion?

**Feedback loop:**
1. User uses AI feature (e.g., "rewrite formal")
2. LLM generates suggestion
3. User accepts, edits, or rejects
4. Accept/reject signal logged (NOT the content)
5. Aggregated signals improve prompts and model selection
6. Better prompts deployed via server config update
7. Cycle repeats automatically

**Privacy:**
- ALL telemetry is opt-in
- Zero document content leaves the user's environment
- Only behavioral metadata (counts, timings, feature IDs)
- Self-hosted customers can disable ALL telemetry
- GDPR compliant, no PII, anonymized user IDs

**Tech stack:**
- Telemetry batched client-side, sent every 60s
- Axum server writes to ClickHouse or TimescaleDB
- Nightly Rust batch job processes analytics
- Grafana or custom Svelte dashboard for product metrics

### Layer 3: Per-Organization Learning (Enterprise)

For enterprise/self-hosted customers, the AI adapts to their organization:
- Custom vocabulary: learns company terms, acronyms, product names
- Style guide enforcement: suggests corrections based on org writing style
- Template intelligence: learns from existing docs to improve templates
- Formula library: learns common spreadsheet patterns
- Smart autocomplete: predicts based on org corpus (local model only)

Runs ENTIRELY on customer infrastructure (self-hosted Ollama).
No org data leaves their servers. Soflution provides the framework.

### Layer 4: AURA Integration (Future)

When the AURA project (Soflution's decentralized self-improving LLM) is ready:
- SofDocs becomes a first-class AURA client
- Users can opt-in to contribute anonymized LoRA gradients
- The AURA model improves from collective SofDocs usage
- Each user gets a better AI assistant without sharing their content
- Federated learning via Flower/FedProx, aggregated in Rust
- New model versions published on IPFS, pulled automatically
- This is the long-term vision: every SofDocs user makes the AI smarter for everyone
