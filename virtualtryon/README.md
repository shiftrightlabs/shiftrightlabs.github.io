# Virtual Hair Color Simulator

**Live Demo**: https://www.shiftrightlabs.com/virtualtryon/

## About

AI-powered virtual hair color try-on application. Upload a photo or use your camera to see how different hair colors look on you.

### Features

- 🎨 **Multiple Hair Colors**: Try various shades of blonde, brunette, black, red, and fashion colors
- 🤖 **AI-Powered**: Automatic hair detection using MediaPipe
- 📸 **Camera & Upload**: Use your camera for real-time preview or upload photos
- 🎨 **Realistic Rendering**: WebGL-based color application with natural blending
- 📱 **Mobile-Friendly**: Fully responsive design optimized for all devices
- 💾 **Download Results**: Save your colored photos
- 🔒 **Privacy-First**: All processing happens in your browser - no data sent to servers

### Technology

- **Frontend**: React 18 + TypeScript + Vite
- **AI**: MediaPipe (Google) for hair segmentation
- **Rendering**: WebGL for realistic color blending
- **Code Protection**: Obfuscated for copyright protection

### Deployment Info

- **Build Date**: November 15, 2025
- **Build Type**: Obfuscated + Offline
- **Base Path**: `/virtualtryon/`
- **Package Size**: ~25 MB (with models)

### Browser Support

- Chrome 90+ ✅
- Edge 90+ ✅
- Firefox 88+ ✅
- Safari 14+ ✅

### Notes

**Camera Mode Limitations**:
GitHub Pages doesn't support custom CORS headers required for WebAssembly SharedArrayBuffer.
- ✅ **Upload mode** works perfectly
- ⚠️ **Camera mode** may have limitations

For full camera support, consider deploying to a server with CORS header configuration.

---

© 2025 ShiftRight Labs. All Rights Reserved.
