# Hot Module Replacement (HMR)

## 1. Architecture

### 1.1 Overview

```rust
pub struct HotModuleReplacer {
    /// Cached component state
    state_store: Arc<RwLock<StateStore>>,
    
    /// Module dependency graph
    module_graph: Arc<RwLock<ModuleGraph>>,
    
    /// Connected clients (WebSocket connections)
    clients: Arc<RwLock<Vec<mpsc::Sender<HMRMessage>>>>,
    
    /// Incremental compiler
    compiler: Arc<RwLock<IncrementalCompiler>>,
}

#[derive(Debug, Clone, Serialize, Deserialize)]
pub enum HMRMessage {
    FullReload,
    HotUpdate {
        module_id: String,
        new_code: Vec<u8>,
        state_to_preserve: Vec<String>,
    },
    CSSUpdate {
        css: String,
    },
}
```

### 1.2 State Preservation

The HMR system intelligently preserves component state during updates by tracking signals and scroll position.

```rust
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct ComponentState {
    pub signals: HashMap<String, serde_json::Value>,
    pub scroll_position: (f32, f32),
    pub focus: Option<String>,
}
```

## 2. Client-Side Runtime

The client runtime handles WebSocket messages and applies updates without reloading the page.

```typescript
// runtime/web/src/hmr-client.ts

export class HMRClient {
    private async applyHotUpdate(message: HotUpdateMessage) {
        const { module_id, new_code, state_to_preserve } = message;
        
        // 1. Save current state
        const savedState = this.captureState(module_id, state_to_preserve);
        
        // 2. Load new WASM module
        const wasmModule = await WebAssembly.instantiate(new Uint8Array(new_code));
        
        // 3. Replace old module
        this.modules.set(module_id, wasmModule);
        
        // 4. Restore state
        this.restoreState(module_id, savedState);
        
        // 5. Re-render
        this.triggerRerender(module_id);
    }
}
```

## 3. Propagation Strategy

1. **Detection**: File watcher detects change.
2. **Analysis**: Dependency graph identifies affected modules.
3. **Decision**:
    - If root changed or >10 files affected → Full Reload.
    - Otherwise → Hot Update.
4. **Broadcast**: Update sent to connected clients.
