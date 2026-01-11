[← Back to Index](../README.md)

# System Architecture

## 1. High-Level Architecture

OmniCraft employs a **Trinity Architecture**, separating computation across CPU (WebAssembly), GPU (WebGPU), and NPU (WebNN) for optimized performance.

### 1.1 System Layers

The system is composed of four distinct layers that handle different aspects of the development and execution lifecycle.

```
┌─────────────────────────────────────────────────────────┐
│  LAYER 1: Developer Interface                           │
│  • .omni files (declarative syntax)                     │
│  • Component definitions                                │
│  • Type annotations                                     │
│  • IDE integration (LSP)                                │
14: └────────────────────┬────────────────────────────────────┘
15:                      │
16:                      ↓ Compile Time
17: ┌────────────────────▼────────────────────────────────────┐
18: │  LAYER 2: Compiler with Incremental Caching             │
19: │  • Lexical analysis                                     │
20: │  • Syntax parsing                                       │
21: │  • Type checking & inference                            │
22: │  • Optimization passes                                  │
23: │  • Code generation                                      │
24: │  • Cache management                                     │
25: │  • Dependency tracking                                  │
26: └────────────────────┬────────────────────────────────────┘
27:                      │
28:                      ↓ Produces
29: ┌────────────────────▼────────────────────────────────────┐
30: │  LAYER 3: Generated Code                                │
31: │  • Rust code                                            │
32: │  • WASM binary                                          │
33: │  • TypeScript definitions                               │
34: │  • Source maps                                          │
35: └────────────────────┬────────────────────────────────────┘
36:                      │
37:                      ↓ Runtime
38: ┌────────────────────▼────────────────────────────────────┐
39: │  LAYER 4: Trinity Runtime                               │
40: │  • Wasm Engine (CPU Logic)                              │
41: │  • WebGPU Engine (Graphics/Compute)                     │
42: │  • WebNN Engine (AI Inference)                          │
43: │  • Shared Memory Arena (Zero-Copy)                      │
44: └─────────────────────────────────────────────────────────┘
```

### 1.2 Trinity Architecture Diagram

The Trinity Architecture integrates CPU, GPU, and NPU into a cohesive runtime environment.

```
                    ┌─────────────────────┐
                    │   Browser Window    │
                    └──────────┬──────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        │                      │                      │
   ┌────▼────┐          ┌─────▼─────┐         ┌─────▼─────┐
   │  Canvas │          │  Controls │         │ Timeline  │
   │ (WebGPU)│          │  (React)  │         │  (React)  │
   └────┬────┘          └─────┬─────┘         └─────┬─────┘
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               │
                ┌──────────────▼──────────────┐
                │   Trinity Engine (Wasm)     │
                │  ┌────────────────────────┐ │
                │  │  Workload Scheduler    │ │
                │  │  Memory Manager        │ │
                │  │  Capability Detector   │ │
                │  └────────────────────────┘ │
                └──┬──────────┬──────────┬───┘
                   │          │          │
        ┌──────────▼───┐  ┌──▼──────┐  ┌▼──────────┐
        │ Wasm Module  │  │  WebGPU │  │  WebNN    │
        │              │  │  Engine │  │  Engine   │
        │ ┌──────────┐ │  │         │  │           │
        │ │   ECS    │ │  │ Render  │  │  Models   │
        │ │  World   │ │  │ Compute │  │  Tensors  │
        │ │ Systems  │ │  │ Shaders │  │  Graph    │
        │ └──────────┘ │  │         │  │           │
        └──────┬───────┘  └───┬─────┘  └─┬─────────┘
               │              │          │
        ┌──────▼──────────────▼──────────▼─────────┐
        │        Shared Memory Arena                │
        │  ┌──────────────────────────────────┐    │
        │  │  SharedArrayBuffer (256 MB)      │    │
        │  │  • CPU View (Wasm memory)        │    │
        │  │  • GPU View (WebGPU buffers)     │    │
        │  │  • NPU View (WebNN tensors)      │    │
        │  └──────────────────────────────────┘    │
        └───────────────────────────────────────────┘
```

