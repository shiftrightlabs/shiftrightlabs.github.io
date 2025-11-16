# MODNet Model - Local Deployment

This directory contains the MODNet model files for offline/local deployment.

## Model Files

The following files are included (downloaded from https://huggingface.co/Xenova/modnet):
- `config.json` - Model configuration
- `model.onnx` - ONNX model file (~25MB)
- `preprocessor_config.json` - Preprocessing configuration

## Configuration

The application is configured to use local models by default when `VITE_USE_LOCAL_MODELS=true` is set in `.env.production`.

**Local Mode (Default for Production)**:
- Models served from `/virtualtryon/models/modnet/`
- No external downloads from Hugging Face
- Fully offline-capable
- Larger initial bundle size (~25MB)

**Remote Mode (Development)**:
- Models downloaded from Hugging Face CDN on first use
- Browser caches downloaded models
- Smaller initial bundle
- Requires internet connection on first use

## Deployment

For production builds, these model files are automatically copied to the `dist/models/modnet/` directory and served as static assets.
