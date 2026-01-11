# OmniCraft Project Documentation

Welcome to the official documentation for **OmniCraft** — a next-generation visual content creation platform exploring a compiler-first approach to web development.

This documentation details the **Trinity Architecture** (v6.0), integrating CPU (Wasm), GPU (WebGPU), and NPU (WebNN) into a unified high-performance runtime.

## Quick Links

- [**System Architecture**](02-architecture/system-architecture.md) - The high-level design and Trinity Engine.
- [**Zero-Copy Design**](05-web-trinity/zero-copy-design.md) - How we achieve shared memory across compute units.
- [**Compiler Pipeline**](03-compiler/overview.md) - From `.omni` syntax to optimized Wasm.

---

## 01. Overview
- [Introduction & Goals](01-overview/README.md)

## 02. Architecture
- [System Architecture](02-architecture/system-architecture.md)
- [Trinity Architecture (v6.0)](02-architecture/trinity-architecture.md)
- [Memory Architecture](02-architecture/memory-architecture.md)

## 03. Compiler
- [Compiler Overview](03-compiler/overview.md)
- [Optimization Pipeline](03-compiler/optimization-pipeline.md)

## 04. Runtime
- [ECS Design & Data Model](04-runtime/ecs-design.md)
- [Reactivity System](04-runtime/reactivity.md)

## 05. Web Trinity Engine (v6.0)
- [Zero-Copy Design](05-web-trinity/zero-copy-design.md)
- [WebGPU Pipeline](05-web-trinity/webgpu-pipeline.md)
- [WebNN AI Pipeline](05-web-trinity/webnn-pipeline.md)
- [Fallback System](05-web-trinity/fallback-system.md)

## 06. Advanced Features
- [Plugin System](06-advanced-features/plugin-system.md)
- [Advanced Optimizations](06-advanced-features/advanced-optimizations.md)
- [Platform Adapters](06-advanced-features/platform-adapters.md)
- [Accessibility](06-advanced-features/accessibility.md)

## 07. Implementation Strategy
- [Development Phases](07-implementation/development-phases.md)
- [Configuration](07-implementation/configuration.md)
- [Testing Strategy](07-implementation/testing-strategy.md)
- [Error Handling](07-implementation/error-handling.md)
- [Observability](07-implementation/observability.md)

## 08. Build & Deployment
- [Build System (Vite/Cargo)](08-build-and-deployment/build-system.md)

## 09. Developer Tools
- [CLI Reference](09-developer-tools/cli-reference.md)
- [Runtime Monitoring](09-developer-tools/runtime-monitoring.md)
- [Debugging](09-developer-tools/debugging.md)
- [HMR System](09-developer-tools/hmr-system.md)
- [LSP Implementation](09-developer-tools/lsp-implementation.md)

## 10. API Reference
 *(Placeholder for API docs)*

## 11. Appendices
 - [Glossary](11-appendices/glossary.md)