## 2. Execution Flow

The execution flow describes how data moves from source code to runtime execution.

### 2.1 Compiler Data Flow

```
Input: hello.omni
      ↓
   [Cache Check] → Cache hit? Return cached output
      ↓ (cache miss)
   [Lexer] → Tokens
      ↓
   [Parser] → AST
      ↓
   [Type Checker] → Typed AST
      ↓
   [Optimizer] → Optimized AST
      ↓
   [Code Generator] → Rust Code
      ↓
   [Rust Compiler] → WASM
      ↓
   [Update Cache] → Store for next time
      ↓
Output: hello.wasm, hello.d.ts, hello.map
```

### 2.2 Runtime Execution Flow

The runtime execution flow details how user interactions trigger changes across the Trinity Engine.

```
USER ACTION (Click/Type)
         │
         ▼
┌─────────────────────┐
│   React Handler     │
│  (TypeScript)       │
└──────────┬──────────┘
           │ 1. Call Wasm function
           ▼
┌─────────────────────┐
│  Trinity Engine     │
│  (Rust/Wasm)        │
│                     │
│  Decision Tree:     │
│  ├─ Simple? → CPU   │
│  ├─ Graphics? → GPU │
│  └─ AI? → NPU       │
└──────────┬──────────┘
           │ 2. Route to appropriate backend
           │
    ┌──────┴──────────────────┐
    │                         │
    ▼                         ▼
┌────────────┐         ┌─────────────┐
│   WebGPU   │         │   WebNN     │
│            │         │             │
│ • Prepare  │         │ • Load model│
│ • Render   │         │ • Inference │
│ • Compute  │         │ • Output    │
└─────┬──────┘         └──────┬──────┘
      │                       │
      │ 3. Write to Shared Memory
      │                       │
      └───────┬───────────────┘
              │
      ┌───────▼────────┐
      │ Shared Memory  │
      │  (Zero-copy)   │
      └───────┬────────┘
              │ 4. Read result
              ▼
      ┌───────────────┐
      │ Trinity Engine│
      │  (Update ECS) │
      └───────┬───────┘
              │ 5. Notify UI
              ▼
      ┌───────────────┐
      │  React Update │
      │  (Re-render)  │
      └───────────────┘
```

## 3. Data Processing Pipelines

Data processing pipelines illustrate how specific tasks like image processing are handled efficiently.

### 3.1 Image Processing Flow (WebGPU + WebNN)

```
┌────────────────────────────────────────────────────────┐
│        IMAGE PROCESSING FLOW (Background Removal)      │
└────────────────────────────────────────────────────────┘

INPUT: User uploads image (1920×1080, RGBA)
  │
  ▼
┌──────────────────────────┐
│ 1. LOAD TO MEMORY        │  Time: ~5ms
│    (JavaScript)          │
├──────────────────────────┤
│ • Create ImageData       │
│ • Copy to Shared Memory  │
│   Offset: 0              │
│   Size: 8.3 MB           │
└────────┬─────────────────┘
         │ Zero-copy reference
         ▼
┌──────────────────────────┐
│ 2. PREPROCESS            │  Time: ~2ms
│    (WebGPU Compute)      │
├──────────────────────────┤
│ Compute Shader:          │
│ • Resize: 512×512        │
│ • Normalize: [0,1]       │
│ • Format: RGB→CHW        │
│ • Output to offset: 8.3MB│
└────────┬─────────────────┘
         │ Zero-copy reference
         ▼
┌──────────────────────────┐
│ 3. AI INFERENCE          │  Time: ~8ms (NPU)
│    (WebNN)               │
├──────────────────────────┤
│ Model: U-Net Segmentation│
│ Input: [1,3,512,512]     │
│ • Encoder layers         │
│ • Bottleneck             │
│ • Decoder layers         │
│ Output: [1,1,512,512]    │
│   (Binary mask)          │
└────────┬─────────────────┘
         │ Zero-copy reference
         ▼
┌──────────────────────────┐
│ 4. APPLY MASK            │  Time: ~1ms
│    (WebGPU Fragment)     │
├──────────────────────────┤
│ Fragment Shader:         │
│ • Read original image    │
│ • Read mask              │
│ • alpha = mask_value     │
│ • Write to output        │
└────────┬─────────────────┘
         │
         ▼
┌──────────────────────────┐
│ 5. UPDATE UI             │  Time: ~0.5ms
│    (React)               │
├──────────────────────────┤
│ • Update state           │
│ • Trigger re-render      │
│ • Show result on canvas  │
└──────────────────────────┘

TOTAL TIME: ~16.5ms (60 FPS capable)
MEMORY COPIES: 0 (all zero-copy)
```

