# Fallback System


## 17. Sistem Fallback

### 17.1 Fallback Chain Implementation

```typescript
// Fallback Chain Manager

class FallbackChainManager {
    private capabilities: Capabilities;
    private fallbackChain: ComputeBackend[] = [];
    
    constructor(capabilities: Capabilities) {
        this.capabilities = capabilities;
        this.buildFallbackChain();
    }
    
    private buildFallbackChain(): void {
        // Build optimal fallback chain based on available features
        
        // AI workloads
        if (this.capabilities.webnnNPU) {
            this.fallbackChain.push('webnn-npu');
        }
        if (this.capabilities.webnnGPU) {
            this.fallbackChain.push('webnn-gpu');
        }
        if (this.capabilities.webgpu) {
            this.fallbackChain.push('webgpu-compute');
        }
        if (this.capabilities.wasmSIMD) {
            this.fallbackChain.push('wasm-simd');
        }
        if (this.capabilities.webassembly) {
            this.fallbackChain.push('wasm-scalar');
        }
        
        // Ultimate fallback
        this.fallbackChain.push('javascript');
        
        console.log('Fallback chain:', this.fallbackChain);
    }
    
    async execute<T>(
        task: Task,
        implementations: Map<ComputeBackend, TaskImplementation<T>>
    ): Promise<T> {
        const errors: Array<{ backend: ComputeBackend; error: Error }> = [];
        
        for (const backend of this.fallbackChain) {
            const impl = implementations.get(backend);
            
            if (!impl) {
                continue; // Skip if implementation not provided
            }
            
            try {
                console.log(`Attempting ${task.name} with ${backend}...`);
                const startTime = performance.now();
                
                const result = await impl.execute(task);
                
                const duration = performance.now() - startTime;
                console.log(`✓ Success with ${backend} (${duration.toFixed(2)}ms)`);
                
                return result;
                
            } catch (error) {
                console.warn(`✗ Failed with ${backend}:`, error);
                errors.push({ backend, error: error as Error });
                continue; // Try next backend
            }
        }
        
        // All backends failed
        const errorMessage = errors
            .map(e => `${e.backend}: ${e.error.message}`)
            .join('\n');
        
        throw new Error(
            `All backends failed for task "${task.name}":\n${errorMessage}`
        );
    }
    
    getAvailableBackends(): ComputeBackend[] {
        return [...this.fallbackChain];
    }
    
    isBackendAvailable(backend: ComputeBackend): boolean {
        return this.fallbackChain.includes(backend);
    }
}

type ComputeBackend = 
    | 'webnn-npu' 
    | 'webnn-gpu' 
    | 'webgpu-compute' 
    | 'webgpu-fragment'
    | 'wasm-simd'
    | 'wasm-scalar'
    | 'javascript';

interface Task {
    name: string;
    type: 'render' | 'compute' | 'ai' | 'logic';
    data: any;
}

interface TaskImplementation<T> {
    execute(task: Task): Promise<T>;
}
```

### 17.2 Feature Detection

```typescript
// Comprehensive Feature Detection

async function detectCapabilities(): Promise<Capabilities> {
    console.log('Detecting browser capabilities...');
    
    const caps: Capabilities = {
        // WebAssembly
        webassembly: typeof WebAssembly !== 'undefined',
        wasmSIMD: await checkWasmSIMD(),
        wasmThreads: await checkWasmThreads(),
        wasmGC: await checkWasmGC(),
        
        // Graphics
        webgpu: 'gpu' in navigator,
        webgl2: checkWebGL2(),
        
        // AI
        webnn: 'ml' in navigator,
        webnnNPU: false,  // Will be checked below
        webnnGPU: false,  // Will be checked below
        
        // Memory
        sharedArrayBuffer: typeof SharedArrayBuffer !== 'undefined',
        
        // Video
        webcodecs: 'VideoEncoder' in window,
        
        // Performance tier
        tier: 'low',  // Will be calculated
    };
    
    // Check WebNN device types
    if (caps.webnn) {
        caps.webnnNPU = await checkWebNNDevice('npu');
        caps.webnnGPU = await checkWebNNDevice('gpu');
    }
    
    // Calculate performance tier
    caps.tier = calculatePerformanceTier(caps);
    
    return caps;
}

async function checkWasmSIMD(): Promise<boolean> {
    try {
        // Minimal SIMD test module
        const simdModule = new Uint8Array([
            0, 97, 115, 109, 1, 0, 0, 0, 1, 5, 1, 96, 0, 1, 123, 3, 2, 1, 0,
            10, 10, 1, 8, 0, 65, 0, 253, 15, 253, 98, 11
        ]);
        
        return WebAssembly.validate(simdModule);
    } catch {
        return false;
    }
}

async function checkWasmThreads(): Promise<boolean> {
    return typeof SharedArrayBuffer !== 'undefined' &&
           typeof Atomics !== 'undefined';
}

async function checkWasmGC(): Promise<boolean> {
    try {
        // Test for GC proposal support
        const gcModule = new Uint8Array([
            0, 97, 115, 109, 1, 0, 0, 0,
            1, 7, 1, 96, 1, 111, 0, 1, 111, 0,
        ]);
        
        return WebAssembly.validate(gcModule);
    } catch {
        return false;
    }
}

function checkWebGL2(): boolean {
    try {
        const canvas = document.createElement('canvas');
        return !!canvas.getContext('webgl2');
    } catch {
        return false;
    }
}

async function checkWebNNDevice(deviceType: 'npu' | 'gpu'): Promise<boolean> {
    try {
        if (!('ml' in navigator)) return false;
        
        const ml = (navigator as any).ml;
        const context = await ml.createContext({ deviceType });
        
        return context.deviceType === deviceType;
    } catch {
        return false;
    }
}

function calculatePerformanceTier(caps: Capabilities): 'high' | 'medium' | 'low' {
    // High tier: WebGPU + WebNN (NPU) + SIMD
    if (caps.webgpu && caps.webnnNPU && caps.wasmSIMD) {
        return 'high';
    }
    
    // Medium tier: WebGPU OR WebNN, with SIMD
    if ((caps.webgpu || caps.webnn) && caps.wasmSIMD) {
        return 'medium';
    }
    
    // Low tier: Basic support
    return 'low';
}

interface Capabilities {
    webassembly: boolean;
    wasmSIMD: boolean;
    wasmThreads: boolean;
    wasmGC: boolean;
    
    webgpu: boolean;
    webgl2: boolean;
    
    webnn: boolean;
    webnnNPU: boolean;
    webnnGPU: boolean;
    
    sharedArrayBuffer: boolean;
    webcodecs: boolean;
    
    tier: 'high' | 'medium' | 'low';
}
```
