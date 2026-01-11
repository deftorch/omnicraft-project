# WebGPU Pipeline Design


## 12. Rendering Pipeline

### 12.1 Render Command Buffer

The `RenderCommandBuffer` abstraction decouples command recording from submission, allowing for multi-threaded command generation.

**Structure:**
- **RenderCommand:** An enum representing individual GPU commands (Draw, Copy, Dispatch).
- **StateChange:** Represents pipeline state modifications (setting pipelines, bind groups, viewports).
- **Batching:** The system collects commands and state changes into vectors before submitting them to the GPU queue in a single batch, minimizing overhead.

```rust
// Render Command Buffer Design

pub struct RenderCommandBuffer {
    commands: Vec<RenderCommand>,
    state_changes: Vec<StateChange>,
}

pub enum RenderCommand {
    // Drawing geometry
    Draw {
        vertex_count: u32,
        instance_count: u32,
        first_vertex: u32,
        first_instance: u32,
    },
    
    // Indexed drawing for optimized mesh rendering
    DrawIndexed {
        index_count: u32,
        instance_count: u32,
        first_index: u32,
        base_vertex: i32,
        first_instance: u32,
    },
    
    // ... (other variants)
}
// ...
```

### 12.2 Instanced Rendering

Instanced rendering allows drawing many identical objects (like particles, grass, or tiles) in a single draw call, which is essential for performance.

**Key Components:**
- **InstanceData:** A compact struct containing per-instance properties like position (transform), color, and UV coordinates. Pod/Zeroable traits allow direct memory mapping.
- **Instance Buffer:** A dedicated GPU buffer holding the `InstanceData` array.
- **Rendering:** The `render` method sets up the pipeline once and issues a single `draw` (or `draw_indexed`) call for all instances.

```rust
// Instanced Rendering Design
// ... implementation details ...
```

### 12.3 Multi-Pass Rendering

A flexible system for handling complex rendering techniques like Deferred Rendering, Shadows, or Post-processing chains.

**Mechanism:**
- **RenderPass:** Defines a single stage of rendering (e.g., "Main Geometry", "Lighting", "Blur"). It specifies inputs (textures to read), outputs (framebuffers to write to), and an execution closure.
- **Framebuffer:**  Manages the textures and views that store pixel data.
- **Execution:** The `MultiPassRenderer` iterates through passes, ensuring dependencies are met and managing the GPU command encoding for transitions between passes.

```rust
// Multi-Pass Rendering System
// ... implementation details ...
```

## 13. Compute Pipeline

### 13.1 Compute Shader Pipeline

Compute shaders enable general-purpose parallel computing on the GPU, useful for physics, simulations, and data processing.

**Example: Particle System**
The WGSL shader below simulates particle physics entirely on the GPU:
- **Bindings:** Uses a storage buffer for reading/writing particle data and a uniform buffer for global parameters (time, gravity).
- **Logic:** Updates position based on velocity and gravity, manages lifecycle (respawning), and handles boundary collisions.
- **Efficiency:** Thousands of particles are updated in parallel.

```wgsl
// Compute Shader: Particle System
// ... shader code ...
```

**Manager Integration:**
The Rust `ComputePipelineManager` handles the creation and dispatch of these compute shaders.

```rust
// Compute Pipeline Manager
// ... implementation details ...
```

### 13.2 Parallel Reduction

Parallel reduction is a common algorithm to sum up values across a large array using the GPU.

**Algorithm:**
- **Shared Memory:** Uses `var<workgroup>` to store intermediate results accessible by all threads in a workgroup.
- **Barrier Synchronization:** `workgroupBarrier()` ensures all threads have finished their current step before proceeding.
- **Iterative Reduction:** Threads cooperatively sum values in a tree-like structure (stride /= 2) until a single value remains for the workgroup.

```wgsl
// Compute Shader: Parallel Sum Reduction
// ... shader code ...
```