## 4. Technology Decisions

The choice of technologies is driven by performance, safety, and modern web standards.

### 4.1 Compiler: Rust

**Reasons:**
- Memory safety without garbage collection
- High performance for compilation tasks
- Good error messages in ecosystem
- Strong type system
- Excellent tooling (cargo, clippy, rustfmt)
- WASM support is first-class

**Trade-offs:**
- Steeper learning curve
- Longer compile times for compiler itself
- Smaller ecosystem than JavaScript

### 4.2 Runtime: Rust → WASM

**Reasons:**
- Near-native performance in browser
- Predictable performance (no JIT variability)
- Small binary size with good compression
- Growing browser support
- Can share code between compiler and runtime

**Trade-offs:**
- No direct DOM access (must use JS glue)
- Debugging is more complex
- Startup time for WASM module

### 4.3 Build Tool: Custom

**Reasons:**
- Full control over compilation process
- Can optimize specifically for our needs
- Tight integration with compiler
- Better error messages

**Trade-offs:**
- More maintenance burden
- Less community support
- Need to implement common features ourselves

### 4.4 Type System: Custom

**Reasons:**
- Can tailor to our specific needs
- Better error messages for our domain
- Tighter integration with compiler
- Can optimize for our patterns

**Trade-offs:**
- Complex to implement correctly
- Need to design from scratch
- More testing required

## 5. Performance Considerations

Performance is a key driver in architectural decisions, focusing on both compilation and runtime efficiency.

### 5.1 Performance Design Goals

**Compilation Performance:**
- Initial compilation: Target < 500ms for typical component
- Incremental compilation: Target < 100ms for single file change
- Type checking: Target < 50ms for typical file

**Runtime Performance:**
- Memory: Design for reasonable memory usage
- Update speed: Design for efficient reactive updates
- Render speed: Design for efficient rendering

### 5.2 Optimization Opportunities

**Compile-Time Optimizations:**
- Constant folding
- Dead code elimination
- Function inlining (for small functions)
- Common subexpression elimination
- Loop unrolling (for static bounds)

**Runtime Optimizations:**
- Fine-grained reactivity
- Efficient memory layout (ECS)
- Batch updates
- Minimize allocations

## 6. Security & Scalability

Security and scalability are fundamental to the architecture, ensuring safe execution and ability to handle growth.

### 6.1 Security

**Compiler Security:**
- Input validation for .omni files
- Prevent code injection in generated code
- Safe handling of file system operations
- Limit resource usage during compilation

**Runtime Security:**
- Sandboxed execution in WASM
- No eval or dynamic code execution
- Safe memory management
- Validate user input

### 6.2 Scalability

**Parallel Compilation:**
Design for parallel compilation of multiple files using a thread pool and dependency graph traversal.

**Entity Scaling:**
Design considerations for large entity counts include chunked vectors for components and spatial hashing for efficient querying.

**Memory Management:**
Use object pooling for frequently allocated types and custom memory allocators (like BumpAllocator) where appropriate.
