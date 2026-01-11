[← Back to Index](../README.md)

# Glossary

This glossary defines core terminology used throughout the OmniCraft documentation.

## A

### AST (Abstract Syntax Tree)
A tree representation of the abstract syntactic structure of source code written in a programming language. The OmniCraft compiler transforms `.omni` files into an AST before optimization.

## C

### Component
A reusable, self-contained building block in OmniCraft. Components are defined in `.omni` files and can contain logic (Script), view (Template), and styling.

## E

### ECS (Entity Component System)
An architectural pattern used in the Trinity Runtime where:
- **Entities** are unique IDs.
- **Components** are raw data attached to entities.
- **Systems** are logic that processes entities with specific components.
This pattern ensures high performance and memory locality.

## I

### Incremental Compilation
A compiler feature that only recompiles the parts of the program that have changed, rather than the entire codebase. This significantly speeds up the development feedback loop.

## T

### Trinity Architecture (v6.0)
OmniCraft's unified runtime architecture that orchestrates workloads across three compute units:
1.  **CPU (Wasm):** Handles logic, orchestration, and ECS management.
2.  **GPU (WebGPU):** Handles graphics rendering and massive parallel compute tasks.
3.  **NPU (WebNN):** Handles AI inference and machine learning workloads.

### Trinity Engine
The core runtime engine written in Rust and compiled to WebAssembly that manages the Trinity Architecture.

## W

### Wasm (WebAssembly)
A binary instruction format for a stack-based virtual machine. OmniCraft compiles user code into Wasm for near-native performance in web browsers.

### WebGPU
A modern web API that exposes the capabilities of the device's GPU for both graphics and computation.

### WebNN (Web Neural Network API)
A web API dedicated to hardware-accelerated neural network inference.

## Z

### Zero-Copy
A data transfer pattern where memory is shared directly between different processing units (CPU, GPU, NPU) without duplication. OmniCraft uses a Shared Memory Arena to achieve this, reducing latency in high-performance pipelines.
