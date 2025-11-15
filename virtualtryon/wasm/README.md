# WebAssembly Directory

This directory contains WebAssembly runtime files for MediaPipe.

## For Development

WASM files are loaded from CDN automatically during runtime.

## For Offline/Enterprise Deployment

Run the model download script to fetch all required WASM files:

```bash
npm run download-models
```

This will download MediaPipe WASM files to:
```
wasm/
└── mediapipe/
    ├── vision_wasm_internal.wasm
    ├── vision_wasm_internal.js
    ├── vision_wasm_nosimd_internal.wasm
    └── vision_wasm_nosimd_internal.js
```

## Purpose

MediaPipe uses WebAssembly for fast AI model execution in the browser.

## Notes

- WASM files are git-ignored (binary files)
- Download script must be run before offline deployment
- For online deployment, WASM files are loaded from CDN automatically
