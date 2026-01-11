# Plugin System

OmniCraft supports a dual-layer plugin system:
1.  **Compiler Plugins (Rust)**: For extending the language, build process, and optimizations.
2.  **Runtime Plugins (TypeScript)**: For extending the editor, UI, and runtime capabilities.

## 1. Compiler Plugins (Rust)

The compiler plugin system allows extending the core compilation pipeline.

### 1.1 Architecture

```rust
pub trait Plugin: Send + Sync {
    fn name(&self) -> &str;
    fn version(&self) -> &str;
    
    // Hooks
    fn on_compile(&self, ast: &mut Component) -> Result<()>;
    fn on_optimize(&self, ast: &mut Component) -> Result<()>;
    fn on_generate(&self, code: &mut String) -> Result<()>;
    
    // Extensions
    fn custom_elements(&self) -> Vec<CustomElement>;
    fn custom_directives(&self) -> Vec<CustomDirective>;
}
```

### 1.2 Capability

#### Compiler Hooks
Plugins can intercept the compilation process at multiple stages:
- **Compile**: Modify AST before type checking.
- **Optimize**: Add custom optimization passes.
- **Generate**: Inject custom code into output.

#### Custom Language Features
Plugins can register:
- **Custom Elements**: New tags like `<markdown>` or `<router-view>`.
- **Directives**: New attributes like `@click` or `v-model`.

### 1.3 Example

```rust
struct AnalyticsPlugin;

impl Plugin for AnalyticsPlugin {
    fn name(&self) -> &str { "analytics" }
    
    fn on_generate(&self, code: &mut String) -> Result<()> {
        code.push_str("\n// Injected by analytics plugin\n");
        Ok(())
    }
}
```

---

## 2. Runtime Plugins (TypeScript)

> **Source:** v6.0 Section 16

The runtime plugin system is designed for the web-based environment, allowing extensions to interact with the Trinity Engine (Wasm/WebGPU/WebNN).

### 2.1 Plugin API Interface

```typescript
// Plugin API Definition

interface PluginAPI {
    // Metadata
    readonly version: string;
    readonly engineVersion: string;
    
    // Core access
    readonly engine: TrinityEngine;
    readonly capabilities: Capabilities;
    
    // Registration
    registerTool(tool: PluginTool): void;
    registerFilter(filter: PluginFilter): void;
    registerExporter(exporter: PluginExporter): void;
    registerAIModel(model: PluginAIModel): void;
    
    // UI Extensions
    addPanel(panel: PluginPanel): void;
    addMenuItem(item: PluginMenuItem): void;
    addToolbarButton(button: PluginToolbarButton): void;
    
    // Events
    on(event: string, handler: Function): void;
    off(event: string, handler: Function): void;
    emit(event: string, data?: any): void;
    
    // Storage
    storage: PluginStorage;
}

interface PluginTool {
    id: string;
    name: string;
    icon: string;
    category: 'draw' | 'select' | 'transform' | 'text' | 'custom';
    
    // Cursor
    cursor?: string;
    
    // Lifecycle
    onActivate?(): void;
    onDeactivate?(): void;
    
    // Events
    onMouseDown?(event: MouseEvent): void;
    onMouseMove?(event: MouseEvent): void;
    onMouseUp?(event: MouseEvent): void;
    onKeyDown?(event: KeyboardEvent): void;
    onKeyUp?(event: KeyboardEvent): void;
    
    // Settings
    settings?: PluginSetting[];
}
```

### 2.2 Plugin Loader Implementation

```typescript
// Plugin Loader & Sandbox

class PluginLoader {
    private plugins: Map<string, LoadedPlugin> = new Map();
    private api: PluginAPI;
    
    constructor(engine: TrinityEngine) {
        this.api = this.createAPI(engine);
    }
    
    async loadPlugin(url: string): Promise<void> {
        try {
            console.log(`Loading plugin from ${url}...`);
            
            // Fetch plugin code
            const response = await fetch(url);
            const code = await response.text();
            
            // Create sandboxed environment
            const sandbox = this.createSandbox();
            
            // Execute plugin code in sandbox
            const pluginFactory = new Function(
                'api',
                `
                'use strict';
                ${code}
                return plugin;
                `
            );
            
            const plugin: Plugin = pluginFactory(this.api);
            
            // Validate plugin
            this.validatePlugin(plugin);
            
            // Call initialization
            if (plugin.onLoad) {
                await plugin.onLoad(this.api);
            }
            
            // Store plugin
            this.plugins.set(plugin.id, {
                plugin,
                url,
                loaded: Date.now(),
            });
            
            console.log(`✓ Plugin loaded: ${plugin.name} v${plugin.version}`);
            
        } catch (error) {
            console.error(`Failed to load plugin from ${url}:`, error);
            throw error;
        }
    }
    
    private createSandbox(): object {
        // Create limited global object
        return {
            console,
            Math,
            Date,
            JSON,
            // No access to: window, document, etc.
        };
    }
}
```

### 2.3 Example: Grid System Plugin

```typescript
// Example Plugin: Advanced Grid System

const gridPlugin: Plugin = {
    id: 'advanced-grid',
    name: 'Advanced Grid System',
    version: '1.0.0',
    author: 'OmniCraft Team',
    description: 'Customizable grid with snap-to-grid functionality',
    
    async onLoad(api: PluginAPI) {
        // Register tool
        api.registerTool({
            id: 'grid-toggle',
            name: 'Toggle Grid',
            icon: 'grid',
            category: 'custom',
            onActivate() { /* ... */ },
            settings: [
                {
                    id: 'gridSize',
                    name: 'Grid Size',
                    type: 'number',
                    default: 20,
                }
            ],
        });
        
        // Add panel
        api.addPanel({
            id: 'grid-panel',
            title: 'Grid Settings',
            icon: 'grid',
            position: 'right',
            render() { return document.createElement('div'); },
        });
    }
};
```
