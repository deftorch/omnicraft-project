# Build System

The build system handles the complex task of orchestrating compilation for Rust, TypeScript, shaders, and AI models into a coherent web application.

## 15. Build System Architecture

The architecture is designed to handle multi-language compilation with dependencies between build steps.

### 15.1 Multi-Target Build Configuration

This configuration uses Vite to handle a complex build pipeline that targets standard JavaScript/TypeScript, WebAssembly, and shaders simultaneously.

**Key Layout:**
- **Plugins:** Integrates React, Wasm, and Top-level Await support (essential for async Wasm initialization).
- **Manual Chunks:** Splits the output into three main functional parts:
    - `wasm-core`: The critical runtime logic loaded immediately.
    - `webgpu`: Graphical pipeline code loaded lazily to save bandwidth if not used.
    - `webnn`: AI model code loaded lazily.
- **Asset Naming:** Ensures build artifacts like `.wasm`, `.wgsl` (shaders), and `.onnx` (models) are named and hashed correctly for efficient browser caching.

```javascript
// vite.config.ts - Multi-target Build Configuration

import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import wasm from 'vite-plugin-wasm';
import topLevelAwait from 'vite-plugin-top-level-await';

export default defineConfig({
    plugins: [
        react(),
        wasm(),
        topLevelAwait(),
        customWasmOptimizer(), // Hypothetical plugin for custom optimization
        shaderCompiler(),       // Plugin to validate/minify WGSL
        modelOptimizer(),       // Plugin to optimize ONNX models
    ],
    
    build: {
        target: 'esnext', // Required for Top-level await
        minify: 'terser',
        
        rollupOptions: {
            output: {
                manualChunks: {
                    // Wasm core (critical path)
                    'wasm-core': ['./pkg/omnicraft_core.js'],
                    
                    // Graphics (lazy load)
                    'webgpu': [
                        './src/webgpu/context.ts',
                        './src/webgpu/pipeline.ts',
                    ],
                    
                    // AI (lazy load)
                    'webnn': [
                        './src/webnn/context.ts',
                        './src/webnn/models.ts',
                    ],
                },
                
                // Asset file names customization
                assetFileNames: (assetInfo) => {
                    const ext = assetInfo.name.split('.').pop();
                    
                    if (/wasm/.test(ext)) {
                        return 'wasm/[name]-[hash][extname]';
                    }
                    if (/wgsl/.test(ext)) {
                        return 'shaders/[name]-[hash][extname]';
                    }
                    if (/onnx/.test(ext)) {
                        return 'models/[name]-[hash][extname]';
                    }
                    
                    return 'assets/[name]-[hash][extname]';
                },
            },
        },
    },
});
```

### 15.2 Cargo Build Configuration

The `Cargo.toml` file is configured to produce the smallest and fastest possible WebAssembly binary. Since Wasm must be downloaded over the network, size impact is critical.

**Optimization Settings:**
- **opt-level = "z":** Instructs the compiler to prioritize binary size over pure execution speed.
- **lto = true:** Enables Link-Time Optimization, allowing the compiler to optimize across crate boundaries.
- **panic = "abort":** Removes stack unwinding code. If the app crashes, it just stops, saving significant space.
- **strip = true:** Removes debug symbols from the final binary, essential for production release.

```toml
# Cargo.toml - Rust/Wasm Build Configuration

[package]
name = "omnicraft-core"
version = "6.0.0"
edition = "2021"

[lib]
crate-type = ["cdylib", "rlib"] # cdylib is required for Wasm

[profile.release]
# Optimize for size
opt-level = "z"

# Enable link-time optimization
lto = true

# Single codegen unit for better optimization at cost of compile time
codegen-units = 1

# Panic = abort for smaller binary (no stack unwinding)
panic = "abort"

# Strip symbols to reduce size
strip = true

[profile.pgo]
inherits = "release"
```

### 15.3 Build Script Automation

This shell script orchestrates the entire build process, ensuring dependencies are built in the correct order.

**Build Pipeline Steps:**
1.  **Clean:** Removes old `dist` and `pkg` directories to ensure no stale artifacts remain.
2.  **Rust Compile:** Builds the core logic into WebAssembly using the release profile.
3.  **Wasm Optimization:** Runs `wasm-opt` (from binaryen) to further shrink and optimize the binary. This step often reduces size by 30-50% compared to raw Cargo output.
4.  **Shader Validation:** Compiles and checks WGSL shaders to catch errors at build time rather than runtime.
5.  **Asset Bundle:** Uses Vite to bundle the final web application, integrating the Wasm and assets generated in previous steps.

```bash
#!/bin/bash
# build.sh - Automated Build Script

set -e  # Exit script immediately on first error

echo "🚀 OmniCraft 6.0 Build System"

# Step 1: Clean previous build
echo "🧹 Cleaning previous build..."
rm -rf dist pkg

# Step 2: Build Rust → Wasm
echo "🦀 Compiling Rust to WebAssembly..."
cargo build --target wasm32-unknown-unknown --profile release

# Step 3: Optimize Wasm
# Uses binaryen's wasm-opt for post-link optimization
echo "⚡ Optimizing Wasm binary..."
wasm-opt -Oz --enable-simd --enable-bulk-memory pkg/omnicraft_core_bg.wasm -o pkg/omnicraft_core_bg.wasm

# Step 4: Compile shaders
# Ensures WGSL code is valid before packaging
echo "🎨 Compiling WGSL shaders..."
# Validation logic here (e.g., naga-cli validate src/shaders/*.wgsl)

# Step 5: Build web assets
echo "📦 Building web assets with Vite..."
npm run build

echo "✨ Build complete!"
```
