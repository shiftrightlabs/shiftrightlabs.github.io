# AI Models Directory

This directory contains AI models for hair segmentation.

## For Development

Models are downloaded from CDN automatically during runtime.

## For Offline/Enterprise Deployment

Run the model download script to fetch all required models:

```bash
npm run download-models
```

This will download:
- MediaPipe segmentation model (~1.2 MB)
- MediaPipe WASM runtime files

## Directory Structure

```
models/
├── mediapipe/
│   └── selfie_multiclass_256x256.tflite  (downloaded)
└── modnet/
    └── README.md  (optional, see OFFLINE_DEPLOYMENT.md)
```

## Model Sources

- **MediaPipe**: https://storage.googleapis.com/mediapipe-models/
- **MODNet**: https://huggingface.co/Xenova/modnet

## Notes

- Model files are git-ignored (too large for repository)
- Download script must be run before offline deployment
- For online deployment, models are fetched from CDN automatically
