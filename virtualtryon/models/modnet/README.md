# MODNet Model

The MODNet model is downloaded from Hugging Face on first use and cached in the browser.

For offline deployment with MODNet:

1. **Option A: Pre-cache the model (Recommended)**
   - Run the application once online to download the model
   - The browser will cache it (IndexedDB)
   - Export the cached model and include it in deployment

2. **Option B: Host the model locally**
   - Download the model from https://huggingface.co/Xenova/modnet
   - Place ONNX files in this directory
   - Configure Transformers.js to use local path (requires code changes)

3. **Option C: Use MediaPipe only**
   - Disable MODNet option in the UI
   - Use only MediaPipe for segmentation (no additional downloads needed)

## Model Files (if hosting locally)

Required files from Hugging Face:
- config.json
- model.onnx
- preprocessor_config.json

Note: Full offline MODNet support requires additional configuration.
For enterprise deployments, Option C (MediaPipe only) is recommended.
