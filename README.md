# Camera Raw Agent

A Photoshop UXP plugin — multimodal LLM retouch assistant for photographers, bloggers, and visual designers.


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
