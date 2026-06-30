# Camera Raw Agent

A Photoshop UXP plugin — multimodal LLM retouch assistant for photographers, bloggers, and visual designers.

## Architecture

```
camera-raw-agent/
├── ui/                   # React Chat UI (UXP panel)
├── llm-service/          # LLM query loop (Claude API, streaming, tool dispatch)
├── tool-service/         # Tool factory & registry (xmp_write, file_read, etc.)
├── xmp-service/          # XMP validation, serialization, BatchPlay apply
├── ps-engine/            # Raw Photoshop/BatchPlay API wrappers
├── product_mrd.md
└── development_instructions.md
```

## Module Responsibilities

| Module | Role |
|--------|------|
| `ui/` | Chat panel + Parameter Slider panel React UI, global store, user input/output |
| `llm-service/` | Stream query loop, tool call dispatch, prompt assembly |
| `tool-service/` | Tool definitions, BaseTool class, `ToolUseContext` |
| `xmp-service/` | XMPParams → XMP XML → BatchPlay apply → PNG export |
| `ps-engine/` | Lowest-level UXP/BatchPlay wrappers used by all services |

## Data Flow

```
User types instruction
      │
      ▼
ui/REPL.tsx → runUserTurn()
  → ps-engine: exportPNG(layerId) ──────────── base64 PNG of active layer
  → llm-service: query({ messages, image })
        │
        ├─ stream text_delta → UI appends to chat
        │
        └─ tool_use: file_write("edit.xmp", <xmp string>)
              │
              ├─ tool-service: executeTool() writes the .xmp to temp storage
              └─ yield tool_result → UI parses crs: params → XMPPreviewCard
                        │
                   User clicks Apply
                        │
                   xmp-service: applyFromFile(layerId, path)
                        │  (validate ranges → ps-engine.applyXMP)
                   ps-engine: setCameraRawSettings via BatchPlay
                        │
                   Parameter Slider panel reflects the new values
```

The Parameter Slider panel can also drive edits directly: moving sliders
updates `xmpParams` in the store, and "Apply to Layer" serializes them to an
`.xmp` and runs the same `xmp-service` apply path.

## Build Order

1. `ps-engine/` — no dependencies on other modules
2. `xmp-service/` — depends on ps-engine
3. `tool-service/` — self-contained tool factory
4. `llm-service/` — depends on tool-service
5. `ui/` — depends on llm-service + xmp-service + ps-engine

## Each Module Has a DEV.md

Read the `DEV.md` inside each sub-directory for:
- Detailed file layout
- Public API signatures
- Key flows with pseudocode
- Constraints and gotchas

## Building the Plugin

```bash
npm install            # install dependencies
npm run build          # production bundle → dist/ (index.js, index.html, manifest.json)
npm run dev            # watch mode — rebuilds dist/ on every source change
```

`require('photoshop')` and `require('uxp')` only resolve inside the UXP
runtime, so the code cannot be exercised with plain `node`. Run it inside
Photoshop via the UXP Developer Tool (below).

## Loading & Debugging in Photoshop

1. **Install the UXP Developer Tool (UDT)** — free, from
   <https://developer.adobe.com/photoshop/uxp/2022/guides/devtool/installation/>.
2. **Start a watch build** so `dist/` stays current:
   ```bash
   npm run dev
   ```
3. **Add the plugin in UDT** → *Add Plugin* → select
   `camera-raw-agent/manifest.json` (the repo root manifest; it points
   `main` at `dist/index.js`).
4. **Load** → UDT pushes the plugin into the running Photoshop instance.
   In Photoshop: **Plugins menu → Camera Raw Agent** opens the panel.
5. **Debug** → click *Debug* next to the loaded plugin in UDT to open
   Chrome DevTools (console, breakpoints, network) attached to the panel.
6. **Iterate** → edit code → webpack rebuilds `dist/` → click *Reload* in
   UDT. No Photoshop restart needed.

Requires Photoshop 2024+ (v25.0, per `manifest.json`).

## Using the Plugin

1. **Set your API key.** On first run, store your Anthropic API key in the
   panel's settings (saved to UXP secure storage under `anthropic_api_key`).
2. **Open an image** and **select the layer** you want to retouch — the
   top-most selected layer is the active target.
3. **Chat tab — describe the edit** in plain language
   (e.g. *"lower the exposure a bit and warm up the whites"*) and press
   **Send**. The plugin exports the layer as a PNG, sends it with your
   instruction to Claude, and streams the response.
4. **Review the preview.** When the model proposes adjustments, an
   **XMP Preview card** shows the parameters. Click **Apply** to commit them
   to the layer via Camera Raw, or **Discard** to reject.
5. **Sliders tab — manual control.** Adjust Camera Raw-style sliders
   (Basic / Color Grading / Calibration) directly, then **Apply to Layer**.
   Both panels share the same XMP state.

> **Permissions:** the plugin requests local file system access (to write
> temporary `.xmp`/PNG files) and network access to `api.anthropic.com`
> only. Your API key lives in UXP secure storage — never commit or log it.
