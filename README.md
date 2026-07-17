# Camera Raw Agent

A Photoshop UXP plugin — multimodal LLM retouch assistant for photographers, bloggers, and visual designers.


## Requirements

1. Requires Photoshop 2024+.
2. Set a Anthropic API key to call the Claude model.


##  Using the Plugin

### Installation
1. **Install the plugin** → Double-click the provided `.ccx` file. The Adobe Creative Cloud desktop app will open to handle and complete the installation.
2. **Launch Photoshop** → Once installed, open or restart Photoshop.
   
### Usage
1. **Open the panel** → In Photoshop, go to the **Plugins** menu and select **Camera Raw Agent** to open the panel.
2. **Set API key.** → On first run, store your Anthropic API key in the panel's settings (saved to UXP secure storage under `anthropic_api_key`).
3. **Open an image** and **select the layer** you want to retouch — the top-most selected layer is the active target.
4. **Convert to Smart Object** → Right-click the selected layer in the Layers panel and choose **Convert to Smart Object** (this ensures Camera Raw adjustments are applied non-destructively as a Smart Filter).
5. **Chat tab — describe the edit** in plain language
   (e.g. *"lower the exposure a bit and warm up the whites"*) and press
   **Send**. The plugin exports the layer as a PNG, sends it with your
   instruction to Claude, and streams the response.
6. **Review the preview.** When the model proposes adjustments, an
   **XMP Preview card** shows the parameters. Click **Apply** to commit them
   to the layer via Camera Raw, or **Discard** to reject.


## Examples

### Screenshot

**Before**:
![Basic](./test/basic.jpg)

**Input**: Adjust the warm yellow in the highlight to a soft orange red, keep the overall style unchanged. 

**Retouch**:
![Retouched1](./test/after1.jpg)

**Input**: The dark cool color in the ridge is getting a little warmer, restore it.

**Retouch**:
![Retouched2](./test/after2.jpg)

### Video

![Walkthrough](./test/camera-raw-agent-examples.gif)