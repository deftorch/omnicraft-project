# OmniCraft Overview

## 1. Introduction

### 1.1 Project Purpose

OmniCraft is designed as a visual content creation platform that explores a compiler-first approach to web development. The project aims to investigate whether compile-time optimizations can deliver meaningful performance improvements over traditional runtime-based frameworks.

### 1.2 Core Concept

The fundamental idea is to shift as much work as possible from runtime to compile-time:

```
Traditional Approach:
Developer Code → Browser → Parse → Execute → Optimize → Render

Proposed Approach:
Developer Code → Compiler → Optimized Code → Browser → Execute → Render
```

### 1.3 Scope

This document outlines the technical design for:
- A component-based syntax (`.omni` files)
- A compiler that transforms declarative code into optimized imperative code
- An incremental compilation system for fast rebuilds
- A lightweight runtime for reactivity and rendering
- Developer tooling for a productive workflow
- Advanced diagnostics and debugging capabilities

---

## 2. Technical Goals

### 2.1 Primary Goals

**Goal 1: Explore Compile-Time Optimization**
- Investigate how much overhead can be eliminated by moving work to compile-time
- Design a compiler that can perform aggressive optimizations
- Implement incremental compilation for fast iteration
- Measure the actual performance impact

**Goal 2: Developer Experience**
- Design an intuitive component syntax
- Provide helpful error messages with suggestions
- Enable fast iteration through incremental compilation
- Support modern IDE features via LSP
- Implement hot module replacement for instant feedback

**Goal 3: Type Safety**
- Implement a type system that catches errors at compile-time
- Provide type inference to reduce annotation burden
- Generate TypeScript definitions for better tooling
- Support union types and advanced type features

### 2.2 Non-Goals

- Not attempting to replace existing frameworks entirely
- Not targeting production use in initial phases
- Not optimizing for framework bundle size initially (focus on generated code)
- Not supporting server-side rendering in v1

### 2.3 Measurable Objectives

Rather than making claims, we'll measure:
- Compilation time (initial and incremental)
- Generated bundle size
- Memory usage at runtime
- Update performance with various entity counts
- Developer time to first working component
- Error resolution time with diagnostics

---

## 3. Architecture Philosophy

### 3.1 Design Principles

**Principle 1: Explicit Over Implicit**
- Make the compilation process transparent
- Show what code is generated
- Provide clear source maps

**Principle 2: Pay-for-What-You-Use**
- Only include runtime code for features actually used
- Dead code elimination by default
- No framework overhead for unused features

**Principle 3: Errors at Compile-Time**
- Catch as many errors as possible during compilation
- Provide actionable error messages with suggestions
- Suggest fixes when possible

**Principle 4: Incremental Complexity**
- Simple components should generate simple code
- Advanced features add complexity only when used
- Clear documentation of what each feature costs

**Principle 5: Fast Iteration**
- Incremental compilation for quick rebuilds
- Hot module replacement for instant feedback
- Preserve state during updates when possible

### 3.2 Trade-offs

| Decision | Trade-off | Rationale |
|----------|-----------|-----------|
| Compile-time optimization | Longer initial compile | Faster runtime, better for iteration with caching |
| Static type checking | More upfront work | Catch errors earlier, better tooling |
| Custom syntax | Learning curve | Better optimization opportunities |
| Rust compiler | Longer build times | Memory safety, performance, WASM support |
| ECS architecture | Different mental model | Better data locality, scalability |
| Incremental compilation | Cache complexity | Dramatically faster rebuilds |
