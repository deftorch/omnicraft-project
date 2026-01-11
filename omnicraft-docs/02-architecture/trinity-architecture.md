# Trinity Architecture

> **Source:** v6.0 Section 1

## Trinity Architecture Concepts

### Design Philosophy

The system is designed based on the principle of separating computation according to hardware characteristics:

```mermaid
graph TD
    subgraph CPU[CPU (WebAssembly) → Logic & Coordination]
        State[State Management]
        Event[Event Handling]
        Scene[Scene Graph]
        BizLogic[Business Logic]
    end

    subgraph GPU[GPU (WebGPU) → Graphics & Parallelism]
        Render[Rendering]
        Compute[Compute Shaders]
        Particles[Particle Systems]
        ImgProc[Image Processing]
    end

    subgraph NPU[NPU (WebNN) → AI & Machine Learning]
        Seg[Image Segmentation]
        Detect[Object Detection]
        Style[Style Transfer]
        Neural[Neural Effects]
    end
```

**Explanation:**
The Trinity Architecture leverages the specialized capabilities of modern hardware/software stacks:
- **CPU (WebAssembly):** Handles complex logic, state management, and orchestration. It acts as the "brain," deciding what needs to be done.
- **GPU (WebGPU):** Handles massive parallelism required for graphics and compute-heavy tasks like physics simulations or image filtering.
- **NPU (WebNN):** Dedicated to AI inference tasks, allowing for efficient on-device machine learning operations without bogging down the main thread.

### Layered Architecture

```mermaid
graph TD
    classDef layer fill:#f9f,stroke:#333,stroke-width:2px;
    
    L1[LAYER 1: PRESENTATION<br>(React + TypeScript)<br>• UI Components<br>• State Management (Zustand)<br>• Event Handlers]:::layer
    L2[LAYER 2: ORCHESTRATION<br>(Rust → WebAssembly)<br>• Trinity Engine Core<br>• Workload Scheduler<br>• Memory Manager<br>• Capability Detector]:::layer
    
    subgraph L3[LAYER 3: EXECUTION ENGINES]
        L3A[LAYER 3A: WebAssembly<br>• ECS System<br>• Reactivity<br>• Algorithms]
        L3B[LAYER 3B: WebGPU<br>• Render<br>• Compute<br>• Shaders]
        L3C[LAYER 3C: WebNN<br>• Models<br>• Inference<br>• Training]
    end
    
    L4[LAYER 4: SHARED MEMORY<br>(SharedArrayBuffer)<br>• Zero-copy data transfer<br>• Memory arena (256 MB)<br>• Cross-technology views]:::layer

    L1 --> L2
    L2 --> L3A
    L2 --> L3B
    L2 --> L3C
    L3A --> L4
    L3B --> L4
    L3C --> L4
```

**Explanation:**
- **Layer 1 (Presentation):** The user interface layer built with standard web technologies. It captures user intent and displays the final output.
- **Layer 2 (Orchestration):** The core logic engine (written in Rust/Wasm) that coordinates tasks between the specialized execution units.
- **Layer 3 (Execution Engines):** The specialized workers. Wasm for logic/physics, WebGPU for graphics/compute, and WebNN for AI.
- **Layer 4 (Shared Memory):** A critical component allowing zero-copy data transfer. Data lives in one place and is accessed by different "views" (CPU, GPU, NPU) to maximize performance.

### Communication Patterns

```text
┌─────────────────────────────────────────────────────┐
│            INTER-LAYER COMMUNICATION                 │
├─────────────────────────────────────────────────────┤
│                                                      │
│  SYNCHRONOUS (UI → Wasm):                           │
│  ┌──────┐                        ┌──────┐          │
│  │  UI  │ ──── Function Call ───→│ Wasm │          │
│  └──────┘ ←─── Return Value ─────└──────┘          │
│                                                      │
│  ASYNCHRONOUS (Wasm → GPU/NPU):                     │
│  ┌──────┐                        ┌──────┐          │
│  │ Wasm │ ──── Submit Task ──────→│ GPU  │          │
│  └──────┘ ←─── Promise/Await ────└──────┘          │
│                                                      │
│  ZERO-COPY (GPU ↔ NPU):                             │
│  ┌──────┐     Shared Memory      ┌──────┐          │
│  │ GPU  │ ←────────────────────→ │ NPU  │          │
│  └──────┘    (No Copy)            └──────┘          │
│                                                      │
└─────────────────────────────────────────────────────┘
```

**Explanation:**
- **Synchronous:** UI interactions with the Wasm core happen instantly to ensure the interface feels responsive.
- **Asynchronous:** Valid heavy lifting (rendering, AI inference) is offloaded asynchronously to prevent freezing the main thread.
- **Zero-Copy:** The unique advantage of this architecture; the GPU and NPU can work on the same data in shared memory without the expensive overhead of copying data back and forth between CPU and device memory.
