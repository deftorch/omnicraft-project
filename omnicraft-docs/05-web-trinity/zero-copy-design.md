# Zero-Copy Memory Design

> **Source:** v6.0 Section 11

## 11. Zero-Copy Design

### 11.1 Zero-Copy Pipeline Pattern

```typescript
// Zero-Copy Data Transfer Pattern

interface ZeroCopyPipeline {
    // Step 1: Allocate shared memory
    allocateShared(size: number): SharedMemoryHandle;
    
    // Step 2: Get views for different technologies
    getWasmView(handle: SharedMemoryHandle): Uint8Array;
    getGPUView(handle: SharedMemoryHandle): GPUBuffer;
    getNPUView(handle: SharedMemoryHandle): MLTensor;
    
    // Step 3: Execute operations without copying
    executeZeroCopy(pipeline: PipelineStage[]): Promise<void>;
}

interface PipelineStage {
    technology: 'wasm' | 'webgpu' | 'webnn';
    operation: string;
    input: SharedMemoryHandle;
    output: SharedMemoryHandle;
}

// Example Implementation
class ZeroCopyImagePipeline implements ZeroCopyPipeline {
    private sharedMemory: SharedMemoryArena;
    private wasmEngine: WasmEngine;
    private gpuContext: WebGPUContext;
    private nnContext: WebNNContext;
    
    async processImage(imageData: ImageData): Promise<ImageData> {
        // Allocate shared memory once
        const inputHandle = this.allocateShared(
            imageData.width * imageData.height * 4
        );
        
        const workHandle = this.allocateShared(
            512 * 512 * 4  // Working size
        );
        
        const outputHandle = this.allocateShared(
            imageData.width * imageData.height * 4
        );
        
        // Write input (only copy: external → shared)
        const inputView = this.getWasmView(inputHandle);
        inputView.set(imageData.data);
        
        // Define pipeline stages
        const pipeline: PipelineStage[] = [
            {
                technology: 'webgpu',
                operation: 'resize',
                input: inputHandle,
                output: workHandle,
            },
            {
                technology: 'webnn',
                operation: 'segment',
                input: workHandle,
                output: workHandle,
            },
            {
                technology: 'webgpu',
                operation: 'apply_mask',
                input: inputHandle,
                output: outputHandle,
            },
        ];
        
        // Execute without copying between stages
        await this.executeZeroCopy(pipeline);
        
        // Read result (only copy: shared → external)
        const outputView = this.getWasmView(outputHandle);
        const result = new ImageData(
            new Uint8ClampedArray(outputView),
            imageData.width,
            imageData.height
        );
        
        // Cleanup
        this.freeShared(inputHandle);
        this.freeShared(workHandle);
        this.freeShared(outputHandle);
        
        return result;
    }
    
    async executeZeroCopy(stages: PipelineStage[]): Promise<void> {
        for (const stage of stages) {
            switch (stage.technology) {
                case 'wasm':
                    await this.executeWasmStage(stage);
                    break;
                case 'webgpu':
                    await this.executeGPUStage(stage);
                    break;
                case 'webnn':
                    await this.executeNPUStage(stage);
                    break;
            }
        }
    }
    
    private async executeGPUStage(stage: PipelineStage): Promise<void> {
        // Get GPU views (no copy)
        const inputBuffer = this.getGPUView(stage.input);
        const outputBuffer = this.getGPUView(stage.output);
        
        // Execute GPU operation
        await this.gpuContext.executeOperation(
            stage.operation,
            inputBuffer,
            outputBuffer
        );
    }
    
    private async executeNPUStage(stage: PipelineStage): Promise<void> {
        // Get NPU views (no copy)
        const inputTensor = this.getNPUView(stage.input);
        const outputTensor = this.getNPUView(stage.output);
        
        // Execute NPU operation
        await this.nnContext.executeOperation(
            stage.operation,
            inputTensor,
            outputTensor
        );
    }
}
```

### 11.2 Memory Barrier & Synchronization

```rust
// Memory Synchronization for Zero-Copy

pub struct MemorySynchronizer {
    // Atomic flags for each memory region
    flags: Vec<AtomicU32>,
}

// Memory states
const STATE_IDLE: u32 = 0;
const STATE_WASM_WRITING: u32 = 1;
const STATE_GPU_READING: u32 = 2;
const STATE_GPU_WRITING: u32 = 3;
const STATE_NPU_READING: u32 = 4;
const STATE_NPU_WRITING: u32 = 5;

impl MemorySynchronizer {
    pub fn new(num_regions: usize) -> Self {
        Self {
            flags: (0..num_regions)
                .map(|_| AtomicU32::new(STATE_IDLE))
                .collect(),
        }
    }
    
    pub fn acquire_read(
        &self,
        region: usize,
        reader: Technology,
    ) -> Result<(), String> {
        let flag = &self.flags[region];
        
        loop {
            let current = flag.load(Ordering::Acquire);
            
            // Can read if idle or already being read
            if current == STATE_IDLE || self.is_reading_state(current) {
                let new_state = self.reading_state(reader);
                
                if flag
                    .compare_exchange(
                        current,
                        new_state,
                        Ordering::Release,
                        Ordering::Relaxed,
                    )
                    .is_ok()
                {
                    return Ok(());
                }
            } else {
                // Someone is writing, wait
                std::thread::yield_now();
            }
        }
    }
    
    pub fn acquire_write(
        &self,
        region: usize,
        writer: Technology,
    ) -> Result<(), String> {
        let flag = &self.flags[region];
        
        loop {
            let current = flag.load(Ordering::Acquire);
            
            // Can only write if idle
            if current == STATE_IDLE {
                let new_state = self.writing_state(writer);
                
                if flag
                    .compare_exchange(
                        STATE_IDLE,
                        new_state,
                        Ordering::Release,
                        Ordering::Relaxed,
                    )
                    .is_ok()
                {
                    return Ok(());
                }
            } else {
                // Wait for current operation to finish
                std::thread::yield_now();
            }
        }
    }
    
    pub fn release(&self, region: usize) {
        self.flags[region].store(STATE_IDLE, Ordering::Release);
    }
    
    fn is_reading_state(&self, state: u32) -> bool {
        matches!(state, STATE_GPU_READING | STATE_NPU_READING)
    }
    
    fn reading_state(&self, tech: Technology) -> u32 {
        match tech {
            Technology::Wasm => STATE_IDLE,  // Wasm doesn't need sync
            Technology::WebGPU => STATE_GPU_READING,
            Technology::WebNN => STATE_NPU_READING,
        }
    }
    
    fn writing_state(&self, tech: Technology) -> u32 {
        match tech {
            Technology::Wasm => STATE_WASM_WRITING,
            Technology::WebGPU => STATE_GPU_WRITING,
            Technology::WebNN => STATE_NPU_WRITING,
        }
    }
}

#[derive(Debug, Clone, Copy)]
pub enum Technology {
    Wasm,
    WebGPU,
    WebNN,
}
```
