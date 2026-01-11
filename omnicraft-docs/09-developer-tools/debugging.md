# Debugging & Source Maps

## 1. Source Maps

### 1.1 Generation Strategy

OmniCraft generates V3 source maps to map generated Rust/WASM code back to the original `.omni` source files.

```rust
pub struct SourceMapGenerator {
    file: String,
    sources: Vec<String>,
    mappings: Vec<Mapping>,
}

pub struct Mapping {
    pub generated_line: u32,
    pub generated_column: u32,
    pub source_file: String,
    pub source_line: u32,
    pub source_column: u32,
}
```

### 1.2 Mappings

The compiler tracks line mappings during code generation, ensuring that:
- Runtime errors point to the `.omni` line.
- Breakpoints work in original source.
- Variable inspection uses original names where possible.

## 2. Time-Travel Debugging

### 2.1 DevTools Integration

The runtime exposes a global API for time-travel debugging:

```typescript
(window as any).omnicraft = {
    // Time-travel
    travelTo: (index: number) => void,
    travelBack: () => void,
    travelForward: () => void,
    
    // Snapshots
    snapshot: () => Snapshot,
    snapshots: () => Snapshot[],
};
```

### 2.2 Snapshot System

Snapshots capture the entire application state at a frame:
- **State**: All signal values.
- **DOM**: Canvas visual state.
- **Performance**: Memory and FPS metrics.

This allows developers to rewind the application to any previous state to investigate bugs.
